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
   header file . Including this file with #include “constants.h” is an
   alternate way to access the value of :math:`\pi`, rather than needing
   to include the unit.

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

In this example, the local variable is set equal to the result,
:math:`4.4983\times 10^{-15}` (to five significant figures).

Physical constants are taken from K. Nahamura *et al.* (Particle Data
Group), J. Phys. G **37**, 075021 (2010).

.. _`sec:PhysicalConstantsAvailable`:

Available Constants and Units
-----------------------------

There are many constants and units available within , see and . Should
the user wish to add additional constants or units to a particular
setup, the routine should be overridden and the new constants added
within the directory of the setup.

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
         | length         | m              |                | meter          |
         +----------------+----------------+----------------+----------------+
         | length         | km             |                | kilometer      |
         +----------------+----------------+----------------+----------------+
         | length         | pc             |                | parsec         |
         +----------------+----------------+----------------+----------------+
         | length         | kpc            |                | kiloparsec     |
         +----------------+----------------+----------------+----------------+
         | length         | Mpc            |                | megaparsec     |
         +----------------+----------------+----------------+----------------+
         | length         | Gpc            |                | gigaparsec     |
         +----------------+----------------+----------------+----------------+
         | length         | Rsun           |                | solar radius   |
         +----------------+----------------+----------------+----------------+
         | length         | AU             |                | astronomical   |
         |                |                |                | unit           |
         +----------------+----------------+----------------+----------------+
         | time           | yr             |                | year           |
         +----------------+----------------+----------------+----------------+
         | time           | Myr            |                | megayear       |
         +----------------+----------------+----------------+----------------+
         | time           | Gyr            |                | gigayear       |
         +----------------+----------------+----------------+----------------+
         | mass           | kg             |                | kilogram       |
         +----------------+----------------+----------------+----------------+
         | mass           | Msun           |                | solar mass     |
         +----------------+----------------+----------------+----------------+
         | mass           | amu            |                | atomic mass    |
         |                |                |                | unit           |
         +----------------+----------------+----------------+----------------+
         | charge         | C              |                | Coulomb        |
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
         | length         | LFLY           |                | 1 Mpc          |
         +----------------+----------------+----------------+----------------+
         | time           | TFLY           |                | :math:`\       |
         |                |                |                | frac{2}{3H_0}` |
         +----------------+----------------+----------------+----------------+
         | mass           | MFLY           |                | Msun           |
         +----------------+----------------+----------------+----------------+

.. _`sec:PhysicalConstantsRP`:

Applicable Runtime Parameters
-----------------------------

There is only one runtime parameter used by the Physical Constants unit:
selects the default system of units for returned constants. It is a
three-character string set to "CGS" or "MKS"; the default is CGS.

.. _`sec:PhysicalConstantsRoutines`:

Routine Descriptions
--------------------

The following routines are supplied by this unit.

-  Request a physical constant given by a string, and returns its real
   value. This routine takes optional arguments for converting units
   from the default. If the constant name or any of the optional unit
   names aren’t recognized, a value of 0 is returned.

-  Initializes the Physical Constants Unit by loading all constants.
   This routine is called by and must be called before the first
   invocation of . In general, the user does not need to invoke this
   call.

-  Lists the available physical constants in a snappy table.

-  Lists all the units available for optional conversion.

-  Lists all physical constants and units, and tests the unit conversion
   routines.

.. container:: flashtip

   The header file must be included in the calling routine due to the
   optional arguments of .

.. _`Sec:PhysConstUnitTest`:

Unit Test
---------

The unit test is a simple exercise of the functionality in the unit. It
does not require time stepping or the grid. “Correct” usage is
indicated, as is erroneous usage.
