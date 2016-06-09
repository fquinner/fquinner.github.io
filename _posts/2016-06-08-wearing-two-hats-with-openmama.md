---
layout:     post
title:      Wearing Two Hats with OpenMAMA
date:       2016-06-08 01:00:00
summary:    My Personal Take on OpenMAMA's relationship with SR Labs
thumbnail:  cogs
tags:
 - OpenMAMA
 - Personal
---

Note that none of the below is me speaking for SR Labs - this is purely my opinion
based on my own personal experiences and opinions.

As many of you may know, I am now the maintainer for the OpenMAMA project. As I
suspect many of you may also know, I am also an SR Labs employee.

When I was over at the STAC convention flying the flag for OpenMAMA, I found myself
asked more than once what exactly the relationship was between OpenMAMA and SR Labs.
The question always feels somewhat loaded because you feel like the question that they
really want to ask is "OpenMAMA sounds like a honey trap - what are SR Labs up to?"

The apparent paradox that I think outsiders of the project struggle to accept is just
how someone who is a full time employee of SR Labs can truly act in the best interests
of a vendor neutral project when they are on the payroll of a vendor. This extends not
just to me, but to all contributors who work for the same company.

The answer is in fact very simple.

I take both roles very seriously and I am fully committed to both of them, but they
are two separate and independent responsibilities for me.

### But SR Labs pay your salary right?

Full time employees are fairly common in Open Source projects. If you look at Clang/LLVM you'll find people
like Chris Lattner who work openly in the community but are also on the payroll of Apple, yet they work
shoulder to shoulder with people from Google and even Microsoft.

The reality of my OpenMAMA involvement is actually very straightforward. When SR Labs allocates me time to
work on OpenMAMA, I work on OpenMAMA tasks which SR Labs have agreed to sponsor efforts around including
day to day maintenance. When I am at home and off the SR Labs clock, you may find me working
on OpenMAMA, the ZeroMQ bridge, OMNM, or finally finishing my watch-through of Star Trek Deep
Space 9 (17 years later). Right now, I'm on leave from work and finishing off the final touches for a 1.0
release of the ZeroMQ OpenMAMA bridge and doing code reviews for recent OpenMAMA submissions.

I think it's important to keep that sort of division to make it clear to the community that when I work
on OpenMAMA, I do so as the OpenMAMA maintainer - not exclusively as an SR Labs employee.

### SR Labs make middlewares, what was the reaction like when the ZeroMQ bridge was released?

The OpenMAMA ZeroMQ bridge was developed by myself in my spare time. It's something which
various people had suggested would benefit the OpenMAMA project but no-one had stepped
forward to develop it so I decided it would be a fun project to take on myself.

When I completed the first OpenMAMA ZeroMQ bridge release, I was greeted in the SR
Labs office by widespread appreciation rather than suspicion, despite the fact that ZeroMQ could
be considered as a competitor to some flavours of SR Labs' Data Fabric product. I saw
this as being down to a combination of the fact that they want OpenMAMA to be a success
and the quiet confidence that ZeroMQ isn't a real threat to their commercial products in
terms of performance.

### OK so... what are those guys up to then?

I can't speak on behalf of SR Labs, but from what I have seen, the anticlimactic truth is that
there is no sinister underlying
agenda - _they just want to see OpenMAMA become a success along with each of the other equally
weighted voters and sponsors in the OpenMAMA Steering committee_ because they are
adopters of the technology and they want others to adopt it too.

To be a little more succinct, the roadmap is signed off on by the Steering Committee in periodic
sync ups - it is absolutely not dicated by SR Labs. OpenMAMA also has an active community of users and
contributors who would cry foul at the first sign of an anti-competitive project direction, so
in many ways, the biggest guardian of the project's integrity is its transparency which is greater
than ever since its move to Github.

### Summing up

OpenMAMA is a vendor neutral platform. The entire point of the API is to remove vendor-specific
dependencies by standardising the API layer. The suggestion that SR Labs who are the biggest sponsor
of the project in terms of manpower are somehow trying to introduce stickiness is pretty ridiculous.
If they were really doing that, the project would be a complete failure. As it stands, there
seems to be a ramp-up in activity, particularly in the last 12 months, and the project
is going from strength to strength. We also know of several companies who have popped up on the
mailing lists who are using OpenMAMA without any SR Labs products whatsoever which I think is a
testament to the vendor neutrality of OpenMAMA.

