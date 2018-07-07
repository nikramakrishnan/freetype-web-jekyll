---
layout: page
title: III. Examples
theme: dark-green
sidebar: docs_sidebar
permalink: docs/tutorial/step3.html
nav: tutorial/step3.html

prev: step2.html
---

For completeness, here again a link to the [example](example1.c) used
and explained in the [first part of the tutorial](step1.html).

[Erik Möller](mailto:erik@timetrap.se) contributed a very nice C++
example that shows renderer callbacks in action to draw a coloured glyph
with a differently coloured outline. The source code can be found
[here](assets/example2.cpp).

[Another example](assets/example3.cpp) demonstrates how to use FreeType\'s
stand-alone B/W rasterizer, `ftraster.c`. You need files from FreeType
version 2.3.10 or newer.

[Róbert Márki](mailto:gsmiko@gmail.com) contributed a small [Qt
demonstration program](assets/example4.cpp) (together with its [qmake
file](assets/example4.pro)) that shows both direct rendering with a callback
and rendering with a buffer, yielding the same result. You need FreeType
2.4.3 or newer.

[Here](assets/example5.cpp) is some simple C++ code (contributed by [Static
Jobs LLC](https://www.staticjobs.com)) that uses
[`FT_Outline_Decompose`](../reference/ft2-outline_processing#FT_Outline_Decompose)
to convert a glyph outline to the SVG format. As an example, here is the
[resulting file](assets/example5.svg) of the call

```
example5 LiberationSerif-Bold.ttf @
```

(you can find the Liberation font family
[here](https://fedorahosted.org/liberation-fonts/)).
