---
layout:     post
title:      Porting OpenMAMA ZeroMQ to Windows
date:       2015-09-20 12:19:50
summary:    When you're fairly sure your code is portable, how hard can it be?
thumbnail:  cogs
tags:
 - OpenMAMA
 - ZeroMQ
 - Scons
 - Windows
---

When I set out to write the OpenMAMA ZeroMQ bridge, I was pretty confident that
my code was going to compile OK for both Windows and Linux. This was for two
primary reasons:

* OpenMAMA's port.h already exists for most of the shortcomings in various MSVC
  compilers to actually conform with ISO C standards. For example, I believe it
  wasn't until Visual Studio 2015, that MSVC devs were spoiled with
  a non-funnily-named snprintf function.
* The vast majority of stuff that is likely not to be portable usually lies in
  the layer of code which deals with system calls. Luckily for me, Libevent and
  ZeroMQ abstract me nicely from all that #ifdef hell.

For these reasons, when asked if I could make the bridge compatible with Windows,
I was very confident that I could. I was also using scons which is available for
both Windows and Linux so surely there wouldn't be much drama there?

It would be an overstatement to say that this was a large effort to address,
but I can certainly say that it wasn't quite as straightforward as I had
imagined.

### Dependencies

Let's start at the start. The dependencies. The OpenMAMA ZeroMQ bridge has 4
known dependencies:

* Uuid
* OpenMAMA
* ZeroMQ
* Libevent

We don't need to worry about uuid on windows because the windows implementation
of the uuid functions are implemented in wombatcommon with no external
dependencies, so let's look at the rest.

#### OpenMAMA

The relevant installation tree for scons when building on windows is:

    include/
    bin/dynamic/lib<libname>md.dll
    bin/dynamic-debug/lib<libname>mdd.dll
    lib/dynamic/lib<libname>md.lib
    lib/dynamic-debug/lib<libname>mdd.lib

Like many projects on windows, you can choose to build using Visual Studio or
via the command line and each option produces a different directory structure,
but we'll stick with the above scons-generated directory structure as it's the
cleanest. Note both the library names and paths will need changing depending
on whether you want to link against the dynamic or dynamic debug versions.

#### ZeroMQ

Yeah I just cheated with ZeroMQ and grabbed the pre-compiled binaries via the
setup installer which placed the install in `C:\Program Files\ZeroMQ <version>`
and followed this installation tree:

    bin/libzmq-v<msvcver>-<flags>-<version>.dll
    lib/libzmq-v<msvcver>-<flags>-<version>.lib

The libraries here follow the 
[boost library naming convention](http://www.boost.org/doc/libs/1_42_0/more/getting_started/windows.html#library-naming)
and includes the MSVC version in the library name.

#### Libevent

This one needed to be compiled from source using nmake. It has known pains with
installing on windows, see
[the OpenMAMA wiki page](https://github.com/OpenMAMA/OpenMAMA/wiki/Building-on-Windows#libevent)
It just dumps the library in the directory where the compilation occurred.

    libevent_core.lib

### Building with Scons

I decided to use Scons for this project for a few reasons:

* OpenMAMA already uses it
* It works on Windows, Linux and OSX
* It's written on python rather than a build-tool-specific language

Part of why I thought this would be an easy task was because I chose scons as
a build tool. What could be different right? Just a few different build flags?

_That would be too easy._

Check out that dependency list above. None of the dependencies have anything in
common! They all follow either different directory structures, different
library names or both... and depending on how the dependency was prepared, even
the directory structure could differ. This is part of why I hate building stuff
on windows so much. Linux has a concept of a prefix and once that is
established, library names and directory paths are all standard. There are no
exceptions. Windows is a free for all of naming conventions which requires no
end of build tool hackery to get it working. Libraries are never installed to
a single standard path and even if they were, the library names would be *just*
different enough not to work out of the box without requiring build script
hackery to consider every distinct library name.

### Porting the Code

As expected, the code was mostly portable. The only issues were
header-precedence issues which were specific to windows so no major changes
were required.


You can find the latest version of the ZeroMQ bridge which now supports windows
here:

https://github.com/fquinner/OpenMAMA-zmq
