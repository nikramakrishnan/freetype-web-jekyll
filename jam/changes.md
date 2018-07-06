---
title:  FT Jam Enhancement
sidebar: home_sidebar
permalink: jam/changes.html
folder: jam
# nav: jam.html
---

## Introduction {#intro}

[FT Jam](http://www.freetype.org/jam) is an enhanced version
of the [Jam](http://www.perforce.com/jam/jam.html) build tool. This
document lists the differences between these two programs. Note that
these changes come from the need of real-world cases that can\'t be
compiled correctly with classic Jam\'s limitations.

***Note that FT Jam is, and will always be, fully backwards compatible
with the regular Jam tool.*** It thus can be used as a *drop-in
replacement* for existing Jam users. If you find something that breaks a
classic Jamfile when used with FT Jam, please send a copy to the
[FreeType developers\' mailing list](mailto:freetype-devel@nongnu.org)
so that it can be fixed as soon as possible.

------------------------------------------------------------------------

## New Built-in Rules {#builtins}

FT Jam has introduced several new built-in rules.

### HDRMACRO -- *Header File Macro Inclusion* {#builtins-hdrmacro}

Jam is capable of scanning C source files to automatically determine
header file dependencies. It does so through the following scheme.

1.  First, it looks for preprocessor inclusion directives like
    `#include "myfile.h"` or `#include <myfile.h>`.

2.  It searches for the corresponding header files according to the
    values of several Jam variables like SEARCH, LOCATE, etc.

3.  When the header file is found, and hasn\'t been processed yet, it is
    added to the dependency tree and scanned recursively to determine
    its own dependencies.

ISO C allows macro expansion to occur in a `#include` directive, as long
as the resulting value is a correct header file name. The following is
thus valid, but is not parsed adequately by Jam.

```c
#define  MYFILE_H  "myfile.h"
#include MYFILE_H
```

FT Jam can parse such files normally by its **HDRMACRO** built-in rule
whose syntax is

```bash
HDRMACRO  filename ;
```

where `filename` is the name of a file that contains the *definitions*
of macros that is used in inclusion directives. When you invoke this
built-in, it does the following automatically.

1.  Scan *filename* for `#define` directives and filter those that do
    not define a potential filename (e.g., it discards macros with
    parameters).

2.  When a potential filename macro is encountered (i.e., when its
    definition expands to something like `"..."` or `<...>`, where `...`
    means 'anything'), it is recorded in a global dictionary with its
    value.

Later on, when FT Jam finds a macro inclusion directive, like
`#include MYFILE_H`, it searches its macro dictionary to see if it can
expand it. If so, it performs the expansion inline and updates the
dependency tree. Otherwise it simply ignores the line.

Note that you should always use the HDRMACRO built-in before other
dependency rules (like Cc or C++), in order to record the macro
definitions before header file scanning occurs

### SUBST -- *Regular Expression Replacement* {#builtins-subst}

This new built-in must be used ***as a function*** and performs regular
expression matching and replacement. Its syntax is

```
RESULT = [ SUBST  source  pattern  replacement ] ;
```

where *source* is the source string, *pattern* is a regular expression
pattern that is searched in the source string, and *replacement* is the
replacement for the first pattern match. Of course, *RESULT* is a
variable that contains the match\'s result.

Note that the character '\$' is used as an escape \$character, with \$1,
\$2, \$3, etc., corresponding to the first matched sub-expression, the
second one, the third one, etc.

Here is a small example. The following Jamfile fragment

```bash
XX_TGZ  = packagename-2.0.2.tar.gz ;
PATTERN = "([A-Za-z][A-Za-a0-9_]*)-(.*)\.tar\.gz" ;
XX_TBZ2 = [ SUBST $(XX_TGZ) "(.*)\.tar\.gz" "$1.tar.bz2" ] ;
XX_TAR  = [ SUBST $(XX_TGZ) "(.*)\.gz" "$1" ] ;
XX_NAME = [ SUBST $(XX_TGZ) $(PATTERN) "$1" ] ;
XX_VER  = [ SUBST $(XX_TGZ) $(PATTERN) "$2" ] ;

ECHO $XX_TBZ2 ;
ECHO $XX_TAR ;
ECHO $XX_NAME ;
ECHO $XX_VER ;
```

prints as

```
packagename-2.0.2.tar.bz2
packagename-2.0.2.tar
packagename
2.0.2
```

### *Indirect Rule Invocation* {#builtins-indirect}

Well, that is not exactly a new built-in, but is sufficiently close to
it to deserve this classification.

FT Jam allows you to use macro expansion to determine which rule to
invoke in a function call. For example, the following code fragment is
not valid in classic Jam but runs with FT Jam:

```
rule  MyFunc
{
return "Boo!" $(<) ;
}

FUNC = MyFunc ;
BOO  = [ $(FUNC) "Ahh!" ] ;
ECHO $(BOO) ;
```

Of course, it prints '`Boo! Ahh!`'. Note also that during debug output,
the expansion is dumped (e.g., 'MyFunc' and not '\$(FUNC)').

Note that this rule invocation method must be used carefully, since the
*whole variable expansion* is used to determine which rule to use,
including any spaces. The following thus fails.

```
rule  MyFunc
{
return "Boo!" $(<) ;
}

FUNC = MyFunc "Ahh!" ;
BOO  = [ $(FUNC) ] ;
ECHO $(BOO) ;
```

The reason is that there is no rule named '`MyFunc   Ahh!`'.

### FAIL\_EXPECTED -- *Action Result Inversion* {#builtins-fail_expected}

Do not use this built-in without a *very good reason*. Its purpose is to
invert the result of a given build action.

-   If the action didn\'t perform correctly (e.g., a file couldn\'t
    compile correctly), Jam will consider the target as built anyway and
    will keep updating dependents.

-   If the action succeeded (e.g., if the same file did compile
    correctly), Jam will treat it as an error, and stop immediately this
    branch of the dependency tree (without building dependents).

As you might probably guess, this built-in is only useful in very
specific cases (e.g., testing system features as in a configure script).
***You probably don\'t need it in your Jamfiles***, so better keep your
hands off of it.

It is actually an 'experimental' feature of FT Jam and could be renamed
or modified heavily in the future, so don\'t rely on it unless you are
in close contact with the FT Jam author(s).

------------------------------------------------------------------------

## Jambase Enhancements {#jambase}

The Jambase is a file containing all default rules and actions used by
the Jamfiles. It is written in normal Jam syntax and is compiled within
the 'jam' executable by default.

### New Windows and OS/2 Toolset Selection Scheme {#jambase-select}

In *classic Jam*, toolset selection on Windows and OS/2 is performed by
setting specific environment variables corresponding to the compiler you
want to use.

<center>
<table>
<tr>
    <td><b>Variable</b></td>
    <td><b>Toolset</b></td>
    <td><b>Value</b></td>
</tr>
<tr>
    <td>BCCROOT</td>
    <td>Borland C++</td>
    <td>installation path</td>
</tr>
<tr>
    <td>MSVC</td>
    <td>Visual C++ 16-bits</td>
    <td>installation path</td>
</tr>
<tr>
    <td>MSVCNT</td>
    <td>Visual C++ 32-bits</td>
    <td>installation path</td>
</tr>
</table>
</center>

Note that the first variable in the list that is detected is used by the
classic Jambase to determine which toolset to use. This is very annoying
when you need to switch frequently between toolsets on the same machine
(e.g., you have to *unset* a variable before setting the previous one,
and you have to retype an installation path on each switch).

The FT Jam Jambase provides a more flexible scheme that allows you to
switch between toolsets very easily, using one additional level of
indirection.

-   When the Jambase is first parsed, it looks for a specific
    environment variable named `JAM_TOOLSET`.

-   When `JAM_TOOLSET` is found, it is used to determine which toolset
    to use. Its value is *the name of another specific environment
    variable* that normally gives the toolset\'s installation path.

-   If it doesn\'t find `JAM_TOOLSET`, it defaults to the classic
    Jambase behaviour and looks for `BCCROOT`, `MSVC` and `MSVCNT` (in
    this order). If *none* of these variables are defined, it prints a
    message.

For example, here is how to use the Borland C++ compiler.

```bash
set BORLANDC=path\to\borland\install
set JAM_TOOLSET=BORLANDC
jam
```

And here is how to use the Visual C++ compiler.

```bash
set VISUALC=path\to\visualc\install
set JAM_TOOLSET=VISUALC
jam
```

You can also change the value of `JAM_TOOLSET` anytime you want to
switch between the toolsets automatically.

```bash
set BORLANDC=path\to\borland\install
set VISUALC=path\to\visualc\install

set JAM_TOOLSET=BORLANDC
jam
jam clean

set JAM_TOOLSET=VISUALC
jam
jam clean
```

The biggest advantage of this scheme is that you *only need to define
toolset-specific variables once*, then later use `JAM_TOOLSET` to switch
between them. This generally allows you to put all variable definitions
into a batch file and forget about their exact values. When using a
large number of toolsets, this can be really helpful.

### New Windows and OS/2 Toolsets Supported {#jambase-toolset}

New toolsets have been added to the FT Jam Jambase for Windows and OS/2.
The best way to see them is simply to invoke 'jam' when `JAM_TOOLSET`
isn\'t defined. This dumps a message containing the list of currently
supported toolsets for your platform.

On Windows, you get output like this.

```
Jam cannot be run because you didn't indicate which compilation toolset
to use. To do so, follow these simple instructions:

- define one of the following environment variable, with the
  appropriate value according to this list:

 Variable    Toolset                      Description

 BORLANDC    Borland C++                  BC++ install path
 VISUALC     Microsoft Visual C++         VC++ install path
 VISUALC16   Microsoft Visual C++ 16 bit  VC++ 16 bit install
 INTELC      Intel C/C++                  IC++ install path
 WATCOM      Watcom C/C++                 Watcom install path
 MINGW       MinGW (gcc)                  MinGW install path
 LCC         Win32-LCC                    LCC-Win32 install path

- define the JAM_TOOLSET environment variable with the *name*
  of the toolset variable you want to use.

e.g.:  set VISUALC=C:Visual6
       set JAM_TOOLSET=VISUALC
```

Here the output on OS/2.

```
Jam cannot be run because you didn't indicate which compilation toolset
to use. To do so, follow these simple instructions:

- define one of the following environment variable, with the
  appropriate value according to this list:

 Variable    Toolset                      Description

 WATCOM      Watcom C/C++                 Watcom install path
 EMX         EMX (gcc)                    EMX install path
 VISUALAGE   IBM Visual Age C/C++         VisualAge install path

- define the JAM_TOOLSET environment variable with the *name*
  of the toolset variable you want to use.

e.g.:  set WATCOM=C:\WATCOM
       set JAM_TOOLSET=WATCOM
```

------------------------------------------------------------------------

## Other Improvements {#other}

FT Jam also provides other important or minor improvements to the
original Jam sources.

### Runs on Windows 9x {#other-win9x}

The classic Jam is not capable of running on Windows 95 and 98 systems.
That is mainly because it relies on the Windows NT command line
processor named `cmd.exe`, while Win9x systems only come with the
infamous `command.com` which isn\'t capable of returning valid exit
codes for most commands.

FT Jam provides an improved execution backend that is capable of
detecting Win9x systems at runtime, and take special measures to deal
with their limitation.

To date, we haven\'t encountered a Jamfile that works on NT and fails on
Win9x with FT Jam. Please send these to the [FreeType developers\'
mailing list](mailto:freetype-devel@nongnu.org) in case you have one.

### Cosmetic Enhancements {#other-cosmetic}

There are also some cosmetic enhancements that might be useful.

-   FT Jam prints

    ```
    updating 1 target
    updating 2 targets
    ```

    instead of

    ```
    updating 1 target(s)
    updating 2 target(s)
    ```

-   Classic Jam 'wraps' debug output inelegantly when it is too nested,
    which can make debugging really hard. This has been fixed in FT Jam

------------------------------------------------------------------------

## About the Author {#about}

[David Turner](mailto:david@freetype.org) is best known as the author of
the open-source [FreeType](http://www.freetype.org) library. He
specializes in writing highly portable code for embedded systems.

The source code of FT Jam is stored on the Perforce Public Depot (peek
at `//guest/david_turner/jam/src`).
