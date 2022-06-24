.. _`Chp:The Configuration Toolchain`:

The Configuration Toolchain
===================================

The tool is the most important component of the . It implements the
inheritance and composability of the software system. It traverses the
Flash-X source tree starting from the directory hosting the specific
application definition. This starting directory is essentially the
selected implementation of the “Simulation” unit. While traversing the
source three the setup tool does the following:

-  arbitrate on map from a key to its definition, in particular if
   alternative definitions exist

-  arbitrate on which implementation of a function to use

-  link selected files (include source code and the key definitions) to
   the directory

-  invoke macroprocessor to translate keys and generate source file

-  alter index order if desired

-  find the target

-  generate the that will build the Flash-X executable.

-  generate the files needed to add runtime parameters to a given
   simulation.

-  generate the files needed to parse the runtime parameter file.

More description of how and the architecture interact may be found in .
Here we describe its usage.

The script determines site-dependent configuration information by
looking for a directory where is the hostname of the machine on which
Flash-X is running. [1]_ Failing this, it looks in for a directory with
the same name as the output of the command. The site and operating
system type can be overridden with the and command-line options to the
command. Only one of these options can be used at one time. The
directory for each site and operating system type contains a makefile
fragment that sets command names, compiler flags, library paths, and any
replacement or additional source files needed to compile Flash-X for
that specific machine and machine type.

The script starts with the file of the specified application in the
Simulation unit, finds its units and then works its way through their
files. This process continues until all the dependencies are met and a
self-consistent set of units has been found. At the end of this
automatic generation, the file is created and placed in the directory,
where it can be edited if necessary. also creates the master makefile ()
and several  include files that are needed by the code in order to parse
the runtime parameters. After running , the user can create the Flash-X
executable by running in the directory. Note that the Flash-X build
system assumes that the command invokes GNU Make and is unlikely to work
properly with other implementations of the command. On some systems it
may be necessary to invoke GNU Make under the name .

.. container:: flashtip

   -  All the setup options can be shortened to unambiguous prefixes,
       instead of ``./setup -auto <problem-name>`` one can just say
      ``./setup -a <problem-name>`` since there is only one option
      starting with .

   -  The same abbreviation holds for the problem name as well. can be
      abbreviated to assuming that is the only problem name which starts
      with .

   -  Unit names are usually specified by their paths relative to the
      source directory. However, also allows unit names to be prefixed
      with an extra “source/”, allowing you to use the TAB-completion
      features of your shell like this

      .. container:: codeseg

         ./setup -a Isen -unit=sou<TAB>rce/IO/IOM<TAB>ain/hd<TAB>f5

   -  If you use a set of options repeatedly, you can define a shortcut
      for them. comes with a number of predefined shortcuts that
      significantly simplify the setup line, particularly when trying to
      match the Grid with a specific I/O implementation. For more
      details on creating shortcuts see . For detailed examples of I/O
      shortcuts please see in the I/O chapter.

Setup Arguments
---------------

The setup script accepts a large number of command line arguments which
affect the simulation in various ways. These arguments are divided into
three categories:

#. *Setup Options* (example: ) begin with a dash and are built into the
   setup script itself. Many of the most commonly used arguments are
   setup options.

#. *Setup Variables* (example: ) are defined by individual units. When
   writing a file for any unit, you can define a setup variable.
   explains how setup variables can be created and used.

#. *Setup Shortcuts* (example: ) begin with a plus symbol and are
   essentially macros which automatically include a set of setup
   variables and/or setup options. New setup shortcuts can be easily
   defined, see for more information.

shows a list of some of the basic setup arguments that every Flash-X
user should know about. A comprehensive list of all setup arguments can
be found in alongside more detailed descriptions of these options.

.. container:: center

   .. container::
      :name: Tbl:CommonSetupArgs

      .. table::  List of Commonly Used Setup Arguments

         ============ ===================================================
         **Argument** **Description**
         ============ ===================================================
         \            this option should almost always be set
         \            include a specified unit
         \            specify a different object directory location
         \            compile for debugging
         \            enable compiler optimization
         \            specify block size in each direction
         \            specify maximum number of blocks per process
         \            specify number of dimensions
         \            specify maximum number of blocks per process
         \            use Cartesian geometry
         \            use cylindrical geometry
         \            use polar geometry
         \            use spherical geometry
         \            disable IO
         \            use the uniform grid in a fixed block size mode
         \            use the uniform grid in a non-fixed block size mode
         \            use the PARAMESH2 grid
         \            use the PARAMESH4.0 grid
         \            use the PARAMESH4DEV grid
         \            use the Unsplit Hydro solver
         \            use the Unsplit Staggered Mesh MHD solver
         \            use a split Hydro solver
         ============ ===================================================

.. _`Sec:ListSetupArgs`:

Comprehensive List of Setup Arguments
-------------------------------------

+----------------------------------+----------------------------------+
|                                  |                                  |
+----------------------------------+----------------------------------+
| \*[1ex]                          | Normally prints summary messages |
|                                  | indicating its progress. Use the |
|                                  | to make the messages more or     |
|                                  | less verbose. The different      |
|                                  | levels (in order of increasing   |
|                                  | verbosity) are . The default is  |
|                                  | .                                |
+----------------------------------+----------------------------------+
|                                  |                                  |
+----------------------------------+----------------------------------+
| \*[1ex]                          | Normally, requires that the user |
|                                  | supply a plain text file called  |
|                                  | (in the directory  [4]_) that    |
|                                  | specifies the units to include.  |
|                                  | A sample file appears in . Each  |
|                                  | line is either a comment         |
|                                  | (preceded by a hash mark ()) or  |
|                                  | the name of a an include         |
|                                  | statement of the form *unit*.    |
|                                  | Specific implementations of a    |
|                                  | unit may be selected by          |
|                                  | specifying the complete path to  |
|                                  | the implementation in question;  |
|                                  | If no specific implementation is |
|                                  | requested, picks the default     |
|                                  | listed in the unit’s file.       |
+----------------------------------+----------------------------------+
|                                  | The option enables to generate a |
|                                  | “rough draft” of a file for the  |
|                                  | user. The file for each problem  |
|                                  | setup specifies its requirements |
|                                  | in terms of other units it       |
|                                  | requires. For example, a problem |
|                                  | may require the perfect-gas      |
|                                  | equation of state () and an      |
|                                  | unspecified hydro solver ().     |
|                                  | With , creates a file by         |
|                                  | converting these requirements    |
|                                  | into unit include statements.    |
|                                  | Most users configuring a problem |
|                                  | for the first time will want to  |
|                                  | run with to generate a file and  |
|                                  | then to edit it directly to      |
|                                  | specify alternate                |
|                                  | implementations of certain       |
|                                  | units. After editing the file,   |
|                                  | the user must re-run without in  |
|                                  | order to incorporate his/her     |
|                                  | changes into the code            |
|                                  | configuration. The user may also |
|                                  | use the command-line option in   |
|                                  | conjunction with the option, in  |
|                                  | order to pick a specific         |
|                                  | implementation of a unit, and    |
|                                  | thus eliminate the need to       |
|                                  | hand-edit the file.              |
+----------------------------------+----------------------------------+
|                                  |                                  |
+----------------------------------+----------------------------------+
| \*[1ex]                          | By default, creates a makefile   |
|                                  | which produces a Flash-X         |
|                                  | executable capable of solving    |
|                                  | two-dimensional problems         |
|                                  | (equivalent to ). To generate a  |
|                                  | makefile with options            |
|                                  | appropriate to three-dimensional |
|                                  | problems, use . To generate a    |
|                                  | one-dimensional code, use .      |
|                                  | These options are mutually       |
|                                  | exclusive and cause to add the   |
|                                  | appropriate compilation option   |
|                                  | to the makefile it generates.    |
+----------------------------------+----------------------------------+
|                                  |                                  |
+----------------------------------+----------------------------------+
| \*[1ex]                          | in constructing the makefile     |
|                                  | compiler options. It determines  |
|                                  | the amount of memory allocated   |
|                                  | at runtime to the adaptive mesh  |
|                                  | refinement (AMR) block data      |
|                                  | structure. For example, to       |
|                                  | allocate enough memory on each   |
|                                  | processor for 500 blocks, use .  |
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
|                                  | than during refinement. Resetup  |
|                                  | the simulation using a larger    |
|                                  | value of this option.            |
+----------------------------------+----------------------------------+
|                                  |                                  |
+----------------------------------+----------------------------------+
| \*[1ex] These options are used   |                                  |
| by in constructing the makefile  |                                  |
| compiler options. The mesh on    |                                  |
| which the problem is solved is   |                                  |
| composed of blocks, and each     |                                  |
| block contains some number of    |                                  |
| cells. The , , and options       |                                  |
| determine how many cells each    |                                  |
| block contains (not counting     |                                  |
| guard cells). The default value  |                                  |
| for each is 8. These options do  |                                  |
| not have any effect when running |                                  |
| in Uniform Grid non-fixed block  |                                  |
| size mode.                       |                                  |
+----------------------------------+----------------------------------+
|                                  |                                  |
+----------------------------------+----------------------------------+
| \*[1ex]                          | The default built by setup will  |
|                                  | use the optimized setting () for |
|                                  | compilation and linking. Using   |
|                                  | will force to use the flags      |
|                                  | relevant for debugging (,        |
|                                  | including in the compilation     |
|                                  | line). The user may use the      |
|                                  | option to experiment with        |
|                                  | different combinations of        |
|                                  | compiler and linker options.     |
|                                  | Exactly which compiler and       |
|                                  | linker options are associated    |
|                                  | with each of these flags is      |
|                                  | specified in where is the        |
|                                  | hostname of the machine on which |
|                                  | Flash-X is running.              |
|                                  |                                  |
|                                  | For example, to tell an Intel    |
|                                  | Fortran compiler to use real     |
|                                  | numbers of size 64 when the      |
|                                  | option is specified, the user    |
|                                  | might add the following line to  |
|                                  | his/her :                        |
|                                  |                                  |
|                                  | .. container:: codeseg           |
|                                  |                                  |
|                                  |    FFLAGS_TEST = -real_size 64   |
+----------------------------------+----------------------------------+
|                                  |                                  |
+----------------------------------+----------------------------------+
| \*[1ex] Overrides the default    |                                  |
| directory with . Using this      |                                  |
| option allows you to have        |                                  |
| different simulations configured |                                  |
| simultaneously in the            |                                  |
| distribution directory.          |                                  |
+----------------------------------+----------------------------------+
| ,                                |                                  |
+----------------------------------+----------------------------------+
| \*[1ex] in setting up the        |                                  |
| problem.                         |                                  |
+----------------------------------+----------------------------------+

.. container:: fcodeseg

   #Units file for Sod generated by setup

   INCLUDE Driver/DriverMain/Split INCLUDE Grid/GridBoundaryConditions
   INCLUDE Grid/GridMain/paramesh/interpolation/Paramesh4/prolong
   INCLUDE Grid/GridMain/paramesh/interpolation/prolong INCLUDE
   Grid/GridMain/paramesh/paramesh4/Paramesh4.0/PM4_package/headers
   INCLUDE
   Grid/GridMain/paramesh/paramesh4/Paramesh4.0/PM4_package/mpi_source
   INCLUDE
   Grid/GridMain/paramesh/paramesh4/Paramesh4.0/PM4_package/source
   INCLUDE
   Grid/GridMain/paramesh/paramesh4/Paramesh4.0/PM4_package/utilities/multigrid
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

+---------+-----------------------------------------------------------+
|         |                                                           |
+---------+-----------------------------------------------------------+
| \*[1ex] | Enable code in that implements geometrically correct data |
|         | restriction for curvilinear coordinates. This setting is  |
|         | automatically enabled if a non- geometry is chosen with   |
|         | the flag; so specifying only has an effect in the         |
|         | Cartesian case.                                           |
+---------+-----------------------------------------------------------+
|         |                                                           |
+---------+-----------------------------------------------------------+
| \*[1ex] | is of the form or . This causes the specified             |
|         | pre-processor symbols to be defined when the code is      |
|         | being compiled. This is mainly useful for debugging the   |
|         | code. For , turns on all debugging messages. Each unit    |
|         | may have its own flag which you can selectively turn on.  |
+---------+-----------------------------------------------------------+
|         |                                                           |
+---------+-----------------------------------------------------------+
| \*[1ex] | Causes the code to be compiled in fixed-block or          |
|         | non-fixed-block size mode. Fixed-block mode is the        |
|         | default. In non-fixed block size mode, all storage space  |
|         | is allocated at runtime. This mode is available only with |
|         | Uniform Grid.                                             |
+---------+-----------------------------------------------------------+
|         |                                                           |
+---------+-----------------------------------------------------------+
| \*[1ex] | Choose one of the supported geometries or . Some Grid     |
|         | implementations require the geometry to be known at       |
|         | compile-time while others don’t. This setup option can be |
|         | used in either case; it is a good idea to specify the     |
|         | geometry here if it is known at -time. Choosing a         |
|         | non-Cartesian geometry here automatically sets the option |
|         | below.                                                    |
+---------+-----------------------------------------------------------+
|         |                                                           |
+---------+-----------------------------------------------------------+
| \*[1ex] | Select a scheme for Grid interpolation. Two schemes are   |
|         | currently supported:                                      |
|         |                                                           |
|         | -  This scheme attempts to ensure that monotonicity is    |
|         |    preserved in interpolation, so that interpolation does |
|         |    not introduce small-scale non-monotonicity in the      |
|         |    data. The scheme is required for curvilinear           |
|         |    coordinates and is automatically enabled if a non-     |
|         |    geometry is chosen with the flag. For AMR Grid         |
|         |    implementations, This flag will automatically add      |
|         |    additional directories so that appropriate data        |
|         |    interpolation methods are compiled it. The scheme is   |
|         |    the default (by way of the shortcut), unlike in .      |
|         |                                                           |
|         | -  Enable the interpolation that is native to the AMR     |
|         |    Grid implementation ( or ) by default. This option is  |
|         |    only appropriate for Cartesian geometries.             |
+---------+-----------------------------------------------------------+

.. container::
   :name: setupclf:particlemethods

   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | normally uses the from the       |
   |                                  | directory determined by the      |
   |                                  | hostname of the machine and the  |
   |                                  | and options. If you have         |
   |                                  | multiple compilers on your       |
   |                                  | machine you can create for       |
   |                                  | different compilers. , you can   |
   |                                  | have a and and for the three     |
   |                                  | different compilers. will still  |
   |                                  | use the file by default, but     |
   |                                  | supplying on the command-line    |
   |                                  | causes to use instead.           |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex] Instructs that indexing  |                                  |
   | of unk and related arrays should |                                  |
   | be changed. This may be needed   |                                  |
   | in for compatibility with        |                                  |
   | alternative grids. This is       |                                  |
   | supported by both the Uniform    |                                  |
   | Grid as well as PARAMESH.        |                                  |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | Ordinarily, the commands being   |
   |                                  | executed during compilation of   |
   |                                  | the Flash-X executable are sent  |
   |                                  | to standard out. It may be that  |
   |                                  | you find this distracting, or    |
   |                                  | that your terminal is not able   |
   |                                  | to handle these long lines of    |
   |                                  | display. Using the option causes |
   |                                  | to generate a so that GNU only   |
   |                                  | displays the names of the files  |
   |                                  | being compiled and not the exact |
   |                                  | compiler call and flags. This    |
   |                                  | information remains available in |
   |                                  | in the directory.                |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | normally removes all code in the |
   |                                  | directory before linking in      |
   |                                  | files for a simulation. The      |
   |                                  | ensuing must therefore compile   |
   |                                  | all source files anew each time  |
   |                                  | is run. The option prevents from |
   |                                  | removing compiled code which has |
   |                                  | not changed from the previous in |
   |                                  | the same directory. This can     |
   |                                  | speed up the process             |
   |                                  | significantly.                   |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | If is unable to find a correct   |
   |                                  | directory it picks the based on  |
   |                                  | the operating system. This       |
   |                                  | option instructs to use the      |
   |                                  | default corresponding to the     |
   |                                  | specified operating system.      |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This causes to copy the          |
   |                                  | specified runtime-parameters     |
   |                                  | file in the simulation directory |
   |                                  | to the directory with the new    |
   |                                  | name .                           |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This option takes a              |
   |                                  | comma-separated list of names of |
   |                                  | parameter files and combines     |
   |                                  | them into one file in the        |
   |                                  | directory. File names without an |
   |                                  | absolute path are taken to be    |
   |                                  | relative to the simulation       |
   |                                  | directory, as for the option.    |
   |                                  |                                  |
   |                                  | To use such a combined in case   |
   |                                  | of runtime parameters occurring  |
   |                                  | more than once, note that when   |
   |                                  | Flash-X reads a parameter file,  |
   |                                  | the last instance of a runtime   |
   |                                  | parameter supersedes previous    |
   |                                  | ones.                            |
   |                                  |                                  |
   |                                  | If both and are used, the files  |
   |                                  | from the list are appended to    |
   |                                  | the single parfile given by the  |
   |                                  | latter in the order listed. If   |
   |                                  | used with , can append one or    |
   |                                  | more parfiles to the one given   |
   |                                  | by . If you only use and not and |
   |                                  | give it fewer than two paths, an |
   |                                  | error will result. If more than  |
   |                                  | one option appears, the lists    |
   |                                  | are concatenated in the order    |
   |                                  | given.                           |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This option instructs to adjust  |
   |                                  | the particle methods for a       |
   |                                  | particular particle type. It can |
   |                                  | only be used when a particle     |
   |                                  | type has already been registered |
   |                                  | with a line in a file (see ). A  |
   |                                  | possible scenario for using this |
   |                                  | option involves the user wanting |
   |                                  | to use a different passive       |
   |                                  | particle initialization method   |
   |                                  | without modifying the line in    |
   |                                  | the simulation file. In this     |
   |                                  | case, an additional adjusts the  |
   |                                  | initialization method associated |
   |                                  | with passive particles in the    |
   |                                  | generated subroutine. Since the  |
   |                                  | specification of a method for    |
   |                                  | mapping and initialization       |
   |                                  | requires inclusions of           |
   |                                  | appropriate implementations of   |
   |                                  | and subunits, and the            |
   |                                  | specification of a method for    |
   |                                  | time advancement requires        |
   |                                  | inclusion of an appropriate      |
   |                                  | implementation under , it is the |
   |                                  | user’s responsibility to adjust  |
   |                                  | the included units               |
   |                                  | appropriately. For example a     |
   |                                  | user may want want to override   |
   |                                  | file defined particle type using |
   |                                  | lattice initialization density   |
   |                                  | based distribution method using  |
   |                                  | the command line. Here the user  |
   |                                  | must first specify to exclude    |
   |                                  | the lattice initialization,      |
   |                                  | followed by specification to     |
   |                                  | include the appropriate          |
   |                                  | implementation. In general,      |
   |                                  | using command line overrides of  |
   |                                  | are not recommended, as this     |
   |                                  | option increases the chance of   |
   |                                  | creating an inconsistent         |
   |                                  | simulation setup. More           |
   |                                  | information on multiple particle |
   |                                  | types can be found in ,          |
   |                                  | especially .                     |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This option causes setup to      |
   |                                  | create a portable directory by   |
   |                                  | copying instead of linking to    |
   |                                  | the source files. The resulting  |
   |                                  | directory can be tarred and sent |
   |                                  | to another machine for actual    |
   |                                  | compilation.                     |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | searches the directory for a     |
   |                                  | directory whose name is the      |
   |                                  | hostname of the machine on which |
   |                                  | setup is being run. This option  |
   |                                  | tells to use the of the          |
   |                                  | specified site. This option is   |
   |                                  | useful if is unable to find the  |
   |                                  | right hostname (which can happen |
   |                                  | on multiprocessor or laptop      |
   |                                  | machines). Also useful when      |
   |                                  | combined with the option.        |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This causes to copy the          |
   |                                  | specified file to the directory  |
   |                                  | as before setting up the         |
   |                                  | problem. This option can be used |
   |                                  | when is not used, to specify an  |
   |                                  | alternate file.                  |
   +----------------------------------+----------------------------------+
   | ,                                |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This option instructs to link in |
   |                                  | the specified library when       |
   |                                  | building the final executable. A |
   |                                  | *library* is a piece of code     |
   |                                  | which is independent of Flash-X. |
   |                                  | Internal libraries are those     |
   |                                  | libraries whose code is included |
   |                                  | with Flash-X. The script         |
   |                                  | supports external as well as     |
   |                                  | internal libraries. Information  |
   |                                  | about external libraries is      |
   |                                  | usually found in the site        |
   |                                  | specific Makefile. The           |
   |                                  | additional if any are            |
   |                                  | library-specific and may be used |
   |                                  | to select among multiple         |
   |                                  | implementations.                 |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This option causes the inclusion |
   |                                  | of an additional Makefile        |
   |                                  | necessary for the operation of   |
   |                                  | Tau, which may be used by the    |
   |                                  | user to profile the code. More   |
   |                                  | information on Tau can be found  |
   |                                  | at http://acts.nersc.gov/tau/    |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | Negates a previously specified   |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This removes all units specified |
   |                                  | in the command line so far,      |
   |                                  | which are children of the        |
   |                                  | specified unit (including the    |
   |                                  | unit itself). It also negates    |
   |                                  | any REQUESTS keyword found in a  |
   |                                  | file for units which are         |
   |                                  | children of the specified unit.  |
   |                                  | However it does not negate a     |
   |                                  | REQUIRES keyword found in a      |
   |                                  | file.                            |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies using    |
   |                                  | basic default settings and is    |
   |                                  | equivalent to the following:     |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation without IO and is     |
   |                                  | equivalent to the following:     |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation with basic IO and is  |
   |                                  | equivalent to the following:     |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using serial IO, it   |
   |                                  | has the effect of setting the    |
   |                                  | setup variable                   |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using serial IO, it   |
   |                                  | has the effect of setting the    |
   |                                  | setup variable                   |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using hdf5 for        |
   |                                  | compatible binary IO output, it  |
   |                                  | has the effect of setting the    |
   |                                  | setup variable                   |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using hdf5, with      |
   |                                  | parallel io capability for       |
   |                                  | compatible binary IO output, and |
   |                                  | is equivalent to the following:  |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation without log           |
   |                                  | capability it is equivalent to   |
   |                                  | the following:                   |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation with the Grid unit,   |
   |                                  | it is equivalent to the          |
   |                                  | following:                       |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using a uniform grid, |
   |                                  | it is equivalent to the          |
   |                                  | following:                       |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using Paramesh2 for   |
   |                                  | the grid, it is equivalent to    |
   |                                  | the following:                   |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using Paramesh4.0 for |
   |                                  | the grid, it is equivalent to    |
   |                                  | the following:                   |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using a version of    |
   |                                  | Paramesh 4 that is closer to the |
   |                                  | version available on             |
   |                                  | sourceforge. It is equivalent    |
   |                                  | to:                              |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using a modified      |
   |                                  | version of Paramesh 4 that       |
   |                                  | includes a more scalable way of  |
   |                                  | filling the array. It is         |
   |                                  | equivalent to:                   |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a MHD    |
   |                                  | simulation using the unsplit     |
   |                                  | staggered mesh hydro solver, if  |
   |                                  | pure hydro mode is used with the |
   |                                  | USM solver add +pureHydro in the |
   |                                  | setup line. It is equivalent to: |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies using    |
   |                                  | pure hydro mode, it is           |
   |                                  | equivalent to:                   |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using a split hydro   |
   |                                  | solver and is equivalent to:     |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using the unsplit     |
   |                                  | hydro solver and is equivalent   |
   |                                  | to:                              |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using the unsplit     |
   |                                  | hydro solver and is equivalent   |
   |                                  | to:                              |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using a specific      |
   |                                  | Hydro method that requires an    |
   |                                  | increased number of guard cells, |
   |                                  | this may need to be combined     |
   |                                  | with where the specified         |
   |                                  | blocksize is greater than or     |
   |                                  | equal to 12 (==2*GUARDCELLS). It |
   |                                  | is equivalent to:                |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation with a block size of  |
   |                                  | 64**3, it is equivalent to:      |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation with a block size of  |
   |                                  | 32**3, it is equivalent to:      |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation with a block size of  |
   |                                  | 16**3, it is equivalent to:      |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using particles and   |
   |                                  | IO for uniform grid, it is       |
   |                                  | equivalent to:                   |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut is used for        |
   |                                  | checking Flash-X with            |
   |                                  | rectangular block sizes and      |
   |                                  | non-fixed block size. It is      |
   |                                  | equivalent to:                   |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using a uniform grid  |
   |                                  | with a non-fixed block size. It  |
   |                                  | is equivalent to:                |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using curvilinear     |
   |                                  | geometry. It is equivalent to:   |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using cartesian       |
   |                                  | geometry. It is equivalent to:   |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using spherical       |
   |                                  | geometry. It is equivalent to:   |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using polar geometry. |
   |                                  | It is equivalent to:             |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using cylindrical     |
   |                                  | geometry. It is equivalent to:   |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using passive         |
   |                                  | particles initialized by         |
   |                                  | density. It is equivalent to:    |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using                 |
   |                                  | NO_PERMANENT_GUARDCELLS mode in  |
   |                                  | Paramesh4. It is equivalent to:  |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | smilulation using multipole      |
   |                                  | gravity, it is equivalent to:    |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using long range      |
   |                                  | active particles. It is          |
   |                                  | equivalent to:                   |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using FFT based       |
   |                                  | gravity solve on a uniform grid  |
   |                                  | with no fixed block size. It is  |
   |                                  | equivalent to:                   |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using a multigrid     |
   |                                  | based gravity solve. It is       |
   |                                  | equivalent to:                   |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | smilulation using multipole      |
   |                                  | gravity, it is equivalent to:    |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation \*not\* using the     |
   |                                  | multipole based gravity solve.   |
   |                                  | It is equivalent to:             |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation \*not\* using the     |
   |                                  | multigrid based gravity solve.   |
   |                                  | It is equivalent to:             |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies a        |
   |                                  | simulation using the new         |
   |                                  | multipole based gravity solve.   |
   |                                  | It is equivalent to:             |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This shortcut specifies use of   |
   |                                  | proper particle units to perform |
   |                                  | PIC (particle in cell) method.   |
   |                                  | It is equivalent to:             |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This setup variable can be used  |
   |                                  | to specify which gridding        |
   |                                  | package to use in a simulation:  |
   |                                  | Name: Type: Values: , ,          |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This setup variable can be used  |
   |                                  | to specify which IO package to   |
   |                                  | use in a simulation: Name: Type: |
   |                                  | Values:                          |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This setup variable can be used  |
   |                                  | to specify which type of IO      |
   |                                  | strategy will be used. A         |
   |                                  | “parallel” strategy will be used |
   |                                  | if the value is true, a “serial” |
   |                                  | strategy otherwise. Name: Type:  |
   |                                  | Values:                          |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This setup variable indicates    |
   |                                  | whether or not a fixed block     |
   |                                  | size is to be used. This         |
   |                                  | variable should not be assigned  |
   |                                  | explicitly on the command line.  |
   |                                  | It defaults to , and the setup   |
   |                                  | options and modify the value of  |
   |                                  | this variable. Name: Type:       |
   |                                  | Values:                          |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This setup variable gives the    |
   |                                  | dimensionality of a simulation.  |
   |                                  | This variable should not be set  |
   |                                  | explicitly on the command line,  |
   |                                  | it is automatically set by the   |
   |                                  | setup options , , and . Name:    |
   |                                  | Type: Values:                    |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This setup variable indicates    |
   |                                  | whether the setup option is in   |
   |                                  | effect. This variable should not |
   |                                  | be assigned explicitly on the    |
   |                                  | command line. Name: Type:        |
   |                                  | Values:                          |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This setup variable gives the    |
   |                                  | number of zones in a block in    |
   |                                  | the X direction. This variable   |
   |                                  | should not be assigned           |
   |                                  | explicitly on the command line,  |
   |                                  | it is automatically set by the   |
   |                                  | setup option . Name: Type:       |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This setup variable gives the    |
   |                                  | number of zones in a block in    |
   |                                  | the Y direction. This variable   |
   |                                  | should not be assigned           |
   |                                  | explicitly on the command line,  |
   |                                  | it is automatically set by the   |
   |                                  | setup option . Name: Type:       |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This setup variable gives the    |
   |                                  | number of zones in a block in    |
   |                                  | the Z direction. This variable   |
   |                                  | should not be assigned           |
   |                                  | explicitly on the command line,  |
   |                                  | it is automatically set by the   |
   |                                  | setup option . Name: Type:       |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This setup variable gives the    |
   |                                  | maximum number of blocks per     |
   |                                  | processor. This variable should  |
   |                                  | not be assigned explicitly on    |
   |                                  | the command line, it is          |
   |                                  | automatically set by the setup   |
   |                                  | option . Name: Type:             |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | If true, the setup script will   |
   |                                  | generate file from template      |
   |                                  | found in either the object       |
   |                                  | directory (preferred) or the     |
   |                                  | setup script (bin) directory.    |
   |                                  | Selects whether Paramesh4 should |
   |                                  | be compiled in LIBRARY mode,     |
   |                                  | i.e., with the preprocessor      |
   |                                  | symbol LIBRARY defined. Name:    |
   |                                  | Type: Values:                    |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | PfftSolver selects a PFFT solver |
   |                                  | variant when the hybrid (,       |
   |                                  | Multigrid with PFFT) Poisson     |
   |                                  | solver is used. Name: Type:      |
   |                                  | Values: (default), , others      |
   |                                  | (unsupported) if recognized in   |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | If True, a Driver implementation |
   |                                  | is requested. Name: Type:        |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | Automatically set by shortcut.   |
   |                                  | When true, this option activates |
   |                                  | the MTMMMT EOS. Name: Type:      |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | mgd_meshgroups \* meshCopyCount  |
   |                                  | sets the MAXIMUM number of       |
   |                                  | radiation groups that can be     |
   |                                  | used in a simulation. The ACTUAL |
   |                                  | number of groups (which must be  |
   |                                  | less than mgd_meshgroups \*      |
   |                                  | meshCopyCount) is set by the     |
   |                                  | rt_mgdNumGroups runtime          |
   |                                  | parameter. Name: Type:           |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
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
   |                                  | species names. Name: Type: ,     |
   |                                  | comma seperated list of strings  |
   |                                  | (,                               |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | Name: Type: Remark: Maximum      |
   |                                  | number of laser pulses (defaults |
   |                                  | to 5)                            |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | Name: Type: Remark: Maximum      |
   |                                  | number of laser beams (defaults  |
   |                                  | to 6)                            |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This is used to turn on block    |
   |                                  | list OPENMP threading of hydro   |
   |                                  | routines. Name: Type: Values:    |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This is used to turn on block    |
   |                                  | list OPENMP threading of the     |
   |                                  | multipole routine. Name: Type:   |
   |                                  | Values:                          |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This is used to turn on block    |
   |                                  | list OPENMP threading of Enery   |
   |                                  | Deposition source term routines. |
   |                                  | Name: Type: Values:              |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This is used to turn on within   |
   |                                  | block OPENMP threading of hydro  |
   |                                  | routines. Name: Type: Values:    |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This is used to turn on within   |
   |                                  | block OPENMP threading of Eos    |
   |                                  | routines. Name: Type: Values:    |
   +----------------------------------+----------------------------------+
   |                                  |                                  |
   +----------------------------------+----------------------------------+
   | \*[1ex]                          | This is used to turn on within   |
   |                                  | block OPENMP threading of then   |
   |                                  | multipole routine. Name: Type:   |
   |                                  | Values:                          |
   +----------------------------------+----------------------------------+

.. _`Sec:SetupShortcuts`:

Using Shortcuts
---------------

Apart from the various setup options the script also allows you to use
shortcuts for frequently used combinations of options. For example,
instead of typing in

.. container:: codeseg

   ./setup -a Sod -with-unit=Grid/GridMain/UG

you can just type

.. container:: codeseg

   ./setup -a Sod +ug

The or any setup option starting with a ‘+’ is considered as a shortcut.
By default, setup looks at for a list of declared shortcuts. You can
also specify a ":" delimited list of files in the environment variable
and will read all the files specified (and ignore those which don’t
exist) for shortcut declarations. See for an example file.

.. container:: fcodeseg

   # comment line

   # each line is of the form # shortcut:arg1:arg2:...: # These
   shortcuts can refer to each other.

   default:–with-library=mpi:-unit=IO/IOMain:-gridinterpolation=monotonic

   # io choices noio:–without-unit=IO/IOMain: io:–with-unit=IO/IOMain:

   # Choice of Grid ug:-unit=Grid/GridMain/UG:
   pm2:-unit=Grid/GridMain/paramesh/Paramesh2:
   pm40:-unit=Grid/GridMain/paramesh/paramesh4/Paramesh4.0:
   pm4dev:-unit=Grid/GridMain/paramesh/paramesh4/Paramesh4dev:

   # frequently used geometries cube64:-nxb=64:-nyb=64:-nzb=64:

The shortcuts are replaced by their expansions in place, so options
which come after the shortcut override (or conflict with) options
implied by the shortcut. A shortcut can also refer to other shortcuts as
long as there are no cyclic references.

The “default" shortcut is special. always prepends to its command line
thus making equivalent to . Thus changing the default IO to
“hdf5/parallel", is as simple as changing the definition of the
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

Setup Variables and Preprocessing Files
---------------------------------------

allows you to assign values to “Setup Variables”. These variables can be
string-valued, integer-valued, or boolean. A call like

.. container:: codeseg

   ./setup -a Sod Foo=Bar Baz=True

sets the variable “Foo" to string “Bar" and “Baz" to boolean True [5]_.
can conditionally include and exclude parts of the file it reads based
on the values of these variables. For example, the file contains

.. container:: shrink

   .. container:: fcodeseg

      DEFAULT serial

      USESETUPVARS parallelIO

      IF parallelIO DEFAULT parallel ENDIF

The code sets IO to its default value of “serial” and then resets it to
“parallel" if the setup variable “parallelIO" is True. The keyword in
the file instructs setup that the specified variables must be defined;
undefined variables will be set to the empty string.

Through judicious use of setup variables, the user can ensure that
specific implementations are included or the simulation is properly
configured. For example, the setup line expands to . The relevant part
of the file is given below:

.. container:: shrink

   .. container:: fcodeseg

      # Requires use of the Grid SetupVariable USESETUPVARS Grid

      DEFAULT paramesh

      IF Grid==’UG’ DEFAULT UG ENDIF IF Grid==’PM2’ DEFAULT
      paramesh/Paramesh2 ENDIF

The file defaults to choosing . But when the setup variable Grid is set
to “UG" through the shortcut , the default implementation is set to
“UG". The same technique is used to ensure that the right IO unit is
automatically included.

See for an exhaustive list of Setup Variables which are used in the
various Config files. For example the setup variable can be test to
ensure that a simulation is configured with the appropriate
dimensionality (see for example ).

.. _`Sec:Config`:

 Files
-----

Information about unit dependencies, default sub-units, runtime
parameter definitions, library requirements, and physical variables,
etc. is contained in plain text files named in the different unit
directories. These are parsed by when configuring the source tree and
are used to create the code needed to register unit variables, to
implement the runtime parameters, to choose specific sub-units when only
a generic unit has been specified, to prevent mutually exclusive units
from being included together, and to flag problems when dependencies are
not resolved by some included unit. Some of the Config files contain
additional information about unit interrelationships. As mentioned
earlier, starts from the file in the Simulation directory of the problem
being built.

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
files have as their first line . If the first line does not match this
string, then static mode is assumed and each line of the file is
interpreted verbatim as a directive.

If python mode is triggered, then the entire file is considered as valid
python source code (as if it were a .py). From this python code, a
function of the form is located and executed to generate the
configuration directives as an array (or any iterable collection) of
strings. The sole argument to genLines is a dictionary that maps setup
variable names to their corresponding string values.

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

-  | 
   | A comment. Can appear as a separate line or at the end of a line.

-  | 
   | Every unit and sub-unit designates one implementation to be the
     “default”, as defined by the keyword in its file. If no specific
     implementation of the unit or its sub-units is selected by the
     application, the designated default implementation gets included.
     For example, the file for the specifies Gamma as the default. If no
     specific implementation is explicitly included (, ), then this
     command instructs to include the Gamma implementation, as though
     had been placed in the file.

-  | 
   | Specifies a list of implementations that cannot be included
     together. If no instruction is given, it is perfectly legal to
     simultaneously include more than one implementation in the code.
     Using “\*” means that at most one implementation can be included.

-  | ...
   | Specifies that the current unit, sub-unit, or specific
     implementation is not compatible with the list of units, sub-units
     or other implementations that follows. issues an error if the user
     attempts a conflicting unit configuration.

-  | 
   | Specifies a unit requirement. Unit requirements can be general,
     without asking for a specific implementation, so that unit
     dependencies are not tied to particular algorithms. For example,
     the statement in a unit’s file indicates to that the physics/Eos
     unit is needed, but no particular equation of state is specified.
     As long as an implementation is included, the dependency will be
     satisfied. More specific dependencies can be indicated by
     explicitly asking for an implementation. For example, if there are
     multiple species in a simulation, the equation of state is the only
     valid option. To ask for it explicitly, use . Giving a complete set
     of unit requirements is helpful, because uses them to generate the
     units file when invoked with the -auto option.

-  | 
   | Requests that a unit be added to the Simulation. All requests are
     upgraded to a “REQUIRES” if they are not negated by a
     "-without-unit" option from the command line. If negated, the is
     ignored. This can be used to turn off profilers and other
     “optional” units which are included by default.

-  | 
   | Unlike , this keyword suggests that the current unit be used along
     with one of the specified units. The setup script will print
     details of the suggestions which have been ignored. This is useful
     in catching inadvertently omitted units before the run starts, thus
     avoiding a waste of computing resources.

-  | []
   | Specifies a runtime parameter. Parameter names are unique up to 20
     characters and may not contain spaces. Admissible types include , ,
     , and . Default values for and parameters must be valid numbers, or
     the compilation will fail. Default values must be enclosed in
     double quotes (). Default values must be or to avoid compilation
     errors. Once defined, runtime parameters are available to the
     entire code. Optionally, any parameter may be specified with the
     attribute (, ). If a user attempts to set a constant parameter via
     the runtime parameter file, an error will occur.

   The range specification is optional and can be used to specify valid
   ranges for the parameters. The range specification is allowed only
   for variables and must be enclosed in ’[]’.

   For a variable, the range specification is a comma-separated list of
   strings (enclosed in quotes). For a variable, the range specification
   is a comma-separated list of (closed) intervals specified by , where
   min and max are the end points of the interval. If min or max is
   omitted, it is assumed to be :math:`-\infty` and :math:`+\infty`
   respectively. Finally is a shortcut for . For example

   .. container:: codeseg

      PARAMETER pres REAL 1.0 [ 0.1 ... 9.9, 25.0 ... ] PARAMETER coords
      STRING "polar" ["polar","cylindrical","2d","3d"]

   indicates that is a REAL variable which is allowed to take values
   between 0.1 and 9.9 or above 25.0. Similarly is a string variable
   which can take one of the four specified values.

-  | 
   | Any line in a file is considered a parameter comment line if it
     begins with the token . The first token after the comment line is
     taken to be the parameter name. The remaining tokens are taken to
     be a description of the parameter’s purpose. A token is delineated
     by one or more white spaces. For example,

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

   Parameter comment lines are special because they are used by to build
   a formatted list of commented runtime parameters for a particular
   problem. This information is generated in the file in the directory.

-  | 
   | Registers variable with the framework with name and a variable type
     defined by . The script collects variables from all the included
     units, and creates a comprehensive list with no duplications. It
     then assigns defined constants to each variable and calculates the
     amount of storage required in the data structures for storing these
     variables. The defined constants and the calculated sizes are
     written to the file .

   The possible types for are as follows:

   -  | 
      | This solution variable is represented in form, , it represents
        the density of a conserved extensive quantity. The prime example
        is a variable directly representing mass density. Energy
        densities, momentum densities, and partial mass densities would
        be other examples (but these quantities are usually represented
        in form instead).

   -  | 
      | This solution variable is represented in form, , it represents
        quantities whose nature is
        :math:`\hbox{extensive quantity}\,\mathop{\mathrm{per}}\,\hbox{mass unit}`.
        Examples are specific energies, velocities of material (since
        they are equal to momentum per mass unit), and abundances or
        mass fractions (partial density divided by density).

   -  | 
      | This is the default and need not be specified. This type should
        be used for any variables that do not clearly belong to one of
        the previous two categories.

   In the current version of the code, the attribute is only used to
   determine which variables should be converted to conservative form
   for certain Grid operations that may require interpolation (,
   prolongation, guardcell filling, and restriction) when one of the
   runtime parameters or is set . Only variables of type are converted:
   values are multiplied cell-by-cell with the value of the variable,
   and potential interpolation results are converted back by
   cell-by-cell division by values after interpolation.

   Note that therefore

   -  variable types are irrelevant for uniform grids,

   -  variable types are irrelevant if neither nor is , and

   -  variable types (and conversion to and from conserved form) only
      take effect if a

      .. container:: codeseg

         VARIABLE dens ...

      exists.

   An has the syntax :math:`|` , where stands for a as defined in .
   These roles are used within implementations of the interface, via the
   subroutines and , to map variables from Grid data structures to the
   array that understands, and back. For example,

   .. container:: codeseg

      VARIABLE eint TYPE: PER_MASS EOSMAPIN: EINT

   means that within , the component of will be treated as the grid
   variable in the “internal energy” role for the purpose of
   constructing input to , and

   .. container:: codeseg

      VARIABLE gamc EOSMAPOUT: GAMC

   means that within , the component of will be treated as the grid
   variable in the role for the purpose of returning results from
   calling to the grid. The specification

   .. container:: codeseg

      VARIABLE pres EOSMAP: PRES

   has the same effect as

   .. container:: codeseg

      VARIABLE pres EOSMAPIN: PRES EOSMAPOUT: PRES

   Note that not all roles defined in are necessarily meaningful or
   actually used in a given Eos implementation. An for a is only used in
   an invocation when the optional argument is absent or has a value of
   .

-  | 
   | This keyword has the same meaning for face-centered variables, that
     does for cell-centered variables. It allocates space in the grid
     data structure that contains face-centered physical variables for
     “name”. See for more information

   For , see above under . An for is only used when is called with an
   optional argument of , , or .

-  | 
   | Registers flux variable with the framework. When using an adaptive
     mesh, flux conservation is needed at fine-coarse boundaries. uses a
     data structure for this purpose, the flux variables provide indices
     into that data structure. See for more information.

-  | 
   | This keyword is used in connection with the grid scope scratch
     space for cell-centered data supported by Flash-X. It allows the
     user to ask for scratch space with “name”. The scratch variables do
     not participate in the process of guardcell filling, and their
     values become invalid after a grid refinement step. While users can
     define scratch variables to be written to the plotfiles, they are
     not by default written to checkpoint files. Note this feature
     wasn’t available in Flash-X2. See for more information.

-  | 
   | This keyword is used in connection with the grid scope scratch
     space for face-centered data, it is identical in every other
     respect to .

-  | 
   | This keyword is used for specifying instances of general purpose
     grid scope scratch space. The same space can support cell-centered
     as well as face-centered data. Like other scratch data structures,
     the variables in this data structure can also be asked with “name”
     and do not participate in guardcell filling.

   For , see above under . An for is only used when is called with an
   optional argument of .

-  | 
   | If a quantity is defined with keyword MASS_SCALAR, space is created
     for it in the grid “unk” data structure. It is treated like any
     other variable by , but the hydrodynamic unit treats it
     differently. It is advected, but other physical characteristics
     don’t apply to it. If the optional “RENORM” is given, this
     mass-scalar will be added to the renormalization group of the
     accompanying group name. The hydrodynamic solver will renormalize
     all mass-scalars in a given group, ensuring that all variables in
     that group will sum to 1 within an individual cell. See

   For , see above under . An for a may be used in an invocation when
   the optional argument is absent or has a value of .

   .. container:: flashtip

      It is inadvisable to name variables, species, and mass scalars
      with the same prefix, as post-processing routines have difficulty
      deciphering the type of data from the output files. For example,
      don’t create a variable “temp” to hold temperature and a mass
      scalar “temp” indicating a temporary variable. Although the file
      can distinguish between these two types of variables, many
      plotting routines cannot.

-  | 
   | This keyword associates a with mapping and initialization sub-units
     of unit to operate on this particle type during the simulation.
     Here, describes the method used to map the particle properties to
     and from the mesh (see ), describes the method used to distribute
     the particles at initialization, and describes the method used to
     advance the associated particle type in time (see , and in general
     ). This keyword has been introduced to facilitate inclusion of
     multiple particle types in the same simulation. It imposes certain
     requirements on the use of the and . Particles (of any type,
     whether called or anything else) do not have default methods for
     initialization, mapping, or time integration, so a directive in a
     file (or an equivalent setup option, see ) is the only way to
     specify the appropriate implementations of the Particles subunits
     to be used. The declaration should be accompanied by appropriate
     “REQUESTS” or “REQUIRES” directives to specify the paths of the
     appropriate subunit implementation directories to be included. For
     clarity, our technique has been to include this information in the
     simulation directory files only. All the currently available
     mapping and initialization methods have a corresponding identifier
     in the form of preprocessor definition in . The user may select any
     name, but the , and must correspond to existing identifiers defined
     in . This is necessary to navigate the data structure that stores
     the particle type and its associated mapping and initialization
     methods. Users desirous of adding new methods for mapping or
     initialization should also update the file with additional
     identifiers and their preprocessor definitions. Note, it is
     possible to use the same methods for different particle types, but
     each particle type name must only appear once. Finally, the
     Simulations file is also expected to request appropriate
     implementations of mapping and initialization using the keyword ,
     since the corresponding Config files do not specify a default
     implementation to include. For example, to include particle types
     with mapping, initialization,and for advancing in time the
     following code segment should appear in the file of the directory.

   .. container:: codeseg

      PARTICLETYPE passive INITMETHOD lattice MAPMETHOD quadratic
      ADVMETHOD Euler REQUIRES Particles/ParticlesMain REQUESTS
      Particles/ParticlesMain/passive/Euler REQUESTS
      Particles/ParticlesMapping/Quadratic REQUESTS
      Particles/ParticlesInitialization/Lattice

-  | 
   | This keyword indicates that the particles data structure will
     allocate space for a sub-variable “NAME_PART_PROP.” For example if
     the Config file contains

   .. container:: codeseg

      PARTICLEPROP dens

   then the code can directly access this property as

   .. container:: codeseg

      particles(DENS_PART_PROP,1:localNumParticles) = densInitial

   may be REAL or INT, however INT is presently unused. See for more
   information and examples.

-  | TO FROM
   | This keyword maps the value of the particle property to the
     variable . can take the values VARIABLE, MASS_SCALAR, SPECIES,
     FACEX, FACEY, FACEZ, or one of SCRATCH types (SCRATCHVAR/
     SCRATCHCENTERVAR, SCRATCHFACEXVAR. SCRATCHFACEYVAR,
     SCRATCHFACEZVAR) These maps are used to generate , which takes the
     particle property and returns and . For example, to have a particle
     property tracing density:

   .. container:: codeseg

      PARTICLEPROP dens REAL PARTICLEMAP TO dens FROM VARIABLE dens

   or, in a more advanced case, particle properties tracing some
   face-valued function Mag:

   .. container:: codeseg

      PARTICLEPROP Mag_x REAL PARTICLEPROP Mag_y REAL PARTICLEPROP Mag_z
      REAL PARTICLEMAP TO Mag_x FROM FACEX Mag PARTICLEMAP TO Mag_y FROM
      FACEY Mag PARTICLEMAP TO Mag_z FROM FACEZ Mag

   Additional information on creating files for particles is obtained in
   .

-  | [TO ]
   | An application that uses multiple species uses this keyword to
     define them. See for more information. The user may also specify an
     optional number of ions for each element, . For example, TO creates
     9 spaces in for Oxygen, that is, a single space for Oxygen and 8
     spaces for each of its ions. This is relevant to simulations using
     the unit. (Omitting the optional specifier is equivalent to
     specifying 0).

-  | 
   | Declares that all files matching the given wildcard in the unit
     directory should be copied over to the object directory. For
     example,

   .. container:: codeseg

      DATAFILES \*.dat

   will copy all the “.dat” files to the object directory.

-  | 
   | Declares that all subdirectories must be recursively included. This
     usually marks the end of the high level architecture of a unit.
     Directories below it may be third party software or a highly
     optimized solver, and are therefore not required to conform to
     Flash-X architecture.

   Without a , the current directory (, the one containing the file with
   the keyword) is marked as a kernel directory, so code from all its
   subdirectories (with the exception of subdirectories whose name
   begins with a dot) is included. When a is given, then that
   subdirectory must exist, and it is treated as a kernel directory in
   the same way.

   Note that currently the script can process only one directive per
   file.

-  | 
   | Specifies a library requirement. Different Flash-X units require
     different libraries, and they must inform so it can link the
     libraries into the executable. Some valid library names are .
     Support for external libraries can be added by modifying the
     site-specific files to include appropriate Makefile macros. It is
     possible to use internal libraries, as well as switch libraries at
     setup time. To use these features, see

-  | 
   | Specifies that the file should be used only when the unit is
     included. This keyword allows a unit to have multiple
     implementations of any part of its functionality, even down to the
     kernel level, without the necessity of creating children for every
     alternative. This is especially useful in Simulation setups where
     users may want to use different implementations of specific
     functions based upon the units included. For instance, a user may
     wish to supply his/her own implementation of , instead of using the
     default one provided by Flash-X. However, this function is aware of
     the internal workings of Grid, and has different implementations
     for different grid packages. The user could therefore specify
     different versions of his/her own file that are intended for use
     with the different grids. For example, adding

   .. container:: codeseg

      LINKIF Grid_markRefineDerefine.F90.ug Grid/GridMain/UG LINKIF
      Grid_markRefineDerefine.F90.pmesh Grid/GridMain/paramesh

   to the file ensures that if the application is built with UG, the
   file will be linked in as , whereas if it is built with Paramesh2 or
   Paramesh4 0 or Paramesh4dev, then the file will be linked in as .
   Alternatively, the user may want to provide only one implementation
   specific to, say, . In this case, adding

   .. container:: codeseg

      LINKIF Grid_markRefineDerefine.F90 Grid/GridMain/paramesh

   to the Config file ensures that the user-supplied file is included
   when using (either version), while the default Flash-X file is
   included when using UG.

-  | 
   | Instructs setup to add the PreProcessor symbols and to the
     generated . Here is converted to uppercase. These pre-process
     symbols can be used in the code to distinguish between which units
     have been used in an application. For example, a Fortran subroutine
     could include

   .. container:: codeseg

      #ifdef Flash-X_GRID_UG ug specific code #endif

      #ifdef Flash-X_GRID_PARAMESH3OR4 pm3+ specific code #endif

   By convention, many preprocessor symbols defined in Config files
   included in the Flash-X code distribution start with the prefix
   “Flash-X\_”.

-  | 
   | This tells that the specified “Setup Variables” are being used in
     this file. The variables initialize to an empty string if no values
     are specified for them. Note that commas are required if listing
     several variables.

-  | 
   | When links several implementations of the same function, it ensures
     that implementations of children override that of the parent. Its
     method is to lexicographically sort all the names and allow
     implementations occurring later to override those occurring
     earlier. This means that if two siblings implement the same code,
     the names of the siblings determine which implementation wins.
     Although it is very rare for two siblings to implement the same
     function, it does occur. This keyword permits the file to override
     the lexicographic order by one preferred by the user. Lexicographic
     ordering will prevail as usual when deciding among implementations
     that are not explicitly listed.

-  | 
   | Allows an application to choose the stencil size for updating grid
     points. The stencil determines the number of guardcells needed. The
     PPM algorithm requires :math:`4` guardcells, hence that is the
     default value. If an application specifies a smaller value, it will
     probably not be able to use the default AMR Grid interpolation; see
     the flag for additional information.

-  | 
   | This causes to abort with the specified error message. This is
     usually used only inside a conditional IF/ENDIF block (see below).

-  | 
   | A conditional block is of the following form:

   .. container:: codeseg

      IF cond ... ELSEIF cond ... ELSE ... ENDIF

   where the and blocks are optional. There is no limit on the number of
   blocks. “...” is any sequence of valid file syntax. The conditional
   blocks may be nested. “cond” is any boolean valued Python expression
   using the setup variables specified in the .

-  | 
   | Declares an array of variables that will be partitioned across the
     replicated meshes. Using various preprocessor macros in
     Simulation.h each copy of the mesh can determine at runtime its own
     subset of indexes into this global array. This allows an easy form
     of parallelism where regular "replicated" mesh variables are
     computed redundantly across processors, but the variables in the
     "non-replicated" array are computed in parallel.

   -  | : must be either or

   -  : the name of this variable array. It is suggested that it be all
      capital letters, and must conform to what the C preprocessor will
      consider as a valid symbol for use in a statement.

   -  | : a positive integer specifying the maximum number of elements
        from the global variable array a mesh can hold. This is the
        actual number of variables that are allocated on each processor,
        though not all of them will necessarily be used.

   -  | : the name of a runtime parameter which dictates the size of
        this global array of variables.

   -  | : a string representing how the elements of the array will be
        named when written to the output files. The question mark
        character is used as a placeholder for the digits of the array
        index. As an example, the format string will generate the
        dataset names , , , etc. This string must be no more than four
        characters in length.

   The number of meshes is dictated by the runtime parameter . The
   following constraint must be satisfied or Flash-X will fail at
   runtime:

   .. math:: globalparam \le meshCopyCount * localmax

   The reason for this restriction is that is the maximum number of
   array elements a mesh can be responsible for, and is the number of
   meshes, so their product bounds the size of the array.

   Example:

   Config file:

   .. container:: fcodeseg

      NONREP MASS_SCALAR A 4 numA a??? NONREP MASS_SCALAR B 5 numB b???

   flash.par file:

   .. container:: fcodeseg

      meshCopyCount = 3 numA = 11 numB = 15

   In this case two non-replicated mass-scalar arrays are defined, and .
   Their lengths are specified by the runtime parameters and
   respectively. is set to its maximum value of
   :math:`5*meshCopyCount=15`, but is one less than its maximum value of
   :math:`4*meshCopyCount=12` so at runtime one of the meshes will not
   have all of its variables in use. The dataset names generated by IO
   will take the form and .

   The preprocessor macros defined in for these arrays will have the
   prefixes and respectively. For details about these macros and how
   they will distribute the array elements across the meshes see .

.. _`Sec:SetupMakefile`:

Creating a Site-specific 
------------------------

If does not find your hostname in the directory it picks a default based
on the operating system. This is not always correct but can be used as a
template to create a for your machine. To create a Makefile specific to
your system follow these instructions.

-  Create the directory , where is the hostname of your machine.

-  Start by copying to

-  Use to help identify the locations of various libraries on your
   system. The script scans your system and displays the locations of
   some libraries. You must note the location of library as well. If
   your compiler is actually an mpi-wrapper (), you must still define in
   your site specific as the empty string.

-  Edit to provide the locations of various libraries on your system.

-  Edit to specify the  and C compilers to be used.

.. container:: flashtip

   The Makefile.h *must* include a compiler flag to promote Fortran to .
   Flash-X performs all communication of Fortran using type, and assumes
   that Fortran are interoperable with C in the I/O unit.

Files Created During the Process
--------------------------------

When is run it generates many files in the directory. They fall into
three major categories:

-  Files not required to build the Flash-X executable, but which contain
   useful information,

-  Generated or C code, and

-  Makefiles required to compile the Flash-X executable.

Informational files
~~~~~~~~~~~~~~~~~~~

These files are generated before compilation by . Each of these files
begins with the prefix for easy identification.

.. container:: tabular

   \|lp0.60\| & contains the options with which was called and the
   command line resulting after shortcut expansion

   & contains the list of libraries and their arguments (if any) which
   was linked in to generate the executable

   & contains the list of all units which were included in the current
   setup

   & contains a list of all pre-process symbols passed to the compiler
   invocation directly

   & contains the exact compiler and linker flags

   & contains the list of runtime parameters defined in the files
   processed by

   & contains the list of variables, fluxes, species, particle
   properties, and mass scalars used in the current setup, together with
   their descriptions.

Code generated by the call
~~~~~~~~~~~~~~~~~~~~~~~~~~

These routines are generated by the setup call and provide
simulation-specific code.

.. container:: tabular

   \|lp0.55\| & contains code for the subroutine which returns the setup
   and build time as well as code for which returns the *uname* of the
   system used to setup the problem

   & contains code which returns build statistics including the actual
   call as well as the compiler flags used for the build

   & contains code to retrieve the number and list of flashUnits used to
   compile code

   & contains code to retrieve the version of Flash-X used for the build

   & contains simulation specific preprocessor macros, which change
   based upon setup unlike . It is described in

   & contains code to map an index described in to a string described in
   the file.

   & contains code to map a string described in the file to an integer
   index described in the file.

   & contains a mapping between particle properties and grid variables.
   Only generated when particles are included in a simulation.

   & contains code to make a data structure with information about the
   mapping and initialization method for each type of particle. Only
   generated when particles are included in a simulation.

.. _`Sec:unitMakefiles`:

Makefiles generated by 
~~~~~~~~~~~~~~~~~~~~~~

Apart from the master , generates a makefile for each unit, which is
“included” in the master . This is true even if the unit is not included
in the application. These unit makefiles are named and are a
concatenation of all the Makefiles found in unit hierarchy processed by
.

For example, if an application uses , the file will be a concatenation
of the Makefiles found in

-  ,

-  ,

-  ,

-  , and

-  

As another example, if an application does not use , then is just the
contents of at the API level.

Since the order of concatenation is arbitrary, the behavior of the
Makefiles should not depend on the order in which they have been
concatenated. The makefiles inside the units contain lines of the form:

.. container:: codeseg

   Unit += file1.o file2.o ...

where is the name of the unit, which was in the example above.
Dependency on data modules files *need not be specified* since the setup
process determines this requirement automatically.

.. _`Sec:hybridSetup`:

Setup a hybrid MPI+OpenMP Flash-X application
---------------------------------------------

There is the experimental inclusion of Flash-X multithreading with
OpenMP in the beta release. The units which have support for
multithreading are split hydrodynamics `[Sec:PPM] <#Sec:PPM>`__, unsplit
hydrodynamics
`[Sec:unsplit hydro algorithm] <#Sec:unsplit hydro algorithm>`__, Gamma
law and multigamma EOS `[Sec:Eos Gammas] <#Sec:Eos Gammas>`__, Helmholtz
EOS `[Sec:Eos Helmholtz] <#Sec:Eos Helmholtz>`__, Multipole Poisson
solver (improved version (support for 2D cylindrical and 3D cartesian))
`[Sec:GridSolversMultipoleImproved] <#Sec:GridSolversMultipoleImproved>`__
and energy deposition
`[Sec:EnergyDeposition] <#Sec:EnergyDeposition>`__.

The Flash-X multithreading requires a MPI-2 installation built with
thread support (building with an MPI-1 installation or an MPI-2
installation without thread support is possible but strongly
discouraged). The Flash-X application requests the thread support level
to ensure that the MPI library is thread-safe and that any OpenMP thread
can call MPI functions safely. You should also make sure that your
compiler provides a version of OpenMP which is compliant with at least
the OpenMP 2.5 (200505) standard (older versions may also work but I
have not checked).

In order to make use of the multithreaded code you must setup your
application with one of the setup variables , or equal to , e.g.

.. container:: codeseg

   ./setup Sedov -auto threadBlockList=True ./setup Sedov -auto
   threadBlockList=True +mpi1 (compatible with MPI-1 - unsafe!)

When you do this the setup script will insert instead of in the
generated Makefile. If it is equal to :math:`1` the Makefile will
prepend an OpenMP variable to the , , variables.

.. container:: flashtip

   In general you should not define , and in your . It is much better to
   define , , , , , , , and in your . The setup script will then
   initialize the , and variables in the Makefile appropriately for an
   optimized, test or debug build.

The OpenMP variables should be defined in your and contain a compiler
flag to recognize OpenMP directives. In most cases it is sufficient to
define a single variable named , but you may encounter special
situations when you need to define , and . If you want to build Flash-X
with the GNU Fortran compiler and the GNU C compiler then your should
contain

.. container:: codeseg

   OPENMP = -fopenmp

If you want to do something more complicated like build Flash-X with the
Lahey Fortran compiler and the GNU C compiler then your should contain

.. container:: codeseg

   OPENMP_FORTRAN = –openmp -Kpureomp OPENMP_C = -fopenmp OPENMP_LINK =
   –openmp -Kpureomp

When you run the hybrid Flash-X application it will print the level of
thread support provided by the MPI library and the number of OpenMP
threads in each parallel region

.. container:: codeseg

   : Called MPI_Init_thread - requested level 2, given level 2
   [Driver_initParallel]: Number of OpenMP threads in each parallel
   region 4

Note that the Flash-X application will still run if the MPI library does
not provide the requested level of thread support, but will print a
warning message alerting you to an unsafe level of MPI thread support.
There is no guarantee that the program will work! I strongly recommend
that you stop using this Flash-X application - you should build a MPI-2
library with thread support and then rebuild Flash-X.

We record extra version and runtime information in the Flash-X log file
for a threaded application. Table `1.4 <#tab:flash_openmp_logs>`__ shows
log file entries from a threaded Flash-X application along with example
safe and unsafe values. All cells colored red show unsafe values.

.. container:: center

   .. container::
      :name: tab:flash_openmp_logs

      .. table:: Log file entries showing safe and unsafe threaded
      Flash-X applications

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
         | OpenMP      | 2        | 2           | 2           | 2           |
         | threads/MPI |          |             |             |             |
         | task:       |          |             |             |             |
         +-------------+----------+-------------+-------------+-------------+
         | OpenMP      | 200805   | 200505      | 200505      | 200805      |
         | version:    |          |             |             |             |
         +-------------+----------+-------------+-------------+-------------+
         | Is          | T        | T           | T           | F           |
         | “\_OPENMP”  |          |             |             |             |
         | macro       |          |             |             |             |
         | defined:    |          |             |             |             |
         +-------------+----------+-------------+-------------+-------------+

The Flash-X applications in Table `1.4 <#tab:flash_openmp_logs>`__ are
unsafe because

#. we are using an MPI-1 implementation.

#. we are using an MPI-2 implementation which is not built with thread
   support - the “MPI thread support in OpenMPI” Flash tip may help.

#. we are using a compiler that does not define the macro when it
   compiles source files with OpenMP support (see OpenMP standard). I
   have noticed that Absoft 64-bit Pro Fortran 11.1.3 for Linux x86_64
   does not define this macro. We use this macro in to conditionally
   initialize MPI with . If you find that is not defined you should
   define it in your in a manner similar to the following:

   .. container:: codeseg

      OPENMP_FORTRAN = -openmp -D_OPENMP=200805

You should not setup a Flash-X application with both and equal to -
nested OpenMP parallelism is not supported. For further information
about Flash-X multithreaded applications please refer to Chapter
`[Chp:MultithreadedFlash-X] <#Chp:MultithreadedFlash-X>`__.

.. [1]
   if a machine has multiple hostnames, setup tries them all

.. [2]
   Formerly, (in Flash-X2) it was located in the Flash-X root directory

.. [3]
   Formerly, (in Flash-X2) it was located in the Flash-X root directory

.. [4]
   Formerly, (in Flash-X2) it was located in the Flash-X root directory

.. [5]
   All non-integral values not equal to True/False/Yes/No/On/Off are
   considered to be string values
