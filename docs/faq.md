---
layout: page
title: FreeType 2 FAQ
theme: dark-green
sidebar: docs_sidebar
permalink: docs/faq.html
toc: false
nav: faq
---

* TOC
{:toc}

## General questions

### What is FreeType 2?

It is a software library that can be used by all kinds of applications to
access the contents of font files.  Most notably, it supports the following
‘features’:

* It provides a uniform interface to access font files.  It supports both
  bitmap and scalable formats, including TrueType, OpenType, Type1, CID, CFF,
  Windows FON/FNT, X11 PCF, and others.

* It supports high-speed anti-aliased glyph bitmap generation with 256 gray
  levels.

* It is extremely modular, each font format being supported by a specific
  module.  A build of the library can be tailored to support only the formats
  you need, thus reducing code size (a minimal anti-aliasing build of FreeType
  can be less than 30KB)

### What can I do with FreeType 2?

FreeType 2 is already used in many products.  For example, it serves as a font
rendering engine

* in graphics subsystem and libraries to display text

* in text layout and pagination services to measure and eventually render text

* in font inspection and conversion tools

Generally speaking, the library allows you to access and manage the contents of
font files in a very easy way.

### What can I not do with FreeType 2?

FreeType 2 doesn't try to perform a number of sophisticated things, because it
focuses on being an excellent _font service_.

This means that the following features are not supported directly by the
library, even though they can be more or less implemented on top of it, or by
using it:

* **rendering glyphs to arbitrary surfaces**:   FreeType 2 doesn't try to be a
  graphics library and thus only supports two pixel formats when rendering
  glyphs: monochrome 1-bit bitmaps, or 8-bit gray-level pixmaps.

  If you need to draw glyphs to other kinds of surfaces (for example, a 24-bit
  RGB pixmap), you need to use your favorite graphics library to do just that.

  _Note however that in the case of rendering scalable glyph outlines to
  anti-aliased pixmaps, an application can also provide its own rendering
  callback in order to draw or compose directly the anti-aliased glyph on any
  target surface._

* **glyph caching**:   Each time you request a glyph image from a font,
  FreeType 2 does it by parsing the relevant portion of the font file or font
  stream and interpreting it according to its font format.  This can be very
  slow for certain formats, including scalable ones like TrueType or Type 1.

  Any decent text-rendering sub-system must thus be capable of caching glyph
  data in order to reach appropriate rendering speed.

  Note that we provide a caching sub-system with FreeType 2 since version 2.0.1
  which has become quite stable at the time of this writing (version 2.2.1).
  However, it might not suit your needs

* **text layout**:   The library doesn't support text layout operations.
  Sophisticated features like glyph substitution, positioning (kerning),
  justification, bi-directional ordering, etc.m are not part of a _font
  service_ in itself.  They must be handled one level higher.

### How portable is FreeType 2?

The FreeType 2 source code is _extremely_ portable for the following reasons:

* Everything is written in standard ANSI C.

* We are very pedantic to avoid any kinds of compiler warnings.  The current
  source code has been compiled with many compilers without producing a single
  warning.

* The library doesn't use any static writable data at all, making it an ideal
  choice on various embedded systems (e.g., it can be run from ROM directly).
  It is completely thread-safe too.

We have made great efforts to ensure that the library is efficient, compact,
and customizable.

### What are the differences between FreeType 1.x and FreeType 2?

The biggest differences are as follows.

* FreeType 1 only supports the TrueType format, while FreeType 2 supports a lot
  more.

* The FreeType 2 API is simpler as well as more powerful than the FreeType 1
  API.

* FreeType 1 includes an extension to support OpenType text layout processing.
  This support hasn't become part of FreeType 2; a much improved version is now
  part of the [Pango](http://www.pango.org) library.
              
### Is FreeType 2 backwards compatible with FreeType 1.x?

Short answer: No.  However, transition from 1.x to 2 should be rather
straightforward.

The FreeType 2 API is a lot simpler than the one in 1.x while being much more
powerful.  We thus encourage you to adapt your source code to it as this should
not involve much work.

### Can I use FreeType 2 to edit fonts or create new ones?

No.  The library has been specifically designed to _read_ font files with small
code size and very low memory usage.

A good, freely available font editor is [FontForge](http://fontforge.org/).

## Compilation & Configuration

### How do I compile the FreeType 2 library?

The library can be compiled in various ways, and detailed documentation is
available in documentation directory of the FreeType 2 source tree.

For compilation on the command line, GNU make is necessary; other build tools
won't work.  The source bundle also comes with project files for some graphical
IDEs like Visual C; note, however, that those files are sometimes not up to
date since it is contributed code not used by the core developers.

### How do I configure my build of the library?

This is fully described in the file `CUSTOMIZATION` in FreeType's documentation
directory.  Basically, you have to edit the header file `ftoption.h` for
compile-time options and to select the modules with the file `modules.cfg`.
Finally, it is possible to replace the standard system interface (dealing with
memory allocation and stream I/O) with a custom one.

### Why does FreeType render differently on different platforms?

Different distributions compile FreeType with different options.  The developer
version of a distribution's FreeType package, which is needed to compile your
program against FreeType, includes the file `ftoption.h`.  Compare each
platform's copy of `ftoption.h` to find the differences.

## The FreeType 2 auto-hinter

### How does the auto-hinter work?

_Please note that the name of auto-hinter module is **autofit**, which is a
reimplementation of the old autohint module._

A rather complete description of the hinting algorithm (which is slightly out
of date regarding the internal structures) can be found in the TUG-boat article
[Real-Time Grid Fitting of Typographic
Outlines](http://www.tug.org/TUGboat/Articles/tb24-3/lemberg.pdf).

The auto-hinter performs grid-fitting on scalable font formats that use Bézier
outlines as their primary glyph image format (this means nearly all scalable
font formats today).  If a given font driver doesn't provide its own hinter,
the auto-hinter is used by default.  If a format-specific hinter is provided,
it is still possible to use the auto-hinter using the `FT_LOAD_FORCE_AUTOHINT`
bit flag when calling `FT_Load_Glyph()`.

Currently, the auto-hinter doesn't use external hints to do its job, as it
automatically computes global metrics (when it ‘opens’ a font for the first
time) and glyph ‘hints’ from their outline.

### Why doesn't the auto-hinter work well with my script?

The auto-hinter was first designed to manage and hint Latin-based fonts, as
they consist of most of the fonts available today.  It now supports Asian
fonts, but not other complex scripts like Arabic.

Hinting various scripts isn't really more difficult than Latin, just different,
with a set of different constraints, which must be hard-coded into the autofit
module.  Volunteers welcome!

## Other questions

### Can I use FreeType to draw text on a pixmap with arbitrary depth?

Not directly, as FreeType is a font library, not a general-purpose graphics
library or text rendering service.  However, note that the anti-aliased
renderer allows you to convert a vectorial glyph outline into a list of ‘spans’
(i.e., horizontal pixel segments with the same coverage) that can be rendered
through user-provided callbacks.

By providing the appropriate span callback, you can render anti-aliased text to
any kind of surface.  You can also use any colour, fill pattern or fill image
if you want to.  This process is called _direct rendering_.

A complete example is given at the end of the [FreeType 2
tutorial](tutorial/step3.html).

Note that direct rendering is _not_ available with monochrome output, as the
current renderer uses a two-pass algorithm to generate glyphs with correct
drop-out control.

### How can I set the colour of text rendered by FreeType?

Basically, you can't do that, because FreeType is simply a font library.  In
general, you need to use your favorite graphics library to draw the FreeType
glyphs with the appropriate colour.

Note that for anti-aliased glyphs, you can ‘set the colour’ by using _direct
rendering_ as described in [this answer](#other-depth).

### I set the pixel size to 8×8, but the resulting glyphs are larger (or smaller) than that.  Why?

A lot of people have difficulties to understand this topic, because they think
of glyphs as fixed-width or fixed-height ‘cells’, like those of fonts used in
terminals/consoles.  This assumption is not valid with most ‘modern’ font
formats, even for bitmapped-based ones like **PCF** or **BDF**.

Be aware that the _character size_ that is set either through
`FT_Set_Char_Size()` or `FT_Set_Pixel_Sizes()` isn't directly related to the
dimension of the generated glyph bitmaps!

Rather, the character size is indeed the size of _an abstract square_, called
the _EM_, used by typographers to design fonts.  Scaling two distinct fonts to
the same character size, be it expressed in points or pixels, generally results
in bitmaps with _distinct dimensions_!

Note that historically, the EM corresponded to the width of a capital ‘M’ in
Latin typefaces.  However, later improvements in typography led to designs that
greatly deviate from this rule.  Today, it is not possible to connect the EM
size to a specific font ‘feature’ in a reliable way.

### How can I compute the bounding box of a text string without loading its glyphs before?

This is not possible in general.  Reason is that hinting distorts the glyph
shape for optimal rasterization, and this process sometimes creates outlines
which have considerably different metrics.  The TrueType format provides the
(optional) ‘hdmx’ table which contains device horizontal metrics for selected
pixel sizes, but even here the vertical metrics are missing.

It is probably best to use both a glyph and a metrics cache to avoid
recomputation.

### Which anti-aliasing algorithm is used by FreeType 2?

The algorithm has been specifically designed for FreeType.  It is based on
ideas that were originally found in the implementation of the
[libArt](http://www.levien.com/libart/) graphics library to compute the _exact
pixel coverage_ of a vector image with no sub-sampling and filtering.

However, these two implementations are radically distinct and use vastly
different models.  The FreeType 2 renderer is optimized specifically for
rendering small complex shapes, like glyphs, at very high speed while using
very few memory.  On the other hand, libArt has been designed for general shape
and polygon processing, especially large ones.

The FreeType 2 anti-aliasing renderer is indeed _faster_ than the monochrome
renderer for small character sizes (typically <20 pixels).  The reason is that
the monochrome renderer must perform two passes on the outline in order to
perform drop-out control according to the TrueType specification.

### When will FreeType 2 support OpenType?

Well, the engine already reads OpenType/CFF files perfectly.  What it doesn't
do is handling ‘OpenType Layout’ tables.

FreeType 1 comes with a set of extensions that are used to load and manage
OpenType Layout tables.  It even has a demonstration program named `ftstrtto`
to show its capabilities.  However, this code is no longer maintained, and we
strongly advise to not use it.

For FreeType 2, we have decided that the layout operations provided through
these tables are better placed in a specific text-layout library like
[Pango](http://www.pango.org).
