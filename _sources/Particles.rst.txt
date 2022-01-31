.. include:: defs.h

.. _`Chp:Particles`:

Particles Unit
==============

The support for particles in |flashx| will eventually be in two flavors,
*active* and *passive*. This version only supports passive particles.
Passive particles acquire their kinematic information (velocities)
directly from the mesh. They are meant to be used as passive flow
tracers and do not make sense outside of a hydrodynamical context. The
governing equation for the :math:`i`\ th passive particle is
particularly simple and requires only the time integration of
interpolated mesh velocities.

.. math::

   \label{Eqn:particle_motion_velocity}
   {d{\bf x}_i\over dt}    =  {\bf v}_i

Active particles experience forces and may themselves contribute to the
problem dynamics (*e.g.*, through long-range forces or through
collisions). They may additionally have their own motion independent of
the grid, so an additional motion equation of

.. math::

   \label{Eq:particle_motion_acceleration}
         {\bf v}_i^{n+1} = {\bf v}_i^n + {\bf a}_i^n\Delta t^n ~.

may come into play. Here :math:`{\bf a}_i` is the particle acceleration.
Solving for the motion of active particles is also referred to as
solving the :math:`N`-body problem. The equations of motion for the
:math:`i`\ th active particle include the equation
`[Eqn:particle_motion_velocity] <#Eqn:particle_motion_velocity>`__ and
another describing the effects of forces.

.. math::

   \label{Eqn:particle_motion_forces}
   m_i{d{\bf v}_i\over dt} =  {\bf F}_{{\rm lr,}i} + {\bf F}_{{\rm sr,}i}\ ,

Here, :math:`{\bf F}_{{\rm lr,}i}` represents the sum of all long-range
forces (coupling all particles, except possibly those handled by the
short-range term) acting on the :math:`i`\ th particle and
:math:`{\bf F}_{{\rm sr,}i}` represents the sum of all short-range
forces (coupling only neighboring particles) acting on the particle.

For both types of particles, the primary challenge is to integrate
`[Eqn:particle_motion_velocity] <#Eqn:particle_motion_velocity>`__
forward through time. Many alternative integration methods are described
in Section  below. Additional information about the mesh to particle
mapping is described in . An introduction to the particle techniques
used in |flashx| is given by R. W. Hockney and J. W. Eastwood in
*Computer Simulation using Particles* (Taylor and Francis, 1988).

.. _`Sec:Particles Integration`:

Time Integration
----------------

The passive particles have many different time integration schemes
available. The subroutine ``Particles/Particles_advance`` handles the
movement of particles through space and time. Because |flashx| will have
support for including different types of both active and passive
particles in a single simulation, the implementation of
``Particles/Particles_advance`` may call several helper routines of the
form ``pt_advanceMETHOD`` (*e.g.*, ``pt_advanceLeapfrog``,
``pt_advanceEuler_active``, ``pt_advanceCustom``), each acting on an
appropriate subset of existing particles. The *METHOD* here is
determined by the ``ADVMETHOD`` part of the ``PARTICLETYPE`` ``Config``
statement (or the ``ADV`` par of a ``-particlemethods`` option) for the
type of particle. See the ``Particles_advance`` source code for the
mapping from ``ADVMETHOD`` keyword to ``pt_advanceMETHOD`` subroutine
call.

.. _`Sec:Passive Partices Integration`:

Passive Particles
~~~~~~~~~~~~~~~~~

Passive particles may be moved using one of several different methods
available in |flashx|. With the exception of **Midpoint**, they are all
single-step schemes. The methods are either first-order or second-order
accurate, and all are explicit, as described below. In all
implementations, particle velocities are obtained by mapping grid-based
velocities onto particle positions as described in .

Numerically solving
Equation `[Eqn:particle_motion_velocity] <#Eqn:particle_motion_velocity>`__
for passive particles means solving a set of simple ODE initial value
problems, separately for each particle, where the velocities
:math:`{\bf v}_i` are given at certain discrete points in time by the
state of the hydrodynamic velocity field at those times. The time step
is thus externally given and cannot be arbitrarily chosen by a particle
motion ODE solver [1]_. Statements about the order of a method in this
context should be understood as referring to the same method if it were
applied in a hypothetical simulation where evaluations of velocities
:math:`{\bf v}_i` could be performed at arbitrary times (and with
unlimited accuracy). Note that |flashx| does not attempt to provide a
particle motion ODE solver of higher accuracy than second order, since
it makes little sense to integrate particle motion to a higher order
than the fluid dynamics that provide its inputs.

In all cases, particles are advanced in time from :math:`t^n` (or, in
the case of ``Midpoint``, from :math:`t^{n-1}`) to
:math:`t^{n+1} = t^n + \Delta
t^n` using one of the difference equations described below. The
implementations assume that at the time when
``Particles/Particles_advance`` is called, the fluid fields have already
been updated to :math:`t^{n+1}`, as is the case with the
``Driver/Driver_evolveFlash`` implementations provided with |flashx|. A
specific implementation of the passive portion of
``Particles/Particles_advance`` is selected by a setup option such as
``-with-unit=Particles/ParticlesMain/passive/Euler``, or by specifying
something like ``REQUIRES Particles/ParticlesMain/passive/Euler`` in a
simulation’s ``Config`` file (or by listing the path in the ``Units``
file if not using the ``-auto`` configuration option). Further details
are given is below.

-  **Forward Euler (``Particles/ParticlesMain/passive/Euler``).**
   Particles are advanced in time from :math:`t^n` to
   :math:`t^{n+1} = t^n + \Delta t^n` using the following difference
   equation:

   .. math::

      \label{Eqn:particle_passive_euler}
            {\bf x}_i^{n+1} = {\bf x}_i^n + {\bf v}_i^n\Delta t^n \quad.

   Here :math:`{\bf v}_i^n` is the velocity of the particle, which is
   obtained using particle-mesh interpolation from the grid at
   :math:`t=t^n`.

   Note that this evaluation of :math:`{\bf v}_i^n` cannot be deferred
   until the time when it is needed at :math:`t=t^{n+1}`, since at that
   point the fluid variables have been updated and the velocity fields
   at :math:`t=t^n` are not available any more. Particle velocities are
   therefore interpolated from the mesh at :math:`t=t^n` and stored as
   particle attributes. Similar concerns apply to the remaining methods
   but will not be explicitly mentioned every time.

-  **Two-Stage Runge-Kutta
   (``Particles/ParticlesMain/passive/RungeKutta``).** This 2-stage
   Runge-Kutta scheme is the preferred choice in |flashx|. It is also the
   default which is compiled in if particles are included in the setup
   but no specific alternative implementation is requested. The scheme
   is also known as Heun’s Method:

   .. math::

      \begin{aligned}
          \label{Eqn:particle_passive_rk2}
            {\bf x}_i^{n+1} &=& {\bf x}_i^n + \frac{\Delta t^n}{2} 
                           \left[ {\bf v}_i^n + {\bf v}_i^{*,n+1} \right]      \;,\\
            \hbox{where}\quad {\bf v}_i^{*,n+1} &=& {\bf v}({\bf x}_i^{*,n+1},t^{n+1})    \;,\nonumber\\
            {\bf x}_i^{*,n+1} &=& {\bf x}_i^n + \Delta t^n
                           {\bf v}_i^n       \quad.\nonumber
            \end{aligned}

   Here :math:`{\bf v}({\bf x},t)` denotes evaluation (interpolation) of
   the fluid velocity field at position :math:`\bf x` and time
   :math:`t`; :math:`{\bf v}_i^{*,n+1}` and :math:`{\bf x}_i^{*,n+1}`
   are intermediate results  [2]_; and
   :math:`{\bf v}_i^n = {\bf v}({\bf x}_i^{n},t^{n})` is the velocity of
   the particle, obtained using particle-mesh interpolation from the
   grid at :math:`t=t^n` as usual.

-  **Midpoint (``Particles/ParticlesMain/passive/Midpoint``).** This
   Midpoint scheme is a two-step scheme. Here, the particles are
   advanced from time :math:`t^{n-1}` to
   :math:`t^{n+1} = t^{n-1} + \Delta t^{n-1} + \Delta t^{n}` by the
   equation

   .. math:: {\bf x}_i^{n+1} = {\bf x}_i^{n-1} + {\bf v}_i^{n}(\Delta t^{n-1} + \Delta t^{n}) \quad.

   The scheme is second order if :math:`\Delta t^n = \Delta t^{n-1}`.

   To get the scheme started, an Euler step (as described for
   ``passive/Euler``) is taken the first time
   ``Particles/ParticlesMain/passive/Midpoint/pt_advancePassive`` is
   called.

   The ``Midpoint`` alternative implementation uses the following
   additional particle attributes:

   .. container:: codeseg

      PARTICLEPROP pos2PrevX REAL # two previous x-coordinate
      PARTICLEPROP pos2PrevY REAL # two previous y-coordinate
      PARTICLEPROP pos2PrevZ REAL # two previous z-coordinate

-  **Estimated Midpoint with Correction
   (``Particles/ParticlesMain/passive/EstiMidpoint2``).** The scheme is
   second order even if :math:`\Delta t^n = \Delta t^{n+1}` is not
   assumed. It is essentially the ``EstiMidpoint`` or
   “Predictor-Corrector” method of previous releases, with a correction
   for non-constant time steps by using additional evaluations (at times
   and positions that are easily available, without requiring more
   particle attributes).

   Particle advancement follows the equation ‘_=8 ‘_=8

   .. math::

      \label{Eqn:EstiMidpoint2}
      {\bf x}_i^{n+1} = {\bf x}_i^n +   {{\Delta t}}^n \, {\bf v}_i^{\mathrm{comb}}\;,

   where

   .. math::

      \label{Eqn:pt_em2-2}
      {\bf v}_i^{\mathrm{comb}}=
      c_1\,{\bf v}({\bf x}_i^{*,n+\frac 12},t^{n})   +
      c_2\,{\bf v}({\bf x}_i^{*,n+\frac 12},t^{n+1})   +
      c_3\,{\bf v}({\bf x}_i^{n},t^{n})   +
      c_4\,{\bf v}({\bf x}_i^{n},t^{n+1})

   is a combination of four evaluations (two each at the previous and
   the current time),

   .. math:: {\bf x}_i^{*,n+\frac 12}= {\bf x}_i^n + \frac12 \Delta t^{n-1} {\bf v}_i^n

   are estimated midpoint positions as before in the Estimated Midpoint
   scheme, and the coefficients

   .. math::

      \begin{aligned}
      c_1&=&c_1({\Delta t}^{n-1},{\Delta t}^n)\;,\\
      c_2&=&c_2({\Delta t}^{n-1},{\Delta t}^n)\;,\\
      c_3&=&c_3({\Delta t}^{n-1},{\Delta t}^n)\;,\\
      c_4&=&c_4({\Delta t}^{n-1},{\Delta t}^n)\end{aligned}

   are chosen dependent on the change in time step so that the method
   stays second order when :math:`{\Delta t}^{n-1} \neq {\Delta t}^n`.

   Conditions for the correction can be derived as follows: Let
   :math:`\Delta t_{*}^{n} =\frac 12 \Delta t^{n-1}` the estimated half
   time step used in the scheme, let
   :math:`t_{*}^{n+\frac 12}=t^n + \Delta t_{*}^{n}` the estimated
   midpoint time, and :math:`t^{n+\frac 12}=t^n + \frac 12 \Delta t^{n}`
   the actual midpoint of the :math:`[t^n,t^{n+1}]` interval. Also write
   :math:`{\bf x}_i^{{\mathrm{E}},n+\frac 12}= {\bf x}_i^{n}+ \frac 12 \Delta t^{n}{\bf v}_i^{n}`
   for first-order (Euler) approximate positions at the actual midpoint
   time :math:`t^{n+\frac 12}`, and we continue to denote with
   :math:`{\bf x}_i^{*,n+\frac 12}` the estimated positions reached at
   the estimated mipoint time :math:`t_{*}^{n+\frac 12}`.

   Assuming reasonably smooth functions :math:`{\bf v}({\bf x},t)`, we
   can then write for the exact value of the velocity field at the
   approximate positions evaluated at the actual midpoint time

   .. math::

      \label{Eqn:vhalf_expansion}
      {\bf v}({\bf x}_i^{{\mathrm{E}},n+\frac 12},t^{n+\frac 12}) = 
         {\bf v}({\bf x}_i^{n},t^{n}) +
            {\bf v}_t({\bf x}_i^{n},t^{n})\frac 12 \Delta t^{n}+
            ({\bf v}_i^{n}\cdot \frac{\partial}{\partial {\bf x}}) {\bf v}({\bf x}_i^{n},t^{n})\frac 12 \Delta t^{n}+
               O((\frac 12 \Delta t^{n})^2)

   by Taylor expansion. It is known that the propagation scheme
   :math:`\widetilde{{\bf x}}_i^{n+1}={\bf x}_i^{n}+{\bf v}({\bf x}_i^{{\mathrm{E}},n+\frac 12},t^{n+\frac 12}) {\Delta t}`
   using these velocities is second order (this is known as the modified
   Euler method).

   On the other hand, expansion of `[Eqn:pt_em2-2] <#Eqn:pt_em2-2>`__
   gives

   .. math::

      \begin{aligned}
      {\bf v}_i^{\mathrm{comb}}&=& 
      (c_1+c_2+c_3+c_4) {\bf v}({\bf x}_i^{n},t^{n}) \nonumber\\
      && +\;
         (c_2+    c_4) {\bf v}_t({\bf x}_i^{n},t^{n}) {\Delta t}\;+\;
        (c_1+c_2)  ({\bf v}_i^{n}\cdot \frac{\partial}{\partial {\bf x}}) {\bf v}({\bf x}_i^{n},t^{n}) \Delta t_{*}^{n} \nonumber\\
      && \quad +    \quad    \mbox{higher order terms in ${\Delta t}$ and $\Delta t_{*}^{n} $.} \nonumber\end{aligned}

   After introducing a time step factor :math:`f` defined by
   :math:`\Delta t_{*}^{n} = f\,{\Delta t}^{n}`, this becomes

   .. math::

      \begin{aligned}
          \label{Eqn:vcomb_expansion}
      {\bf v}_i^{\mathrm{comb}}&=& 
      (c_1+c_2+c_3+c_4) {\bf v}({\bf x}_i^{n},t^{n}) \\
      && +\;
         (c_2+    c_4) {\bf v}_t({\bf x}_i^{n},t^{n}) {\Delta t}\;+\;
        (c_1+c_2)({\bf v}_i^{n}\cdot \frac{\partial}{\partial {\bf x}}) {\bf v}({\bf x}_i^{n},t^{n}) \,f\,{\Delta t}\nonumber\\
      && \quad  +  \; O(({\Delta t})^2)\quad. \nonumber\end{aligned}

   One can derive conditions for second order accuracy by comparing
   `[Eqn:vcomb_expansion] <#Eqn:vcomb_expansion>`__ with
   `[Eqn:vhalf_expansion] <#Eqn:vhalf_expansion>`__ and requiring that

   .. math:: {\bf v}_i^{\mathrm{comb}}=  {\bf v}({\bf x}_i^{{\mathrm{E}},n+\frac 12},t^{n+\frac 12}) +  O(({\Delta t})^2)\quad.

   It turns out that the coefficients have to satisfy three conditions
   in order to eliminate from the theoretical difference between
   numerical and exact solution all :math:`O({\Delta t}^{n-1})` and
   :math:`O({\Delta t}^{n})` error terms:

   .. math::

      \begin{aligned}
         c_1+c_2+c_3+c_4&=& 1  \quad  \mbox{(otherwise the scheme will not even be of first order)}\;,\\
            c_2+c_4&=& \frac12 \quad  \mbox{(and thus also $c_1+c_3=\frac12$)}\;, \\ 
         c_1+c_2&=& \frac{ {\Delta t}^n } { {\Delta t}^{n-1} } \quad.\end{aligned}

   The provided implementation chooses :math:`c_4=0` (this can be easily
   changed if desired by editing in the code). All four coefficients are
   then determined:

   .. math::

      \begin{aligned}
      c_1&=&\frac{ {\Delta t}^n } { {\Delta t}^{n-1} } - \frac12\;,\\
      c_2&=&\frac12\;,\\
      c_3&=&1 - \frac{ {\Delta t}^n } { {\Delta t}^{n-1} }\;,\\
      c_4&=&0\quad.
            \end{aligned}

   Note that when the time step remains unchanged we have
   :math:`c_1=c_2=\frac12` and :math:`c_3=c_4=0`, and so
   `[Eqn:EstiMidpoint2] <#Eqn:EstiMidpoint2>`__ simplifies
   significantly.

   An Euler step, as described for ``passive/Euler`` in
   `[Eqn:particle_passive_euler] <#Eqn:particle_passive_euler>`__, is
   taken the first time when
   ``Particles/ParticlesMain/passive/EstiMidpoint2/pt_advancePassive``
   is called and when the time step has changed too much. Since the
   integration scheme is tolerant of time step changes, it should
   usually not be necessary to apply the second criterion; even when it
   is to be employed, the criteria should be less strict than for an
   uncorrected ``EstiMidpoint`` scheme. For ``EstiMidPoint2`` the
   timestep is considered to have changed too much if either of the
   following is true:

   .. math::

      \Delta t^{n} > \Delta t^{n-1} 
      \quad\mbox{and}\quad
          \left| \Delta t^{n} - \Delta t^{n-1} \right| \geq
                                                            {\tt pt\_dtChangeToleranceUp} \times  \Delta t^{n-1}

   or

   .. math::

      \Delta t^{n} < \Delta t^{n-1} 
      \quad\mbox{and}\quad
          \left| \Delta t^{n} - \Delta t^{n-1} \right| \geq
                                                            {\tt pt\_dtChangeToleranceDown} \times  \Delta t^{n-1},

   where ``Particles/pt_dtChangeToleranceUp`` and
   ``Particles/pt_dtChangeToleranceDown`` are runtime parameter specific
   to the ``EstiMidPoint2`` alternative implementation.

   The ``EstiMidpoint2`` alternative implementation uses the following
   additional particle attributes for storing the values of
   :math:`{\bf x}_i^{*,n+\frac 12}` and :math:`{\bf v}_i^{*,n+\frac 12}`
   between the ``Particles_advance`` calls at :math:`t^n` and
   :math:`t^{n+1}`:

   .. container:: codeseg

      PARTICLEPROP velPredX REAL PARTICLEPROP velPredY REAL PARTICLEPROP
      velPredZ REAL PARTICLEPROP posPredX REAL PARTICLEPROP posPredY
      REAL PARTICLEPROP posPredZ REAL

The time integration of passive particles is tested in the
``ParticlesAdvance`` unit test, which can be used to examine the
convergence behavior, see .

.. _`Sec:Particles Mapping`:

Mesh/Particle Mapping
---------------------

Particles behave in a fundamentally different way than grid-based
quantities. Lagrangian, or passive particles are essentially independent
of the grid mesh and move along with the velocity field. Active
particles may be located independently of mesh refinement. In either
case, there is a need to convert grid-based quantities into similar
attributes defined on particles, or vice versa. The method for
interpolating mesh quantities to tracer particle positions must be
consistent with the numerical method to avoid introducing systematic
error. In the case of a finite-volume methods such as those used in
|flashx|, the mesh quantities have cell-averaged rather than point
values, which requires that the interpolation function for the particles
also represent cell-averaged values. Cell averaged quantities are
defined as

.. math::

   f_i\left(x\right) \: \equiv \: \frac{1}{\Delta x} \int_{x_{i-1/2}}^{x_{i-1/2}}
   f\left(x^\prime \right) dx^\prime

where :math:`i` is the cell index and :math:`\Delta x` is the spatial
resolution. The mapping back and forth from the mesh to the particle
properties are defined in the routines ``Particles_mapFromMesh`` and
``Particles_mapToMeshOneBlk``.

Specifying the desired mapping method is accomplished by designating the
``MAPMETHOD`` in the Simulation ``Config`` file for each type of
particle. See for more details.

Quadratic Mesh Mapping
~~~~~~~~~~~~~~~~~~~~~~

The quadratic mapping package defines an interpolation back and forth to
the mesh which is second order. This implementation is primarily meant
to be used with passive tracer particles.

To derive it, first consider a second-order interpolation function of
the form

.. math:: f\left(x\right) \: = \: A + B \left(x - x_i\right) + C \left(x - x_i\right)^2 \; .

Then integrating gives

.. math::

   \begin{aligned}
   f_{i-1} \: & = & \: \frac{1}{\Delta x}\left[ A + \left. \frac{1}{2}
   B \left(x - x_i\right)^2 \right|^{x_{i-1/2}}_{x_{i-3/2}} + \left. \frac{1}{3}
   C \left(x - x_i\right)^3 \right|^{x_{i-1/2}}_{x_{i-3/2}} \right]\nonumber  \\
              & = & \: A - B\Delta x + \frac{13}{12} C \Delta x^2 \;,\end{aligned}

.. math::

   \begin{aligned}
   f_{i} \: & = & \: \frac{1}{\Delta x}\left[ A + \left. \frac{1}{2} B \left(x - x_i\right)^2 \right|^{x_{i+1/2}}_{x_{i-1/2}}
   + \left. \frac{1}{3}  C \left(x - x_i\right)^3 \right|^{x_{i+1/2}}_{x_{i-1/2}}  \right]\nonumber \\
              & = & \: A + \frac{1}{12} C \Delta x^2 \; ,\end{aligned}

and

.. math::

   \begin{aligned}
   f_{i+1} \: & = & \: \frac{1}{\Delta x}\left[ A + \left.\frac{1}{2} B \left(x - x_i\right)^2 \right|^{x_{i+3/2}}_{x_{i+1/2}}.
   + \left. \frac{1}{3}  C \left(x - x_i\right)^3 \right|^{x_{i-1/2}}_{x_{i-3/2}} \right]\nonumber  \\
              & = & \: A - B\Delta x + \frac{13}{12} C \Delta x^2 \;,\end{aligned}

We may write these as

.. math::

   \left[ \begin{array}{c}
   f_{i+1} \\ f_{i} \\ f_{i-1}
   \end{array} \right] \: = \:
   \left[ \begin{array}{ccc}
   1 & -1 & \frac{13}{12} \\ 1 & 0 & \frac{1}{12} \\ 1 & 1 & \frac{13}{12}
   \end{array} \right]
   \left[ \begin{array}{ccc}
   A \\ B\Delta x \\ C \Delta x^2
   \end{array} \right] \; .

Inverting this gives expressions for :math:`A`, :math:`B`, and
:math:`C`,

.. math::

   \left[ \begin{array}{c}
   A \\ B\Delta x \\ C \Delta x^2
   \end{array} \right] \: = \:
   \left[ \begin{array}{ccc}
   -\frac{1}{24} & \frac{13}{12} & -\frac{1}{24} \\ -\frac{1}{2} & 0 & \frac{1}{2} \\
   \frac{1}{2} & -1 & \frac{1}{2}
   \end{array} \right]
   \left[ \begin{array}{ccc}
   f_{i+1} \\ f_{i} \\ f_{i-1}
   \end{array} \right] \; .

In two dimensions, we want a second-order interpolation function of the
form

.. math::

   f\left(x,y\right) \: = \: A + B \left(x - x_i\right) + C \left(x - x_i\right)^2
   + D \left(y - y_j\right) + E \left(y - y_j\right)^2 +
   F \left(x - x_i\right)\left(y - y_j\right) \; .

In this case, the cell averaged quantities are given by

.. math::

   f_{i,j}\left(x,y\right) \: \equiv \: \frac{1}{\Delta y}{\Delta x} \int_{x_{i-1/2}}^{x_{i+1/2}}
   dx^\prime
   \int_{y_{j-1/2}}^{x_{j-1/2}}  dy^\prime
   f\left(x^\prime,y^\prime \right) \; .

Integrating the 9 possible cell averages gives, after some algebra,

.. math::

   \left[ \begin{array}{c}
   f_{i-1,j-1} \\ f_{i,j-1} \\ f_{i+1,j-1} \\
   f_{i-1,j} \\ f_{i,j} \\ f_{i+1,j} \\
   f_{i-1,j+1} \\ f_{i,j+1} \\ f_{i+1,j+1}
   \end{array} \right] \: = \:
   \left[ \begin{array}{cccccc}
   1 & -1 & \frac{13}{12} & -1 &  \frac{13}{12} &  1 \\
   1 &  0 & \frac{1}{12}  & -1 &  \frac{13}{12} &  0 \\
   1 &  1 & \frac{13}{12} & -1 &  \frac{13}{12} & -1 \\
   1 & -1 & \frac{13}{12} &  0 &  \frac{1}{12}  &  0 \\
   1 &  0 & \frac{1}{12}  &  0 &  \frac{1}{12}  &  0 \\
   1 &  1 & \frac{13}{12} &  0 &  \frac{1}{12}  &  0 \\
   1 & -1 & \frac{13}{12} &  1 &  \frac{13}{12} & -1 \\
   1 &  0 & \frac{1}{12}  &  1 &  \frac{13}{12} &  0 \\
   1 &  1 & \frac{13}{12} &  1 &  \frac{13}{12} &  1
   \end{array} \right]
   \left[ \begin{array}{c}
   A \\ B\Delta x \\ C \Delta x^2 \\
   D \Delta y \\ E \Delta y^2 \\ F \Delta x \Delta y
   \end{array} \right] \; .

At this point we note that there are more constraints than unknowns, and
we must make a choice of the constraints. We chose to ignore the cross
terms and take only the face-centered cells next to the cell containing
the particle, giving

.. math::

   \left[ \begin{array}{c}
   f_{i,j-1} \\ f_{i-1,j} \\ f_{i,j} \\
   f_{i+1,j} \\ f_{i,j+1}
   \end{array} \right] \: = \:
   \left[ \begin{array}{cccccc}
   1 &  0 & \frac{1}{12}  & -1 &  \frac{13}{12}  \\
   1 & -1 & \frac{13}{12} &  0 &  \frac{1}{12}   \\
   1 &  0 & \frac{1}{12}  &  0 &  \frac{1}{12}   \\
   1 &  1 & \frac{13}{12} &  0 &  \frac{1}{12}   \\
   1 &  0 & \frac{1}{12}  &  1 &  \frac{13}{12}
   \end{array} \right]
   \left[ \begin{array}{c}
   A \\ B\Delta x \\ C \Delta x^2 \\
   D \Delta y \\ E \Delta y^2
   \end{array} \right] \; .

Inverting gives

.. math::

   \left[ \begin{array}{c}
   A \\ B\Delta x \\ C \Delta x^2 \\
   D \Delta y \\ E \Delta y^2
   \end{array} \right]
   \: = \:
   \left[ \begin{array}{cccccc}
   -\frac{1}{24} & -\frac{1}{24}  & \frac{7}{6}  & -\frac{1}{24}  & -\frac{1}{24}  \\
   0             & -\frac{1}{2}   & 0            &  \frac{1}{2}   &  0   \\
   0             & \frac{1}{2}    & -1           &  \frac{1}{2}   &  0   \\
   -\frac{1}{2}  &  0             & 0            &  0             &  \frac{1}{2}   \\
   \frac{1}{2}    &  0             & -1           &  0             &  \frac{1}{2}
   \end{array} \right]
   \left[ \begin{array}{c}
   f_{i,j-1} \\ f_{i-1,j} \\ f_{i,j} \\
   f_{i+1,j} \\ f_{i,j+1}
   \end{array} \right]  \; .

Similarly, in three dimensions, the interpolation function is

.. math::

   f\left(x,y,z\right) \: = \: A + B \left(x - x_i\right) + C \left(x - x_i\right)^2
   + D \left(y - y_j\right) + E \left(y - y_j\right)^2 +
   F \left(z - z_k\right) + G  \left(z - z_k\right)^2 \; .

and we have

.. math::

   \left[ \begin{array}{c}
   A \\ B \Delta x \\ C {\Delta x}^2 \\
   D \Delta y \\ E {\Delta y}^2 \\ F \Delta z \\ G {\Delta z}^2
   \end{array} \right]
   \: = \:
   \left[ \begin{array}{ccccccc}
   -\frac{1}{24} & -\frac{1}{24} & -\frac{1}{24} & \frac{5}{4} & -\frac{1}{24} & -\frac{1}{24} & -\frac{1}{24} \\
   0             & 0             & -\frac{1}{2}  & 0           &  \frac{1}{2}  & 0             &  0   \\
   0             & 0             &  \frac{1}{2}  & -1          &  \frac{1}{2}  & 0             &  0   \\
   0             & -\frac{1}{2}  &  0            &  0          &  0            & \frac{1}{2}   &  0   \\
   0             &  \frac{1}{2}  &  0            & -1          &  0            & \frac{1}{2}   &  0   \\
   -\frac{1}{2}  & 0             &  0            &  0          &  0            & 0             &  \frac{1}{2}   \\
    \frac{1}{2}  & 0             &  0            & -1          &  0            & 0             &  \frac{1}{2}
   \end{array} \right]
   \left[ \begin{array}{c}
   f_{i,j,k-1} \\ f_{i,j-1,k} \\ f_{-i,j,k} \\
   f_{i,j,k} \\ f_{i+1,j,k} \\ f_{i,j+1,k} \\ f_{i,j,k+1}
   \end{array} \right]  \; .

Finally, the above expressions apply only to Cartesian coordinates. In
the case of cylindrical :math:`\left(r,z\right)` coordinates, we have

.. math::

   \begin{aligned}
   f\left(r,z\right) \; = & & \nonumber \\
   & A + B \left(r - r_i\right) + C \left(r - r_i\right)^2
   + D \left(z - z_j\right) & \nonumber \\
   & + E \left(z - z_j\right)^2 +
   F \left(r - r_i\right)\left(z - z_j\right) \; . &\end{aligned}

and

.. math::

   \begin{aligned}
   \left[ \begin{array}{c}
   A \\ B\Delta r \\ C\Delta r^{\frac{2}{6}} \\ D\Delta z  \\ E \Delta z^2
   \end{array} \right] \: = \hspace{-1.0in} & & \nonumber \\
   & \left[ \begin{array}{cccccc}
   -\frac{1}{24} & -\frac{h_1-1}{24h_1}  & \frac{7}{6}      & -\frac{h_1-1}{24h1}  & -\frac{1}{24} \\
   0
   & - \frac{\left(7+6h_1\right)\left(h_1-1\right)}{3h_2}
   & \frac{2h_1}{3h2}
   & \frac{\left(7+6h_1\right)\left(h_1-1\right)}{3h_2}
   & 0  \\
   0
   & \frac{\left(12h_1^2 + 12 h_1 - 1\right)\left(h_1 - 1\right)}{h_1h_2}
   & -2 \frac{12 h_1^2 -13}{h_2}
   & - \frac{\left(12h_1^2 + 12 h_1 - 1\right)\left(h_1 - 1\right)}{h_1h_2}
   & 0 \\
   - \frac{1}{2}
   & 0
   & 0
   & \frac{1}{2}
   & 0 \\
   0
   & \frac{1}{2}
   & - 1
   & \frac{1}{2}
   & 0 \\
   \end{array} \right]
   \left[ \begin{array}{c}
   f_{i,j-1} \\ f_{i-1,j} \\ f_{i,j} \\
   f_{i+1,j} \\ f_{i,j+1}
   \end{array} \right]  \; . &\end{aligned}

.. _`Sec:CIC`:

Cloud in Cell Mapping
~~~~~~~~~~~~~~~~~~~~~

Other interpolation routines can be defined that take into account the
actual quantities defined on the grid. These “mesh-based” algorithms are
represented in |flashx| by the Cloud-in-Cell mapping, where the
interpolation to/from the particles is defined as a simple linear
weighting from nearby grid points. The weights are defined by
considering only the region of one “cell” size around each particle
location; the proportional volume of the particle “cloud” corresponds to
the amount allocated to/from the mesh. The ``CIC`` method can be used
with both types of particles. When using it with active particles the
MapToMesh methods should also be selected. In order to include the CIC
method with passive particles, the ``setup`` command line option is
``-with-unit=Particles/ParticlesMapping/CIC``. Two additional command
line option ``-with-unit=Particles/ParticlesMapping/MapToMesh`` and
``-with-unit=Grid/GridParticles/MapToMesh`` are necessary when using the
active particles. All of these command line options can be replaced by
placing the appropriate ``REQUIRES/REQUESTS`` directives in the
Simulation ``Config`` file.

.. _`Sec:ParticlesUsing`:

Using the Particles Unit
------------------------

The Particles unit encompasses nearly all aspects of Lagrangian
particles. The exceptions are input/output the movement of related data
structures between different blocks as the particles move from one block
to another, and mapping the particle attributes to and from the grid.

Particle types must be specified in the ``Config`` file of the
Simulations unit setup directory for the application, and the syntax is
explained in . At configuration time, the setup script parses the
``PARTICLETYPE`` specifications in the Config files, and generates an
F90 file ``Particles/Particles_specifyMethods``\ ``.F90`` that populates
a data structure ``gr_ptTypeInfo``. This data structure contains
information about the method of initialization and interpolation methods
for mapping the particle attributes to and from the grid for each
included particle type. Different time integration schemes are applied
to active and passive particles. However, in one simulation, all active
particles are integrated using the same scheme, regardless of how many
active types exists. Similarly, only one passive integration scheme is
used. The added complexity of multiple particle types allows different
methods to be used for initialization of particles positions and their
mapping to and from the grid quantities. Because several different
implementations of each type of functionality can co-exist in one
simulation, there are no defaults in the ``Particles`` unit ``Config``
files. These various functionalities are organized into different
subunits; a brief description of each subunit is included below and
further expanded in subsections in this chapter.

-  The ``ParticlesInitialization`` subunit distributes a given set of
   particles through the spatial domain at the simulation startup. Some
   type of spatial initialization is always required; the functionality
   is provided by ``Particles/Particles_initPositions``. The users of
   active particles typically have their own custom initialization. The
   following two implementations of initialization techniques are
   included in the |flashx| distribution (they are more likely to used
   with the passive tracer particles):

   ``Lattice``
      distributes particles regularly along the axes directions
      throughout a subsection of the physical grid.

   ``WithDensity``
      distributes particles randomly, with particle density being
      proportional to the grid gas density.

   Users have two options for implementing custom initialization
   methods. The two files involved in the process are:
   ``Particles/Particles_initPositions`` and pt_initPositions. The
   former does some housekeeping such as allowing for inclusion of one
   of the available methods along with the user specified one, and
   assigning tags at the end. A user wishing to add one custom method
   with no constraints on tags etc is advised to implement a custom
   version of the latter. This approach allows the user to focus the
   implementation on the placement of particles only. Users desirous of
   refining the grid based on particles count during initialiation
   should see the setup ``PoisParticles`` for an example implementation
   of the Particles_initPositions routine. If more than one
   implementation of pt_initPositions is desired in the same simulation
   then it is necessary to implement each one separately with different
   names (as we do for tracer particles: pt_initPositionsLattice and
   pt_initPositionsWithDensity) in their simulation setup directory. In
   addition, a modified copy of Particles_initPostions, which calls
   these new routines in the loop over types, must also be placed in the
   same directory.

-  The ``ParticlesMain`` subunit contains the various time-integration
   options for both active and passive particles. A detailed overview of
   the different schemes is given in .

-  The ``ParticlesMapping`` subunit controls the mapping of particle
   properties to and from the grid. |flashx| currently supplies the
   following mapping schemes:

   ``Cloud-in-cell``
      (``ParticlesMapping/meshWeighting/CIC``), which weights values at
      nearby grid cells; and

   ``Quadratic``
      (``ParticlesMapping/Quadratic``), which performs quadratic
      interpolation.

Some form of mapping must always be included when running a simulation
with particles. As mentioned in the quadratic mapping scheme is only
available to map *from* the grid quantities to the corresponding
particle attributes. Since active particles require the same mapping
scheme to be used in mapping to and from the mesh, they cannot use the
quadratic mapping scheme as currently implemented in |flashx|. The CIC
scheme may be used by both the active and passive particles.

After particles are moved during time integration or by forces, they may
end up on other blocks within or without the current processor. The
redistribution of particles among processors is handled by the
``GridParticles`` subunit, as the algorithms required vary considerably
between the grid implementations. The boundary conditions are also
implemented by the GridParticles unit. See for more details of these
redistribution algorithms. The user should include the option
``-with-unit=Grid/GridParticles`` on the setup line, or
``REQUIRES Grid/GridParticles`` in the Config file.

In addition, the input-output routines for the Particles unit are
contained in a subunit ``IOParticles``. Particles are written to the
main checkpoint files. If the user desires, a separate output file can
be created which contains only the particle information. See below as
well as for more details. The user should include the option
``-with-unit=IO/IOParticles`` on the setup line, or
``REQUIRES IO/IOParticles`` in the Config file.

In |flashx|, the initial particle positions can be used to construct an
appropriately refined grid, i.e. more refined in places where there is a
clustering of particles. To use this feature the ``flash.par`` file must
include: ``refine_on_particle_count=.true.`` and
``max_particles_per_blk=[some value]``. Please be aware that |flashx|
will abort if the criterion is too demanding. To overcome the abort,
specify a less demanding criterion, or increase the value of
``lrefine_max``.

.. _`Sec:Particles Runtime Parameters`:

Particles Runtime Parameters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are several general runtime parameters applicable to the
``Particles`` unit, which affect every implementation. The variable
``Particles/useParticles`` obviously must be set equal to ``.true.`` to
utilize the Particles unit. The time stepping is controlled with
``Particles/pt_dtFactor``; a value less than one ensures that particles
will not step farther than one entire cell in any given time interval.
The ``Lattice`` initialization routines have additional parameters. The
number of evenly spaced particles is controlled in each direction by
``Particles/pt_numX`` and similar variables in :math:`Y` and :math:`Z`.
The physical range of initialization is controlled by
``Particles/pt_initialXMin`` and the like. Finally, note that the output
of particle properties to special particle files is controlled by
runtime parameters found in the ``IO`` unit. See for more details.

.. _`Sec:Particle Properties`:

Particle Attributes
~~~~~~~~~~~~~~~~~~~

By default, particles are defined to have eight real properties or
attributes: 3 positions in x,y,z; 3 velocities in x,y,z; the current
block identification number; and a tag which uniquely identifies the
particle. Additional properties can be defined for each particle. For
example, active particles usually have the additional properties of mass
and acceleration (needed for the integration routines, see Table ).
Depending upon the simulation, the user can define particle properties
in a manner similar to that used for mesh-based solution variables. To
define a particle attribute, add to a ``Config`` file a line of the form

   ``PARTICLEPROP`` *property-name*

For attributes that are meant to merely sample and record the state of
certain mesh variables along trajectories, |flashx| can automatically
invoke interpolation (or, in general, some map method) to generate
attribute values from the appropriate grid quantities. (For passive
tracer particles, these are typically the only attributes beyond the
default set of eight mentioned above.) The routine
``Particles/Particles_updateAttributes`` is invoked by |flashx| at
appropriate times to effect this mapping, namely before writing particle
data to checkpoint and particle plot files. To direct the default
implementation of ``Particles_updateAttributes`` to act as desired for
tracer attributes, the user must define the association of the particle
attribute with the appropriate mesh variable by including the following
line in the ``Config`` file:

   ``PARTICLEMAP TO`` *property-name* ``FROM VARIABLE`` *variable-name*

These particle attributes are carried along in the simulation and output
in the checkpoint files. At runtime, the user can specify the attributes
to output through runtime parameters ``Particles/particle_attribute_1``,
``Particles/particle_attribute_2``, etc. These specified attributes are
collected in an array by the ``Particles/Particles_init`` routine. This
array in turn is used by ``Particles/Particles_updateAttributes`` to
calculate the values of the specified attributes from the corresponding
mesh quantities before they are output.

.. _`Sec:Particles IO`:

Particle I/O
~~~~~~~~~~~~

Particle data are written to and read from checkpoint files by the I/O
modules (). For more information on the format of particle data written
to output files, see and .

Particle data can also be written out to the ``flash.dat`` file. The
user should include a local copy of ``IO/IO_writeIntegralQuantities`` in
their Simulation directory. The ``Orbit`` test problem supplies an
example ``IO_writeIntegralQuantities`` routine that is useful for
writing individual particle trajectories to disk at every timestep.

There is also a utility routine ``Particles/Particles_dump`` which can
be used to dump particle output to a plain text file. An example of
usage can be found in ``Particles/Particles_unitTest``. Output from this
routine can be read using the fidlr routine ``particles_dump.pro``.

.. [1]
   Even though it is possible to do so, see
   ``Particles/Particles_computeDt``, one does not in general wish to
   let particles integration dictate the time step of the simulation.

.. [2]
   They can be considered “predicted” positions and velocities.
