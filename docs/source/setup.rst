.. include:: defs.h

.. _`Chp:The |flashx| configuration script`:

The |flashx| configuration script (``setup``)
============================================

The ``setup`` tool is the most important component of the . It
implements the inheritance and composability of the |flashx| software
system. It traverses the |flashx| source tree starting from the directory
hosting the specific application definition. This starting directory is
essentially the selected implementation of the “Simulation” unit. While
traversing the source three the setup tool does the following:

-  arbitrate on map from a key to its definition, in particular if
   alternative definitions exist

-  arbitrate on which implementation of a function to use

-  link selected files (include source code and the key definitions) to
   the ``object/`` directory

-  invoke macroprocessor to translate keys and generate source file

-  alter index order if desired

-  find the target ``Makefile.h``

-  generate the ``Makefile`` that will build the |flashx| executable.

-  generate the files needed to add runtime parameters to a given
   simulation.

-  generate the files needed to parse the runtime parameter file.

More description of how ``setup`` and the |flashx| architecture interact
may be found in . Here we describe its usage.

The ``setup`` script determines site-dependent configuration information
by looking for a directory ``sites/<hostname>`` where ``<hostname>`` is
the hostname of the machine on which |flashx| is running. [1]_ Failing
this, it looks in ``sites/Prototypes/`` for a directory with the same
name as the output of the ``uname`` command. The site and operating
system type can be overridden with the ``-site`` and ``-ostype``
command-line options to the ``setup`` command. Only one of these options
can be used at one time. The directory for each site and operating
system type contains a makefile fragment ``Makefile.h`` that sets
command names, compiler flags, library paths, and any replacement or
additional source files needed to compile |flashx| for that specific
machine and machine type.

The ``setup`` script starts with the ``Config`` file of the specified
application in the Simulation unit, finds its ``REQUIRED`` units and
then works its way through their ``Config`` files. This process
continues until all the dependencies are met and a self-consistent set
of units has been found. At the end of this automatic generation, the
``Units`` file is created and placed in the ``object/`` directory, where
it can be edited if necessary. ``setup`` also creates the master
makefile (``object/Makefile``) and several FORTRAN include files that
are needed by the code in order to parse the runtime parameters. After
running ``setup``, the user can create the |flashx| executable by running
``make`` in the ``object`` directory. Note that the |flashx| build system
assumes that the command ``make`` invokes GNU Make and is unlikely to
work properly with other implementations of the ``make`` command. On
some systems it may be necessary to invoke GNU Make under the name
``gmake``.

.. container:: flashtip

   -  All the setup options can be shortened to unambiguous prefixes,
      *e.g.* instead of ``./setup -auto <problem-name>`` one can just
      say ``./setup -a <problem-name>`` since there is only one
      ``setup`` option starting with ``a``.

   -  The same abbreviation holds for the problem name as well.
      ``./setup -a IsentropicVortex`` can be abbreviated to
      ``./setup -a Isen`` assuming that ``IsentropicVortex`` is the only
      problem name which starts with ``Isen``.

   -  Unit names are usually specified by their paths relative to the
      source directory. However, ``setup`` also allows unit names to be
      prefixed with an extra “source/”, allowing you to use the
      TAB-completion features of your shell like this

      .. container:: codeseg

         ./setup -a Isen -unit=sou<TAB>rce/IO/IOM<TAB>ain/hd<TAB>f5

   -  If you use a set of options repeatedly, you can define a shortcut
      for them. |flashx| comes with a number of predefined shortcuts that
      significantly simplify the setup line, particularly when trying to
      match the Grid with a specific I/O implementation. For more
      details on creating shortcuts see . For detailed examples of I/O
      shortcuts please see in the I/O chapter.

Setup Arguments
---------------

The setup script accepts a large number of command line arguments which
affect the simulation in various ways. These arguments are divided into
three categories:

#. *Setup Options* (example: ``-auto``) begin with a dash and are built
   into the setup script itself. Many of the most commonly used
   arguments are setup options.

#. *Setup Variables* (example: ``species=air,h2o``) are defined by
   individual units. When writing a ``Config`` file for any unit, you
   can define a setup variable. explains how setup variables can be
   created and used.

#. *Setup Shortcuts* (example: ``+ug``) begin with a plus symbol and are
   essentially macros which automatically include a set of setup
   variables and/or setup options. New setup shortcuts can be easily
   defined, see for more information.

shows a list of some of the basic setup arguments that every |flashx|
user should know about. A comprehensive list of all setup arguments can
be found in alongside more detailed descriptions of these options.

.. container:: center

   .. container::
      :name: Tbl:CommonSetupArgs

      .. table::  List of Commonly Used Setup Arguments

         ================ ===================================================
         **Argument**     **Description**
         ================ ===================================================
         \                this option should almost always be set
         ``-unit=<unit>`` include a specified unit
         \                specify a different object directory location
         ``-debug``       compile for debugging
         \                enable compiler optimization
         ``-n[xyb]b=#``   specify block size in each direction
         \                specify maximum number of blocks per process
         ``-[123]d``      specify number of dimensions
         \                specify maximum number of blocks per process
         ``+cartesian``   use Cartesian geometry
         \                use cylindrical geometry
         ``+polar``       use polar geometry
         \                use spherical geometry
         ``+noio``        disable IO
         \                use the uniform grid in a fixed block size mode
         ``+nofbs``       use the uniform grid in a non-fixed block size mode
         \                use the PARAMESH2 grid
         ``+pm40``        use the PARAMESH4.0 grid
         \                use the PARAMESH4DEV grid
         ``+uhd``         use the Unsplit Hydro solver
         \                use the Unsplit Staggered Mesh MHD solver
         ``+splitHydro``  use a split Hydro solver
         ================ ===================================================

.. _`Sec:ListSetupArgs`:

Comprehensive List of Setup Arguments
-------------------------------------

+----------------------------------+----------------------------------+
| ``-verbose=<verbosity>``         |                                  |
+----------------------------------+----------------------------------+
| \*[1ex]                          | Normally ``setup`` prints        |
|                                  | summary messages indicating its  |
|                                  | progress. Use the ``-verbose``   |
|                                  | to make the messages more or     |
|                                  | less verbose. The different      |
|                                  | levels (in order of increasing   |
|                                  | verbosity) are                   |
|                                  | ``                               |
|                                  | ERROR,IMPINFO,WARN,INFO,DEBUG``. |
|                                  | The default is ``WARN``.         |
+----------------------------------+----------------------------------+
| ``-auto``                        |                                  |
+----------------------------------+----------------------------------+
| \*[1ex]                          | Normally, ``setup`` requires     |
|                                  | that the user supply a plain     |
|                                  | text file called ``Units`` (in   |
|                                  | the ``object`` directory  [4]_)  |
|                                  | that specifies the units to      |
|                                  | include. A sample ``Units`` file |
|                                  | appears in . Each line is either |
|                                  | a comment (preceded by a hash    |
|                                  | mark (``#``)) or the name of a   |
|                                  | an include statement of the form |
|                                  | ``INCLUDE``\ *unit*. Specific    |
|                                  | implementations of a unit may be |
|                                  | selected by specifying the       |
|                                  | complete path to the             |
|                                  | implementation in question; If   |
|                                  | no specific implementation is    |
|                                  | requested, ``setup`` picks the   |
|                                  | default listed in the unit’s     |
|                                  | ``Config`` file.                 |
+----------------------------------+----------------------------------+
|                                  | The ``-auto`` option enables     |
|                                  | ``setup`` to generate a “rough   |
|                                  | draft” of a ``Units`` file for   |
|                                  | the user. The ``Config`` file    |
|                                  | for each problem setup specifies |
|                                  | its requirements in terms of     |
|                                  | other units it requires. For     |
|                                  | example, a problem may require   |
|                                  | the perfect-gas equation of      |
|                                  | state                            |
|                                  | (``physics/Eos/EosMain/Gamma``)  |
|                                  | and an unspecified hydro solver  |
|                                  | (``physics/Hydro``). With        |
|                                  | ``-auto``, ``setup`` creates a   |
|                                  | ``Units`` file by converting     |
|                                  | these requirements into unit     |
|                                  | include statements. Most users   |
|                                  | configuring a problem for the    |
|                                  | first time will want to run      |
|                                  | ``setup`` with ``-auto`` to      |
|                                  | generate a ``Units`` file and    |
|                                  | then to edit it directly to      |
|                                  | specify alternate                |
|                                  | implementations of certain       |
|                                  | units. After editing the         |
|                                  | ``Units`` file, the user must    |
|                                  | re-run ``setup`` without         |
|                                  | ``-auto`` in order to            |
|                                  | incorporate his/her changes into |
|                                  | the code configuration. The user |
|                                  | may also use the command-line    |
|                                  | option ``-with-unit=<path>`` in  |
|                                  | conjunction with the ``-auto``   |
|                                  | option, in order to pick a       |
|                                  | specific implementation of a     |
|                                  | unit, and thus eliminate the     |
|                                  | need to hand-edit the ``Units``  |
|                                  | file.                            |
+----------------------------------+----------------------------------+
| ``-[123]d``                      |                                  |
+----------------------------------+----------------------------------+
| \*[1ex]                          | By default, ``setup`` creates a  |
|                                  | makefile which produces a        |
|                                  | |flashx| executable capable of    |
|                                  | solving two-dimensional problems |
|                                  | (equivalent to ``-2d``). To      |
|                                  | generate a makefile with options |
|                                  | appropriate to three-dimensional |
|                                  | problems, use ``-3d``. To        |
|                                  | generate a one-dimensional code, |
|                                  | use ``-1d``. These options are   |
|                                  | mutually exclusive and cause     |
|                                  | ``setup`` to add the appropriate |
|                                  | compilation option to the        |
|                                  | makefile it generates.           |
+----------------------------------+----------------------------------+
| ``-maxblocks=#``                 |                                  |
+----------------------------------+----------------------------------+
| \*[1ex]                          | ``setup`` in constructing the    |
|                                  | makefile compiler options. It    |
|                                  | determines the amount of memory  |
|                                  | allocated at runtime to the      |
|                                  | adaptive mesh refinement (AMR)   |
|                                  | block data structure. For        |
|                                  | example, to allocate enough      |
|                                  | memory on each processor for 500 |
|                                  | blocks, use ``-maxblocks=500``.  |
|                                  | If the default block buffer size |
|                                  | is too large for your system,    |
|                                  | you may wish to try a smaller    |
|                                  | number here; the default value   |
|                                  | depends upon the dimensionality  |
|                                  | of the simulation and the grid   |
|                                  | type. Alternatively, you may     |
|                                  | wish to experiment with larger   |
|                                  | buffer sizes, if your system has |
|                                  | enough memory. A common cause of |
|                                  | aborted simulations occurs when  |
|                                  | the AMR grid creates greater     |
|                                  | than ``maxblocks`` during        |
|                                  | refinement. Resetup the          |
|                                  | simulation using a larger value  |
|                                  | of this option.                  |
+----------------------------------+----------------------------------+
| ``-nxb=# -nyb=# -nzb=#``         |                                  |
+----------------------------------+----------------------------------+
| \*[1ex] These options are used   |                                  |
| by ``setup`` in constructing the |                                  |
| makefile compiler options. The   |                                  |
| mesh on which the problem is     |                                  |
| solved is composed of blocks,    |                                  |
| and each block contains some     |                                  |
| number of cells. The ``-nxb``,   |                                  |
| ``-nyb``, and ``-nzb`` options   |                                  |
| determine how many cells each    |                                  |
| block contains (not counting     |                                  |
| guard cells). The default value  |                                  |
| for each is 8. These options do  |                                  |
| not have any effect when running |                                  |
| in Uniform Grid non-fixed block  |                                  |
| size mode.                       |                                  |
+----------------------------------+----------------------------------+
| ``[-debug|-opt|-test]``          |                                  |
+----------------------------------+----------------------------------+
| \*[1ex]                          | The default ``Makefile`` built   |
|                                  | by setup will use the optimized  |
|                                  | setting (``-opt``) for           |
|                                  | compilation and linking. Using   |
|                                  | ``-debug`` will force ``setup``  |
|                                  | to use the flags relevant for    |
|                                  | debugging (*e.g.*, including     |
|                                  | ``-g`` in the compilation line). |
|                                  | The user may use the option      |
|                                  | ``-test`` to experiment with     |
|                                  | different combinations of        |
|                                  | compiler and linker options.     |
|                                  | Exactly which compiler and       |
|                                  | linker options are associated    |
|                                  | with each of these flags is      |
|                                  | specified in                     |
|                                  | ``sites/<hostname>/Makefile*``   |
|                                  | where ``<hostname>`` is the      |
|                                  | hostname of the machine on which |
|                                  | |flashx| is running.              |
|                                  |                                  |
|                                  | For example, to tell an Intel    |
|                                  | Fortran compiler to use real     |
|                                  | numbers of size 64 when the      |
|                                  | ``-test`` option is specified,   |
|                                  | the user might add the following |
|                                  | line to his/her ``Makefile.h``:  |
|                                  |                                  |
|                                  | .. container:: codeseg           |
|                                  |                                  |
|                                  |    FFLAGS_TEST = -real_size 64   |
+----------------------------------+----------------------------------+
| ``-objdir=<dir>``                |                                  |
+----------------------------------+----------------------------------+
| \*[1ex] Overrides the default    |                                  |
| ``object`` directory with        |                                  |
| ``<dir>``. Using this option     |                                  |
| allows you to have different     |                                  |
| simulations configured           |                                  |
| simultaneously in the |flashx|    |                                  |
| distribution directory.          |                                  |
+----------------------------------+----------------------------------+
| ``-with-unit=<unit>``,           |                                  |
| ``-unit=<unit>``                 |                                  |
+----------------------------------+----------------------------------+
| \*[1ex] ``<unit>`` in setting up |                                  |
| the problem.                     |                                  |
+----------------------------------+----------------------------------+

.. container:: fcodeseg

   #Units file for Sod generated by setup

   INCLUDE Driver/DriverMain/Split INCLUDE Grid/GridBoundaryConditions
   INCLUDE Grid/GridMain/paramesh/interpolation/|paramesh|4/prolong
   INCLUDE Grid/GridMain/paramesh/interpolation/prolong INCLUDE
   Grid/GridMain/paramesh/paramesh4/|paramesh|4.0/PM4_package/headers
   INCLUDE
   Grid/GridMain/paramesh/paramesh4/|paramesh|4.0/PM4_package/mpi_source
   INCLUDE
   Grid/GridMain/paramesh/paramesh4/|paramesh|4.0/PM4_package/source
   INCLUDE
   Grid/GridMain/paramesh/paramesh4/|paramesh|4.0/PM4_package/utilities/multigrid
   INCLUDE Grid/localAPI INCLUDE IO/IOMain/hdf5/serial/PM INCLUDE
   IO/localAPI INCLUDE PhysicalConstants/PhysicalConstantsMain INCLUDE
   RuntimeParameters/RuntimeParametersMain INCLUDE
   Simulation/SimulationMain/Sod INCLUDE
   flashUtilities/contiguousConversion INCLUDE flashUtilities/general
   INCLUDE flashUtilities/interpolation/oneDim INCLUDE
   flashUtilities/nameValueLL INCLUDE monitors/Logfile/LogfileMain
   INCLUDE monitors/Timers/TimersMain/MPINative INCLUDE
   physics/Eos/EosMain/Gamma INCLUDE
   physics/Hydro/HydroMain/split/PPM/PPMKernel

+---------------------------------+-----------------------------------+
| ``-curvilinear``                |                                   |
+---------------------------------+-----------------------------------+
| \*[1ex]                         | Enable code in ``PARAMESH`` 4     |
|                                 | that implements geometrically     |
|                                 | correct data restriction for      |
|                                 | curvilinear coordinates. This     |
|                                 | setting is automatically enabled  |
|                                 | if a non-``cartesian`` geometry   |
|                                 | is chosen with the ``-geometry``  |
|                                 | flag; so specifying               |
|                                 | ``-curvilinear`` only has an      |
|                                 | effect in the Cartesian case.     |
+---------------------------------+-----------------------------------+
| ``-defines=<def>[,<def>]…``     |                                   |
+---------------------------------+-----------------------------------+
| \*[1ex]                         | ``<def>`` is of the form          |
|                                 | ``SYMBOL`` or ``SYMBOL=value``.   |
|                                 | This causes the specified         |
|                                 | pre-processor symbols to be       |
|                                 | defined when the code is being    |
|                                 | compiled. This is mainly useful   |
|                                 | for debugging the code. For       |
|                                 | *e.g.*, ``-defines=DEBUG_ALL``    |
|                                 | turns on all debugging messages.  |
|                                 | Each unit may have its own        |
|                                 | ``DEBUG_UNIT`` flag which you can |
|                                 | selectively turn on.              |
+---------------------------------+-----------------------------------+
| ``[-fbs|-nofbs]``               |                                   |
+---------------------------------+-----------------------------------+
| \*[1ex]                         | Causes the code to be compiled in |
|                                 | fixed-block or non-fixed-block    |
|                                 | size mode. Fixed-block mode is    |
|                                 | the default. In non-fixed block   |
|                                 | size mode, all storage space is   |
|                                 | allocated at runtime. This mode   |
|                                 | is available only with Uniform    |
|                                 | Grid.                             |
+---------------------------------+-----------------------------------+
| ``-geometry=<geometry>``        |                                   |
+---------------------------------+-----------------------------------+
| \*[1ex]                         | Choose one of the supported       |
|                                 | geometries                        |
|                                 | ``car                             |
|                                 | tesian, cylindrical, spherical,`` |
|                                 | or ``polar``. Some ``Grid``       |
|                                 | implementations require the       |
|                                 | geometry to be known at           |
|                                 | compile-time while others don’t.  |
|                                 | This setup option can be used in  |
|                                 | either case; it is a good idea to |
|                                 | specify the geometry here if it   |
|                                 | is known at ``setup``-time.       |
|                                 | Choosing a non-Cartesian geometry |
|                                 | here automatically sets the       |
|                                 | ``-gridinterpolation=monotonic``  |
|                                 | option below.                     |
+---------------------------------+-----------------------------------+
| ``-gridinterpolation=<scheme>`` |                                   |
+---------------------------------+-----------------------------------+
| \*[1ex]                         | Select a scheme for ``Grid``      |
|                                 | interpolation. Two schemes are    |
|                                 | currently supported:              |
|                                 |                                   |
|                                 | -  **monotonic** This scheme      |
|                                 |    attempts to ensure that        |
|                                 |    monotonicity is preserved in   |
|                                 |    interpolation, so that         |
|                                 |    interpolation does not         |
|                                 |    introduce small-scale          |
|                                 |    non-monotonicity in the data.  |
|                                 |    The ``monotonic`` scheme is    |
|                                 |    required for curvilinear       |
|                                 |    coordinates and is             |
|                                 |    automatically enabled if a     |
|                                 |    non-``cartesian`` geometry is  |
|                                 |    chosen with the ``-geometry``  |
|                                 |    flag. For AMR ``Grid``         |
|                                 |    implementations, This flag     |
|                                 |    will automatically add         |
|                                 |    additional directories so that |
|                                 |    appropriate data interpolation |
|                                 |    methods are compiled it. The   |
|                                 |    ``monotonic`` scheme is the    |
|                                 |    default (by way of the         |
|                                 |    ``+default`` shortcut), unlike |
|                                 |    in |flashx|.                    |
|                                 |                                   |
|                                 | -  **native** Enable the          |
|                                 |    interpolation that is native   |
|                                 |    to the AMR ``Grid``            |
|                                 |    implementation (``PARAMESH`` 2 |
|                                 |    or ``PARAMESH`` 4) by default. |
|                                 |    This option is only            |
|                                 |    appropriate for Cartesian      |
|                                 |    geometries.                    |
+---------------------------------+-----------------------------------+

.. container::
   :name: setupclf:particlemethods

   +----------------------------------+----------------------------------+
   | ``-makefile=<extension>``        |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | ``setup`` normally uses the      |
   |                                  | ``Makefile.h`` from the          |
   |                                  | directory determined by the      |
   |                                  | hostname of the machine and the  |
   |                                  | ``-site`` and ``-os`` options.   |
   |                                  | If you have multiple compilers   |
   |                                  | on your machine you can create   |
   |                                  | ``Makefile.h.<extension>`` for   |
   |                                  | different compilers. *e.g.*, you |
   |                                  | can have a ``Makefile.h`` and    |
   |                                  | ``Makefile.h.intel`` and         |
   |                                  | ``Makefile.h.lahey`` for the     |
   |                                  | three different compilers.       |
   |                                  | ``setup`` will still use the     |
   |                                  | ``Makefile.h`` file by default,  |
   |                                  | but supplying                    |
   |                                  | ``-makefile=intel`` on the       |
   |                                  | command-line causes ``setup`` to |
   |                                  | use ``Makefile.h.intel``         |
   |                                  | instead.                         |
   +----------------------------------+----------------------------------+
   | ``-index-reorder``               |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex] Instructs ``setup`` that |                                  |
   | indexing of unk and related      |                                  |
   | arrays should be changed. This   |                                  |
   | may be needed in |flashx| for     |                                  |
   | compatibility with alternative   |                                  |
   | grids. This is supported by both |                                  |
   | the Uniform Grid as well as      |                                  |
   | PARAMESH.                        |                                  |
   +----------------------------------+----------------------------------+
   | ``-makehide``                    |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | Ordinarily, the commands being   |
   |                                  | executed during compilation of   |
   |                                  | the |flashx| executable are sent  |
   |                                  | to standard out. It may be that  |
   |                                  | you find this distracting, or    |
   |                                  | that your terminal is not able   |
   |                                  | to handle these long lines of    |
   |                                  | display. Using the option        |
   |                                  | ``-makehide`` causes ``setup``   |
   |                                  | to generate a ``Makefile`` so    |
   |                                  | that GNU ``make`` only displays  |
   |                                  | the names of the files being     |
   |                                  | compiled and not the exact       |
   |                                  | compiler call and flags. This    |
   |                                  | information remains available in |
   |                                  | ``setup_flags`` in the           |
   |                                  | ``object/`` directory.           |
   +----------------------------------+----------------------------------+
   | ``-noclobber``                   |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | ``setup`` normally removes all   |
   |                                  | code in the ``object`` directory |
   |                                  | before linking in files for a    |
   |                                  | simulation. The ensuing ``make`` |
   |                                  | must therefore compile all       |
   |                                  | source files anew each time      |
   |                                  | ``setup`` is run. The            |
   |                                  | ``-noclobber`` option prevents   |
   |                                  | ``setup`` from removing compiled |
   |                                  | code which has not changed from  |
   |                                  | the previous ``setup`` in the    |
   |                                  | same directory. This can speed   |
   |                                  | up the ``make`` process          |
   |                                  | significantly.                   |
   +----------------------------------+----------------------------------+
   | ``-os=<os>``                     |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | If ``setup`` is unable to find a |
   |                                  | correct ``sites/`` directory it  |
   |                                  | picks the ``Makefile`` based on  |
   |                                  | the operating system. This       |
   |                                  | option instructs ``setup`` to    |
   |                                  | use the default ``Makefile``     |
   |                                  | corresponding to the specified   |
   |                                  | operating system.                |
   +----------------------------------+----------------------------------+
   | ``-parfile=<filename>``          |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This causes ``setup`` to copy    |
   |                                  | the specified runtime-parameters |
   |                                  | file in the simulation directory |
   |                                  | to the ``object`` directory with |
   |                                  | the new name ``flash.par`` .     |
   +----------------------------------+----------------------------------+
   | ``-appe                          |                                  |
   | nd-parfiles=[location1/]<filenam |                                  |
   | e1>[,[location2/]<filename2>]…`` |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This option takes a              |
   |                                  | comma-separated list of names of |
   |                                  | parameter files and combines     |
   |                                  | them into one ``flash.par`` file |
   |                                  | in the ``object`` directory.     |
   |                                  | File names without an absolute   |
   |                                  | path are taken to be relative to |
   |                                  | the simulation directory, as for |
   |                                  | the ``-parfile`` option.         |
   |                                  |                                  |
   |                                  | To use such a combined           |
   |                                  | ``flash.par`` in case of runtime |
   |                                  | parameters occurring more than   |
   |                                  | once, note that when |flashx|     |
   |                                  | reads a parameter file, the last |
   |                                  | instance of a runtime parameter  |
   |                                  | supersedes previous ones.        |
   |                                  |                                  |
   |                                  | If both ``-append-parfiles`` and |
   |                                  | ``-parfile`` are used, the files |
   |                                  | from the list are appended to    |
   |                                  | the single parfile given by the  |
   |                                  | latter in the order listed. If   |
   |                                  | used with ``-parfile``,          |
   |                                  | ``-append-parfiles`` can append  |
   |                                  | one or more parfiles to the one  |
   |                                  | given by ``-parfile``. If you    |
   |                                  | only use ``-append-parfiles``    |
   |                                  | and not ``-parfile`` and give it |
   |                                  | fewer than two paths, an error   |
   |                                  | will result. If more than one    |
   |                                  | ``-append-parfiles`` option      |
   |                                  | appears, the lists are           |
   |                                  | concatenated in the order given. |
   +----------------------------------+----------------------------------+
   | ``-particlemet                   |                                  |
   | hods=TYPE=<particle type>[,INIT= |                                  |
   | <init method>]``\ ``[,MAP=<map m |                                  |
   | ethod>][,ADV=<advance method>]`` |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This option instructs ``setup``  |
   |                                  | to adjust the particle methods   |
   |                                  | for a particular particle type.  |
   |                                  | It can only be used when a       |
   |                                  | particle type has already been   |
   |                                  | registered with a                |
   |                                  | ``PARTICLETYPE`` line in a       |
   |                                  | ``Config`` file (see ). A        |
   |                                  | possible scenario for using this |
   |                                  | option involves the user wanting |
   |                                  | to use a different passive       |
   |                                  | particle initialization method   |
   |                                  | without modifying the            |
   |                                  | ``PARTICLETYPE`` line in the     |
   |                                  | simulation ``Config`` file. In   |
   |                                  | this case, an additional         |
   |                                  | ``-particlemeth                  |
   |                                  | ods=TYPE=passive,INIT=cellmass`` |
   |                                  | adjusts the initialization       |
   |                                  | method associated with passive   |
   |                                  | particles in the ``setup``       |
   |                                  | generated                        |
   |                                  | ``Particles_specifyMethods()``   |
   |                                  | subroutine. Since the            |
   |                                  | specification of a method for    |
   |                                  | mapping and initialization       |
   |                                  | requires inclusions of           |
   |                                  | appropriate implementations of   |
   |                                  | ``ParticlesMapping`` and         |
   |                                  | ``ParticlesInitialization``      |
   |                                  | subunits, and the specification  |
   |                                  | of a method for time advancement |
   |                                  | requires inclusion of an         |
   |                                  | appropriate implementation under |
   |                                  | ``ParticlesMain``, it is the     |
   |                                  | user’s responsibility to adjust  |
   |                                  | the included units               |
   |                                  | appropriately. For example a     |
   |                                  | user may want want to override   |
   |                                  | ``Config`` file defined particle |
   |                                  | type ``passive`` using lattice   |
   |                                  | initialization ``CellMassBins``  |
   |                                  | density based distribution       |
   |                                  | method using the ``setup``       |
   |                                  | command line. Here the user must |
   |                                  | first specify                    |
   |                                  | ``-without-unit=Particles/P      |
   |                                  | articlesInitialization/Lattice`` |
   |                                  | to exclude the lattice           |
   |                                  | initialization, followed by      |
   |                                  | ``-with-u                        |
   |                                  | nit=Particles/ParticlesInitializ |
   |                                  | ation/WithDensity/CellMassBins`` |
   |                                  | specification to include the     |
   |                                  | appropriate implementation. In   |
   |                                  | general, using command line      |
   |                                  | overrides of                     |
   |                                  | ``-particlemethods`` are not     |
   |                                  | recommended, as this option      |
   |                                  | increases the chance of creating |
   |                                  | an inconsistent simulation       |
   |                                  | setup. More information on       |
   |                                  | multiple particle types can be   |
   |                                  | found in , especially .          |
   +----------------------------------+----------------------------------+
   | ``-portable``                    |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This option causes setup to      |
   |                                  | create a portable ``object``     |
   |                                  | directory by copying instead of  |
   |                                  | linking to the source files. The |
   |                                  | resulting ``object`` directory   |
   |                                  | can be tarred and sent to        |
   |                                  | another machine for actual       |
   |                                  | compilation.                     |
   +----------------------------------+----------------------------------+
   | ``-site=<site>``                 |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | ``setup`` searches the           |
   |                                  | ``sites/`` directory for a       |
   |                                  | directory whose name is the      |
   |                                  | hostname of the machine on which |
   |                                  | setup is being run. This option  |
   |                                  | tells ``setup`` to use the       |
   |                                  | ``Makefile`` of the specified    |
   |                                  | site. This option is useful if   |
   |                                  | ``setup`` is unable to find the  |
   |                                  | right hostname (which can happen |
   |                                  | on multiprocessor or laptop      |
   |                                  | machines). Also useful when      |
   |                                  | combined with the ``-portable``  |
   |                                  | option.                          |
   +----------------------------------+----------------------------------+
   | ``-unitsfile=<filename>``        |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This causes ``setup`` to copy    |
   |                                  | the specified file to the        |
   |                                  | ``object`` directory as          |
   |                                  | ``Units`` before setting up the  |
   |                                  | problem. This option can be used |
   |                                  | when ``-auto`` is not used, to   |
   |                                  | specify an alternate ``Units``   |
   |                                  | file.                            |
   +----------------------------------+----------------------------------+
   | ``-                              |                                  |
   | with-library=<libname>[,args]``, |                                  |
   | ``-library=<libname>[,args]``    |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This option instructs ``setup``  |
   |                                  | to link in the specified library |
   |                                  | when building the final          |
   |                                  | executable. A *library* is a     |
   |                                  | piece of code which is           |
   |                                  | independent of |flashx|. Internal |
   |                                  | libraries are those libraries    |
   |                                  | whose code is included with      |
   |                                  | |flashx|. The ``setup`` script    |
   |                                  | supports external as well as     |
   |                                  | internal libraries. Information  |
   |                                  | about external libraries is      |
   |                                  | usually found in the site        |
   |                                  | specific Makefile. The           |
   |                                  | additional ``args`` if any are   |
   |                                  | library-specific and may be used |
   |                                  | to select among multiple         |
   |                                  | implementations.                 |
   +----------------------------------+----------------------------------+
   | ``-tau=<makefile>``              |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This option causes the inclusion |
   |                                  | of an additional Makefile        |
   |                                  | necessary for the operation of   |
   |                                  | Tau, which may be used by the    |
   |                                  | user to profile the code. More   |
   |                                  | information on Tau can be found  |
   |                                  | at http://acts.nersc.gov/tau/    |
   +----------------------------------+----------------------------------+
   | ``-without-library=<libname>``   |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | Negates a previously specified   |
   |                                  | ``                               |
   |                                  | -with-library=<libname>[,args]`` |
   +----------------------------------+----------------------------------+
   | ``-without-unit=<unit>``         |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This removes all units specified |
   |                                  | in the command line so far,      |
   |                                  | which are children of the        |
   |                                  | specified unit (including the    |
   |                                  | unit itself). It also negates    |
   |                                  | any REQUESTS keyword found in a  |
   |                                  | ``Config`` file for units which  |
   |                                  | are children of the specified    |
   |                                  | unit. However it does not negate |
   |                                  | a REQUIRES keyword found in a    |
   |                                  | ``Config`` file.                 |
   +----------------------------------+----------------------------------+
   | ``+default``                     |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies using    |
   |                                  | basic default settings and is    |
   |                                  | equivalent to the following:     |
   |                                  | ``–with-library=mpi +io +gr      |
   |                                  | id-gridinterpolation=monotonic`` |
   +----------------------------------+----------------------------------+
   | ``+noio``                        |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation without IO and is     |
   |                                  | equivalent to the following:     |
   |                                  | ``–without                       |
   |                                  | -unit=physics/sourceTerms/Energy |
   |                                  | Deposition/EnergyDepositionMain/ |
   |                                  | Laser/LaserIO –without-unit=IO`` |
   +----------------------------------+----------------------------------+
   | ``+io``                          |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation with basic IO and is  |
   |                                  | equivalent to the following:     |
   |                                  | ``–with-unit=IO``                |
   +----------------------------------+----------------------------------+
   | ``+serialIO``                    |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using serial IO, it   |
   |                                  | has the effect of setting the    |
   |                                  | setup variable                   |
   +----------------------------------+----------------------------------+
   | ``+parallelIO``                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using serial IO, it   |
   |                                  | has the effect of setting the    |
   |                                  | setup variable                   |
   +----------------------------------+----------------------------------+
   | ``+hdf5``                        |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using hdf5 for        |
   |                                  | compatible binary IO output, it  |
   |                                  | has the effect of setting the    |
   |                                  | setup variable                   |
   +----------------------------------+----------------------------------+
   | ``+hdf5TypeIO``                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using hdf5, with      |
   |                                  | parallel io capability for       |
   |                                  | compatible binary IO output, and |
   |                                  | is equivalent to the following:  |
   |                                  | ``+io                            |
   |                                  |  +parallelIO +hdf5 typeIO=True`` |
   +----------------------------------+----------------------------------+
   | ``+nolog``                       |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation without log           |
   |                                  | capability it is equivalent to   |
   |                                  | the following:                   |
   +----------------------------------+----------------------------------+
   | ``+grid``                        |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation with the Grid unit,   |
   |                                  | it is equivalent to the          |
   |                                  | following:                       |
   +----------------------------------+----------------------------------+
   | ``+ug``                          |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using a uniform grid, |
   |                                  | it is equivalent to the          |
   |                                  | following:                       |
   +----------------------------------+----------------------------------+
   | ``+pm2``                         |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using |paramesh|2 for   |
   |                                  | the grid, it is equivalent to    |
   |                                  | the following:                   |
   +----------------------------------+----------------------------------+
   | ``+pm40``                        |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using |paramesh|4.0 for |
   |                                  | the grid, it is equivalent to    |
   |                                  | the following:                   |
   +----------------------------------+----------------------------------+
   | ``+pm4dev_clean``                |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using a version of    |
   |                                  | |paramesh| 4 that is closer to the |
   |                                  | version available on             |
   |                                  | sourceforge. It is equivalent    |
   |                                  | to:                              |
   |                                  | ``+grid Grid=P                   |
   |                                  | M4DEV |paramesh|LibraryMode=True`` |
   +----------------------------------+----------------------------------+
   | ``+pm4dev``                      |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using a modified      |
   |                                  | version of |paramesh| 4 that       |
   |                                  | includes a more scalable way of  |
   |                                  | filling the ``surr_blks`` array. |
   |                                  | It is equivalent to:             |
   |                                  | ``+pm4d                          |
   |                                  | ev_clean FlashAvoidOrrery=True`` |
   +----------------------------------+----------------------------------+
   | ``+usm``                         |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a MHD    |
   |                                  | simulation using the unsplit     |
   |                                  | staggered mesh hydro solver, if  |
   |                                  | pure hydro mode is used with the |
   |                                  | USM solver add +pureHydro in the |
   |                                  | setup line. It is equivalent to: |
   |                                  | ``–with-unit=physics/H           |
   |                                  | ydro/HydroMain/unsplit/MHD_Stagg |
   |                                  | eredMesh –without-unit=physics/H |
   |                                  | ydro/HydroMain/split/MHD_8Wave`` |
   +----------------------------------+----------------------------------+
   | ``+pureHydro``                   |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies using    |
   |                                  | pure hydro mode, it is           |
   |                                  | equivalent to:                   |
   |                                  | ``physicsMode=hydro``            |
   +----------------------------------+----------------------------------+
   | ``+splitHydro``                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using a split hydro   |
   |                                  | solver and is equivalent to:     |
   |                                  | ``–uni                           |
   |                                  | t=physics/Hydro/HydroMain/split  |
   |                                  | -without-unit=physics/Hydro/Hydr |
   |                                  | oMain/unsplit SplitDriver=True`` |
   +----------------------------------+----------------------------------+
   | ``+unsplitHydro``                |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using the unsplit     |
   |                                  | hydro solver and is equivalent   |
   |                                  | to:                              |
   |                                  | ``–with-unit=physics/Hydro/H     |
   |                                  | ydroMain/unsplit/Hydro_Unsplit`` |
   +----------------------------------+----------------------------------+
   | ``+uhd``                         |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using the unsplit     |
   |                                  | hydro solver and is equivalent   |
   |                                  | to:                              |
   |                                  | ``–with-unit=physics/Hydro/H     |
   |                                  | ydroMain/unsplit/Hydro_Unsplit`` |
   +----------------------------------+----------------------------------+
   | ``+supportPPMUpwind``            |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using a specific      |
   |                                  | Hydro method that requires an    |
   |                                  | increased number of guard cells, |
   |                                  | this may need to be combined     |
   |                                  | with ``-nxb=... -nyb=... etc.``  |
   |                                  | where the specified blocksize is |
   |                                  | greater than or equal to 12      |
   |                                  | (==2*GUARDCELLS). It is          |
   |                                  | equivalent to:                   |
   |                                  | ``SupportPpmUpwind=True``        |
   +----------------------------------+----------------------------------+
   | ``+cube64``                      |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation with a block size of  |
   |                                  | 64**3, it is equivalent to:      |
   |                                  | ``-nxb=64 -nyb=64 -nzb=64``      |
   +----------------------------------+----------------------------------+
   | ``+cube32``                      |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation with a block size of  |
   |                                  | 32**3, it is equivalent to:      |
   |                                  | ``-nxb=32 -nyb=32 -nzb=32``      |
   +----------------------------------+----------------------------------+
   | ``+cube16``                      |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation with a block size of  |
   |                                  | 16**3, it is equivalent to:      |
   |                                  | ``-nxb=16 -nyb=16 -nzb=16``      |
   +----------------------------------+----------------------------------+
   | ``+ptio``                        |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using particles and   |
   |                                  | IO for uniform grid, it is       |
   |                                  | equivalent to:                   |
   +----------------------------------+----------------------------------+
   | ``+rnf``                         |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut is used for        |
   |                                  | checking |flashx| with            |
   |                                  | rectangular block sizes and      |
   |                                  | non-fixed block size. It is      |
   |                                  | equivalent to:                   |
   +----------------------------------+----------------------------------+
   | ``+nofbs``                       |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using a uniform grid  |
   |                                  | with a non-fixed block size. It  |
   |                                  | is equivalent to:                |
   +----------------------------------+----------------------------------+
   | ``+curvilinear``                 |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using curvilinear     |
   |                                  | geometry. It is equivalent to:   |
   +----------------------------------+----------------------------------+
   | ``+cartesian``                   |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using cartesian       |
   |                                  | geometry. It is equivalent to:   |
   +----------------------------------+----------------------------------+
   | ``+spherical``                   |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using spherical       |
   |                                  | geometry. It is equivalent to:   |
   +----------------------------------+----------------------------------+
   | ``+polar``                       |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using polar geometry. |
   |                                  | It is equivalent to:             |
   +----------------------------------+----------------------------------+
   | ``+cylindrical``                 |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using cylindrical     |
   |                                  | geometry. It is equivalent to:   |
   +----------------------------------+----------------------------------+
   | ``+ptdens``                      |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using passive         |
   |                                  | particles initialized by         |
   |                                  | density. It is equivalent to:    |
   |                                  | ``-without-unit=Particles/P      |
   |                                  | articlesInitialization/Lattice`` |
   |                                  | ``-without-u                     |
   |                                  | nit=Particles/ParticlesInitializ |
   |                                  | ation/WithDensity/CellMassBins`` |
   |                                  | `                                |
   |                                  | `-unit=Particles/ParticlesMain`` |
   |                                  | ``-unit=Particles/Parti          |
   |                                  | clesInitialization/WithDensity`` |
   |                                  | ``-particlemethods=              |
   |                                  | TYPE=passive,INIT=With_Density`` |
   +----------------------------------+----------------------------------+
   | ``+npg``                         |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using                 |
   |                                  | NO_PERMANENT_GUARDCELLS mode in  |
   |                                  | |paramesh|4. It is equivalent to:  |
   +----------------------------------+----------------------------------+
   | ``+mpole``                       |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | smilulation using multipole      |
   |                                  | gravity, it is equivalent to:    |
   +----------------------------------+----------------------------------+
   | ``+longrange``                   |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using long range      |
   |                                  | active particles. It is          |
   |                                  | equivalent to:                   |
   +----------------------------------+----------------------------------+
   | ``+gravPfftNofbs``               |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using FFT based       |
   |                                  | gravity solve on a uniform grid  |
   |                                  | with no fixed block size. It is  |
   |                                  | equivalent to:                   |
   |                                  | ``                               |
   |                                  | +ug +nofbs -with-unit=physics/Gr |
   |                                  | avity/GravityMain/Poisson/Pfft`` |
   +----------------------------------+----------------------------------+
   | ``+gravMgrid``                   |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using a multigrid     |
   |                                  | based gravity solve. It is       |
   |                                  | equivalent to:                   |
   |                                  | ``                               |
   |                                  | +pm40 -with-unit=physics/Gravity |
   |                                  | /GravityMain/Poisson/Multigrid`` |
   +----------------------------------+----------------------------------+
   | ``+gravMpole``                   |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | smilulation using multipole      |
   |                                  | gravity, it is equivalent to:    |
   +----------------------------------+----------------------------------+
   | ``+noDefaultMpole``              |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation \*not\* using the     |
   |                                  | multipole based gravity solve.   |
   |                                  | It is equivalent to:             |
   +----------------------------------+----------------------------------+
   | ``+noMgrid``                     |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation \*not\* using the     |
   |                                  | multigrid based gravity solve.   |
   |                                  | It is equivalent to:             |
   +----------------------------------+----------------------------------+
   | ``+newMpole``                    |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using the new         |
   |                                  | multipole based gravity solve.   |
   |                                  | It is equivalent to:             |
   +----------------------------------+----------------------------------+
   | ``+pic``                         |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies use of   |
   |                                  | proper particle units to perform |
   |                                  | PIC (particle in cell) method.   |
   |                                  | It is equivalent to:             |
   |                                  | ``+ug -unit=Grid/G               |
   |                                  | ridParticles/GridParticlesMove`` |
   |                                  | ``-without-unit=Grid/Grid        |
   |                                  | Particles/GridParticlesMove/UG`` |
   |                                  | ``-wi                            |
   |                                  | thout-unit=Grid/GridParticles/Gr |
   |                                  | idParticlesMove/UG/Directional`` |
   +----------------------------------+----------------------------------+
   | ``Grid``                         |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This setup variable can be used  |
   |                                  | to specify which gridding        |
   |                                  | package to use in a simulation:  |
   |                                  | Name: ``Grid`` Type: ``String``  |
   |                                  | Values: ``PM4DEV``, ``PM40``,    |
   |                                  | ``UG``                           |
   +----------------------------------+----------------------------------+
   | ``IO``                           |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This setup variable can be used  |
   |                                  | to specify which IO package to   |
   |                                  | use in a simulation: Name:       |
   |                                  | ``IO`` Type: ``String`` Values:  |
   |                                  | ``hdf5, pnetc                    |
   |                                  | df, MPIHybrid, MPIDump, direct`` |
   +----------------------------------+----------------------------------+
   | ``parallelIO``                   |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This setup variable can be used  |
   |                                  | to specify which type of IO      |
   |                                  | strategy will be used. A         |
   |                                  | “parallel” strategy will be used |
   |                                  | if the value is true, a “serial” |
   |                                  | strategy otherwise. Name:        |
   |                                  | ``parallelIO`` Type: ``Boolean`` |
   |                                  | Values: ``True, False``          |
   +----------------------------------+----------------------------------+
   | ``fixedBlockSize``               |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This setup variable indicates    |
   |                                  | whether or not a fixed block     |
   |                                  | size is to be used. This         |
   |                                  | variable should not be assigned  |
   |                                  | explicitly on the command line.  |
   |                                  | It defaults to ``True``, and the |
   |                                  | setup options ``-nofbs`` and     |
   |                                  | ``-fbs`` modify the value of     |
   |                                  | this variable. Name:             |
   |                                  | ``fixedBlockSize`` Type:         |
   |                                  | ``Boolean`` Values:              |
   |                                  | ``True, False``                  |
   +----------------------------------+----------------------------------+
   | ``nDim``                         |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This setup variable gives the    |
   |                                  | dimensionality of a simulation.  |
   |                                  | This variable should not be set  |
   |                                  | explicitly on the command line,  |
   |                                  | it is automatically set by the   |
   |                                  | setup options ``-1d``, ``-2d``,  |
   |                                  | and ``-3d``. Name: ``nDim``      |
   |                                  | Type: ``integer`` Values:        |
   |                                  | ``1,2,3``                        |
   +----------------------------------+----------------------------------+
   | ``GridIndexOrder``               |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This setup variable indicates    |
   |                                  | whether the ``-index-reorder``   |
   |                                  | setup option is in effect. This  |
   |                                  | variable should not be assigned  |
   |                                  | explicitly on the command line.  |
   |                                  | Name: ``GridIndexOrder`` Type:   |
   |                                  | ``Boolean`` Values:              |
   |                                  | ``True, False``                  |
   +----------------------------------+----------------------------------+
   | ``nxb``                          |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This setup variable gives the    |
   |                                  | number of zones in a block in    |
   |                                  | the X direction. This variable   |
   |                                  | should not be assigned           |
   |                                  | explicitly on the command line,  |
   |                                  | it is automatically set by the   |
   |                                  | setup option ``-nxb``. Name:     |
   |                                  | ``nxb`` Type: ``integer``        |
   +----------------------------------+----------------------------------+
   | ``nyb``                          |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This setup variable gives the    |
   |                                  | number of zones in a block in    |
   |                                  | the Y direction. This variable   |
   |                                  | should not be assigned           |
   |                                  | explicitly on the command line,  |
   |                                  | it is automatically set by the   |
   |                                  | setup option ``-nyb``. Name:     |
   |                                  | ``nyb`` Type: ``integer``        |
   +----------------------------------+----------------------------------+
   | ``nzb``                          |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This setup variable gives the    |
   |                                  | number of zones in a block in    |
   |                                  | the Z direction. This variable   |
   |                                  | should not be assigned           |
   |                                  | explicitly on the command line,  |
   |                                  | it is automatically set by the   |
   |                                  | setup option ``-nzb``. Name:     |
   |                                  | ``nzb`` Type: ``integer``        |
   +----------------------------------+----------------------------------+
   | ``maxBlocks``                    |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This setup variable gives the    |
   |                                  | maximum number of blocks per     |
   |                                  | processor. This variable should  |
   |                                  | not be assigned explicitly on    |
   |                                  | the command line, it is          |
   |                                  | automatically set by the setup   |
   |                                  | option ``-maxblocks``. Name:     |
   |                                  | ``maxBlocks`` Type: ``integer``  |
   +----------------------------------+----------------------------------+
   | ``|paramesh|LibraryMode``          |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | If true, the setup script will   |
   |                                  | generate file                    |
   |                                  | ``amr_runtime_parameters`` from  |
   |                                  | template                         |
   |                                  | ``amr_runtime_parameters.tpl``   |
   |                                  | found in either the object       |
   |                                  | directory (preferred) or the     |
   |                                  | setup script (bin) directory.    |
   |                                  | Selects whether |paramesh|4 should |
   |                                  | be compiled in LIBRARY mode,     |
   |                                  | i.e., with the preprocessor      |
   |                                  | symbol LIBRARY defined. Name:    |
   |                                  | ``|paramesh|LibraryMode`` Type:    |
   |                                  | ``Boolean`` Values:              |
   |                                  | ``True, False``                  |
   +----------------------------------+----------------------------------+
   | ``PfftSolver``                   |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | PfftSolver selects a PFFT solver |
   |                                  | variant when the hybrid (*i.e.*, |
   |                                  | Multigrid with PFFT) Poisson     |
   |                                  | solver is used. Name:            |
   |                                  | ``PfftSolver`` Type: ``String``  |
   |                                  | Values: ``DirectSolver``         |
   |                                  | (default), ``HomBcTrigSolver``,  |
   |                                  | others (unsupported) if          |
   |                                  | recognized in                    |
   |                                  | ``source/Grid/GridSolvers/Mult   |
   |                                  | igrid/PfftTopLevelSolve/Config`` |
   +----------------------------------+----------------------------------+
   | ``SplitDriver``                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | If True, a ``Split`` ``Driver``  |
   |                                  | implementation is requested.     |
   |                                  | Name: ``SplitDriver`` Type:      |
   |                                  | ``Boolean``                      |
   +----------------------------------+----------------------------------+
   | ``Mtmmmt``                       |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | Automatically set ``True`` by    |
   |                                  | ``+mtmmmt`` shortcut. When true, |
   |                                  | this option activates the MTMMMT |
   |                                  | EOS. Name: ``Mtmmmt`` Type:      |
   |                                  | ``Boolean``                      |
   +----------------------------------+----------------------------------+
   | ``mgd_meshgroups``               |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | mgd_meshgroups \* meshCopyCount  |
   |                                  | sets the MAXIMUM number of       |
   |                                  | radiation groups that can be     |
   |                                  | used in a simulation. The ACTUAL |
   |                                  | number of groups (which must be  |
   |                                  | less than mgd_meshgroups \*      |
   |                                  | meshCopyCount) is set by the     |
   |                                  | rt_mgdNumGroups runtime          |
   |                                  | parameter. Name:                 |
   |                                  | ``mgd_meshgroups`` Type:         |
   |                                  | ``Integer``                      |
   +----------------------------------+----------------------------------+
   | ``species``                      |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This setup variable can be used  |
   |                                  | as an alternative specifying     |
   |                                  | species using the SPECIES Config |
   |                                  | file directive by listing the    |
   |                                  | species in the setup command.    |
   |                                  | Some units, like the             |
   |                                  | Multispecies Opacity unit, will  |
   |                                  | ONLY work when the species setup |
   |                                  | variable is set. This is because |
   |                                  | they use the species name to     |
   |                                  | automatically create runtime     |
   |                                  | paramters which include the      |
   |                                  | species names. Name: ``species`` |
   |                                  | Type: ``String``, comma          |
   |                                  | seperated list of strings        |
   |                                  | (*e.g.*, ``species=air,sf6)``    |
   +----------------------------------+----------------------------------+
   | ``ed_maxPulses``                 |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | Name: ``ed_maxPulses`` Type:     |
   |                                  | ``integer`` Remark: Maximum      |
   |                                  | number of laser pulses (defaults |
   |                                  | to 5)                            |
   +----------------------------------+----------------------------------+
   | ``ed_maxBeams``                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | Name: ``ed_maxBeams`` Type:      |
   |                                  | ``integer`` Remark: Maximum      |
   |                                  | number of laser beams (defaults  |
   |                                  | to 6)                            |
   +----------------------------------+----------------------------------+
   | ``threadHydroBlockList``         |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This is used to turn on block    |
   |                                  | list OPENMP threading of hydro   |
   |                                  | routines. Name:                  |
   |                                  | ``threadHydroBlockList`` Type:   |
   |                                  | ``Boolean`` Values:              |
   |                                  | ``True, False``                  |
   +----------------------------------+----------------------------------+
   | ``threadMpoleBlockList``         |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This is used to turn on block    |
   |                                  | list OPENMP threading of the     |
   |                                  | multipole routine. Name:         |
   |                                  | ``threadMpoleBlockList`` Type:   |
   |                                  | ``Boolean`` Values:              |
   |                                  | ``True, False``                  |
   +----------------------------------+----------------------------------+
   | ``threadRayTrace``               |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This is used to turn on block    |
   |                                  | list OPENMP threading of Enery   |
   |                                  | Deposition source term routines. |
   |                                  | Name: ``threadRayTrace`` Type:   |
   |                                  | ``Boolean`` Values:              |
   |                                  | ``True, False``                  |
   +----------------------------------+----------------------------------+
   | ``threadHydroWithinBlock``       |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This is used to turn on within   |
   |                                  | block OPENMP threading of hydro  |
   |                                  | routines. Name:                  |
   |                                  | ``threadHydroWithinBlock`` Type: |
   |                                  | ``Boolean`` Values:              |
   |                                  | ``True, False``                  |
   +----------------------------------+----------------------------------+
   | ``threadEosWithinBlock``         |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This is used to turn on within   |
   |                                  | block OPENMP threading of Eos    |
   |                                  | routines. Name:                  |
   |                                  | ``threadEosWithinBlock`` Type:   |
   |                                  | ``Boolean`` Values:              |
   |                                  | ``True, False``                  |
   +----------------------------------+----------------------------------+
   | ``threadMpoleWithinBlock``       |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This is used to turn on within   |
   |                                  | block OPENMP threading of then   |
   |                                  | multipole routine. Name:         |
   |                                  | ``threadMpoleWithinBlock`` Type: |
   |                                  | ``Boolean`` Values:              |
   |                                  | ``True, False``                  |
   +----------------------------------+----------------------------------+

.. _`Sec:SetupShortcuts`:

Using Shortcuts
---------------

Apart from the various setup options the ``setup`` script also allows
you to use shortcuts for frequently used combinations of options. For
example, instead of typing in

.. container:: codeseg

   ./setup -a Sod -with-unit=Grid/GridMain/UG

you can just type

.. container:: codeseg

   ./setup -a Sod +ug

The ``+ug`` or any setup option starting with a ‘+’ is considered as a
shortcut. By default, setup looks at ``bin/setup_shortcuts.txt`` for a
list of declared shortcuts. You can also specify a ":" delimited list of
files in the environment variable ``SETUP_SHORTCUTS`` and ``setup`` will
read all the files specified (and ignore those which don’t exist) for
shortcut declarations. See for an example file.

.. container:: fcodeseg

   # comment line

   # each line is of the form # shortcut:arg1:arg2:...: # These
   shortcuts can refer to each other.

   default:–with-library=mpi:-unit=IO/IOMain:-gridinterpolation=monotonic

   # io choices noio:–without-unit=IO/IOMain: io:–with-unit=IO/IOMain:

   # Choice of Grid ug:-unit=Grid/GridMain/UG:
   pm2:-unit=Grid/GridMain/paramesh/|paramesh|2:
   pm40:-unit=Grid/GridMain/paramesh/paramesh4/|paramesh|4.0:
   pm4dev:-unit=Grid/GridMain/paramesh/paramesh4/|paramesh|4dev:

   # frequently used geometries cube64:-nxb=64:-nyb=64:-nzb=64:

The shortcuts are replaced by their expansions in place, so options
which come after the shortcut override (or conflict with) options
implied by the shortcut. A shortcut can also refer to other shortcuts as
long as there are no cyclic references.

The “default" shortcut is special. ``setup`` always prepends
``+default`` to its command line thus making ``./setup -a Sod``
equivalent to ``./setup +default -a Sod``. Thus changing the default IO
to “hdf5/parallel", is as simple as changing the definition of the
“default" shortcut.

Some of the more commonly used shortcuts are described below:

.. container:: center

   .. container::
      :name: Tab:setup_shortcuts

      .. table::  Shortcuts for often-used options

         ============ ===================================================
         Shortcut     Description
         ============ ===================================================
         +cartesian   use cartesian geometry
         +cylindrical use cylindrical geometry
         +noio        omit IO
         +nolog       omit logging
         +pm4dev      use the PARAMESH4DEV grid
         +polar       use polar geometry
         +spherical   use spherical geometry
         +ug          use the uniform grid in a fixed block size mode
         +nofbs       use the uniform grid in a non-fixed block size mode
         +usm         use the Unsplit Staggered Mesh MHD solver
         +8wave       use the 8-wave MHD solver
         +splitHydro  use a split Hydro solver
         ============ ===================================================

.. container:: center

   .. container::
      :name: Tab:setup_shortcuts_hedp

      .. table::  Shortcuts for HEDP options

         ======== =================================================
         Shortcut Description
         ======== =================================================
         +mtmmmt  Use the 3-T, multimaterial, multitype EOS
         +uhd3t   Use the 3-T version of Unsplit Hydro
         +usm3t   Use the 3-T version of Unsplit Staggered Mesh MHD
         +mgd     Use Multigroup Radiation Diffusion and Opacities
         +laser   Use the Laser Ray Trace package
         ======== =================================================

.. _`Sec:setupvariables`:

Setup Variables and Preprocessing ``Config`` Files
--------------------------------------------------

``setup`` allows you to assign values to “Setup Variables”. These
variables can be string-valued, integer-valued, or boolean. A ``setup``
call like

.. container:: codeseg

   ./setup -a Sod Foo=Bar Baz=True

sets the variable “Foo" to string “Bar" and “Baz" to boolean True [5]_.
``setup`` can conditionally include and exclude parts of the ``Config``
file it reads based on the values of these variables. For example, the
``IO/IOMain/hdf5/Config`` file contains

.. container:: shrink

   .. container:: fcodeseg

      DEFAULT serial

      USESETUPVARS parallelIO

      IF parallelIO DEFAULT parallel ENDIF

The code sets IO to its default value of “serial” and then resets it to
“parallel" if the setup variable “parallelIO" is True. The
``USESETUPVARS`` keyword in the ``Config`` file instructs setup that the
specified variables must be defined; undefined variables will be set to
the empty string.

Through judicious use of setup variables, the user can ensure that
specific implementations are included or the simulation is properly
configured. For example, the setup line ``./setup -a Sod +ug`` expands
to ``./setup -a Sod -unit=Grid/GridMain/ Grid=UG``. The relevant part of
the ``Grid/GridMain/Config`` file is given below:

.. container:: shrink

   .. container:: fcodeseg

      # Requires use of the Grid SetupVariable USESETUPVARS Grid

      DEFAULT paramesh

      IF Grid==’UG’ DEFAULT UG ENDIF IF Grid==’PM2’ DEFAULT
      paramesh/|paramesh|2 ENDIF

The ``Grid/GridMain/Config`` file defaults to choosing ``PARAMESH``. But
when the setup variable Grid is set to “UG" through the shortcut
``+ug``, the default implementation is set to “UG". The same technique
is used to ensure that the right IO unit is automatically included.

See ``bin/Readme.SetupVars`` for an exhaustive list of Setup Variables
which are used in the various Config files. For example the setup
variable ``nDim`` can be test to ensure that a simulation is configured
with the appropriate dimensionality (see for example
``Simulation/SimulationMain/unitTest/Eos/Config``).

.. _`Sec:Config`:

``Config`` Files
----------------

Information about unit dependencies, default sub-units, runtime
parameter definitions, library requirements, and physical variables,
etc. is contained in plain text files named ``Config`` in the different
unit directories. These are parsed by ``setup`` when configuring the
source tree and are used to create the code needed to register unit
variables, to implement the runtime parameters, to choose specific
sub-units when only a generic unit has been specified, to prevent
mutually exclusive units from being included together, and to flag
problems when dependencies are not resolved by some included unit. Some
of the Config files contain additional information about unit
interrelationships. As mentioned earlier, ``setup`` starts from the
``Config`` file in the Simulation directory of the problem being built.

.. _`Sec:ConfigFileSyntax`:

Configuration file syntax
~~~~~~~~~~~~~~~~~~~~~~~~~

Configuration files come in two syntactic flavors: static text and
python. In static mode, configuration directives are listed as lines in
a plain text file. This mode is the most readable and intuitive of the
two, but it lacks flexibility. The python mode has been introduced to
circumvent this inflexibility by allowing the configuration file author
to specify the configuration directives as a function of the setup
variables with a python procedure. This allows the content of each
directive and the number of directives in total to be amenable to
general programming.

The rule the setup script uses for deciding which flavor of
configuration file it’s dealing with is simple. Python configuration
files have as their first line ``##python:genLines``. If the first line
does not match this string, then static mode is assumed and each line of
the file is interpreted verbatim as a directive.

If python mode is triggered, then the entire file is considered as valid
python source code (as if it were a .py). From this python code, a
function of the form ``def genLines(setupvars)`` is located and executed
to generate the configuration directives as an array (or any iterable
collection) of strings. The sole argument to genLines is a dictionary
that maps setup variable names to their corresponding string values.

As an example, here is a configuration file in python mode that
registers runtime parameters named indexed_parameter_x where x ranges
from 1 to NP and NP is a setup line variable.

.. container:: fcodeseg

   ##python:genLines

   # We define genLines as a generator with the very friendly "yield"
   syntax. # Alternatively, we could have genLines return an array of
   strings or even # one huge multiline string. def genLines(setupvars):
   # emit some directives that dont depend on any setup variables yield
   """ REQUIRES Driver REQUIRES physics/Hydro REQUIRES physics/Eos """ #
   read a setup variable value from the dictionary np =
   int(setupvars("NP")) # must be converted from a string # loop from 0
   to np-1 for x in xrange(np): yield "PARAMETER indexed_parameter_%d
   REAL 0." % (x+1)

When setting up a problem with NP=5 on the setup command line, the
following directives will be processed:

.. container:: fcodeseg

   REQUIRES Driver REQUIRES physics/Hydro REQUIRES physics/Eos PARAMETER
   indexed_parameter_1 REAL 0. PARAMETER indexed_parameter_2 REAL 0.
   PARAMETER indexed_parameter_3 REAL 0. PARAMETER indexed_parameter_4
   REAL 0. PARAMETER indexed_parameter_5 REAL 0.

.. _`Sec:ConfigDirectives`:

Configuration directives
~~~~~~~~~~~~~~~~~~~~~~~~

The syntax of the configuration directives is described here.
Arbitrarily many spaces and/or tabs may be used, but all keywords must
be in uppercase. Lines not matching an admissible pattern will raise an
error when running setup.

-  | ``# comment``
   | A comment. Can appear as a separate line or at the end of a line.

-  | ``DEFAULT`` *sub-unit*
   | Every unit and sub-unit designates one implementation to be the
     “default”, as defined by the keyword ``DEFAULT`` in its ``Config``
     file. If no specific implementation of the unit or its sub-units is
     selected by the application, the designated default implementation
     gets included. For example, the ``Config`` file for the ``EosMain``
     specifies Gamma as the default. If no specific implementation is
     explicitly included (*i.e.*,
     ``INCLUDE physics/Eos/EosMain/Multigamma``), then this command
     instructs ``setup`` to include the Gamma implementation, as though
     ``INCLUDE physics/Eos/EosMain/Gamma`` had been placed in the
     ``Units`` file.

-  | ``EXCLUSIVE`` *implementation...*
   | Specifies a list of implementations that cannot be included
     together. If no ``EXCLUSIVE`` instruction is given, it is perfectly
     legal to simultaneously include more than one implementation in the
     code. Using “``EXCLUSIVE`` \*” means that at most one
     implementation can be included.

-  | ``CONFLICTS`` *unit1[/sub-unit[/implementation...]]* ...
   | Specifies that the current unit, sub-unit, or specific
     implementation is not compatible with the list of units, sub-units
     or other implementations that follows. ``setup`` issues an error if
     the user attempts a conflicting unit configuration.

-  | ``REQUIRES`` *unit[/sub-unit[/implementation...]] [* ``OR``
     *unit[/sub-unit...]]...*
   | Specifies a unit requirement. Unit requirements can be general,
     without asking for a specific implementation, so that unit
     dependencies are not tied to particular algorithms. For example,
     the statement ``REQUIRES physics/Eos`` in a unit’s ``Config`` file
     indicates to ``setup`` that the physics/Eos unit is needed, but no
     particular equation of state is specified. As long as an ``Eos``
     implementation is included, the dependency will be satisfied. More
     specific dependencies can be indicated by explicitly asking for an
     implementation. For example, if there are multiple species in a
     simulation, the ``Multigamma`` equation of state is the only valid
     option. To ask for it explicitly, use
     ``REQUIRES physics/Eos/EosMain/Multigamma``. Giving a complete set
     of unit requirements is helpful, because ``setup`` uses them to
     generate the units file when invoked with the -auto option.

-  | ``REQUESTS`` *unit[/sub-unit[/implementation...]]*
   | Requests that a unit be added to the Simulation. All requests are
     upgraded to a “REQUIRES” if they are not negated by a
     "-without-unit" option from the command line. If negated, the
     ``REQUEST`` is ignored. This can be used to turn off profilers and
     other “optional” units which are included by default.

-  | ``SUGGEST`` *unitname unitname ...*
   | Unlike ``REQUIRES``, this keyword suggests that the current unit be
     used along with one of the specified units. The setup script will
     print details of the suggestions which have been ignored. This is
     useful in catching inadvertently omitted units before the run
     starts, thus avoiding a waste of computing resources.

-  | ``PARAMETER`` *name type [``CONSTANT``] default* [*range-spec*]
   | Specifies a runtime parameter. Parameter names are unique up to 20
     characters and may not contain spaces. Admissible types include
     ``REAL``, ``INTEGER``, ``STRING``, and ``BOOLEAN``. Default values
     for ``REAL`` and ``INTEGER`` parameters must be valid numbers, or
     the compilation will fail. Default ``STRING`` values must be
     enclosed in double quotes (``"``). Default ``BOOLEAN`` values must
     be ``.true.`` or ``.false.`` to avoid compilation errors. Once
     defined, runtime parameters are available to the entire code.
     Optionally, any parameter may be specified with the ``CONSTANT``
     attribute (*e.g.*, ``PARAMETER foo REAL CONSTANT 2.2``). If a user
     attempts to set a constant parameter via the runtime parameter
     file, an error will occur.

   The range specification is optional and can be used to specify valid
   ranges for the parameters. The range specification is allowed only
   for ``REAL, INTEGER, STRING`` variables and must be enclosed in ’[]’.

   For a ``STRING`` variable, the range specification is a
   comma-separated list of strings (enclosed in quotes). For a
   ``INTEGER, REAL`` variable, the range specification is a
   comma-separated list of (closed) intervals specified by
   ``min ... max``, where min and max are the end points of the
   interval. If min or max is omitted, it is assumed to be
   :math:`-\infty` and :math:`+\infty` respectively. Finally ``val`` is
   a shortcut for ``val ... val``. For example

   .. container:: codeseg

      PARAMETER pres REAL 1.0 [ 0.1 ... 9.9, 25.0 ... ] PARAMETER coords
      STRING "polar" ["polar","cylindrical","2d","3d"]

   indicates that ``pres`` is a REAL variable which is allowed to take
   values between 0.1 and 9.9 or above 25.0. Similarly ``coords`` is a
   string variable which can take one of the four specified values.

-  | ``D`` *parameter-name* *comment*
   | Any line in a ``Config`` file is considered a parameter comment
     line if it begins with the token ``D``. The first token after the
     comment line is taken to be the parameter name. The remaining
     tokens are taken to be a description of the parameter’s purpose. A
     token is delineated by one or more white spaces. For example,

   .. container:: codeseg

      D SOME_PARAMETER The purpose of this parameter is whatever

   If the parameter comment requires additional lines, the ``&`` is used
   to indicate continuation lines. For example,

   .. container:: codeseg

      D SOME_PARAMETER The purpose of this parameter is whatever D &
      This is a second line of description

   You can also use this to describe other variables, fluxes, species,
   etc. For example, to describe a species called "xyz", create a
   comment for the parameter “xyz_species”. In general the name should
   be followed by an underscore and then by the lower case name of the
   keyword used to define the name.

   Parameter comment lines are special because they are used by
   ``setup`` to build a formatted list of commented runtime parameters
   for a particular problem. This information is generated in the file
   ``setup_params`` in the ``object`` directory.

-  | ``VARIABLE`` *name* *[``TYPE:`` vartype]* *[eosmap-spec]*
   | Registers variable with the framework with name *name* and a
     variable type defined by *vartype*. The ``setup`` script collects
     variables from all the included units, and creates a comprehensive
     list with no duplications. It then assigns defined constants to
     each variable and calculates the amount of storage required in the
     data structures for storing these variables. The defined constants
     and the calculated sizes are written to the file ``Simulation.h``.

   The possible types for *vartype* are as follows:

   -  | ``PER_VOLUME``
      | This solution variable is represented in **conserved** form,
        *i.e.*, it represents the density of a conserved extensive
        quantity. The prime example is a variable directly representing
        mass density. Energy densities, momentum densities, and partial
        mass densities would be other examples (but these quantities are
        usually represented in ``PER_MASS`` form instead).

   -  | ``PER_MASS``
      | This solution variable is represented in **mass-specific** form,
        *i.e.*, it represents quantities whose nature is
        :math:`\hbox{extensive quantity}\,\mathop{\mathrm{per}}\,\hbox{mass unit}`.
        Examples are specific energies, velocities of material (since
        they are equal to momentum per mass unit), and abundances or
        mass fractions (partial density divided by density).

   -  | ``GENERIC``
      | This is the default *vartype* and need not be specified. This
        type should be used for any variables that do not clearly belong
        to one of the previous two categories.

   In the current version of the code, the ``TYPE`` attribute is only
   used to determine which variables should be converted to conservative
   form for certain ``Grid`` operations that may require interpolation
   (*i.e.*, prolongation, guardcell filling, and restriction) when one
   of the runtime parameters ``Grid/convertToConsvdForMeshCalls`` or
   ``Grid/convertToConsvdInMeshInterp`` is set ``true``. Only variables
   of type ``PER_MASS`` are converted: values are multiplied
   cell-by-cell with the value of the ``"dens"`` variable, and potential
   interpolation results are converted back by cell-by-cell division by
   ``"dens"`` values after interpolation.

   Note that therefore

   -  variable types are irrelevant for uniform grids,

   -  variable types are irrelevant if neither
      ``Grid/convertToConsvdForMeshCalls`` nor
      ``Grid/convertToConsvdInMeshInterp`` is ``true``, and

   -  variable types (and conversion to and from conserved form) only
      take effect if a

      .. container:: codeseg

         VARIABLE dens ...

      exists.

   An *eosmap-spec* has the syntax *``EOSMAP:`` eos-role* :math:`|` *(
   [``EOSMAPIN:`` eos-role] [``EOSMAPOUT:`` eos-role ])*, where
   *eos-role* stands for a **role** as defined in ``Eos_map.h``. These
   roles are used within implementations of the
   ``physics/Eos/Eos_wrapped`` interface, via the subroutines
   ``physics/Eos/Eos_getData`` and ``physics/Eos/Eos_putData``, to map
   variables from ``Grid`` data structures to the ``eosData`` array that
   ``physics/Eos/Eos`` understands, and back. For example,

   .. container:: codeseg

      VARIABLE eint TYPE: PER_MASS EOSMAPIN: EINT

   means that within ``Eos_wrapped``, the ``EINT_VAR`` component of
   ``unk`` will be treated as the grid variable in the “internal energy”
   role for the purpose of constructing input to ``physics/Eos/Eos``,
   and

   .. container:: codeseg

      VARIABLE gamc EOSMAPOUT: GAMC

   means that within ``Eos_wrapped``, the ``GAMC_VAR`` component of
   ``unk`` will be treated as the grid variable in the ``EOSMAP_GAMC``
   role for the purpose of returning results from calling
   ``physics/Eos/Eos`` to the grid. The specification

   .. container:: codeseg

      VARIABLE pres EOSMAP: PRES

   has the same effect as

   .. container:: codeseg

      VARIABLE pres EOSMAPIN: PRES EOSMAPOUT: PRES

   Note that not all roles defined in ``Eos_map.h`` are necessarily
   meaningful or actually used in a given ``Eos`` implementation. An
   *eosmap-spec* for a ``VARIABLE`` is only used in an
   ``physics/Eos/Eos_wrapped`` invocation when the optional
   ``gridDataStruct`` argument is absent or has a value of ``CENTER``.

-  | ``FACEVAR`` *name* *[eosmap-spec]*
   | This keyword has the same meaning for face-centered variables, that
     ``VARIABLE`` does for cell-centered variables. It allocates space
     in the grid data structure that contains face-centered physical
     variables for “name”. See for more information

   For *eosmap-spec*, see above under ``VARIABLE``. An *eosmap-spec* for
   ``FACEVAR`` is only used when ``physics/Eos/Eos_wrapped`` is called
   with an optional ``gridDataStruct`` argument of ``FACEX``, ``FACEY``,
   or ``FACEZ``.

-  | ``FLUX`` *name*
   | Registers flux variable *name* with the framework. When using an
     adaptive mesh, flux conservation is needed at fine-coarse
     boundaries. ``PARAMESH`` uses a data structure for this purpose,
     the flux variables provide indices into that data structure. See
     for more information.

-  | ``SCRATCHCENTERVAR`` *name* *[eosmap-spec]*
   | This keyword is used in connection with the grid scope scratch
     space for cell-centered data supported by |flashx|. It allows the
     user to ask for scratch space with “name”. The scratch variables do
     not participate in the process of guardcell filling, and their
     values become invalid after a grid refinement step. While users can
     define scratch variables to be written to the plotfiles, they are
     not by default written to checkpoint files. Note this feature
     wasn’t available in |flashx|2. See for more information.

-  | ``SCRATCHFACEVAR`` *name* *[eosmap-spec]*
   | This keyword is used in connection with the grid scope scratch
     space for face-centered data, it is identical in every other
     respect to ``SCRATCHCENTERVAR``.

-  | ``SCRATCHVAR`` *name* *[eosmap-spec]*
   | This keyword is used for specifying instances of general purpose
     grid scope scratch space. The same space can support cell-centered
     as well as face-centered data. Like other scratch data structures,
     the variables in this data structure can also be asked with “name”
     and do not participate in guardcell filling.

   For *eosmap-spec*, see above under ``VARIABLE``. An *eosmap-spec* for
   ``SCRATCHVAR`` is only used when ``physics/Eos/Eos_wrapped`` is
   called with an optional ``gridDataStruct`` argument of ``SCRATCH``.

-  | ``MASS_SCALAR`` *name* *[RENORM: group-name]* *[eosmap-spec]*
   | If a quantity is defined with keyword MASS_SCALAR, space is created
     for it in the grid “unk” data structure. It is treated like any
     other variable by ``PARAMESH``, but the hydrodynamic unit treats it
     differently. It is advected, but other physical characteristics
     don’t apply to it. If the optional “RENORM” is given, this
     mass-scalar will be added to the renormalization group of the
     accompanying group name. The hydrodynamic solver will renormalize
     all mass-scalars in a given group, ensuring that all variables in
     that group will sum to 1 within an individual cell. See

   For *eosmap-spec*, see above under ``VARIABLE``. An *eosmap-spec* for
   a ``MASS_SCALAR`` may be used in an ``physics/Eos/Eos_wrapped``
   invocation when the optional ``gridDataStruct`` argument is absent or
   has a value of ``CENTER``.

   .. container:: flashtip

      It is inadvisable to name variables, species, and mass scalars
      with the same prefix, as post-processing routines have difficulty
      deciphering the type of data from the output files. For example,
      don’t create a variable “temp” to hold temperature and a mass
      scalar “temp” indicating a temporary variable. Although the
      ``Simulation.h`` file can distinguish between these two types of
      variables, many plotting routines cannot.

-  | ``PARTICLETYPE`` *particle-type* ``INITMETHOD``
     *initialization-method* ``MAPMETHOD`` *map-method* ``ADVMETHOD``
     *time-advance-method*
   | This keyword associates a **particle type** with mapping and
     initialization sub-units of ``Particles`` unit to operate on this
     particle type during the simulation. Here, *map-method* describes
     the method used to map the particle properties to and from the mesh
     (see ), *initialization-method* describes the method used to
     distribute the particles at initialization, and
     *time-advance-method* describes the method used to advance the
     associated particle type in time (see , and in general ). This
     keyword has been introduced to facilitate inclusion of multiple
     particle types in the same simulation. It imposes certain
     requirements on the use of the ``ParticlesMapping`` and
     ``ParticlesInitialization`` subunits. Particles (of any type,
     whether called ``passive`` or anything else) do not have default
     methods for initialization, mapping, or time integration, so a
     ``PARTICLETYPE`` directive in a ``Config`` file (or an equivalent
     ``-particlemethods=`` setup option, see ) is the only way to
     specify the appropriate implementations of the ``Particles``
     subunits to be used. The declaration should be accompanied by
     appropriate “REQUESTS” or “REQUIRES” directives to specify the
     paths of the appropriate subunit implementation directories to be
     included. For clarity, our technique has been to include this
     information in the simulation directory ``Config`` files only. All
     the currently available mapping and initialization methods have a
     corresponding identifier in the form of preprocessor definition in
     ``Particles.h``. The user may select any *particle-type* name, but
     the *map-method*, *initialization-method* and *time-advance-method*
     must correspond to existing identifiers defined in ``Particles.h``.
     This is necessary to navigate the data structure that stores the
     particle type and its associated mapping and initialization
     methods. Users desirous of adding new methods for mapping or
     initialization should also update the ``Particles.h`` file with
     additional identifiers and their preprocessor definitions. Note, it
     is possible to use the same methods for different particle types,
     but each particle type name must only appear once. Finally, the
     Simulations ``Config`` file is also expected to request appropriate
     implementations of mapping and initialization subunits using the
     keyword ``REQUESTS``, since the corresponding Config files do not
     specify a default implementation to include. For example, to
     include ``passive`` particle types with ``Quadratic`` mapping,
     ``Lattice`` initialization,and ``Euler`` for advancing in time the
     following code segment should appear in the ``Config`` file of the
     ``Simulations`` directory.

   .. container:: codeseg

      PARTICLETYPE passive INITMETHOD lattice MAPMETHOD quadratic
      ADVMETHOD Euler REQUIRES Particles/ParticlesMain REQUESTS
      Particles/ParticlesMain/passive/Euler REQUESTS
      Particles/ParticlesMapping/Quadratic REQUESTS
      Particles/ParticlesInitialization/Lattice

-  | ``PARTICLEPROP`` *name* *type*
   | This keyword indicates that the particles data structure will
     allocate space for a sub-variable “NAME_PART_PROP.” For example if
     the Config file contains

   .. container:: codeseg

      PARTICLEPROP dens

   then the code can directly access this property as

   .. container:: codeseg

      particles(DENS_PART_PROP,1:localNumParticles) = densInitial

   *type* may be REAL or INT, however INT is presently unused. See for
   more information and examples.

-  | ``PARTICLEMAP`` TO *partname* FROM *vartype* *varname*
   | This keyword maps the value of the particle property *partname* to
     the variable *varname*. *vartype* can take the values VARIABLE,
     MASS_SCALAR, SPECIES, FACEX, FACEY, FACEZ, or one of SCRATCH types
     (SCRATCHVAR/ SCRATCHCENTERVAR, SCRATCHFACEXVAR. SCRATCHFACEYVAR,
     SCRATCHFACEZVAR) These maps are used to generate
     ``Simulation_mapParticlesVar``, which takes the particle property
     *partname* and returns *varname* and *vartype*. For example, to
     have a particle property tracing density:

   .. container:: codeseg

      PARTICLEPROP dens REAL PARTICLEMAP TO dens FROM VARIABLE dens

   or, in a more advanced case, particle properties tracing some
   face-valued function Mag:

   .. container:: codeseg

      PARTICLEPROP Mag_x REAL PARTICLEPROP Mag_y REAL PARTICLEPROP Mag_z
      REAL PARTICLEMAP TO Mag_x FROM FACEX Mag PARTICLEMAP TO Mag_y FROM
      FACEY Mag PARTICLEMAP TO Mag_z FROM FACEZ Mag

   Additional information on creating ``Config`` files for particles is
   obtained in .

-  | ``SPECIES`` *name* [TO *number of ions*]
   | An application that uses multiple species uses this keyword to
     define them. See for more information. The user may also specify an
     optional number of ions for each element, *name*. For example,
     ``SPECIES`` *o* TO *8* creates 9 spaces in ``unk`` for Oxygen, that
     is, a single space for Oxygen and 8 spaces for each of its ions.
     This is relevant to simulations using the ``ionize`` unit.
     (Omitting the optional ``TO`` specifier is equivalent to specifying
     ``TO`` 0).

-  | ``DATAFILES`` *wildcard*
   | Declares that all files matching the given wildcard in the unit
     directory should be copied over to the object directory. For
     example,

   .. container:: codeseg

      DATAFILES \*.dat

   will copy all the “.dat” files to the object directory.

-  | ``KERNEL`` *[subdir]*
   | Declares that all subdirectories must be recursively included. This
     usually marks the end of the high level architecture of a unit.
     Directories below it may be third party software or a highly
     optimized solver, and are therefore not required to conform to
     |flashx| architecture.

   Without a *subdir*, the current directory (*i.e.*, the one containing
   the ``Config`` file with the ``KERNEL`` keyword) is marked as a
   kernel directory, so code from all its subdirectories (with the
   exception of subdirectories whose name begins with a dot) is
   included. When a *subdir* is given, then that subdirectory must
   exist, and it is treated as a kernel directory in the same way.

   Note that currently the ``setup`` script can process only one
   ``KERNEL`` directive per ``Config`` file.

-  | ``LIBRARY`` *name*
   | Specifies a library requirement. Different |flashx| units require
     different libraries, and they must inform ``setup`` so it can link
     the libraries into the executable. Some valid library names are
     ``HDF5, MPI``. Support for external libraries can be added by
     modifying the site-specific ``Makefile.h`` files to include
     appropriate Makefile macros. It is possible to use internal
     libraries, as well as switch libraries at setup time. To use these
     features, see

-  | ``LINKIF`` *filename unitname*
   | Specifies that the file *filename* should be used only when the
     unit ``unitname`` is included. This keyword allows a unit to have
     multiple implementations of any part of its functionality, even
     down to the kernel level, without the necessity of creating
     children for every alternative. This is especially useful in
     Simulation setups where users may want to use different
     implementations of specific functions based upon the units
     included. For instance, a user may wish to supply his/her own
     implementation of ``Grid_markRefineDerefine.F90``, instead of using
     the default one provided by |flashx|. However, this function is
     aware of the internal workings of ``Grid``, and has different
     implementations for different grid packages. The user could
     therefore specify different versions of his/her own file that are
     intended for use with the different grids. For example, adding

   .. container:: codeseg

      LINKIF Grid_markRefineDerefine.F90.ug Grid/GridMain/UG LINKIF
      Grid_markRefineDerefine.F90.pmesh Grid/GridMain/paramesh

   to the ``Config`` file ensures that if the application is built with
   ``UG``, the file ``Grid_markRefineDerefine.F90.ug`` will be linked in
   as ``Grid_markRefineDerefine.F90``, whereas if it is built with
   ``|paramesh|2`` or ``|paramesh|4.0`` or ``|paramesh|4dev``, then the file
   ``Grid_markRefineDerefine.F90.pmesh`` will be linked in as
   ``Grid_markRefineDerefine.F90``. Alternatively, the user may want to
   provide only one implementation specific to, say, ``PARAMESH``. In
   this case, adding

   .. container:: codeseg

      LINKIF Grid_markRefineDerefine.F90 Grid/GridMain/paramesh

   to the Config file ensures that the user-supplied file is included
   when using ``PARAMESH``\ (either version), while the default |flashx|
   file is included when using ``UG``.

-  | ``PPDEFINE`` *sym1 sym2 ...*
   | Instructs setup to add the PreProcessor symbols ``SYM1`` and
     ``SYM2`` to the generated ``Simulation.h``. Here ``SYM1`` is *sym1*
     converted to uppercase. These pre-process symbols can be used in
     the code to distinguish between which units have been used in an
     application. For example, a Fortran subroutine could include

   .. container:: codeseg

      #ifdef |flashx|_GRID_UG ug specific code #endif

      #ifdef |flashx|_GRID_PARAMESH3OR4 pm3+ specific code #endif

   By convention, many preprocessor symbols defined in Config files
   included in the |flashx| code distribution start with the prefix
   “|flashx|\_”.

-  | ``USESETUPVARS var1, var2, …``
   | This tells ``setup`` that the specified “Setup Variables” are being
     used in this ``Config`` file. The variables initialize to an empty
     string if no values are specified for them. Note that commas are
     required if listing several variables.

-  | ``CHILDORDER`` *child1 child2 …*
   | When ``setup`` links several implementations of the same function,
     it ensures that implementations of children override that of the
     parent. Its method is to lexicographically sort all the names and
     allow implementations occurring later to override those occurring
     earlier. This means that if two siblings implement the same code,
     the names of the siblings determine which implementation wins.
     Although it is very rare for two siblings to implement the same
     function, it does occur. This keyword permits the ``Config`` file
     to override the lexicographic order by one preferred by the user.
     Lexicographic ordering will prevail as usual when deciding among
     implementations that are not explicitly listed.

-  | ``GUARDCELLS`` *num*
   | Allows an application to choose the stencil size for updating grid
     points. The stencil determines the number of guardcells needed. The
     PPM algorithm requires :math:`4` guardcells, hence that is the
     default value. If an application specifies a smaller value, it will
     probably not be able to use the default ``monotonic`` AMR Grid
     interpolation; see the ``-gridinterpolation`` ``setup`` flag for
     additional information.

-  | ``SETUPERROR error message``
   | This causes ``setup`` to abort with the specified error message.
     This is usually used only inside a conditional IF/ENDIF block (see
     below).

-  | ``IF, ELSEIF, ELSE, ENDIF``
   | A conditional block is of the following form:

   .. container:: codeseg

      IF cond ... ELSEIF cond ... ELSE ... ENDIF

   where the ``ELSEIF`` and ``ELSE`` blocks are optional. There is no
   limit on the number of ``ELSEIF`` blocks. “...” is any sequence of
   valid ``Config`` file syntax. The conditional blocks may be nested.
   “cond” is any boolean valued Python expression using the setup
   variables specified in the ``USESETUPVARS``.

-  | ``NONREP`` *unktype* *name* *localmax* *globalparam* *ioformat*
   | Declares an array of ``UNK`` variables that will be partitioned
     across the replicated meshes. Using various preprocessor macros in
     Simulation.h each copy of the mesh can determine at runtime its own
     subset of indexes into this global array. This allows an easy form
     of parallelism where regular "replicated" mesh variables are
     computed redundantly across processors, but the variables in the
     "non-replicated" array are computed in parallel.

   -  | *unktype*: must be either ``MASS_SCALAR`` or ``VARIABLE``

   -  *name*: the name of this variable array. It is suggested that it
      be all capital letters, and must conform to what the C
      preprocessor will consider as a valid symbol for use in a
      ``#define`` statement.

   -  | *localmax*: a positive integer specifying the maximum number of
        elements from the global variable array a mesh can hold. This is
        the actual number of ``UNK`` variables that are allocated on
        each processor, though not all of them will necessarily be used.

   -  | *globalparam*: the name of a runtime parameter which dictates
        the size of this global array of variables.

   -  | *ioformat*: a string representing how the elements of the array
        will be named when written to the output files. The question
        mark character ``?`` is used as a placeholder for the digits of
        the array index. As an example, the format string ``x???`` will
        generate the dataset names ``x001``, ``x002``, ``x003``, etc.
        This string must be no more than four characters in length.

   The number of meshes is dictated by the runtime parameter
   ``meshCopyCount``. The following constraint must be satisfied or
   |flashx| will fail at runtime:

   .. math:: globalparam \le meshCopyCount * localmax

   The reason for this restriction is that ``localmax`` is the maximum
   number of array elements a mesh can be responsible for, and
   ``meshCopyCount`` is the number of meshes, so their product bounds
   the size of the array.

   Example:

   Config file:

   .. container:: fcodeseg

      NONREP MASS_SCALAR A 4 numA a??? NONREP MASS_SCALAR B 5 numB b???

   flash.par file:

   .. container:: fcodeseg

      meshCopyCount = 3 numA = 11 numB = 15

   In this case two non-replicated mass-scalar arrays are defined, ``A``
   and ``B``. Their lengths are specified by the runtime parameters
   ``numA`` and ``numB`` respectively. ``numB`` is set to its maximum
   value of :math:`5*meshCopyCount=15`, but ``numA`` is one less than
   its maximum value of :math:`4*meshCopyCount=12` so at runtime one of
   the meshes will not have all of its ``UNK`` variables in use. The
   dataset names generated by IO will take the form ``a001 …a011`` and
   ``b001 …b015``.

   The preprocessor macros defined in ``Simulation.h`` for these arrays
   will have the prefixes ``A_`` and ``B_`` respectively. For details
   about these macros and how they will distribute the array elements
   across the meshes see .

.. _`Sec:SetupMakefile`:

Creating a Site-specific ``Makefile``
-------------------------------------

If ``setup`` does not find your hostname in the ``sites/`` directory it
picks a default ``Makefile`` based on the operating system. This
``Makefile`` is not always correct but can be used as a template to
create a ``Makefile`` for your machine. To create a Makefile specific to
your system follow these instructions.

-  Create the directory ``sites/<hostname>``, where ``<hostname>`` is
   the hostname of your machine.

-  Start by copying ``os/<your os>/Makefile.h`` to ``sites/<hostname>``

-  Use ``bin/suggestMakefile.sh`` to help identify the locations of
   various libraries on your system. The script scans your system and
   displays the locations of some libraries. You must note the location
   of ``MPI`` library as well. If your compiler is actually an
   mpi-wrapper (*e.g.*\ ``mpif90``), you must still define ``LIB_MPI``
   in your site specific ``Makefile.h`` as the empty string.

-  Edit ``sites/<hostname>/Makefile.h`` to provide the locations of
   various libraries on your system.

-  Edit ``sites/<hostname>/Makefile.h`` to specify the FORTRAN and C
   compilers to be used.

.. container:: flashtip

   The Makefile.h *must* include a compiler flag to promote Fortran
   ``Reals`` to ``Double Precision``. |flashx| performs all ``MPI``
   communication of Fortran ``Reals`` using ``MPI_DOUBLE_PRECISION``
   type, and assumes that Fortran ``Reals`` are interoperable with C
   ``doubles`` in the I/O unit.

Files Created During the ``setup`` Process
------------------------------------------

When ``setup`` is run it generates many files in the ``object``
directory. They fall into three major categories:

-  Files not required to build the |flashx| executable, but which contain
   useful information,

-  Generated F90 or C code, and

-  Makefiles required to compile the |flashx| executable.

Informational files
~~~~~~~~~~~~~~~~~~~

These files are generated before compilation by ``setup``. Each of these
files begins with the prefix ``setup_`` for easy identification.

+---------------------+-----------------------------------------------+
|                     | contains the options with which ``setup`` was |
|                     | called and the command line resulting after   |
|                     | shortcut expansion                            |
+---------------------+-----------------------------------------------+
| \*[1ex]             | contains the list of libraries and their      |
|                     | arguments (if any) which was linked in to     |
| ``setup_libraries`` | generate the executable                       |
+---------------------+-----------------------------------------------+
| \*[1ex]             | contains the list of all units which were     |
|                     | included in the current setup                 |
+---------------------+-----------------------------------------------+
| \*[1ex]             | contains a list of all pre-process symbols    |
|                     | passed to the compiler invocation directly    |
| ``setup_defines``   |                                               |
+---------------------+-----------------------------------------------+
| \*[1ex]             | contains the exact compiler and linker flags  |
+---------------------+-----------------------------------------------+
| \*[1ex]             | contains the list of runtime parameters       |
|                     | defined in the ``Config`` files processed by  |
| ``setup_params``    | ``setup``                                     |
+---------------------+-----------------------------------------------+
| \*[1ex]             | contains the list of variables, fluxes,       |
|                     | species, particle properties, and mass        |
|                     | scalars used in the current setup, together   |
|                     | with their descriptions.                      |
+---------------------+-----------------------------------------------+
| \*[1ex]             |                                               |
+---------------------+-----------------------------------------------+

Code generated by the ``setup`` call
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

These routines are generated by the setup call and provide
simulation-specific code.

+----------------------------------+----------------------------------+
| ``setup_buildstamp.F90``         | contains code for the subroutine |
|                                  | ``setup_buildstamp`` which       |
|                                  | returns the setup and build time |
|                                  | as well as code for              |
|                                  | ``setup_systemInfo`` which       |
|                                  | returns the *uname* of the       |
|                                  | system used to setup the problem |
+----------------------------------+----------------------------------+
| \*[1ex]                          | contains code which returns      |
|                                  | build statistics including the   |
|                                  | actual ``setup`` call as well as |
|                                  | the compiler flags used for the  |
|                                  | build                            |
+----------------------------------+----------------------------------+
| \*[1ex]                          | contains code to retrieve the    |
|                                  | number and list of flashUnits    |
| ``setup_getFlashUnits.F90``      | used to compile code             |
+----------------------------------+----------------------------------+
| \*[1ex]                          | contains code to retrieve the    |
|                                  | version of |flashx| used for the  |
|                                  | build                            |
+----------------------------------+----------------------------------+
| \*[1ex]                          | contains simulation specific     |
|                                  | preprocessor macros, which       |
| ``Simulation.h``                 | change based upon setup unlike   |
|                                  | ``constants.h``. It is described |
|                                  | in                               |
+----------------------------------+----------------------------------+
| \*[1ex]                          | contains code to map an index    |
|                                  | described in ``Simulation.h`` to |
|                                  | a string described in the        |
|                                  | ``Config`` file.                 |
+----------------------------------+----------------------------------+
| \*[1ex]                          | contains code to map a string    |
|                                  | described in the ``Config`` file |
| ``Simulation/Si                  | to an integer index described in |
| mulation_mapStrToInt``\ ``.F90`` | the ``Simulation.h`` file.       |
+----------------------------------+----------------------------------+
| \*[1ex]                          | contains a mapping between       |
|                                  | particle properties and grid     |
|                                  | variables. Only generated when   |
|                                  | particles are included in a      |
|                                  | simulation.                      |
+----------------------------------+----------------------------------+
| \*[1ex]                          | contains code to make a data     |
|                                  | structure with information about |
| ``Particles/Part                 | the mapping and initialization   |
| icles_specifyMethods``\ ``.F90`` | method for each type of          |
|                                  | particle. Only generated when    |
|                                  | particles are included in a      |
|                                  | simulation.                      |
+----------------------------------+----------------------------------+
| \*[1ex]                          |                                  |
+----------------------------------+----------------------------------+

.. _`Sec:unitMakefiles`:

Makefiles generated by ``setup``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Apart from the master ``Makefile``, ``setup`` generates a makefile for
each unit, which is “included” in the master ``Makefile``. This is true
even if the unit is not included in the application. These unit
makefiles are named ``Makefile.Unit`` and are a concatenation of all the
Makefiles found in unit hierarchy processed by ``setup``.

For example, if an application uses
``Grid/GridMain/paramesh/paramesh4/|paramesh|4.0``, the file
``Makefile.Grid`` will be a concatenation of the Makefiles found in

-  ``Grid``,

-  ``Grid/GridMain``,

-  ``Grid/GridMain/paramesh``,

-  ``Grid/GridMain/paramesh/paramesh4``, and

-  ``Grid/GridMain/paramesh/paramesh4/|paramesh|4.0``

As another example, if an application does not use
``PhysicalConstants``, then ``Makefile.PhysicalConstants`` is just the
contents of ``PhysicalConstants/Makefile`` at the API level.

Since the order of concatenation is arbitrary, the behavior of the
Makefiles should not depend on the order in which they have been
concatenated. The makefiles inside the units contain lines of the form:

.. container:: codeseg

   Unit += file1.o file2.o ...

where ``Unit`` is the name of the unit, which was ``Grid`` in the
example above. Dependency on data modules files *need not be specified*
since the setup process determines this requirement automatically.

.. _`Sec:hybridSetup`:

Setup a hybrid MPI+|openmp| |flashx| application
---------------------------------------------

There is the experimental inclusion of |flashx| multithreading with
|openmp| in the |flashx| beta release. The units which have support for
multithreading are split hydrodynamics `[Sec:PPM] <#Sec:PPM>`__, unsplit
hydrodynamics
`[Sec:unsplit hydro algorithm] <#Sec:unsplit hydro algorithm>`__, Gamma
law and multigamma EOS `[Sec:Eos Gammas] <#Sec:Eos Gammas>`__, Helmholtz
EOS `[Sec:Eos Helmholtz] <#Sec:Eos Helmholtz>`__, Multipole Poisson
solver (improved version (support for 2D cylindrical and 3D cartesian))
`[Sec:GridSolversMultipoleImproved] <#Sec:GridSolversMultipoleImproved>`__
and energy deposition
`[Sec:EnergyDeposition] <#Sec:EnergyDeposition>`__.

The |flashx| multithreading requires a MPI-2 installation built with
thread support (building with an MPI-1 installation or an MPI-2
installation without thread support is possible but strongly
discouraged). The |flashx| application requests the thread support level
``MPI_THREAD_SERIALIZED`` to ensure that the MPI library is thread-safe
and that any |openmp| thread can call MPI functions safely. You should
also make sure that your compiler provides a version of |openmp| which is
compliant with at least the |openmp| 2.5 (200505) standard (older versions
may also work but I have not checked).

In order to make use of the multithreaded code you must setup your
application with one of the setup variables ``threadBlockList``,
``threadWithinBlock`` or ``threadRayTrace`` equal to ``True``, e.g.

.. container:: codeseg

   ./setup Sedov -auto threadBlockList=True ./setup Sedov -auto
   threadBlockList=True +mpi1 (compatible with MPI-1 - unsafe!)

When you do this the setup script will insert ``USEOPENMP = 1`` instead
of ``USEOPENMP = 0`` in the generated Makefile. If it is equal to
:math:`1` the Makefile will prepend an |openmp| variable to the
``FFLAGS``, ``CFLAGS``, ``LFLAGS`` variables.

.. container:: flashtip

   In general you should not define ``FLAGS``, ``CFLAGS`` and ``LFLAGS``
   in your ``Makefile.h``. It is much better to define ``FFLAGS_OPT``,
   ``FFLAGS_TEST``, ``FFLAGS_DEBUG``, ``CFLAGS_OPT``, ``CFLAGS_TEST``,
   ``CFLAGS_DEBUG``, ``LFLAGS_OPT``, ``LFLAGS_TEST`` and
   ``LFLAGS_DEBUG`` in your ``Makefile.h``. The setup script will then
   initialize the ``FFLAGS``, ``CFLAGS`` and ``LFLAGS`` variables in the
   Makefile appropriately for an optimized, test or debug build.

The |openmp| variables should be defined in your ``Makefile.h`` and
contain a compiler flag to recognize |openmp| directives. In most cases it
is sufficient to define a single variable named ``OPENMP``, but you may
encounter special situations when you need to define ``OPENMP_FORTRAN``,
``OPENMP_C`` and ``OPENMP_LINK``. If you want to build |flashx| with the
GNU Fortran compiler ``gfortran`` and the GNU C compiler ``gcc`` then
your ``Makefile.h`` should contain

.. container:: codeseg

   OPENMP = -fopenmp

If you want to do something more complicated like build |flashx| with the
Lahey Fortran compiler ``lf90`` and the GNU C compiler ``gcc`` then your
``Makefile.h`` should contain

.. container:: codeseg

   OPENMP_FORTRAN = –openmp -Kpureomp OPENMP_C = -fopenmp OPENMP_LINK =
   –openmp -Kpureomp

When you run the hybrid |flashx| application it will print the level of
thread support provided by the MPI library and the number of |openmp|
threads in each parallel region

.. container:: codeseg

   : Called MPI_Init_thread - requested level 2, given level 2
   [Driver_initParallel]: Number of |openmp| threads in each parallel
   region 4

Note that the |flashx| application will still run if the MPI library does
not provide the requested level of thread support, but will print a
warning message alerting you to an unsafe level of MPI thread support.
There is no guarantee that the program will work! I strongly recommend
that you stop using this |flashx| application - you should build a MPI-2
library with thread support and then rebuild |flashx|.

We record extra version and runtime information in the |flashx| log file
for a threaded application. Table `1.4 <#tab:flash_openmp_logs>`__ shows
log file entries from a threaded |flashx| application along with example
safe and unsafe values. All cells colored red show unsafe values.

.. container:: center

   .. container::
      :name: tab:flash_openmp_logs

      .. table:: Log file entries showing safe and unsafe threaded
      |flashx| applications

         +-------------+----------+-------------+-------------+-------------+
         | **Log file  | **safe** | **unsafe    | **unsafe    | **unsafe    |
         | stamp**     |          | (1)**       | (2)**       | (3)**       |
         +=============+==========+=============+=============+=============+
         | Number of   | 1        | 1           | 1           | 1           |
         | MPI tasks:  |          |             |             |             |
         +-------------+----------+-------------+-------------+-------------+
         | MPI         | 2        | 1           | 2           | 2           |
         | version:    |          |             |             |             |
         +-------------+----------+-------------+-------------+-------------+
         | MPI         | 2        | 2           | 1           | 2           |
         | subversion: |          |             |             |             |
         +-------------+----------+-------------+-------------+-------------+
         | MPI thread  | T        | F           | F           | F           |
         | support:    |          |             |             |             |
         +-------------+----------+-------------+-------------+-------------+
         | |openmp|      | 2        | 2           | 2           | 2           |
         | threads/MPI |          |             |             |             |
         | task:       |          |             |             |             |
         +-------------+----------+-------------+-------------+-------------+
         | |openmp|      | 200805   | 200505      | 200505      | 200805      |
         | version:    |          |             |             |             |
         +-------------+----------+-------------+-------------+-------------+
         | Is          | T        | T           | T           | F           |
         | “\_OPENMP”  |          |             |             |             |
         | macro       |          |             |             |             |
         | defined:    |          |             |             |             |
         +-------------+----------+-------------+-------------+-------------+

The |flashx| applications in Table `1.4 <#tab:flash_openmp_logs>`__ are
unsafe because

#. we are using an MPI-1 implementation.

#. we are using an MPI-2 implementation which is not built with thread
   support - the “MPI thread support in |openmp|I” Flash tip may help.

#. we are using a compiler that does not define the macro ``_OPENMP``
   when it compiles source files with |openmp| support (see |openmp|
   standard). I have noticed that Absoft 64-bit Pro Fortran 11.1.3 for
   Linux x86_64 does not define this macro. We use this macro in
   ``Driver_initParallel.F90`` to conditionally initialize MPI with
   ``MPI_Init_thread``. If you find that ``_OPENMP`` is not defined you
   should define it in your ``Makefile.h`` in a manner similar to the
   following:

   .. container:: codeseg

      OPENMP_FORTRAN = -openmp -D_OPENMP=200805

You should not setup a |flashx| application with both ``threadBlockList``
and ``threadWithinBlock`` equal to ``True`` - nested |openmp| parallelism
is not supported. For further information about |flashx| multithreaded
applications please refer to Chapter
`[Chp:Multithreaded|flashx|] <#Chp:Multithreaded|flashx|>`__.

.. [1]
   if a machine has multiple hostnames, setup tries them all

.. [2]
   Formerly, (in Flash2 it was located in the |flash| root directory

.. [3]
   Formerly, (in Flash2) it was located in the |flash| root directory

.. [4]
   Formerly, (in Flash2) it was located in the |flash| root directory

.. [5]
   All non-integral values not equal to True/False/Yes/No/On/Off are
   considered to be string values
