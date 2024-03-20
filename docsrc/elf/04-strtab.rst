.. _String-Table:

************
String Table
************

String table sections hold null-terminated character sequences,
commonly called strings.
The object file uses these strings to represent symbol and section names.
One references a string as an index into the
string table section.
The first byte, which is index zero, is defined to hold
a null character.
Likewise, a string table’s last byte is defined to hold
a null character, ensuring null termination for all strings.
A string whose index is zero specifies
either no name or a null name, depending on the context.
An empty string table section is permitted; its section header’s ``sh_size``
member would contain zero.
Non-zero indexes are invalid for an empty string table.

A section header’s ``sh_name``
member holds an index into the section header string table
section, as designated by the ``e_shstrndx``
member of the ELF header.
The following figures show a string table with 25 bytes
and the strings associated with various indexes.

.. figure:: /svg/figure-3.svg
   :alt: Example String Table
   :width: 540pt

   Example String Table

.. table:: String Table Indexes

   =====  ======================
   Index  String
   =====  ======================
   0      *none*
   1      ``“name.”``
   7      ``“Variable”``
   11     ``“able”``
   16     ``“able”``
   22     ``“xx”``
   24     ``“”`` (*null string*)
   =====  ======================

As the example shows, a string table index may refer
to any byte in the section.
A string may appear more than once,
references to substrings may exist,
and a single string may be referenced multiple times.
Unreferenced strings also are allowed.
