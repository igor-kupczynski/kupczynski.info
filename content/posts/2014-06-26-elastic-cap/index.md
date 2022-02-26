---
title: CAP Theorem and Elasticsearch
tags:
- elastic
aliases:
- /2014/06/26/elastic-cap.html
---
Will it fail together with the network?

Let us check the Elasticsearch behavior in context of the CAP theorem.

CAP Theorem
===========

[CAP theorem](http://en.wikipedia.org/wiki/CAP_theorem) stays that a
distributed system communicating over an asynchronous network can't
provide these three properties at once:

Consistency
: A read sees all previous writes;

Availability
: Reads and writes succeed on all nodes;

Partition tolerance
: These properties stay intact even in case of a network failure of
some of the communication links between nodes.


This theorem is often used (and misused), but luckily there are some
good clarifications over the Internet (see e.g. [1][fdb], [2][chale]
or even
[plain english, yet sexist and overly simplified explanation][plain]).

[fdb]: https://foundationdb.com/key-value-store/white-papers/the-cap-theorem
[chale]: http://codahale.com/you-cant-sacrifice-partition-tolerance/
[plain]: http://ksat.me/a-plain-english-introduction-to-cap-theorem/


Call me Maybe
=============

Recently, a colleague of mine has sent me an article where the author
tests Elasticsearch through CAP theorem lenses. The article,
*[Call me maybe: Elasticsearch](http://aphyr.com/posts/317-call-me-maybe-elasticsearch)*,
written by [Aphyr](http://aphyr.com/) is a part of the
[Jepsen](http://aphyr.com/posts/281-call-me-maybe-carly-rae-jepsen-and-the-perils-of-network-partitions)
series. Aphyr tests how various databases and distributed systems
behave in case of network failures. It is worth adding that the blog
posts are very detailed and technical and, moreover, fun to read :-).

Quite frankly, I was terrified. The bottom line is that in case of
various network failures Elasticsearch stays available, but
inconsistent. I.e. clients do not see all of the acknowledged writes
(not to mention updates, etc.).

Since network partitions are unavoidable, then according to CAP
theorem, once can only build CP or AP system. Looks like in case of
Elasticsearch then AP decision was not conscious. E.g. Shay Banon (the
creator of Elasticsearch), says this on its
[mailing list](http://elasticsearch-users.115913.n3.nabble.com/CAP-theorem-td891925.html):

> I personally believe that "within the same data center", network
> partitions very rarely happen, and when they do, its a small set
> (many times single) machine that gets "partitioned out of the
> network". When a single machine gets disconnected from the network,
> then thats not going to affect elasticsearch.

The documentation is not very verbose on Elasticsearch CAP properties
and various people give different explanations on the mailing
list. According to Aphyr tests it sometimes acts as AP (transitive
partitions), usually it acts as CP (because of the minimum master
nodes settings) and sometimes it just drops some writes, but it is not
available (so we have only P :-)).


Ye've Been Warned
=================

[Some people](https://groups.google.com/forum/#!msg/elasticsearch/OIZzjgIWA-I/4u75TGtJRWEJ)
say that you can use Elasticsearch as a primary data source. Don't do
it. As proven in the linked article it can lost your data.

Luckily, there is solace to us working with Elasticsearch on
production systems. First of all, you use it to index your data. The
data are consistent and safe in your authoritative database. In case
of an inconsistency you just reindex. Secondly, Aphyr's post made Shay
and the Elastisearch folks reconsider their system behaviour and fix
(at least some of the problems). The fix is already on trunk (pls see
[this issue](https://github.com/elasticsearch/elasticsearch/issues/2488)).

I think that Elasticsearch is a great tool for its job (you know,
for search). Big thumbs up for Shay and his crew and also for the
community. As you can see from the linked issue, a good bug report
with a set of tests can improve the project greatly.
