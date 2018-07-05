---
layout: page
title: I. Basic Typographic Concepts
theme: dark-green
sidebar: docs_sidebar
permalink: docs/glyphs/glyphs-1.html
nav: glyphs/glyphs-1.html

prev: index.html
next: glyphs-2.html
---

### 1\. Font files, format and information {#section-1}

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

### 2\. Character images and mappings {#section-2}

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

### 3\. Character and font metrics {#section-3}

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
