---
title: FreeType Downloads
sidebar: home_sidebar
permalink: download.html
folder: mydoc
last_updated: June 29, 2018
nav: download.html
---

## Stable Releases

Stable releases of the FreeType packages, including source code,
documentation, demo programs, and support tools can be downloaded
directly from

[https://savannah.nongnu.org/download/freetype/](https://download.savannah.gnu.org/releases/freetype/)

or from

<https://sourceforge.net/projects/freetype/files/>

Both sites have mirrors worldwide.

Windows DLLs of FreeType can also be downloaded directly from [a github
repository](https://github.com/ubawurinna/freetype-windows-binaries)
(version 2.7.1, built with VS Express 2012). Older DLLs compiled with
MinGW are available from [download
page](http://gnuwin32.sourceforge.net/packages/freetype.htm) (version
2.3.5) of the [GnuWin32](http://gnuwin32.sourceforge.net/) project, or
from the [GTK+ for Windows download
page](https://sourceforge.net/projects/gtk-mingw/files/freetype/)
(version 2.4.10).

The latest Freetype 1 release, today mainly of historical interest only,
can be downloaded
[here](https://sourceforge.net/projects/gnuwin32/files/freetype/1.4/).
Other projects related to FreeType are only available as CVS source code
modules; see the [developer page](developer.html#source-code) for more
details.


## Development Versions

Using Savannah\'s [cgit](https://git.savannah.gnu.org/cgit/freetype/)
interface you can download snapshots created on the fly for any commit
entry.

Downloading and compiling a development snapshot is one of the first
things to do when you encounter something that looks like a bug in
FreeType. Note that we do not guarantee that all development snapshots
work on a given day, though this should be the normal case.

For more details on compilation, please read the file
[`README.git`](https://git.savannah.gnu.org/cgit/freetype/freetype2.git/tree/README.git),
located in FreeType\'s top directory.


## Support Tools and Binaries {#tools-and-binaries}

### GNU Make for Win32

[GNU Make](https://sourceforge.net/projects/mingw/files/MinGW/Extension/make/)
is required to use the FreeType build system from a command line prompt.

Note that the binary comes from the excellent [MinGW
distribution](http://www.mingw.org/), which we highly recommend over
Cygwin, if you don\'t need POSIX-compliance, in terms of ease of use and
performance.

After installation you should check that it is correctly invoked by
typing:

    make -v

The first lines should read:

    GNU Make 3.81
    Copyright (C) 2006  Free Software Foundation, Inc.
    This is free software; see the source for copying conditions.
    There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A
    PARTICULAR PURPOSE.

Newer version work also, of course.

### Project Files

The FreeType source code bundle contains project files for various
versions of Microsoft Visual C++ and Visual Studio for x64, Windows 32,
and Windows CE. However, these files have been contributed and might be
out of date, thus use them with care.

For classic MacOS, support files for MPW and CodeWarrior are provided.


### FreeType Jam for Win32 and GNU/Linux {#ftjam}

Since release 2.0.2, FreeType and its demo programs can be compiled with
an alternative build tool called 'jam'. Jam is a small, portable, and
open-source replacement for 'make' that is a lot easier to use, and
which transparently supports multiple platforms *and* compilers.

Binaries are available from the [download page on
SourceForge](https://sourceforge.net/projects/freetype/files).

[This page](jam/index.html) gives more information regarding Jam and
FreeType.

{% include callout.html content="Due to lack of resources, there is no Jam
support currently for FreeType. Volunteers are highly welcomed."
type="danger" %}