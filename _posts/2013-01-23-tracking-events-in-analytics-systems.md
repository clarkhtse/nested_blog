---
layout: post
title: Tracking events in analytics systems
published: true
---

Lately, I have been working on ShopStream, an analytics-as-a-service for online stores. I would like to reflect on the choices I made and the things I learned during building it.

The terms I use in the article:

* analytics systems — systems that allow to track and vizualize the data (sometimes third-party, sometimes not)
* metrics — some numbers computed for the tracked data, that give some sense of that set of data. "Average purchase" for a specific time period for a shop is a nice example of that

From the standpoint of the users of analytic systems, there is only one thing to be done to enjoy their numbers, charts, and so on — that thing is to track events they are interested in. New page load, item added to the cart, check out.

As I have explored, there are basically two options of tracking those events at programmatic level.

### "Why do we need the original payload"

It is so simple to update metric values right after the event was sent to be tracked. As simple as `db.metrics.update({shop_id: 1, year: 2013, month: 1, day: 10, hour: 10}, {$inc: {pageviews: 1} })`. No need to store actual event after updating the metrics.

As anything, this approach comes with its upsides and downsides. 

You benefit by:

* having smaller DB and index size. There is no raw event data to be stored and indexed
* having metric value as a field in your document, so simple
* having generally better performance

But there are a plenty of things where you loose, namely:

* if you add a new kind of metric that you system calculates, it is calculated only for new events
* there is quite smaller number of ways you can analyze your data. You just have numbers over there, you can't do anything with already tracked data — because there is no actual data

True, it works just nicely in some cases. One example of that are systems which track data that is actual for short periods of times — systems to track average server load, to record your logs and similar.

But there are cases when you need something more sophisticated — when you want to keep track of everything that ever happened and be able to analyze all of that. There is a solution for that. 

### Always. Save. Raw. Event. Data

As the section title implies, it is all about tracking original event payloads and storing all of them.

You benefit by:

* having access to all the event data
* being able to do so many things to dissect them
* having any of your new metric make use of all the data that was ever collected

Storing all the events is not always a good idea in all cases. It is helpful if you, say, track shop's purchases (as a 3rd party) — you could analyze the order events to compute top sold products for any period you'd like. You just have a bunch of events, you don't have to track each of them at different granularity levels (e.g. # of requests in this second/hour/day/month/year), like the first approach does. You could even apply some machine learning on those sets of data to predict profitable seasons and stuff like that.

Yes, there are downsides:

* you have to store **all** the data. If you track zillions events an hour, that could result in hilarious size of your database & indexes
* processing all of the data when you need to compute some number on set of those events could be particularly slow on huge datasets. There are technology & techniques to make it faster, I'll get more into it in my next blog post

There is no single "right" way after all. It's all about trade-offs — deciding what is important for *your* system and what can be left off in favor of something else.

Speaking of ShopStream — we have decided to take the latter approach. We are constantly changing our views on what metrics we need and are planning on some really neat features that require events being present in their raw view.

I'm planning on writing a follow-up post on processing tracked events. Keep posted.
