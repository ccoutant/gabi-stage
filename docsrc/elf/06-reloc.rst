.. _Relocation:

**********
Relocation
**********

Relocation is the process of connecting symbolic references
with symbolic definitions.
For example, when a program calls a function, the associated call
instruction must transfer control to the proper destination address
at execution.
Relocatable files must have relocation entries,
which describe how to modify the section contents, thus allowing
executable and shared object files to hold
the right information for a process’s program image.

Executable files may also have relocation entries,
which are necessary when the code has unbound references
to a shared object, or when the code is position-independent.

Most relocations are *symbolic,* computing the value of an
expression involving a symbol and an offset (called the *addend*),
applying the result to a location in the object code.
A *relocation type* encodes an *operation* (how the expression is computed)
and a *format* (how the result is to be applied at the location).

In executable files, some relocations are *relative.*
A relative relocation marks a location that holds a 32-bit or 64-bit address
that must be relocated when a program segment is loaded at a runtime
address that is different from its link-time address.
These relocations do not require a symbol or an addend.

Relocation Entry
================

Relocation entries have the following formats.

.. code-block:: c
   :caption: Relocation Entries

   typedef struct {
       Elf32_Addr   r_offset;
       Elf32_Word   r_info;
   } Elf32_Rel;

   typedef struct {
       Elf32_Addr   r_offset;
       Elf32_Word   r_info;
       Elf32_Sword  r_addend;
   } Elf32_Rela;

   typedef struct {
       Elf64_Addr   r_offset;
       Elf64_Xword  r_info;
   } Elf64_Rel;

   typedef struct {
       Elf64_Addr   r_offset;
       Elf64_Xword  r_info;
       Elf64_Sxword r_addend;
   } Elf64_Rela;

``r_offset``
    This member gives the location at which to apply the
    relocation action.
    For a relocatable file,
    the value is the byte offset from the beginning of the section
    to the storage unit affected by the relocation.
    For an executable file or a shared object,
    the value is the virtual address
    of the storage unit affected by the relocation.

``r_info``
    This member gives both the symbol table index with respect to which
    the relocation must be made, and the type of relocation to apply.
    For example, a call instruction’s relocation entry
    would hold the symbol table index of the function being called.
    If the index is ``STN_UNDEF``,
    the undefined symbol index,
    the relocation uses 0 as the “symbol value”.
    Relocation types are processor-specific;
    descriptions of their behavior appear in the psABI supplement.
    When the text below refers to a relocation entry’s
    relocation type or symbol table index, it means the result of applying
    ``ELF32_R_TYPE`` (or ``ELF64_R_TYPE``\ ) or ``ELF32_R_SYM`` (or ``ELF64_R_SYM``\ ),
    respectively, to the entry’s ``r_info`` member.

    .. code-block:: c

       #define ELF32_R_SYM(i)     ((i)>>8)
       #define ELF32_R_TYPE(i)    ((unsigned char)(i))
       #define ELF32_R_INFO(s,t)  (((s)<<8)+(unsigned char)(t))

       #define ELF64_R_SYM(i)     ((i)>>32)
       #define ELF64_R_TYPE(i)    ((i)&0xffffffffL)
       #define ELF64_R_INFO(s,t)  (((s)<<32)+((t)&0xffffffffL))

``r_addend``
    This member specifies a constant addend used to
    compute the value to be stored into the relocatable field.

As specified previously, only
``Elf32_Rela`` and ``Elf64_Rela``
entries contain an explicit addend.
Entries of type ``Elf32_Rel`` and ``Elf64_Rel``
store an implicit addend in the location to be modified.
Depending on the processor architecture, one form or the other
might be necessary or more convenient.
Consequently, an implementation for a particular machine
may use one form exclusively or either form depending on context.

A relocation section references two other sections:
a symbol table and a section to modify.
The section header’s ``sh_info`` and ``sh_link`` members,
described in :numref:`Section {number}, {name} <shlink-shinfo-fields>`,
specify these relationships.
Relocation entries for different object files have
slightly different interpretations for the
``r_offset`` member.

* In relocatable files, ``r_offset``
  holds a section offset.
  The relocation section itself describes how to
  modify another section in the file; relocation offsets
  designate a storage unit within the second section.

* In executable and shared object files,
  ``r_offset`` holds a virtual address.
  To make these files’ relocation entries more useful
  for the dynamic linker, the section offset (file interpretation)
  gives way to a virtual address (memory interpretation).

Although the interpretation of ``r_offset``
changes for different object files to
allow efficient access by the relevant programs,
the relocation types’ meanings stay the same.

The typical application of an ELF relocation is to determine the
referenced symbol value, extract the addend (either from the
field to be relocated or from the addend field contained in
the relocation record, as appropriate for the type of relocation
record), apply the expression implied by the relocation type
to the symbol and addend, extract the desired part of the expression
result, and place it in the field to be relocated.

If multiple *consecutive* relocation records are applied
to the same relocation location (\ ``r_offset``\ ),
they are *composed* instead
of being applied independently, as described above.
By *consecutive*, we mean that the relocation records are
contiguous within a single relocation section.  By *composed*,
we mean that the standard application described above is modified
as follows:

* In all but the last relocation operation of a composed sequence,
  the result of the relocation expression is retained, rather
  than having part extracted and placed in the relocated field.
  The result is retained at full pointer precision of the
  applicable psABI supplement.

* In all but the first relocation operation of a composed sequence,
  the addend used is the retained result of the previous relocation
  operation, rather than that implied by the relocation type.

Note that a consequence of the above rules is that the location specified
by a relocation type is relevant for the
first element of a composed sequence (and then only for relocation
records that do not contain an explicit addend field) and for the
last element, where the location determines where the relocated value
will be placed.  For all other relocation operands in a composed
sequence, the location specified is ignored.

A psABI supplement may specify individual relocation types
that always stop a composition sequence, or always start a new one.

.. _relative-relocation-table:

Relative Relocation Table
=========================

.. code-block:: c
   :caption: Relative Relocation Table Entries

   typedef Elf32_Word Elf32_Relr;
   typedef Elf64_Xword Elf64_Relr;

Relative relocations are used to identify virtual-address-sized storage
units within the object whose contents are independent of any dynamic
binding, but must still be relocated at load time to support position
independence. Before the program can begin execution, these locations
must be relocated by reading their contents and adding a relocation
factor, which is computed as the difference between the object's actual
load-time virtual address and its link-time virtual address. If the
object is loaded at the address for which it was linked, the relocation
factor is 0, and relative relocations may be ignored.

A relative relocation table is encoded as a sequence of ``Elf32_Relr``
entries for ``ELFCLASS32`` objects or ``Elf64_Relr`` entries for
``ELFCLASS64`` objects. The relative relocation table entries decode to
a list of virtual addresses that refer to storage units within the
object. Each of these storage units is the size of an ``Elf32_Addr`` (in
the case of ``ELFCLASS32`` objects) or an ``Elf64_Addr`` (in the case of
``ELFCLASS64`` objects).

.. note::

   Relative relocations could be represented simply as a list of virtual
   addresses that require relocation, which would be considerably more
   compact than using ``Elf32_Rel`` or ``Elf32_rela`` relocations.
   Because many such relocations occur in clusters, however, we can use a
   simple encoding scheme to compress the relative relocation table even
   further.

   A relative relocation table cannot describe relocations at odd
   addresses. For such relocations, a ``Rel``- or ``Rela``-style
   relocation must be used.

The encoded sequence of ``Elf32_Relr`` or ``Elf64_Relr`` entries starts
with an address entry (which must have a 0 in the least-significant
bit). This encodes one relative relocation at that address. This address
entry may be followed by zero or more bitmap entries, each of which has
a 1 in the least-significant bit.

Bitmap entries describe a block of ``Elf32_Addr`` or ``Elf64_Addr``
consecutive storage units immediately following the one to which the
address entry applied. Each bitmap entry covers 31 (for ``Elf32_Relr``)
or 63 (for ``Elf64_Relr``) storage units. Each bit in the bitmap entry,
excluding the least-significant bit, corresponds to a storage unit in
the block, the second-least-significant bit corresponding to the first,
and the most-significant bit corresponding to the last. For each 1 in
the bitmap entry, the corresponding storage unit is relocatable.

.. note::

   This encoding scheme has the property that a simple list of (even)
   addresses is a valid encoding.
