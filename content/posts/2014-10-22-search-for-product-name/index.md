---
title: Searching for Product Name in Elasticsearch
tags:
- elastic
aliases:
- /2014/10/22/search-for-product-name.html
---
How to implement good search on product name in Elasticsearch.

A lot of elasticsearch clusters will have a usecase of searching for
product name. It doesn't really matter if the products are consumer
goods, articles or files. The important thing is that users what to
search by product name and find matching items. The products should be
found if a user types their exact name or just type something close
enough. In the post I'll describe a possible implementation of this
usecase.


# File name indexing

Let's assume we deal with file names. We have a file named
`TheSmallYellowDog.txt`. If a user types `small` or `yellow` or `dog`
then we want to find it. We also what to find it when she types part
of the name or a prefix. We'll do the following:

#### 1. Tokenize into words

    TheSmallYellowDog.txt --> TheSmallYellowDog.txt

In our case there is no whitespace, so this will be a single token.

#### 2. Split words into subwords

    TheSmallYellowDog.txt --> The Small Yellow Dog txt


#### 3. Normalize the case

    The Small Yellow Dog txt --> the small yellow dog txt


#### 4. Remove the stopwords

    the small yellow dog txt --> small yellow dog txt


As a result we'll have the following tokens in the index: `small`,
`yellow`, `dog`, `txt`.

The query on this field will go through the same processing steps, so
if a user types `SmaLL` this will be changed to `small` and match the
token in our inverted index.

## Implementation

#### 1. Tokenization

Bulk of our documents have English names -- for them a whitespace
based tokenization is enough. In our corpus we have also CJKT
(Chinese, Japanese, Thai, Korean) named files. CJKT languages often
use no spaces between words.

Example: `Hello. I'm from Bangkok.`, in Thai `สวัสดี
ผมมาจากกรุงเทพฯ`. Although there is just one whitespace, this should be
split into following words `สวัสดี, ผม, มา, จาก, กรุงเทพฯ`.

Because of this requirement, we need to use `icu_tokenizer` instead of
the regular one. This is available as a plugin for elasticsearch -
[icu-plugin][icu].

#### 2. Splitting the word into subwords

People quite often name their files without using whitespaces, so step
1 is not enough to meet our requirements; for article titles or other
products this may not be needed. There is a ready made solution
provided by elasticsearch - the
[word delimiter token filter][wordsplit]. It splits *single* words
into subwords based on few rules, like non-alphanumeric characters,
case transitions, etc. We can configure it in
`settings.analysis.filter` section of the index mapping.


    "word_split": {
        "type": "word_delimiter",
        "preserve_original": 1
    }


#### 3. Normalize the case

We use `icu_folding` filter from the `icu_plugin`. It not only
lowercases, but also *folds* national characters to their basic form
(e.g. Polish letter with accents to their unaccented forms `ą --> a`,
`ń --> n`).

No configuration is needed here.

#### 4. Stopwords

This step may not be needed for you, but since bulk of our corpus are
documents with English names we apply english stopwords dictionary
from elasticsearch.


    "english_stop": {
        "type": "stop",
        "stopwords": "_english_"
    }


## Putting it all together

We have configured a set of filters, but to make it work we need to
define an analyzer based on the filters (`settings.analysis.analyzer`).


    "generic_name_analyzer": {
        "type": "custom",
        "tokenizer": "icu_tokenizer",
        "filter": [
            "word_split",
            "icu_folding",
            "english_stop"
        ]
    }


And apply it on the field we want to index this way (`mappings.type`).

    "fileName": {
        "type": "multi_field",
        "fields": {
            "fileName": {
                "type": "string",
                "analyzer": "generic_name_analyzer"
            }
        }
    }


Now we can use a simple match query to find our document. Example:


    curl -XGET localhost:9200/idx-name/type/_search?pretty=true -d `
    {
        "query": {
            "match": {
                "fileName": "yellow"
            }
        }
    }
    `

Although this works nice for most of the cases, we still need to solve
the issue with prefix or fuzzy searches.

# Fuzzy search

For fuzzy search we can use ngrams. Ngrams are basically pieces of
words obtained by sliding a window of a certain length on each
word. For example *3-3-grams*, or *trigrams* if you will.


    nice weather we have today --> nic ice   wea eat ath the her    hav ave   tod oda day
    yellow --> yel ell llo low


By applying trigrams to both the indexed tokens and the query we can
achieve a fuzzy match. If the file name is `yellow` and the query is
`yellowish`, then we have the following


    File  [yellow   ]: yel ell llo low
    Query [yellowish]: yel ell llo low owi wis ish


Most of the trigrams where matched, so we can assume that we have a
hit.

Let us configure the trigram filter and analyzer.

    "trigram_filter": {
        "type": "ngram",
        "min_gram": 3,
        "max_gram": 3
    }
    (...)
    "trigram_name_analyzer": {
        "type": "custom",
        "tokenizer": "icu_tokenizer",
        "filter": [
            "icu_folding",
            "english_stop",
            "trigram_filter"
        ]
    }


We also have to extend our multifield to use the trigram analyzer.

    "fileName": {
        "type": "multi_field",
        "fields": {
            "fileName": {
                "type": "string",
                "analyzer": "generic_name_analyzer"
            },
            "trigram": {
                "type": "string",
                "analyzer": "trigram_name_analyzer"
            }
        }
    }


We have the file name indexed in a regular way as `fileName` (this is
a shortcut for `fileName.fileName`) and as trigrams
`filename.trigram`.

There is a small performance impact when indexing trigrams. Also, the
inverted index with trigrams takes more space on disk. In my case
there was only a marginal penalty to apply trigrams to the file
metadata, but you should check with your own document corpus; I'd
expect that if you want to index long fields like article contents or
product descriptions this way, the price to pay may be higher.

# Query

We can use the following query to find our files

    curl -XGET localhost:9200/idx-name/type/_search?pretty=true -d `
    {
        "query": {
            "bool": {
                "should": [
                    {
                        "match": {
                            "fileName": {
                                "query": "yellow",
                                "boost": 3
                            }
                        }
                    },
                    {
                        "match": {
                            "fileName.trigram": {
                                "query": "yellow",
                                "minimum_should_match": "50%",
                                "boost": 1
                            }
                        }
                    }
                ]
            }
        }
    }
    `

To sum up, searching by product name is a common usecase. In this
fast-paced post I showed how to implement it in elasticsearch. This
was just a basic implementation and there are a lot of parameters to
tune (bigrams vs trigrams vs 2-20-grams, tokenizers, stop words,
stemmers, etc.). I encourage you to experiment with elasticsearch to
get better results for your corpus and requirements.


# Full example

You can experiment with the whole example on [found.no/play][play] or
locally on the gist below.

<script src="https://gist.github.com/puszczyk/383ea2b1cbdb21d41e62.js?file=play.sh" type="text/javascript"></script>


[icu]: http://www.elasticsearch.org/guide/en/elasticsearch/guide/current/icu-plugin.html
[wordsplit]: http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/analysis-word-delimiter-tokenfilter.html
[play]: https://www.found.no/play/gist/383ea2b1cbdb21d41e62
