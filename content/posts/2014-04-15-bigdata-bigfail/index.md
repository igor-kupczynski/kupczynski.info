---
title: Near real-time analytics
tags:
- fail
aliases:
- /2014/04/15/bigdata-bigfail.html
---
Big data = big fail?

Martin Fowler recently summarized a "Reporting Database" pattern. The
core idea behind this pattern is to split analytical and transactional
processing by providing a separate database for the former. A separate
database for reporting purposes is nothing new and this pattern is
widespread in the *enterprise* space. What caught my interest though
was his opinion on near real-time analytics.

> These days the desire seems to be for near-real time analytics. I'm
> skeptical of the value of this. Often when analyzing data trends you don't
> need to react right away, and your thinking improves when you give it time
> for a proper mulling. Reacting too quickly leads to a form of information
> hysteresis, where you react badly to data that's changing too rapidly to get
> a proper picture of what's going on.
[Source](http://martinfowler.com/bliki/ReportingDatabase.html)

I think that Martin Fowler raised an interesting point here. There is
a lot of hype around big data lately. There are tools to store and
query terabytes of data and algorithms to analyze them. Big data
offers a big promise that we can gain new insights from the data we
already have in an easy and cost-effective way. To give one example
here is a link to
[China Mobile case study](http://strata.oreilly.com/2013/07/near-realtime-streaming-and-perpetual-analytics.html).

Unfortunately, big data seems not to be a silver bullet. Recent failures
(e.g.  google flu trends
[\[1\]](http://www.nature.com/news/when-google-got-flu-wrong-1.12413)
[\[2\]](http://commonhealth.wbur.org/2013/01/google-flu-trends-cdc))
shows that one can't blindly apply algorithms and tools in a hope
that they will reveal some hidden knowledge. A proper model and good
domain knowledge are important ingredients of a successful big data
system and one should think before blindly jumping on this bandwagon.
