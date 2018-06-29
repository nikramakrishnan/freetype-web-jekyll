---
title: FreeType Overview
tags: [getting_started, overview]
sidebar: home_sidebar
permalink: overview.html
folder: mydoc
nav: overview.html
---

## What is FreeType? {#what}

FreeType is a software font engine that is designed to be small,
efficient, highly customizable, and portable while capable of producing
high-quality output (glyph images). It can be used in graphics
libraries, display servers, font conversion tools, text image generation
tools, and many other products as well.

Note that FreeType is a *font service* and doesn\'t provide APIs to
perform higher-level features like text layout or graphics processing
(e.g., colored text rendering, 'hollowing', etc.). However, it greatly
simplifies these tasks by providing a simple, easy to use, and uniform
interface to access the content of font files.

FreeType is released under two open-source licenses: our own BSD-like
[FreeType
License](http://git.savannah.gnu.org/cgit/freetype/freetype2.git/tree/docs/FTL.TXT)
and the [GNU Public License,
Version 2](http://git.savannah.gnu.org/cgit/freetype/freetype2.git/tree/docs/GPLv2.TXT).
It can thus be used by any kind of projects, be they proprietary or not.

Please note that 'FreeType' is also called 'FreeType 2', to distinguish
it from the old, deprecated 'FreeType 1' library, a predecessor no
longer maintained and supported.

## Features {#features}

The following is a non-exhaustive list of features provided by FreeType.

-   FreeType provides a simple and easy-to-use API to access font
    content in a uniform way, independently of the file format.
    Additionally, some format-specific APIs can be used to access
    special data in the font file.

-   The design of FreeType is based on modules that can be either linked
    statically to the library at compile time, or loaded on demand at
    runtime. Modules are used to support specific font formats, or even
    new glyph image formats!

-   FreeType was written with embedded systems in mind. This means that
    it doesn\'t use static writable data (i.e., it can be run from ROM
    directly), and that client applications can provide their own memory
    manager and I/O stream implementation. The latter allows you to
    easily read from ROM-based, compressed or remote font files with the
    same API. Several stream implementations can be used concurrently
    with a single FreeType instance.

    You can also reduce the size of the FreeType code by only compiling
    the modules you need for your embedded project or environment.

-   By default, FreeType supports the following font formats.

    -   TrueType fonts (TTF) and TrueType collections (TTC)
    -   CFF fonts
    -   WOFF fonts
    -   OpenType fonts (OTF, both TrueType and CFF variants) and
        OpenType collections (OTC)
    -   Type 1 fonts (PFA and PFB)
    -   CID-keyed Type 1 fonts
    -   SFNT-based bitmap fonts, including color Emoji
    -   X11 PCF fonts
    -   Windows FNT fonts
    -   BDF fonts (including anti-aliased ones)
    -   PFR fonts
    -   Type 42 fonts (limited support)

-   From a given glyph outline, FreeType is capable of producing a
    high-quality monochrome bitmap, or anti-aliased pixmap, using 256
    levels of 'gray'.

-   FreeType supports all the character mappings defined by the TrueType
    and OpenType specifications. It is also capable of automatically
    synthetizing a Unicode charmap from Type 1 fonts, avoiding painful
    'encoding translation' problems common with this format (of course,
    original encodings are also available if necessary).

-   The FreeType core API provides simple functions to access advanced
    information like glyph names or basic kerning data.

-   A full-featured and efficient TrueType bytecode interpreter, trying
    to match the results of the Windows bytecode engine. There is also
    the possibility to activate subpixel hinting (a.k.a. *ClearType*,
    still under development).

-   For those who don\'t need or want to use the bytecode interpreter
    for TrueType fonts, we developed our own *automatic hinter* module.
    It is also used by other scalable formats.

-   Due to its modular design, it is easy to enhance the library,
    providing additional format-specific information through optional
    APIs (as an example, an optional API is provided to retrieve SFNT
    tables from TrueType and OpenType fonts).

-   FreeType provides its own caching subsystem. It can be used to cache
    either face instances or glyph images efficiently.

-   A bundle of demo programs demonstrate the usage of FreeType; look
    for the 'ft2demos-*x.x.x*' archive (or 'ftdmo*xxx*' in case you are
    on a Windows platform) at the locations given
    [here](../../download.html). `*x.x.x*` (or `*xxx*`) gives the
    version number, for example `2.4.10` or `2410`.

## Requirements {#requirements}

FreeType is written in industry-standard ANSI C and should compile
easily with any compliant C or C++ compiler. We have even taken great
care to eliminate *all warnings* when compiling with popular compilers
like gcc, Visual C++, and Borland C++.

Apart from a standard ANSI C library, FreeType doesn\'t have any
external dependencies and can be compiled and installed on its own on
any kind of system. Some modules *need* external libraries (e.g., for
handling fonts compressed with gzip or bz2), however, they are optional
and can be disabled.


## Patent Issues {#patents}

All patents related to the TrueType bytecode interpreter have expired
since May 2010. Support for the still patented ClearType color filtering
model is disabled by default.

More information regarding this topic is available at our [patents
page](../../patents.html).
