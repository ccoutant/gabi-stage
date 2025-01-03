######################
ELF Object File Format
######################

This document describes the
object file format, called ELF (Executable and Linking Format).
There are three main types of object files.

* A *relocatable file*
  holds code and data suitable for linking
  with other object files to create an executable
  or a shared object file.

* An *executable file*
  holds a program suitable for execution;
  the file specifies how the system (e.g., ``exec()``)
  creates a program’s process image.

* A *shared object file*
  holds code and data suitable for linking in two contexts.
  First, the link editor processes the shared object file with other relocatable
  and shared object files to create another object file.
  Second, the dynamic linker combines it with an executable file and other
  shared objects to create a process image.

Created by the assembler and link editor, object files are binary
representations of programs intended to be executed directly on
a processor.  Programs that require other abstract machines, such
as shell scripts, are excluded.

Sections 2–6 focus on the file format
and how it pertains to building programs.
Sections 7–9 also describe parts of the object file,
concentrating on the information necessary to execute a program.

.. toctree::
   :maxdepth: 2
   :numbered:

   01-intro
   02-eheader
   03-sheader
   04-strtab
   05-symtab
   06-reloc
   07-loading-intro
   08-pheader
   09-dynamic

.. toctree::
   :maxdepth: 1

   a-emachine
   b-osabi
   c-history
