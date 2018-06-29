---
layout: page
title: FreeType 2 Glyph Conventions
sidebar: docs_sidebar
permalink: docs/glyphs.html
nav: glyphs
---

## I. Basic Typographic Concepts

### 1\. Font files, format and information

A _font_ is a collection of various character images that can be used to
display or print text.  The images in a single font share some common
properties, including look, style, serifs, etc.  Typographically speaking, one
has to distinguish between a _font family_ and its multiple _font faces_, which
usually differ in style though come from the same template.

For example,  'Palatino Regular' and'Palatino Italic' are two distinct
_faces_ from the same _family_, called 'Palatino' itself.

The single term  'font' is nearly always used in ambiguous ways to refer to
either a given family or given face, depending on the context.  For example,
most users of word-processors use  'font' to describe a font family (e.g.
'Courier','Palatino', etc.); however, most of these families are implemented
through several data files depending on the file format: For TrueType, this is
usually one per face (i.e. `arial.ttf` for 'Arial Regular',
`ariali.ttf` for 'Arial Italic', etc.).  The file is also called a
'font' but really contains a font face.

A _digital font_ is thus a data file that may contain _one or more font faces_.
For each of these, it contains character images, character metrics, as well as
other kind of information important to the layout of text and the processing of
specific character encodings.  In some formats, like Adobe's Type 1, a single
font face is described through several files (i.e., one contains the character
images, another one the character metrics).  We will ignore this implementation
issue in most parts of this document and consider digital fonts as single
files, though FreeType 2 is able to support multiple-files fonts correctly.

As a convenience, a font file containing more than one face is called a _font
collection_.  This case is rather rare but can be seen in many Asian fonts,
which contain images for two or more representation forms of a given scripts
(usually for horizontal and vertical layout.

### 2\. Character images and mappings

The character images are called _glyphs_.  A single character can have several
distinct images, i.e. several glyphs, depending on script, usage or context.
Several characters can also take a single glyph (good examples are Roman
ligatures like  'fi' and 'fl' which can be represented by a single glyph).
The relationship between characters and glyphs can be very complex but won't be
discussed in this document.  Moreover, some formats use more or less
complicated schemes to store and access glyphs.  For the sake of clarity, we
only retain the following notions when working with FreeType:

* A font file contains a set of glyphs; each one can be stored as a bitmap, a
  vector representation, or any other scheme (most scalable formats use a
  combination of mathematical representation and control data/programs).  These
  glyphs can be stored in any order in the font file, and are typically
  accessed through a simple glyph index.

* The font file contains one or more tables, called _character maps_ (also
  called 'charmaps' or'cmaps' for short), which are used to convert character
  codes for a given encoding (e.g. ASCII, Unicode, DBCS, Big5, etc.) into glyph
  indices relative to the font file.  A single font face may contain several
  charmaps.  For example, many TrueType fonts contain an Apple-specific charmap
  as well as a Unicode charmap, which makes them usable on both Mac and Windows
  platforms.

### 3\. Character and font metrics

Each glyph image is associated with various metrics which describe how to place
and manage it when rendering text; see [section III](glyphs-3.html) for more.
Metrics relate to glyph placement, cursor advances as well as text layout.
They are extremely important to compute the flow of text when rendering a
string of text.

Each scalable format also contains some global metrics, expressed in notional
(i.e. font) units, to describe some properties of all glyphs in the same face.
Examples for global metrics are the maximum glyph bounding box, the ascender,
descender, and text height of the font.

Non-scalable formats contain metrics also.  However, they only apply to a set
of given character dimensions and resolutions, and are usually expressed in
pixels then.

## II. Glyph Outlines

This section describes the way scalable representations of glyph images, called
outlines, are used by FreeType as well as client applications.

### 1\. Pixels, points and device resolutions

Though it is a very common assumption when dealing with computer graphics
programs, the physical dimensions of a given pixel (be it for screens or
printers) are not squared.  Often, the output device, be it a screen device or
a printer, exhibits varying resolutions in both horizontal and vertical
directions, and this must be taken care of when rendering text.

It is thus common to define a device's characteristics through two numbers
expressed in _dpi_ (dots per inch).  For example, a printer with a resolution
of 300×600 dpi has 300 pixels per inch in the horizontal direction, and 600 in
the vertical one.  The resolution of a typical computer monitor varies with its
size (10″ and 25″ monitors don't have the same pixel sizes at 1024×768), and of
course the graphics mode resolution.

As a consequence, the size of text is usually given in _points_, rather than
device-specific pixels.  Points are a _physical_ unit, where 1 point equals
1/72th of an inch in digital typography.  As an example, most books using the
Latin script are printed with a body text size somewhere between 10 and 14
points.

It is thus possible to compute the size of text in pixels from the size in
points with the following formula:

<center>`pixel_size = point_size * resolution / 72`</center>

The resolution is expressed in _dpi_.  Since horizontal and vertical
resolutions may differ, a single point size usually defines a different text
width and height in pixels.

Unlike what is often thought, the  'size of text in pixels' is not directly
related to the real dimensions of characters when they are displayed or
printed.  The relationship between these two concepts is a bit more complex and
depend on some design choices made by the font designer.  This is described in
more detail in the next sub-section (see the explanations on the EM square).

### 2\. Vectorial representation

The source format of outlines is a collection of closed paths called
_contours_.  Each contour delimits an outer or inner _region_ of the glyph, and
can be made of either _line segments_ or _Bézier arcs_.

The arcs are defined through _control points_, and can be either second-order
(these are _conic_ Béziers) or third-order (_cubic_ Béziers) polynomials,
depending on the font format.  Note that conic Béziers are usually called
_quadratic_ Béziers in the literature.  Hence, FreeType associates each point
of the outline with flag to indicate its type (normal or control point).  And
scaling the points will scale the whole outline.

Each glyph's original outline points are located on a grid of indivisible
units.  The points are usually stored in a font file as 16-bit integer grid
coordinates, with the grid's origin being at (0,0); they thus range from -32768
to 32767\.  (Even though point coordinates can be floats in other formats such
as Type 1, we will restrict our analysis to integer values for simplicity).

The grid is always oriented like the traditional mathematical two-dimensional
plane, i.e., the _X_ axis goes from the left to the right, and the _Y_ axis
from bottom to top.

In creating the glyph outlines, a type designer uses an imaginary square called
the _EM square_.  Typically, the EM square can be thought of as a tablet on
which the characters are drawn.  The square's size, i.e., the number of grid
units on its sides, is very important for two reasons:

* It is the reference size used to scale the outlines to a given text
  dimension.  For example, a size of 12pt at 300×300 dpi corresponds to
  12\*300 / 72 = 50 pixels.  This is the size the EM square would appear on the
  output device if it was rendered directly.  In other words, scaling from grid
  units to pixels uses the formula:

      `pixel_size = point_size * resolution / 72`
      `pixel_coord = grid_coord * pixel_size / EM_size`

* The greater the EM size is, the larger resolution the designer can use when
  digitizing outlines.  For example, in the extreme example of an EM size of 4
  units, there are only 25 point positions available within the EM square which
  is clearly not enough.  Typical TrueType fonts use an EM size of 2048 units;
  Type 1 or CFF PostScript fonts traditionally use an EM size of 1000 grid
  units (but point coordinates can be expressed as floating values).

Note that glyphs can freely extend beyond the EM square if the font designer
wants so.  The EM square is thus just a convention in traditional typography.

Grid units are very often called _font units_ or _EM units_.

As said before, `pixel_size` computed in the above formula does not directly
relate to the size of characters on the screen.  It simply is the size of the
EM square if it was to be displayed.  Each font designer is free to place its
glyphs as it pleases him within the square.  This explains why the letters of
the following text have not the same height, even though they are displayed at
the same point size with distinct fonts:

![Comparison of font heights](body_comparison.png)

As one can see, the glyphs of the Courier family are smaller than those of
Times New Roman, which themselves are slightly smaller than those of Arial,
even though everything is displayed or printed at a size of 16 points.  This
only reflects design choices.

### 3\. Hinting and Bitmap rendering

The outline as stored in a font file is called the 'master' outline, as its
points coordinates are expressed in font units.  Before it can be converted
into a bitmap, it must be scaled to a given size and resolution.  This is done
with a very simple transformation, but always creates undesirable artifacts, in
particular stems of different widths or heights in letters like  'E' or 'H'.

As a consequence, proper glyph rendering needs the scaled points to be aligned
along the target device pixel grid, through an operation called _grid-fitting_
(often called _hinting_).  One of its main purposes is to ensure that important
widths and heights are respected throughout the whole font (for example, it is
very often desirable that the  'I' and the 'T' have their central vertical line
of the same pixel width), as well as to manage features like stems and
overshoots, which can cause problems at small pixel sizes.

There are several ways to perform grid-fitting properly; most scalable formats
associate some control data or programs with each glyph outline.  Here is an
overview:

* explicit grid-fitting

  The TrueType format defines a stack-based virtual machine, for which programs
  can be written with the help of more than 200 opcodes (most of them relating
  to geometrical operations).  Each glyph is thus made of both an outline and a
  control program to perform the actual grid-fitting in the way defined by the
  font designer.

* implicit grid-fitting (also called hinting)

  The Type 1 and CFF formats take a much simpler approach: Each glyph is made
  of an outline as well as several pieces called _hints_ which are used to
  describe some important features of the glyph, like the presence of stems,
  some width regularities, and the like.  There aren't a lot of hint types, and
  it is up to the final renderer to interpret the hints in order to produce a
  fitted outline.

* automatic grid-fitting

  Some formats include no control information with each glyph outline, apart
  from font metrics like the advance width and height.  It is then up to the
  renderer to  'guess' the more interesting features of the outline in order to
  perform some decent grid-fitting.

The following table summarizes the pros and cons of each scheme.

<table><thead><tr><th align="center">**grid-fitting scheme**</th><th align="center">**advantages**</th><th align="center">**disadvantages**</th></tr></thead><tbody><tr><td valign="top" align="center">

**explicit**

</td><td valign="top">

**Quality.**  Excellent results at small sizes
                    are possible.  This is very important for screen
                    display.

**Consistency.**  All renderers produce the
                    same glyph bitmaps (at least in theory).

</td><td valign="top">

**Speed.**  Interpreting bytecode can be slow
                    if the glyph programs are complex.

**Size.**  Glyph programs can be long.

**Technical difficulty.**  It is extremely
                    difficult to write good hinting programs.  Very
                    few tools available.

</td></tr><tr><td valign="top" align="center">

**implicit**

</td><td valign="top">

**Size.**  Hints are usually much smaller than
                    explicit glyph programs.

**Speed.**  Grid-fitting is usually a fast
                    process.

</td><td valign="top">

**Quality.**  Often questionable at small
                    sizes.  Better with anti-aliasing though.

**Inconsistency.**  Results can vary between
                    different renderers, or even distinct versions of
                    the same engine.

</td></tr><tr><td valign="top" align="center">

**automatic**

</td><td valign="top">

**Size.**  No need for control information,
                    resulting in smaller font files.

**Speed.**  Depends on the grid-fitting
                    algorithm.  Usually faster than explicit
                    grid-fitting.

</td><td valign="top">

**Quality.**  Often questionable at small
                    sizes.  Better with anti-aliasing though.

**Speed.**  Depends on the grid-fitting
                    algorithm.

**Inconsistency.**  Results can vary between
                    different renderers, or even distinct versions
                    of the same engine.

</td></tr></tbody></table>

## III. Glyph Metrics

### 1\. Baseline, pens and layouts

The baseline is an imaginary line that is used to 'guide' glyphs when rendering
text.  It can be horizontal (e.g. Latin, Cyrillic, Arabic, etc.) or vertical
(e.g. Chinese, Japanese, Mongolian, etc.).  Moreover, to render text, a virtual
point, located on the baseline, called the _pen position_ or _origin_, is used
to locate glyphs.

Each layout uses a different convention for glyph placement:

* With horizontal layout, glyphs simply 'rest' on the baseline.  Text is
  rendered by incrementing the pen position, either to the right or to the
  left.

  The distance between two successive pen positions is glyph-specific and is
  called the _advance width_.  Note that its value is _always_ positive, even
  for right-to-left oriented scripts like Arabic.  This introduces some
    differences in the way text is rendered.

  _The pen position is always placed on the baseline._

  ![horizontal layout](Image1.png)

* With a vertical layout, glyphs are centered around the baseline:

  ![vertical layout](Image2.png)

### 2\. Typographic metrics and bounding boxes

A various number of face metrics are defined for all glyphs in a given font.

* Ascent

  The distance from the baseline to the highest or upper grid coordinate used
  to place an outline point.  It is a positive value, due to the grid's
  orientation with the _Y_ axis upwards.

* Descent

  The distance from the baseline to the lowest grid coordinate used to place an
  outline point.  In FreeType, this is a negative value, due to the grid's
  orientation.  Note that in some font formats this is a positive value.

* Linegap

  The distance that must be placed between two lines of text.  The
  baseline-to-baseline distance should be computed as

  `linespace = ascent - descent + linegap`

  if you use the typographic values.

Other, simpler metrics are:

* Bounding box

  This is an imaginary box that encloses all glyphs from the font, usually as
  tightly as possible.  It is represented by four parameters, namely `xMin`,
  `yMin`, `xMax`, and `yMax`, that can be computed for any outline.  Their
  values can be in font units if measured in the original outline, or in
  integer (or fractional) pixel units when measured on scaled outlines.

  A common shorthand for the bounding box is 'bbox'.

* Internal leading

  This concept comes directly from the world of traditional typography.  It
  represents the amount of space within the _leading_ which is reserved for
  glyph features that lay outside of the EM square (like accentuation).  It
  usually can be computed as

  `internal leading = ascent - descent - EM_size`

* External leading

  This is another name for the line gap.

### 3\. Bearings and Advances

Each glyph has also distances called _bearings_ and _advances_.  The actual
values depend on the layout, as the same glyph can be used to render text
either horizontally or vertically:

* Left side bearing

  The horizontal distance from the current pen position to the glyph's left
  bbox edge.  It is positive for horizontal layouts, and in most cases negative
  for vertical ones.

  In the FreeType API, this is also called `bearingX`.  Another shorthand is
  'lsb'.

* Top side bearing

  The vertical distance from the baseline to the top of the glyph's bbox.  It
  is usually positive for horizontal layouts, and negative for vertical ones.

  In the FreeType API, this is also called `bearingY`.

* Advance width

  The horizontal distance to increment (for left-to-right writing) or decrement
  (for right-to-left writing) the pen position after a glyph has been rendered
  when processing text.  It is always positive for horizontal layouts, and zero
  for vertical ones.

  In the FreeType API, this is also called `advanceX`.

* Advance height

  The vertical distance to decrement the pen position after a glyph has been
  rendered.  It is always zero for horizontal layouts, and positive for
  vertical layouts.

  In the FreeType API, this is also called `advanceY`.

* Glyph width

  The glyph's horizontal extent.  For unscaled font coordinates, it is

  `glyph width = bbox.xMax - bbox.xMin`

  For scaled glyphs, its computation requests specific care, described in the
  grid-fitting chapter below.

* Glyph height

  The glyph's vertical extent. For unscaled font coordinates, it is

  `glyph height = bbox.yMax - bbox.yMin`

  For scaled glyphs, its computation requests specific care, described in the
  grid-fitting chapter below.

* Right side bearing

  Only used for horizontal layouts to describe the distance from the bbox's
  right edge to the advance width.  In most cases it is a non-negative number:

  `right side bearing = advance_width - left_side_bearing - (xMax-xMin)`

  A common shorthand for this value is 'rsb'.

Here is a picture giving all the details for horizontal metrics:

![horizontal glyph metrics](Image3.png)

And here is another one for the vertical metrics:

![vertical glyph metrics](Image4.png)

### 4\. The effects of grid-fitting

Because hinting aligns the glyph's control points to the pixel grid, this
process slightly modifies the dimensions of character images in ways that
differ from simple scaling.

For example, the image of the lowercase 'm' letter sometimes fits a square in
the master grid.  However, to make it readable at small pixel sizes, hinting
tends to enlarge its scaled outline horizontally in order to keep its three
legs distinctly visible, resulting in a wider character bitmap.

The glyph metrics are also influenced by the grid-fitting process:

* The image's width and height are altered.  Even if this is only by one pixel,
  it can make a big difference at small pixel sizes.

* The image's bounding box is modified, thus modifying the bearings.

* The advances must be updated.  For example, the advance width must be
  incremented if the hinted bitmap is larger than the scaled one, to reflect
  the augmented glyph width.

This has some implications:

* Because of hinting, simply scaling the font ascent or descent might not give
  correct results.  A possible solution is to keep the ceiling of the scaled
  ascent, and floor of the scaled descent.

* There is no easy way to get the hinted glyph and advance widths of a range of
  glyphs, as hinting works differently on each outline.  The only solution is
  to hint each glyph separately and record the returned values (for example in
  a cache).  Some formats, like TrueType, even include a table of pre-computed
  values for a small set of common character pixel sizes.

* Hinting depends on the final character width and height in pixels, which
  means that it is highly resolution-dependent.  This property makes correct
  WYSIWYG layouts difficult to implement.

Performing 2D transformations on glyph outlines is very easy with FreeType.
However, when using translation on hinted outlines, one should always take care
of **exclusively using integer pixel distances** (which means that the
parameters to the `FT_Outline_Translate` API function should all be multiples
of 64, as the point coordinates are in 26.6 fixed-point format).  Otherwise,
the translation will simply _ruin the hinter's work_, resulting in very low
quality bitmaps!

### 5\. Text widths and bounding box

As seen before, the  'origin' of a given glyph corresponds to the position of
the pen on the baseline.  It is not necessarily located on one of the glyph's
bounding box corners, unlike many typical bitmapped font formats.  In some
cases, the origin can be out of the bounding box, in others, it can be within
it, depending on the shape of the given glyph.

Likewise, the glyph's'advance width ' is the increment to apply to the pen
position during layout, and is not related to the glyph's'width ', which really
is the glyph's bounding width.

The same conventions apply to strings of text.  This means that:

* The bounding box of a given string of text doesn't necessarily contain the
  text cursor, nor is the latter located on one of its corners.

* The string's advance width isn't related to its bounding box dimensions.
  Especially if it contains beginning and terminal spaces or tabs.

* Finally, additional processing like kerning creates strings of text whose
  dimensions are not directly related to the simple juxtaposition of individual
  glyph metrics.  For example, the advance width of  'VA' isn't the sum of the
  advances of'V ' and'A ' take separately.

## IV. Kerning

The term _kerning_ refers to specific information used to adjust the relative
positions of successive glyphs in a string of text.  This section describes
several types of kerning information, as well as the way to process them when
performing text layout.

### 1\. Kerning pairs

Kerning consists of modifying the spacing between two successive glyphs
according to their outlines.  For example, a  'T' and a 'y' can be easily moved
closer, as the top of the  'y' fits nicely under the upper right bar of the
'T'.

When laying out text with only their standard widths, some consecutive glyphs
seem a bit too close or too distant.  For example, the space between the 'A'
and the 'V' in the following word seems a little wider than needed.

![the word 'bravo' unkerned](bravo_unkerned.png)

Compare this to the same word, where the distance between these two letters has
been slightly reduced:

![the word 'bravo' with kerning](bravo_kerned.png)

As you can see, this adjustment can make a great difference.  Some font faces
thus include a table containing kerning distances for a set of given glyph
pairs for text layout.

* The pairs are ordered, i.e., the space for pair '(A,V)' isn't necessarily the
  space for pair  '(V,A)'.  They also use glyph indices, not character codes.

* Kerning distances can be expressed in horizontal or vertical directions,
  depending on the layout and/or the script.  For example, some horizontal
  layouts like Arabic can make use of vertical kerning adjustments between
  successive glyphs.  A vertical script can have vertical kerning distances.

* Kerning distances are expressed in grid units.  They are usually oriented in
  the _X_ axis, which means that a negative value indicates that two glyphs
  must be set closer in a horizontal layout.

Note that OpenType fonts (OTF) provide two distinct mechanisms for kerning,
using the  'kern' and 'GPOS' tables, respectively, which are part of the OTF
files.  Older fonts only contain the former, while recent fonts contain both
tables or even 'GPOS' data only.  FreeType only supports kerning via the
(rather simple)  'kern' table.  For the interpretation of kerning data in the
(highly sophisticated)  'GPOS' table you need a higher-level library like
[ICU](http://icu-project.org/) or [HarfBuzz](http://harfbuzz.org) since it can
be context dependent (this is, the kerning may vary depending on the position
within a text string, for example).

### 2\. Applying kerning

Applying kerning when rendering text is a rather easy process.  It merely
consists in adding the scaled kern distance to the pen position before
rendering the next glyph.  However, the typographically correct renderer must
take a few more details in consideration.

The  'sliding dot' problem is a good example: Many font faces include a kerning
distance between capital letters like  'T' or 'F' and a following dot ( '.'),
in order to slide the latter glyph just right to their main leg.

![example for sliding dots](twlewis1.png)

This sometimes requires additional adjustments between the dot and the letter
following it, depending on the shapes of the enclosing letters.  When applying
'standard' kerning adjustments, the previous sentence would become

![example for too much kerning](twlewis2.png)

This is clearly too contracted.  The solution here, as exhibited in the first
example, is to only slide the dots if the conditions fit.  Of course, this
requires a certain knowledge of the text's meaning, and this is exactly what
'GPOS' kerning is good for: Depending on the context, different kerning values
can be applied to get a typographically correct result.

## V. Text Processing 

This section demonstrate algorithms which use the concepts previously defined
to render text, whatever layout you use.  It assumes _simple_ text handling
suitable for scripts like Latin or Cyrillic, using a one-to-one relationship
between input character codes and output glyphs indices.  Scripts like Arabic
or Khmer, which need a  'shaping engine' to do the character code to glyph
index conversion, are beyond the scope (and should be done by proper layout
engines like [Pango](http://www.pango.org/) anyway).

### 1\. Writing simple text strings

In this first example, we will generate a simple string of text in the Latin
script, i.e. with a horizontal left-to-right layout.  Using exclusively pixel
metrics, the process looks like:

1. Convert the character string into a series of glyph indices.

2. Place the pen to the cursor position.

3. Get or load the glyph image.

4. Translate the glyph so that its  'origin' matches the pen position.

5. Render the glyph to the target device.

6. Increment the pen position by the glyph's advance width (in pixels).

7. Start over at step 3 for each of the remaining glyphs.

8. When all glyphs are done, set the text cursor to the new pen position.

Note that kerning isn't part of this algorithm.

### 2\. Pseudo-subpixel positioning

It is somewhat useful to use subpixel positioning when rendering text.  This is
crucial, for example, to provide semi-WYSIWYG text layouts.  Text rendering is
very similar to the algorithm described in subsection 1, with the following few
differences:

* The pen position is expressed in fractional pixels.

* Because translating a hinted outline by a non-integer distance will ruin its
  grid-fitting, the position of the glyph origin must be rounded before
  rendering the character image.

* The advance width is expressed in fractional pixels, and isn't necessarily an
  integer.

Here an improved version of the algorithm:

1. Convert the character string into a series of glyph indices.

2. Place the pen to the cursor position.  This can be a non-integer point.

3. Get or load the glyph image.

4. Translate the glyph so that its  'origin' matches the rounded pen position.

5. Render the glyph to the target device.

6. Increment the pen position by the glyph's advance width in fractional
pixels.

7. Start over at step 3 for each of the remaining glyphs.

8. When all glyphs are done, set the text cursor to the new pen position.

Note that with fractional pixel positioning, the space between two given
letters isn't fixed, but determined by the accumulation of previous rounding
errors in glyph positioning.  For auto-hinted glyphs, this problem can be
alleviated by using the `lsb_delta` and `rsb_delta` values (see the
documentation of the
[FT_GlyphSlotRec](../reference/ft2-base_interface.html#FT_GlyphSlotRec)
structure for more details).

TODO: Real subpixel positioning with glyph shifting before hinting.

### 3\. Simple kerning

Adding kerning to the basic text rendering algorithm is easy: When a kerning
pair is found, simply add the scaled kerning distance to the pen position
before step 4.  Of course, the distance should be rounded in the case of
algorithm 1, though it doesn't need to for algorithm 2.  This gives us:

Algorithm 1 with kerning:

1. Convert the character string into a series of glyph indices.

2. Place the pen to the cursor position.

3. Get or load the glyph image.

4. Add the rounded scaled kerning distance, if any, to the pen position.

5. Translate the glyph so that its  'origin' matches the pen position.

6. Render the glyph to the target device.

7. Increment the pen position by the glyph's advance width in pixels.

8. Start over at step 3 for each of the remaining glyphs.

Algorithm 2 with kerning:

1. Convert the character string into a series of glyph indices.

2. Place the pen to the cursor position.

3. Get or load the glyph image.

4. Add the scaled unrounded kerning distance, if any, to the pen position.

5. Translate the glyph so that its  'origin' matches the rounded pen position.

6. Render the glyph to the target device.

7. Increment the pen position by the glyph's advance width in fractional
   pixels.

8. Start over at step 3 for each of the remaining glyphs.

### 4\. Right-to-left layout

The process of laying out right-to-left scripts like (modern) Hebrew text is
very similar.  The only difference is that the pen position must be decremented
before the glyph rendering (remember: the advance width is always positive,
even for Hebrew glyphs).

Right-to-left algorithm 1:

1. Convert the character string into a series of glyph indices.

2. Place the pen to the cursor position.

3. Get or load the glyph image.

4. Decrement the pen position by the glyph's advance width in pixels.

5. Translate the glyph so that its  'origin' matches the pen position.

6. Render the glyph to the target device.

7. Start over at step 3 for each of the remaining glyphs.

The changes to algorithm 2, as well as the inclusion of kerning are left as an
exercise to the reader.

### 5\. Vertical layouts

Laying out vertical text uses exactly the same processes, with the following
significant differences:

* The baseline is vertical, and the vertical metrics must be used instead of
  the horizontal one.

* The left bearing is usually negative, but this doesn't change the fact that
  the glyph origin must be located on the baseline.

* The advance height is always positive, so the pen position must be
  decremented if one wants to write top to bottom (assuming the _Y_ axis is
  oriented upwards).

Here the algorithm:

1. Convert the character string into a series of glyph indices.

2. Place the pen to the cursor position.

3. Get or load the glyph image.

4. Translate the glyph so that its  'origin' matches the pen position.

5. Render the glyph to the target device.

6. Decrement the vertical pen position by the glyph's advance height in pixels.

7. Start over at step 3 for each of the remaining glyphs.

8. When all glyphs are done, set the text cursor to the new pen position.

## VI. FreeType outlines

The purpose of this section is to present the way FreeType manages vectorial
outlines, as well as the most common operations that can be applied on them.

### 1\. FreeType outline description and structure

#### a. Outline curve decomposition

An outline is described as a series of closed contours in the 2D plane.  Each
contour is made of a series of line segments and Bézier arcs.  Depending on the
file format, these can be second-order or third-order polynomials.  The former
are also called quadratic or conic arcs, and they are used in the TrueType
format.  The latter are called cubic arcs and are mostly used in the PostScript
Type 1 and Type formats.

Each arc is described through a series of start, end, and control points.  Each
point of the outline has a specific tag which indicates whether it is describes
a line segment or an arc.  The tags can take the following values:

Tag                  | Description
---------------------|------------
`FT_CURVE_TAG_ON`    | Used when the point is  'on' the curve. This corresponds to start and end points of segments and arcs.  The other tags specify what is called an 'off' point, i.e., a point which isn't located on the contour itself, but serves as a control point for a Bézier arc.
`FT_CURVE_TAG_CONIC` | Used for an  'off' point used to control a conic Bézier arc.
`FT_CURVE_TAG_CUBIC` | Used for an  'off' point used to control a cubic Bézier arc.

Use the `FT_CURVE_TAG(tag)` macro to filter out other, internally used flags.

The following rules are applied to decompose the contour's points into segments
and arcs:

* Two successive  'on' points indicate a line segment joining them.

* One conic  'off' point between two 'on' points indicates a conic Bézier arc,
  the  'off' point being the control point, and the  'on' ones the start and
  end points.

* Two successive cubic  'off' points between two  'on' points indicate a cubic
  Bézier arc.  There must be exactly two cubic control points and two  'on'
  points for each cubic arc (using a single cubic  'off' point between two
  'on' points is forbidden, for example).

* Two successive conic  'off' points force the rasterizer to create (during the
  scan-line conversion process exclusively) a virtual 'on' point inbetween, at
  their exact middle.  This greatly facilitates the definition of successive
  conic Bézier arcs.  Moreover, it is the way outlines are described in the
  TrueType specification.

* The last point in a contour uses the first as an end point to create a closed
  contour.  For example, if the last two points of a contour were an  'on'
  point followed by a conic  'off' point, the first point in the contour would
  be used as final point to create an  'on' – 'off' –'on' sequence as described
  above.

* The first point in a contour can be a conic 'off' point itself; in that case,
  use the last point of the contour as the contour's starting point.  If the
  last point is a conic  'off' point itself, start the contour with the virtual
  'on' point between the last and first point of the contour.

Note that it is possible to mix conic and cubic arcs in a single contour,
however, no font driver of FreeType produces such outlines currently.

<table><tbody><tr><td>![segment example](points_segment.png)</td><td>![conic arc example](points_conic.png)</td></tr><tr><td>![cubic arc example](points_cubic.png)</td><td>![cubic arc with virtual 'on' point](points_conic2.png)</td></tr></tbody></table>

#### b. The `FT_Outline` descriptor

A FreeType outline is described through a simple structure called
[`FT_Outline`](../reference/ft2-outline_processing.html#FT_Outline).  Right
now, the following fields are of interest:

Field        | Description
-------------|------------
`n_points`   | The number of points in the outline
`n_contours` | The number of contours in the outline
`points`     | Array of point coordinates
`contours`   | Array of contour end indices
`tags`       | Array of point flags

Here, `points` is a pointer to an array of
[`FT_Vector`](../reference/ft2-basic_types.html#FT_Vector) records, used to
store the vectorial coordinates of each outline point.  These are expressed in
1/64th of a pixel, which is also known as the _26.6 fixed-point format_.

`contours` is an array of point indices to delimit contours in the outline.
For example, the first contour always starts at point 0, and ends at point
`contours[0]`.  The second contour starts at point `contours[0]+1` and ends at
`contours[1]`, etc.  To traverse these points in a callback based manner, use
[`FT_Outline_Decompose`](../reference/ft2-outline_processing.html#FT_Outline_Decompose).

Note that each contour is closed, and that value of `n_points` should be equal
to `contours[n_contours-1]+1` for a valid outline.

Finally, `tags` is an array of bytes, used to store each outline point's tag.

### 2\. Bounding and control box computations

As described earlier, a _bounding box_ (also called _bbox_) is simply a
rectangle that completely encloses the shape of a given outline.  The
interesting case is the smallest bounding box possible, and in the following we
subsume this under the term 'bounding box'.  Because of the way arcs are
defined, Bézier control points are not necessarily contained within an
outline's (smallest) bounding box.

Such a situation happens if one Bézier arc is, for example, the upper edge of
an outline and an 'off' point happens to be above the bbox.  However, it is
very rare in the case of character outlines because most font designers and
creation tools always place  'on' points at the extrema of each curved edges
(as both the TrueType and PostScript specifications recommend), as it makes
hinting much easier.

We thus define the _control box_ (also called _cbox_) as the smallest possible
rectangle that encloses all points of a given outline (including its 'off'
points).  Clearly, it always includes the bbox, and the two boxes are identical
in most cases.

Unlike the bbox, the cbox is much faster to compute.

<table><tbody><tr><td>![a glyph with different bbox and
cbox](bbox1.png)</td><td>![a glyph with identical bbox and
cbox](bbox2.png)</td></tr></tbody></table>

Control and bounding boxes can be computed automatically using the functions
[`FT_Outline_Get_CBox`](../reference/ft2-outline_processing.html#FT_Outline_Get_CBox)
and
[`FT_Outline_Get_BBox`](../reference/ft2-outline_processing.html#FT_Outline_Get_BBox).
The former function is always very fast, while the latter _may_ be slow in the
case of 'outside' control points (as it needs to find the extreme of conic and
cubic arcs for 'perfect' computations).  If this isn't the case, it is as fast
as computing the control box.

Note also that even though most glyph outlines have equal cbox and bbox values
to ease hinting, this is not necessarily the case if a transformation like
rotation is applied to them.

### 3\. Coordinates, scaling and grid-fitting

An outline point's vectorial coordinates are expressed in the 26.6 format, i.e.
in 1/64th of a pixel, hence the coordinates  '(1.0,-2.5)' is stored as the
integer pair  '(64,-192)', to name an example.

After a glyph outline is scaled from the EM grid (in font units) to the current
character dimensions, the hinter or grid-fitter is in charge of aligning
important outline points (mainly edge delimiters) to the pixel grid.  Even
though this process is much too complex to be described in a few lines, its
purpose is mainly to round point positions, while trying to preserve important
properties like widths, stems, etc.

The following operations can be used to round vectorial distances in the 26.6
format to the grid:

    round( x )   == ( x + 32 ) & -64
    floor( x )   ==          x & -64
    ceiling( x ) == ( x + 63 ) & -64

Once a glyph outline is grid-fitted or transformed, it often is interesting to
compute the glyph image's pixel dimensions before rendering it.  To do so, one
has to consider the following:

The scan-line converter draws all the pixels whose _centers_ fall inside the
glyph shape.  In B/W rendering mode, it can also detect _drop-outs_, i.e.,
discontinuities coming from extremely thin shape fragments, in order to draw
the  'missing' pixels.  These new pixels are always located at a distance less
than half of a pixel but it is not easy to predict where they will appear
before rendering.

This leads to the following computations:

* compute the bbox

* grid-fit the bounding box with the following:

      xmin = floor( bbox.xMin )
      xmax = ceiling( bbox.xMax )
      ymin = floor( bbox.yMin )
      ymax = ceiling( bbox.yMax )

* return pixel dimensions, i.e.

      width = (xmax - xmin)/64
  
  and

      height = (ymax - ymin)/64

By grid-fitting the bounding box, it is guaranteed that all the pixel centers
that are to be drawn, _including those coming from drop-out control_, will be
_within_ the adjusted box.  Then the box's dimensions in pixels can be
computed.

Note also that, when translating a grid-fitted outline, one should _always use
integer distances_ to move an outline in the 2D plane.  Otherwise, glyph edges
won't be aligned on the pixel grid anymore, and the hinter's work will be lost,
producing _very low quality_ bitmaps and pixmaps.

## VII. FreeType Bitmaps

The purpose of this section is to present the way FreeType manages bitmaps and
pixmaps, and how they relate to the concepts previously defined.  The
relationship between vectorial and pixel coordinates is explained.

### 1\. Vectorial versus pixel coordinates

This sub-section explains the difference between vectorial and pixel
coordinates.  To make things clear, brackets will be used to describe pixel
coordinates, e.g.  '[3,5]', while parentheses will be used for vectorial ones,
e.g.  '(-2, 3.5)'.

In the pixel case, as we use the _Y upwards_ convention; the coordinate [0, 0]
always refers to the _lower left pixel_ of a bitmap, while coordinate [width-1,
rows-1] to its _upper right pixel_.

In the vectorial case, point coordinates are expressed in floating units, like
(1.25, -2.3).  Such a position doesn't refer to a given pixel, but simply to an
immaterial point in the 2D plane.

The pixels themselves are indeed _square boxes_ of the 2D plane, whose centers
lie in half pixel coordinates.  For example, the lower left pixel of a bitmap
is delimited by the square (0, 0)-(1, 1), its center being at location (0.5,
0.5).

This introduces some differences when computing distances.  For example, the
_length_ in pixels of the line [0, 0]-[10, 0] is 11\.  However, the vectorial
distance between (0, 0)-(10, 0) covers exactly 10 pixel centers, hence its
length is 10.

![bitmap and vector grid](grid_1.png)

### 2\. The `FT_Bitmap` descriptor

In FreeType, a bitmap or pixmap is described through a single structure, called
[`FT_Bitmap`](../reference/ft2-basic_types.html#FT_Bitmap).  The fields we are
interested in are:

Field        | Description
-------------|------------
`rows`       | The number of rows, i.e. lines, in the bitmap
`width`      | The number of horizontal pixels in the bitmap
`pitch`      | Its absolute value is the number of bytes per bitmap line; it can be either positive or negative depending on the bitmap's vertical orientation
`buffer`     | A typeless pointer to the bitmap pixel buffer
`pixel_mode` | An enumeration used to describe the pixel format of the bitmap; examples are `ft_pixel_mode_mono` for 1-bit monochrome bitmaps and `ft_pixel_mode_grays` for 8-bit anti-aliased 'gray' values
`num_grays`  |  This is only used for  'gray' pixel modes, it gives the number of gray levels used to describe the anti-aliased gray levels (256 by default with FreeType 2)

Note that the sign of the `pitch` field determines whether the rows in the
pixel buffer are stored in ascending or descending order.

Remember that FreeType uses the _Y upwards_ convention in the 2D plane, which
means that a coordinate of (0, 0) always refer to the _lower-left corner_ of a
bitmap.

If the pitch is positive, the rows are stored in decreasing vertical position;
the first bytes of the pixel buffer are part of the _upper_ bitmap row.

On the opposite, if the pitch is negative, the first bytes of the pixel buffer
are part of the _lower_ bitmap row.

In all cases, one can see the pitch as the byte increment needed to skip to the
_next lower scanline_ in a given bitmap buffer.

<table><tbody><tr><td>![negative
'pitch'](up_flow.png)</td><td>![positive'pitch'](down_flow.png)</td></tr></tbody></table>

The  'positive pitch' convention is very often used, though some systems might
need the other.

### 3\. Converting outlines into bitmaps and pixmaps

Generating a bitmap or pixmap image from a vectorial image is easy with
FreeType.  However, one must understand a few points regarding the positioning
of the outline in the 2D plane before converting it to a bitmap:

* The glyph loader and hinter always places the outline in the 2D plane so that
  (0, 0) matches its character origin.  This means that the glyph's outline
  (and corresponding bounding box), can be placed anywhere in the 2D plane (see
  the graphics in section III).

* The target bitmap's area is mapped to the 2D plane, with its lower left
  corner at (0, 0).  This means that a bitmap or pixmap of dimensions [`w`,
  `h`] will be mapped to a 2D rectangle window delimited by (0, 0)-(`w`, `h`).

* When scan-converting the outline, everything that falls within the bitmap
  window is rendered, the rest is ignored.

A common mistake made by many developers when they begin using FreeType is
believing that a loaded outline can be directly rendered in a bitmap of
adequate dimensions.  The following images illustrate why this is a problem.

* The first image shows a loaded outline in the 2D plane.

* The second one shows the target window for a bitmap of arbitrary dimensions
  [`w`, `h`].

* The third one shows the juxtaposition of the outline and window in the 2D
  plane.

* The last image shows what will really be rendered in the bitmap.

![clipping algorithm](clipping.png)

Indeed, in nearly all cases, the loaded or transformed outline must be
translated before it is rendered into a target bitmap, in order to adjust its
position relative to the target window.

For example, the correct way of creating a _standalone_ glyph bitmap is as
follows:

* Compute the size of the glyph bitmap.  It can be computed directly from the
  glyph metrics, or by computing its bounding box (this is useful when a
  transformation has been applied to the outline after loading it, as the glyph
  metrics are not valid anymore).

* Create the bitmap with the computed dimensions.  Don't forget to fill the
  pixel buffer with the background color.

* Translate the outline so that its lower left corner matches (0, 0).  Don't
  forget that in order to preserve hinting, one should use integer, i.e.,
  rounded distances (of course, this isn't required if preserving hinting
  information doesn't matter, like with rotated text).  Usually, this means
  translating with a vector (`-ROUND(xMin)`, `-ROUND(yMin)`).

* Call the rendering function (it can be
  [`FT_Outline_Render`](../reference/ft2-outline_processing.html#FT_Outline_Render),
  for example).

In the case where one wants to write glyph images directly into a large bitmap,
the outlines must be translated so that their vectorial position corresponds to
the current text cursor or character origin.
