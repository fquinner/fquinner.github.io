---
layout:     post
title:      OpenMAMA ZeroMQ 0.3 Released - PGM and IPC support added
date:       2016-05-08 13:21:00
summary:    Multicast and inter-process transports added
thumbnail:  cogs
tags:
 - OpenMAMA
 - ZeroMQ
 - Scons
 - Architecture
---

One of the things that has always interested me about ZeroMQ is its
approach to different transports. The theory goes that the transports
should be fairly pluggable with a few small code changes and it turns
out that this is in fact true.

### So how hard was it?

The previous release of the ZeroMQ bridge already provided URI support and the
design of the bridge always had alternative transport bridges in mind so it
was actually fairly straightforward. The main change that you need to make to
support different transport types is to figure out whether or not you should
'bind' or 'connect' which varies according to transport types. Apart from that,
everything else seems to be pretty much the same.

### What does the configuration look like?

I have provided new `mama.properties` transport details for all the new transport
types in the configuration but here they are for reference:

    # Ensure loopback is running multicast for these to work
    mama.zmq.transport.pub_pgm.outgoing_url=pgm://127.0.0.1;239.192.1.1:5657
    mama.zmq.transport.pub_pgm.incoming_url=pgm://127.0.0.1;239.192.1.1:5656
    
    mama.zmq.transport.sub_pgm.outgoing_url=pgm://127.0.0.1;239.192.1.1:5656
    mama.zmq.transport.sub_pgm.incoming_url=pgm://127.0.0.1;239.192.1.1:5657
    
    # Ensure loopback is running multicast for these to work
    mama.zmq.transport.pub_epgm.outgoing_url=epgm://127.0.0.1;239.192.1.1:5657
    mama.zmq.transport.pub_epgm.incoming_url=epgm://127.0.0.1;239.192.1.1:5656
    
    mama.zmq.transport.sub_epgm.outgoing_url=epgm://127.0.0.1;239.192.1.1:5656
    mama.zmq.transport.sub_epgm.incoming_url=epgm://127.0.0.1;239.192.1.1:5657
    
    # Note destination here must be writable
    mama.zmq.transport.pub_ipc.outgoing_url=ipc:///tmp/feeds_0
    mama.zmq.transport.pub_ipc.incoming_url=ipc:///tmp/req_0
    
    mama.zmq.transport.sub_ipc.outgoing_url=ipc:///tmp/req_0
    mama.zmq.transport.sub_ipc.incoming_url=ipc:///tmp/feeds_0

One of the interesting things about pgm is it seems to lack the major pieces
of functionality that is useful for the market data world - namely topic and
multicast group resolution which would have vastly improved the usability of
pgm. Looks like that's something which continues to distinguish the commercial
messaging middlewares like SR Labs' Data Fabric or Informatica's UMS from PGM
(though my knowledge of PGM is weak so please correct me if I'm wrong).

### So what's the general state of the ZeroMQ bridge now?

The ZeroMQ bridge is effectively complete. It now supports all of the ZeroMQ
bridges that I had planned to support and it has a 100% pass rate with the OpenMAMA
unit tests.

### If it's so complete, why not provide a 1.0 release?

I might consider a 1.0 release in time, but there are a few things that I would like
to see completed before doing so:

* https://github.com/fquinner/OpenMAMA-zmq/issues/5: Make pool sizes configurable
* https://github.com/fquinner/OpenMAMA-zmq/issues/4: Make various zeromq socket options configurable
* https://github.com/fquinner/OpenMAMA-zmq/issues/2: Should see if the zeromq's proc is a worthy queue implementation
* https://github.com/fquinner/OpenMAMA-zmq/issues/1: Should move memory pools being used for message passing to queues

Once these are sorted, I think we'll be ready for a 1.0 because it would be the
finally settled-on architecture for internal queuing and it would be completely
tunable. As long as these things are up in the air, I'd prefer not to call it
a "1.x" release.

