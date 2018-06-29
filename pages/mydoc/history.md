---
title: FreeType History
sidebar: home_sidebar
permalink: history.html
folder: mydoc
nav: history.html
---

## FreeType 1 {#freetype1}

The main author of FreeType was David Turner, who created the library in
1996 to render TrueType fonts, including an interpreter for handling
TrueType bytecode. It was originally written in the
[Pascal](http://en.wikipedia.org/wiki/Pascal_%28programming_language%29)
programming language. In 1997, Robert Wilhelm ported it
to [C](http://en.wikipedia.org/wiki/C_%28programming_language%29), and
Werner Lemberg joined the team. Originally, the C and Pascal versions
were developed in parallel, however, development of the Pascal version
stopped in 2000.

Using FreeType 1, as this original version is called today, some
programs were written which then had some impact, for example
[FreeType/2](ft_os2/index.html), a better TrueType font driver for the
[OS/2](http://en.wikipedia.org/wiki/Os/2) operating system, or
[ttf2pk](http://www.tug.org/svn/texlive/trunk/Build/source/texk/ttf2pk/),
a converter from TrueType fonts to
[TeX](http://en.wikipedia.org/wiki/TeX)\'s PK bitmap format (this
program is still part of [TeXLive](http://www.tug.org/texlive/)).

Robert Wilhelm stopped working on FreeType in 2000.

For historical reasons you can find here links to the documentation of
FreeType 1. Note that those pages are no longer maintained.

- [Overview](freetype1/index.html)
- [Features](freetype1/index.html#features)
- [Requirements](freetype1/index.html#requirements)
- [Screenshots](freetype1/screenshots.html)
- [Documentation](freetype1/documentation.html)
- [Patent issues](patents.html)
- [OpenType support](opentype/index.html)
- [Related projects](freetype1/projects.html)


## FreeType 2 {#freetype2}

After several beta releases, version 2.0 of FreeType was released in
2000, providing a complete rewrite to make it modular. To distinguish it
from FreeType 1, it was then called FreeType 2 (which is no longer
necessary today). Most important new features were the support of
multiple font formats, in particular PostScript fonts (together with
hinting engine), a much improved rendering backend for gray-level
images, and a module for caching glyphs images and glyph metrics. Later
versions added support for even more font formats and LCD rendering
modes.

In the year 1999 we were informed by Apple Inc. that FreeType was
infringing patents related to TrueType hinting (see [here](patents.html)
for more). As a consequence, we disabled the bytecode interpreter
completely by default. Later on we were able to refine this radical cut
by disabling only the patented instructions, handling more than 95% of
the other instructions just fine and circumventing problems with the
remaining 5% to a certain extent. Another consequence was the invention
of the auto-hinter which needs no hints at all (you can find
[here](http://www.tug.org/TUGboat/tb24-3/lemberg.pdf) an early scholarly
paper). Today, the TrueType patents have expired.

Around 2004 it was decided to restrict FreeType to a single target:
Creating (rasterized) glyph images. Attempts to handle OpenType layout
features and text shaping were abandoned since this could be handled
much better with a separate library on top of FreeType (cf.
[ICU](http://www.icu-project.org/) or [Harfbuzz](http://harfbuzz.org)).

In 2009, the source code repository has been migrated from
[CVS](http://www.nongnu.org/cvs/) to [git](http://git-scm.com/), and
David Turner stopped active development on FreeType. Since then the only
maintainer is Werner Lemberg, with the help of others who contribute
fixes and improvements.