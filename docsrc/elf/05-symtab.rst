.. _Symbol-Table:

************
Symbol Table
************

An object file’s symbol table holds information
needed to locate and relocate a program’s symbolic
definitions and references.
A symbol table index is a subscript into this array.
Index 0 both designates the first entry in the table
and serves as the undefined symbol index.  The contents of the
initial entry are specified in :ref:`Symbol-Table-Entry-0`.

.. table:: Special Symbol Table Index

   =============  =====
   Name           Value
   =============  =====
   ``STN_UNDEF``  ``0``
   =============  =====

Symbol Table Entry
==================

A symbol table entry has the following format.

.. code-block:: c
   :caption: Symbol Table Entry

   typedef struct {
       Elf32_Word     st_name;
       Elf32_Addr     st_value;
       Elf32_Word     st_size;
       unsigned char  st_info;
       unsigned char  st_other;
       Elf32_Half     st_shndx;
   } Elf32_Sym;

   typedef struct {
       Elf64_Word     st_name;
       unsigned char  st_info;
       unsigned char  st_other;
       Elf64_Half     st_shndx;
       Elf64_Addr     st_value;
       Elf64_Xword    st_size;
   } Elf64_Sym;

``st_name``
    This member holds an index into the object file’s
    symbol string table, which
    holds the character representations of the symbol names.
    If the value is non-zero, it represents a string table
    index that gives the symbol name.
    Otherwise, the symbol table entry has no name.

    .. note::

       External C symbols have the same names in C
       and object files’ symbol tables.

``st_value``
    This member gives the value of the associated symbol.
    Depending on the context, this may be an absolute value,
    an address, and so on; details appear below.

``st_size``
    Many symbols have associated sizes.
    For example, a data object’s size is the number
    of bytes contained in the object.
    This member holds 0 if the symbol has no size or an unknown size.

``st_info``
    This member specifies the symbol’s type and binding attributes.
    A list of the values and meanings appears below.
    The following code shows how to manipulate the values for
    both 32 and 64-bit objects.

    .. code-block:: c

       #define ELF32_ST_BIND(i)   ((i)>>4)
       #define ELF32_ST_TYPE(i)   ((i)&0xf)
       #define ELF32_ST_INFO(b,t) (((b)<<4)+((t)&0xf))

       #define ELF64_ST_BIND(i)   ((i)>>4)
       #define ELF64_ST_TYPE(i)   ((i)&0xf)
       #define ELF64_ST_INFO(b,t) (((b)<<4)+((t)&0xf))

``st_other``
    This member currently specifies a symbol’s visibility.
    A list of the values and meanings appears below.
    The following code shows how to manipulate the values for
    both 32 and 64-bit objects.  Other bits contain 0 and have
    no defined meaning.

    .. code-block:: c

       #define ELF32_ST_VISIBILITY(o) ((o)&0x3)
       #define ELF64_ST_VISIBILITY(o) ((o)&0x3)

``st_shndx``
    Every symbol table entry is *defined* in relation
    to some section. This member holds the relevant
    section header table index.
    As the ``sh_link`` and ``sh_info`` interpretation
    table
    and the related text describe,
    some section indexes indicate special meanings.

    If this member contains ``SHN_XINDEX``,
    then the actual section header index is too large to fit in this field.
    The actual value is contained in the associated
    section of type ``SHT_SYMTAB_SHNDX``.

Symbol Binding
==============

A symbol’s binding determines the linkage visibility
and behavior.

.. table:: Symbol Binding

   ==============  ======
   Name            Value
   ==============  ======
   ``STB_LOCAL``   ``0``
   ``STB_GLOBAL``  ``1``
   ``STB_WEAK``    ``2``
   ``STB_LOOS``    ``10``
   ``STB_HIOS``    ``12``
   ``STB_LOPROC``  ``13``
   ``STB_HIPROC``  ``15``
   ==============  ======

``STB_LOCAL``
    Local symbols are not visible outside the object file
    containing their definition.
    Local symbols of the same name may exist in
    multiple files without interfering with each other.

``STB_GLOBAL``
    Global symbols are visible to all object files being combined.
    One file’s definition of a global symbol will satisfy
    another file’s undefined reference to the same global symbol.

``STB_WEAK``
    Weak symbols resemble global symbols, but their
    definitions have lower precedence.

``STB_LOOS`` through \ ``STB_HIOS``
    Values in this inclusive range
    are reserved for operating system-specific semantics.

``STB_LOPROC`` through \ ``STB_HIPROC``
    Values in this inclusive range
    are reserved for processor-specific semantics.  If meanings are
    specified, the psABI supplement explains them.

Global and weak symbols differ in two major ways.

* When the link editor combines several relocatable object files,
  it does not allow multiple definitions of ``STB_GLOBAL``
  symbols with the same name.
  On the other hand, if a defined global symbol exists,
  the appearance of a weak symbol with the same name
  will not cause an error.
  The link editor honors the global definition and ignores
  the weak ones.
  Similarly, if a common symbol exists
  (that is, a symbol whose ``st_shndx``
  field holds ``SHN_COMMON``\ ),
  the appearance of a weak symbol with the same name will
  not cause an error.
  The link editor honors the common definition and
  ignores the weak ones.

* When the link editor searches archive libraries,
  it extracts archive members that contain definitions of
  undefined global symbols.
  The member’s definition may be either a global or a weak symbol.
  The link editor does not
  extract archive members to resolve undefined weak symbols.
  Unresolved weak symbols have a zero value.

.. note::

   The behavior of weak symbols in areas not specified by this document is
   implementation defined.
   Weak symbols are intended primarily for use in system software.
   Applications using weak symbols are unreliable
   since changes in the runtime environment
   might cause the execution to fail.

In each symbol table, all symbols with ``STB_LOCAL``
binding precede the weak and global symbols.
As described in :ref:`Sections`,
a symbol table section’s ``sh_info``
section header member holds the symbol table index
for the first non-local symbol.

Symbol Type
===========

A symbol’s type provides a general classification for
the associated entity.

.. table:: Symbol Types

   ===============  ======
   Name             Value
   ===============  ======
   ``STT_NOTYPE``   ``0``
   ``STT_OBJECT``   ``1``
   ``STT_FUNC``     ``2``
   ``STT_SECTION``  ``3``
   ``STT_FILE``     ``4``
   ``STT_COMMON``   ``5``
   ``STT_TLS``      ``6``
   ``STT_LOOS``     ``10``
   ``STT_HIOS``     ``12``
   ``STT_LOPROC``   ``13``
   ``STT_HIPROC``   ``15``
   ===============  ======

``STT_NOTYPE``
    The symbol’s type is not specified.

``STT_OBJECT``
    The symbol is associated with a data object,
    such as a variable, an array, and so on.

``STT_FUNC``
    The symbol is associated with a function or other executable code.

``STT_SECTION``
    The symbol is associated with a section.
    Symbol table entries of this type exist primarily for relocation
    and normally have ``STB_LOCAL`` binding.

``STT_FILE``
    Conventionally, the symbol’s name gives the name of
    the source file associated with the object file.
    A file symbol has ``STB_LOCAL``
    binding, its section index is ``SHN_ABS``,
    and it precedes the other ``STB_LOCAL``
    symbols for the file, if it is present.

``STT_COMMON``
    The symbol labels an uninitialized common block.
    See below for details.

``STT_TLS``
    The symbol specifies a *Thread-Local Storage* entity.
    When defined, it gives the assigned offset for the symbol,
    not the actual address.
    Symbols of type ``STT_TLS`` can be referenced
    by only special thread-local storage relocations
    and thread-local storage relocations can only reference
    symbols with type ``STT_TLS``.
    Implementations need not support thread-local storage.

``STT_LOOS`` through \ ``STT_HIOS``
    Values in this inclusive range
    are reserved for operating system-specific semantics.

``STT_LOPROC`` through \ ``STT_HIPROC``
    Values in this inclusive range
    are reserved for processor-specific semantics.
    If meanings are specified, the psABI supplement explains them.

Function symbols (those with type
``STT_FUNC``\ ) in shared object files have special significance.
When another object file references a function from
a shared object, the link editor automatically creates a procedure
linkage table entry for the referenced symbol.
Shared object symbols with types other than
``STT_FUNC`` will not
be referenced automatically through the procedure linkage table.

Symbols with type ``STT_COMMON`` label uninitialized
common blocks.  In relocatable objects, these symbols are
not allocated and must have the special section index
``SHN_COMMON`` (see below\ ).
In shared objects and executables these symbols must be
allocated to some section in the defining object.

In relocatable objects, symbols with type ``STT_COMMON``
are treated just as other symbols with index ``SHN_COMMON``.
If the link-editor allocates space for the ``SHN_COMMON``
symbol in an output section of the object it is producing, it
must preserve the type of the output symbol as ``STT_COMMON``.

When the dynamic linker encounters a reference to a symbol
that resolves to a definition of type ``STT_COMMON``,
it may (but is not required to) change its symbol resolution
rules as follows: instead of binding the reference to
the first symbol found with the given name, the dynamic linker searches
for the first symbol with that name with type other
than ``STT_COMMON``.  If no such symbol is found,
it looks for the ``STT_COMMON`` definition of that
name that has the largest size.

Symbol Visibility
=================

A symbol’s visibility, although it may be specified in a relocatable
object, defines how that symbol may be accessed once it has
become part of an executable or shared object.

.. table:: Symbol Visibility

   =================  =====
   Name               Value
   =================  =====
   ``STV_DEFAULT``    ``0``
   ``STV_INTERNAL``   ``1``
   ``STV_HIDDEN``     ``2``
   ``STV_PROTECTED``  ``3``
   =================  =====

``STV_DEFAULT``
    The visibility of symbols with the ``STV_DEFAULT``
    attribute is as specified by the symbol’s binding type.
    That is, global and weak symbols are visible
    outside of their defining *component*
    (executable file or shared object).
    Local symbols are *hidden*, as described below.
    Global and weak symbols are also *preemptable*,
    that is, they may by preempted by definitions of the same
    name in another component.

    .. note::

       An implementation may restrict the set of global and weak
       symbols that are externally visible.

``STV_PROTECTED``
    A symbol defined in the current component is *protected*
    if it is visible in other components but not preemptable,
    meaning that any reference to such a symbol from within the
    defining component must be resolved to the definition in
    that component, even if there is a definition in another
    component that would preempt by the default rules.
    A symbol with ``STB_LOCAL`` binding may not have
    ``STV_PROTECTED`` visibility.

    If a symbol definition with ``STV_PROTECTED`` visibility
    from a shared object is taken as resolving a reference
    from an executable or another shared object,
    the ``SHN_UNDEF`` symbol table entry created
    has ``STV_DEFAULT`` visibility.

    .. note::

       The presence of the ``STV_PROTECTED`` flag on a symbol
       in a given load module does not affect the symbol resolution
       rules for references to that symbol from outside the containing
       load module.

``STV_HIDDEN``
    A symbol defined in the current component is *hidden*
    if its name is not visible to other components.  Such a symbol
    is necessarily protected.  This attribute may be used to
    control the external interface of a component.  Note that
    an object named by such a symbol may still be referenced
    from another component if its address is passed outside.

    A hidden symbol contained in a relocatable object must be
    either removed or converted to ``STB_LOCAL`` binding
    by the link-editor when the relocatable object is included in an
    executable file or shared object.

``STV_INTERNAL``
    The meaning of this visibility attribute may be defined by psABI
    supplements to further constrain hidden symbols.  A psABI
    supplement’s definition should be such that generic tools
    can safely treat internal symbols as hidden.

    An internal symbol contained in a relocatable object must be
    either removed or converted to ``STB_LOCAL`` binding
    by the link-editor when the relocatable object is included in an
    executable file or shared object.

None of the visibility attributes affects resolution of symbols
within an executable or shared object during link-editing -- such
resolution is controlled by the binding type.  Once the link-editor
has chosen its resolution, these attributes impose two requirements,
both based on the fact that references in the code being linked may
have been optimized to take advantage of the attributes.

* First, all of the non-default visibility attributes, when applied
  to a symbol reference, imply that a definition to satisfy that
  reference must be provided within the current executable or
  shared object.  If such a symbol reference has no definition within the
  component being linked, then the reference must have
  ``STB_WEAK`` binding and is resolved to zero.

* Second, if any reference to or definition of a name is a symbol with
  a non-default visibility attribute, the visibility attribute
  must be propagated to the resolving symbol in the linked object.
  If different visibility attributes are specified for distinct
  references to or definitions of a symbol, the most constraining
  visibility attribute must be propagated to the resolving symbol
  in the linked object.  The attributes, ordered from least
  to most constraining, are: ``STV_PROTECTED``,
  ``STV_HIDDEN`` and ``STV_INTERNAL``.

Section Index
=============

If a symbol’s value refers to a
specific location within a section,
its section index member, ``st_shndx``,
holds an index into the section header table.
As the section moves during relocation, the symbol’s value
changes as well, and references to the symbol
continue to “point” to the same location in the program.
Some special section index values give other semantics.

``SHN_ABS``
    The symbol has an absolute value that will not change
    because of relocation.

``SHN_COMMON``
    The symbol labels a common block that has not yet been allocated.
    The symbol’s value gives alignment constraints,
    similar to a section’s
    ``sh_addralign`` member.
    The link editor will allocate the storage for the symbol
    at an address that is a multiple of
    ``st_value``.
    The symbol’s size tells how many bytes are required.
    Symbols with section index ``SHN_COMMON`` may
    appear only in relocatable objects.

``SHN_UNDEF``
    This section table index means the symbol is undefined.
    When the link editor combines this object file with
    another that defines the indicated symbol,
    this file’s references to the symbol will be linked
    to the actual definition.

``SHN_XINDEX``
    This value is an escape value.
    It indicates that the symbol refers to a specific location within a section,
    but that the section header index for that section is too large to be
    represented directly in the symbol table entry.
    The actual section header index is found in the associated
    ``SHT_SYMTAB_SHNDX`` section.
    The entries in that section correspond one to one
    with the entries in the symbol table.
    Only those entries in ``SHT_SYMTAB_SHNDX``
    that correspond to symbol table entries with ``SHN_XINDEX``
    will hold valid section header indexes;
    all other entries will have value ``0``.

.. _Symbol-Table-Entry-0:

Symbol Table Entry 0
====================

The symbol table entry for index 0 (\ ``STN_UNDEF``\ ) is reserved;
it holds the following.

.. table:: Symbol Table Entry 0

   ============  =============  ======================
   Name          Value          Note
   ============  =============  ======================
   ``st_name``   ``0``          No name
   ``st_value``  ``0``          Zero value
   ``st_size``   ``0``          No size
   ``st_info``   ``0``          No type, local binding
   ``st_other``  ``0``          Default visibility
   ``st_shndx``  ``SHN_UNDEF``  No section
   ============  =============  ======================

Symbol Value
============

Symbol table entries for different object file types have
slightly different interpretations for the ``st_value`` member.

* In relocatable files, ``st_value`` holds alignment constraints for a symbol
  whose section index is ``SHN_COMMON``.

* In relocatable files, ``st_value`` holds
  a section offset for a defined symbol.
  ``st_value`` is an offset from the beginning of the section that
  ``st_shndx`` identifies.

* In executable and shared object files,
  ``st_value`` holds a virtual address.
  To make these files’ symbols more useful
  for the dynamic linker, the section offset (file interpretation)
  gives way to a virtual address (memory interpretation)
  for which the section number is irrelevant.

Although the symbol table values have similar meanings
for different object files, the data allows
efficient access by the appropriate programs.
