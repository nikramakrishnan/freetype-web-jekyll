---
layout: page
title: V. Text Processing 
theme: dark-green
sidebar: docs_sidebar
permalink: docs/glyphs/glyphs-5.html
nav: glyphs/glyphs-5.html

prev: glyphs-4.html
next: glyphs-6.html
---

This section demonstrate algorithms which use the concepts previously defined
to render text, whatever layout you use.  It assumes _simple_ text handling
suitable for scripts like Latin or Cyrillic, using a one-to-one relationship
between input character codes and output glyphs indices.  Scripts like Arabic
or Khmer, which need a  'shaping engine' to do the character code to glyph
index conversion, are beyond the scope (and should be done by proper layout
engines like [Pango](http://www.pango.org/) anyway).

### 1\. Writing simple text strings {#section-1}

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

### 2\. Pseudo-subpixel positioning {#section-2}

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

### 3\. Simple kerning {#section-3}

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

### 4\. Right-to-left layout {#section-4}

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

### 5\. Vertical layouts {#section-5}

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
