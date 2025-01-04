######################
ELF Object File Format
######################

This document describes the
object file format, called ELF (Executable and Linking Format).

Chapter 1 presents an overview of the file format
and its data representation.

Chapters 2–6 describe the file header and the structures that
pertain to building programs.

Chapters 7–9 describe additional structures that are
necessary to execute a program.

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
