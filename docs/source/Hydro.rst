.. include:: defs.h

.. _`Chp:Hydrodynamics Unit`:

Hydrodynamics Units
===================

The ``Hydro`` unit solves Euler’s equations for compressible gas
dynamics in one, two, or three spatial dimensions. We first describe the
basic functionality; see implementation sections below for various
extensions.

The Euler equations can be written in conservative form as

.. math::

   \begin{aligned}
   {{\partial \rho} \over {\partial t}}
    + {\bf \nabla} \cdot \left ( \rho {\bf v} \right ) & = & 0\\
   {\partial \rho {\bf v} \over \partial t} +
    {\bf \nabla}  \cdot \left ( \rho {\bf v} {\bf v} \right ) +
    {\bf \nabla}  P
    & = & \rho {\bf g}\\
   {\partial \rho E \over \partial t} +
    {\bf \nabla} \cdot \left [ \left ( \rho E + P \right ) {\bf v}
    \right ] & = &
    \rho {\bf v} \cdot {\bf g}\ ,\end{aligned}

where :math:`\rho` is the fluid density, :math:`{\bf v}` is the fluid
velocity, :math:`P` is the pressure, :math:`E` is the sum of the
internal energy :math:`\epsilon` and kinetic energy per unit mass,

.. math:: E = \epsilon + {1 \over 2} |{\bf v}|^2\ ,

:math:`{\bf g}` is the acceleration due to gravity, and :math:`t` is the
time coordinate. The pressure is obtained from the energy and density
using the equation of state. For the case of an ideal gas equation of
state, the pressure is given by

.. math:: P = (\gamma - 1) \rho \epsilon\ ,

where :math:`\gamma` is the ratio of specific heats. More general
equations of state are discussed in and .

In regions where the kinetic energy greatly dominates the total energy,
computing the internal energy using

.. math:: \label{Eqn:intener} \epsilon = E - \frac{1}{2}|{\bf v}|^2

can lead to unphysical values, primarily due to truncation error. This
results in inaccurate pressures and temperatures. To avoid this problem,
we can separately evolve the internal energy according to

.. math::

   \label{Eqn:evolve_eint}
      \frac{\partial \rho \epsilon}{\partial t}
      + \nabla \cdot \left [ \left (\rho \epsilon + P \right){\bf v} \right ]
      - {\bf v}\cdot \nabla P = 0\ .

If the internal energy is a small fraction of the kinetic energy
(determined via the runtime parameter ``Eos/eintSwitch``), then the
total energy is recomputed using the internal energy from
`[Eqn:evolve_eint] <#Eqn:evolve_eint>`__ and the velocities from the
momentum equation. Numerical experiments using the PPM solver included
with |flashx| showed that using `[Eqn:evolve_eint] <#Eqn:evolve_eint>`__
when the internal energy falls below :math:`10^{-4}` of the kinetic
energy helps avoid the truncation errors while not affecting the
dynamics of the simulation.

For reactive flows, a separate advection equation must be solved for
each chemical or nuclear species

.. math::

   {{\partial \rho X_\ell} \over {\partial t}}
   + {\bf \nabla} \cdot \left ( \rho X_\ell {\bf v} \right ) = 0\ ,

where :math:`X_\ell` is the mass fraction of the :math:`\ell`\ th
species, with the constraint that :math:`\sum_\ell X_\ell = 1`. |flashx|
will enforce this constraint if you set the runtime parameter
``irenorm`` equal to 1. Otherwise, |flashx| will only restrict the
abundances to fall between ``smallx`` and 1. The quantity
:math:`\rho X_\ell` represents the partial density of the
:math:`\ell`\ th fluid. The code does not explicitly track interfaces
between the fluids, so a small amount of numerical mixing can be
expected during the course of a calculation.

The ``hydro`` unit has a capability to advect mass scalars. Mass scalars
are field variables advected with density, similar to species mass
fractions,

.. math::

   \label{eq:massscalar}
   {{\partial \rho \phi_\ell} \over {\partial t}}
   + {\bf \nabla} \cdot \left ( \rho \phi_\ell {\bf v} \right ) = 0\ ,

where :math:`\phi_\ell` is the :math:`\ell`\ th mass scalar. Note that
mass scalars are optional variables; to include them specify the name of
each mass scalar in a ``Config`` file using the ``MASS_SCALAR`` keyword.
Mass scalars are not renormalized in order to sum to 1, except when they
are declared to be part of a renormalization group. See for more
details.

Gas hydrodynamics
-----------------

.. _`Sec:PPM usage`:

Usage
~~~~~

The two gas hydrodynamic solvers supplied in the release of |flashx| are
organized into two different operator splitting methods: directionally
split and unsplit. The directionally split piecewise-parabolic method
(PPM) makes use of second-order Strang time splitting, and the new
directionally unsplit solver is based on Monotone Upstream-centered
Scheme for Conservation Laws (MUSCL) Hancock type second-order scheme.

The algorithms are described in and and implemented in the directory
tree under ``physics/Hydro/HydroMain/split/PPM`` and
``physics/Hydro/HydroMain/unsplit/Hydro_Unsplit``.

Current and future implementations of ``Hydro`` use the runtime
parameters and solution variables described in and . Additional runtime
parameters used either solely by the PPM method or the unsplit hydro
solver are described in ``Hydro/HydroMain``.

.. container:: center

   .. container::
      :name: Tab:hydro parameters

      .. table:: Runtime parameters used with the hydrodynamics
      (``Hydro``) unit.

         +----------------+---------+---------+-----------------------+
         | Variable       | Type    | Default | Description           |
         +================+=========+=========+=======================+
         | ``eintSwitch`` | real    | 0       | If                    |
         |                |         |         | :math:`\epsil         |
         |                |         |         | on < {\tt eintSwitch} |
         |                |         |         |   \cdot {1            |
         |                |         |         | \over 2}|{\bf v}|^2`, |
         |                |         |         | use the internal      |
         |                |         |         | energy equation to    |
         |                |         |         | update the pressure   |
         +----------------+---------+---------+-----------------------+
         | ``irenorm``    | integer | 0       | If equal to one,      |
         |                |         |         | renormalize           |
         |                |         |         | multifluid abundances |
         |                |         |         | following a hydro     |
         |                |         |         | update; else restrict |
         |                |         |         | their values to lie   |
         |                |         |         | between ``smallx``    |
         |                |         |         | and 1.                |
         +----------------+---------+---------+-----------------------+
         | ``cfl``        | real    | 0.8     | Co                    |
         |                |         |         | urant-Friedrichs-Lewy |
         |                |         |         | (CFL) factor; must be |
         |                |         |         | less than 1 for       |
         |                |         |         | stability in explicit |
         |                |         |         | schemes               |
         +----------------+---------+---------+-----------------------+

.. container:: center

   .. container::
      :name: Tab:hydro variables

      .. table:: Solution variables used with the hydrodynamics
      (``Hydro``) unit.

         ======== ========== ===================================
         Variable Type       Description
         ======== ========== ===================================
         ``dens`` PER_VOLUME density
         ``velx`` PER_MASS   :math:`x`-component of velocity
         ``vely`` PER_MASS   :math:`y`-component of velocity
         ``velz`` PER_MASS   :math:`z`-component of velocity
         ``pres`` GENERIC    pressure
         ``ener`` PER_MASS   specific total energy (:math:`T+U`)
         ``temp`` GENERIC    temperature
         ======== ========== ===================================

.. _`Sec:MUSCL-Hancock`:

The unsplit hydro solver
~~~~~~~~~~~~~~~~~~~~~~~~

A directionally unsplit pure hydrodynamic solver (unsplit hydro) is an
alternate gas dynamics solver to the split PPM scheme. The method
basically adopts a predictor-corrector type formulation (zone-edge
data-extrapolated method) that provides second-order solution accuracy
for smooth flows and first-order accuracy for shock flows in both space
and time. Recently, the order of spatial accuracy in data reconstruction
for the normal direction has been extended to implement the 3rd order
PPM and 5th order Weighted ENO (WENO) methods. This unsplit hydro solver
can be considered as a reduced version of the Unsplit Staggered Mesh
(USM) MHD solver (see details in ) that has been available in previous
|flashx| releases.

The unsplit hydro implementation can solve 1D, 2D and 3D problems with
added capabilities of exploring various numerical implementations:
different types of Riemann solvers; slope limiters; first, second, third
and fifth reconstruction methods; a strong shock/rarefaction detection
algorithm as well as two different entropy fix routines for Roe’s
linearized Riemann solver.

One of the notable features of the unsplit hydro scheme is that it
particularly improves the preservation of flow symmetries as compared to
the splitting formulation. Also, the scheme used in this unsplit
algorithm can take a wide range of CFL stability limits (e.g., CFL
:math:`<` 1) for all three dimensions, which is based on using upwinded
transverse flux formulations developed in the multidimensional USM MHD
solver (Lee, 2006; Lee and Deane, 2009; Lee, 2013).

.. container:: center

   .. container::
      :name: Tab:unsplit hydro parameters

      .. table::  Additional runtime parameters for *Interpolation
      Schemes* in the unsplit hydro solver
      (``physics/Hydro/HydroMain/unsplit/Hydro_Unsplit``)

         +----------------------+---------+-----------+----------------------+
         | Variable             | Type    | Default   | Description          |
         +======================+=========+===========+======================+
         | ``order``            | integer | 2         | Order of method in   |
         |                      |         |           | data reconstruction: |
         |                      |         |           | 1st order Godunov    |
         |                      |         |           | (FOG), 2nd order     |
         |                      |         |           | MUSCL-Hancock (MH),  |
         |                      |         |           | 3rd order PPM, 5th   |
         |                      |         |           | order WENO.          |
         +----------------------+---------+-----------+----------------------+
         | ``transOrder``       | integer | 1         | Interpolation order  |
         |                      |         |           | of accuracy of       |
         |                      |         |           | taking upwind biased |
         |                      |         |           | transverse flux      |
         |                      |         |           | derivatives in the   |
         |                      |         |           | unsplit data         |
         |                      |         |           | reconstruction: 1st, |
         |                      |         |           | 2nd, 3rd. The choice |
         |                      |         |           | of using             |
         |                      |         |           | ``transOrder``\ =4   |
         |                      |         |           | adopts a slope       |
         |                      |         |           | limiter between the  |
         |                      |         |           | 1st and 3rd order    |
         |                      |         |           | accurate methods to  |
         |                      |         |           | minimize             |
         |                      |         |           | oscillations in      |
         |                      |         |           | upwinding at         |
         |                      |         |           | discontinuities.     |
         +----------------------+---------+-----------+----------------------+
         | ``slopeLimiter``     | string  | “vanLeer” | Slope limiter:       |
         |                      |         |           | “MINMOD”, “MC”,      |
         |                      |         |           | “VANLEER”, “HYBRID”, |
         |                      |         |           | “LIMITED”            |
         +----------------------+---------+-----------+----------------------+
         | ``LimitedSlopeBeta`` | real    | 1.0       | Slope parameter      |
         |                      |         |           | specific for the     |
         |                      |         |           | “LIMITED" slope by   |
         |                      |         |           | Toro                 |
         +----------------------+---------+-----------+----------------------+
         | ``charLimiting``     | logical | .true.    | Enable/disable       |
         |                      |         |           | limiting on          |
         |                      |         |           | characteristic       |
         |                      |         |           | variables (.false.   |
         |                      |         |           | will use limiting on |
         |                      |         |           | primitive variables) |
         +----------------------+---------+-----------+----------------------+
         | ``use_steepening``   | logical | .false.   | Enable/disable       |
         |                      |         |           | contact              |
         |                      |         |           | discontinuity        |
         |                      |         |           | steepening for PPM   |
         |                      |         |           | and WENO             |
         +----------------------+---------+-----------+----------------------+
         | ``use_flattening``   | logical | .false.   | Enable/disable       |
         |                      |         |           | flattening (or       |
         |                      |         |           | reducing) numerical  |
         |                      |         |           | oscillations for MH, |
         |                      |         |           | PPM, and WENO        |
         +----------------------+---------+-----------+----------------------+
         | ``use_avisc``        | logical | .false.   | Enable/disable       |
         |                      |         |           | artificial viscosity |
         |                      |         |           | for FOG, MH, PPM,    |
         |                      |         |           | and WENO             |
         +----------------------+---------+-----------+----------------------+
         | ``cvisc``            | real    | 0.1       | Artificial viscosity |
         |                      |         |           | coefficient          |
         +----------------------+---------+-----------+----------------------+
         | ``use_upwindTVD``    | logical | .false.   | Enable/disable       |
         |                      |         |           | upwinded TVD slope   |
         |                      |         |           | limiter PPM. NOTE:   |
         |                      |         |           | This requires        |
         |                      |         |           | NGUARD=6             |
         +----------------------+---------+-----------+----------------------+
         | ``use_hybridOrder``  | logical | .false.   | Enable an adaptively |
         |                      |         |           | varying              |
         |                      |         |           | reconstruction order |
         |                      |         |           | scheme reducing its  |
         |                      |         |           | order from a         |
         |                      |         |           | high-order to        |
         |                      |         |           | first-order          |
         |                      |         |           | depending on         |
         |                      |         |           | monotonicity         |
         |                      |         |           | constraints          |
         +----------------------+---------+-----------+----------------------+
         | ``                   | logical | .false.   | On/off gravitational |
         | use_gravHalfUpdate`` |         |           | acceleration source  |
         |                      |         |           | terms at the half    |
         |                      |         |           | time Riemann state   |
         |                      |         |           | update               |
         +----------------------+---------+-----------+----------------------+
         | ``use_3dFullCTU``    | logical | .true.    | Enable a full CTU    |
         |                      |         |           | (e.g., similar to    |
         |                      |         |           | the standard         |
         |                      |         |           | 12-Riemann solve)    |
         |                      |         |           | algorithm that       |
         |                      |         |           | provides full CFL    |
         |                      |         |           | stability in 3D. If  |
         |                      |         |           | .false., then the    |
         |                      |         |           | theoretical CFL      |
         |                      |         |           | bound for 3D becomes |
         |                      |         |           | less than 0.5 based  |
         |                      |         |           | on the linear        |
         |                      |         |           | Fourier analysis.    |
         +----------------------+---------+-----------+----------------------+

.. container:: center

   .. container::
      :name: Tab:unsplit hydro parameters - Riemann

      .. table::  Additional runtime parameters for *Riemann Solvers* in
      the unsplit hydro solver
      (``physics/Hydro/HydroMain/unsplit/Hydro_Unsplit``)

         +------------------+---------+------------------+------------------+
         | Variable         | Type    | Default          | Description      |
         +==================+=========+==================+==================+
         | `                | string  | “Roe"            | Different        |
         | `RiemannSolver`` |         |                  | choices for      |
         |                  |         |                  | Riemann solver.  |
         |                  |         |                  | “LLF (local      |
         |                  |         |                  | L                |
         |                  |         |                  | ax-Friedrichs)”, |
         |                  |         |                  | “HLL", “HLLC",   |
         |                  |         |                  | “HYBRID", “ROE", |
         |                  |         |                  | and “Marquina"   |
         +------------------+---------+------------------+------------------+
         | ``shockDetect``  | logical | .false.          | On/off           |
         |                  |         |                  | attempting to    |
         |                  |         |                  | detect strong    |
         |                  |         |                  | sho              |
         |                  |         |                  | cks/rarefactions |
         |                  |         |                  | (and saving flag |
         |                  |         |                  | in ``"shok"``    |
         |                  |         |                  | variable)        |
         +------------------+---------+------------------+------------------+
         | `                | logical | .false.          | On/off lowering  |
         | `shockLowerCFL`` |         |                  | of CFL factor    |
         |                  |         |                  | where strong     |
         |                  |         |                  | shocks are       |
         |                  |         |                  | detected,        |
         |                  |         |                  | automatically    |
         |                  |         |                  | sets             |
         |                  |         |                  | ``shockDetect``  |
         |                  |         |                  | if on.           |
         +------------------+---------+------------------+------------------+
         | `                | logical | .false.          | Enable/disable   |
         | `EOSforRiemann`` |         |                  | calling EOS in   |
         |                  |         |                  | computing each   |
         |                  |         |                  | Godunov flux     |
         +------------------+---------+------------------+------------------+
         | ``entropy``      | logical | .false.          | On/off entropy   |
         |                  |         |                  | fix algorithm    |
         |                  |         |                  | for Roe solver   |
         +------------------+---------+------------------+------------------+
         | ``en             | string  | `                | Entropy fix      |
         | tropyFixMethod`` |         | `“HARTENHYMAN”`` | method for the   |
         |                  |         |                  | Roe solver.      |
         |                  |         |                  | “HARTEN",        |
         |                  |         |                  | “HARTENHYMAN"    |
         +------------------+---------+------------------+------------------+

The above set of runtime parameters provide various types of different
combinations that help in obtaining numerical accuracy, efficiency and
stability. However, there are some important tips users should know
before using them.

-  [Extended stencil]: When ``NGUARD=6`` is used, users should also use
   ``nxb, nyb``, and ``nzb`` larger than ``2*NGUARD``. For example,
   specifying ``-nxb=16`` in the setup works well for 1D cases. Once
   setting up ``NGUARD=6``, users still can use FOG, MH, PPM, or WENO
   without changing ``NGUARD`` back to 4.

-  [``transOrder``]: The first order method ``transOrder=1`` is a
   default and only supported method that is stable according to the
   linear Fourier stability analysis. The choices for higher-order
   interpolations are no longer available in this release.

-  [``EOSforRiemann``]: ``EOSforRiemann = .true.`` will call (expensive)
   EOS routines to compute consistent adiabatic indices (*i.e.*,
   ``gamc, game``) according to the given left and right states in
   Riemann solvers. For the ideal gamma law, in which those adiabatic
   indices are constant, it is not required to call EOS at all and users
   can set it ``.false.`` to reduce computation time. On the other hand,
   for a degenerate gas, one can enable this switch to compute
   thermodynamically consistent ``gamc, game``, which in turn are used
   to compute the sound speed and internal energy in Riemann flux
   calculations. When disabled, interpolations will be used instead to
   get approximations of ``gamc, game``. This interpolation method has
   been tested and proven to gain significant computational efficiency
   and accuracy, giving reliable numerical solutions even for simulating
   a degenerate gas.

-  [Gravity coupling with Unplit Hydro Solvers]: When gravity is
   included in a simulation using the unsplit hydro and MHD solvers,
   users can choose to include gravitational source terms in the Riemann
   state update at :math:`n+1/2` time step (i.e.,
   ``use_gravHalfUpdate``\ =.true.). This will provide an improved
   second-order accuracy with respect to coupling gravitational
   accelerations to hydrodynamics. It should be noted that current
   optimized unsplit hydro/MHD codes (*e.g.*, those selected with
   ``+uhd, +usm``) do not support the runtime parameters
   ``use_gravPotUpdate`` and ``use_gravConsv`` of some previous |flashx|
   versions any more.

-  [Reduced CTU vs. Full CTU for 3D in the unsplit hydro (UHD) and
   staggered mesh (USM) solvers]: ``use_3dFullCTU`` is a new switch that
   enhances a numerical stability for 3D simulations in the unsplit
   solvers using the corner transport upwind (CTU) algorithm by Colella.
   The unsplit solvers of |flashx| are different from many other shock
   capturing codes, in that neither UHD nor USM solvers need
   intermediate Riemann solver solutions for updating transverse fluxes
   in multidimensional problems. This provides a computational
   efficiency because there is a reduced number of calls to Riemann
   solvers per cell per time step. The total number of required Riemann
   solver solutions are two for 2D and three for 3D (except for extra
   Riemann calls for constraint-transport (CT) update in USM). This is
   smaller than the usual stabilty requirement in many other codes which
   needs four for 2D and twelve for 3D in order to provide a full CFL
   limit (*i.e.*, CFL :math:`<1`).

   In general for 3D, there is another computationally efficient
   approach that only uses six Riemann solutions (aka, 6-CTU) instead of
   solving twelve Riemann problems (aka, 12-CTU). In this efficient
   6-CTU, however, the numerical stability limit becomes
   CFL\ :math:`<0.5`.

   For solving 3D problems in UHD and USM, enabling the new switch
   ``use_3dFullCTU=.true.`` (i.e., full-CTU) will make the solution
   evolution scheme similar to 12-CTU while requiring to solve three
   Riemann problems only (again, except for the CT update in USM). On
   the other hand, ``use_3dFullCTU=.false.`` (i.e., reduced-CTU) will be
   similar to the 6-CTU integration algorithm with a reduced CFL limit
   (*i.e.*, CFL :math:`<0.5`).

.. container:: flashtip

   The unsplit solver can take a wide range of CFL limits in all three
   dimensions (*i.e.*, CFL :math:`<` 1). However, in some circumstances
   where there are strong shocks and rarefactions,
   ``shockLowerCFL=.true.`` could be useful to gain more numerical
   stability by lowering the CFL accordingly (e.g., default settings
   provide 0.45 for 2D and 0.25 for 3D for the Donor scheme). This
   approach will automatically revert such reduced stability conditions
   to any given original condition set by users when there are no
   significant shocks and rarefactions detected.
