---
layout:     post
title:      Using Multiple Subscribers with OpenMAMA ZeroMQ
date:       2015-12-05 15:00:00
summary:    To fan out, or to delegate
thumbnail:  cogs
tags:
 - OpenMAMA
 - ZeroMQ
 - Scons
 - Architecture
 - Fault Tolerance
---

A question came up on the ZeroMQ bridge recently and when it did, I realized
that I had never actually tried it before, but thankfully with a small change,
fanout to multiple clients using ZeroMQ could be supported just fine. After
this change, there are a few options which I'll outline in this blog post.

### Using an intermediary

An intermediary is effectively [a forwarder](http://zguide.zeromq.org/page:all#ZeroMQ-s-Built-In-Proxy-Function) -
a lightweight proxy (note that you need XSUB / XPUB rather than ROUTER / DEALER in that code) which is
something that you typically compile yourself, but literally all the code is
required to write one is in that link.

If you compile the code from that link using XSUB / XPUB, you'll have an
intermediary which:
* Is going to subscribe to everything over TCP port 5559
* Is going to publish everything over TCP port 5560

So how do you connect to this intermediary? It's actually fairly straightforward -
simply point every single node on the messaging bus to it. In mama.properties
for the zeromq bridge, it would look like this:
    mama.zmq.transport.broker.outgoing_url=tcp://localhost:5559
    mama.zmq.transport.broker.incoming_url=tcp://localhost:5560

This is always what I had in mind when I originally wrote the ZeroMQ bridge,
because I remember the pain that I had trying to get point to point fanout
to work well with Qpid Proton without a broker, and a lightweight stateless
proxy will always be a lot faster than a fully featured broker.

It's also a nice benefit that using an intermediary makes the job of configuring your
messaging infrastructure a **lot** easier for most deployments, particularly when
it comes to having multiple publishers.

### Cutting out the middleman

If you want to connect multiple consumers to a small number of publishers, you can
try connecting straight to the publisher. You can do this by adopting a similar
approach to the intermediary where both publisher and subscriber ports are binded
on the publisher side:
    mama.zmq.transport.fanout.outgoing_url=tcp://*:5556
    mama.zmq.transport.fanout.incoming_url=tcp://*:5557

Then on each client, you can re-use the same transport settings:
    mama.zmq.transport.client.outgoing_url=tcp://localhost:5557
    mama.zmq.transport.client.incoming_url=tcp://localhost:5556

### Why does both client and publisher subscribe and publish?

In OpenMAMA, subscriptions involve messages as well as simply opening up communication
channels, so subscribing to data involves publishing as well as consuming. The same
principle applies to request / reply so you always need both an outgoing and incoming
data stream.

### Won't this mess up ZeroMQ's Request / Reply pattern?

I don't actually use the ZeroMQ request / reply mechanism for subscriber-driven recovery.
Instead, a recovering / new subscriber will subscribe to a UUID based topic, then 
publish a request for initial / recap out to a known topic which will contain this UUID
based topic. The publisher will then 'reply' by publishing out to this UUID based topic
which only the recovering client will be subscribed to, therefore you're not messing up
state for sibling subscribers. It's actually a trick which was lifted from the original
OpenMAMA avis bridge.

This approach was taken for a few reasons rather than ZeroMQ's Request / Reply pattern:
* I was keeping one eye on wanting to use a mechanism which could be adapted relatively
  unchanged to PGM, therefore I didn't want any publisher to have to store any context 
  bout its downstream consumers.
* I remember reading somewhere that ZeroMQ has weird race conditions with mixing pub /
  sub and request / reply patterns in the same context.
* The approach is already used in the Qpid Proton bridge where a broker is used.

### Summing up

It all revolves around striking a balance between what OpenMAMA expects and what ZeroMQ
provides. It's a series of mappings and tradeoffs that every OpenMAMA bridge developer
needs to evaluate. In this case, I wanted the design to focus on making the Bridge's choice of
ZeroMQ 'transport' pluggable so that the same code path is hit and well tested for each
ZeroMQ transport, whether it's pgm, tcp, inproc or ipc.