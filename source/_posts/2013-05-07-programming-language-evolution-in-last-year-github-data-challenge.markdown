---
layout: post
title: "Programming language evolution in last year"
date: 2013-05-07 12:00
comments: true
---

[The second github data challenge](https://github.com/blog/1450-the-github-data-challenge-ii)
has been announced for a while. Github records every
[events](http://developer.github.com/v3/activity/events/types/)
happened and opens it to the public as a
[service](http://www.githubarchive.org/). It is a big data archive:
last year there is up to 16G, and people have made a lot of
[fun](https://github.com/blog/1118-the-github-data-challenge) with it.

The sheer amount of events data seems be able to answer a lot of
questions and reveal some interesting stuff. I wonder what I can find
out from these events, something at least not boring. Finally I came
up with this question: how programming languages evolve in the last
year?

Why do I choose this question? The main reason of it is probably that
it is simpler to analyze those data from a language point of
view. There are a lot of information in each event record. You
typically can find out who did this event, what repository is related
to this event, and possibly the personal information such as location,
company of the actor if they ever input those information when they
created their github account. If the event has something to do with a
repository, you can also get information such as language, owner,
description of the repository; there are also event specific data
(e.g. an `IssuesEvent` has some issue specific data). So it is
possible to analyze those events data by when(time), where(people's
locations) and what(events and its related information). However the
location of an event is not always available, so a location based
analysis will suffer from incompleteness of data. Fortunately most
events are repository related(e.g. `CreateEvent`, `PushEvent`,
`WatchEvent` etc.), and nearly all these events are marked with a
language, so the information is abundant if we want to know how
language changes with time.

To complete this work I used `ruby`, `JavaScript` and `R` languages,
bit of shell script to download events data from github
archive. [`d3`](http://d3js.org) is used to draw the alluvial diagrams
shown below. The work is pushed to
[my github repository](https://github.com/exceedhl/github.data.challenge).

## Data preparation

Github archive service exposes events data by hours, so in order to
get all data from the last year, I have to download data of every hour
in the last year. The download process takes me several hours and the
data eats 16G of my disk space.

Then I extracted the counts of every event type for all languages and
every month using a `ruby` script and `yajl` gem. It takes less one
hour to extract the csv file I want. `yajl` is a pretty fast library.

The rest of work is handed over to `R`. Once I read the csv file into
`R`, I filter out those event types with zero count, which leaves me
14 types events(`CommitCommentEvent`, `CreateEvent`, `DeleteEvent`,
`DownloadEvent`, `ForkEvent`, `GollumEvent`, `IssueCommentEvent`, `IssuesEvent`,
`MemberEvent`, `PublicEvent`, `PullRequestEvent`,
`PullRequestReviewCommentEvent`, `PushEvent`, `WatchEvent`). So finally I
got a `data.frame` with 16 columns (14 event types plus language and
month), in which each row represents a language's events count in some
month.

## Analysis

Taking a closer look at our initial question, I basically want to know
how active a language is comparing with others, how their activity
changes with time. Since every language has 14 type of events for each
month, I choose to compute the magnitude of this 14 dimension vector
as the indicator of overall activity of each language each month.

In each month, we can also do clustering analysis of those languages,
so that we learn that some languages have similar activity and how
these groups change by time. Here I perform a
[principal component analysis](https://en.wikipedia.org/wiki/Principal_component_analysis)
of the 14 dimension vectors then use basic
[kmeans](http://en.wikipedia.org/wiki/K-means_clustering) clustering
algorithms to do the clustering analysis. The principal component
analysis can reduce the dimension of those vectors to 2 and still
reserve almost all variability. The reduced 2 dimension vectors are
good for visual representation later on.

By clustering the languages each month and compute their overall
activity, we can draw the following alluvial diagram:

<div id="alluvial"></div>

<script type="text/javascript" src="/javascripts/d3.v3.min.js"></script>
<script type="text/javascript" src="/javascripts/alluvial.js"></script>

In the above diagram, each column is a month, and languages are
grouped and separated by empty line. Groups are ordered by their
maximum overall activity, and languages in each group are ordered by
their overall activity.

There are a few interesting points revealed by this diagram:

* JavaScript is the most active language in the last year, and it has
  a significant lead comparing with other languages because it is
  never grouped with any other languages.
* Java, Ruby, Python and PHP are secondly active languages, C and C++
  are less active than them. In June, July, August and September Ruby
  took a lead in this group and Java took it back in the last 7 months.
* Shell is more active than Objective-C in a few months and less
  active in other months. I am not sure if it is because continuous
  delivery was very hot in last year.
* VimL is the 13th most popular language on
  github(https://github.com/languages/VimL). The diagram reflects this
  point and in most cases it is more active than CoffeeScript and
  perl, sometimes Scala.
* AutoHotKey became very active in 2012 August, similarly Nemerle's
  activity peaked in this February. By examining the data we know
  there are 10184 `IssuesEvent` in 20013 August for Nemerle, which is
  extraordinarily high considering its normal performance.
* The number of languages are increasing, especially in the last two
  months, github has over 10 more languages in 2013 March.
  
There are probably more interesting stuff you can find out of the
result diagram, at least by doing analysis I learned a lot of
languages that I have never heard before. The diagram is not perfect
and I hope it is not too bad.

For those who are interested in the clustering analysis, I also post
the 2 dimension clustering plot of those languages in each month
below. The plot is generated by running PCA dimension reduction then
kmeans clustering.

<object data="/images/cluster.svg" type="image/svg+xml"></object>

## Conclusion

Unfortunately there are not many firm conclusions that I can draw from
this exercise. In short, we have a lot of languages on github and the
number is still increasing. JavaScript is the hottest language in the
last year, taking a huge lead among all languages. If you have any
thoughts drop me a line here.
