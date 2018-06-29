---
title: FreeType Development
sidebar: home_sidebar
permalink: development.html
folder: mydoc
toc: false
last_updated: June 29, 2018
nav: development.html
---

## Accessing the Source Code {#source-code}

There are several ways to access the source code for the FreeType
packages.

-   [Download a source archive](download.html)

    Use this to browse, compile, and use the source locally.

-   [Use the cgit page](http://git.savannah.gnu.org/cgit/freetype/)

    It will let you view our source code and the changes that occurred
    lately.

-   Use git access

    The complete FreeType repositories can be downloaded using git.
    Enter these commands for anonymous read-only access of the
    repositories:

    `git clone git://git.sv.nongnu.org/freetype/freetype2.git`  
    `git clone git://git.sv.nongnu.org/freetype/freetype2-demos.git`

    If you are behind a Firewall which disables port 9418, you might try
    the HTTP protocol:

    `git clone http://git.sv.nongnu.org/r/freetype/freetype2.git`  
    `git clone http://git.sv.nongnu.org/r/freetype/freetype2-demos.git`

    Please read

    <http://sv.nongnu.org/maintenance/UsingGit>

    for more details on the git setup of Savannah (for example, how to
    check out the git repositories with a CVS client, if necessary).

-   Use anonymous CVS access

    Other, no longer maintained FreeType repositories which exist for
    historical interest only can be downloaded through CVS and viewed
    [here](http://cvs.savannah.gnu.org/viewvc/?root=freetype). Use the
    following value for the `CVSROOT` environment variable (or the
    equivalent `-d` command line option of the cvs client):

    `:pserver:anonymous@cvs.sv.nongnu.org:/sources/freetype`

    When asked for a password, simply press Enter for `cvs login`. The
    module names are `freetype` (Freetype version 1.x, C code),
    `freetype1-contrib` (contributed programs), and `ftpascal` (Freetype
    version 1.x, Pascal code).

    The `freetype2` and `ft2demos` modules have been transformed to git
    (see above) and shouldn\'t be used anymore {#bug-report}

## Reporting a Bug

In case you find a bug which you think is related to FreeType, please
check the problematic font with one of our demo programs, for example
**ftview** or **ftstring**. In case the problem persists, go to the
[Savannah bug database for
FreeType](https://savannah.nongnu.org/bugs/?group=freetype) and check
whether the problem has been reported already. Otherwise, please submit
a bug report. It might be useful to use the git version (see above) of
FreeType for testing since we don\'t release FreeType very often.

## Language Bindings {#language-bindings}

For a couple of programming languages there exist wrappers around
FreeType. Note that some of them are quite old and probably no longer
maintained.

### [Caml](http://caml.inria.fr/)

Bindings are part of the
[CamlImages](http://cristal.inria.fr/camlimages/eng.html) package.

### [D](http://dlang.org)

Bindings have been submitted to
[Dsource.org](http://svn.dsource.org/projects/bindings/trunk/freetype/).

### [Factor](http://factorcode.org/)

This dynamic programming language comes with [FreeType
bindings](http://docs.factorcode.org/content/vocab-freetype.html).

### [Io](http://iolanguage.org/)

Here is a link to its [font
interface](http://iolanguage.org/scm/io/docs/reference/index.html#/Graphics/Font/Font).

### [Perl](http://www.perl.org)

More than a single binding package is available. Try
[here](http://search.cpan.org/search?m=all&q=freetype&s=1) for a search
on CPAN.

### [Python](https://www.python.org)

Bindings are available [here](https://github.com/rougier/freetype-py/).
The Python Imaging Library
([PIL](http://www.pythonware.com/products/pil/)) can be built to use
FreeType for font rendering, giving high-level access to fonts.

A more extensive set of Python bindings, designed to work together to
cover the main parts of the Linux typography stack, are

-   [python\_freetype](https://github.com/ldo/python_freetype) for
    FreeType,
-   [Qahirah](https://github.com/ldo/qahirah) for the Cairo graphics
    library,
-   [PyBidi](https://github.com/ldo/pybidi) for the FriBidi
    bidirectional-layout library,
-   [HarfPy](https://github.com/ldo/harfpy) for the HarfBuzz
    type-shaping library, and
-   [python\_fontconfig](https://github.com/ldo/python_fontconfig) for
    the Fontconfig font-matching library.

### [Ruby](https://www.ruby-lang.org)

See [ft2-ruby](https://rubygems.org/gems/ft2-ruby) for bindings.

## Support Tools {#support-tools}

One of the simplest way to compile FreeType is from the command line
using one of the following tools.

### GNU Make

FreeType comes with a sophisticated build system that is based on GNU
Make. This really means a set of Makefiles and sub-Makefiles that are
used to perform the following operations.

-   Detect the current operating system in order to select the
    appropriate default compiler settings for the build.

-   Select the settings corresponding to a given compiler for a given
    platform. For example, on Windows, the following compilers are
    supported: Visual C++, GCC, Borland C++, Watcom C++, Win32-LCC. On
    Unix, gcc, lcc and standard cc are also supported through a
    traditional `configure` script.

-   Build the list of FreeType modules automatically from the
    sub-directories present in the `src` directory.

-   Finally, build the library and its module as a static library or
    DLL, depending on the platform and compiler.

The build system is capable of supporting several compilers, on several
platforms. However, you **must** have a recent version of GNU Make
installed to use it. The build does *not work* with other `make` tools
(like BSD Make, NMake, etc.).

See the [download section](download.html) for binaries of GNU Make for
Windows.

### FT Jam

Unfortunately, the GNU Make-based build system described above is rather
complex due to various technical reasons, one of them being the *really*
weird syntax used in Makefiles. Since release 2.0.2, the FreeType
library can also be created with an alternative build tool named
**Jam**.

Briefly, Jam is a small, efficient, portable and open-source replacement
for `make` that is both a lot easier to use and more powerful.

-   Jam control files (named *Jamfiles*) are portable among platforms
    and compilers and thus do not need to be edited for each specific
    build (unlike the ugly `Makefile.in` trick used commonly on Unix).

-   The syntax of Jamfiles is simple, expressive and allows you to
    define your own functions.

-   Jam performs lots of nifty things for you, like automatic header
    dependencies computations.

-   Jam only does project builds, it is not a configuration tool and is
    trivially compatible with Autoconf on Unix.

Find more information on [the FT Jam page](jam/index.html).
