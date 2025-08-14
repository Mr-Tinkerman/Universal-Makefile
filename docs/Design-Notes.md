# Design Notes: a random place to dump my ideas

This document's purpose is to provide a place for me to jot ideas down.  This is
**NOT** a formal document and will likely be unformatted and messy.  Nothing in
this document is official or should be treated as such.  Any potential features
and/or details mentioned **SHOULD NOT** be treated as promises or guarentees.



### 8/5/2025 - Scott Tinkerman

I decided that trying to mentally keep tracking every detail was too dificult
and decided to make this document for my own sanity.  I've been trying to
establish a set of rules to determine the structure of this project.

Rules:

- Projects **CAN** contain multiple targets.
- Projects **MUST** be able to compile independently.
- All targets in a project share build configs **BUT** can define custom args. 
- Executable targets **CANNOT** depend on other executables or be depended on.
- Library targets **CAN** depend on other libraries but care must be taken to
  prevent circular dependencies.

Additionally, I must address the unification of the multi-project and
single-project makefiles.  Ignoring the linguistical flaw of having two,
"Universal" makefiles, it was always my intention to have a single file to copy
and paste around.  I separated them initially to help myself organise my ideas
but as I haven't started the multi-project makefile yet I think I'll unify them
once I start figuring out the multi-project side of things.  I do believe that a
proper definition of the two is in order.

- single-project: independently compilable project, can stand alone.
- multi-project: not compilable alone.  Exists to unify compilation settings
  across multiple projects and manage co-dependencies where including the deps
  source directly doesn't make sense (IE: libraries that don't directly relate
  to a project but may be developed alongside it).



### 8/13/2025 - Scott Tinkerman

I'm mentally trapped again so another entry it is.  I'm struggling to prioritize
what parts of the makefile to work on first so here's a list:

- Defining a better format for adding targets.
- Finalizing the build rules. (compiling only, no extras like compile_commands)
- Adding caching. (especially for saving the config)

After laying these out, my dumbass realized that the obvious answer is defining
the target format, as they must be defined to finish writing the build rules.
Also, I have reordered the list in the order of priority I've decided.  Caching
is not nearly as critical as being able to build a project so it should come
after the build rules.

As for defining the targets, I'm also still not sure what direction to take this
yet.  It seems like a trivial task at first but on further inspection it gets
more complicated.  For one, it must follow the rules listed in the previous
entry.  More problems come from two self imposed restrictions, that being:

- The user configuration settings must be as close to the top of the file as
  reasonably possible.  This makes it easier to find and edit the file as the
  end user.  A few lines above are okay but full functions are too big.  That's
  not a problem with an `include` but...
- The makefile must be usable as a single file and must not require other files
  to work.  Including other files are allowed if they are user extensions, such
  as a `custom-rules.mk` file, or as a generated file, like `cache.mk`.  This
  means we can't include built in functions with the `include` tag unless they
  are evaluated after their definition.



### 8/14/2025

Frankly, I went to get food, then started doom scrolling.  Because of this.  I
didn't finish my notes from before.  It's currently 12:30 AM and I've finally
returned to finish this.

With the restraints listed before, I am somewhat limited in my options for
managing targets.  There are two primary things that the targets need to have:

- Target Type
- Target Dependencies (On other targets)

...

I've been sitting here for about thirty minutes thinking about possible ways to
meet these conditions and I believe I've found my answer.  If I have a variable
for each type of target, I can define targets by adding the target id to them.

`TARGET_EXECUTABLES = EXAMPLE_EXE`
`TARGET_STATICLIBS = EXAMPLE_LIB`
`TARGET_SHAREDLIBS = EXAMPLE_DLL`
`TARGET_BINARYS = EXAMPLE_BIN`

For custom targets, you can do this

`TARGET_CUSTOM = EXAMPLE_CUSTOM`

but custom targets will need to register as a target type to correctly integrate
with the automagic system this makefile uses.

`SUPPORTED_TARGET_TYPES += CUSTOM`

There are three constraints that must exist to make this work.

- Target ids must be valid makefile variables.
- Duplicate target ids are strictly prohibited regardless of target type.
- There must always be at least one target defined.

Dependencies will be defined like

`TARGET_EXECUTABLES = EXAPP`
`TARGET_STATICLIBS = EXLIB1 EXLIB2`

`TARGETDEPS_EXAPP = EXLIB1 EXLIB2`

Additionally, I think it'd be a good idea to give the option to change the name
of the built target.

`TARGET_EXECUTABLE = EXAMPLE`

`TARGETNAME_EXAMPLE = example2`

Anyways, I'll work on it tomorrow (technically today but it's 1:45 and I need
sleep).  I have a root beer in the fridge and I'm definitely making a scuffed
dirty girl in the morning.

Scuffed Dirty Girl Recipe:

- Root Beer
- Heavy Cream
- Honey Whisky

Mix proportions to taste because I sure as hell don't measure, or even have the
numbers in the first place.
