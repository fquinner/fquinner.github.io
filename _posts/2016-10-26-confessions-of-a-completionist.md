---
layout:     post
title:      Confessions of a Completionist
date:       2016-10-26 01:00:00
summary:    Confessions of a Completionist
thumbnail:  cogs
tags:
 - OpenMAMA
 - OMNM
---

A few years ago when I was completing the qpid payload bridge, one of my colleagues at the time referred to be as a "completionist" in a tone which was loaded with both amusement and disdain in equal doses. He said it after I insisted on wanting each and every interface we had a unit test for to pass for the bridge, and that the payload would support every possible data type, even when the underlying qpid proton body didn't naturally support it.

There was an element of justification to the mockery. Quite literally no-one used the mamaDateTime or mamaPrice data formats. However, because they were... incomplete, they were begging for someone to implement them. I found it difficult to turn my back on mapping out functionality for them even though I knew no-one was likely to use these methods.

Amusingly I assumed that the term "completionist" was a word. Turns out that it's not. Much like the word "performant", I hear it used regularly, yet it hasn't been promoted to a dictionary word yet. It's actually a term which is commonly used among the gaming community to refer to people who insist on completing every mission and every achievement / trophy so that the game is 100% completed. Ironically the term couldn't be any less accurate when it comes to describing the way I play games. I'm very much a hit and run guy who plays games on casual for the experience, then swiftly moves on to the next one (if you listen carefully you can hear hardcore gamers tut-tuting right now).

When I wrote the ZeroMQ bridge for OpenMAMA, it annoyed me that I had no decent way to test it because the qpid payload bridge was so cripplingly slow that it was impossible to properly push ZeroMQ anywhere near its limits. Out of that frustration, the OMNM payload bridge / implementation hybrid was born. The OMNM bridge was always intended to be incomplete. The idea was purely to create a minimal payload implementation which allowed me to find bottlenecks in the ZeroMQ bridge with standard MAMA performance tools (mamaproducerc / mamaconsumerc). All it really needed was support for some basic scalar fields, a means to handle marshalling and to be reasonably fast.

Then it began.

The daydreaming in the car trip home considering what a payload bridge might look like and arriving home without realizing I didn't even turn on the radio.

The thoughts about how it would be nice if I didn't have to say to people looking at the ZeroMQ bridge that they still have to use the qpid payload.

Consideration of the interesting challenges associated with nested vector messages and whether or not the qpid payload bridge implementation had adopted the right approach or if it could be improved.

Eventually enough was enough and I decided to pull down a copy of OMNM and have another crack at it. The goal is now to take OMNM to that illusive 1.0 release with full functionality, no memory leaks and all unit tests passing. It's something i started poking around with a few weeks ago and I now have an implementation which passes all unit tests but still needs a lot more testing and tracking down of memory leaks (which I intentionally didn't clean up as I went along because I was always ready to pivot and start again if I decided I didn't like the design).

After this has been completed, the ZeroMQ bridge will have the OMNM bridge included in future releases and will default to the omnm payload bridge. This should provide significantly improved performance for all future users of the bridge, and make it much easier to get up and running with ZeroMQ.

You can have a look at the recent changes over at https://github.com/fquinner/OpenMAMA-omnm.