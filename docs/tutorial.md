---
layout: page
title: FreeType 2 Tutorial
theme: dark-green
sidebar: docs_sidebar
permalink: docs/tutorial.html
nav: tutorial
---

## I. Simple Glyph Loading

### 1\. Header Files

The following are instructions required to compile an application that uses the
FreeType 2 library.

1. Locate the FreeType 2 `include` directory.

   You have to add it to your compilation include path.

   In Unix-like environments you can run the `freetype-config` script with the
   `--cflags` option to retrieve the appropriate compilation flags.  This
   script can also be used to check the version of the library that is
   installed on your system, as well as the required librarian and linker
   flags.

2. Include the file named `ft2build.h`.

   It contains various macro declarations that are later used to `#include` the
   appropriate public FreeType 2 header files.

3. Include the main FreeType 2 API header file.

   You should do that using the macro `FT_FREETYPE_H`, like in the following
   example.

       #include <ft2build.h>
       #include FT_FREETYPE_H

   `FT_FREETYPE_H` is a special macro defined in file `ftheader.h`.  It
   contains some installation-specific macros to name other public header files
   of the FreeType 2 API.

   You can read [this section of the FreeType 2 API
   Reference](../reference/ft2-header_file_macros) for a complete listing
   of the header macros.

The use of macros in `#include` statements is ANSI-compliant.  It is used for
several reasons.

* It avoids conflicts with (deprecated) FreeType 1.x public header files.
* The macro names are not limited to the DOS 8.3 file naming limit; names like
  `FT_MULTIPLE_MASTERS_H` or `FT_SFNT_NAMES_H` are a lot more readable and
  explanatory than the real file names `ftmm.h` and `ftsnames.h`.
* It allows special installation tricks that will not be discussed here.

### 2\. Library Initialization

To initialize the FreeType library, create a variable of type
[`FT_Library`](../reference/ft2-base_interface#FT_Library) named, for
example, `library`, and call the function
[`FT_Init_FreeType`](../reference/ft2-base_interface#FT_Init_FreeType).

    #include <ft2build.h>
    #include FT_FREETYPE_H

    FT_Library  library;

    ...

    error = FT_Init_FreeType( &library );
    if ( error )
    {
        ... an error occurred during library initialization ...
    }

This function is in charge of

* creating a new instance of the FreeType 2 library and setting the handle
  `library` to it, and
* loading each module that FreeType knows about in the library.  Among others,
  your new `library` object is able to handle TrueType, Type 1, CID-keyed &
  OpenType/CFF fonts gracefully.

As you can see, the function returns an error code, like most other functions
of the FreeType API.  An error code of 0 (also known as `FT_Err_Ok`) _always_
means that the operation was successful; otherwise, the value describes the
error, and `library` is set to NULL.

### 3\. Loading a Font Face

#### a. From a Font File

Create a new `face` object by calling
[`FT_New_Face`](../reference/ft2-base_interface#FT_New_Face).  A _face_
describes a given typeface and style.  For example, 'Times New Roman Regular'
and 'Times New Roman Italic' correspond to two different faces.

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

As you can certainly imagine, `FT_New_Face` opens a font file, then tries to
extract one face from it.  Its parameters are as follows.

`library`      | A handle to the FreeType library instance where the face object is created.
`filepathname` | The font file pathname (a standard C string).
`face_index`   | Certain font formats allow several font faces to be embedded in a single file. This index tells which face you want to load.  An error is returned if its value is too large. Index 0 always works, though.
`face`         | A _pointer_ to the handle that is set to describe the new face object. It is set to NULL in case of error.

To know how many faces a given font file contains, load its first face (this
is, `face_index` should be set to zero), then check the value of
`face->num_faces`, which indicates how many faces are embedded in the font
file.

#### b. From Memory

In the case where you have already loaded the font file into memory, you can
similarly create a new face object for it by calling
[`FT_New_Memory_Face`](../reference/ft2-base_interface#FT_New_Memory_Face).

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

As you can see, `FT_New_Memory_Face` takes a pointer to the font file buffer
and its size in bytes instead of a file pathname.  Other than that, it has
exactly the same semantics as `FT_New_Face`.

Note that you must not deallocate the memory before calling
[`FT_Done_Face`](../reference/ft2-base_interface#FT_Done_Face).

#### c. From Other Sources (Compressed Files, Network, etc.)

There are cases where using a file pathname or preloading the file into memory
is not sufficient.  With FreeType 2, it is possible to provide your own
implementation of I/O routines.

This is done through the
[`FT_Open_Face`](../reference/ft2-base_interface#FT_Open_Face) function,
which can be used to open a new font face with a custom input stream, select a
specific driver for opening, or even pass extra parameters to the font driver
when creating the object.  We advise you to look up the [FreeType 2 reference
manual](../reference/ft2-toc) in order to learn how to use it.

### 4\. Accessing the Face Data

A _face object_ models all information that globally describes the face.
Usually, this data can be accessed directly by dereferencing a handle, like in
`face−>num_glyphs`.

The complete list of available fields is in the
[`FT_FaceRec`](../reference/ft2-base_interface#FT_FaceRec) structure
description.  However, we describe here a few of them in more detail.

`num_glyphs` | This variable gives the number of _glyphs_ available in the font face. A glyph is a character image, nothing more – it thus doesn't necessarily correspond to a _character code_.
`face_flags` | A 32-bit integer containing bit flags that describe some face properties. For example, the flag `FT_FACE_FLAG_SCALABLE` indicates that the face's font format is scalable and that glyph images can be rendered for all character pixel sizes. For more information on face flags, please read the [FreeType 2 API Reference](../reference/ft2-base_interface#FT_FACE_FLAG_XXX).
`units_per_EM` | This field is only valid for scalable formats (it is set to 0 otherwise). It indicates the number of font units covered by the EM.
`num_fixed_sizes` | This field gives the number of embedded bitmap strikes in the current face. A _strike_ is a series of glyph images for a given character pixel size. For example, a font face could include strikes for pixel sizes 10, 12, and 14\. Note that even scalable font formats can have embedded bitmap strikes!
`available_sizes` | A pointer to an array of [`FT_Bitmap_Size`](../reference/ft2-base_interface#FT_Bitmap_Size) elements.  Each `FT_Bitmap_Size` indicates the horizontal and vertical _character pixel sizes_ for each of the strikes that are present in the face.

Note that, generally speaking, these are _not_ the _cell size_ of the bitmap
strikes.

### 5\. Setting the Current Pixel Size

FreeType 2 uses _size objects_ to model all information related to a given
character size for a given face.  For example, a size object holds the value of
certain metrics like the ascender or text height, expressed in 1/64th of a
pixel, for a character size of 12 points.

When the `FT_New_Face` function is called (or one of its siblings), it
_automatically_ creates a new size object for the returned face.  This size
object is directly accessible as `face−>size`.

NOTE: A single face object can deal with one or more size objects at a time;
however, this is something that few programmers really need to do.  We have
thus decided to simplify the API for the most common use (i.e., one size per
face) while keeping this feature available through additional functions.

When a new face object is created, all elements are set to 0 during
initialization.  To populate the structure with sensible values, you should
call
[`FT_Set_Char_Size`](../reference/ft2-base_interface#FT_Set_Char_Size).
Here is an example, setting the character size to 16pt for a 300×300dpi device:

    error = FT_Set_Char_Size( face,    /* handle to face object           */
                              0,       /* char_width in 1/64th of points  */
                              16*64,   /* char_height in 1/64th of points */
                              300,     /* horizontal device resolution    */
                              300 );   /* vertical device resolution      */

Some notes.

* The character widths and heights are specified in 1/64th of points.  A point
  is a _physical_ distance, equaling 1/72th of an inch.  Normally, it is not
  equivalent to a pixel.
* Value of 0 for the character width means 'same as character height', value of
  0 for the character height means 'same as character width'.  Otherwise, it is
  possible to specify different character widths and heights.
* The horizontal and vertical device resolutions are expressed in
  _dots-per-inch_, or _dpi_.  Standard values are 72 or 96 dpi for display
  devices like the screen.  The resolution is used to compute the character
  pixel size from the character point size.
* Value of 0 for the horizontal resolution means 'same as vertical resolution',
  value of 0 for the vertical resolution means 'same as horizontal resolution'.
  If both values are zero, 72 dpi is used for both dimensions.
* The first argument is a handle to a face object, not a size object.

This function computes the character pixel size that corresponds to the
character width and height and device resolutions.  However, if you want to
specify the pixel sizes yourself, you can call
[`FT_Set_Pixel_Sizes`](../reference/ft2-base_interface#FT_Set_Pixel_Sizes).

    error = FT_Set_Pixel_Sizes( face,   /* handle to face object */
                                0,      /* pixel_width           */
                                16 );   /* pixel_height          */

This example sets the character pixel sizes to 16×16 pixels.  As previously, a
value of 0 for one of the dimensions means 'same as the other'.

Note that both functions return an error code.  Usually, an error occurs with a
fixed-size font format (like FNT or PCF) when trying to set the pixel size to a
value that is not listed in the `face->fixed_sizes` array.

### 6\. Loading a Glyph Image

#### a. Converting a Character Code Into a Glyph Index

Normally, an application wants to load a glyph image based on its _character
code_, which is a unique value that defines the character for a given
_encoding_.  For example, code 65 (0x41) represents character 'A' in ASCII
encoding.

A face object contains one or more tables, called _charmaps_, to convert
character codes to glyph indices.  For example, most older TrueType fonts
contain two charmaps: One is used to convert Unicode character codes to glyph
indices, the other one is used to convert Apple Roman encoding to glyph
indices.  Such fonts can then be used either on Windows (which uses Unicode)
and old MacOS versions (which use Apple Roman).  Note also that a given charmap
might not map to all the glyphs present in the font.

By default, when a new face object is created, it selects a Unicode charmap.
FreeType tries to emulate a Unicode charmap if the font doesn't contain such a
charmap, based on glyph names.  Note that it is possible that the emulation
misses glyphs if glyph names are non-standard.  For some fonts like symbol
fonts, no Unicode emulation is possible at all.

Later on we will describe how to look for specific charmaps in a face.  For
now, we assume that the face contains at least a Unicode charmap that was
selected during a call to `FT_New_Face`.  To convert a Unicode character code
to a font glyph index, we use
[`FT_Get_Char_Index`](../reference/ft2-base_interface#FT_Get_Char_Index).

    glyph_index = FT_Get_Char_Index( face, charcode );

This code line looks up the glyph index corresponding to the given `charcode`
in the charmap that is currently selected for the face.  You should use the
UTF-32 representation form of Unicode; for example, if you want to load
character U+1F028, use value 0x1F028 as the value for `charcode`.

If no charmap was selected, the function returns the charcode.

Note that this is one of the rare FreeType functions that do not return an
error code.  However, when a given character code has no glyph image in the
face, value 0 is returned.  By convention, it always corresponds to a special
glyph image called the _missing glyph_, which is commonly displayed as a box or
a space.

#### b. Loading a Glyph From the Face

Once you have a glyph index, you can load the corresponding glyph image.  The
latter can be stored in various formats within the font file.  For fixed-size
formats like FNT or PCF, each image is a bitmap.  Scalable formats like
TrueType or CFF use vectorial shapes (_outlines_) to describe each glyph.  Some
formats may have even more exotic ways of representing glyphs (e.g., MetaFont –
but this format is not supported).  Fortunately, FreeType 2 is flexible enough
to support any kind of glyph format through a simple API.

The glyph image is always stored in a special object called a _glyph slot_.  As
its name suggests, a glyph slot is a container that is able to hold one glyph
image at a time, be it a bitmap, an outline, or something else.  Each face
object has a single glyph slot object that can be accessed as `face->glyph`.
Its fields are explained by the
[`FT_GlyphSlotRec`](../reference/ft2-base_interface#FT_GlyphSlotRec)
structure documentation.

Loading a glyph image into the slot is performed by calling
[`FT_Load_Glyph`](../reference/ft2-base_interface#FT_Load_Glyph).

    error = FT_Load_Glyph( face,          /* handle to face object */
                           glyph_index,   /* glyph index           */
                           load_flags );  /* load flags, see below */

The `load_flags` value is a set of bit flags to indicate some special
operations.  The default value `FT_LOAD_DEFAULT` is 0.

This function tries to load the corresponding glyph image from the face.

* If a bitmap is found for the corresponding glyph and pixel size, it is loaded
  into the slot.  Embedded bitmaps are always favoured over native image
  formats, because we assume that they are higher-quality versions of the same
  glyph.  This can be changed by using the `FT_LOAD_NO_BITMAP` flag.
* Otherwise, a native image for the glyph is loaded.  It is also scaled to the
  current pixel size, as well as hinted for certain formats like TrueType and
  Type 1.

The field `face−>glyph−>format` describes the format used for storing the glyph
image in the slot.  If it is not `FT_GLYPH_FORMAT_BITMAP`, one can immediately
convert it to a bitmap through
[`FT_Render_Glyph`](../reference/ft2-base_interface#FT_Render_Glyph).

    error = FT_Render_Glyph( face->glyph,   /* glyph slot  */
                             render_mode ); /* render mode */

The parameter `render_mode` is a set of bit flags to specify how to render the
glyph image.  `FT_RENDER_MODE_NORMAL`, the default, renders an anti-aliased
coverage bitmap with 256 gray levels (also called a _pixmap_), as this is the
default.  You can alternatively use `FT_RENDER_MODE_MONO` if you want to
generate a 1-bit monochrome bitmap.  More values are available for the
[`FT_Render_Mode`](../reference/ft2-base_interface#FT_Render_Mode)
enumeration value.

Once you have a bitmapped glyph image, you can access it directly through
`glyph->bitmap` (a simple descriptor for bitmaps or pixmaps), and position it
through `glyph->bitmap_left` and `glyph->bitmap_top`.  For optimal rendering on
a screen the bitmap should be used as an alpha channel in linear blending with
gamma correction.

Note that `bitmap_left` is the horizontal distance from the current pen
position to the leftmost border of the glyph bitmap, while `bitmap_top` is the
vertical distance from the pen position (on the baseline) to the topmost border
of the glyph bitmap.  _It is positive to indicate an upwards distance_.

#### c. Using Other Charmaps

As said before, when a new face object is created, it looks for a Unicode
charmap and select it.  The currently selected charmap can be accessed via
`face->charmap`.  This field is NULL if no charmap is selected, which typically
happens when you create a new `FT_Face` object from a font file that doesn't
contain a Unicode charmap (which is rather infrequent today).

There are two ways to select a different charmap with FreeType.  It's easiest
if the encoding you need already has a corresponding enumeration defined in
  `FT_FREETYPE_H`, for example `FT_ENCODING_BIG5`.  In this case, you can call
  [`FT_Select_Charmap`](../reference/ft2-base_interface#FT_Select_Charmap).

    error = FT_Select_Charmap( face,               /* target face object */
                               FT_ENCODING_BIG5 ); /* encoding           */

Another way is to manually parse the list of charmaps for the face; this is
accessible through the fields `num_charmaps` and `charmaps` (notice the 's') of
the face object.  As you could expect, the first is the number of charmaps in
the face, while the second is _a table of pointers to the charmaps_ embedded in
the face.

Each charmap has a few visible fields to describe it more precisely.  The most
important ones are `charmap->platform_id` and `charmap->encoding_id`, defining
a pair of values that describe the charmap in a rather generic way: Each value
pair corresponds to a given encoding.  For example, the pair (3,1) corresponds
to Unicode.  The list is defined in the TrueType specification; you can also
use the file `FT_TRUETYPE_IDS_H`, which defines several helpful constants to
deal with them.

To select a specific encoding, you need to find a corresponding value pair in
the specification, then look for it in the charmaps list.  Don't forget that
there are encodings that correspond to several value pairs due to historical
reasons.

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

Once a charmap has been selected, either through `FT_Select_Charmap` or
[`FT_Set_Charmap`](../reference/ft2-base_interface#FT_Set_Charmap), it is
used by all subsequent calls to `FT_Get_Char_Index`.

#### d. Glyph Transformations

It is possible to specify an affine transformation with
[`FT_Set_Transform`](../reference/ft2-base_interface#FT_Set_Transform), to
be applied to glyph images when they are loaded.  Of course, this only works
for scalable (vectorial) font formats.

    error = FT_Set_Transform( face,       /* target face object    */
                              &matrix,    /* pointer to 2x2 matrix */
                              &delta );   /* pointer to 2d vector  */

This function sets the current transformation for a given face object.  Its
second parameter is a pointer to an
[`FT_Matrix`](../reference/ft2-basic_types#FT_Matrix) structure that
describes a 2×2 affine matrix.  The third parameter is a pointer to an
[`FT_Vector`](../reference/ft2-basic_types#FT_Vector) structure,
describing a two-dimensional vector that translates the glyph image _after_ the
2×2 transformation.

Note that the matrix pointer can be set to NULL, in which case the identity
transformation is used.  Coefficients of the matrix are otherwise in 16.16
fixed-point units.

The vector pointer can also be set to NULL (in which case a delta of (0,0) is
used).  The vector coordinates are expressed in 1/64th of a pixel (also known
as 26.6 fixed-point numbers).

The transformation is applied to every glyph that is loaded through
`FT_Load_Glyph` and is _completely independent of any hinting process_.  This
means that you won't get the same results if you load a glyph at the size of 24
pixels, or a glyph at the size of 12 pixels scaled by 2 through a
transformation, because the hints are computed differently (except if you have
disabled hints).

If you ever need to use a non-orthogonal transformation with optimal hints, you
first have to decompose your transformation into a scaling part and a
rotation/shearing part.  Use the scaling part to compute a new character pixel
size, then the other one to call `FT_Set_Transform`.  This is explained in more
detail in part II of this tutorial.

Rotation usually disables hinting.

Loading a glyph bitmap with a non-identity transformation works; the
transformation is ignored in this case.

### 7\. Simple Text Rendering

We now present a simple example to render a string of 8-bit Latin-1 text,
assuming a face that contains a Unicode charmap.

The idea is to create a loop that loads one glyph image on each iteration,
converts it to a pixmap, draws it on the target surface, then increments the
current pen position.

#### a. Basic Code

The following code performs our simple text rendering with the functions
previously described.

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

This code needs a few explanations.

* We define a handle named `slot` that points to the face object's glyph slot.
  (The type [`FT_GlyphSlot`](../reference/ft2-base_interface#FT_GlyphSlot)
  is a pointer).  That is a convenience to avoid using `face->glyph->XXX` every
  time.
* We increment the pen position with the vector `slot->advance`, which
  correspond to the glyph's _advance width_ (also known as its _escapement_).
  The advance vector is expressed in 1/64th of pixels, and is truncated to
  integer pixels on each iteration.
* The function `my_draw_bitmap` is not part of FreeType but must be provided by
  the application to draw the bitmap to the target surface.  In this example,
  it takes a pointer to an
  [`FT_Bitmap`](../reference/ft2-basic_types#FT_Bitmap) descriptor and the
  position of its top-left corner as arguments.  For ideal rendering on a
  screen this function should perform linear blending with gamma correction,
  using the bitmap as an alpha channel.
* The value of `slot->bitmap_top` is positive for an _upwards_ vertical
  distance.  Assuming that the coordinates taken by `my_draw_bitmap` use the
  opposite convention (increasing Y corresponds to downwards scanlines), we
  subtract it from `pen_y`, instead of adding to it.

#### b.Refined code

The following code is a refined version of the example above.  It uses features
and functions of FreeType that have not yet been introduced, and which are
explained below.

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

We have reduced the size of our code, but it does exactly the same thing.

* We use the function
  [`FT_Load_Char`](../reference/ft2-base_interface#FT_Load_Char) instead
  of `FT_Load_Glyph`.  As you probably imagine, it is equivalent to calling
  `FT_Get_Char_Index`, then `FT_Load_Glyph`.
* We do not use `FT_LOAD_DEFAULT` for the loading mode, but the bit flag
  `FT_LOAD_RENDER`.  It indicates that the glyph image must be immediately
  converted to an anti-aliased bitmap.  This is of course a shortcut that
  avoids calling `FT_Render_Glyph` explicitly but is strictly equivalent.

    Note that you can also specify that you want a monochrome bitmap instead by
    using the additional `FT_LOAD_MONOCHROME` load flag.

#### c. More Advanced Rendering

Let us try to render transformed text now (for example through a rotation).  We
can do this using `FT_Set_Transform`.

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

Some remarks.

* We now use a vector of type `FT_Vector` to store the pen position, with
  coordinates expressed as 1/64th of pixels, hence a multiplication.  The
  position is expressed in cartesian space.
* Glyph images are always loaded, transformed, and described in the cartesian
  coordinate system within FreeType (which means that increasing Y corresponds
  to upper scanlines), unlike the system typically used for bitmaps (where the
  topmost scanline has coordinate 0).  We must thus convert between the two
  systems when we define the pen position, and when we compute the topleft
  position of the bitmap.
* We set the transformation on each glyph to indicate the rotation matrix as
  well as a delta that moves the transformed image to the current pen position
  (in cartesian space, not bitmap space).  As a consequence, the values of
  `bitmap_left` and `bitmap_top` correspond to the bitmap origin in target
  space pixels.  We thus don't add `pen.x` or `pen.y` to their values when
  calling `my_draw_bitmap`.
* The advance width is always returned transformed, which is why it can be
  directly added to the current pen position.  Note that it is _not_ rounded
  this time.

A complete source code example can be found [here](example1.c).

It is important to note that, while this example is a bit more complex than the
previous one, it is strictly equivalent for the case where the transformation
is the identity.  Hence it can be used as a replacement (but a more powerful
one).

The still present few shortcomings will be explained, and solved, in the next
part of this tutorial.

## II. Managing Glyphs

### 1\. Glyph Metrics

Glyph metrics are, as the name suggests, certain distances associated with each
glyph that describe how to position this glyph while creating a text layout.

There are usually two sets of metrics for a single glyph: Those used to
represent glyphs in horizontal text layouts (Latin, Cyrillic, Arabic, Hebrew,
etc.), and those used to represent glyphs in vertical text layouts (Chinese,
Japanese, Korean, Mongolian, etc.).

Note that only a few font formats provide vertical metrics.  You can test
whether a given face object contains them by using the macro
[`FT_HAS_VERTICAL`](../reference/ft2-base_interface#FT_HAS_VERTICAL),
which returns true if appropriate.

Individual glyph metrics can be accessed by first loading the glyph in a face's
glyph slot, then accessing them through the `face->glyph->metrics` structure,
whose type is
[`FT_Glyph_Metrics`](../reference/ft2-base_interface#FT_Glyph_Metrics).
We will discuss this in more detail below; for now, we only note that it
contains the following fields.

`width`| This is the width of the glyph image's bounding box. It is independent of the layout direction.
`height`| This is the height of the glyph image's bounding box. It is independent of the layout direction. Be careful not to confuse it with the 'height' field in the [`FT_Size_Metrics`](../reference/ft2-base_interface#FT_Size_Metrics) structure.
`horiBearingX`| For _horizontal text layouts_, this is the horizontal distance from the current cursor position to the leftmost border of the glyph image's bounding box.
`horiBearingY`| For _horizontal text layouts_, this is the vertical distance from the current cursor position (on the baseline) to the topmost border of the glyph image's bounding box.
`horiAdvance`| For _horizontal text layouts_, this is the horizontal distance to increment the pen position when the glyph is drawn as part of a string of text.
`vertBearingX`| For _vertical text layouts_, this is the horizontal distance from the current cursor position to the leftmost border of the glyph image's bounding box.
`vertBearingY`| For _vertical text layouts_, this is the vertical distance from the current cursor position (on the baseline) to the topmost border of the glyph image's bounding box.
`vertAdvance`| For _vertical text layouts_, this is the vertical distance used to increment the pen position when the glyph is drawn as part of a string of text.

As not all fonts do contain vertical metrics, the values of `vertBearingX`,
`vertBearingY` and `vertAdvance` should not be considered reliable if
`FT_HAS_VERTICAL` returns false.

The following graphics illustrate the metrics more clearly.  In case a distance
is directed, it is marked with a single arrow, indicating a positive value.
The first image displays horizontal metrics, where the baseline is the
horizontal axis.

<div class="figure">![horizontal layout](metrics.png)</div>

For vertical text layouts, the baseline is vertical, identical to the vertical
axis.  Contrary to all other arrows, `bearingX` shows a negative value in this
image.

<div class="figure">![vertical layout](metrics2.png)</div>

The metrics found in `face->glyph->metrics` are normally expressed in 26.6
pixel format (i.e., 1/64th of pixels), unless you use the `FT_LOAD_NO_SCALE`
flag when calling `FT_Load_Glyph` or `FT_Load_Char`.  In this case, the metrics
are expressed in original font units.

The glyph slot object has also a few other interesting fields that eases a
developer's work.  You can access them through `face->glyph->xxx`, where `xxx`
is one of the following fields.

`advance`| This field is a `FT_Vector` that holds the transformed advance for the glyph. That is useful when you are using a transformation through `FT_Set_Transform`, as shown in the [rotated text example](step1.html#transformed-text) of part I. Other than that, its value is by default (metrics.horiAdvance,0), unless you specify `FT_LOAD_VERTICAL` when loading the glyph image; it is then (0,metrics.vertAdvance).
`linearHoriAdvance`| This field contains the linearly scaled value of the glyph's horizontal advance width. Indeed, the value of `metrics.horiAdvance` that is returned in the glyph slot is normally rounded to integer pixel coordinates (i.e., being a multiple of 64) by the font driver that actually loads the glyph image. `linearHoriAdvance` is a 16.16 fixed-point number that gives the value of the original glyph advance width in 1/65536th of pixels. It can be use to perform pseudo device-independent text layouts.
`linearVertAdvance`| This is the similar to `linearHoriAdvance` but for the glyph's vertical advance height. Its value is only reliable if the font face contains vertical metrics.

### 2\. Managing Glyph Images

The glyph image that is loaded in a glyph slot can be converted into a bitmap,
either by using `FT_LOAD_RENDER` when loading it, or by calling
[`FT_Render_Glyph`](../reference/ft2-base_interface#FT_Render_Glyph).
Each time you load a new glyph image, the previous one is erased from the glyph
slot.

There are situations, however, where you may need to extract this image from
the glyph slot in order to cache it within your application, and even perform
additional transformations and measures on it before converting it to a bitmap.

The FreeType 2 API has a specific extension that is capable of dealing with
glyph images in a flexible and generic way.  To use it, you first need to
include the [`FT_GLYPH_H`](../reference/ft2-header_file_macros#FT_GLYPH_H)
header file.

    #include FT_GLYPH_H

#### a.Extracting the Glyph Image

You can extract a single glyph image very easily.  Here some code that shows
how to do it.

    FT_Glyph  glyph; /* a handle to the glyph image */

    ...
    error = FT_Load_Glyph( face, glyph_index, FT_LOAD_NORMAL );
    if ( error ) { ... }

    error = FT_Get_Glyph( face->glyph, &glyph );
    if ( error ) { ... }

The following steps are performed.

* Create a variable named `glyph`, of type
  [`FT_Glyph`](../reference/ft2-glyph_management#FT_Glyph).  This is a
  handle (pointer) to an individual glyph image.
* Load the glyph image in the normal way into the face's glyph slot.  We don't
  use `FT_LOAD_RENDER` because we want to grab a scalable glyph image that we
  can transform later on.
* Copy the glyph image from the slot into a new `FT_Glyph` object by calling
  [`FT_Get_Glyph`](../reference/ft2-glyph_management#FT_Get_Glyph).  This
  function returns an error code and sets `glyph`.

It is important to note that the extracted glyph is in the same format as the
original one that is still in the slot.  For example, if we are loading a glyph
from a TrueType font file, the glyph image is really a scalable vector outline.
You can access the field `glyph->format` if you want to know exactly how the
glyph is modeled and stored.

A new glyph object can be destroyed with a call to
[`FT_Done_Glyph`](../reference/ft2-glyph_management#FT_Done_Glyph).

The glyph object contains exactly one glyph image and a 2D vector representing
the glyph's advance in 16.16 fixed-point coordinates.  The latter can be
accessed directly as `glyph->advance`

Note that unlike other FreeType objects, the library doesn't keep a list of all
allocated glyph objects.  This means you have to destroy them yourself instead
of relying on `FT_Done_FreeType` doing all the clean-up.

#### b. Transforming & Copying the Glyph Image

If the glyph image is scalable (i.e., if `glyph->format` is not equal to
`FT_GLYPH_FORMAT_BITMAP`), it is possible to transform the image anytime by a
call to
[`FT_Glyph_Transform`](../reference/ft2-glyph_management#FT_Glyph_Transform).

You can also copy a single glyph image with
[`FT_Glyph_Copy`](../reference/ft2-glyph_management#FT_Glyph_Copy).

    FT_Glyph   glyph, glyph2;
    FT_Matrix  matrix;
    FT_Vector  delta;

    ... load glyph image in `glyph' ...

    /* copy glyph to glyph2 */

    error = FT_Glyph_Copy( glyph, &glyph2 );
    if ( error ) { ... could not copy (out of memory) ... }

    /* translate `glyph' */

    delta.x = -100 * 64; /* coordinates are in 26.6 pixel format */
    delta.y =   50 * 64;

    FT_Glyph_Transform( glyph, 0, &delta );

    /* transform glyph2 (horizontal shear) */

    matrix.xx = 0x10000L;
    matrix.xy = 0.12 * 0x10000L;
    matrix.yx = 0;
    matrix.yy = 0x10000L;

    FT_Glyph_Transform( glyph2, &matrix, 0 );

Note that the 2×2 transformation matrix is always applied to the 16.16 advance
vector in the glyph; you thus don't need to recompute it.

#### c. Measuring the Glyph Image

You can also retrieve the control (bounding) box of any glyph image (scalable
or not) through the
[`FT_Glyph_Get_CBox`](../reference/ft2-glyph_management#FT_Glyph_Get_CBox)
function.

    FT_BBox  bbox;

    ...
    FT_Glyph_Get_CBox( glyph, bbox_mode, &bbox );

Coordinates are relative to the glyph origin (0,0), using the y upwards
convention.  This function takes a special argument, the _bbox mode_, to
indicate how box coordinates are expressed.

If the glyph has been loaded with `FT_LOAD_NO_SCALE`, `bbox_mode` must be set
to `FT_GLYPH_BBOX_UNSCALED` to get unscaled font units in 26.6 pixel format.
The value `FT_GLYPH_BBOX_SUBPIXELS` is another name for this constant.

Note that the box's maximum coordinates are exclusive, which means that you can
always compute the width and height of the glyph image (regardless of using
integer or 26.6 coordinates) with a simple subtraction.

    width  = bbox.xMax - bbox.xMin;
    height = bbox.yMax - bbox.yMin;

Note also that for 26.6 coordinates, if `FT_GLYPH_BBOX_GRIDFIT` is used as the
bbox mode, the coordinates are also grid-fitted, which corresponds to the
following four lines.

    bbox.xMin = FLOOR( bbox.xMin )
    bbox.yMin = FLOOR( bbox.yMin )
    bbox.xMax = CEILING( bbox.xMax )
    bbox.yMax = CEILING( bbox.yMax )

To get the bbox in _integer_ pixel coordinates, set `bbox_mode` to
`FT_GLYPH_BBOX_TRUNCATE`.

Finally, to get the bounding box in grid-fitted pixel coordinates, set
`bbox_mode` to `FT_GLYPH_BBOX_PIXELS`.

[Computing _exact_ bounding boxes can be done with function
[`FT_Outline_Get_BBox`](../reference/ft2-outline_processing#FT_Outline_Get_BBox),
at the cost of slower execution.  You probably don't need with the possible
exception of rotated glyphs.]

#### d. Converting the Glyph Image to a Bitmap

You may need to convert the glyph object to a bitmap once you have conveniently
cached or transformed it.  This can be done easily with the
[`FT_Glyph_To_Bitmap`](../reference/ft2-glyph_management) function, which
handles any glyph object.

    FT_Vector  origin;

    origin.x = 32; /* 1/2 pixel in 26.6 format */
    origin.y = 0;

    error = FT_Glyph_To_Bitmap( &glyph,
                                render_mode,
                                &origin,
                                1 );          /* destroy original image == true */

Some notes.

* The first parameter is the address of the source glyph's handle.  When the
  function is called, it reads it to access the source glyph object.  After the
  call, the handle points to a _new_ glyph object that contains the rendered
  bitmap.
* The second parameter is a standard render mode to specify what kind of bitmap
  we want.  For example, it can be `FT_RENDER_MODE_DEFAULT` for an 8-bit
  anti-aliased pixmap, or `FT_RENDER_MODE_MONO` for a 1-bit monochrome bitmap.
* The third parameter is a pointer to a two-dimensional vector to translate the
  source glyph image before the conversion.  After the call, the source image
  is translated back to its original position (and is thus left unchanged).  If
  you do not need to translate the source glyph before rendering, set this
  pointer to NULL.
* The last parameter is a boolean that indicates whether the source glyph
  object should be destroyed by the function.  If false, the original glyph
  object is never destroyed, even if its handle is lost (it is up to client
  applications to keep it).

The new glyph object always contains a bitmap (if no error is returned), and
you must _typecast_ its handle to the `FT_BitmapGlyph` type in order to access
its content.  This type is a sort of 'subclass' of `FT_Glyph` that contains
additional fields (see
[`FT_BitmapGlyphRec`](../reference/ft2-glyph_management#FT_BitmapGlyphRec)).

`left`| Just like the `bitmap_left` field of a glyph slot, this is the horizontal distance from the glyph origin (0,0) to the leftmost pixel of the glyph bitmap.  It is expressed in integer pixels.
`top`| Just like the `bitmap_top` field of a glyph slot, this is the vertical distance from the glyph origin (0,0) to the topmost pixel of the glyph bitmap (more precise, to the pixel just above the bitmap).  This distance is expressed in integer pixels, and is positive for upwards y.
`bitmap`| This is a bitmap descriptor for the glyph object, just like the `bitmap` field in a glyph slot.

### 3\. Global Glyph Metrics

Unlike glyph metrics, global metrics are used to describe distances and
features of a whole font face.  They can be expressed either in 26.6 pixel
format or in (unscaled) font units for scalable formats.

#### a. Design global metrics

For scalable formats, all global metrics are expressed in font units in order
to be later scaled to the device space, according to the rules described in the
last section of this tutorial part.  You can access them directly as fields of
a `FT_Face` handle.

However, you need to check that the font face's format is scalable before using
them.  One can do it with macro `FT_IS_SCALABLE`, which returns true when
appropriate.

Here a table of the global design metrics for scalable faces.

`units_per_EM`| This is the size of the EM square for the font face.  It is used by scalable formats to scale design coordinates to device pixels, as described in the last section of this tutorial part.  Its value usually is 2048 (for TrueType) or 1000 (for Type 1 or CFF), but other values are possible, too.  It is set to 1 for fixed-size formats like FNT, FON, PCF, or BDF.
`bbox`| The global bounding box is defined as the smallest rectangle that can enclose all the glyphs in a font face.
`ascender`| The ascender is the vertical distance from the horizontal baseline to the highest 'character' coordinate in a font face. Unfortunately, font formats don't define the ascender in a uniform way.  For some formats, it represents the ascent of all capital latin characters (without accents), for others it is the ascent of the highest accented character, and finally, other formats define it as being equal to `bbox.yMax`.
`descender`| The descender is the vertical distance from the horizontal baseline to the lowest 'character' coordinate in a font face.  Unfortunately, font formats don't define the descender in a uniform way. For some formats, it represents the descent of all capital latin characters (without accents), for others it is the ascent of the lowest accented character, and finally, other formats define it as being equal to `bbox.yMin`. This field is negative for values below the baseline.
`height`| This field represents a _default line spacing_ (i.e., the baseline-to-baseline distance) when writing text with this font. Note that it usually is larger than the sum of the ascender and descender taken as absolute values.  There is also no guarantee that no glyphs extend above or below subsequent baselines when using this distance – think of it as a value the designer of the font finds appropriate.
`max_advance_width`| This field gives the maximum horizontal cursor advance for all glyphs in the font.  It can be used to quickly compute the maximum advance width of a string of text.  _It doesn't correspond to the maximum glyph image width!_
`max_advance_height`| Same as `max_advance_width` but for vertical text layout.
`underline_position`| When displaying or rendering underlined text, this value corresponds to the vertical position, relative to the baseline, of the underline bar's center.  It is negative if it is below the baseline.
`underline_thickness`| When displaying or rendering underlined text, this value corresponds to the vertical thickness of the underline.

Notice that the values of the ascender and the descender are not reliable (due
to various discrepancies in font formats), unfortunately.

#### b. Scaled Global Metrics

Each size object also contains a scaled version of some of the global metrics
described above, to be directly accessed through the `face->size->metrics`
structure (of type
[`FT_Size_Metrics`](../reference/ft2-base_interface#FT_Size_Metrics)).
_No rounding or grid-fitting is performed for those values_.  They are also
completely independent of any hinting process.  In other words, don't rely on
them to get exact metrics at the pixel level.  They are expressed in 26.6 pixel
format.

`ascender` | The scaled version of the original design ascender.
`descender`| The scaled version of the original design descender.
`height`   |   The scaled version of the original design text height (the vertical distance from one baseline to the next).  This is probably the only field you should really use in this structure.  Be careful not to confuse it with the 'height' field in the [`FT_Glyph_Metrics`](../reference/ft2-base_interface#FT_Glyph_Metrics) structure.
`max_advance`| The scaled version of the original design maximum advance.

Note that the `face->size->metrics` structure contains other fields that are
used to scale design coordinates to device space.  They are described in the
last section.

#### c. Kerning

Kerning is the process of adjusting the position of two subsequent glyph images
in a string of text in order to improve the general appearance of text.  For
example, if a glyph for an uppercase 'A' is followed by a glyph for an
uppercase 'V', the space between the two glyphs can be slightly reduced to
avoid extra 'diagonal whitespace'.

Note that in theory kerning can happen both in the horizontal and vertical
direction between two glyphs; however, it only happens in a single direction in
nearly all cases.

Not all font formats contain kerning information, and not all kerning formats
are supported by FreeType; in particular, for TrueType fonts, the API can only
access kerning via the 'kern' table.  <span class="important">OpenType kerning
via the 'GPOS' table is not supported!</span>  You need a higher-level library
like [HarfBuzz](http://www.harfbuzz.org), [Pango](http://www.pango.org), or
[ICU](http://www.icu-project.org), since GPOS kerning requires contextual
string handling.

Sometimes, the font file is associated with an additional file that contains
various glyph metrics, including kerning, but no glyph images.  A good example
is the Type 1 format where glyph images are stored in files with extension
`.pfa` or `.pfb`, while kerning metrics can be found in files with extension
`.afm` or `.pfm`.

FreeType 2 allows you to deal with this, by providing the
[`FT_Attach_File`](../reference/ft2-base_interface#FT_Attach_File) and
[`FT_Attach_Stream`](../reference/ft2-base_interface#FT_Attach_Stream)
APIs.  Both functions are used to load additional metrics into a face object by
reading them from an additional format-specific file.  Here an example, opening
a Type 1 font.

    error = FT_New_Face( library, "/usr/share/fonts/cour.pfb",
                         0, &face );
    if ( error ) { ... }

    error = FT_Attach_File( face, "/usr/share/fonts/cour.afm" );
    if ( error )
    { ... could not read kerning and additional metrics ... }

Note that `FT_Attach_Stream` is similar to `FT_Attach_File` except that it
doesn't take a C string to name the extra file but an
[`FT_Stream`](../reference/ft2-system_interface#FT_StreamRec) handle.
Also, _reading a metrics file is in no way mandatory_.

Finally, the file attachment APIs are very generic and can be used to load any
kind of extra information for a given face.  The nature of the additional
content is entirely font format specific.

FreeType 2 allows you to retrieve the kerning information between two glyphs
through the
[`FT_Get_Kerning`](../reference/ft2-base_interface#FT_Get_Kerning)
function.

    FT_Vector  kerning;

    ...
    error = FT_Get_Kerning( face,          /* handle to face object */
                            left,          /* left glyph index      */
                            right,         /* right glyph index     */
                            kerning_mode,  /* kerning mode          */
                            &kerning );    /* target vector         */

This function takes a handle to a face object, the indices of the left and
right glyph for which the kerning value is desired, an integer, called the
_kerning mode_, and a pointer to a destination vector that receives the
corresponding distances.

The kerning mode is very similar to the _bbox mode_ described in a previous
section.  It is a enumeration that indicates how the kerning distances are
expressed in the target vector.

The default value is `FT_KERNING_DEFAULT`, which has value 0\.  It corresponds
to kerning distances expressed in 26.6 grid-fitted pixels (which means that the
values are multiples of 64).  For scalable formats, this means that the design
kerning distance is scaled, then rounded.

The value `FT_KERNING_UNFITTED` corresponds to kerning distances expressed in
26.6 unfitted pixels (i.e., that do not correspond to integer coordinates).  It
is the design kerning distance that is scaled without rounding.

Finally, the value `FT_KERNING_UNSCALED` indicates to return the design kerning
distance, expressed in font units.  You can later scale it to the device space
using the computations explained in the last section of this part.

Note that the 'left' and 'right' positions correspond to the _visual order_ of
the glyphs in the string of text.  This is important for bidirectional or
right-to-left text.

### 4\. Simple Text Rendering: Kerning and Centering

In order to show off what we have just learned, we now demonstrate how to
modify the [example code](step1.html#basic-code) that was provided in part I to
render a string of text, and enhance it to support kerning and delayed
rendering.

#### a. Kerning Support

Adding support for kerning to our code is trivial, as long as we consider that
we are still dealing with a left-to-right script like Latin.  We simply need to
retrieve the kerning distance between two glyphs in order to alter the pen
position appropriately.

    FT_GlyphSlot  slot = face->glyph;  /* a small shortcut */
    FT_UInt       glyph_index;
    FT_Bool       use_kerning;
    FT_UInt       previous;
    int           pen_x, pen_y, n;

    ... initialize library ...
    ... create face object ...
    ... set character size ...

    pen_x = 300;
    pen_y = 200;

    use_kerning = FT_HAS_KERNING( face );
    previous    = 0;

    for ( n = 0; n < num_chars; n++ )
    {
      /* convert character code to glyph index */
      glyph_index = FT_Get_Char_Index( face, text[n] );

      /* retrieve kerning distance and move pen position */
      if ( use_kerning && previous && glyph_index )
      {
        FT_Vector  delta;


        FT_Get_Kerning( face, previous, glyph_index,
                        FT_KERNING_DEFAULT, &delta );

        pen_x += delta.x >> 6;
      }

      /* load glyph image into the slot (erase previous one) */
      error = FT_Load_Glyph( face, glyph_index, FT_LOAD_RENDER );
      if ( error )
        continue;  /* ignore errors */

      /* now draw to our target surface */
      my_draw_bitmap( &slot->bitmap,
                      pen_x + slot->bitmap_left,
                      pen_y - slot->bitmap_top );

      /* increment pen position */
      pen_x += slot->advance.x >> 6;

      /* record current glyph index */
      previous = glyph_index;
    }

We are done.  Some notes.

* As kerning is determined by glyph indices, we need to explicitly convert our
  character codes into glyph indices, then later call `FT_Load_Glyph` instead
  of `FT_Load_Char`.
* We use a boolean named `use_kerning`, which is set to the result of the macro
  `FT_HAS_KERNING`.  It is certainly faster not to call `FT_Get_Kerning` when
  we know that the font face does not contain kerning information.
* We move the position of the pen _before_ a new glyph is drawn.
* We initialize the variable `previous` with the value 0, which always
  corresponds to the 'missing glyph' (also called `.notdef` in the PostScript
  world).  There is never any kerning distance associated with this glyph.
* We do not check the error code returned by `FT_Get_Kerning`.  This is because
  the function always sets the content of `delta` to (0,0) if an error occurs.

#### b. Centering

Our code begins to become interesting but it is still a bit too simple for
normal use.  For example, the position of the pen is determined before we do
the rendering; normally, you would rather determine the layout of the text and
measure it before computing its final position (centering, etc.), or perform
things like word-wrapping.

Let us now decompose our text rendering function into two distinct but
successive parts: The first one positions individual glyph images on the
baseline, while the second one renders the glyphs.  As we will see, this has
many advantages.

We thus start by storing individual glyph images, as well as their position on
the baseline.

    FT_GlyphSlot  slot = face->glyph;   /* a small shortcut */
    FT_UInt       glyph_index;
    FT_Bool       use_kerning;
    FT_UInt       previous;
    int           pen_x, pen_y, n;

    FT_Glyph      glyphs[MAX_GLYPHS];   /* glyph image    */
    FT_Vector     pos   [MAX_GLYPHS];   /* glyph position */
    FT_UInt       num_glyphs;

    ... initialize library ...
    ... create face object ...
    ... set character size ...

    pen_x = 0;   /* start at (0,0) */
    pen_y = 0;

    num_glyphs  = 0;
    use_kerning = FT_HAS_KERNING( face );
    previous    = 0;

    for ( n = 0; n < num_chars; n++ )
    {
      /* convert character code to glyph index */
      glyph_index = FT_Get_Char_Index( face, text[n] );

      /* retrieve kerning distance and move pen position */
      if ( use_kerning && previous && glyph_index )
      {
        FT_Vector  delta;


        FT_Get_Kerning( face, previous, glyph_index,
                        FT_KERNING_DEFAULT, &delta );

        pen_x += delta.x >> 6;
      }

      /* store current pen position */
      pos[num_glyphs].x = pen_x;
      pos[num_glyphs].y = pen_y;

      /* load glyph image into the slot without rendering */
      error = FT_Load_Glyph( face, glyph_index, FT_LOAD_DEFAULT );
      if ( error )
        continue;  /* ignore errors, jump to next glyph */

      /* extract glyph image and store it in our table */
      error = FT_Get_Glyph( face->glyph, &glyphs[num_glyphs] );
      if ( error )
        continue;  /* ignore errors, jump to next glyph */

      /* increment pen position */
      pen_x += slot->advance.x >> 6;

      /* record current glyph index */
      previous = glyph_index;

      /* increment number of glyphs */
      num_glyphs++;
    }

This is a very slight variation of our previous code; we extract each glyph
image from the slot, then store it, along with the corresponding position, in
our tables.

Note also that `pen_x` contains the total advance for the string of text.  We
can now compute the bounding box of the text string with a simple function.

    void  compute_string_bbox( FT_BBox  *abbox )
    {
      FT_BBox  bbox;
      FT_BBox  glyph_bbox;


      /* initialize string bbox to "empty" values */
      bbox.xMin = bbox.yMin =  32000;
      bbox.xMax = bbox.yMax = -32000;

      /* for each glyph image, compute its bounding box, */
      /* translate it, and grow the string bbox          */
      for ( n = 0; n < num_glyphs; n++ )
      {
        FT_Glyph_Get_CBox( glyphs[n], ft_glyph_bbox_pixels,
                           &glyph_bbox );

        glyph_bbox.xMin += pos[n].x;
        glyph_bbox.xMax += pos[n].x;
        glyph_bbox.yMin += pos[n].y;
        glyph_bbox.yMax += pos[n].y;

        if ( glyph_bbox.xMin < bbox.xMin )
          bbox.xMin = glyph_bbox.xMin;

        if ( glyph_bbox.yMin < bbox.yMin )
          bbox.yMin = glyph_bbox.yMin;

        if ( glyph_bbox.xMax > bbox.xMax )
          bbox.xMax = glyph_bbox.xMax;

        if ( glyph_bbox.yMax > bbox.yMax )
          bbox.yMax = glyph_bbox.yMax;
      }

      /* check that we really grew the string bbox */
      if ( bbox.xMin > bbox.xMax )
      {
        bbox.xMin = 0;
        bbox.yMin = 0;
        bbox.xMax = 0;
        bbox.yMax = 0;
      }

      /* return string bbox */
      *abbox = bbox;
    }

The resulting bounding box dimensions are expressed in integer pixels and can
then be used to compute the final pen position before rendering the string.

In general, the above function does _not_ compute an exact bounding box of a
string!  As soon as hinting is involved, glyph dimensions _must_ be derived
from the resulting outlines.  For anti-aliased pixmaps, `FT_Outline_Get_BBox`
then yields proper results.  In case you need 1-bit monochrome bitmaps, it is
even necessary to actually render the glyphs because the rules for the
conversion from outline to bitmap can also be controlled by hinting
instructions.

    void  compute_string_bbox( FT_BBox  *abbox )
    {
      FT_BBox  bbox;
      FT_BBox  glyph_bbox;


      /* initialize string bbox to "empty" values */
      bbox.xMin = bbox.yMin =  32000;
      bbox.xMax = bbox.yMax = -32000;

      /* for each glyph image, compute its bounding box, */
      /* translate it, and grow the string bbox          */
      for ( n = 0; n < num_glyphs; n++ )
      {
        FT_Glyph_Get_CBox( glyphs[n], ft_glyph_bbox_pixels,
                           &glyph_bbox );

        glyph_bbox.xMin += pos[n].x;
        glyph_bbox.xMax += pos[n].x;
        glyph_bbox.yMin += pos[n].y;
        glyph_bbox.yMax += pos[n].y;

        if ( glyph_bbox.xMin < bbox.xMin )
          bbox.xMin = glyph_bbox.xMin;

        if ( glyph_bbox.yMin < bbox.yMin )
          bbox.yMin = glyph_bbox.yMin;

        if ( glyph_bbox.xMax > bbox.xMax )
          bbox.xMax = glyph_bbox.xMax;

        if ( glyph_bbox.yMax > bbox.yMax )
          bbox.yMax = glyph_bbox.yMax;
      }

      /* check that we really grew the string bbox */
      if ( bbox.xMin > bbox.xMax )
      {
        bbox.xMin = 0;
        bbox.yMin = 0;
        bbox.xMax = 0;
        bbox.yMax = 0;
      }

      /* return string bbox */
      *abbox = bbox;
    }

Some remarks.

* The pen position is expressed in the Cartesian space (i.e., y upwards).
* We call `FT_Glyph_To_Bitmap` with the `destroy` parameter set to 0 (false),
  in order to avoid destroying the original glyph image.  The new glyph bitmap
  is accessed through `image` after the call and is typecast to
  `FT_BitmapGlyph`.
* We use translation when calling `FT_Glyph_To_Bitmap`.  This ensures that the
  `left` and `top` fields of the bitmap glyph object are already set to the
  correct pixel coordinates in the Cartesian space.
* Of course, we still need to convert pixel coordinates from Cartesian to
  device space before rendering, hence the `my_target_height - bitmap->top` in
  the call to `my_draw_bitmap`.

The same loop can be used to render the string anywhere on our display surface,
without the need to reload our glyph images each time.

### 5\. Advanced Text Rendering: Transformation and Centering and Kerning

We are now going to modify our code in order to be able to easily transform the
rendered string, for example, to rotate it.  First, some minor improvements.

#### a. Packing and Translating Glyphs

We start by packing the information related to a single glyph image into a
single structure instead of parallel arrays.

    typedef struct  TGlyph_
    {
      FT_UInt    index;  /* glyph index                  */
      FT_Vector  pos;    /* glyph origin on the baseline */
      FT_Glyph   image;  /* glyph image                  */

    } TGlyph, *PGlyph;

We also translate each glyph image directly after it is loaded to its position
on the baseline at load time.  As we will see, this has several advantages.
Here is our new glyph sequence loader.

    FT_GlyphSlot  slot = face->glyph;  /* a small shortcut */
    FT_UInt       glyph_index;
    FT_Bool       use_kerning;
    FT_UInt       previous;
    int           pen_x, pen_y, n;

    TGlyph        glyphs[MAX_GLYPHS];  /* glyphs table */
    PGlyph        glyph;               /* current glyph in table */
    FT_UInt       num_glyphs;


    ... initialize library ...
    ... create face object ...
    ... set character size ...

    pen_x = 0;   /* start at (0,0) */
    pen_y = 0;

    num_glyphs  = 0;
    use_kerning = FT_HAS_KERNING( face );
    previous    = 0;

    glyph = glyphs;
    for ( n = 0; n < num_chars; n++ )
    {
      glyph->index = FT_Get_Char_Index( face, text[n] );

      if ( use_kerning && previous && glyph->index )
      {
        FT_Vector  delta;


        FT_Get_Kerning( face, previous, glyph->index,
                        FT_KERNING_MODE_DEFAULT, &delta );

        pen_x += delta.x >> 6;
      }

      /* store current pen position */
      glyph->pos.x = pen_x;
      glyph->pos.y = pen_y;

      error = FT_Load_Glyph( face, glyph_index, FT_LOAD_DEFAULT );
      if ( error ) continue;

      error = FT_Get_Glyph( face->glyph, &glyph->image );
      if ( error ) continue;

      /* translate the glyph image now */
      FT_Glyph_Transform( glyph->image, 0, &glyph->pos );

      pen_x   += slot->advance.x >> 6;
      previous = glyph->index;

      /* increment number of glyphs */
      glyph++;
    }

    /* count number of glyphs loaded */
    num_glyphs = glyph - glyphs;

Note that translating glyphs now has several advantages.  The first one is that
we don't need to translate the glyph bbox when we compute the string's bounding
box.

    void  compute_string_bbox( FT_BBox  *abbox )
    {
      FT_BBox  bbox;


      bbox.xMin = bbox.yMin =  32000;
      bbox.xMax = bbox.yMax = -32000;

      for ( n = 0; n < num_glyphs; n++ )
      {
        FT_BBox  glyph_bbox;


        FT_Glyph_Get_CBox( glyphs[n], ft_glyph_bbox_pixels,
                           &glyph_bbox );

        if (glyph_bbox.xMin < bbox.xMin)
          bbox.xMin = glyph_bbox.xMin;

        if (glyph_bbox.yMin < bbox.yMin)
          bbox.yMin = glyph_bbox.yMin;

        if (glyph_bbox.xMax > bbox.xMax)
          bbox.xMax = glyph_bbox.xMax;

        if (glyph_bbox.yMax > bbox.yMax)
          bbox.yMax = glyph_bbox.yMax;
      }

      if ( bbox.xMin > bbox.xMax )
      {
        bbox.xMin = 0;
        bbox.yMin = 0;
        bbox.xMax = 0;
        bbox.yMax = 0;
      }

      *abbox = bbox;
    }

With the above modifications, the `compute_string_bbox` function can now
compute the bounding box of a transformed glyph string, which allows further
code simplications.

    FT_BBox    bbox;
    FT_Matrix  matrix;
    FT_Vector  delta;


    ... load glyph sequence ...
    ... set up `matrix' and `delta' ...

    /* transform glyphs */
    for ( n = 0; n < num_glyphs; n++ )
      FT_Glyph_Transform( glyphs[n].image, &matrix, &delta );

    /* compute bounding box of transformed glyphs */
    compute_string_bbox( &bbox );

#### b. Rendering a Transformed Glyph Sequence

However, directly transforming the glyphs in our sequence is not a good idea if
we want to reuse them in order to draw the text string with various angles or
transformations.  It is better to perform the affine transformation just before
the glyph is rendered.

    FT_Vector  start;
    FT_Matrix  matrix;

    FT_Glyph   image;
    FT_Vector  pen;
    FT_BBox    bbox;


    /* get bbox of original glyph sequence */
    compute_string_bbox( &string_bbox );

    /* compute string dimensions in integer pixels */
    string_width  = (string_bbox.xMax - string_bbox.xMin) / 64;
    string_height = (string_bbox.yMax - string_bbox.yMin) / 64;

    /* set up start position in 26.6 Cartesian space */
    start.x = ( ( my_target_width  - string_width  ) / 2 ) * 64;
    start.y = ( ( my_target_height - string_height ) / 2 ) * 64;

    /* set up transform (a rotation here) */
    matrix.xx = (FT_Fixed)( cos( angle ) * 0x10000L );
    matrix.xy = (FT_Fixed)(-sin( angle ) * 0x10000L );
    matrix.yx = (FT_Fixed)( sin( angle ) * 0x10000L );
    matrix.yy = (FT_Fixed)( cos( angle ) * 0x10000L );

    pen = start;

    for ( n = 0; n < num_glyphs; n++ )
    {
      /* create a copy of the original glyph */
      error = FT_Glyph_Copy( glyphs[n].image, &image );
      if ( error ) continue;

      /* transform copy (this will also translate it to the */
      /* correct position                                   */
      FT_Glyph_Transform( image, &matrix, &pen );

      /* check bounding box; if the transformed glyph image      */
      /* is not in our target surface, we can avoid rendering it */
      FT_Glyph_Get_CBox( image, ft_glyph_bbox_pixels, &bbox );
      if ( bbox.xMax <= 0 || bbox.xMin >= my_target_width  ||
           bbox.yMax <= 0 || bbox.yMin >= my_target_height )
        continue;

      /* convert glyph image to bitmap (destroy the glyph copy!) */
      error = FT_Glyph_To_Bitmap(
                &image,
                FT_RENDER_MODE_NORMAL,
                0,                  /* no additional translation */
                1 );                /* destroy copy in "image"   */
      if ( !error )
      {
        FT_BitmapGlyph  bit = (FT_BitmapGlyph)image;


        my_draw_bitmap( bit->bitmap,
                        bit->left,
                        my_target_height - bit->top );

        /* increment pen position --                       */
        /* we don't have access to a slot structure,       */
        /* so we have to use advances from glyph structure */
        /* (which are in 16.16 fixed float format)         */
        pen.x += image.advance.x >> 10;
        pen.y += image.advance.y >> 10;

        FT_Done_Glyph( image );
      }
    }

There are a few changes compared to the original version of this code.

* We keep the original glyph images untouched; instead, we transform a copy.
* We perform clipping computations in order to avoid rendering and drawing
  glyphs that are not within our target surface.
* We always destroy the copy when calling `FT_Glyph_To_Bitmap` in order to get
  rid of the transformed scalable image.  Note that the image is not destroyed
  if the function returns an error code (which is why `FT_Done_Glyph` is only
    called within the compound statement).
* The translation of the glyph sequence to the start pen position is integrated
  into the call to `FT_Glyph_Transform` instead of `FT_Glyph_To_Bitmap`.

It is possible to call this function several times to render the string with
different angles, or even change the way `start` is computed in order to move
it to different place.

This code is the basis of the FreeType 2 demonstration program named
[`ftstring.c`](http://git.savannah.gnu.org/cgit/freetype/freetype2-demos.git/tree/src/ftstring.c).
It could be easily extended to perform advanced text layout or word-wrapping in
the first part, without changing the second one.

Note, however, that a normal implementation would use a glyph cache in order to
reduce memory needs.  For example, let us assume that our text string is
'FreeType'.  We would store three identical glyph images in our table for the
letter 'e', which isn't optimal (especially when you consider longer lines of
text, or even whole pages).

A FreeType demo program that shows how glyph caching can be implemented is
[`ftview.c`](http://git.savannah.gnu.org/cgit/freetype/freetype2-demos.git/tree/src/ftview.c).
In general, 'ftview' is the main program used by the FreeType developer team to
check the validity of loading, parsing, and rendering fonts.

### 6\. Accessing Metrics in Design Font Units, and Scaling Them

Scalable font formats usually store a single vectorial image, called an
_outline_, for each glyph in a face.  Each outline is defined in an abstract
grid called the _design space_, with coordinates expressed in _font units_.
When a glyph image is loaded, the font driver usually scales the outline to
device space according to the current character pixel size found in an
[`FT_Size`](../reference/ft2-base_interface#FT_Size) object.  The driver
may also modify the scaled outline in order to significantly improve its
appearance on a pixel-based surface (a process known as _hinting_ or
_grid-fitting_).

This section describes how design coordinates are scaled to the device space,
and how to read glyph outlines and metrics in font units.  This is important
for a number of things.

* 'True' WYSIWYG text layout.
* Accessing font content for conversion or analysis purposes.

#### a. Scaling Distances to Device Space

Design coordinates are scaled to the device space using a simple scaling
transformation whose coefficients are computed with the help of the _character
pixel size_.

    device_x = design_x * x_scale
    device_y = design_y * y_scale

    x_scale  = pixel_size_x / EM_size
    y_scale  = pixel_size_y / EM_size

Here, the value `EM_size` is font-specific and corresponds to the size of an
abstract square of the design space (called the _EM_), which is used by font
designers to create glyph images.  It is thus expressed in font units.  It is
also accessible directly for scalable font formats as `face->units_per_EM`.
You should check that a font face contains scalable glyph images by using the
`FT_IS_SCALABLE` macro, which returns true if appropriate.

When you call the function
[`FT_Set_Pixel_Sizes`](../reference/ft2-base_interface#FT_Set_Pixel_Sizes),
you are specifying the value of `pixel_size_x` and `pixel_size_y` FreeType
shall use.  The library will immediately compute the values of `x_scale` and
`y_scale`.

When you call the function
[`FT_Set_Char_Size`](../reference/ft2-base_interface#FT_Set_Char_Size),
you are specifying the character size in physical _points_, which is used,
along with the device's resolutions, to compute the character pixel size and
the corresponding scaling factors.

Note that after calling any of these two functions, you can access the values
of the character pixel size and scaling factors as fields of the
`face->size->metrics` structure.

`x_ppem`| The field name stands for 'x pixels per EM'; this is the horizontal size in integer pixels of the EM square, which also is the _horizontal character pixel size_, called `pixel_size_x` in the above example.
`y_ppem`| The field name stands for 'y pixels per EM'; this is the vertical size in integer pixels of the EM square, which also is the _vertical character pixel size_, called `pixel_size_y` in the above example.
`x_scale`| This is a 16.16 fixed-point scale to directly scale horizontal distances from design space to 1/64th of device pixels.
`y_scale`| This is a 16.16 fixed-point scale to directly scale vertical distances from design space to 1/64th of device pixels.

You can scale a distance expressed in font units to 26.6 pixel format directly
with the help of the
[`FT_MulFix`](../reference/ft2-computations#FT_MulFix) function.

    /* convert design distances to 1/64th of pixels */
    pixels_x = FT_MulFix( design_x, face->size->metrics.x_scale );
    pixels_y = FT_MulFix( design_y, face->size->metrics.y_scale );

Alternatively, you can also scale the value directly with more accuracy by
using doubles.

    FT_Size_Metrics*  metrics = &face->size->metrics; /* shortcut */
    double            pixels_x, pixels_y;
    double            em_size, x_scale, y_scale;


    /* compute floating point scale factors */
    em_size = 1.0 * face->units_per_EM;
    x_scale = metrics->x_ppem / em_size;
    y_scale = metrics->y_ppem / em_size;

    /* convert design distances to floating point pixels */
    pixels_x = design_x * x_scale;
    pixels_y = design_y * y_scale;

#### b. Accessing Design Metrics (Glyph & Global)

You can access glyph metrics in font units simply by specifying the
`FT_LOAD_NO_SCALE` bit flag in `FT_Load_Glyph` or `FT_Load_Char`.  The metrics
returned in `face->glyph->metrics` will all be in font units.

You can access unscaled kerning data using the `FT_KERNING_MODE_UNSCALED` mode.

Finally, a few global metrics are available directly in font units as fields of
the `FT_Face` handle, as described in [section 3](#section-3) of this part.

### Conclusion

This is the end of the second part of the FreeType tutorial.  You are now able
to access glyph metrics, manage glyph images, and render text much more
intelligently (kerning, measuring, transforming & caching); this is sufficient
knowledge to build a pretty decent text service on top of FreeType.

The demo programs in the 'ft2demos' bundle (especially 'ftview') are a kind of
reference implementation, and are a good resource to turn to for answers.  They
also show how to use additional features, such as the glyph stroker and cache.

## III. Examples

For completeness, here again a link to the [example](example1.c) used and
explained in the [first part of the tutorial](step1.html).

[Erik Möller](mailto:erik@timetrap.se) contributed a very nice C++ example that
shows renderer callbacks in action to draw a coloured glyph with a differently
coloured outline.  The source code can be found [here](example2.cpp).

[Another example](example3.cpp) demonstrates how to use FreeType's stand-alone
rasterizer, `ftraster.c`, both in B/W and 5-levels gray mode.  You need files
from FreeType version 2.3.10 or newer.

[Róbert Márki](mailto:gsmiko@gmail.com) contributed a small [Qt demonstration
program](example4.cpp) (together with its [qmake file](example4.pro)) that
shows both direct rendering with a callback and rendering with a buffer,
yielding the same result.  You need FreeType 2.4.3 or newer.
