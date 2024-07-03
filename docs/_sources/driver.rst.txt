.. include:: defs.h

.. _`Chp:Driver Unit`:

Driver Unit
===========

The *Driver* unit controls the initialization and finalization of |flashx|
simulations. Initialization can be from
scratch or from a stored checkpoint file. It also call the methods
for advancing the solution, and calls the *IO* unit at the end of
every timestep to produce checkpoint files, plot files, or other output.

.. _Driver Routines:

Driver Routines
---------------

The most important routines in the *Driver* API are those that
initialize, evolve, and finalize the |flashx| program. The file
*main.F90* contains the main |flashx| program (equivalent to
*main()* in C). The default top-level program of |flashx|,
*Simulation/main.F90*, calls *Driver* routines in this order:

.. container:: codeseg

   program Flashx

   implicit none

   call Driver_initParallel()

   call Driver_initAll()

   call Driver_finalizeAll( )

   end program Flashx

Therefore the no-operation stubs for these routines in the Driver
source directory must be overridden by an implementation function in a
unit implementation directory under the Driver or *Simulation*
directory trees, in order for a simulation to perform any meaningful
actions. The most commonly used implementations for these files
are located in the *Driver/DriverMain* unit implementation directory.

*Driver_initAll*
~~~~~~~~~~~~~~~~~~~~

The first of these routines is *Driver/Driver_initParallel*, which
initializes the parallel environment for the simulation. New in |flashx|
is an ability to replicate the mesh where more than one copy of the
discretized mesh may exist with some overlapping and some
non-overlapping variables. Because of this feature, the parallel
environment differentiates between global and mesh communicators. All
the necessary communicators, and the attendant meta-data is generated in
this routine. Also because of this modification, runtime parameters such
as iProcs, jProcs etc, which were under the control of the Grid unit in
|flashx|, are now under the control of the Driver unit. Several new
accessor interface allow other code units to query the driver unit for
this information. The *Driver/Driver_initAll*, the next routine, in
general calls the initialization routines in each of the units. If a
unit is not included in a simulation, its stub (or empty) implementation
is called. Having stub implementations is very useful in the Driver
unit because it allows the user to avoid writing a new driver for each
simulation. For a more detailed explanation of stub implementations
please see . It is important to note that when individual units are
being initialized, order is often very important and the order of
initialization is different depending on whether the run is from scratch
or being restarted from a checkpoint file.


Runtime Parameters
^^^^^^^^^^^^^^^^^^

The Driver unit supplies certain runtime parameters regardless of
which type of driver is chosen. These are described in the online
\ Driver/Runtime Parameters Documentation page.

Driver_finalizeAll
~~~~~~~~~~~~~~~~~~~~~~~~

Finally, the the Driver unit calls *Driver/Driver_finalizeAll*
which calls the finalize routines for each unit. Typically this involves
deallocating memory and any other necessary cleanup.

Driver accessor functions
~~~~~~~~~~~~~~~~~~~~~~~~~

Driver unit also provides a number of accessor
functions to get data stored in the Driver unit, for example
*Driver/Driver_getDt*, *Driver/Driver_getNStep*,
*Driver/Driver_getElapsedWCTime*, *Driver/Driver_getSimTime*.

The Driver unit API also defines two interfaces for halting the
code, *Driver/Driver_abort* and
*Driver/Driver_abortC* .c. The ’c ’ routine version
is available for calls written in C, so that the user does not have to
worry about any name mangling. Both of these routines print an error
message and call *MPI_Abort*.

Time Step Limiting
~~~~~~~~~~~~~~~~~~

The Driver unit is responsible for determining the time step
:math:`\Delta t` that is used to advance the solution from time
:math:`t=t^{n-1}` to :math:`t^n`. At startup, a tentative
:math:`\Delta t` is chosen based in runtime parameters, in particular
*Driver/dtinit*. The routine *Driver/Driver_verifyInitDt* (usually
invoked from Driver/Driver_initAll) is used to check and, if
necessary, modify the initial :math:`\Delta t`. Subsequently, the
routine *Driver/Driver_computeDt* (usually invoked from
Driver/Driver_evolveAll *at the end* of an iteration of the main
evolution loop) is used to recompute :math:`\Delta t` for the next
evolution step.

The implementation of Driver/Driver_computeDt can lower or increase
the time step. It takes various runtime parameters into consideration,
including *Driver/dtmin*, *Driver/dtmax*, and
*Driver/tstep_change_factor*. For the most part, however,
Driver/Driver_computeDt calls on *UNIT_computeDt* routines of
various code units to query those units for their time step
requirements, and usually chooses the smallest time step that is
acceptable to all units queried.

The code units that participate in this negotiation return a
:math:`\Delta t`, and usually some additional information about which
location in the simulaton domain caused the reequired step to be as low
as returned. A unit’s time step requirement can depend on the current
state of the solution as well as on further runtime parameters. For
example, the dt returned by *physics/Hydro/Hydro_computeDt*
depends on the state of density, pressure, and velocities (and possibly
additional variables) in the cells of the domain, as well as on the
runtime parameter *Hydro/cfl*.

.. _`Sec:dr_posdef`:

Generic time step limiting for positive definiteness
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A generic method for limiting time steps, based on a requirement that
certain (user-specified) variables should never become negative, has
been added in |flashx|. To understand the basic idea, think of each
variable that this limiter is applied to as a density of some quantity
:math:`q`. (Variables of PER_MASS type are actually converted to
their PER_VOLUME counterparts in the implementation of the method.)

The algorithm is based on examining the distribution of :math:`q` in
neighboring cells, and making for each cell a near-worst-case estimate
of the rate of depletion :math:`\dot q` based on projected fluxes of
:math:`q` out of the cell over its faces. This can be applied to any
variable, it is not required that the variable represents any quantity
that is actually advected as described. See for how to use. This feature
may be particularly useful when applied to "eion" in
multidimensional 3T simulations in order to avoid some kinds of
“negative ion temperature” failures, as an alternative to lowering the
*Hydro* runtime parameter *Hydro/cfl*.

.. container:: center

   .. container::
      :name: Tab:dr_posdef parameters

      .. table:: Runtime parameters for positive definiteness.

         +---------------------+---------+------------+---------------------+
         | Parameter           | Type    | Default    | Description         |
         +=====================+=========+============+=====================+
         | Driver/dr_u       | boolean | .false.    | Set to .true. to    |
         | sePosdefComputeDt |         |            | turn                |
         |                     |         |            | positive-definite   |
         |                     |         |            | time step limiter   |
         |                     |         |            | on.                 |
         +---------------------+---------+------------+---------------------+
         |                     |         |            |                     |
         +---------------------+---------+------------+---------------------+
         | Driver            | integer | 4          | Number of variables |
         | /dr_numPosdefVars |         |            | for which positive  |
         |                     |         |            | definiteness should |
         |                     |         |            | be enforced         |
         +---------------------+---------+------------+---------------------+
         |                     |         |            |                     |
         +---------------------+---------+------------+---------------------+
         | dr_posdefVar_N  | string  | "none" | Name of Nth         |
         |                     |         |            | variable for        |
         |                     |         |            | positive-definite   |
         |                     |         |            | dt limiter          |
         +---------------------+---------+------------+---------------------+
         |                     |         |            |                     |
         +---------------------+---------+------------+---------------------+
         |                   | real    | 1.0        | Scaling factor for  |
         | dr_posdefDtFactor |         |            | dt limit from       |
         |                     |         |            | positive-definite   |
         |                     |         |            | time step limiter.  |
         |                     |         |            | Similar to CFL      |
         |                     |         |            | factor. If set to   |
         |                     |         |            | -1, use             |
         |                     |         |            | Hydro/CFL       |
         |                     |         |            | factor from         |
         |                     |         |            | Hydro.          |
         +---------------------+---------+------------+---------------------+
