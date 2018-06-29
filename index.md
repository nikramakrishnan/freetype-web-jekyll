---
title: "The FreeType Project"
keywords: FreeType fonts renderer TrueType OpenType Type1 CFF TTF OTF TTC OTC library free fast quality
sidebar: home_sidebar
permalink: index.html
#summary: The FreeType Project is a free, high-quality and portable Font engine
toc: false
nav: index.html
---

FreeType is a freely available software library to render fonts.

It is written in C, designed to be small, efficient, highly
customizable, and portable while capable of producing high-quality
output (glyph images) of most vector and bitmap font formats.

Some products that use FreeType for rendering fonts on screen or on
paper, either exclusively or partially:

-   [GNU/Linux](http://www.gnu.org/gnu/why-gnu-linux.html) and other
    free Unix operating system derivates like
    [FreeBSD](http://www.freebsd.org/) or
    [NetBSD](http://www.netbsd.org/);
-   [iOS](http://www.apple.com/ios/), Apple\'s mobile operating system
    for iPhones and iPads;
-   [Android](http://www.android.com/), Google\'s operating system for
    smartphones and tablet computers;
-   [ChromeOS](http://www.chromium.org/chromium-os), Google\'s operating
    system for laptop computers;
-   [ReactOS](http://reactos.org), a free open source operating system
    based on the best design principles found in the Windows NT
    architecture;
-   [Ghostscript](http://www.ghostscript.com/), a PostScript interpreter
    used in many printers.

Counting the above products only, you get more than a *billion* devices
that contain FreeType.

## News & Updates {#news}

### FreeType 2.9.1

2018-05-02

This is a maintenance release; most importantly fixing correct handling
of Type 1 fonts with flex features, which was broken in version 2.9. An
overview of the remaining changes is given
[here](https://sourceforge.net/projects/freetype/files/freetype2/2.9.1/).
All users should upgrade.

### FreeType 2.9

2018-01-08

FreeType version 2.9, the first release of a new 'minor' series, is now
available for download. The main reason for starting a new series is
Ewald Hew\'s GSoC contribution of making Adobe\'s CFF engine handle
Type 1 fonts also, greatly improving the rendering quality of this
ancient but still important font format.

If you are going to use variation fonts, please update to this version
since it comes with some important fixes. More information on this and
other changes can be found
[here](https://sourceforge.net/projects/freetype/files/freetype2/2.9/).

### FreeType 2.8.1

2017-09-16

FreeType 2.8.1 has been released. This is mainly a maintenance release
with one important change: By default, FreeType now offers high quality
LCD-optimized output without resorting to ClearType techniques of
resolution tripling and filtering. In this method, called *Harmony*,
each color channel is generated separately after shifting the glyph
outline, capitalizing on the fact that the color grids on LCD panels are
shifted by a third of a pixel. This output is indistinguishable from
ClearType with a light 3-tap filter.

See
[here](https://sourceforge.net/projects/freetype/files/freetype2/2.8.1/)
for a extensive list of changes; noteworthy bug fixes are correct
handling of B/W TrueType hinting and some OpenType variation font
handling issues.

### FreeType 2.8

2017-05-13

FreeType 2.8 has been released. CFF2 support and OpenType variation font
handling is now complete; the auto-hinter now understands 25 more
scripts, for example N\'Ko and Tifinagh.

See
[here](https://sourceforge.net/projects/freetype/files/freetype2/2.8/)
for a list of changes; noteworthy bug fixes are the handling of TrueType
fonts: unhinted loading didn\'t work as expected, and the light
auto-hinter used incorrect metrics.

### GSoC

2017-02-28

The FreeType project was accepted to be part of [Google Summer of
Code](https://developers.google.com/open-source/gsoc/) 2017! Here\'s [a
link](gsoc.html) to our ideas list -- if you have another one, please
write to our [mailing list](mailto:freetype-devel@nongnu.org) so that we
can discuss your suggestions, eventually adding them to the list. And if
you want to participate as a student, now is the time to discuss
everything, again using the mailing list.

### FreeType 2.7.1

2016-12-30

FreeType 2.7.1 has been released. The most important news is preliminary
support of Adobe\'s new CFF2 font format and variation fonts as
specified in the new OpenType specification version 1.8. It also fixes
the handling of raw CID fonts (which might be found in PDF files)

See
[here](https://sourceforge.net/projects/freetype/files/freetype2/2.7.1/)
for a list of changes.

### FreeType 2.7

2016-09-08

We start a new 'minor' series, which finally allows us to activate a new
default mode for bytecode hinting (see also the announcements below for
version 2.6.4 and 2.6.5): Subpixel hinting, also known as ClearType
hinting.

In case you are already using subpixel hinting (for example, by using
the 'Infinality patches' as provided by some GNU/Linux or BSD
distributions, or directly from [bohoomil.com](https://bohoomil.com/)),
be noted that the new mode might provide subtle differences; the code
was simplified to make it *much* faster. If you are used to the old
full-pixel hinting, you will see many rendering changes. If you really
dislike them, you can disable them at compile time or using the new
`FREETYPE_PROPERTIES` environment variable.

A description of the remaining changes can be found
[here](https://sourceforge.net/projects/freetype/files/freetype2/2.7/),
as usual.

### FreeType 2.6.5

2016-07-12

This release is almost identical to the previous version, with two
differences.

-   It compiles again on Mac OS X, and
-   it reverts the activation of subpixel hinting by default; it will be
    enabled by default in the forthcoming 2.7.x series. Main reason for
    reverting this feature is the principle of least surprise: a sudden
    change in appearance of all fonts (even if the rendering improves
    for almost all recent fonts) should not be expected in a new micro
    version of a series.

### FreeType 2.6.4

2016-07-05

FreeType 2.6.4 has been released. The most important change is a new
bytecode hinting mode for TrueType fonts that finally activates subpixel
hinting (a.k.a. ClearType hinting) by default.

The new release also brings support for the following new scripts in the
auto-hinter: Armenian, Cherokee, Ethiopic, Georgian, Gujarati, Gurmukhi,
Malayalam, Sinhala, and Tamil.

See
[here](https://sourceforge.net/projects/freetype/files/freetype2/2.6.4/)
for a detailed list of changes.

### FreeType 2.6.3

2016-02-09

FreeType 2.6.3 has been released. It brings support for four new Asian
scripts in the auto-hinter (Khmer, Myanmar, Kannada, and Bengali),
together with other, minor improvements and bug fixes.

See
[here](https://sourceforge.net/projects/freetype/files/freetype2/2.6.3/)
for a detailed list of changes.

### More on the 2.6.2 release for users and developers

2015-11-30

FreeType 2.6.2 ships with three interesting details for users and
developers of rendering libraries that deal with text. [Read
more](freetype2/docs/text-rendering-general.html).

### FreeType 2.6.2

2015-11-28

FreeType 2.6.2 has been released. This is a minor release that mainly
provides better handling of malformed fonts. All users should upgrade.

A new feature is stem darkening support for the auto-hinter. Note,
however, that it is off by default, since most graphic systems don\'t
provide correct linear alpha blending with gamma correction, which is
crucial for a good appearance. For the same reason, stem darkening for
the CFF engine is now off by default, too.

See
[here](https://sourceforge.net/projects/freetype/files/freetype2/2.6.2/)
for a more detailed list of changes.

### FreeType 2.6.1

2015-10-04

FreeType 2.6.1 has been released. This is a minor release that corrects
problems with CFF metrics, and that provides better handling of
malformed fonts. Two notably new features are auto-hinting support for
the Lao script and a simple interface for accessing named instances in
GX TrueType variation fonts.

See
[here](https://sourceforge.net/projects/freetype/files/freetype2/2.6.1/)
for a list of changes.

### FreeType 2.6

2015-06-08

FreeType 2.6 has been released. This is a new major release that
provides a better (and simpler) thread-safety model. Among other new
features we now have auto-hinting support for Arabic and Thai, together
with much improved handling of Apple\'s GX TrueType variation font
format.

See
[here](https://sourceforge.net/projects/freetype/files/freetype2/2.6/)
for a list of changes.

### FreeType 2.5.5

2014-12-30

FreeType 2.5.5 has been released. This is a minor bug fix release: All
users of PCF fonts should update, since version 2.5.4 introduced a bug
that prevented reading of such font files if not compressed.

### FreeType 2.5.4

2014-12-06

FreeType 2.5.4 has been released. All users should upgrade due to
another fix for vulnerability CVE-2014-2240 in the CFF driver. The
library also contains a new round of patches for better protection
against malformed fonts.

The main new feature, which is also one of the targets mentioned in the
pledgie roadmap below, is auto-hinting support for Devanagari and
Telugu, two widely used Indic scripts. A more detailed description of
the remaining changes and fixes can be found
[here](https://sourceforge.net/projects/freetype/files/freetype2/2.5.4/).

### New Pledgie Campaign

2014-03-11

This is a call for a [new Pledgie
campaign](https://web.archive.org/web/20160324122145/https://pledgie.com/campaigns/24434)
to support my (Werner Lemberg) expenses in 2014. Thanks to all donors,
the [last
campaign](https://web.archive.org/web/20180102004811/https://pledgie.com/campaigns/18808)
was successful, and all goals have been reached!

If your company is using FreeType in your product, and you care about
continuing support and further development, please contribute to my
funding effort so I can continue to bring the best text rendering to
your devices!

Alternatively, direct donations to my PayPal account are also highly
welcome `:-)`

#### Roadmap

Besides user support and fixing bugs, your money will help me implement
the following issues.

-   Setting up a test framework for FreeType. This is a huge, long-term
    undertaking that will ensure both stability and reliability of the
    library. The idea is to collect test cases (mainly broken fonts)
    that cover as much source code as possible. Another idea to
    investigate is the development of scripts that can generate both
    valid and invalid input data to systematically increase the coverage
    of executed library code, including the unlikely cases. Finally,
    images of valid, well-rendered input fonts could be collected: As
    soon as a change to the rendering image gets applied, a comparison
    run with those images should detect rendering regressions.
-   Further improvements to the auto-hinter. Right now, the module for
    Indic support is a dummy, and support for the family of Arabic
    scripts is completely missing. [\[FreeType 2.9 comes with
    auto-hinting support for almost all scripts where hinting makes
    sense.\]] Both investigation and research is necessary to
    find out how much auto-hinting is possible and useful, and whether
    other, completely different scripts can be supported at all.
-   Right now, rendering Type 1 and CID-keyed fonts is the weakest part
    of FreeType. However, we now have a brand-new module for handling
    CFF. Given that CFF is very similar to Type 1, it should be not too
    difficult to use and/or extend the CFF code so that Type 1 fonts can
    be handled, too. [\[This was a GSoC project in 2017, and the
    resulting code has been merged into FreeType 2.9.\]]
-   Explore whether it makes sense to merge FreeType with (parts of) the
    [HarfBuzz](http://harfbuzz.org) library. Since version 2.5.3,
    FreeType already links to HarfBuzz to use its abilities for scanning
    OpenType layout features, and more integration might be sensible for
    both libraries.
-   More improvements to this website. Last year I've redesigned the
    FreeType website. However, a large bunch of documents are still
    using the old design, and some of them are also no longer up to
    date. [\[As with version 2.6.0, the FreeType Tutorial has been
    updated.\]]

### FreeType 2.5.3

2014-03-08

FreeType 2.5.3 has been released. All users should upgrade due to fixed
vulnerability in the CFF driver (CVE-2014-2240).

Its main new feature is much enhanced support of auto-hinting SFNT fonts
(i.e., TrueType and CFF fonts) due to the use of the
[HarfBuzz](http://harfbuzz.org) library. A more detailed description of
this and other changes can be found
[here](https://sourceforge.net/projects/freetype/files/freetype2/2.5.3/).

### FreeType 2.5.2

2013-12-08

FreeType 2.5.2 has been released. It fixes a serious bug introduced in
version 2.5.1; all users should upgrade.

A listing of the changes can be found
[here](https://sourceforge.net/projects/freetype/files/freetype2/2.5.2/).

### FreeType 2.5.1

2013-11-25

FreeType 2.5.1 has been released, providing three major new features.

-   Support for the WOFF font format, contributed by Behdad Esfahbod.
-   The auto-hinter now supports Hebrew, together with improved support
    for Cyrillic and Greek.
-   The directory layout of the (installed) FreeType header files has
    been simplified.

Among other changes I want to mention that FreeType\'s TrueType debugger
(`ttdebug`) has been made more versatile. An exhaustive list of changes
can be found
[here](https://sourceforge.net/projects/freetype/files/freetype2/2.5.1/).

### Pledgie Campaign Was Successful!

2013-06-25

Thanks to a very generous donation by Pierre Arnaud from
[Epsitec](http://www.epsitec.ch), the pledgie campaign for FreeType has
reached its goal. I want to say thank you again to all donors! Of
course, noone stops you from further donating to the campaign :-)

After integration of Adobe\'s CFF module and Google\'s color emoji
support, I will use the next months to work on the remaining issues that
I\'ve promised to implement. Stay tuned!

### FreeType 2.5

2013-06-19

FreeType 2.5 has been released. A major new feature is support for color
embedded bitmaps (eg. color emoji), contributed by Behdad Esfahbod on
behalf of Google. Additionally, Adobe\'s CFF engine is now the default,
which makes a good reason to change from the 2.4.x to the 2.5.x series.

On the technical side, the property API to access FreeType module
parameters (`FT_Property_Set` and `FT_Property_Get`) is now declared as
stable.

As usual, see [this
file](https://sourceforge.net/projects/freetype/files/freetype2/2.5.0/)
for the complete release notes, which give more details. And we have
again blog entries from
[Adobe](http://blogs.adobe.com/typblography/2013/06/adobe-cff-font-rasterizer-accepted-by-freetype.html)
and
[Google](http://google-opensource.blogspot.com/2013/06/youve-got-cff.html).

\[Please download the 2.5.0.1 bundle of the FreeType library, which
fixes a packaging error.\]

## Links {#links}

The links collected in this section are useful if you want to put
FreeType into a larger frame of understanding.

### Reference Sites

[Microsoft Typography](http://www.microsoft.com/typography/) --
Microsoft\'s OpenType specification and developing tools\
[Apple Fonts](http://developer.apple.com/fonts/) -- Apple\'s TrueType
specification and other things\
[Adobe Typography](http://www.adobe.com/products/type.html) --
PostScript fonts specifications and developing tools

Detailed information on the font formats supported by FreeType can be
found in the file
[`formats.txt`](http://git.savannah.gnu.org/cgit/freetype/freetype2.git/tree/docs/formats.txt),
which is part of the FreeType source code bundle.

### Font Tools

[TTX](https://github.com/fonttools/fonttools) -- an OpenType assembler
and disassembler\
[FontForge](http://fontforge.org) -- a free, powerful graphical font
editor, including a TrueType instructions debugger (using FreeType)\
[TrueTypeViewer](http://home.kabelfoon.nl/~slam/fonts/truetypeviewer.html)
-- a free, powerful OpenType viewing tool with a TrueType instructions
debugger (*not* using FreeType)\
[ttfautohint](ttfautohint/index.html) -- a tool to auto-hint TrueType
fonts, based on FreeType\'s auto-hinting engine

### Font Shaping and Layout Engines

These libraries work on top of font rendering libraries like FreeType to
provide sophisticated text (string) layout, being able to handle
OpenType features in particular. All of them use
[Unicode](http://www.unicode.org) for font and text encoding.

[Pango](http://www.pango.org) -- the layout library used by
[Gnome](http://www.gnome.org)\'s [GTK+](http://www.gtk.org) framework\
[ICU](http://www.icu-project.org) -- a layout library originally
developed by IBM, used for example in [XeTeX](http://tug.org/xetex/), an
internationalized successor of [TeX](http://en.wikipedia.org/wiki/TeX)\
[HarfBuzz](http://harfbuzz.org) -- a text shaping library, originally
based on FreeType 1\'s OpenType layout support

### Other Font-related Libraries

[T1Lib](http://www.t1lib.org/) -- a Type 1 fonts library (no longer
under development)\
[VFLib](http://www-masu.ist.osaka-u.ac.jp/~kakugawa/VFlib/) -- a library
especially for accessing TeX fonts (no longer under development)

<small>This page is maintained by [Werner Lemberg](mailto:wl@gnu.org). The
FreeType logo has been designed by [Manuel Colom](http://www.colom.org).</small>
