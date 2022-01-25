.. _`Chp:The Setup Tool`:

The Setup Tool
===================================

The configuration toolchain effectively implements the Flash-X
software architecture. The encapsulation and inheritance of the code
is implemented and enforced by this tool. It relies upon Flash-X's
domain specific configuration language (DSCL) that encodes
meta-information about the components in the accompanying
"Config" files. A Config file is unique to a unix directory and has
all the meta-information about files in that directory. It can also
include information about how to traverse subdirectories of the
corresponding component.

.. _`Sec:DSCL Keywords`:

Config file Keywords and syntax
~~~~~~~~~~~~~~~~~~~~~~~~~
The syntax of DSCL includes two types of keywords. The directive
keywords define actions to be taken by the setup tool, while the non
directive keywords encode information about the directives. The non
directive keywords are associated with specific directives keyworks. 

*DEFAULT* governs the traversal of the subtree by the setup
tool. If one of the subdirectories is not explicitly specified for
traversal the default one is chosen.

*REQUIRES* specifies a unit, subunit, or an arbitray component to be
included in the application. The usage is 
::
   REQUIRES <path>, where path starts from subdirectories in
   $FLASHXHOME/source.

For example "RQUIRES Grid" would indicate that Grid unit is to be
included but specifics of implementation can be specified
later. Whereas "REQUIRES Grid/GridMain/AMR/AMReX indicates that it is
AMReX based implementation of the Grid unit that is to be
included. Note that there is no restriction on depth of directory
hierarchy that can be specified through this keyword. It specifies a
hard dependency; the application cannot be built without include files
from the specified path.

 *REQUESTS* is similar to REQUIRES except that it is a soft
 dependency. It can be overridden through either a commandline
 specification ot through dependencies specified in higher priority
 locations in the source tree.

 *CONFLICTS* indicates units, subunits or components that are
 incompatible with the current component. For example
 ::
    CONFLICTS Paramesh4

appearing in the Config file of Grid/GridSolvers/Multigrid tells the setup tool
that this implementation of muligrid solver will not work with PARAMESH.

 *EXCLUSIVE*  specifies a list of implementations of the corresponding
 component that cannot co-exist in an instance of application
 configuration. The usage is
 ::
    EXCLUSIVE UG AMR

 indicating that UG and AMR cannot be included in the same application.

 *VARIANTS* is meant to enable inclusion of multiple alternative
 implementations of a component in the same instance of application
 configuration. For example, one may want to use both CPU and GPU to
 compute. A component that has valid implementations for both can
 specify
 ::
    VARIANTS Inhost Offload Null
    
 Then if the setup options specify inclusion of both implementations,
 the names of the relevant files and function/subroutine interfaces are
 modified to have unique names. If, on the other hand, only one of the
 implementations is to be included, the default is Null and all the
 names remain unmodified.

 *SUGGEST* is a helper keyword used to indicate that the "suggested"
 components ought to be included. Its only use is to catch
 inadverntenly omitted components before execution starts.

*PARAMETER* declares runtime parameters. The non-directive keywords
associated with PARAMETER are *REAL*, *INTEGER*, *STRING*, *BOOLEAN*
and *CONSTANT* used to describe the datatype of the
PARAMETER. Optionally, a range of values can also be associate with a
PARAMETER as shown in the examples.
::
   PARAMETER pres REAL 1.0 [ 0.1 ... 9.9, 25.0 ... ]
   PARAMETER coords  STRING "polar" ["polar","cylindrical","2d","3d"]


*VARIABLE* defines cell-centered state variables needed by the component. The
variables can be of various *TYPE*. *PER_VOLUME* are conserved
quantities and represent the density of the conserved
quantity such as mass density, energy density etc. *PER_MASS* that are
by nature extensive quantities such as specific energies, velocities
of materials, abundances etc.  *GENERIC* variables are those that do
not fit the above two categories.  VARIABLES also have other
attributes associated with them. These are *ADVECT/NOADVECT* depending
on whether the variable obeys advection equation, *RENORM/NORENORM*
that are to be renormalized or not, and *CONSERVE/NOCONSERVE*
depending upon whether they obey conservation laws. *EOSMAPIN* and
*EOSMAPOUT* define the map between the state variables index in the
data structure handed to the equation of state unit from that of the
Unit invoking the equation of state, as shown in the example below.
::
   VARIABLE eint TYPE: PER_MASS EOSMAPIN: EINT
   VARIABLE gamc EOSMAPOUT: GAMC


*FACEVAR* are similar to VARIABLES except that they are face-centered
as opposed to cell-centered quantities.

*MASS_SCALAR* defines space holders in the state variables data
structure that are only advected but do not participate otherwise in
the evolution of the solution.

*PARTICLETYPE* defines the type of Particles being used in the
simulation. Flash-X currently supports only tracer particles, though
hooks and infrastructure exists for several other flavors such as
active particles with mass or charge deposition.

*INITMETHOD*, *MAPMETHOD*, and *ADVMETHOD* are keywords associated
with PARTICLETYPE, where they indicate the methods to be used for
initialization of particles positions, interpolation methods to be
used to map associated quantities to-and-from the particle position
and mesh, and the method for advancing particles of that type in
time. To function correctly, for all the  methods specified by the
above three keywords the compoent implementing the corresponding
implementation should be specified with either a REQUIRES or a
REQUESTS directive in the Simulations directory config file, or should
be asked to be included through commandline options on the setup
command. An example of specifying particletype to be included along with
it corresponding methods is
::
   PARTICLETYPE passive INITMETHOD lattice MAPMETHOD quadratic
   ADVMETHOD Euler
   REQUIRES Particles/ParticlesMain
   REQUESTS Particles/ParticlesMain/passive/Euler
   REQUESTS Particles/ParticlesMapping/Quadratic
   REQUESTS Particles/ParticlesInitialization/Lattice


  *TO*, and  *FROM* indicate the map of which particle property
  corresponds to which state variable. This information is needed to
  interpolate between the particle position and the mesh.


*SPECIES* defines multiples species if they are being used in the
simulation.

*DATAFILES* is used to indicate that files with non-standard
extensions that are not usually copied over to the object directory
are copied. For example
::
   DATAFILES *.dat

 will copy all files with .dat extension in the component directory to
 the object directory.

*KERNEL* is a special keyword that indicates that all the
subdirectories from this level are to be treated as Unix organization
directories only, and are to be recursively included.

*LIBRARY* indicates that this component needs this specified library
to function correctly. Internal libraries are built using the
accompanying build script, while for the external libraries the
correct path must be provided in the Makefile.h for the site
directory.

*LINKIF* is keywork that permits a file to be included only if the
corresponding component is also included in the simulation. Typically
these files may have non-standard extensions which are stripped off at
the time of inclusion. For example
::
   LINKIF Grid_markRefineDerefine.F90.ug Grid/GridMain/UG
   LINKIF Grid_markRefineDerefine.F90.pmesh Grid/GridMain/paramesh


Will include Grid_markRefineDerefine.F90.ug as
Grid_markRefineDerefine.F90 if UG is the Grid implementation included
while if PARAMESH Is included then Grid_markRefineDerefine.F90.pmesh
will become  Grid_markRefineDerefine.F90.  

*PPDEFINE* sets up preprocessor symbols that are included in the
generated global constants in the file "Simulation.h". These symbols
can be handy in correlating the behavior of components through
#ifdefs depending on which implementation of the component is
included as needed without resorting to wholesale code duplication. 

*USESETUPVARS* tells the setup tool which setup variables are being
used in this Config file.  
  
*CHILDORDER*  controls the order of traversal of subtrees by the setup
 tool. By default the instance of an implementation encountered later
 overrides an earlier instance. Thus specifying CHILDORDER ensures
 that the desired implementation is picked if it is not explicitly
 specified. 

*GUARDCELLS* specifies the number of guardcells to be used in the
stencil for updating the grid points.

*SETUPERROR* causes the setup tool to abort with the specified error
message

*IF*, *ELSEIF*, *ELSE*, *ENDIF* provide the functionality of
conditional block to the DSCL. 

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

As an example, here is a Config file in python mode that
registers runtime parameters named indexed_parameter_x where x ranges
from 1 to NP and NP is a setup line variable.

.. container:: fcodeseg

   ##python:genLines

   # We define genLines as a generator with the very friendly "yield"
   syntax. # Alternatively, we could have genLines return an array of
   strings or even # one huge multiline string. def genLines(setupvars):
   # emit some directives that dont depend on any setup variables yield

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

  
-

.. _`Sec:The Setup Tool`:

The Setup Tool
~~~~~~~~~~~~~~~~~~~~~~~~~

The setup tool parses and interprets the Config files to assemble an
application instance in the specified object directory. It has three
primary actions to perform.


 #.  Compile a list of source files to be included in the
     application. This compilation process ends with all the needed
     files assembled in the object directory. Some may be linked
     directly from their location in the source tree, while others may
     have been generated through some additional actions taken by the
     setup tool described in Figure ... 

 #.  Compile and initialize a list of runtime parameters needed by the simulation

 #.  Generate Makefile.Unit for each unit that will be included in the
     main Makefile used for compiling the source code. Note that the
     Flash-X build system assumes GNU Make. 


The setup tool traverses the Flash-X source tree starting from the
directory hosting the specific application definition. This starting
directory is essentially the selected implementation of the
“Simulation” unit.  It accumulates  units, subunits and other
compoments from where source code is to be collected through recursive
traversal of all encountered dependencies. While traversing the source
tree the setup tool uses the following inheritance rules to arbitrate
on versions of the source code functions, versions of key definitions
and initial values of runtime parameters. 

* All files with valid source extensions (such as .F90, .c, .h) are
  added to the list and are linked into the object directory 

  * If a file with identical name in already included in the list the
    existing link in the object directory is removed and replaced with
    a link to the newly encountered file 
  
  * The source files in the Simulation unit are the last ones to be
    added to  the list, and therefore can override any file from the
    source tree in the object directory 
  
* If a file has a .ini extension it indicates to the setup tool that
  it contains definitions for the keys. All the keys defined in the
  file are added to the list of available keys.
  
  * If a key already existed in the list, its defintion is replaced by
    the most recently encountered definition.  
  * Similar to source files, a definition in the Simulation unit
    overrides any existing definition of a key.  
  
* An -mc appended to the name of a file indicates to the setup tool
  that the file is augmented with keys and needs to be translated 

  * An augmented file foo.F90-mc may have a *Novariant* directive, in
    which case the keys are replaced by the corresponding
    definitions and the emitted code is placed in a file foo.F90 in
    the object directory. Note that foo.F90 never comes into 
    existence anywhere in the source tree, so foo.F90 in the object
    directory is a physical file, not a link.
  * If the aumented file does not have *Novariant* directive then the
    setup tool generates its variants. 
  
    * It looks for paths specified for the variants in the list of
      REQUIRES. 
    * For every variant it temporarily addes the keys defined in the
      path to list of available keys, which are removed from the list
      as soon as the translated code is emitted. 
    * The emitted code is placed in the object directory with the name
      foo_thisvariant.F90.
    * It is assumed that the function or subroutine names are defined
      to be variant dependent in these files. For details on how to do
      this correctly see .....  
    * In Makefile.Unit instances of foo.o are replaced with
      foo_variants.o for all available variants. 

* If any file has the *Reorder* directive it undergoes a code
  transformation where the order of indices in the arrays is changed
  as specified. The emitted code is placed in a file with the same
  name as the original file and the link to the source tree is removed. 
    
* Runtime parameters defined in the Config file are added to the list
  of runtime parameters and initialized with the specified value.
  
  * If a runtime parameter already exists in the list its value is
    replaced with the most recently encountered value  
  * The Simulation unit is the last one to be processed, therefore
    initial values specified in its Config file override other values. 

The setup tool looks for site-dependent configuration information by
looking for a directory where is the hostname of the machine on which
Flash-X is running. [1]_ Failing this, it looks in for a directory with
the same name as the output of the command. The site and operating
system type can be overridden with the and command-line options to the
command. Only one of these options can be used at one time. The
directory for each site and operating system type contains a makefile
fragment that sets command names, compiler flags, library paths, and any
replacement or additional source files needed to compile Flash-X for
that specific machine and machine type.

Note that the Flash-X build
system assumes that the command invokes GNU Make and is unlikely to work
properly with other implementations of the command. On some systems it
may be necessary to invoke GNU Make under the name .


Setup Options
---------------

The setup script accepts a large number of command line arguments which
affect the simulation in various ways. These arguments are divided into
three categories:

#. *Setup Options* (example: -auto ) begin with a dash and are built into the
   setup script itself. Many of the most commonly used arguments are
   setup options.

#. *Setup Variables* (example: species=air ) are defined by individual
   units as explained earlier ...

#. *Setup Shortcuts* (example: +noio) begin with a plus symbol and are
   essentially macros which automatically include a set of setup
   variables and/or setup options. New setup shortcuts can be easily
   defined by including them in bin/setup_shortcuts

.. _`Sec:ListSetupArgs`:

Comprehensive List of Setup Arguments
-------------------------------------
  *-verbose=*  instructs the level of verbosity in progress messages
  printed by the setup tool. Different levels (in order of increasing verbosity) are
  ERROR, IMPINFO, WARN, INFO, DEBUG}. The default is WARN.
  
 *-auto* indicates to the setup tool that it should select *DEFAULT*
 in every traversed path unless an alternative is explicitly specified
 either at commandline or in one of the already traversed Config
 files. In the process a text file containing a list of all traversed
 paths is created and is placed in the object directory. 

 *-[123]d specifies the dimensionality of the simulation. The default
 is 2d.

 *-maxblocks=* is relevant only when using PARAMESH. It lets the AMR
 know how many blocks to allocate for the state variables. Note that
 if the number of blocks generated on a process exceeds maxblocks the
 execution will abort.

*-nxb=/ -nyb=  / -nzb= * specify the number of data points (also
called cells)  along each dimension of the every block in the setup 

*-debug /-opt /-test* are options used to select the level of
optimization to used by the compilers. The defauls is -opt. The
Makefile.h in the site directory defines the flags associated with
each of these options, and the users can customize them as they want.

*-objdir=<dir>* allows the user to specify the path where they wish
the executable to be built. The default is *object* at the same level
as the source directory. 

*-with-unit=<path>* is used to specify to the setup tool that the
source code at the specified path is to be included. The specification
of the path on commandline can override any dependency in the
traversal that had a *REQUESTS* associated with it. And it can add
dependencies that were not included with -auto option.

*-without-unit=<path>* is used to tell the setup tool not to include
the code in the specified path. This option can also override
*REQUIRES* encountered during the traversal. 

*-geometry=<geometry>* is used to specify one of the supported
geometries <cartesian, cylindrical, spherical, polar>, the default is
cartesian.

*-defines=<def>*/  causes the specified pre-processor symbols to be defined when
the code is being compiled. This is useful for code segments
surrounded by preprocessor directives that are to be included only if
a particular implementation of a unit is selected.


*-nofbs* non-fixed-block size mode where nxb, nyb and nzb are not
fixed at compile time. 

*-gridinterpolation=<scheme>*  is used to select a scheme for Grid interpolation.

*-makefile=<extension>}* is used when a site can work with more than
one compiler suite. In such situations the site can have several
Makefile.h's named as Makefile.h.<extension> and the one specified
with this option will be selected

*-index-reorder*  instructs setup order the indices in state variables
as specified. This feature is needed because the base data structure
in PARAMESH expects the variable index to be first while AMReX needs
it to be last. This feature permits the same code base to work in both
modes. 

*-parfile=<filename>*  causes setup to copy the specified runtime-parameters file in
the simulation directory to the object directory and name it
flash.par. By default every setup distributed has a flash.par which is
copied into the object directory if this option is not used.

*-append-parfiles=[location1]<filename1>,[location2]<filename2> ...
takes a comma-separated list of names of parameter
files and combines them into one flash.par file in the object directory.
File names without an absolute path are taken to be relative to the simulation
directory, in a way similat to that  for the -parfile option.

*-portable* this option causes setup copy instead of linking source files.

*-site=<site>* specifies the suddirectory of the "sites" directory
from where to fetch Makefile.h

*-with-library=<libname>[,args]*  instructs setup to link in the
specified library when building the final executable. The libraty in
questoin can be internal or external. If external the user must make
sure that the appropriate path to the library is provided in their
site's Makefile.h. An internal library will be built by the setup if
it hasn't already been done so by an earlier invocation.


*-without-library=<libname>* is used to override the need for a
library specified by one the Config files traversed by the setup
tool. 


.. _`Sec:SetupShortcuts`:

Using Shortcuts
---------------

Apart from the various setup options the tool also allows you to use
shortcuts for frequently used combinations of options. For example,
instead of typing in

.. container:: codeseg

   ./setup -a Sod -with-unit=Grid/GridMain/UG

you can just type

.. container:: codeseg

   ./setup -a Sod +ug

Any setup option starting with a ‘+’ is considered as a shortcut.
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
         ============ ===================================================


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
configured. 

.. _`Sec:SetupMakefile`:

Creating a Site-specific 
------------------------
To create a Makefile specific to
your system follow these instructions.

-  Create a directory under sites with hostname of your machine.

-  Find an existing site that is similar to yours.

-  Use bin/suggestMakefile.sh to help identify the locations of various libraries on your
   system. The script scans your system and displays the locations of
   some libraries. You must note the location of MPI library as well. If
   your compiler is actually an mpi-wrapper, you must still define
   LIB_MPI  in your site specific Makefile.h as the empty string.

-  Edit to provide the locations of various libraries on your system.

-  Edit to specify the Fortran and C compilers to be used.

.. container:: flashtip

   The Makefile.h *must* include a compiler flag to promote Fortran to
   double precision
   .
   Flash-X performs all communication of Fortran using type, and assumes
   that Fortran are interoperable with C in the I/O unit.

Files Created During the Process
--------------------------------

The setup tool generates many files in the directory. They fall into
three major categories:

-  Files not required to build the Flash-X executable, but which contain
   useful information,

-  Generated or C code, and

-  Makefiles required to compile the Flash-X executable.

Informational files
~~~~~~~~~~~~~~~~~~~

These files are generated before compilation by . Each of these files
begins with the prefix *setup* for easy identification.

.. container:: tabular

   setup_call & contains the options with which was called and the
   command line resulting after shortcut expansion

   setup_libraries & contains the list of libraries and their arguments (if any) which
   was linked in to generate the executable

   setup_units & contains the list of all units which were included in the current
   setup

   setup_defines & contains a list of all pre-process symbols passed to the compiler
   invocation directly

  setup_flags & contains the exact compiler and linker flags

  setup_parames & contains the list of runtime parameters defined in the files
   processed by

   setup_vars & contains the list of variables, fluxes, species, particle
   properties, and mass scalars used in the current setup, together with
   their descriptions.


