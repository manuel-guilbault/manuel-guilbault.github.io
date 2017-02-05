---
layout: post
title: "How to use a message bus, and how not to"
tags: [Design, .NET]
image:
  feature: 2017-02-06-How-to-use-a-message-bus-and-how-not-to/Letters.jpg
  teaser: 2017-02-06-How-to-use-a-message-bus-and-how-not-to/Letters.jpg
published : true
---

A message bus is an extremely useful piece of technology. There are
many implementations available out there: [RabbitMQ](https://www.rabbitmq.com/){:target="_blank"},
[ActiveMQ](http://activemq.apache.org/){:target="_blank"}, 
[MSMQ](https://msdn.microsoft.com/en-us/library/ms711472(v=vs.85).aspx){:target="_blank"},
or [Azure Service Bus](https://azure.microsoft.com/en-us/services/service-bus/){:target="_blank"}, 
to name a few. However, their usage seems to be misunderstood by many developers.

Sometime last week, a couple of collegues and I were discussing what we envisioned
for the system's architecture in the mid-long term. Some of them brought up
introducing a message bus to enable asynchronous processing.
The way my teammates seemed to see it, a message bus would magically fix
performance problems. I disagreeed.

## The context

We work together on a reservation system for a world-leading tour operator. The
system is built using .NET technologies and exposes web APIs allowing various web 
sites to search for flights and to book seats, along with a back office allowing
the company's staff to manage flights, capacity and pricing.

The system's architecture follows a pattern which is pretty common in old .NET projects
and which I call a *layered monolith*. Layered, because the code is sliced in
layers:

![Layers]({{ site.baseurl }}/images/posts/2017-02-06-How-to-use-a-message-bus-and-how-not-to/Layers.svg)

Monolith, for two very distinct reasons. First, the classes making those layers are 
most of the time tightly coupled together, meaning that it is impossible to change the
composition of the system without changing the code. Second, the boundaries between
the functional domains are sometimes blurry, often unexistant. The system is
a [Big Ball of Mud](http://www.laputan.org/mud/){:target="_blank"} in all of its glory.

## The bus layer

What my collegues had in mind when they proposed a messaging bus was something in the like:

![The bus layer]({{ site.baseurl }}/images/posts/2017-02-06-How-to-use-a-message-bus-and-how-not-to/The-bus-layer.svg)

With this amended architecture, the presentation layer would send operations to the
message bus. Additionally, some dispatcher process would monitor the bus and would
dispatch all received operations to the proper service on the business layer.

The initial argument for this pattern is that scalability could be boosted by adding
more instances of the dispatcher process, so more asynchronous operations could
be processed in parallel.

Let's see the implicit problems of this solution.

## Not all operations are commands

The system we work on globally follows the [CQRS](https://en.wikipedia.org/wiki/Command–query_separation){:target="_blank"}
principles: operations are either commands or queries. Commands are used to change 
the system's state, while queries simply retrieve some part of the system's current state and
are side-effect free.

While asynchronism can work for commands, it doesn't work for queries, because 
they follow a request-response flow: a request querying for some data is received 
by the system, and it must respond with the proper data.

With the amended architecture, information can only flow down from the presentation 
layer. There is no way to get responses from the business layer. In order to
enable such a thing, we would need to add a second bus for responses:

![The bus layer]({{ site.baseurl }}/images/posts/2017-02-06-How-to-use-a-message-bus-and-how-not-to/Response-bus.svg)

This means that the presentation layer, upon receiving a query, will first send
a message on the bus, and will monitor the second bus, waiting for a response
message.

This is completely useless. It adds a lot of complexity, and the presentation layer 
will still block while waiting for a response, because such queries are implicitely 
synchronous operations (don't forget that here, the presentation layer is a web API).

Of course, the argument stating that it can scale more easily by adding more
dispatching processes still stands. However, the same is true by simply adding 
instances of the web API itself, and such a solution is much simpler than this
amended architecture. Additionally, it requires almost no changes in the existing
architecture.

## Not all commands are fire-and-forget

We could say that queries would bypass the whole bus layer, and would be sent
synchronously by the presentation layer to the business layer, and that commands
would be sent asynchronously through the bus layer.

![Bypasing the bus layer]({{ site.baseurl }}/images/posts/2017-02-06-How-to-use-a-message-bus-and-how-not-to/Bypassing-the-bus-layer.svg)

However, not all commands are fire-and-forget. Some commands can be executed
only when the system is in a specific state. Such commands must be validated
before they can be executed to make sure they don't violate any system invariant.
If the validation process fails, the caller - may it be a user or another system - 
usually needs to be notified that the command failed and, more importantly,
why it failed.

We now go back to the same problem we had with queries: information must flow
both ways.

## When is a message bus relevant?

For me, a message bus really shines when it is used to let other parts of
the system - or even other systems - know that something *just happened*.

A command's side effects can be asynchronously applied only if they support 
[eventual consistency](https://en.wikipedia.org/wiki/Eventual_consistency){:target="_blank"}. 
If a command's side effect must be in the same logical transaction as its command, 
it can't be processed asynchronously.

This forces developers to explicitely define the system's commands, their
side effects, and the consistency model of each side effect.

## Summary

A message bus is not a silver bullet. It adds complexity to a system, and
must be used when it adds real value. In short:

* Don't use it as an over-complex call stack
* Use it to notify the outside world (other systems or bounded contexts) that something *already* happened
* Use it for side-effects that can be eventually consistent
