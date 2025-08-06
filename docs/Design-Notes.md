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
