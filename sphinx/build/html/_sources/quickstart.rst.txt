.. include:: defs.h

.. _`Chp:Quickstart`:

Quick Start
===========

This chapter describes how to get up-and-running quickly with |flashx|
with an example simulation, the Sedov explosion. We explain how to
configure a problem, build it, run it, and examine the output using IDL.

System requirements
-------------------

You should verify that you have the following:

-  A copy of the |flashx| source code distribution. To obtain a copy of
   the source code, please visit https://github.com/Flash-X and request
   access to the |flashx| repository as a collaborator.

-  A FORTRAN compiler, a C compiler, and a C++ compiler, and Python 3 or above.

-  An installed copy of the Message-Passing Interface (MPI) library, and
   the HDF5 library.

-  To use |amrex|  for AMR, it must be installed. |amrex|  requires
   a different version each for 1, 2 or 3 dimensional application
   configurations. (for details see .....). |paramesh| is included in the
   source code.

-  |amrex| is compatible with PETSc and Hypre math libraries, and
   |paramesh| is compatible with Hypre. To use either of these
   libraries they must be installed on the platform and their
   location should be made known to |amrex| . 

-  The GNU make utility

-  A copy of the Python language, version 3.0 or later is required.

.. _unpack:

Obtaining and configuring |flashx| for quick start
-------------------------------------------------

To begin, clone the |flashx| source code distribution. 

The code of the PARAMESH library for Flash-X is provided as a git 
submodule by a separate code repository named PARAMESH-for-Flash-X. 
After successful initialization of the submodule the code will 
appear as a subtree under Grid/GridMain/AMR in the PM4_package 
directory. The Flash-X setup and build mechanisms will take care 
of compiling this code (if needed for a simulation configuration); 
PARAMESH code is not organized or built as a separate library.

## Git with Submodules

To prepare for building simulations that use libraries whose code must be retrieved
from a separate git repository, the following modified `git` commands can be used:

- `git pull --recurse-submodules=yes` (in place of the usual `git pull`)
- `git submodule update --init` (additionally, after `git pull`)
- `git submodule update --init source/Grid/GridMain/AMR/Paramesh4/PM4_package`
  (variant of the previous item, in case you want to only get the Paramesh package submodule)

This quick start example uses |paramesh| for AMR and does not depend
on PETSc or Hypre. It also does not use accelerators or other devices on
the node, neither does it exercise advanced features of the
Orchestration System. It requires MPI and HDF5 installed, and runs using
one CPU per MPI rank.

Once you have the source code and all the necessary libraries installed
configure the |flashx| source tree for the Sedov explosion problem using
the *setup* command Type

   *./setup Sedov -auto*

This configures |flashx| for the 2d *Sedov* problem using the default
hydrodynamic solver, equation of state, Grid unit, and I/O format
defined for this problem, linking all necessary files into a new
directory, called *object*. For the purpose of this example, we
will use the default I/O format, serial HDF5. In order to compile a
problem on a given machine |flashx| allows the user to create a file
called *Makefile.h* which sets the paths to compilers and libraries
specific to a given platform. This file is located in the directory
*sites/mymachine.myinstitution.mydomain/*. The setup script will
attempt to see if your machine/platform has a *Makefile.h already
created, and if so, this will be linked into the object directory.
If one is not created the setup script will use a prototype Makefile.h
with guesses as to the locations of libraries on your machine. The
current distribution includes prototypes for *Linux*, *OSX* operating
systems. In any case, it is advisable to create a Makefile.h
specific to your machine. See for details.

Type the command *cd object* to enter the object directory which was
created when you setup the Sedov problem, and then execute *make*.
This will compile the |flashx| code.

::

   cd object
   make

If you have problems and need to recompile, *make clean* will remove
all object files from the object directory, leaving the source
configuration intact; *make realclean* will remove all files and
links from object. After ‘make realclean’, a new invocation of
setup is required before the code can be built. The building can
take a long time on some machines; doing a parallel build (make -j
for example) can significantly increase compilation speed, even on
single processor systems.

Assuming compilation and linking were successful, you should now find an
executable named *flashx* in the object directory. You may wish
to check that this is the case.

If compilation and linking were not successful, here are a few common
suggestions to diagnose the problem:

-  Make sure the correct compilers are in your path, and that they
   produce a valid executable.

-  The default Sedov problem uses HDF5 in serial. Make sure you have
   HDF5 installed. If you do not have HDF5, you can still setup and
   compile |flashx|, but you will not be able to generate either a
   checkpoint or a plot file. You can setup |flashx| without I/O by
   typing

      *./setup Sedov -auto +noio*

-  Make sure the paths to the MPI and HDF libraries are correctly set in
   the Makefile.h in the object directory.

-  Make sure your version of MPI creates a valid executable that can run
   in parallel.

|flashx| by default expects to find a text file named *flash.par* in
the directory from which it is run. This file sets the values of various
runtime parameters that determine the behavior of |flashx|. If it is not
present, |flashx| will abort; flash.par must be created in order for
the program to run (note: all of the distributed setups already come
with a flash.par which is *copied* into the object directory at
setup time). There is command-line option to use a different name for
this file, described in the next section. Here we will create a simple
flash.par that sets a few parameters and allows the rest to take on
default values. With your text editor, edit the flash.par in the
object directory so it looks like .

.. container:: shrink

   .. container:: fcodeseg

      # runtime parameters

      basenm = "sedov_"

      lrefine_max = 5 refine_var_1 = "dens" refine_var_2 = "pres"

      restart = .false. checkpointFileIntervalTime = 0.01

      nend = 10000 tmax = 0.05

      gamma = 1.4

      xl_boundary_type = "outflow" xr_boundary_type = "outflow"

      yl_boundary_type = "outflow" yr_boundary_type = "outflow"

      plot_var_1 = "dens" plot_var_2 = "temp" plot_var_3 = "pres"

      sim_profFileName = "/dev/null"

This example first instructs |flashx| to name output files appropriately
(through the *basenm* parameter). |flashx| is then instructed to use up
to five levels of adaptive mesh refinement (AMR) (through the
*lrefine_max* parameter), with the actual refinement adapted based on
the density and pressure of the solution (parameters *refine_var_1*
and *refine_var_2*). We will not be starting from a checkpoint file
(“*restart = .false.*” — this is the default, but here it is
explicitly set for clarity). Output files are to be written every 0.01
time units (*checkpointFileIntervalTime*) and will be created until
:math:`t=0.05` or 10000 timesteps have been taken (*tmax* and *nend*
respectively), whichever comes first. The ratio of specific heats for
the gas (*gamma* = :math:`\gamma`) is taken to be 1.4, and all four
boundaries of the two-dimensional grid have outflow (zero-gradient or
Neumann) boundary conditions (set via the [xyz][lr]_boundary_type parameters).

Note the format of the file – each line is of the form *variable* =
*value*, a comment (denoted by a hash mark, #), or a blank. String
values are enclosed in double quotes ("). Boolean values are
indicated in the FORTRAN style, .true. or .false.. Be sure to
insert a carriage return after the last line of text. A full list of the
parameters available for your current setup is contained in the file
*setup_params* located in the object directory, which also
includes brief comments for each parameter. If you wish to skip the
creation of a flash.par, a complete example is provided in the
*source/Simulation/SimulationMain/Sedov/* directory.

Running |flashx|
---------------

We are now ready to run |flashx|. To run |flashx| on *N* processors, type

   *mpirun -np N ./flashx*

remembering to replace *N* with the appropriate values. Some
systems may require you to start MPI programs with a different command;
use whichever command is appropriate for your system. The |flashx|
executable accepts an optional command-line argument for the runtime
parameters file. If “*-par_file* *filename*” is present, |flashx| reads
the file specified on command line for runtime parameters, otherwise it
reads flash.par.

You should see a number of lines of output indicating that |flashx| is
initializing the Sedov problem, listing the initial parameters, and
giving the timestep chosen at each step. After the run is finished, you
should find several files in the current directory:

-  *sedov.log* echoes the runtime parameter settings and indicates the
   run time, the build time, and the build machine. During the run, a
   line is written for each timestep, along with any warning messages.
   If the run terminates normally, a performance summary is written to
   this file.

-  *sedov.dat* contains a number of integral quantities as functions
   of time: total mass, total energy, total momentum, *etc.* This file
   can be used directly by plotting programs such as *gnuplot*; note
   that the first line begins with a hash (#) and is thus ignored by
   gnuplot.

-  *sedov_hdf5_chk_000** are the different checkpoint files. These are
   complete dumps of the entire simulation state at intervals of
   *checkpointFileIntervalTime* and are suitable for use in restarting
   the simulation.

-  *sedov_hdf5_plt_cnt_000** are plot files. In this example, these
   files contain density, temperature, and pressure in single precision.
   If needed, more variables can be dumped in the plotfiles by
   specifying them in *flash.par*. They are usually written more
   frequently than checkpoint files, since they are the primary output
   of |flashx| for analyzing the results of the simulation. They are also
   used for making simulation movies. Checkpoint files can also be used
   for analysis and sometimes it is necessary to use them since they
   have comprehensive information about the state of the simulation at a
   given time. However, in general, plotfiles are preferred since they
   have more frequent snapshots of the time evolution. Please see for
   more information about IO outputs.

|flashx| is intended to be customized by the user to work with
interesting initial and boundary conditions. In the following sections,
we will cover in more detail the algorithms and structure of |flashx| and
the sample problems and tools distributed with it.
