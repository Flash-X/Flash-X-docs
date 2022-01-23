.. _`Chp:Quickstart`:

Quick Start
===========

This chapter describes how to get up-and-running quickly with Flash-X
with an example hydrodynamics simulation, the Sedov explosion.

System requirements
-------------------

You should verify that you have the following:

-  A copy of the Flash-X source code distribution. To obtain a copy of
   the source code, please visit https://github.com/Flash-X and request
   access to the Flash-X repository.

-  A Fortran compiler, a C compiler, and C++ compiler, and Python 3 or above.

-  An installed copy of the Message-Passing Interface (MPI) library, and
   the HDF5 library.

-  To use for AMReX, it must be installed. It requires a different version
   each for 1, 2 or 3 dimensional application configurations. (for
   details see https://github.com/AMReX-Codes/amrex). Alternatively,
   you can use PARAMESH which is included in the distribution, and
   does not need a separate build.

-  The GNU make utility


.. _unpack:

-------------------------------------------------

To begin, clone the Flash-X source code distribution.

::This quick start example uses PARAMESH for managing the mesh and does
not use accelerators or other devices on the node, neither does it
exercise any of the advanced tools that enable use of such devices. 
It requires MPI and HDF5 installed, and runs using one CPU per MPI rank.

Once you have the source code and all the necessary libraries installed
configure the Flash-X source tree for the Sedov explosion problem using
the command


::

   ./setup Sedov -auto -site=<your site>

This configures Flash-X for a 2d explosion using the default
hydrodynamic solver, ideal gas equation of state, Grid unit, and I/O format
defined for this problem, linking all necessary files into a new
directory, called "object".  For the purpose of this example, we will use the
default I/O format, serial HDF5. In order to compile a problem on a
given machine Flash-X allows the user to create a file called
Makefile.h which sets
the paths to compilers and libraries specific to a given platform. This
file is located in the directory "sites". The script will attempt to see if
your machine/platform has one already created, and if so, this will be
linked into the object directory. If you do not already have one,
create a directory with your site name under "sites" and copy over a
Makefile.h from one of the existing sites that has the same operating
system. Customize is for your site by editing the paths for various
libraries. 


Type the command to enter the object directory which was created when
you setup the Sedov problem, and then execute . This will compile the
Flash-X code.

::

   cd object
   make

Assuming compilation and linking were successful, you should now find an
executable named "flashx" in the object directory. You may wish to check that this is
the case.

If compilation and linking were not successful, here are a few common
suggestions to diagnose the problem:

-  Make sure the correct compilers are in your path, and that they
   produce a valid executable.

-  The default Sedov problem uses serial HDF5. If you do not have
   HDF5, you can setup Flash-X without I/O by typing +noio in the
   setup command. If you have parallel HDF5 please see the chapter on
   setup options to overwrite the default option.

-  Make sure the paths to the MPI and HDF libraries are correctly set in
   the in the directory.

Flash-X by default expects to find a text file named flash.par in the directory
from which it is run. This file sets the values of various runtime
parameters that determine the behavior of Flash-X. If it is not present,
Flash-X will abort. Note that all of the distributed setups include a
"flash.par" file which is *copied* into the directory at setup time). A command-line option
is available to use a different name for this file, described in the next section.
Here we describe a few parameters that can be included the flash.par
file for the Sedov setup. 

.. container:: shrink

   .. container:: fcodeseg

      # runtime parameters

      basenm = "sedov_"

      lrefine_max = 5

      refine_var_1 = "dens"
      
      refine_var_2 = "pres"

      restart = .false. checkpointFileIntervalTime = 0.01

      nend = 10000 tmax = 0.05

      gamma = 1.4

      xl_boundary_type = "outflow"
      
      xr_boundary_type = "outflow"

      yl_boundary_type = "outflow"
      
      yr_boundary_type = "outflow"

      plot_var_1 = "dens"
      
      plot_var_2 = "temp"

      plot_var_3 = "pres"

 This example first instructs Flash-X to name output files appropriately
(through the parameter basenm). Flash-X is then instructed to use up to five
levels of adaptive mesh refinement (AMR) (through the parameter lrefine_max), with
the actual refinement based on the density and pressure of the
solution (parameters plot_vars ). We will not be starting from a checkpoint
file ( — this is the default, but here it is explicitly set for
clarity). Output files are to be written every 0.01 time units () and
will be created until :math:`t=0.05` or 10000 timesteps have been taken
( and respectively), whichever comes first. The ratio of specific heats
for the gas ( = :math:`\gamma`) is taken to be 1.4, and all four
boundaries of the two-dimensional grid have outflow (zero-gradient or
Neumann) boundary conditions (set via the parameters).

Note the format of the file – each line is of the form = , a comment
(denoted by a hash mark, ), or a blank. String values are enclosed in
double quotes " ". Boolean values are indicated in the Fortran style. Be
sure to insert a carriage return after the last line of text. A full
list of the parameters available for your current setup is contained in
the file located in the directory, which also includes brief comments
for each parameter.

Running Flash-X
---------------

You can run the code using mpiexec as follows, where <N> is the number
of MPI ranks.

::

   mpiexec -np <N> ./flashx


If your system uses a different command for running MPI programs
replace mpiexec with the appropriate command.

You should see a number of lines of output indicating that Flash-X is
initializing the Sedov problem, listing the initial parameters, and
giving the timestep chosen at each step. After the run is finished, you
should find several files in the current directory:

-  sedov.log echoes the runtime parameter settings and indicates the run time, the
   build time, and the build machine. During the run, a line is written
   for each timestep, along with any warning messages. If the run
   terminates normally, a performance summary is written at the end.

-  sedov.dat contains a number of integral quantities as functions of time: total
   mass, total energy, total momentum, This file can be used directly by
   plotting programs such as gnuplot. 

-  sedov_hdf5_chk_000* are the different checkpoint files. These are complete dumps of the
   entire simulation state at intervals of and are suitable for use in
   restarting the simulation.

-  sedov_hdf5_plt_cnt_000* are plot files. In this example, these files contain density,
   temperature, and pressure in single precision. If needed, more
   variables can be dumped in the plotfiles by specifying them in
   *flash.par*. They are usually written more frequently than checkpoint
   files, since they are the primary output of Flash-X for analyzing the
   results of the simulation. They are also used for making simulation
   movies. Checkpoint files can also be used for analysis and sometimes
   it is necessary to use them since they have comprehensive information
   about the state of the simulation at a given time. However, in
   general, plotfiles are preferred since they have more frequent
   snapshots of the time evolution. Please see for more information
   about IO outputs.

Following sections cover architecture, algorithms,  
sample problems, and the tools distributed with Flash-X.
