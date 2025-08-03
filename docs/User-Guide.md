# Universal Makefile User Guide

- [Overview](#overview)
- [Features](#features)
- [Variables](#variables)

This project is a work in progress.  [Forgive me please](https://riskofrain2.wiki.gg/images/7/78/Forgive_Me_Please.png?723ede)



## Overview

As the name of this project suggests, the two (2) Makefiles provided are
intended to be usable in any project, with minimal user setup required. At a
minimum, these Makefiles should support standard and embedded projects. For a
full list of supported features, [see the list below.](#features)

It is important to understand that the multi-project Makefile is intended to be
the root Makefile and expects the projects in each subdirectory to contain a
compatible version of the single-project Makefile.  If the multi-project
Makefile references a project that doesn't exist or contain a Makefile, it will
automatically create a template project with a valid single-project Makefile.

Before we get too far, let's get familiar with the layout of this document.  All
chapters are going to have their contents separated based on which Makefile is
expected to be used.  For example:

### Shared

> Content that applies to both single/multi-project Makefiles.


### Single Project Environment

> Content that applies exclusively to the single-project Makefile.


### Multiple Project Environment

> Content that applies exclusively to the multi-project Makefile.


I've decided to separate this guide like this to make it easy to locate the
relevant information given the nature of the desired knowledge.



## Features

***Disclaimer:** Some features listed here may not be implemented currently, but
will either be added before the first "official" release or removed from this
document prior to release.  Be sure to check this repo's releases.*


### Shared

- Automatic host system detection.  Supports Windows, Linux, and MacOS, on
  architectures x86, x86_64, ARM32, and ARM64.
- Support for generating "compile_commands.json".
- VSCode detection and integration.


### Single

- Ability to list sources by directories non-recursively.  No more listing each
  file explicitly, individually, one at a time... controversial I know.
  (Yes you can still list files explicitly... if you must)
- Source directories and files can be included on a per configuration basis.
- Sources can be excluded explicitly, also on a per configuration basis.



## Variables

#### ***Usage***
When setting the value of a variable using another, it's important to use `=`
instead of `:=` so that the substitution applies correctly.  

```Makefile
# Good, evaluates to bin/Linux (On Linux of course)
BINARY_DIR = bin/$(HOST_OS)

# Bad, evaluates to bin/
BINARY_DIR := bin/$(HOST_OS)
```

This rule doesn't *technically* apply unless other variables need to be
referenced, but it is recommended to follow convention for consistency.  For
those concerned about optimization, most variables will be redefined as `VARNAME
:= $(VARNAME)` after formatting and evaluation and before any rules are used.


#### ***Notation***
Variable names may contain variations, which are wrapped in code blocks.  Text
surrounded by, `[]` is optional.  The bracket pair, `{}` is used to define a
list, usually appearing as, `{,opt1,opt2}`.  In addition, other variables can be
combined with the make notation, `$(VARNAME)` such as `CFLAGS_$(CONFIG)`.  All
variations of variables can be used simultaneously unless otherwise specified.
Different types of variations can be used together.  See below for examples.

Examples:
`[]`:
- BASE`[_OPTSUFFIX]` -> `BASE`, `BASE_OPTSUFFIX`
- `[OPTPREFIX_]`BASE -> `BASE`, `OPTPREFIX_BASE`
- BASE_`[OPTMID_]`NAME -> `BASE_NAME`, `BASE_OPTMID_NAME`
- `[PRE_]`BASE`[_SUF]` -> `BASE`, `PRE_BASE`, `BASE_SUF`, `PRE_BASE_SUF`

`{}`:
- BASE`{1,2,3}` -> `BASE1`, `BASE2`, `BASE3`
- BASE`{,1}` -> `BASE`, `BASE1`
- `{C1,C2}`_BASE -> `C1_BASE`, `C2_BASE`
- BASE`{,_{N,P}{1,2}}` -> `BASE`, `BASE_N1`, `BASE_N2`, `BASE_P1`, `BASE_P2`

`$()`:
- BASE_`$(SUFFIX)`
  - `SUFFIX = S1` -> `BASE_S1`
  - `SUFFIX = S2` -> `BASE_S2`

`[]`, `{}`, `$()` Examples:
- `CFLAGS[_$(CONFIG)]` -> `CFLAGS`
  - `CONFIG = Release` -> `CFLAGS_Release`
  - `CONFIG = Debug` -> `CFLAGS_Debug`


#### ***Command Variables***
Commands such as `$(AR)` or `$(CC)` are automatically formatted to include the
`$(TOOL_PATH)`, `$(TOOLCHAIN_PREFIX)`, and `$(TOOLCHAIN_SUFFIX)`.  For example,
if the command `$(FLASH)` is set to `mcuflash -f 8mb`, then the final command
might look something like `path/to/tools/prefix-mcuflash-suffix -f 8mb`.  If
this behavior is not desired, you can surround the command in quotes, which will
be removed before being used (Ex. `"mcuflash -f 8mb"` -> `mcuflash -f 8mb`).


### Single

**TARGET_PLATFORM`[_$(CONFIG)]`:**
  *The intended platform the target is expected to be built for.  Used internally
  to perform platform specific actions, such as file extension substitution.  If
  empty, the host platform is implied*

  *Default value: ` `*

**TARGET_FILE`[_$(CONFIG)]`:**
  *The file to be built by Make.  Omitting the extension tells this makefile to
  interpret the file type by the `$(TARGET_PLATFORM)`, or throws an error if the
  file type can't be determined.  If the extension exists, it may be replaced
  with a different extension to match the `$(TARGET_PLATFORM)` (Ex. example.elf
  -> example.exe).  If no suitable substitution is found, the extension will be
  left as is.*

**BINARY_DIR:**
  *Directory to output the program to.  This includes ELFs and raw binaries, as
  well as libraries and map files.*

**BUILD_DIR:**
  *Directory where intermediate outputs are saved, object files for example.*

**LIBRARY_DIR:**
  Directory to search for the project's libraries.

**PUBLIC_API_DIR:**
  *Directory containing headers intended to provide a public API, commonly known
  as the `include/` folder.*

**`[EXCLUDE_]`SOURCE_DIRS`[_$(CONFIG)]`:**
  *List of directories containing source code.  Source files will be included
  non-recursively from each directory in the list.  If you wish to include files
  individually, you can use the `$(SOURCE_FILES)` variable.  You can also
  exclude unwanted files with the `$(EXCLUDE_SOURCE_FILES)` variable.*

**`[EXCLUDE_]`SOURCE_FILES`[_$(CONFIG)]`:**
  *List of source files to be compiled or explicitly removed from compilation
  when prefixed with `EXCLUDE_`.*

**CONFIG:**
  *Variable set to choose the current configuration to build.  If empty, it will
  be set to the value of `$(LAST_CONFIG)` if it exists, or errors otherwise.*

  *Default value: `$(LAST_CONFIG)`*

**LAST_CONFIG:**
  *Saved to cache.mk when make is ran and `$(CONFIG)` is set.  If `$(CONFIG)` is
  empty, it will be set to this value instead.*

**AR`[_$(CONFIG)]`:**
  *[Command](#command-variables) to generate archives, or better known as static
  libraries.*

  *Default value: `ar`*

**AS`[_$(CONFIG)]`:**
  *[Command](#command-variables) to compile assembly code.*

  *Default value: `as`*

**CC`[_$(CONFIG)]`:**
  *[Command](#command-variables) to compile C code.*

  *Default value: `cc`*

**CXX`[_$(CONFIG)]`:**
  *[Command](#command-variables) to compile C++ code.*

  *Default value: `g++`*

**PP`[_$(CONFIG)]`:**
  *[Command](#command-variables) to run the preprocessor.  Set this variable
  instead of `$(CPP)`*

  *Default value: ` `*

**LD`[_$(CONFIG)]`:**
  *[Command](#command-variables) to link the program.*

  *Default value: ` `*

**MKDIR`[_$(CONFIG)]`:**
  *[Command](#command-variables) to make directories.*

  *Default value: `"$(TOOL_PATH)mkdir -p"`*

**COPY`[_$(CONFIG)]`:**
  *[Command](#command-variables) to copy files from source to destination.*

  *Default value: `"$(TOOL_PATH)cp"`*

**DEL`[_$(CONFIG)]`:**
  *[Command](#command-variables) to delete files.  Please use caution when using
  this command.  Set this variable instead of `$(RM)`*

  *Default value: `"$(TOOL_PATH)$(RM)"`*

**OBJCOPY`[_$(CONFIG)]`:**
  *[Command](#command-variables) to convert a file formatted as an application,
  such as an ELF, to a raw binary file, stripping the extra ELF meta-data.  This
  command is most useful in embedded projects.*

  *Default value: `"$(TOOL_PATH)objcopy"`*

**MONITOR`[_$(CONFIG)]`:**
  *[Command](#command-variables) to connect to an external output stream,
  typically through serial via a COM port.  This command is intended to be used
  in an embedded environment.*

  *Default value: ` `*

**FLASH`[_$(CONFIG)]`:**
  *[Command](#command-variables) to flash the program's binary to an external
  device, typically through serial via a COM port.  This command is intended to
  be used in an embedded environment.*

  *Default value: ` `*

**AR_FLAGS`[_$(CONFIG)]`:**
  *Flags for the **archiver (static library generator), `$(AR)`***

**AS_FLAGS`[_$(CONFIG)]`:**
  *Flags for the **Assembler, `$(AS)`***

**C_FLAGS`[_$(CONFIG)]`:**
  *Flags for the **C compiler, `$(CC)`***

**CXX_FLAGS`[_$(CONFIG)]`:**
  *Flags for the **C++ compiler, `$(CXX)`***

**PP_FLAGS`[_$(CONFIG)]`:**
  *Flags for the **preprocessor, `$(PP)`***

**LD_FLAGS`[_$(CONFIG)]`:**
  *Flags for the **linker, `$(LD)`***

**OBJCOPY_FLAGS`[_$(CONFIG)]`:**
  *Flags for the **application (ELF/EXE) to binary (BIN) converter, `$(OBJCOPY)`***
**AR_FLAGS`[_$(CONFIG)]`:**
  *Flags for the **archiver (static library generator), `$(AR)`***

**AS_FLAGS`[_$(CONFIG)]`:**
  *Flags for the **Assembler, `$(AS)`***

**C_FLAGS`[_$(CONFIG)]`:**
  *Flags for the **C compiler, `$(CC)`***

**CXX_FLAGS`[_$(CONFIG)]`:**
  *Flags for the **C++ compiler, `$(CXX)`***

**PP_FLAGS`[_$(CONFIG)]`:**
  *Flags for the **preprocessor, `$(PP)`***

**LD_FLAGS`[_$(CONFIG)]`:**
  *Flags for the **linker, `$(LD)`***

**OBJCOPY_FLAGS`[_$(CONFIG)]`:**
  *Flags for the **application (ELF/EXE) to binary (BIN) converter, `$(OBJCOPY)`***

**MONITOR_FLAGS`[_$(CONFIG)]`:**
  *Flags for **monitoring serial output, `$(MONITOR)`***

**FLASH_FLAGS`[_$(CONFIG)]`:**
  *Flags for **flashing a binary file (BIN), `$(FLASH)`***

**TOOL_PATH`[_$(CONFIG)]`:**
  *Custom path to search for tools and [Commands](#command-variables).*

  *Default value: ` `*

**TOOLCHAIN_PREFIX`[_$(CONFIG)]`:**
  *Prefix to add to formatted [Commands](#command-variables).*

  *Default value: ` `*

**TOOLCHAIN_SUFFIX`[_$(CONFIG)]`:**
  *Suffix to append to formatted [Commands](#command-variables).*

  *Default value: ` `*
