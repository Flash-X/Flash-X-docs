.. include:: defs.h

.. _`Chp:Driver Unit`:

Driver Unit
===========

The ``Driver`` unit controls the initialization and evolution of |flashx|
simulations. In addition, at the highest level, the ``Driver`` unit
organizes the interaction between units. Initialization can be from
scratch or from a stored checkpoint file. For advancing the solution,
the drivers can use either an operator-splitting technique adapted to
directionally split physics operators like split ``Hydro`` (``Split``),
or a more generic “``Unsplit``” implementation. The ``Driver`` unit also
calls the ``IO`` unit at the end of every timestep to produce checkpoint
files, plot files, or other output.

.. _Driver Routines:

Driver Routines
---------------

The most important routines in the ``Driver`` API are those that
initialize, evolve, and finalize the |flashx| program. The file
``Flash.F90`` contains the main |flashx| program (equivalent to
``main()`` in C). The default top-level program of |flashx|,
``Simulation/Flash.F90``, calls ``Driver`` routines in this order:

.. container:: codeseg

   program Flash

   implicit none

   call Driver_initParallel() call Driver_initFlash() call
   Driver_evolveFlash( ) call Driver_finalizeFlash ( )

   end program Flash

Therefore the no-operation stubs for these routines in the ``Driver``
source directory must be overridden by an implementation function in a
unit implementation directory under the ``Driver`` or ``Simulation``
directory trees, in order for a simulation to perform any meaningful
actions. The most commonly used implementations for most of these files
are located in the ``Driver/DriverMain`` unit implementation directory,
with a few specialized ones in either ``Driver/DriverMain/Split`` or
``Driver/DriverMain/Unsplit``.

``Driver_initFlash``
~~~~~~~~~~~~~~~~~~~~

The first of these routines is ``Driver/Driver_initParallel``, which
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
this information. The ``Driver/Driver_initFlash``, the next routine, in
general calls the initialization routines in each of the units. If a
unit is not included in a simulation, its stub (or empty) implementation
is called. Having stub implementations is very useful in the ``Driver``
unit because it allows the user to avoid writing a new driver for each
simulation. For a more detailed explanation of stub implementations
please see . It is important to note that when individual units are
being initialized, order is often very important and the order of
initialization is different depending on whether the run is from scratch
or being restarted from a checkpoint file.

``Driver_evolveFlash``
~~~~~~~~~~~~~~~~~~~~~~

The next routine is ``Driver/Driver_evolveFlash`` which controls the
timestepping of the simulation, as well as the normal termination of
|flashx| based on time. ``Driver_evolveFlash`` checks the parameters
``tmax``, ``nend`` and ``zFinal`` to determine that the run should end,
having reached a particular point in time, a certain number of steps, or
a particular cosmological redshift, respectively. Likewise the initial
simulation time, step number and cosmological redshift for a simulation
can be set using the runtime parameters ``tmin``, ``nbegin``, and
``zInitial``. This version of |flashx| includes versions of
``Driver_evolveFlash`` for directionally-split and unsplit staggered
mesh operators.

Strang Split Evolution
^^^^^^^^^^^^^^^^^^^^^^

The code in the ``Driver/DriverMain/Split`` unit implementation
directory has been the default time update method earlier, and can still
be be used for many setups that can be configured with |flashx|. The
routine ``Driver_evolveFlash`` implements a Strang-split method of time
advancement where each physics unit updates the solution data for two
equal timesteps – thus the sequence of calls to physics and other units
in each time step goes something like this: ``Hydro``, diffusive terms,
source terms, ``Particles``, ``Gravity``; ``Hydro``, diffusive terms,
source terms, ``Particles``, ``Gravity``, ``IO`` (for output), ``Grid``
(for grid changes). The hydrodynamics update routines take a “sweep
order” argument since they must be directionally split to work with this
driver. Here, the first call usually uses the ordering :math:`x-y-z`,
and the second call uses :math:`z-y-x`. Each of the update routines is
assumed to directly modify the solution variables. At the end of one
loop of timestep advancement, the condition for updating the mesh
refinement pattern is tested if the adaptive mesh is being used, and a
refinement update is carried out if required.

Unsplit Evolution
^^^^^^^^^^^^^^^^^

The driver implementation in the ``Driver/DriverMain/Unsplit`` directory
is the default since |flashx|. It is required specifically for the two
unsplit solvers: unsplit staggered mesh MHD solver () and the unsplit
gas hydrodynamics solver (). This implementation in general calls each
of the physics routines only once per time step, and each call advances
solution vectors by one timestep. At the end of one loop of timestep
advancement, the condition for updating the adaptive mesh refinement
pattern is tested and applied.

Super-Time-Stepping (STS)
^^^^^^^^^^^^^^^^^^^^^^^^^

A new timestepping method implemented in |flashx| is a technique called
Super-Time-Stepping (STS). The STS is a simple explicit method which is
used to accelerate restrictive parabolic timestepping advancements
(:math:`\Delta t_{\mbox{CFL\_para}}\approx \Delta x^2`) by relaxing the
CFL stability condition of parabolic equation system.

The STS has been proposed by Alexiades et al., (1996), and used in
computational astrophysics and sciences recenly (see Mignone et al.,
2007; O’Sullivan & Downes, 2006; Commerçon et al., 2011; Lee, D. et al,
2011) for solving systems of parabolic PDEs numerically. The method
increases its effective time steps :math:`\Delta t_{\mbox{sts}}` using
two properties of stability and optimality in Chebychev polynomial of
degree :math:`n`. These properties optimally maximize the time step
:math:`\Delta t_{\mbox{sts}}` by which a solution vector can be evolved.
A stability condition is imposed only after each time step
:math:`\Delta t_{\mbox{sts}}`, which is further subdivided into smaller
:math:`N_{\mbox{sts}}` sub-time steps, :math:`\tau_i`, that is,
:math:`\Delta t_{\mbox{sts}}=\sum^{N_{\mbox{sts}}}_{i=1} \tau_i`, where
the sub-time step is given by

.. math::

   \tau_i=\Delta t_{\mbox{CFL\_para}}\big[ (-1+\nu_{\mbox{sts}}) cos\big(\frac{\pi(2j-1)}{2N_{\mbox{sts}}}  + 1+\nu_{\mbox{sts}}  \big)\big]^{-1},
   \label{eqn:STS_original}

where :math:`\Delta t_{\mbox{CFL\_para}}` is an explicit time step for a
given parabolic system based on the CFL stability condition. :math:`\nu`
(``nuSTS``) is a free parameter less than unity. For
:math:`\nu \rightarrow 0`, STS is asymptotically :math:`N_{\mbox{sts}}`
times faster than the conventional explicit scheme based on the CFL
condition. During the :math:`N_{sts}` sub-time steps, the STS method
still solves solutions at each intermediate step :math:`\tau_i`;
however, such solutions should not be considered as meaningful
solutions.

Extended from the original STS method for accelerating parabolic
timestepping, our STS method advances advection and/or diffusion
(hyperbolic and/or parabolic) system of equations. This means that the
STS algorithm in |flashx| invokes a single :math:`\Delta t_{sts}` for
both advection and diffusion, and does not use any sub-cycling for
diffusion based on a given advection time step. In this case,
:math:`\tau_i` is given by

.. math::

   \tau_i=\Delta t_{\mbox{CFL}}\big[ (-1+\nu_{\mbox{sts}}) cos\big(\frac{\pi(2j-1)}{2N_{\mbox{sts}}}  + 1+\nu_{\mbox{sts}} \big)\big]^{-1},
   \label{eqn:STS_flash}

where :math:`\Delta t_{\mbox{CFL}}` can be an explicit time step for
advection (:math:`\Delta t_{\mbox{CFL\_adv}}`) or parabolic
(:math:`\Delta t_{\mbox{CFL\_para}}`) systems. In cases of
advection-diffusion system, :math:`\Delta t_{\mbox{CFL}}` takes
(:math:`\Delta t_{\mbox{CFL\_para}}`) when it is smaller than
(:math:`\Delta t_{\mbox{CFL\_adv}}`); otherwise, |flashx|’s timestepping
will proceed without using STS iterations (i.e., using standard explicit
timestepping that is either Strang split evolution or unsplit evolution.

Since the method is explicit, it works equally well on both a uniform
grid and AMR grids without modification. The STS method is first-order
accurate in time.

Both directionally-split and unsplit hydro solvers can use the STS
method, simply by invoking a runtime parameter ``useSTS = .true.`` in
``flash.par``. There are couple of runtime parameters that control
solution accuracy and stability. They are decribed in Table
`1.1 <#Tab:sts parameters>`__.

.. container:: center

   .. container::
      :name: Tab:sts parameters

      .. table::  Runtime parameters for STS

         +-----------------------+---------+---------+-----------------------+
         | Variable              | Type    | Default | Description           |
         +=======================+=========+=========+=======================+
         | ``useSTS``            | logical | .false. | Enable STS            |
         +-----------------------+---------+---------+-----------------------+
         | ``nstepTotalSTS``     | integer | 5       | Suggestion:           |
         |                       |         |         | :math:`\sim` 5 for    |
         |                       |         |         | hyperbolic;           |
         |                       |         |         | :math:`\sim` 10 for   |
         |                       |         |         | parabolic             |
         +-----------------------+---------+---------+-----------------------+
         | ``nuSTS``             | real    | 0.2     | Suggestion:           |
         |                       |         |         | :math:`\sim` 0.2 for  |
         |                       |         |         | hyperbolic;           |
         |                       |         |         | :math:`\sim` 0.01 for |
         |                       |         |         | parabolic. It is      |
         |                       |         |         | known that a very low |
         |                       |         |         | value of :math:`\nu`  |
         |                       |         |         | may result in         |
         |                       |         |         | unstable temporal     |
         |                       |         |         | integrations, while a |
         |                       |         |         | value close to unity  |
         |                       |         |         | can decrease the      |
         |                       |         |         | expected efficiency   |
         |                       |         |         | of STS.               |
         +-----------------------+---------+---------+-----------------------+
         | `                     | logical | .false. | This setup will use   |
         | `useSTSforDiffusion`` |         |         | the STS for           |
         |                       |         |         | overcoming small      |
         |                       |         |         | diffusion time steps  |
         |                       |         |         | assuming              |
         |                       |         |         | :m                    |
         |                       |         |         | ath:`\Delta t_{\mbox{ |
         |                       |         |         | CFL\_adv}} > \Delta t |
         |                       |         |         | _{\mbox{CFL\_para}}`. |
         |                       |         |         | In implementation, it |
         |                       |         |         | will set              |
         |                       |         |         | :math:`\Delta t       |
         |                       |         |         | _{\mbox{CFL}}=\Delta  |
         |                       |         |         | t_{\mbox{CFL\_para}}` |
         |                       |         |         | in Eqn.               |
         |                       |         |         | `[eqn:STS_flash]      |
         |                       |         |         |  <#eqn:STS_flash>`__. |
         |                       |         |         | Do not allow to turn  |
         |                       |         |         | on this switch when   |
         |                       |         |         | there is no diffusion |
         |                       |         |         | (viscosity,           |
         |                       |         |         | conductivity, and     |
         |                       |         |         | magnetic resistivity) |
         |                       |         |         | used.                 |
         +-----------------------+---------+---------+-----------------------+
         | `                     | logical | .false. | If true, this will    |
         | `allowDtSTSDominate`` |         |         | allow to have         |
         |                       |         |         | :m                    |
         |                       |         |         | ath:`\tau_i > \Delta  |
         |                       |         |         | t_{\mbox{CFL\_adv}}`, |
         |                       |         |         | which may result in   |
         |                       |         |         | unstable              |
         |                       |         |         | integrations.         |
         +-----------------------+---------+---------+-----------------------+

Runtime Parameters
^^^^^^^^^^^^^^^^^^

The ``Driver`` unit supplies certain runtime parameters regardless of
which type of driver is chosen. These are described in the online
````\ Driver/Runtime Parameters Documentation page.

``Driver_finalizeFlash``
~~~~~~~~~~~~~~~~~~~~~~~~

Finally, the the ``Driver`` unit calls ``Driver/Driver_finalizeFlash``
which calls the finalize routines for each unit. Typically this involves
deallocating memory and any other necessary cleanup.

Driver accessor functions
~~~~~~~~~~~~~~~~~~~~~~~~~

In |flashx| the ``Driver`` unit also provides a number of accessor
functions to get data stored in the ``Driver`` unit, for example
``Driver/Driver_getDt``, ``Driver/Driver_getNStep``,
``Driver/Driver_getElapsedWCTime``, ``Driver/Driver_getSimTime``.

The ``Driver`` unit API also defines two interfaces for halting the
code, ``Driver/Driver_abortFlash`` and
``Driver/Driver_abortFlashC``\ ``.c``. The ’\ ``c``\ ’ routine version
is available for calls written in C, so that the user does not have to
worry about any name mangling. Both of these routines print an error
message and call ``MPI_Abort``.

Time Step Limiting
~~~~~~~~~~~~~~~~~~

The ``Driver`` unit is responsible for determining the time step
:math:`\Delta t` that is used to advance the solution from time
:math:`t=t^{n-1}` to :math:`t^n`. At startup, a tentative
:math:`\Delta t` is chosen based in runtime parameters, in particular
``Driver/dtinit``. The routine ``Driver/Driver_verifyInitDt`` (usually
invoked from ``Driver/Driver_initFlash``) is used to check and, if
necessary, modify the initial :math:`\Delta t`. Subsequently, the
routine ``Driver/Driver_computeDt`` (usually invoked from
``Driver/Driver_evolveFlash`` *at the end* of an iteration of the main
evolution loop) is used to recompute :math:`\Delta t` for the next
evolution step.

The implementation of ``Driver/Driver_computeDt`` can lower or increase
the time step. It takes various runtime parameters into consideration,
including ``Driver/dtmin``, ``Driver/dtmax``, and
``Driver/tstep_change_factor``. For the most part, however,
``Driver/Driver_computeDt`` calls on ``UNIT_computeDt`` routines of
various code units to query those units for their time step
requirements, and usually chooses the smallest time step that is
acceptable to all units queried.

The code units that participate in this negotiation return a
:math:`\Delta t`, and usually some additional information about which
location in the simulaton domain caused the reequired step to be as low
as returned. A unit’s time step requirement can depend on the current
state of the solution as well as on further runtime parameters. For
example, the ``dt`` returned by ``physics/Hydro/Hydro_computeDt``
depends on the state of density, pressure, and velocities (and possibly
additional variables) in the cells of the domain, as well as on the
runtime parameter ``Hydro/cfl``.

.. _`Sec:dr_posdef`:

Generic time step limiting for positive definiteness
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A generic method for limiting time steps, based on a requirement that
certain (user-specified) variables should never become negative, has
been added in |flashx|. To understand the basic idea, think of each
variable that this limiter is applied to as a density of some quantity
:math:`q`. (Variables of ``PER_MASS`` type are actually converted to
their ``PER_VOLUME`` counterparts in the implementation of the method.)

The algorithm is based on examining the distribution of :math:`q` in
neighboring cells, and making for each cell a near-worst-case estimate
of the rate of depletion :math:`\dot q` based on projected fluxes of
:math:`q` out of the cell over its faces. This can be applied to any
variable, it is not required that the variable represents any quantity
that is actually advected as described. See for how to use. This feature
may be particularly useful when applied to ``"eion"`` in
multidimensional 3T simulations in order to avoid some kinds of
“negative ion temperature” failures, as an alternative to lowering the
``Hydro`` runtime parameter ``Hydro/cfl``.

.. container:: center

   .. container::
      :name: Tab:dr_posdef parameters

      .. table:: Runtime parameters for positive definiteness.

         +---------------------+---------+------------+---------------------+
         | Parameter           | Type    | Default    | Description         |
         +=====================+=========+============+=====================+
         | ``Driver/dr_u       | boolean | .false.    | Set to .true. to    |
         | sePosdefComputeDt`` |         |            | turn                |
         |                     |         |            | positive-definite   |
         |                     |         |            | time step limiter   |
         |                     |         |            | on.                 |
         +---------------------+---------+------------+---------------------+
         |                     |         |            |                     |
         +---------------------+---------+------------+---------------------+
         | ``Driver            | integer | 4          | Number of variables |
         | /dr_numPosdefVars`` |         |            | for which positive  |
         |                     |         |            | definiteness should |
         |                     |         |            | be enforced         |
         +---------------------+---------+------------+---------------------+
         |                     |         |            |                     |
         +---------------------+---------+------------+---------------------+
         | ``dr_posdefVar_N``  | string  | ``"none"`` | Name of Nth         |
         |                     |         |            | variable for        |
         |                     |         |            | positive-definite   |
         |                     |         |            | dt limiter          |
         +---------------------+---------+------------+---------------------+
         |                     |         |            |                     |
         +---------------------+---------+------------+---------------------+
         | ``                  | real    | 1.0        | Scaling factor for  |
         | dr_posdefDtFactor`` |         |            | dt limit from       |
         |                     |         |            | positive-definite   |
         |                     |         |            | time step limiter.  |
         |                     |         |            | Similar to CFL      |
         |                     |         |            | factor. If set to   |
         |                     |         |            | -1, use             |
         |                     |         |            | ``Hydro/CFL``       |
         |                     |         |            | factor from         |
         |                     |         |            | ``Hydro``.          |
         +---------------------+---------+------------+---------------------+
