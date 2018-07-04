---
layout: page
title: FreeType Design
theme: dark-green
sidebar: docs_sidebar
permalink: docs/design.html
nav: design
---

Copyright 1998--2000 David Turner ([david@freetype.org](mailto:david@freetype.org))<br>
Copyright 2000 The FreeType Development Team ([devel@freetype.org](mailto:devel@freetype.org))

## Introduction

This document provides details on the design and implementation of the FreeType
2 library.  Its goal is to allow developers to better understand the way how
FreeType 2 is organized, in order to let them extend, customize, and debug it.

Before anything else, it is important to understand the _purpose_ of this
library, i.e., why it has been written:

* It allows client applications to _access font files easily_, wherever they
  could be stored, and as independently of the font format as possible.

* Easy _retrieval of global font data_ most commonly found in normal font
  formats (i.e. global metrics, encoding/charmaps, etc.).

* It allows easy _retrieval of individual glyph data_ (metrics, images, name,
  anything else).

* _Access to font format-specific "features"_ whenever possible (e.g. SFNT
  tables, Multiple Masters, OpenType Layout tables, etc.).

Its design has also severely been influenced by the following requirements:

* _High portability_:  The library must be able to run on any kind of
  environment.  This requirement introduces a few drastic choices that are part
  of FreeType 2's low-level system interface.

* _Extendability_:  New features should be added with the least modifications
  in the library's code base.  This requirement induces an extremely simple
  design where nearly all operations are provided by modules.

* _Customization_:  It should be easy to build a version of the library that
  only contains the features needed by a specific project.  This really is
  important when you need to integrate it in a font server for embedded
  graphics libraries.

* _Compactness_ and _efficiency_:  The primary target for this library are
  embedded systems with low cpu and memory resources.

The rest of this document is divided in several sections.  First, a few
chapters will present the library's basic design as well as the objects/data
managed internally by FreeType 2.

A later section is then dedicated to library customization, relating such
topics as system-specific interfaces, how to write your own module and how to
tailor library initialization & compilation to your needs.

## I. Components and APIs

It's better to describe FreeType 2 as a collection of _components_.  Each one
of them is a more or less abstract part of the library that is in charge of one
specific task.  We will now explicit the connections and relationships between
them.

A first brief description of this system of components could be:

* Client applications typically call the FreeType 2 **high-level API**, whose
  functions are implemented in a single component called the _Base Layer_.

* Depending on the context or the task, the base layer then calls one or more
  _module_ components to perform the work.  In most cases, the client
  application doesn't need to know which module was called.

* The base layer also contains a set of routines that are used for generic
  things like memory allocation, list processing, i/o stream parsing,
  fixed-point computation, etc.  these functions can also be called by a module
  at any time, and they form what is called the **low-level base API**.

This is illustrated by the following graphics (note that component entry points
are represented as colored triangles):

<center>![Basic FreeType design](basic-design.png)</center>

Now, a few additional things must be added to complete this picture:

* Some parts of the base layer can be replaced for specific builds of the
  library, and can thus be considered as components themselves.  This is the
  case for the `ftsystem` component, which is in charge of implementing memory
  management & input stream access, as well as `ftinit`, which is in charge of
  library initialization (i.e.  implementing the `FT_Init_FreeType()`
  function).

* FreeType 2 comes also with a set of _optional components_, which can be used
  either as a convenience for client applications (e.g. the `ftglyph`
  component, used to provide a simple API to manage glyph images independently
  of their internal representation), or to access format-specific features
  (e.g. the `ftmm` component used to access and manage Multiple Masters data in
  Type 1 fonts).

* Finally, a module is capable of calling functions provided by another module.
  This is very useful to share code and tables between several font driver
  modules (for example, the `truetype` and `cff` modules both use the routines
  provided by the `sfnt` module).

Hence, a more complete picture would be:

<center>![Detailed FreeType design](detailed-design.png)</center>

Please take note of the following important points:

* An optional component can use either the high-level or base API.  This is the
  case of `ftglyph` in the above picture.

* Some optional components can use module-specific interfaces ignored by the
  base layer.  In the above example, `ftmm` directly accesses the Type 1 module
  to set/query data.

* A replaceable component can provide a function of the high-level API. For
  example, `ftinit` provides `FT_Init_FreeType()` to client applications.

## II. Public Objects and Classes

We will now explain the abstractions provided by FreeType 2 to client
applications to manage font files and data.  As you would normally expect,
these are implemented through objects/classes.

### 1\. Object Orientation in FreeType 2

Though written in ANSI C, the library employs a few techniques, inherited from
object-oriented programming, to make it easy to extend.  Hence, the following
conventions apply in the FreeType 2 source code:

1.  Each object type/class has a corresponding _structure type_ **and** a
    corresponding _structure pointer type_.  The latter is called the _handle
    type_ for the type/class.

    Consider that we need to manage objects of type "foo" in FreeType 2\.  We
    would define the following structure and handle types as follows:

        typedef struct FT_FooRec_*  FT_Foo;

        typedef struct  FT_FooRec_
        {
          // fields for the "foo" class
          ...
        } FT_FooRec;

    As a convention, handle types use simple but meaningful identifiers
    beginning with `FT_`, as in `FT_Foo`, while structures use the same name
    with a `Rec` suffix appended to it ("Rec" is short for "record").  _Note
    that each class type has a corresponding handle type_.

2.  Class derivation is achieved internally by wrapping base class structures
    into new ones.  As an example, we define a "foobar" class that is derived
    from "foo".  We would do something like:

        typedef struct FT_FooBarRec_*  FT_FooBar;

        typedef struct  FT_FooBarRec_
        {
          // the base "foo" class fields
          FT_FooRec  root;

          // fields proper to the "foobar" class
          ...
        } FT_FooBarRec;

    As you can see, we ensure that a "foobar" object is also a "foo" object by
    placing a `FT_FooRec` at the start of the `FT_FooBarRec` definition.  It is
    called **root** by convention.

    Note that a `FT_FooBar` handle also points to a "foo" object and can be
    typecasted to `FT_Foo`.  Similarly, when the library returns a `FT_Foo`
    handle to client applications, the object can be really implemented as a
    `FT_FooBar` or any derived class from "foo".

In the following sections of this chapter, we will refer to "the `FT_Foo`
class" to indicate the type of objects handled through `FT_Foo` pointers, be
they implemented as "foo" or "foobar".

### 2\. The `FT_Library` class

This type corresponds to a handle to a single instance of the library.  Note
that the corresponding structure `FT_LibraryRec` is not defined in public
header files, making client applications unable to access its internal fields.

The library object is the _parent_ of all other objects in FreeType 2\.  You
need to create a new library instance before doing anything else with the
library.  Similarly, destroying it will automatically destroy all its children
(i.e. faces and modules).

Typical client applications should call `FT_Init_FreeType()` in order to create
a new library object, ready to be used for further actions.

Another alternative is to create a fresh new library instance by calling the
function `FT_New_Library()`, defined in the `<freetype/ftmodule.h>` public
header file.  This function will however return an "empty" library instance
with no module registered in it.  You can "install" modules in the instance by
calling `FT_Add_Module()` manually.

Calling `FT_Init_FreeType()` is a lot more convenient, because this function
basically registers a set of default modules into each new library instance.
The way this list is accessed and/or computed is determined at build time, and
depends on the content of the `ftinit` component.  This process is explained in
details later in this document.

For now, one should consider that library objects are created with
`FT_Init_FreeType()`, and destroyed along with all children with
`FT_Done_FreeType()`.

### 3\. The `FT_Face` class

A face object corresponds to a single _font face_, i.e., a specific typeface
with a specific style.  For example, "Arial" and "Arial Italic" correspond to
two distinct faces.

A face object is normally created through `FT_New_Face()`.  This function takes
the following parameters: an `FT_Library` handle, a C file pathname used to
indicate which font file to open, an index used to decide which face to load
from the file (a single file may contain several faces in certain cases), and
the address of a `FT_Face` handle.  It returns an error code:

    FT_Error  FT_New_Face( FT_Library   library,
                           const char*  filepathname,
                           FT_Long      face_index,
                           FT_Face*     face ); 

In case of success, the function will return 0, and the handle pointed to by
the `face` parameter will be set to a non-NULL value.

Note that the face object contains several fields used to describe global font
data that can be accessed directly by client applications.  For example, the
total number of glyphs in the face, the face's family name, style name, the EM
size for scalable formats, etc.  For more details, look at the `FT_FaceRec`
definition in the FreeType 2 API Reference.

### 4\. The `FT_Size` class

Each `FT_Face` object _has_ one or more `FT_Size` objects.  A _size object_ is
used to store data specific to a given character width and height.  Each newly
created face object has one size, which is directly accessible as `face->size`.

The contents of a size object can be changed by calling either
`FT_Set_Pixel_Sizes()` or `FT_Set_Char_Size()`.

A new size object can be created with `FT_New_Size()`, and destroyed manually
with `FT_Done_Size()`.  Note that typical applications don't need to do this
normally: they tend to use the default size object provided with each
`FT_Face`.

The public fields of `FT_Size` objects are defined in a very small structure
named `FT_SizeRec`.  However, it is important to understand that some font
drivers define their own derivatives of `FT_Size` to store important internal
data that is re-computed each time the character size changes.  Most of the
time, these are size-specific _font hints_.

For example, the TrueType driver stores the scaled CVT table that results from
the execution of the "cvt" program in a `TT_Size` structure, while the Type 1
driver stores scaled global metrics (like blue zones) in a `T1_Size` object.
Don't worry if you don't understand the current paragraph; most of this stuff
is highly font format specific and doesn't need to be explained to client
developers :-)

### 5\. The `FT_GlyphSlot` class

The purpose of a glyph slot is to provide a place where glyph images can be
loaded one by one easily, independently of the glyph image format (bitmap,
vector outline, or anything else).

Ideally, once a glyph slot is created, any glyph image can be loaded into it
without additional memory allocation.  In practice, this is only possible with
certain formats like TrueType which explicitly provide data to compute a slot's
maximum size.

Another reason for glyph slots is that they are also used to hold
format-specific hints for a given glyphs as well as all other data necessary to
correctly load the glyph.

The base `FT_GlyphSlotRec` structure only presents glyph metrics and
images to client applications, while actual implementation may contain more
sophisticated data.

As an example, the TrueType-specific `TT_GlyphSlotRec` structure
contains additional fields to hold glyph-specific bytecode, transient outlines
used during the hinting process, and a few other things.

The Type 1-specific `T1_GlyphSlotRec` structure holds glyph hints during
glyph loading, as well as additional logic used to properly hint the glyphs
when a native Type 1 hinter is used.

Finally, each face object has a single glyph slot that is directly accessible
as `face->glyph`.
    
### 6\. The `FT_CharMap` class

The `FT_CharMap` type is used as a handle to character map objects, or
_charmaps_.  A charmap is simply some sort of table or dictionary which is used
to translate character codes in a given encoding into glyph indices for the
font.

A single face may contain several charmaps.  Each one of them corresponds to a
given character repertoire, like Unicode, Apple Roman, Windows codepages, and
other encodings.

Each `FT_CharMap` object contains a "platform" and an "encoding" field
used to identify precisely the character repertoire corresponding to it.

Each font format provides its own derivative of `FT_CharMapRec` and thus
needs to implement these objects.

### 7\. Objects relationships

The following diagram summarizes what we have just said regarding the public
objects managed by the library, as well as explicitly describes their
relationships

<center>![Simple library model](simple-model.png)</center>

Note that this picture will be updated at the end of the next chapter, related
to _internal objects_.

## III. Internal Objects and Classes

Let us have a look now at the _internal_ objects that FreeType 2 uses, i.e.,
those not directly available to client applications, and see how they fit into
the picture.

### 1\. Memory management

All memory management operations are performed through three specific routines
of the base layer, namely: `FT_Alloc()`, `FT_Realloc()`, and `FT_Free()`.  Each
one of these functions expects a `FT_Memory` handle as its first parameter.

The latter is a pointer to a simple object used to describe the current memory
pool/manager.  It contains a simple table of alloc/realloc/free functions.  A
memory manager is created at library initialization time by
`FT_Init_FreeType()`, calling the function `FT_New_Memory()` provided by the
`ftsystem` component.

By default, this manager uses the ANSI `malloc()`, `realloc()`, and `free()`
functions.  However, as `ftsystem` is a replaceable part of the base layer, a
specific build of the library could provide a different default memory manager.

Even with a default build, client applications are still able to provide their
own memory manager by not calling `FT_Init_FreeType()` but follow these simple
steps:

1. Create a new `FT_Memory` object by hand.  The definition of `FT_MemoryRec`
   is located in the public file `<freetype/ftsystem.h>`.

2. Call `FT_New_Library()` to create a new library instance using your custom
   memory manager.  This new library doesn't yet contain any registered
   modules.

3. Register the set of default modules by calling the function
   `FT_Add_Default_Modules()` provided by the `ftinit` component, or manually
   register your drivers by repeatedly calling `FT_Add_Module()`.

### 2\. Input streams

Font files are always read through `FT_Stream` objects.  The definition of
`FT_StreamRec` is located in the public file `<freetype/ftsystem.h>`, which
allows client developers to provide their own implementation of streams if they
wish so.

The function `FT_New_Face()` will always automatically create a new stream
object from the C pathname given as its second argument.  This is achieved by
calling the function `FT_New_Stream()` provided by the `ftsystem` component.
As the latter is replaceable, the implementation of streams may vary greatly
between platforms.

As an example, the default implementation of streams is located in the file
`src/base/ftsystem.c` and uses the ANSI `fopen()`, `fseek()`, and `fread()`
calls.  However, the Unix build of FreeType 2 provides an alternative
implementation that uses memory-mapped files, when available on the host
platform, resulting in a significant access speed-up.

FreeType distinguishes between memory-based and disk-based streams.  In the
first case, all data is directly accessed in memory (e.g.  ROM-based,
write-only static data and memory-mapped files), while in the second, portions
of the font files are read in chunks called _frames_, and temporarily buffered
similarly through typical seek/read operations.

The FreeType stream sub-system also implements extremely efficient algorithms
to very quickly load structures from font files while ensuring complete safety
in the case of a "broken file".

The function `FT_New_Memory_Face()` can be used to directly create/open a
`FT_Face` object from data that is readily available in memory (including
ROM-based fonts).

Finally, in the case where a custom input stream is needed, client applications
can use the function `FT_Open_Face()`, which can accept custom input streams.
This may be useful in the case of compressed or remote font files, or even
embedded font files that need to be extracted from certain documents.

Note that each face owns a single stream, which is also destroyed by
`FT_Done_Face()`.  Generally speaking, it is certainly _not_ a good idea to
keep numerous `FT_Face` objects opened.

### 3\. Modules

A FreeType 2 module is itself a piece of code.  However, the library creates a
single `FT_Module` object for each module that is registered when
`FT_Add_Module()` is called.

The definition of `FT_ModuleRec` is not publicly available to client
applications.  However, each _module type_ is described by a simple public
structure named `FT_Module_Class`, defined in `<freetype/ftmodule.h>`, and is
described later in this document:

You need a pointer to an `FT_Module_Class` structure when calling
`FT_Add_Module()`, whose declaration is:

    FT_Error  FT_Add_Module( FT_Library              library,
                             const FT_Module_Class*  clazz );

Calling this function will do the following:

* It will check whether the library already holds a module object corresponding
  to the same module name as the one found in `FT_Module_Class`.

* If this is the case, it will compare the module version number to see whether
  it is possible to _upgrade_ the module to a new version.  If the module
  class's version number is smaller than the already installed one, the
  function returns immediately.  Similarly, it checks that the version of
  FreeType 2 that is running is correct compared to the one required by the
  module.

* It creates a new `FT_Module` object, using data and flags of the module class
  to determine its byte size and how to properly initialize it.

* If a module initializer is present in the module class, it will be called to
  complete the module object's initialization.

* The new module is added to the library's list of "registered" modules.  In
  case of an upgrade, the previous module object is simply destroyed.

Note that this function doesn't return an `FT_Module` handle, given that module
objects are completely internal to the library (and client applications
shouldn't normally mess with them :-)

Finally, it is important to understand that FreeType 2 recognizes and manages
several kinds of modules.  These will be explained in more details later in
this document, but we will list for now the following types:

* _Renderer_ modules are used to convert native glyph images to
  bitmaps/pixmaps.  FreeType 2 comes with two renderer modules by default: one
  to generate monochrome bitmaps, the other to generate high-quality
  anti-aliased pixmaps.

* _Font driver_ modules are used to support one or more font formats.
  Typically, each font driver provides a specific implementation/derivative of
  `FT_Face`, `FT_Size`, `FT_GlyphSlot`, as well as `FT_CharMap`.

* _Helper_ modules are shared by several font drivers.  For example, the `sfnt`
  module parses and manages tables found in SFNT-based font formats; it is then
  used by both the TrueType and OpenType font drivers.

* Finally, the _auto-hinter_ module has a specific place in the library's
  design, as its role is to process vectorial glyph outlines, independently of
  their native font format, to produce optimal results at small pixel sizes.

Note that every `FT_Face` object is _owned_ by the corresponding font driver,
depending on the original font file's format.  This means that all face objects
are destroyed when a module is removed/unregistered from a library instance
(typically by calling the `FT_Remove_Module()` function).

_Because of this, you should always take care that no `FT_Face` object is
opened when you upgrade or remove a module from a library, as this could cause
unexpected object deletion!_

### 4\. Libraries

We now come back to our well-known `FT_Library` object.  From what have been
said before, we already know that a library instance owns at least the
following:

* A memory manager object (`FT_Memory`), used for all allocation/releases
  within the instance.

* A list of `FT_Module` objects, corresponding to the "installed" or
  "registered" modules of the instance.  This list can be changed at any time
  through `FT_Add_Module()` and `FT_Remove_Module()`.

* Remember that face objects are owner by font drivers that are themselves
  modules owned by the library.

There is however another object owned by the library instance that hasn't been
described yet: the _raster pool_.

The _raster pool_ is simply a block of memory of fixed size that is used
internally as a "scratch area" for various memory-hungry transient operations,
avoiding memory allocation.  For example, it is used by each renderer when
converting a vectorial glyph outline into a bitmap (actually, that's where its
name comes from :-).

The size of the raster pool is fixed at initialisation time (it defaults to
16kByte) and cannot be changed at run-time (though we could fix this if there
is a real need for that).

When a transient operation needs more memory than the pool's size, it can
decide to either allocate a heap block as an exceptional condition, or
sub-divide recursively the task to perform in order to never exceed the pool's
threshold.

This extremely memory-conservative behaviour is certainly one of the keys to
FreeType's performance in certain areas (most importantly in glyph
rendering/scanline-conversion).

### 5\. Summary

Finally, the following picture illustrates what has been said in this section,
as well as the previous, by presenting the complete object graph of FreeType
2's base design:

<center>![Complete library model](library-model.png)</center>

## IV. Module Classes

We will now try to explain more precisely the _types_ of modules that FreeType
2 is capable of managing.  Note that each one of them is described with more
details in the following chapters of this document.

* _Renderer_ modules are used to manage scalable glyph images.  This means
  _transforming_ them, computing their _bounding box_, and _converting_ them to
  either _monochrome_ or _anti-aliased_ bitmaps.

  Note that FreeType 2 is capable of dealing with _any_ kind of glyph images,
  as long as a renderer module is provided for it.  The library comes by
  default with two renderers:

  Renderer |Description
  ---------|-----------
  `raster` | Supports the conversion of vectorial outlines (described by a `FT_Outline` object) to _monochrome_ bitmaps.
  `smooth` | Supports the conversion of the same outlines to high-quality _anti-aliased_ pixmaps (using 256 levels of gray).  Note that this renderer also supports direct span generation.

* _Font driver_ modules are used to support one or more specific font format.
  By default, FreeType 2 comes with the following font drivers:

  Driver     |Description
  -----------|-----------
  `truetype` | Supports TrueType font files.
  `type1`    | Supports Postscript Type 1 fonts, both in binary (`.pfb`) or ASCII (`.pfa`) formats, including Multiple Master fonts.
  `cid`      | Supports Postscript CID-keyed fonts.
  `cff`      | Supports OpenType, CFF as well as CEF fonts (CEF is a derivative of CFF used by Adobe in its SVG viewer).
  `winfonts` | Supports Windows bitmap fonts (i.e. `.fon` and `.fnt`).

  Note that font drivers can support bitmapped or scalable glyph images.  A
  given font driver that supports Bézier outlines through `FT_Outline` can also
  provide its own hinter, or rely on FreeType's `autohinter` module.

* _Helper_ modules are used to hold shared code that is often used by several
  font drivers, or even other modules.  Here are the default helpers:

  Helper     |Description
  -----------|-----------
  `sfnt`     | Used to support font formats based on the `SFNT` storage scheme: TrueType & OpenType fonts as well as other variants (like TrueType fonts that only contain embedded bitmaps)
  `psnames`  | Used to provide various useful functions related to glyph names ordering and Postscript encodings/charsets.  For example, this module is capable of automatically synthetizing a Unicode charmap from a Type 1 glyph name dictionary.
  `psaux`    | Used to provide various useful functions related to Type 1 charstring decoding, as this "feature" is needed by the `type1`, `cid`, and `cff` drivers.

* Finally, the _autohinter_ module has a specific role in FreeType 2, as it can
  be used automatically during glyph loading to process individual glyph
  outlines when a font driver doesn't provide its own hinting engine.

  This module's purpose and design is also heavily described on the FreeType
  web site.

We will now study how modules are described, then managed by the library.

### 1\. The `FT_Module_Class` structure

As described later in this document, library initialization is performed by
calling the `FT_Init_FreeType()` function.  The latter is in charge of creating
a new "empty" `FT_Library` object, then register each "default" module by
repeatedly calling the `FT_Add_Module()` function.

Similarly, client applications can call `FT_Add_Module()` any time they wish in
order to register a new module in the library.  Let us take a look at this
function's declaration:

    extern FT_Error  FT_Add_Module( FT_Library              library,
                                    const FT_Module_Class*  clazz );

As one can see, this function expects a handle to a library object, as well as
a pointer to a `FT_Module_Class` structure.  It returns an error code.  In case
of success, a new module object is created and added to the library.  Note by
the way that the module isn't returned directly by the call!

Here the definition of `FT_Module_Class`, with some explanation.  The following
code is taken from `<freetype/ftmodule.h>`:

    typedef struct  FT_Module_Class_
    {
      FT_ULong               module_flags;
      FT_Int                 module_size;
      const FT_String*       module_name;
      FT_Fixed               module_version;
      FT_Fixed               module_requires;
    
      const void*            module_interface;
    
      FT_Module_Constructor  module_init;
      FT_Module_Destructor   module_done;
      FT_Module_Requester    get_interface;
    
    } FT_Module_Class;

A description of its fields:

Field          | Description
---------------|------------
`module_flags` | A set of bit flags used to describe the module's category.  Valid values are:
               | `ft_module_font_driver` if the module is a font driver
               | `ft_module_renderer` if the module is a renderer
               | `ft_module_hinter` if the module is an auto-hinter
               | `ft_module_driver_scalable` if the module is a font driver supporting scalable glyph formats
               | `ft_module_driver_no_outlines` if the module is a font driver supporting scalable glyph formats that _cannot_ be described by an `FT_Outline` object
               | `ft_module_driver_has_hinter` if the module is a font driver that provides its own hinting scheme/algorithm
`module_size`  | An integer that gives the size in _bytes_ of a given module object.  This should _never_ be less than `sizeof(FT_ModuleRec)`, but can be more if the module needs to sub-class the base `FT_ModuleRec` class.
`module_name`  | The module's internal name, coded as a simple ASCII C string.  There can't be two modules with the same name registered in a given `FT_Library` object.  However, `FT_Add_Module()` uses the `module_version` field to detect module upgrades and perform them cleanly, even at run-time.
`module_version`| A 16.16 fixed-point number giving the module's major and minor version numbers.  It is used to determine whether a module needs to be upgraded when calling `FT_Add_Module()`.
`module_requires` | A 16.16 fixed-point number giving the version of FreeType 2 that is required to install this module.  The default value is 0x20000 for FreeType version  2.0
`module_interface`| Most modules support one or more "interfaces", i.e. tables of function pointers.  This field is used to point to the module's main interface, if there is one.  It is a short-cut that prevents users of the module to call `get_interface()` each time they need to access one of the object's common entry points. Note that is is optional, and can be set to NULL.  Other interfaces can also be accessed through the `get_interface()` field.
`module_init` | A pointer to a function used to initialize the fields of a fresh new `FT_Module` object.  It is called _after_ the module's base fields have been set by the library, and is generally used to initialize the fields of `FT_ModuleRec` subclasses. Most module classes set it to NULL to indicate that no extra initialization is necessary.
`module_done` | A pointer to a function used to finalize the fields of a given `FT_Module` object.  Note that it is called _before_ the library unsets the module's base fields, and is generally used to finalize the fields of `FT_ModuleRec` subclasses. Most module classes set it to NULL to indicate that no extra finalization is necessary
`get_interface` | A pointer to a function used to request the address of a given module interface.  Set it to NULL if you don't need to supportadditional interfaces but the main one.

### 2\. The `FT_Module` type

The `FT_Module` type is a handle (i.e. a pointer) to a given module
object/instance, whose base structure is given by the internal `FT_ModuleRec`
type.  We will intentionally _not_ describe this structure here, as there is no
point to look so far into the library's design.

When `FT_Add_Module` is called, it first allocates a new module instance, using
the `module_size` class field to determine its byte size.  The function
initializes the root `FT_ModuleRec` field, then calls the class-specific
initializer `module_init` when this field is not set to NULL.

Note that the library defines several sub-classes of `FT_ModuleRec`, which are,
as you could have guessed:

* `FT_Renderer` for renderer modules

* `FT_Driver` for font driver modules

* `FT_AutoHinter` for the auto-hinter

Helper modules use the base `FT_ModuleRec` type.  We will describe these
classes in the next chapters.

## To be continued...
