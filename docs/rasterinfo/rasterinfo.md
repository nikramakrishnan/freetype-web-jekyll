---
layout: page
title: The RasterInfo Font
theme: dark-green
sidebar: docs_sidebar
permalink: docs/rasterinfo/rasterinfo.html
nav: rasterinfo/rasterinfo.html
---

<style type="text/css">
  @font-face {
    font-family: "RasterInfo";
    src: url("RasterInfo.ttf");
    font-weight: normal;
    font-style: normal;
    text-rendering: optimizeLegibility; }	

  span.RasterInfo {
    font-family: "RasterInfo"; }
  span.RasterInfo-large {
    font-family: "RasterInfo";
    font-size: 96px;
    line-height: 100%; }

  div.zebra {
    float: right;
    text-align:center;
    line-height: 300%; }
</style>

[`RasterInfo.ttf`](RasterInfo.ttf) is a means to show various properties
of the TrueType bytecode interpreter used to hint the TrueType fonts on
the current HTML page. If you only see one or more letters 'E' instead
of digits, the TrueType bytecode interpreter is not active.

At the same time, it is a showcase to demonstrate how to change glyph
shapes with TrueType instructions.

Now the characters contained in the font. The value in parentheses gives
the character to be used in the RasterInfo font. For more details on the
various parameters, please consult the source code file for the font, to
be found on
[github](https://github.com/lemzwerg/RasterInfo/blob/master/RasterInfo.m4).
The special zebra glyphs ('\#\$') are designed to test gamma correction
and LCD filtering; they should not, ideally, have any moiré or color
fringes.

<div class="zebra"><span class="RasterInfo-large">#$</span><br>
  #$</div>

<div class="quote">
  <p>The current PPEM value used in this paragraph
  (&lsquo;0&rsquo;): <span class="RasterInfo">0</span></p>

  <p>The version of the TrueType bytecode interpreter
  (&lsquo;1&rsquo;): <span class="RasterInfo">1</span></p>

  <p>If rotation is on, digit&nbsp;1 is shown
  (&lsquo;2&rsquo;): <span class="RasterInfo">2</span></p>

  <p>If stretching is on, digit&nbsp;1 is shown
  (&lsquo;3&rsquo;): <span class="RasterInfo">3</span></p>

  <p>If grayscale rendering is on, digit&nbsp;1 is shown
  (&lsquo;4&rsquo;): <span class="RasterInfo">4</span></p>

  <p>If ClearType is enabled, digit&nbsp;1 is shown
  (&lsquo;5&rsquo;): <span class="RasterInfo">5</span></p>

  <p>If ClearType's compatible width mode is on, digit&nbsp;1
  is shown
  (&lsquo;6&rsquo;): <span class="RasterInfo">6</span></p>

  <p>If ClearType uses vertical LCD subpixels (this is, a
  subpixel's longer side is horizontally positioned),
  digit&nbsp;1 is shown
  (&lsquo;7&rsquo;): <span class="RasterInfo">7</span></p>

  <p>If ClearType uses BGR (blue-green-red) rendering instead
  of RGB (red-green-blue), digit&nbsp;1 is shown
  (&lsquo;8&rsquo;): <span class="RasterInfo">8</span></p>

  <p>If ClearType uses subpixel positioning, digit&nbsp;1 is
  shown
  (&lsquo;9&rsquo;): <span class="RasterInfo">9</span></p>

  <p>If ClearType symmetrical smoothing is active,
  digit&nbsp;1 is shown
  (&lsquo;:&rsquo;): <span class="RasterInfo">:</span></p>

  <p>If Gray ClearType is active, digit&nbsp;1 is shown
  (&lsquo;;&rsquo;): <span class="RasterInfo">;</span></p>
</div>

As an example, the image below shows the values as displayed by the Edge
browser (version 40) on Windows 10.

![RasterInfo display on Windows 10 using Edge
40](assets/RasterInfo_Windows_10_Edge_40.png)

On this page, the font is set up with an `@font-face` CSS instruction,
to be used within a `span`.

```html
<head>
...
<style type="text/css">
@font-face {
font-family: "RasterInfo";
src: url("RasterInfo.ttf");
font-weight: normal;
font-style: normal;
text-rendering: optimizeLegibility; }
...
span.RasterInfo { font-family: "RasterInfo"; }
</style>
</head>

<body>
...
<span class="RasterInfo">1</span>
...
</body>
```

[`RasterInfo.ttf`](RasterInfo.ttf) was written by Werner Lemberg; the
font is licensed under the [SIL Open Font
License](http://scripts.sil.org/OFL). The current version is 1.02.
