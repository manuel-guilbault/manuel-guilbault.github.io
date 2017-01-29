---
layout: post
title: "Properly using a message bus"
tags: [Design]
image:
  feature: sewing.jpg
  teaser: sewing-teaser.jpg
  credit:
  creditlink:
published : false
--- 

Message queues - or buses - are an extremely useful piece of
technology. However, their usage seem to be sometimes misunderstood.

I had a discussion with some collegues about adding a message bus to
enable asynchronous processing to the system we are working on. The
way my teammates seemed to see it, a message bus would magically fix
load problems. I disagreeed.

We work on a reservation system for a world-leading tour operator. This
system exposes web APIs allowing various web sites to search for
flights and to book seats. This system's architecture is a monolith,
layered as such:

* Web API
* Business logic
* Data access

What they had in mind was something in the like:

* Web API
* **Message queue**
* Business logic
* Data access

The problem with that new architecture is that some operations need to
be synchronous. For example, when a website sends a booking request, 

* Using it in cases when the caller waits synchronously for the response
  makes the queue act like an over-complex call stack...
* Use it to notify the outside world (outer bounded contexts) that something *already* happened
* Use it for side-effects for which eventual consistency is okay