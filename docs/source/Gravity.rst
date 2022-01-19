.. _`chp:Gravity`:

Gravity Unit
============

Introduction
------------

The Gravity unit supplied with computes gravitational source terms for
the code. These source terms can take the form of the gravitational
potential :math:`\phi({\bf x})` or the gravitational acceleration
:math:`{\bf g}({\bf x})`,

.. math:: {\bf g}({\bf x}) = -\nabla \phi({\bf x})\ .

The gravitational field can be externally imposed or self-consistently
computed from the gas density via the Poisson equation,

.. math::

   \label{Eqn:Poisson}
   \nabla^2\phi({\bf x}) = 4\pi G \rho({\bf x})\ ,

where :math:`G` is Newton’s gravitational constant. In the latter case,
either periodic or isolated boundary conditions can be applied.

.. _`Sec:GravityExternal`:

Externally Applied Fields
-------------------------

The Flash-X distribution includes three externally applied gravitational
fields, along with a placeholder module for you to create your own. Each
provides the acceleration vector :math:`{\bf g}({\bf x})` directly,
without using the gravitational potential :math:`\phi({\bf x})` (with
the exception of , see below).

When building an application that uses an external, time-independent
Gravity implementation, no additional storage in for holding
gravitational potential or accelerations is needed or defined.

Constant Gravitational Field
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This implementation creates a spatially and temporally constant field
parallel to one of the coordinate axes. The magnitude and direction of
the field can be set at runtime. This unit is called .

Plane-parallel Gravitational field
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This version implements a time-constant gravitational field that is
parallel to one of the coordinate axes and falls off with the square of
the distance from a fixed location. The field is assumed to be generated
by a point mass or by a spherically symmetric mass distribution. A
finite softening length may optionally be applied.

This type of gravitational field is useful when the computational domain
is large enough in the direction radial to the field source that the
field is not approximately constant, but the domain’s dimension
perpendicular to the radial direction is small compared to the distance
to the source. In this case the angular variation of the field direction
may be ignored. The field is cheaper to compute than the field described
below, since no fractional powers of the distance are required. The
acceleration vector is parallel to one of the coordinate axes, and its
magnitude drops off with distance along that axis as the inverse
distance squared. Its magnitude and direction are independent of the
other two coordinates.

Gravitational Field of a Point Mass
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This implementation describes the gravitational field due to a point
mass at a fixed location. A finite softening length may optionally be
applied. The acceleration falls off with the square of the distance from
a given point. The acceleration vector is everywhere directed toward
this point.

User-Defined Gravitational Field
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The implementation is a placeholder module for the user to create their
own external gravitational field. All of the subroutines in this module
are stubs, and the user may copy these stubs to their setup directory to
write their own implementation, either by specifying the gravitational
acceleration directly or by specifying the gravitational potential and
taking its gradient. If your user-defined gravitational field is
time-varying, you may also want to set in your setup’s Config file.

.. _`Sec:GravitySelfgravity`:

Self-gravity
------------

The self-consistent gravity algorithm supplied with Flash-X computes the
Newtonian gravitational field produced by the matter. The produced
potential function satisfies Poisson’s equation
`[Eqn:Poisson] <#Eqn:Poisson>`__. This unit’s implementation can also
return the acceleration field :math:`{\bf g}({\bf x})` computed by
finite-differencing the potential using the expressions

.. math::

   \begin{aligned}
   \nonumber
   g_{x;ijk} &= {1\over2\Delta x}\left(\phi_{i-1,j,k} - \phi_{i+1,j,k}\right) +
     {\cal O}(\Delta x^2) \\
   g_{y;ijk} &= {1\over2\Delta y}\left(\phi_{i,j-1,k} - \phi_{i,j+1,k}\right) +
     {\cal O}(\Delta y^2) \\
   \nonumber
   g_{z;ijk} &= {1\over2\Delta z}\left(\phi_{i,j,k-1} - \phi_{i,j,k+1}\right) +
     {\cal O}(\Delta z^2) \ .\end{aligned}

In order to preserve the second-order accuracy of these expressions at
jumps in grid refinement, it is important to use quadratic interpolants
when filling guard cells at such locations. Otherwise, the truncation
error of the interpolants will produce unphysical forces at these block
boundaries.

Two algorithms are available for solving the Poisson equations: and .
The initialization routines for these algorithms are contained in the
unit, but the actual implementations are contained below the unit due to
code architecture constraints.

The multipole-based solver described in for self gravity is appropriate
for spherical or nearly-spherical mass distributions with isolated
boundary conditions. For non-spherical mass distributions higher order
moments of the solver must be used. Note that storage and CPU costs
scale roughly as the square of number of moments used, so it is best to
use this solver only for nearly spherical matter distributions.

The multigrid solver described in is appropriate for general mass
distributions and can solve problems with more general boundary
conditions.

The tree solver described in
`[Sec:GridSolversBHTree] <#Sec:GridSolversBHTree>`__ is appropriate for
general mass distributions and can solve problems with both isolated and
periodic boundary conditions set independently in individual directions.

Coupling Gravity with Hydrodynamics
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The gravitational field couples to the Euler equations only through the
momentum and energy equations. If we define the total energy density as

.. math:: \rho E \equiv {1\over 2}\rho v^2 + \rho\epsilon\ ,

where :math:`\epsilon` is the specific internal energy, then the
gravitational source terms for the momentum and energy equations are
:math:`\rho{\bf g}` and :math:`\rho{\bf v}\cdot{\bf g}`, respectively.
Because of the variety of ways in which different hydrodynamics schemes
treat these source terms, the gravity module only supplies the potential
:math:`\phi` and acceleration :math:`{\bf g}`, leaving the
implementation of the fluid coupling to the hydrodynamics module.
Finite-difference and finite-volume hydrodynamic schemes apply the
source terms in their advection steps, sometimes at multiple
intermediate timesteps and sometimes using staggered meshes for vector
quantities like :math:`{\bf v}` and :math:`{\bf g}`.

For example, the PPM algorithm supplied with Flash-X uses the following
update steps to obtain the momentum and energy in cell :math:`i` at
timestep :math:`n+1`

.. math::

   \begin{aligned}
   \nonumber
   (\rho v)_i^{n+1} & =  (\rho v)_i^n + {\Delta t\over 2} g_i^{n+1}
     \left(\rho_i^n + \rho_i^{n+1}\right)\\
   (\rho E)_i^{n+1} & =  (\rho E)_i^n + {\Delta t\over 4} g_i^{n+1}
     \left(\rho_i^n + \rho_i^{n+1}\right)\left(v_i^n + v_i^{n+1}\right)\ .\end{aligned}

Here :math:`g_i^{n+1}` is obtained by extrapolation from
:math:`\phi_i^{n-1}` and :math:`\phi_i^n`. The :math:`\code{Poisson}`
gravity implementation supplies a mesh variable to contain the potential
from the previous timestep; future releases of Flash-X may permit the
storage of several time levels of this quantity for hydrodynamics
algorithms that require more steps. Currently, :math:`{\bf g}` is
computed at cell centers.

Note that finite-volume schemes do not retain explicit conservation of
momentum and energy when gravity source terms are added. Godunov schemes
such as PPM, require an additional step in order to preserve
second-order time accuracy. The gravitational acceleration component
:math:`g_i` is fitted by interpolants along with the other state
variables, and these interpolants are used to construct
characteristic-averaged values of :math:`{\bf g}` in each cell. The
velocity states :math:`v_{L,i+1/2}` and :math:`v_{R,i+1/2}`, which are
used as inputs to the Riemann problem solver, are then corrected to
account for the acceleration using the following expressions

.. math::

   \begin{aligned}
   \nonumber
   v_{L,i+1/2} &\rightarrow& v_{L,i+1/2} + {\Delta t\over
   4}\left(g^+_{L,i+1/2} +
     g^-_{L,i+1/2}\right)\\
   v_{R,i+1/2} &\rightarrow& v_{R,i+1/2} + {\Delta t\over
   4}\left(g^+_{R,i+1/2} +
     g^-_{R,i+1/2}\right)~.\end{aligned}

Here :math:`g^\pm_{X,i+1/2}` is the acceleration averaged using the
interpolant on the :math:`X` side of the interface (:math:`X=L,R`) for
:math:`v\pm c` characteristics, which bring material to the interface
between cells :math:`i` and :math:`i+1` during the timestep.

Tree Gravity
~~~~~~~~~~~~

The implementation of the gravity unit in is meant to be used together
with the tree solver implementation ``Grid/GridSolvers/BHTree/Wunsch``.
It either calculates the gravitational potential field which is
subsequently differentiated in subroutine to obtain the gravitational
acceleration, or it calculates the gravitational acceleration directly.
The latter approach is more accurate, because the error due to numerical
differentiation is avoided, however, it consumes more memory for storing
three components of the gravitational acceleration. The direct
acceleration calculation can be switched on by specifying as a command
line argument of the setup script.

The gravity unit provides subroutines for building and walking the tree
called by the tree solver. In this version, only monopole moments (node
masses) are used for the potential/acceleration calculation. It also
defines new multipole acceptance criteria (MACs) that estimate the error
in gravitational acceleration of a contribution of a single node to the
potential (hereafter partial error) much better than purely geometrical
MAC defined in the tree solver. They are: (1) the approximate partial
error (APE), and (2) the maximum partial error (MPE). The first one is
based on an assumption that the partial error is proportional to the
multipole moment of the node. The node is accepted for calculation of
the potential if

.. math:: D^{m+2} > \frac{GMS_\mathrm{node}^m}{\Delta a_\mathrm{p,APE}}

where :math:`D` is distance between the *point-of-calculation* and the
node, :math:`M` is the node mass, :math:`S_\mathrm{node}` is the node
size, :math:`m` is a degree of the multipole approximation and
:math:`\Delta a_\mathrm{p,APE}` is the requested maximum error in
acceleration (controlled by runtime parameter ). Since only monopole
moments are used for the potential calculation, the most reasonable
choice of :math:`m` seems to be :math:`m=2`. This MAC is similar to the
one used in Gadget2 (see Springel, 2005, MNRAS, 364, 1105).

The second MAC (maximum partial error, MPE) calculates the error in
acceleration of a single node contribution
:math:`\Delta a_\mathrm{p,MPE}` according to formula 9 from
Salmon&Warren (1994; see this paper for details):

.. math::

   \Delta a_\mathrm{p,MPE} \le \frac{1}{D^2}\frac{1}{(1-S_\mathrm{node}/D)^2}\left(
   \frac{3\lceil B_2\rceil}{D^2} - \frac{2\lfloor B_3 \rfloor}{D^3}\right)

where :math:`B_n = \Sigma_i m_i |\mathbf{r}_i-\mathbf{r}_0|^n` where
:math:`m_i` and :math:`\mathbf{r}_i` are masses and positions of
individual grid cells within the node and :math:`\mathbf{r}_0` is the
node mass center position. Moment :math:`B_2` can be easily determined
during the tree build, moment :math:`B_3` can be estimated as
:math:`B_3^2 \ge
B_2^3/M`. The maximum allowed partial error in gravitational
acceleration is controlled by runtime parameters and (see
`1.4.1 <#Sec:GravityBHTreeUsing>`__).

During the tree walk, subroutine adds contributions of tree nodes to the
gravitational potential or acceleration fields. In case of the
potential, the contribution is

.. math:: \Phi = -\frac{GM}{|\vec{r}|}

if ``isolated`` boundary conditions are used, or

.. math:: \Phi = -GMf_\mathrm{EF,\Phi}(\vec{r})

if periodic ``periodic`` or ``mixed`` boundary conditions are used. In
case of the acceleration, the contributions are

.. math:: \vec{a}_g = \frac{GM\vec{r}}{|\vec{r}|^3}

for ``isolated`` boundary conditions, or

.. math:: \vec{a}_g = GMf_\mathrm{EF,a}(\vec{r})

for ``periodic`` or ``mixed`` boundary conditions. In In the above
formulae, :math:`G` is the constant of gravity, :math:`M` is the node
mass, :math:`\vec{r}` is the position vector between
*point-of-calculation* and the node mass center and
:math:`f_\mathrm{EF,\Phi}` and :math:`f_\mathrm{EF,a}` are the Ewald
fields for the potential and the acceleration (see below).

Boundary conditions can be isolated or periodic, set independently for
each direction. If they are periodic at least in one direction, the
Ewald method is used for the potential calculation (Ewald, P. P., 1921,
Ann. Phys. 64, 253). The original Ewald method is an efficient method
for computing gravitational field for problems with periodic boundary
conditions in three directions. Ewald speeded up evaluation of the
gravitational potential by splitting it into two parts,
:math:`Gm/r=Gm \, \erf(\alpha r)/r + Gm \, \erfc(\alpha r)/r`
(:math:`\alpha` is an arbitrary constant) and then by applying Poisson
summation formula on erfc terms, gravitational field at position
:math:`\vec{r}` can be written in the form

.. math::

   \phi (\vec{r})  =  -G \sum_{a=1}^N m_a \left( \sum_{i_1,i_2,i_3} A_S(\vec{r},\vec{r}_a,\vec{l}_{i_1,i_2,i_3}) + 
   A_L(\vec{r},\vec{r}_a,\vec{l}_{i_1,i_2,i_3})  \right)
   = -G \sum_{a=1}^N m_a f_\mathrm{EF,\Phi}(\vec{r}_a - \vec{r}) \ ,
   \label{ewald_sum}

the first sum runs over whole computational domain, where at position
:math:`\vec{r}_a` is mass :math:`m_a`. Second sum runs over all
neigbouring computational domains, which are at positions
:math:`\vec{l}_{i_1,i_2,i_3}` and
:math:`A_S(\vec{r},\vec{r}_a,\vec{l}_{i_1,i_2,i_3})` and
:math:`A_L(\vec{r},\vec{r}_a,\vec{l}_{i_1,i_2,i_3})` are short and
long–range contributions, respectively. It is sufficient to take into
account only few terms in eq. `[ewald_sum] <#ewald_sum>`__. The Ewald
field for the acceleration, :math:`f_\mathrm{EF,a}`, is obtained using a
similar decomposition. We modified Ewald method for problems with
periodic boundary conditions in two directions and isolated boundary
conditions in the third direction and for problems with periodic
boundary conditions in one direction and isolated in two directions.

The gravity unit allows also to use a static external gravitational
field read from file. In this version, the field can be either
spherically symmetric or planar being only a function of the
z-coordinate. The external field file is a text file containing three
columns of numbers representing the coordinate, the potential and the
acceleration. The coordinate is the radial distance or z-distance from
the center of the external field given by runtime parameters. The
external field if mapped to a grid using a linear interpolation each
time the gravitational acceleration is calculated (in subroutine ).

.. _`Sec:Gravity Usage`:

Usage
-----

To include the effects of gravity in your Flash-X executable, include
the option

   -with-unit=physics/Gravity

on your command line when you configure the code with . The default
implementation is , which can be overridden by including the entire path
to the specific implementation in the command line or file. The other
available implementations are , and . The unit provides accessor
functions to get gravitational acceleration and potential. However, none
of the external field implementations of Section  explicitly compute the
potential, hence they inherit the null implementation from the API for
accessing potential. The gravitation acceleration can be obtained either
on the whole domain, a single block or a single row at a time.

When building an application that solves the Possion equation for the
gravitational potential, additional storage is needed in for holding the
last, as well as (usually) the previous, gravitational potential field;
and, depending on the Poisson solver used, additional variables may be
needed. The variables and , and others as needed, will be automatically
defined in in those cases. See for more information.

.. _`Sec:GravityBHTreeUsing`:

Tree Gravity Unit Usage
~~~~~~~~~~~~~~~~~~~~~~~

Calculation of gravitational potential can be enabled by compiling in
this unit and setting the runtime parameter true. The constant of
gravity can be set independently by runtime parameter ; if it is not
positive, the constant ``Newton`` from the Flash-X PhysicalConstants
database is used. If parameters or are set, the gravity unit MAC is used
and it can be chosen by setting to either ``ApproxPartialErr`` or
``MaxPartialErr``. If the first one is used, the order of the multipole
approximation is given by .

The maximum allowed partial error in gravitational acceleration is set
with the runtime parameter . It has either the meaning of an error in
absolute acceleration or in relative acceleration normalized by the
acceleration from the previous time-step. The latter is used if is set
to True, and in this case the first call of the tree solver calculates
the potential using purely geometrical MAC (because the acceleration
from the previous time-step does not exist).

Boundary conditions are set by the runtime parameter and they can be
``isolated``, ``periodic`` or ``mixed``. In the case of mixed boundary
conditions, runtime parameters , and specify along which coordinate
boundary conditions are periodic and isolated (possible values are
``periodic`` or ``isolated``). Arbitrary combination of these values is
permitted, thus suitable for problems with planar resp. linear symmetry.
It should work for computational domain with arbitrary dimensions. The
highest accuracy is reached with blocks of cubic physical dimensions.

If runtime parameter is ``periodic`` or ``mixed``, then the Ewald field
for appropriate symmetry is calculated at the beginning of the
simulation. Parameter controls the range of indices :math:`i_1,i_2,i_3`
in (eq. `[ewald_sum] <#ewald_sum>`__). There are two implementations of
the Ewald method: the new one (default) requires less memory and it
should be faster and of comparable accuracy as the old one. The default
implementation computes Ewald field minus the singular :math:`1/r` term
and its partial derivatives on a single cubic grid, and the Ewald field
is then approximated by the first order Taylor formula. Parameter
controls number of grid points in the :math:`x` direction in the case of
``periodic`` or in periodic direction(s) in the case of ``mixed``
boundary conditions. Since an elongated computational domain is often
desired when is ``mixed``, the cubic grid would lead to a huge field of
data. In this case, the amount of necessary grid points is reduced by
using an analytical estimate to the Ewald field sufficiently far away of
the symmetry plane or axis.

The old implementation (from Flash4.2) is still present and is enabled
by adding on the setup command line. The Ewald field is then stored in a
nested set of grids, the first of them corresponds in size to full
computational domain, and each following grid is half the size (in each
direction) of the previous grid. Number of nested grids is controlled by
runtime parameter . If is too low to cover origin (where is the Ewald
field discontinuous), then the run is terminated. Each grid is composed
of :math:`\times` :math:`\times` points. When evaluation of the Ewald
Field at particular point is needed at any time during a run, the field
value is found by interpolation in a suitable level of the grid. Linear
or semi-quadratic interpolation can be chosen by runtime parameter
(option corresponds to linear interpolation). Semi-quadratic
interpolation is recommended only in the case when there are periodic
boundary conditions in two directions.

The external gravitational field can be switched on by setting true. The
parameter gives the name of the file with the external potential and
specifies the field symmetry: ``spherical`` for the spherical symmetry
and ``planez`` for the planar symmetry with field being a function of
the z-coordinate. Parameters , and specify the position (in the
simulation coordinate system) of the external field origin (the point
where the radial or z-coordinate is zero).

.. container:: center

   .. table:: Tree gravity unit parameters controlling the accuracy of
   calculation.

      +----------+---------+--------------------+----------------------+
      | Variable | Type    | Default            | Description          |
      +==========+=========+====================+======================+
      |          | real    | -1.0               | constant of gravity; |
      |          |         |                    | if :math:`<` 0, it   |
      |          |         |                    | is obtained          |
      +----------+---------+--------------------+----------------------+
      |          |         |                    | from internal        |
      |          |         |                    | physical constants   |
      |          |         |                    | database             |
      +----------+---------+--------------------+----------------------+
      |          | string  | "ApproxPartialErr" | MAC, other option:   |
      |          |         |                    | "MaxPartialErr"      |
      +----------+---------+--------------------+----------------------+
      |          | integer | 2                  | degree of multipole  |
      |          |         |                    | in error estimate in |
      |          |         |                    | APE MAC              |
      +----------+---------+--------------------+----------------------+
      |          | logical | .false.            | if .true.,           |
      |          |         |                    | grv_bhAccErr has     |
      |          |         |                    | meaning of           |
      +----------+---------+--------------------+----------------------+
      |          |         |                    | relative error,      |
      |          |         |                    | otherwise absolute   |
      +----------+---------+--------------------+----------------------+
      |          | real    | 0.1                | maximum allowed      |
      |          |         |                    | error in             |
      |          |         |                    | gravitational        |
      +----------+---------+--------------------+----------------------+
      |          |         |                    | acceleration         |
      +----------+---------+--------------------+----------------------+

.. container:: center

   .. table:: Tree gravity unit parameters controlling the boundary
   conditions.

      +----------+---------+-------------------+----------------------+
      | Variable | Type    | Default           | Description          |
      +==========+=========+===================+======================+
      |          | string  | "isolated"        | or "periodic" or     |
      |          |         |                   | "mixed"              |
      +----------+---------+-------------------+----------------------+
      |          | string  | "isolated"        | or "periodic"        |
      +----------+---------+-------------------+----------------------+
      |          | string  | "isolated"        | or "periodic"        |
      +----------+---------+-------------------+----------------------+
      |          | string  | "isolated"        | or "periodic"        |
      +----------+---------+-------------------+----------------------+
      |          | boolean | true              | whether Ewald field  |
      |          |         |                   | should be            |
      |          |         |                   | regenerated          |
      +----------+---------+-------------------+----------------------+
      |          | integer | :math:`10`        | number of terms in   |
      |          |         |                   | the Ewald series     |
      +----------+---------+-------------------+----------------------+
      |          | integer | 32                | number of points+1   |
      |          |         |                   | of the Taylor        |
      |          |         |                   | expansion            |
      +----------+---------+-------------------+----------------------+
      |          | string  | "ewald_coeffs"    | file with            |
      |          |         |                   | coefficients of the  |
      |          |         |                   | Ewald field Taylor   |
      |          |         |                   | expansion            |
      +----------+---------+-------------------+----------------------+
      |          | integer | :math:`32`        | size of the Ewald    |
      |          |         |                   | field grid in        |
      |          |         |                   | x-direction          |
      +----------+---------+-------------------+----------------------+
      |          | integer | :math:`32`        | size of the Ewald    |
      |          |         |                   | field grid in        |
      |          |         |                   | y-direction          |
      +----------+---------+-------------------+----------------------+
      |          | integer | :math:`32`        | size of the Ewald    |
      |          |         |                   | field grid in        |
      |          |         |                   | z-direction          |
      +----------+---------+-------------------+----------------------+
      |          | integer | -1                | number of refinement |
      |          |         |                   | levels (nested       |
      |          |         |                   | grids) for the Ewald |
      +----------+---------+-------------------+----------------------+
      |          |         |                   | field; if :math:`<`  |
      |          |         |                   | 0, determined        |
      |          |         |                   | automatically        |
      +----------+---------+-------------------+----------------------+
      |          | logical | .true.            | if .false.,          |
      |          |         |                   | semi-quadratic       |
      |          |         |                   | interpolation is     |
      |          |         |                   | used for             |
      +----------+---------+-------------------+----------------------+
      |          |         |                   | interpolation in the |
      |          |         |                   | Ewald field          |
      +----------+---------+-------------------+----------------------+
      |          | string  | "ewald_field_acc" | file with the Ewald  |
      |          |         |                   | field for            |
      |          |         |                   | acceleration         |
      +----------+---------+-------------------+----------------------+
      |          | string  | "ewald_field_pot" | file with            |
      |          |         |                   | coefficients of the  |
      |          |         |                   | Ewald field for      |
      |          |         |                   | potential            |
      +----------+---------+-------------------+----------------------+

Tree gravity unit parameters controlling the external gravitational
field.

.. container:: center

   +----------+---------+----------------------+----------------------+
   | Variable | Type    | Default              | Description          |
   +==========+=========+======================+======================+
   |          | logical | .false.              | whether to use       |
   |          |         |                      | external field       |
   +----------+---------+----------------------+----------------------+
   |          | logical | .true.               | whether to use       |
   |          |         |                      | gravitational field  |
   |          |         |                      | calculated by the    |
   +----------+---------+----------------------+----------------------+
   |          |         |                      | tree solver          |
   +----------+---------+----------------------+----------------------+
   |          | string  | "ext                 | file containing the  |
   |          |         | ernal_potential.dat" | external             |
   |          |         |                      | gravitational field  |
   +----------+---------+----------------------+----------------------+
   |          | string  | "planez"             | type of the external |
   |          |         |                      | field: planar or     |
   |          |         |                      | spherical symmetry   |
   +----------+---------+----------------------+----------------------+
   |          | real    | 0.0                  | x-coordinate of the  |
   |          |         |                      | center of the        |
   |          |         |                      | external field       |
   +----------+---------+----------------------+----------------------+
   |          | real    | 0.0                  | y-coordinate of the  |
   |          |         |                      | center of the        |
   |          |         |                      | external field       |
   +----------+---------+----------------------+----------------------+
   |          | real    | 0.0                  | z-coordinate of the  |
   |          |         |                      | center of the        |
   |          |         |                      | external field       |
   +----------+---------+----------------------+----------------------+

.. _`Sec:GravityUnitTests`:

Unit Tests
----------

There are two unit tests for the gravity unit. is essentially the
Maclaurin spheroid problem described in . Because an analytical solution
exists, the accuracy of the gravitational solver can be quantified. The
second test, is a modification of to test the mapping of particles in .
Some of the mesh density is redistributed onto particles, and the
particles are then mapped back to the mesh, using the analytical
solution to verify completeness. This test is similar to the simulation
discussed in . is based on the Huang-Greengard Poisson gravity test
described in .
