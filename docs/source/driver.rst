.. _`Chp:Driver Unit`:

Driver Unit
===========

The Driver unit controls the initialization and evolution of Flash-X
simulations. In addition, at the highest level, the Driver unit
organizes the interaction between units. Initialization can be from
scratch or from a stored checkpoint file. For advancing the solution,
the drivers in the current distribution operator-split time integration by
default. Alternative methods of timestepping are under development.
The Driver also calls the IO unit at the end of every timestep to
produce checkpoint files, plot files, or other output.

.. _Driver Routines:

Driver Routines
---------------

The most important routines in the Driver API are those that initialize,
evolve, and finalize the Flash-X program. The file contains the main
Flash-X program (equivalent to main in C). The default top-level program of
Flash-X,  calls Driver routines in this order:

.. container:: codeseg

   program Flash

   implicit none

   call Driver_initParallel()
   call Driver_initFlash()
   call Driver_evolveFlash( )
   call Driver_finalizeFlash ( )

   end program Flash

Therefore the no-operation stubs for these routines in the source
directory must be overridden by an implementation function in a under
the Driver or directory trees, in order for a simulation to perform any
meaningful actions. 

The first of these routines is Driver_initParallel, which initializes the parallel
environment for the simulation. It is possible to replicate the
mesh where more than one copy of the discretized mesh may exist with
some overlapping and some non-overlapping variables. Because of this
feature, the parallel environment differentiates between global and mesh
communicators. All the necessary communicators, and the attendant
meta-data is generated in this routine. The Driver_initFlash calls the
initialization routines in each of the units. All Unit API functions
have null implementations which get compiled in if the unit is not 
 included in a simulation. This feature allows a generic driver to
 suffice for the vast majority of application instances. 

.. _section-1:

The next routine is Driver_evolveFlash which controls the timestepping of the simulation,
as well as the normal termination of Flash-X based on some specified
criteria such as elapsed time, number of time steps etc. 

Unsplit Evolution
^^^^^^^^^^^^^^^^^

The default driver implementation in general calls each of the physics routines once
per time step, and each call advances solution vectors by one timestep.
At the end of one loop of timestep advancement, the condition for
updating the adaptive mesh refinement pattern is tested and applied.

Super-Time-Stepping (STS)
^^^^^^^^^^^^^^^^^^^^^^^^^

Another method implemented in is a technique called
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
() is a free parameter less than unity. For :math:`\nu \rightarrow 0`,
STS is asymptotically :math:`N_{\mbox{sts}}` times faster than the
conventional explicit scheme based on the CFL condition. During the
:math:`N_{sts}` sub-time steps, the STS method still solves solutions at
each intermediate step :math:`\tau_i`; however, such solutions should
not be considered as meaningful solutions.

Extended from the original STS method for accelerating parabolic
timestepping, our STS method advances advection and/or diffusion
(hyperbolic and/or parabolic) system of equations. This means that the
STS algorithm in Flash-X invokes a single :math:`\Delta t_{sts}` for
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
(:math:`\Delta t_{\mbox{CFL\_adv}}`); otherwise, Flash-X’s timestepping
will proceed without using STS iterations (i.e., using standard explicit
timestepping that is either Strang split evolution or unsplit evolution.

Since the method is explicit, it works equally well on both a uniform
grid and AMR grids without modification. The STS method is first-order
accurate in time.

Both directionally-split and unsplit hydro solvers can use the STS
method, simply by invoking a runtime parameter in . There are couple of
runtime parameters that control solution accuracy and stability. They
are decribed in Table `1.1 <#Tab:sts parameters>`__.

.. container:: center

   .. container::
      :name: Tab:sts parameters

      .. table::  Runtime parameters for STS

         +----------+---------+---------+------------------------------------+
         | Variable | Type    | Default | Description                        |
         +==========+=========+=========+====================================+
         |          | logical | .false. | Enable STS                         |
         +----------+---------+---------+------------------------------------+
         |          | integer | 5       | Suggestion: :math:`\sim` 5 for     |
         |          |         |         | hyperbolic; :math:`\sim` 10 for    |
         |          |         |         | parabolic                          |
         +----------+---------+---------+------------------------------------+
         |          | real    | 0.2     | Suggestion: :math:`\sim` 0.2 for   |
         |          |         |         | hyperbolic; :math:`\sim` 0.01 for  |
         |          |         |         | parabolic. It is known that a very |
         |          |         |         | low value of :math:`\nu` may       |
         |          |         |         | result in unstable temporal        |
         |          |         |         | integrations, while a value close  |
         |          |         |         | to unity can decrease the expected |
         |          |         |         | efficiency of STS.                 |
         +----------+---------+---------+------------------------------------+
         |          | logical | .false. | This setup will use the STS for    |
         |          |         |         | overcoming small diffusion time    |
         |          |         |         | steps assuming                     |
         |          |         |         | :math:`\Delta t_{\mbox{CFL\_adv    |
         |          |         |         | }} > \Delta t_{\mbox{CFL\_para}}`. |
         |          |         |         | In implementation, it will set     |
         |          |         |         | :math:`\Delta t_{\mbox{            |
         |          |         |         | CFL}}=\Delta t_{\mbox{CFL\_para}}` |
         |          |         |         | in Eqn.                            |
         |          |         |         | `[e                                |
         |          |         |         | qn:STS_flash] <#eqn:STS_flash>`__. |
         |          |         |         | Do not allow to turn on this       |
         |          |         |         | switch when there is no diffusion  |
         |          |         |         | (viscosity, conductivity, and      |
         |          |         |         | magnetic resistivity) used.        |
         +----------+---------+---------+------------------------------------+
         |          | logical | .false. | If true, this will allow to have   |
         |          |         |         | :math:`\ta                         |
         |          |         |         | u_i > \Delta t_{\mbox{CFL\_adv}}`, |
         |          |         |         | which may result in unstable       |
         |          |         |         | integrations.                      |
         +----------+---------+---------+------------------------------------+

Runtime Parameters
^^^^^^^^^^^^^^^^^^

The Driver unit supplies certain runtime parameters regardless of which
type of driver is chosen.

.. _section-2:

Finally, the the Driver unit calls which calls the finalize routines for
each unit. Typically this involves deallocating memory and any other
necessary cleanup.


Time Step Limiting
~~~~~~~~~~~~~~~~~~

The Driver unit is responsible for determining the time step
:math:`\Delta t` that is used to advance the solution from time
:math:`t=t^{n-1}` to :math:`t^n`. At startup, a tentative
:math:`\Delta t` is chosen based in runtime parameters, in particular .
The routine (usually invoked from ) is used to check and, if necessary,
modify the initial :math:`\Delta t`. Subsequently, the routine (usually
invoked from *at the end* of an iteration of the main evolution loop) is
used to recompute :math:`\Delta t` for the next evolution step.

The implementation of can lower or increase the time step. It takes
various runtime parameters into consideration, including , , and . For
the most part, however, calls on routines of various code units to query
those units for their time step requirements, and usually chooses the
smallest time step that is acceptable to all units queried.

The code units that participate in this negotiation return a
:math:`\Delta t`, and usually some additional information about which
location in the simulaton domain caused the reequired step to be as low
as returned. A unit’s time step requirement can depend on the current
state of the solution as well as on further runtime parameters. For
example, the returned by depends on the state of density, pressure, and
velocities (and possibly additional variables) in the cells of the
domain, as well as on the runtime parameter .

.. _`Sec:dr_posdef`:

