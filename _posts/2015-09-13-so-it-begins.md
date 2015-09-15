---
layout:     post
title:      And so it begins...
date:       2015-09-13 20:11:50
summary:    Writing an OpenMAMA middleware bridge - a disappointingly straightforward affair.
thumbnail:  cogs
tags:
 - OpenMAMA
 - ZeroMQ
---

## Writing an OpenMAMA Middleware Bridge for ZeroMQ

After we completed the Qpid Proton bridge for OpenMAMA, I found myself 
constantly wondering how some of the problems we faced could be solved 
differently if we had a chance to write another bridge. It finally 
annoyed me enough that I decided to take on writing a ZeroMQ bridge 
myself with no motivation other than my own amusement.

The Qpid Proton bridge was intentionally written to try and be as modular as
possible. The queue is completely independent, as is the timer and io modules.
This is so much the case that when it came to writing the ZeroMQ bridge, I
could just lift these components out of qpid almost verbatim. Even the 
endpoint functionality could be reused which was extremely helpful and 
thankfully because it wasn’t part of the bridge interface itself (and 
therefore didn’t have ‘middleware name’ prefixed function names), you’ll 
notice that the ZeroMQ bridge just compiles this code straight from the 
Qpid bridge.

This all made writing the ZeroMQ bridge a disappointingly straightforward affair.
The vast majority of the code changes required were in the middleware msg.c
and transport.c files. Almost the rest of the code remained untouched
(except for renaming the functions for the bridge of course). I know there is
an effort at the moment around dynamic bridge loading, but after the dust
settles on that, I would like to look at we could make these individual
components pluggable, because practically the same code currently exists
across the qpid, avis and zmq bridges now. The ZeroMQ bridge also makes use of 
the same threading model as Qpid Proton (with a dispatcher thread handling all
of the main IO) and the same mechanisms for request / reply (pub / sub on uuid
based topic).

One of the problems you face when writing an OpenMAMA bridge is how to manage
memory. This is always a tricky subject when you want to minimize latency.
The Qpid bridge had its own bespoke memory pool to manage this which was
perfectly functional, but the implementation itself was embedded into the
bridge transport so it wasn't reusable for any other implementation. This is
why I decided to take the qpid bridge's memory pool functions and combine them
with an elastic buffer implementation which would be useful for a ZeroMQ
bridge, and bundle them up as memoryNode and memoryPool implementations and
submit them back into OpenMAMA (you may have seen these dropping into the
OpenMAMA `next` branch recently - that's why). I have also raised a pull 
request to update the Qpid bridge to use the same generic memoryPool.

Once basic pub / sub was up and running, I decided to run some mamaproducer / 
consumer tests to establish how close the results of the bridge were to that 
of a native zmq implementation. If you look in the src directory of the code 
you’ll find `hammer.c` and `nail.c` which really exist to give a feel for what 
the capacity of the middleware actually is on the machine being tested (it was
only a laptop communicating over loopback and power saving modes greatly 
impacted performance). Hammer and Nail will always be faster than producer / 
consumer anyway because the former are not multithreaded and they don’t really
do any payload decoding as such - just buffer type casts.

After a few attempts at gathering the results, I realized that the performance
was ghastly when compared with zeromq and a quick profiled run immediately 
pointed to why. The performance of the qpid payload was very poor. In my poorly 
tuned environment, zeromq hammer and nail were hitting around 1m msg/s and the 
qpid payload was hitting closer to 90,000. I think this was due to a combination 
of longer execution time and larger serialized message sizes (see 
http://zeromq.org/results:0mq-tests-v03#toc4 for the effect message size has on 
ZeroMQ performance).

At this point I went shopping for payload implementations to write an OpenMAMA 
bridge for as well - I had a look at bson, capnproto and google protocol buffers 
but they all seemed to be either *flexible and slow* or *inflexible and fast*. 
I really wanted self-describing messages which were still quite fast, so I 
decided I would just hack together my own straight into the bridge. That project 
(OpenMAMA Native Message - omnm) is going to be released in the coming weeks as
well, so details of that project is a job for another post.

After throwing together omnm, I ran the tests again to discover that the 
performance was vastly improved. I was suddenly sending / receiving around 300k
msg/s over producer / consumer. With some further optimizations this crept up a 
little more. Then I used the rdtsc option to perform time stamping in 
mamaproducerc / mamaconsumerc and I saw the numbers shoot up to ~800k (let’s not 
get too excited here - this was all over loopback so it never left the box) 
compared with about 1m for hammer / nail (though admittedly, hammer / nail were 
still using gettimeofday, but it wasn’t doing any payload decoding). The latency
was quite poor throughout the testing at these sorts of rates, but I pretty much 
ignored that at this point because the whole dynamic of the communications will
change once we start communicating over a network anyway - at this point, I was
more interested in throughput.

Please ignore all of the numbers in the last few paragraphs - these were all 
development-box stuff (on battery) and don’t even involve a network, so they 
mean nothing in isolation. I may test properly at some point in the future, but 
this was merely an exercise in bottleneck elimination.

So there you have it - a summary of some of the trials involved in writing a ZeroMQ
middleware bridge for OpenMAMA. It was fairly straightforward to get to the point 
where I could do a first release - in no small part due to the existing qpid bridge
and how clean the ZeroMQ API is.

You can find the ZeroMQ bridge here:

https://github.com/fquinner/OpenMAMA-zmq
