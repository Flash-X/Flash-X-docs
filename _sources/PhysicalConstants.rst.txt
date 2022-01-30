.. include:: defs.h

.. _`Chp:PhysicalConstants`:

Physical Constants Unit
=======================

The Physical Constants unit provides a set of common constants, such as
Pi and the gravitational constant, in various systems of measurement
units. The default system of units is CGS, so named for having a length
unit in centimeters, a mass unit in grams, and a time unit in seconds.
In CGS, the charge unit is the esu, and the temperature unit is the
Kelvin. The constants can also be obtained in the standard MKS system of
units, where length is in meters, mass in kilograms, and time in
seconds. For MKS units, charge is in Coloumbs, and temperature in
Kelvin.

.. container:: flashtip

   For ease of usage, the constant PI=3.14159.... is defined in the
   header file ``constants.h``. Including this file with #include
   “constants.h” is an alternate way to access the value of :math:`\pi`,
   rather than needing to include the ``PhysicalConstants`` unit.

Any constant can optionally be converted from the standard units into
any other available units. This facility makes it easy to ensure that
all parts of the code are using a consistent set of physical constant
values and unit conversions.

For example, a program using this unit might obtain the value of
Newton’s gravitational constant :math:`G` in units of
Mpc\ :math:`^3` Gyr\ :math:`^{-2}M_\odot^{-1}` by calling

   .. container:: codeseg

      call PhysicalConstants_get ("Newton", G, len_unit="Mpc",
      time_unit="Gyr", mass_unit="Msun")

In this example, the local variable ``G`` is set equal to the result,
:math:`4.4983\times 10^{-15}` (to five significant figures).

Physical constants are taken from K. Nahamura *et al.* (Particle Data
Group), J. Phys. G **37**, 075021 (2010).

.. _`sec:PhysicalConstantsAvailable`:

Available Constants and Units
-----------------------------

There are many constants and units available within |flashx|, see and .
Should the user wish to add additional constants or units to a
particular setup, the routine
``PhysicalConstants/PhysicalConstants_init`` should be overridden and
the new constants added within the directory of the setup.

.. container:: center

   .. container::
      :name: Tab:PhysicalConstants

      .. table:: Available Physical Constants

         =================== ==============================
         **String Constant** **Description**
         =================== ==============================
         Newton              Gravitational constant G
         speed of light      Speed of light
         Planck              Planck’s constant
         electron charge     charge of an electron
         electron mass       mass of an electron
         proton mass         Mass of a proton
         fine-structure      fine-structure constant
         Avogadro            Avogadro’s Mole Fraction
         Boltzmann           Boltzmann’s constant
         ideal gas constant  ideal gas constant
         Wien                Wien displacement law constant
         Stefan-Boltzmann    Stefan-Boltzman constant
         pi                  Pi
         e                   e
         Euler               Euler-Mascheroni constant
         =================== ==============================

.. container:: center

   .. container::
      :name: Tab:PhysicalConstantsUnits

      .. table:: Available Units for Conversion of Physical Constants

         +----------------+----------------+----------------+----------------+
         | **Base unit**  | **String       | **Value in CGS | *              |
         |                | Constant**     | units**        | *Description** |
         +================+================+================+================+
         | length         | cm             | 1.0            | centimeter     |
         +----------------+----------------+----------------+----------------+
         | time           | s              | 1.0            | second         |
         +----------------+----------------+----------------+----------------+
         | temperature    | K              | 1.0            | degree Kelvin  |
         +----------------+----------------+----------------+----------------+
         | mass           | g              | 1.0            | gram           |
         +----------------+----------------+----------------+----------------+
         | charge         | esu            | 1.0            | ESU charge     |
         +----------------+----------------+----------------+----------------+
         | length         | m              | ``1.0E2``      | meter          |
         +----------------+----------------+----------------+----------------+
         | length         | km             | ``1.0E5``      | kilometer      |
         +----------------+----------------+----------------+----------------+
         | length         | pc             | ``3.0          | parsec         |
         |                |                | 856775807E18`` |                |
         +----------------+----------------+----------------+----------------+
         | length         | kpc            | ``3.0          | kiloparsec     |
         |                |                | 856775807E21`` |                |
         +----------------+----------------+----------------+----------------+
         | length         | Mpc            | ``3.0          | megaparsec     |
         |                |                | 856775807E24`` |                |
         +----------------+----------------+----------------+----------------+
         | length         | Gpc            | ``3.0          | gigaparsec     |
         |                |                | 856775807E27`` |                |
         +----------------+----------------+----------------+----------------+
         | length         | Rsun           | ``6.96E10``    | solar radius   |
         +----------------+----------------+----------------+----------------+
         | length         | AU             | ``1.49         | astronomical   |
         |                |                | 597870662E13`` | unit           |
         +----------------+----------------+----------------+----------------+
         | time           | yr             | ``             | year           |
         |                |                | 3.15569252E7`` |                |
         +----------------+----------------+----------------+----------------+
         | time           | Myr            | ``3            | megayear       |
         |                |                | .15569252E13`` |                |
         +----------------+----------------+----------------+----------------+
         | time           | Gyr            | ``3            | gigayear       |
         |                |                | .15569252E16`` |                |
         +----------------+----------------+----------------+----------------+
         | mass           | kg             | ``1.0E3``      | kilogram       |
         +----------------+----------------+----------------+----------------+
         | mass           | Msun           | ``             | solar mass     |
         |                |                | 1.9889225E33`` |                |
         +----------------+----------------+----------------+----------------+
         | mass           | amu            | ``1.6          | atomic mass    |
         |                |                | 60538782E-24`` | unit           |
         +----------------+----------------+----------------+----------------+
         | charge         | C              | ``             | Coulomb        |
         |                |                | 2.99792458E9`` |                |
         +----------------+----------------+----------------+----------------+
         |                |                |                |                |
         +----------------+----------------+----------------+----------------+
         | Cosm           |                |                |                |
         | ology-friendly |                |                |                |
         | units using    |                |                |                |
         | :ma            |                |                |                |
         | th:`H_0 = 100` |                |                |                |
         | km/s/Mpc:      |                |                |                |
         +----------------+----------------+----------------+----------------+
         | length         | LFLY           | ``3.0          | 1 Mpc          |
         |                |                | 856775807E24`` |                |
         +----------------+----------------+----------------+----------------+
         | time           | TFLY           | ``2.05759E17`` | :math:`\       |
         |                |                |                | frac{2}{3H_0}` |
         +----------------+----------------+----------------+----------------+
         | mass           | MFLY           | ``9.8847E45``  | ``5.23e12``    |
         |                |                |                | Msun           |
         +----------------+----------------+----------------+----------------+

.. _`sec:PhysicalConstantsRP`:

Applicable Runtime Parameters
-----------------------------

There is only one runtime parameter used by the Physical Constants unit:
``PhysicalConstants/pc_unitsBase`` selects the default system of units
for returned constants. It is a three-character string set to "CGS" or
"MKS"; the default is CGS.

.. _`sec:PhysicalConstantsRoutines`:

Routine Descriptions
--------------------

The following routines are supplied by this unit.

-  ``PhysicalConstants/PhysicalConstants_get`` Request a physical
   constant given by a string, and returns its real value. This routine
   takes optional arguments for converting units from the default. If
   the constant name or any of the optional unit names aren’t
   recognized, a value of 0 is returned.

-  ``PhysicalConstants/PhysicalConstants_init`` Initializes the Physical
   Constants Unit by loading all constants. This routine is called by
   ``Driver/Driver_initFlash`` and must be called before the first
   invocation of ``PhysicalConstants_get``. In general, the user does
   not need to invoke this call.

-  ``PhysicalConstants/PhysicalConstants_list`` Lists the available
   physical constants in a snappy table.

-  ``PhysicalConstants/PhysicalConstants_listUnits`` Lists all the units
   available for optional conversion.

-  ``PhysicalConstants/PhysicalConstants_unitTest`` Lists all physical
   constants and units, and tests the unit conversion routines.

.. container:: flashtip

   The header file ``PhysicalConstants.h`` must be included in the
   calling routine due to the optional arguments of
   ``PhysicalConstants_get``.

.. _`Sec:PhysConstUnitTest`:

Unit Test
---------

The ``PhysicalConstants`` unit test
``PhysicalConstants/PhysicalConstants_unitTest`` is a simple exercise of
the functionality in the unit. It does not require time stepping or the
grid. “Correct” usage is indicated, as is erroneous usage.
