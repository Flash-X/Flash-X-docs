.. include:: defs.h

.. _`Chp:EOS Unit`:

Equation of State Unit
======================

.. _`Sec:Eos Intro`:

Introduction
------------

The ``Eos`` unit implements the equation of state needed by the
hydrodynamics and nuclear burning solvers. The function
``physics/Eos/Eos`` provides the interface for operating on a
single data point. A macro (``eos_args``) is defined to provide a list
of arguments. It is highly recommended using this macro in both, the
the declaration section of the caller and in the call to the function
itself. This recommendation stems from the possibility of the
interface needing to change in future if other variants of Eos are
introduced. This function can be
used to find the thermodynamic quantities either from the density,
temperature, and composition or from the density, internal energy, and
composition.

For user’s convenience, two wrapper functions are
provided. (``physics/Eos/Eos_vector``) takes in a two dimensional
array of data points where the first dimension is the length of the
vector, and the second dimension is for the physical
quantities, both input and output. The index of each physical quantity for the second
dimension is included in ``Eos.h``. In |flash| all interfaces of Eos
used a single-dimensional vector and one had to find the offset for
the desired physical quantity. Use of two-dimensional array has
obviated the need for this additional step. The second wrapper
function is  ``physics/Eos/Eos_multiDim``  which takes a pointer to a multidimensional array of data,
computes kinetic energy if velocities are included in the data, and
converts the data to the two-dimension vector format. It then calls
the ``physics/Eos/Eos_vector``  function, and upon return from
the function, it translates the returned data back to the same
mutidimensional array.. 

Two implementations of the (``Eos``) unit are available in the
|flashx| distribution, and a third can be imported if desired (see the
Readme at the top level of the source for instructions to import).  The
two that are included in the distribution are: 
``Gamma`` which implements a perfect-gas equation of
state; and ``Helmholtz`` which uses a fast Helmholtz free-energy table
interpolation to handle degenerate/relativistic electrons/positrons
and includes radiation pressure and ions (via the perfect gas
approximation). For ``WeakLib``, the imported implementation, the
necessary functions to make the calls compatible with |flashx|'s API
are included in the distribution. A hybrid implementation that can
call either the ``Helmholtz`` or the ``WeakLib`` version depending on
the state of the matter is also included in the distribution. 

As described in previous sections, |flashx| evolves the Euler equations
for compressible, inviscid flow. This system of equations must be closed
by an additional equation that provides a relation between the
thermodynamic quantities of the gas. This relationship is known as the
equation of state for the material, and its structure and properties
depend on the composition of the gas.

It is common to call an equation of state (henceforth EOS) routine more
than :math:`10^9` times during a two-dimensional simulation and more
than :math:`10^{11}` times during the course of a three-dimensional
simulation of stellar phenomena. Thus, it is very desirable to have an
EOS that is as efficient as possible, yet accurately represents the
relevant physics. While |flashx| is capable of using any general equation
of state, we discuss here the two primary versions that are supplied: an ideal-gas or gamma-law EOS, and a tabular Helmholtz free energy EOS
appropriate for stellar interiors. The gamma-law EOS consists of
simple analytic expressions that make for a very fast EOS routine 
in the case of a single gas. The Helmholtz EOS
includes much more physics and relies on a table look-up scheme for
performance.

.. _`Sec:Eos Gammas`:

Gamma Law
------------------------

|flashx| uses the method of Colella & Glaz (1985) to handle general
equations of state. General equations of state contain 4 adiabatic
indices (Chandrasekhar 1939), but the method of Colella & Glaz
parameterizes the EOS and requires only two of the adiabatic indices The
first is necessary to calculate the adiabatic sound speed and is given
by

.. math:: \gamma_1 = \frac{\rho}{P}\frac{\partial P}{\partial \rho} \; .

The second relates the pressure to the energy and is given by

.. math:: \label{Eqn:game}\gamma_4 = 1 + \frac{P}{\rho\epsilon} \; .

These two adiabatic indices are stored as the mesh-based variables
``GAMC_VAR`` and ``GAME_VAR``. All EOS routines must return
:math:`\gamma_1`, and :math:`\gamma_4` is calculated from
`[Eqn:game] <#Eqn:game>`__.

The gamma-law EOS models a simple ideal gas with a constant adiabatic
index :math:`\gamma`. Here we have dropped the subscript on
:math:`\gamma`, because for an ideal gas, all adiabatic indices are
equal. The relationship between pressure :math:`P`, density
:math:`\rho`, and specific internal energy :math:`\epsilon` is

.. math::

   \label {Eqn:eos2b}
   %{P = {N_a \ k \over \bar{A}} \rho T}
   P = \left(\gamma - 1\right)\rho\epsilon~.
   %\epsilon = {1 \over \gamma - 1} \ {P \over \rho}

We also have an expression relating pressure to the temperature
:math:`T`

.. math::

   \label {Eqn:eos2a}
   P = \frac{N_a k}{\bar{A}} \rho T ~,

where :math:`N_a` is the Avogadro number, :math:`k` is the Boltzmann
constant, and :math:`\bar{A}` is the average atomic mass, defined as

.. math:: \frac{1}{\bar{A}} = \sum_{i}\frac{X_{i}}{A_{i}}~,

where :math:`X_i` is the mass fraction of the :math:`i`\ th element.
Equating these expressions for pressure yields an expression for the
specific internal energy as a function of temperature

.. math:: \epsilon = \frac{1}{\gamma - 1} \frac{N_a k} {\bar{A}} T~.


We note that the analytic expressions apply to both the forward
(internal energy as a function of density, temperature, and composition)
and backward (temperature as a function of density, internal energy and
composition) relations. Because the backward relation requires no
iteration in order to obtain the temperature, this EOS is quite
inexpensive to evaluate. Despite its fast performance, use of the
gamma-law EOS is limited, due to its restricted range of applicability
for astrophysical problems.

In the current distribution of |flashx| we have excluded the
multigamma version because of lack of use-cases. It will be included
in future distributions.

.. _`Sec:Eos Helmholtz`:

Helmholtz
---------

The Helmholtz EOS provided with the |flashx| distribution contains more
physics and is appropriate for addressing astrophysical phenomena in
which electrons and positrons may be relativistic and/or degenerate and
in which radiation may significantly contribute to the thermodynamic
state. Full details of the Helmholtz equation of state are provided in
Timmes & Swesty (1999). This EOS includes contributions from radiation,
completely ionized nuclei, and degenerate/relativistic electrons and
positrons. The pressure and internal energy are calculated as the sum
over the components

.. math::

   \label {Eqn:eos3a}
   P_{\rm tot} = P_{\rm rad} + P_{\rm ion} + P_{\rm ele} + P_{\rm pos} + P_{\rm
   coul}

.. math::

   \label {Eqn:eos3b}
   \epsilon_{\rm tot} = \epsilon_{\rm rad} + \epsilon_{\rm ion} +
   \epsilon_{\rm ele} + \epsilon_{\rm pos}  + \epsilon_{\rm coul} \; .

Here the subscripts “rad,” “ion,” “ele,” “pos,” and “coul” represent the
contributions from radiation, nuclei, electrons, positrons, and
corrections for Coulomb effects, respectively. The radiation portion
assumes a blackbody in local thermodynamic equilibrium, the ion portion
(nuclei) is treated as an ideal gas with :math:`\gamma \, = \, 5/3`, and
the electrons and positrons are treated as a non-interacting Fermi gas.

The blackbody pressure and energy are calculated as

.. math::

   \label {Eqn:eos4a}
   P_{\rm rad} = {a T^4 \over 3}

.. math::

   \label {Eqn:eos4b}
   \epsilon_{\rm rad} = { 3 P_{\rm rad} \over \rho} \,

where :math:`a` is related to the Stephan-Boltzmann constant
:math:`\sigma_B \, = \,
a c/4`, and :math:`c` is the speed of light. The ion portion of each
routine is the ideal gas of () with :math:`\gamma \, =
\, 5/3`. The number densities of free electrons :math:`N_{\rm ele}` and
positrons :math:`N_{\rm pos}` in the noninteracting Fermi gas formalism
are given by

.. math::

   \label {Eqn:eos6a}
   N_{\rm ele} = {8 \pi \sqrt{2} \over h^3} \
                     m_{\rm e}^3 \ c^3 \ \beta^{3/2} \
             \left[ F_{1/2}(\eta,\beta) \ + \ F_{3/2}(\eta,\beta) \right]

.. math::

   \label {Eqn:eos6b}
   N_{\rm pos} = {8 \pi \sqrt{2} \over h^3} \
                     m_{\rm e}^3 \ c^3 \ \beta^{3/2}
     \left[
            F_{1/2} \left( -\eta - 2/\beta, \beta \right)
            \ + \ \beta \
            F_{3/2} \left( -\eta - 2 /\beta, \beta \right)
       \right] \enskip ,

where :math:`h` is Planck’s constant, :math:`m_{\rm e}` is the electron
rest mass, :math:`\beta \: = \: k T / (m_{\rm e} c^2)` is the relativity
parameter, :math:`\eta \: = \: \mu / k T` is the normalized chemical
potential energy :math:`\mu` for electrons, and
:math:`F_{k}(\eta,\beta)` is the Fermi-Dirac integral

.. math::

   \label{Eqn:eos7}
       F_{k}(\eta,\beta) = \int\limits_{0}^{\infty} \
       {x^{k} \ (1 + 0.5 \ \beta \ x)^{1/2} \ dx
        \over
        \exp(x - \eta) + 1
       }\ .

Because the electron rest mass is not included in the chemical
potential, the positron chemical potential must have the form
:math:`\eta_{{\rm pos}} \, = \, -\eta - 2/\beta`. For complete
ionization, the number density of free electrons in the matter is

.. math::

   \label{Eqn:eos8}
   N_{\rm ele,matter}
      = {\bar{Z} \over \bar{A}} \ N_a \ \rho = \bar{Z} \ N_{\rm ion}
   \ ,

and charge neutrality requires

.. math::

   \label{Eqn:eos9}
   N_{\rm ele,matter} = N_{\rm ele} - N_{\rm pos} \ .

Solving this equation with a standard one-dimensional root-finding
algorithm determines :math:`\eta`. Once :math:`\eta` is known, the
Fermi-Dirac integrals can be evaluated, giving the pressure, specific
thermal energy, and entropy due to the free electrons and positrons.
From these, other thermodynamic quantities such as :math:`\gamma_1` and
:math:`\gamma_4` are found. Full details of this formalism may be found
in Fryxell *et al.* (2000) and references therein.

The above formalism requires many complex calculations to evaluate the
thermodynamic quantities, and routines for these calculations typically
are designed for accuracy and thermodynamic consistency at the expense
of speed. The Helmholtz EOS in |flashx| provides a table of the Helmholtz
free energy (hence the name) and makes use of a thermodynamically
consistent interpolation scheme obviating the need to perform the
complex calculations required of the above formalism during the course
of a simulation. The interpolation scheme uses a bi-quintic Hermite
interpolant resulting in an accurate EOS that performs reasonably well.

The Helmholtz free energy,

.. math::

   \label {Eqn:eos13a}
   F = \epsilon - T \ S

.. math::

   \label {Eqn:eos13b}
   dF = -S \ dT + {P \over \rho^2} \ d\rho
   \enskip ,

is the appropriate thermodynamic potential for use when the temperature
and density are the natural thermodynamic variables. The free energy
table distributed with |flashx| was produced from the Timmes EOS (Timmes
& Arnett 1999). The Timmes EOS evaluates the Fermi-Dirac integrals
`[Eqn:eos7] <#Eqn:eos7>`__ and their partial derivatives with respect to
:math:`\eta` and :math:`\beta` to machine precision with the efficient
quadrature schemes of Aparicio (1998) and uses a Newton-Raphson
iteration to obtain the chemical potential of
`[Eqn:eos9] <#Eqn:eos9>`__. All partial derivatives of the pressure,
entropy, and internal energy are formed analytically. Searches through
the free energy table are avoided by computing hash indices from the
values of any given :math:`(T,\rho \bar{Z}/\bar{A})` pair. No
computationally expensive divisions are required in interpolating from
the table; all of them can be computed and stored the first time the EOS
routine is called.

We note that the Helmholtz free energy table is constructed for only the
electron-positron plasma, and it is a 2-dimensional function of density
and temperature, *i.e.* :math:`F(\rho,{\rm T})`. It is made with
:math:`{\bar {\rm A}} \, = \, {\bar {\rm Z}} = 1` (pure hydrogen), with
an electron fraction :math:`Y_{\rm e} \, = \, 1`. One reason for not
including contributions from photons and ions in the table is that these
components of the Helmholtz EOS are very simple (), and one doesn’t need
fancy table look-up schemes to evaluate simple analytical functions. A
more important reason for only constructing an electron-positron EOS
table with :math:`Y_{\rm e} \, = \, 1` is that the 2-dimensional table
is valid for *any* composition. Separate planes for each
:math:`Y_{\rm e}` are not necessary (or desirable), since simple
multiplication by :math:`Y_{\rm
e}` in the appropriate places gives the desired composition scaling. If
photons and ions were included in the table, then this valuable
composition independence would be lost, and a 3-dimensional table would
be necessary.

The Helmholtz EOS has been subjected to considerable analysis and
testing (Timmes & Swesty 2000), and particular care was taken to reduce
the numerical error introduced by the thermodynamical models below the
formal accuracy of the hydrodynamics algorithm (Fryxell, et al. 2000;
Timmes & Swesty 2000). The physical limits of the Helmholtz EOS are
:math:`10^{-10}\,<\,\rho\,<\,10^{11}~({\rm g~cm}^{-3})` and
:math:`10^{4}\,<\,T\,<\,10^{11}` (K). As with the gamma-law EOS, the
Helmholtz EOS provides both forward and backward relations. In the case
of the forward relation (:math:`\rho, T`, given along with the
composition) the table lookup scheme and analytic formulae directly
provide relevant thermodynamic quantities. In the case of the backward
relation (:math:`\rho, \epsilon`, and composition given), the routine
performs a Newton-Rhaphson iteration to determine temperature. It is
possible for the input variables to be changed in the iterative modes
since the solution is not exact. The returned quantities are
thermodynamically consistent.

.. _`Sec:Eos Usage`:

Usage
-----

.. _`Sec:Eos Initialization`:

Initialization
~~~~~~~~~~~~~~

The initialization function of the Eos unit ``physics/Eos/Eos_init`` is
fairly simple for theideal gas gamma law implementation included.
It gathers the runtime parameters and the physical constants needed by
the equation of state and stores them in the data module. The Helmholtz
EOS ``physics/Eos/Eos_init`` routine is a little more complex. The
``Helmholtz`` EOS requires an input file ``helm_table.dat`` that
contains the lookup table for the electron contributions. This table is
currently stored in ASCII for portability purposes. When the table is
first read in, a binary version called ``helm_table.bdat`` is created.
This binary format can be used for faster subsequent restarts on the
same machine but may not be portable across platforms. The ``Eos_init``
routine reads in the table data on processor 0 and broadcasts it to all
other processors.

.. _`Sec:Eos Runtime Parameters`:

Runtime Parameters
~~~~~~~~~~~~~~~~~~

Runtime parameters for the ``Gamma`` unit require the user to set the
thermodynamic properties for the single gas. ``Eos/gamma``,
``Eos/eos_singleSpeciesA``, ``Eos/eos_singleSpeciesZ`` set the ratio of
specific heats and the nucleon and proton numbers for the gas. 

For the Helmholtz equation of state, the table-lookup algorithm requires
a given temperature and density. When temperature or internal energy are
supplied as the input parameter, an iterative solution is found.
Therefore, no matter what mode is selected for ``Helmholtz`` input, the
best initial value of temperature should be provided to speed
convergence of the iterations. The iterative solver is controlled by two
vruntime parameters ``Eos/eos_maxNewton`` and ``Eos/eos_tolerance`` which
define the maximum number of iterations and convergence tolerance. An
additional runtime parameter for ``Helmholtz``, ``Eos/eos_coulumbMult``,
indicates whether or not to apply Coulomb corrections. In some regions
of the :math:`\rho`-:math:`T` plane, the approximations made in the
Coulomb corrections may be invalid and result in negative pressures.
When the parameter ``eos_coulombMult`` is set to zero, the Coulomb
corrections are not applied.

.. _`Sec:Eos Interfaces`:

Direct and Wrapped Calls
~~~~~~~~~~~~~~~~~~~~~~~~

The primary function in the ``Eos`` unit operates on a single data point, taking
density, composition, and either temperature, internal energy, or
pressure as input, and returning :math:`\gamma_1`, and either the
pressure, temperature or internal energy (whichever was not used as
input). This equation of state interface is particularly useful for initializing an
application instance. The user is given direct control over the input and output,
since everything is passed through the argument list. Certain optional quantities such
electron pressure, degeneracy parameter, and thermodynamic derivatives
can be calculated by the ``physics/Eos/Eos`` function if needed. They
are computed if an optional argument ``derivs`` is present in the call.

The hydrodynamic and burning computations repeatedly call the Eos
function to update pressure and temperature during the course of their
calculation. Typically, values in all the cells of the block need of be
updated in these calls. Additionally, the kinetic energy in the system
should be taken into account for accuracy. The single point interface
is not aware of velocities, and therefore cannot consider the effects
of kinetic energey directly. For user convenience several interfaces
are provided to make the invocation of Eos more convenient.
The interface ``physics/Eos/Eos_fillEosData`` convert data coming in as a
multidimensional array of grid data into a two dimensional data
structure  and ``physics/Eos/Eos_getFromEosData`` does the reverse. It
puts the updated values returned in the two dimensional array into the
multidimensional grid data.  

The interface ``physics/Eos/Eos_vector`` applies EOS to the two
dimensional vectorised data points given to it. Similar to the
``physics/Eos/Eos``, this interface does not have access to the
velocity and other grid data values. It assumes that the calling
routine will have either invoked  ``physics/Eos/Eos_fillEosData`` or
otherwise accounted for kinetice energy if it is desired before
calling this routine. Similarly it expects an invocation of
``physics/Eos/Eos_getFromEosData`` by the 
calling routine afterwards if those same effects are to be accounted
for. The difference between ``physics/Eos/Eos`` and
``physics/Eos/Eos_vector`` is that the former works on a single data
point while the latter works on a vector of data point.

An additional interface ``physics/Eos/Eos_multiDim`` provides a means by which the
details of translating the data from block to vector and back, and
accounting for kinetic energy  are hidden
from the calling unit. This  interface permits the caller to
define a multidimensional collection of cells along each their lower
and upper bounds. These are typically either a whole block or a
section of block.  The ``Eos_multiDim`` returns updated values in the
same multidimensional array where the input values were provided. The
internal mechanisms for manipulating the data are transparent to the
user. The main implementation of this routine calls
``physics/Eos/Eos_fillEosData``, ``physics/Eos/Eos_vector`` and
``physics/Eos/Eos_getFromEosData`` in that order. 
This wrapper routine does not calculate the optional derivative
quantities. If they are needed, one needs to call either point-wise of
vector based interfaces with the optional arguement ``derivs``
in the argument list of the invoking call..


.. _`Sec:Eos Unit Test`:

Unit Test
---------

The unit test of the Eos function can exercise all the
implementations. Because the Gamma law allows only one species, the
setup required for each implementations is specific. To invoke any
three-dimensional ``Eos`` unit test, the command is:

   ``./setup unitTest/Eos/``\ *implementation* ``-auto -3d``

where *implementation* is one of ``Gamma``, ``Helmholtz``, or
``Hybrid/Helmholtz_Weaklib`` . The ``Eos`` unit test works on the assumption that if the
four physical variables in question (density, pressure, energy and
temperature) are in thermal equilibrium with one another, then applying
the equation of state to any two of them should leave the other two
completely unchanged. Hence, if we initialize density and temperature
with some arbitrary values, and apply the equation of state to them in
``MODE_DENS_TEMP``, then we should get pressure and energy values that
are thermodynamically consistent with density and temperature. Now after
saving the original temperature value, we apply the equation of state to
density and newly calculated pressure. The new value of the temperature
should be identical to the saved original value. This verifies that the
``Eos`` unit is computing correctly in ``MODE_DENS_PRES`` mode. By
repeating this process for the remaining two modes, we can say with
great confidence that the ``Eos`` unit is functioning normally.

In our implementation of the Eos unit test, the initial conditions
applied to the domain create a gradient for density along the :math:`x`
axis and gradients for temperature and pressure along the :math:`y`
axis. If the test is being run for the Helmholtz implementation, then
the species are initialized to have gradients along the :math:`z`
axis. The initial conditions for the Hybrid unit test make sure that
physical conditions are generated that would cause one set of cells to
call the Helmholtz implementation, another set to call the Weaklib
implementation and yet another set to call both. 

