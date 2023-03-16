###################################
Program Loading and Dynamic Linking
###################################

The following sections describe the object file
information and system actions that create running programs.
Some information here applies to all systems;
information specific to one processor resides in
sections marked accordingly.

Executable and shared object files statically represent programs.
To execute such programs, the system uses the files to create
dynamic program representations, or process images.
A process image has segments that hold its text, data, stack, and so on.
This is described by the psABI supplement for the specific machine.

The next sections discuss the following:

* Program Header.
  This section complements the :ref:`Section Header Table <Sections>`,
  describing object file structures that relate directly to program execution.
  The primary data structure, a program header table, locates
  segment images within the file and contains other information
  necessary to create the memory image for the program.

* Program Loading.
  Given an object file, the system must load
  it into memory for the program to run.

* Dynamic linking.
  After the system loads the program it must complete
  the process image by resolving symbolic references among the object
  files that compose the process.

.. note::

   The psABI supplement defines a naming convention for ELF constants
   that have processor ranges specified.  Names such as ``DT_``, ``PT_``,
   for processor specific extensions, incorporate the name of the
   processor: ``DT_M32_SPECIAL``, for example.
   Pre-existing processor extensions not using this convention will be supported.

   .. table::

      +-------------------------+
      | Pre-Existing Extensions |
      +=========================+
      | ``DT_JUMP_REL``         |
      +-------------------------+
