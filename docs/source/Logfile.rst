.. _`Chp:Logfile Unit`:

Logfile Unit
============

Flash-X supplies the Logfile unit to manage an output log during a
Flash-X simulation. The logfile contains various types of useful
information, warnings, and error messages produced by a Flash-X run.
Other units can add information to the logfile through the Logfile unit
interface. The routines enable a program to open and close a log file,
write time or date stamps to the file, and write arbitrary messages to
the file. The file is kept closed and is only opened for appending when
information is to be written, thus avoiding problems with unflushed
buffers. For this reason, routines should not be called within
time-sensitive loops, as system calls are generated. Even when starting
from scratch, the logfile is opened in append mode to avoid deleting
important logfiles. Two kinds of Logfiles are supported. The first kind
is similar to that in Flash-X2 and early releases of , where the master
processor has exclusive access to the logfile and writes global
information to it. The newer kind gives all processors access to their
own private logfiles if they need to have one. Similar to the
traditional logfile, the private logfiles are opened in append mode, and
they are created the first time a processor writes to one. The private
logfiles are extremely useful to gather information about failures
causes by a small fraction of processors; something that cannot be done
in the traditional logfile.

The Logfile unit is included by default in all the provided Flash-X
simulations because it is required by the . As with all the other units
in Flash-X, the data specific to the Logfile unit is stored in the
module . Logfile unit scope data variables begin with the prefix and
they are initialized in the routine .

By default, the logfile is named and found in the output directory. The
user may change the name of the logfile by altering the runtime
parameter in the .

.. container:: codeseg

   # names of files basenm = "cellular_" log_file = "cellular.log"

Meta Data
---------

The stores meta data about a given run including the time and date of
the run, the number of MPI tasks, dimensionality, compiler flags and
other information about the run. The snippet below is an example from a
showing the basic setup and compilation information:

Runtime Parameters, Physical Constants, and Multispecies Data
-------------------------------------------------------------

The also records which units were included in a simulation, the runtime
parameters, physical constants, and any species and their properties
from the Multispecies unit. The logfile keeps track of whether a runtime
parameter is a default value or whether its value has been redefined in
the The symbol will occur next to a runtime parameter if its value has
been redefined in the . Note that the runtime parameters are output in
alphabetical order within the Fortran datatype – so integer parameters
are shown first, then real, then string, then Boolean. The snippet below
shows the this portion of the ; omitted sections are indicated with
“...".

.. container:: codeseg

   ==============================================================================
   Flash-X Units used: Driver Driver/DriverMain
   Driver/DriverMain/TimeDep Grid Grid/GridMain Grid/GridMain/paramesh
   Grid/GridMain/paramesh/paramesh4 ... Multispecies Particles
   PhysicalConstants PhysicalConstants/PhysicalConstantsMain
   RuntimeParameters RuntimeParameters/RuntimeParametersMain ...
   physics/utilities/solvers/LinearAlgebra
   ==============================================================================
   RuntimeParameters:

   ==============================================================================
   algebra = 2 [CHANGED] bndpriorityone = 1 bndprioritythree = 3 ... cfl
   = 0.800E+00 checkpointfileintervaltime = 0.100E-08 [CHANGED] cvisc =
   0.100E+00 derefine_cutoff_1 = 0.200E+00 derefine_cutoff_2 = 0.200E+00
   ... zmax = 0.128E+02 [CHANGED] zmin = 0.000E+00 basenm = cellular\_
   [CHANGED] eosmode = dens_ie eosmodeinit = dens_ie geometry =
   cartesian log_file = cellular.log [CHANGED] output_directory =
   pc_unitsbase = CGS plot_grid_var_1 = none plot_grid_var_10 = none
   plot_grid_var_11 = none plot_grid_var_12 = none plot_grid_var_2 =
   none ... yr_boundary_type = periodic zl_boundary_type = periodic
   zr_boundary_type = periodic bytepack = F chkguardcells = F
   converttoconsvdformeshcalls = F converttoconsvdinmeshinterp = F ...
   useburn = T [CHANGED] useburntable = F

   ==============================================================================

   Known units of measurement:

   Unit CGS Value Base Unit 1 cm 1.0000 cm 2 s 1.0000 s 3 K 1.0000 K 4 g
   1.0000 g 5 esu 1.0000 esu 6 m 100.00 cm 7 km 0.10000E+06 cm 8 pc
   0.30857E+19 cm ... Known physical constants:

   Constant Name Constant Value cm s g K esu 1 Newton 0.66726E-07 3.00
   -2.00 -1.00 0.00 0.00 2 speed of light 0.29979E+11 1.00 -1.00 0.00
   0.00 0.00 ... 15 Euler 0.57722 0.00 0.00 0.00 0.00 0.00
   ==============================================================================

   Multifluid database contents:

   Initially defined values of species: Name Index Total Positive
   Neutral Negative bind Ener Gamma ar36 12 3.60E+01 1.80E+01 -9.99E+02
   -9.99E+02 3.07E+02 -9.99E+02 c12 13 1.20E+01 6.00E+00 -9.99E+02
   -9.99E+02 9.22E+01 -9.99E+02 ca40 14 4.00E+01 2.00E+01 -9.99E+02
   -9.99E+02 3.42E+02 -9.99E+02 ... ti44 24 4.40E+01 2.20E+01 -9.99E+02
   -9.99E+02 3.75E+02 -9.99E+02
   ==============================================================================

Accessor Functions and Timestep Data
------------------------------------

Other units within Flash-X may make calls to write information, or
stamp, the logfile. For example, the Driver unit calls the API routine
after each timestep. The Grid unit calls whenever refinement occurs in
an adaptive grid simulation. If there is an error that is caught in the
code the API routine stamps the before aborting the code. Any unit can
stamp the logfile with one of two routines which includes a data and
time stamp along with a logfile message, or which simply writes a string
to the .

The routine is overloaded so the user must use the interface file in the
calling routine. The next snippit shows logfile output during the
evolution loop of a Flash-X run.

.. container:: codeseg

   ==============================================================================
   [ 04-19-2006 16:40.43 ] [Simulation_init]: initializing Sod problem
   [GRID amr_refine_derefine] initiating refinement [GRID
   amr_refine_derefine] min blks 0 max blks 1 tot blks 1 [GRID
   amr_refine_derefine] min leaf blks 0 max leaf blks 1 tot leaf blks 1
   [GRID amr_refine_derefine] refinement complete [ 04-19-2006 16:40.43
   ] [GRID gr_expandDomain]: create level=2 ... [GRID
   amr_refine_derefine] initiating refinement [GRID amr_refine_derefine]
   min blks 250 max blks 251 tot blks 501 [GRID amr_refine_derefine] min
   leaf blks 188 max leaf blks 188 tot leaf blks 376 [GRID
   amr_refine_derefine] refinement complete [ 04-19-2006 16:40.44 ]
   [GRID gr_expandDomain]: create level=7 [ 04-19-2006 16:40.44 ] [GRID
   gr_expandDomain]: create level=7 [ 04-19-2006 16:40.44 ] [GRID
   gr_expandDomain]: create level=7 [ 04-19-2006 16:40.44 ]
   [IO_writeCheckpoint] open: type=checkpoint name=sod_hdf5_chk_0000 [
   04-19-2006 16:40.44 ] [io_writeData]: wrote 501 blocks [ 04-19-2006
   16:40.44 ] [IO_writeCheckpoint] close: type=checkpoint
   name=sod_hdf5_chk_0000 [ 04-19-2006 16:40.44 ] [IO writePlotfile]
   open: type=plotfile name=sod_hdf5_plt_cnt_0000 [ 04-19-2006 16:40.44
   ] [io_writeData]: wrote 501 blocks [ 04-19-2006 16:40.44 ]
   [IO_writePlotfile] close: type=plotfile name=sod_hdf5_plt_cnt_0000 [
   04-19-2006 16:40.44 ] [Driver_evolveFlash]: Entering evolution loop [
   04-19-2006 16:40.44 ] step: n=1 t=0.000000E+00 dt=1.000000E-10 ... [
   04-19-2006 16:41.06 ] [io_writeData]: wrote 501 blocks [ 04-19-2006
   16:41.06 ] [IO_writePlotfile] close: type=plotfile
   name=sod_hdf5_plt_cnt_0002 [ 04-19-2006 16:41.06 ]
   [Driver_evolveFlash]: Exiting evolution loop
   ==============================================================================

.. _`Sec:LogfilePerformance`:

Performance Data
----------------

Finally, the records performance data for the simulation. The Timers
unit (see ) is responsible for storing, collecting and interpreting the
performance data. The Timers unit calls the API routine to format the
performance data and write it to the logfile. The snippet below shows
the performance data section of a logfile.

.. container:: codeseg

   ==============================================================================
   perf_summary: code performance summary beginning : 04-19-2006
   16:40.43 ending : 04-19-2006 16:41.06 seconds in monitoring period :
   23.188 number of subintervals : 21 number of evolved zones : 16064
   zones per second : 692.758 —————————————————————————— accounting unit
   time sec num calls secs avg time pct ——————————————————————————
   initialization 1.012 1 1.012 4.366 guardcell internal 0.155 17 0.009
   0.669 writeCheckpoint 0.085 1 0.085 0.365 writePlotfile 0.061 1 0.061
   0.264 evolution 22.176 1 22.176 95.633 hydro 18.214 40 0.455 78.549
   guardcell internal 2.603 80 0.033 11.227 sourceTerms 0.000 40 0.000
   0.002 particles 0.000 40 0.000 0.001 Grid_updateRefinement 1.238 20
   0.062 5.340 tree 1.126 10 0.113 4.856 guardcell tree 0.338 10 0.034
   1.459 guardcell internal 0.338 10 0.034 1.458 markRefineDerefine
   0.339 10 0.034 1.460 guardcell internal 0.053 10 0.005 0.230
   amr_refine_derefine 0.003 10 0.000 0.011 updateData 0.002 10 0.000
   0.009 guardcell 0.337 10 0.034 1.453 guardcell internal 0.337 10
   0.034 1.452 eos 0.111 10 0.011 0.481 update particle refinemen 0.000
   10 0.000 0.000 io 2.668 20 0.133 11.507 writeCheckpoint 0.201 2 0.101
   0.868 writePlotfile 0.079 2 0.039 0.340 diagnostics 0.040 20 0.002
   0.173
   ==============================================================================
   [ 04-19-2006 16:41.06 ] LOGFILE_END: Flash-X run complete.

Example Usage
-------------

An example program using the Logfile unit might appear as follows:

.. container:: codeseg

   program testLogfile

   use Logfile_interface, ONLY: Logfile_init, Logfile_stamp,
   Logfile_open, Logfile_close use Driver_interface, ONLY:
   Driver_initParallel use RuntimeParameters_interface, ONLY:
   RuntimeParameters_init use PhysicalConstants_interface, ONLY:
   PhysicalConstants_init

   implicit none

   integer :: i integer :: log_lun integer :: myPE, numProcs logical ::
   restart, localWrite

   call Driver_initParallel(myPE, numProcs) !will initialize MPI call
   RuntimeParameters_init(myPE, restart) ! Logfile_init needs runtime
   parameters call PhysicalConstants_init(myPE) ! PhysicalConstants
   information adds to logfile call Logfile_init(myPE, numProcs) ! will
   end with Logfile_create(myPE, numProcs)

   call Logfile_stamp (myPE, "beginning log file test...",
   "[programtestLogfile]") localWrite=.true. call
   Logfile_open(log_lun,localWrite) !! open the local logfile do i = 1,
   10 write (log_lun,*) ’i = ’, i enddo call Logfile_stamp (myPE,
   "finished logfile test", "[program testLogfile]") call
   Logfile_close(myPE, log_lun)

   end program testLogfile
