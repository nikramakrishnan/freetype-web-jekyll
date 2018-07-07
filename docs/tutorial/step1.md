---
layout: page
title: I. Simple Glyph Loading
theme: dark-green
sidebar: docs_sidebar
permalink: docs/tutorial/step1.html
nav: tutorial/step1.html

prev: index.html
next: step2.html
---

## 1. Header Files {#section-1}

The following are instructions required to compile an application that
uses the FreeType 2 library.

1.  Locate the FreeType 2 `include` directory.

    You have to add it to your compilation include path.

    In Unix-like environments you can use the `pkg-config` program to
    retrieve the appropriate compilation flags; say.

          pkg-config --cflags freetype2

    to get the compilation flags.

    This program can also be used to check the version of the library
    that is installed on your system, as well as the required librarian
    and linker flags.

    Another solution is the `freetype-config` script. However, its use
    is deprecated since it can\'t be used reliably for cross
    compilation.

2.  Include the file named `ft2build.h`.

    It contains various macro declarations that are later used to
    `#include` the appropriate public FreeType 2 header files.

3.  Include the main FreeType 2 API header file.

    You should do that using the macro `FT_FREETYPE_H`, like in the
    following example.

    ```c
    #include <ft2build.h>
    #include FT_FREETYPE_H
    ```

    `FT_FREETYPE_H` is a special macro defined in file `ftheader.h`. It
    contains some installation-specific macros to name other public
    header files of the FreeType 2 API.

    You can read [this section of the FreeType 2 API
    Reference](../reference/ft2-header_file_macros) for a complete
    listing of the header macros.

The use of macros in `#include` statements is ANSI-compliant. It is used
for several reasons.

-   It avoids conflicts with (deprecated) FreeType 1.x public header
    files.
-   The macro names are not limited to the DOS 8.3 file naming limit;
    names like `FT_MULTIPLE_MASTERS_H` or `FT_SFNT_NAMES_H` are a lot
    more readable and explanatory than the real file names `ftmm.h` and
    `ftsnames.h`.
-   It allows special installation tricks that will not be discussed
    here.

## 2. Library Initialization {#section-2}

To initialize the FreeType library, create a variable of type
[`FT_Library`](../reference/ft2-base_interface#FT_Library) named,
for example, `library`, and call the function
[`FT_Init_FreeType`](../reference/ft2-base_interface#FT_Init_FreeType).

```c
#include <ft2build.h>
#include FT_FREETYPE_H

FT_Library  library;


...

error = FT_Init_FreeType( &library );
if ( error )
{
... an error occurred during library initialization ...
}
```

This function is in charge of

-   creating a new instance of the FreeType 2 library and setting the
    handle `library` to it, and
-   loading each module that FreeType knows about in the library. Among
    others, your new `library` object is able to handle TrueType,
    Type 1, CID-keyed & OpenType/CFF fonts gracefully.

As you can see, the function returns an error code, like most other
functions of the FreeType API. An error code of 0 (also known as
`FT_Err_Ok`) *always* means that the operation was successful;
otherwise, the value describes the error, and `library` is set to NULL.

A list of all FreeType error codes can be found in file `fterrdef.h`.

## 3. Loading a Font Face {#section-3}

### a. From a Font File

Create a new `face` object by calling
[`FT_New_Face`](../reference/ft2-base_interface#FT_New_Face). A
*face* describes a given typeface and style. For example, 'Times New
Roman Regular' and 'Times New Roman Italic' correspond to two different
faces.

```c
FT_Library  library;   /* handle to library     */
FT_Face     face;      /* handle to face object */


error = FT_Init_FreeType( &library );
if ( error ) { ... }

error = FT_New_Face( library,
                    "/usr/share/fonts/truetype/arial.ttf",
                    0,
                    &face );
if ( error == FT_Err_Unknown_File_Format )
{
... the font file could be opened and read, but it appears
... that its font format is unsupported
}
else if ( error )
{
... another error code means that the font file could not
... be opened or read, or that it is broken...
}
```

As you can certainly imagine, `FT_New_Face` opens a font file, then
tries to extract one face from it. Its parameters are as follows.

library
:   A handle to the FreeType library instance where the face object is
    created.

filepathname
:   The font file pathname (a standard C string).

face\_index

:   Certain font formats allow several font faces to be embedded in a
    single file.

    This index tells which face you want to load. An error is returned
    if its value is too large.

    Index 0 always works, though.

face

:   A *pointer* to the handle that is set to describe the new face
    object.

    It is set to NULL in case of error.

To know how many faces a given font file contains, set `face_index`
to `-1`, then check the value of `face->num_faces`, which indicates how
many faces are embedded in the font file.

### b. From Memory

In the case where you have already loaded the font file into memory, you
can similarly create a new face object for it by calling
[`FT_New_Memory_Face`](../reference/ft2-base_interface#FT_New_Memory_Face).

```c
FT_Library  library;   /* handle to library     */
FT_Face     face;      /* handle to face object */


error = FT_Init_FreeType( &library );
if ( error ) { ... }

error = FT_New_Memory_Face( library,
                            buffer,    /* first byte in memory */
                            size,      /* size in bytes        */
                            0,         /* face_index           */
                            &face );
if ( error ) { ... }
```

As you can see, `FT_New_Memory_Face` takes a pointer to the font file
buffer and its size in bytes instead of a file pathname. Other than
that, it has exactly the same semantics as `FT_New_Face`.

Note that you must not deallocate the font file buffer before calling
[`FT_Done_Face`](../reference/ft2-base_interface#FT_Done_Face).

### c. From Other Sources (Compressed Files, Network, etc.)

There are cases where using a file pathname or preloading the file into
memory is not sufficient. With FreeType 2, it is possible to provide
your own implementation of I/O routines.

This is done through the
[`FT_Open_Face`](../reference/ft2-base_interface#FT_Open_Face)
function, which can be used to open a new font face with a custom input
stream, select a specific driver for opening, or even pass extra
parameters to the font driver when creating the object. We advise you to
look up the [FreeType 2 reference manual](../reference/ft2-toc) in
order to learn how to use it.

## 4. Accessing the Face Data {#section-4}

A *face object* models all information that globally describes the face.
Usually, this data can be accessed directly by dereferencing a handle,
like in `face−>num_glyphs`.

The complete list of available fields is in the
[`FT_FaceRec`](../reference/ft2-base_interface#FT_FaceRec)
structure description. However, we describe here a few of them in more
detail.

num\_glyphs
:   This variable gives the number of *glyphs* available in the font
    face. A glyph is a character image, nothing more -- it thus doesn\'t
    necessarily correspond to a *character code*.

face\_flags
:   A 32-bit integer containing bit flags that describe some face
    properties. For example, the flag `FT_FACE_FLAG_SCALABLE` indicates
    that the face\'s font format is scalable and that glyph images can
    be rendered for all character pixel sizes. For more information on
    face flags, please read the [FreeType 2 API
    Reference](../reference/ft2-base_interface#FT_FACE_FLAG_XXX).

units\_per\_EM
:   This field is only valid for scalable formats (it is set to 0
    otherwise). It indicates the number of font units covered by the EM.

num\_fixed\_sizes
:   This field gives the number of embedded bitmap strikes in the
    current face. A *strike* is a series of glyph images for a given
    character pixel size. For example, a font face could include strikes
    for pixel sizes 10, 12, and 14. Note that even scalable font formats
    can have embedded bitmap strikes!

available\_sizes

:   A pointer to an array of
    [`FT_Bitmap_Size`](../reference/ft2-base_interface#FT_Bitmap_Size)
    elements. Each `FT_Bitmap_Size` indicates the horizontal and
    vertical *character pixel sizes* for each of the strikes that are
    present in the face.

    Note that, generally speaking, these are *not* the *cell size* of
    the bitmap strikes.

## 5. Setting the Current Pixel Size {#section-5}

FreeType 2 uses *size objects* to model all information related to a
given character size for a given face. For example, a size object holds
the value of certain metrics like the ascender or text height, expressed
in 1/64th of a pixel, for a character size of 12 points (however, those
values are rounded to integers, i.e., multiples of 64).

When the `FT_New_Face` function is called (or one of its siblings), it
*automatically* creates a new size object for the returned face. This
size object is directly accessible as `face−>size`.

NOTE: A single face object can deal with one or more size objects at a
time; however, this is something that few programmers really need to do.
We have thus decided to make this feature available through additional
functions.

When a new face object is created, all elements are set to 0 during
initialization. To populate the structure with sensible values, you
should call
[`FT_Set_Char_Size`](../reference/ft2-base_interface#FT_Set_Char_Size).
Here is an example, setting the character size to 16pt for a 300×300dpi
device:

```c
error = FT_Set_Char_Size(
        face,    /* handle to face object           */
        0,       /* char_width in 1/64th of points  */
        16*64,   /* char_height in 1/64th of points */
        300,     /* horizontal device resolution    */
        300 );   /* vertical device resolution      */
```

Some notes.

-   The character widths and heights are specified in 1/64th of points.
    A point is a *physical* distance, equaling 1/72th of an inch.
    Normally, it is not equivalent to a pixel.
-   Value of 0 for the character width means 'same as character height',
    value of 0 for the character height means 'same as character width'.
    Otherwise, it is possible to specify different character widths and
    heights.
-   The horizontal and vertical device resolutions are expressed in
    *dots-per-inch*, or *dpi*. Standard values are 72 or 96 dpi for
    display devices like the screen. The resolution is used to compute
    the character pixel size from the character point size.
-   Value of 0 for the horizontal resolution means 'same as vertical
    resolution', value of 0 for the vertical resolution means 'same as
    horizontal resolution'. If both values are zero, 72 dpi is used for
    both dimensions.
-   The first argument is a handle to a face object, not a size object.

This function computes the (possibly fractional) character pixel size
that corresponds to the character width and height and device
resolutions. A common acronym for the pixel size is *ppem* (pixel per
em).

If you want to specify the (integer) pixel sizes yourself, you can call
[`FT_Set_Pixel_Sizes`](../reference/ft2-base_interface#FT_Set_Pixel_Sizes).

```c
error = FT_Set_Pixel_Sizes(
        face,   /* handle to face object */
        0,      /* pixel_width           */
        16 );   /* pixel_height          */
```

This example sets the character pixel sizes to 16×16 pixels. As
previously, a value of 0 for one of the dimensions means 'same as the
other'.

Note that both functions return an error code. Usually, an error occurs
with a fixed-size font format (like FNT or PCF) when trying to set the
pixel size to a value that is not listed in the `face->fixed_sizes`
array.

Be aware that fractional ppem values are not always supported. For
example, the native bytecode engine for hinting TrueType fonts (TTFs)
only supports integer ppem values, and FreeType rounds fractional ppem
values accordingly.

## 6. Loading a Glyph Image {#section-6}

### a. Converting a Character Code Into a Glyph Index

Normally, an application wants to load a glyph image based on its
*character code*, which is a unique value that defines the character for
a given *encoding*. For example, code 65 (0x41) represents character 'A'
in ASCII encoding.

A face object contains one or more tables, called *charmaps*, to convert
character codes to glyph indices. For example, most older TrueType fonts
contain two charmaps: One is used to convert Unicode character codes to
glyph indices, the other one is used to convert Apple Roman encoding to
glyph indices. Such fonts can then be used either on Windows (which uses
Unicode) and old MacOS versions (which use Apple Roman). Note also that
a given charmap might not map to all the glyphs present in the font.

By default, when a new face object is created, FreeType tries to select
a Unicode charmap. It emulates a Unicode charmap if the font doesn\'t
contain such a charmap, based on glyph names. Note that it is possible
that the emulation misses glyphs if glyph names are non-standard. For
some fonts like symbol fonts, no Unicode emulation is possible at all.

Later on we will describe how to look for specific charmaps in a face.
For now, we assume that the face contains at least a Unicode charmap
that was selected during a call to `FT_New_Face`. To convert a Unicode
character code to a font glyph index, we use
[`FT_Get_Char_Index`](../reference/ft2-base_interface#FT_Get_Char_Index).

```c
glyph_index = FT_Get_Char_Index( face, charcode );
```

This code line looks up the glyph index corresponding to the given
`charcode` in the charmap that is currently selected for the face. You
should use the UTF-32 representation form of Unicode; for example, if
you want to load character U+1F028, use value 0x1F028 as the value for
`charcode`.

If no charmap was selected, the function returns the charcode.

Note that this is one of the rare FreeType functions that do not return
an error code. However, when a given character code has no glyph image
in the face, value 0 is returned. By convention, it always corresponds
to a special glyph image called the *missing glyph*, which is commonly
displayed as a box or a space.

### b. Loading a Glyph From the Face

Once you have a glyph index, you can load the corresponding glyph image.
The latter can be stored in various formats within the font file. For
fixed-size formats like FNT or PCF, each image is a bitmap. Scalable
formats like TrueType or CFF use vectorial shapes (*outlines*) to
describe each glyph. Some formats may have even more exotic ways of
representing glyphs (e.g., MetaFont -- but this format is not
supported). Fortunately, FreeType 2 is flexible enough to support any
kind of glyph format through a simple API.

The glyph image is always stored in a special object called a *glyph
slot*. As its name suggests, a glyph slot is a container that is able to
hold one glyph image at a time, be it a bitmap, an outline, or something
else. Each face object has a single glyph slot object that can be
accessed as `face->glyph`. Its fields are explained by the
[`FT_GlyphSlotRec`](../reference/ft2-base_interface#FT_GlyphSlotRec)
structure documentation.

Loading a glyph image into the slot is performed by calling
[`FT_Load_Glyph`](../reference/ft2-base_interface#FT_Load_Glyph).

```c
error = FT_Load_Glyph(
        face,          /* handle to face object */
        glyph_index,   /* glyph index           */
        load_flags );  /* load flags, see below */
```

The `load_flags` value is a set of bit flags to indicate some special
operations. The default value `FT_LOAD_DEFAULT` is 0.

This function tries to load the corresponding glyph image from the face.

-   If a bitmap is found for the corresponding glyph and pixel size, it
    is loaded into the slot. Embedded bitmaps are always favoured over
    native image formats, because we assume that they are higher-quality
    versions of the same glyph. This can be changed by using the
    `FT_LOAD_NO_BITMAP` flag.
-   Otherwise, a native image for the glyph is loaded. It is also scaled
    to the current pixel size, as well as hinted for certain formats
    like TrueType and Type 1.

The field `face−>glyph−>format` describes the format used for storing
the glyph image in the slot. If it is not `FT_GLYPH_FORMAT_BITMAP`, one
can immediately convert it to a bitmap through
[`FT_Render_Glyph`](../reference/ft2-base_interface#FT_Render_Glyph).

```c
error = FT_Render_Glyph( face->glyph,   /* glyph slot  */
                        render_mode ); /* render mode */
```

The parameter `render_mode` is a set of bit flags to specify how to
render the glyph image. `FT_RENDER_MODE_NORMAL`, the default, renders an
anti-aliased coverage bitmap with 256 gray levels (also called a
*pixmap*), as this is the default. You can alternatively use
`FT_RENDER_MODE_MONO` if you want to generate a 1-bit monochrome bitmap.
More values are available for the
[`FT_Render_Mode`](../reference/ft2-base_interface#FT_Render_Mode)
enumeration value.

Once you have a bitmapped glyph image, you can access it directly
through `glyph->bitmap` (a simple descriptor for bitmaps or pixmaps),
and position it through `glyph->bitmap_left` and `glyph->bitmap_top`.
For optimal rendering on a screen the bitmap should be used as an alpha
channel in linear blending with gamma correction.

Note that `bitmap_left` is the horizontal distance from the current pen
position to the leftmost border of the glyph bitmap, while `bitmap_top`
is the vertical distance from the pen position (on the baseline) to the
topmost border of the glyph bitmap. *It is positive to indicate an
upwards distance*.

### c. Using Other Charmaps

As said before, when a new face object is created, it looks for a
Unicode charmap and select it. The currently selected charmap can be
accessed via `face->charmap`. This field is NULL if no charmap is
selected, which typically happens when you create a new `FT_Face` object
from a font file that doesn\'t contain a Unicode charmap (which is
rather infrequent today).

There are two ways to select a different charmap with FreeType. It\'s
easiest if the encoding you need already has a corresponding enumeration
defined in `FT_FREETYPE_H`, for example `FT_ENCODING_BIG5`. In this
case, you can call
[`FT_Select_Charmap`](../reference/ft2-base_interface#FT_Select_Charmap).

```c
error = FT_Select_Charmap(
        face,               /* target face object */
        FT_ENCODING_BIG5 ); /* encoding           */
```

Another way is to manually parse the list of charmaps for the face; this
is accessible through the fields `num_charmaps` and `charmaps` (notice
the 's') of the face object. As you could expect, the first is the
number of charmaps in the face, while the second is *a table of pointers
to the charmaps* embedded in the face.

Each charmap has a few visible fields to describe it more precisely. The
most important ones are `charmap->platform_id` and
`charmap->encoding_id`, defining a pair of values that describe the
charmap in a rather generic way: Each value pair corresponds to a given
encoding. For example, the pair (3,1) corresponds to Unicode. The list
is defined in the OpenType specification; you can also use the file
`FT_TRUETYPE_IDS_H`, which defines several helpful constants to deal
with them. Note that we use the OpenType enumeration values for
non-OpenType fonts also (by defining additional constants where
necessary).

To select a specific encoding, you need to find a corresponding value
pair in the specification, then look for it in the charmaps list.

```c
FT_CharMap  found = 0;
FT_CharMap  charmap;
int         n;


for ( n = 0; n < face->num_charmaps; n++ )
{
charmap = face->charmaps[n];
if ( charmap->platform_id == my_platform_id &&
        charmap->encoding_id == my_encoding_id )
{
    found = charmap;
    break;
}
}

if ( !found ) { ... }

/* now, select the charmap for the face object */
error = FT_Set_Charmap( face, found );
if ( error ) { ... }
```

Once a charmap has been selected, either through `FT_Select_Charmap` or
[`FT_Set_Charmap`](../reference/ft2-base_interface#FT_Set_Charmap),
it is used by all subsequent calls to `FT_Get_Char_Index`.

### d. Glyph Transformations

It is possible to specify an affine transformation with
[`FT_Set_Transform`](../reference/ft2-base_interface#FT_Set_Transform),
to be applied to glyph images when they are loaded. Of course, this only
works for scalable (vectorial) font formats.

```c
error = FT_Set_Transform(
        face,       /* target face object    */
        &matrix,    /* pointer to 2x2 matrix */
        &delta );   /* pointer to 2d vector  */
```

This function sets the current transformation for a given face object.
Its second parameter is a pointer to an
[`FT_Matrix`](../reference/ft2-basic_types#FT_Matrix) structure
that describes a 2×2 affine matrix. The third parameter is a pointer to
an [`FT_Vector`](../reference/ft2-basic_types#FT_Vector) structure,
describing a two-dimensional vector that translates the glyph image
*after* the 2×2 transformation.

Note that the matrix pointer can be set to NULL, in which case the
identity transformation is used. Coefficients of the matrix are
otherwise in 16.16 fixed-point units.

The vector pointer can also be set to NULL (in which case a delta of
(0,0) is used). The vector coordinates are expressed in 1/64th of a
pixel (also known as 26.6 fixed-point numbers).

The transformation is applied to every glyph that is loaded through
`FT_Load_Glyph` and is *completely independent of any hinting process*.
This means that you won\'t get the same results if you load a glyph at
the size of 24 pixels, or a glyph at the size of 12 pixels scaled by 2
through a transformation, because the hints are computed differently
(except if you have disabled hints).

If you ever need to use a non-orthogonal transformation with optimal
hints, you first have to decompose your transformation into a scaling
part and a rotation/shearing part. Use the scaling part to compute a new
character pixel size, then the other one to call `FT_Set_Transform`.
This is explained in more detail in part II of this tutorial.

Rotation usually disables hinting.

Loading a glyph bitmap with a non-identity transformation works; the
transformation is ignored in this case.

## 7. Simple Text Rendering {#section-7}

We now present a simple example to render a string of 8-bit Latin-1
text, assuming a face that contains a Unicode charmap.

The idea is to create a loop that loads one glyph image on each
iteration, converts it to a pixmap, draws it on the target surface, then
increments the current pen position.

### a. Basic Code {#basic-code}

The following code performs our simple text rendering with the functions
previously described.

```c
FT_GlyphSlot  slot = face->glyph;  /* a small shortcut */
int           pen_x, pen_y, n;


... initialize library ...
... create face object ...
... set character size ...

pen_x = 300;
pen_y = 200;

for ( n = 0; n < num_chars; n++ )
{
FT_UInt  glyph_index;


/* retrieve glyph index from character code */
glyph_index = FT_Get_Char_Index( face, text[n] );

/* load glyph image into the slot (erase previous one) */
error = FT_Load_Glyph( face, glyph_index, FT_LOAD_DEFAULT );
if ( error )
    continue;  /* ignore errors */

/* convert to an anti-aliased bitmap */
error = FT_Render_Glyph( face->glyph, FT_RENDER_MODE_NORMAL );
if ( error )
    continue;

/* now, draw to our target surface */
my_draw_bitmap( &slot->bitmap,
                pen_x + slot->bitmap_left,
                pen_y - slot->bitmap_top );

/* increment pen position */
pen_x += slot->advance.x >> 6;
pen_y += slot->advance.y >> 6; /* not useful for now */
}
```

This code needs a few explanations.

-   We define a handle named `slot` that points to the face object\'s
    glyph slot. (The type
    [`FT_GlyphSlot`](../reference/ft2-base_interface#FT_GlyphSlot)
    is a pointer). That is a convenience to avoid using
    `face->glyph->XXX` every time.
-   We increment the pen position with the vector `slot->advance`, which
    correspond to the glyph\'s *advance width* (also known as its
    *escapement*). The advance vector is expressed in 1/64th of pixels,
    and is truncated to integer pixels on each iteration.
-   The function `my_draw_bitmap` is not part of FreeType but must be
    provided by the application to draw the bitmap to the target
    surface. In this example, it takes a pointer to an
    [`FT_Bitmap`](../reference/ft2-basic_types#FT_Bitmap)
    descriptor and the position of its top-left corner as arguments. For
    ideal rendering on a screen this function should perform linear
    blending with gamma correction, using the bitmap as an alpha
    channel.
-   The value of `slot->bitmap_top` is positive for an *upwards*
    vertical distance. Assuming that the coordinates taken by
    `my_draw_bitmap` use the opposite convention (increasing Y
    corresponds to downwards scanlines), we subtract it from `pen_y`,
    instead of adding to it.

### b.Refined code

The following code is a refined version of the example above. It uses
features and functions of FreeType that have not yet been introduced,
and which are explained below.

```c
FT_GlyphSlot  slot = face->glyph;  /* a small shortcut */
FT_UInt       glyph_index;
int           pen_x, pen_y, n;


... initialize library ...
... create face object ...
... set character size ...

pen_x = 300;
pen_y = 200;

for ( n = 0; n < num_chars; n++ )
{
/* load glyph image into the slot (erase previous one) */
error = FT_Load_Char( face, text[n], FT_LOAD_RENDER );
if ( error )
    continue;  /* ignore errors */

/* now, draw to our target surface */
my_draw_bitmap( &slot->bitmap,
                pen_x + slot->bitmap_left,
                pen_y - slot->bitmap_top );

/* increment pen position */
pen_x += slot->advance.x >> 6;
}
```

We have reduced the size of our code, but it does exactly the same
thing.

-   We use the function
    [`FT_Load_Char`](../reference/ft2-base_interface#FT_Load_Char)
    instead of `FT_Load_Glyph`. As you probably imagine, it is
    equivalent to calling `FT_Get_Char_Index`, then `FT_Load_Glyph`.
-   We do not use `FT_LOAD_DEFAULT` for the loading mode, but the bit
    flag `FT_LOAD_RENDER`. It indicates that the glyph image must be
    immediately converted to an anti-aliased bitmap. This is of course a
    shortcut that avoids calling `FT_Render_Glyph` explicitly but is
    strictly equivalent.

    Note that you can also specify that you want a monochrome bitmap
    instead by using the additional `FT_LOAD_MONOCHROME` load flag.

### c. More Advanced Rendering {#transformed-text}

Let us try to render transformed text now (for example through a
rotation). We can do this using `FT_Set_Transform`.

```c
FT_GlyphSlot  slot;
FT_Matrix     matrix;              /* transformation matrix */
FT_UInt       glyph_index;
FT_Vector     pen;                 /* untransformed origin */
int           n;


... initialize library ...
... create face object ...
... set character size ...

slot = face->glyph;                /* a small shortcut */

/* set up matrix */
matrix.xx = (FT_Fixed)( cos( angle ) * 0x10000L );
matrix.xy = (FT_Fixed)(-sin( angle ) * 0x10000L );
matrix.yx = (FT_Fixed)( sin( angle ) * 0x10000L );
matrix.yy = (FT_Fixed)( cos( angle ) * 0x10000L );

/* the pen position in 26.6 cartesian space coordinates */
/* start at (300,200)                                   */
pen.x = 300 * 64;
pen.y = ( my_target_height - 200 ) * 64;

for ( n = 0; n < num_chars; n++ )
{
/* set transformation */
FT_Set_Transform( face, &matrix, &pen );

/* load glyph image into the slot (erase previous one) */
error = FT_Load_Char( face, text[n], FT_LOAD_RENDER );
if ( error )
    continue;  /* ignore errors */

/* now, draw to our target surface (convert position) */
my_draw_bitmap( &slot->bitmap,
                slot->bitmap_left,
                my_target_height - slot->bitmap_top );

/* increment pen position */
pen.x += slot->advance.x;
pen.y += slot->advance.y;
}
```

Some remarks.

-   We now use a vector of type `FT_Vector` to store the pen position,
    with coordinates expressed as 1/64th of pixels, hence a
    multiplication. The position is expressed in cartesian space.
-   Glyph images are always loaded, transformed, and described in the
    cartesian coordinate system within FreeType (which means that
    increasing Y corresponds to upper scanlines), unlike the system
    typically used for bitmaps (where the topmost scanline has
    coordinate 0). We must thus convert between the two systems when we
    define the pen position, and when we compute the topleft position of
    the bitmap.
-   We set the transformation on each glyph to indicate the rotation
    matrix as well as a delta that moves the transformed image to the
    current pen position (in cartesian space, not bitmap space). As a
    consequence, the values of `bitmap_left` and `bitmap_top` correspond
    to the bitmap origin in target space pixels. We thus don\'t add
    `pen.x` or `pen.y` to their values when calling `my_draw_bitmap`.
-   The advance width is always returned transformed, which is why it
    can be directly added to the current pen position. Note that it is
    *not* rounded this time.

A complete source code example can be found [here](/assets/example1.c).

It is important to note that, while this example is a bit more complex
than the previous one, it is strictly equivalent for the case where the
transformation is the identity. Hence it can be used as a replacement
(but a more powerful one).

The still present few shortcomings will be explained, and solved, in the
next part of this tutorial.
