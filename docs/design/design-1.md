---
layout: page
title: Introduction
theme: dark-green
sidebar: docs_sidebar
permalink: docs/design/design-1.html
nav: design/design-1.html

prev: index.html
next: design-2.html
---

This document provides details on the design and implementation of the
FreeType 2 library. Its goal is to help developers better understand how
FreeType 2 is organized, in order to let them extend, customize, and
debug it.

Before anything else, it is important to understand the *purpose* of
this library, i.e., why it has been written.

-   It allows client applications to *access font files easily*,
    wherever they could be stored, and as independently of the font
    format as possible.

-   Easy *retrieval of global font data* most commonly found in normal
    font formats (i.e., global metrics, encoding/charmaps, etc.).

-   It allows easy *retrieval of individual glyph data* (metrics,
    images, name, anything else).

-   *Access to font format-specific 'features'* whenever possible (e.g.,
    SFNT tables, Multiple Masters, OpenType layout tables, font
    variations, etc.).

Its design has also severely been influenced by the following
requirements.

-   *High portability*. The library must be able to run on any kind of
    environment. This requirement introduces a few drastic choices that
    are part of FreeType 2\'s low-level system interface.

-   *Extendability*. New features should be added with the least
    modifications in the library\'s code base. This requirement induces
    an extremely simple design where nearly all operations are provided
    by modules or services.

-   *Customization*. It should be easy to build a version of the library
    that only contains the features needed by a specific project. This
    really is important when you need to integrate it into a font server
    for embedded graphics libraries, say.

-   *Compactness* and *efficiency*. The primary target for this library
    originally were embedded systems with low CPU and memory resources.
    Today, however, memory constraints are much less strict, and the
    focus of development shifted to support as much font features as
    possible.

The rest of this document is divided in several sections. First, a few
chapters will present the library\'s basic design as well as the objects
and data managed internally by FreeType 2.

It is intended to eventually add sections that cover library
customization, relating to topics as system-specific interfaces, how to
write your own modules or services and how to tailor library
initialization and compilation to your needs. Those sections are not
written yet, however.
