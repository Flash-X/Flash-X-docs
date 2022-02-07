.. include:: defs.h

.. _`Chp:Runtime Parameters Unit`:

Runtime Parameters Unit
=======================

The ``RuntimeParameters`` Unit stores and maintains a global linked
lists of runtime parameters that are used during program execution.
Runtime parameters can be added to the lists, have their values
modified, and be queried. This unit handles adding the default runtime
parameters to the lists as well as reading any overwritten parameters
from the ``flash.par`` file.

Defining Runtime Parameters
---------------------------

All parameters must be declared in a ``Config`` file with the keyword
declaration ``PARAMETER``. In the ``Config`` file, assign a data type
and a default value for the parameter. If possible, assign a range of
valid values for the parameter. You can also provide a short description
of the parameter’s function in a comment line that begins with D.

.. container:: shrink

   .. container:: codeseg

      #section of Config file for a Simulation D myParameter Description
      of myParameter PARAMETER myParameter REAL 22.5 [20 to 60]

To change the runtime parameter’s value from the default, assign a new
value in the flash.par for the simulation.

.. container:: shrink

   .. container:: codeseg

      #snippet from a flash.par myParameter = 45.0

See for more information on declaring parameters in a ``Config`` file.

Identifying Valid Runtime Parameters
------------------------------------

The values of runtime parameters are initialized either from default
values defined in the ``Config`` files, or from values explicitly set in
the file ``flash.par``. Variables that have been changed from default
are noted in the simulation’s output log. For example, the
RuntimeParameters section of the output log shown in indicates that
``Particles/pt_numx`` and ``Particles/pt_numy`` have been read in from
``flash.par`` and are different than the default values, whereas the
runtime parameter ``IO/checkpointFileIntervalStep`` has been left at the
default value of 0.

.. container:: fcodeseg

   =======================================================
   RuntimeParameters:
   ======================================================= pt_numx = 10
   [CHANGED] pt_numy = 5 [CHANGED] checkpointfileintervalstep = 0

After a simulation has been configured with a ``setup`` call, all
possible valid runtime parameters are listed in the file
``setup_params`` located in the ``object`` directory (or whatever
directory was chosen with ``-objdir=``) with their default values. This
file groups the runtime parameters according to the units with which
they are associated and in alphabetical order. A short description of
the runtime parameter, and the valid range or values if known, are also
given. See for an example listing.

.. container:: fcodeseg

   physics/Eos/EosMain/Multigamma gamma [REAL] [1.6667] Valid Values:
   Unconstrained Ratio of specific heats for gas

   physics/Hydro/HydroMain cfl [REAL] [0.8] Valid Values: Unconstrained
   Courant factor

Routine Descriptions
--------------------

The Runtime Parameters unit is included by default in all of the
provided |flashx| simulation examples, through a dependence within the
``Driver`` unit. The main |flashx| initialization routine
(``Driver/Driver_initFlash``) and the initialization code created by
``setup`` handles the creation and initialization of the runtime
parameters, so users will mainly be interested in querying parameter
values. Because the RuntimeParameters routines are overloaded functions
which can handle character, real, integer, or logical arguments, the
user *must* make sure to ``use`` the interface file
``RuntimeParameters_Interfaces`` in the calling routines.

The user will typically only have to use one routine from the Runtime
Parameters API, ``RuntimeParameters/RuntimeParameters_get``. This
routine retrieves the value of a parameter stored in the linked list in
the ``RuntimeParameters_data`` module. In |flashx| the value of runtime
parameters for a given unit are stored in that unit’s ``Unit_data``
Fortran module and they are typically initialized in the unit’s
``Unit_init`` routine. Each unit’s ’init’ routine is only called once at
the beginning of the simulation by ``Driver/Driver_initFlash``. For more
documentation on the |flashx| code architecture please see . It is
important to note that even though runtime parameters are declared in a
specific unit’s ``Config`` file, the runtime parameters linked list is a
global space and so any unit can fetch a parameter, even if that unit
did not declare it. For example, the ``Driver`` unit declares the
logical parameter ``restart``, however, many units, including the ``IO``
unit get ``restart`` parameter with the
``RuntimeParameters/RuntimeParameters_get`` interface. If a section of
the code asks for a runtime parameter that was not declared in a
``Config`` file and thus is not in the runtime parameters linked list,
the |flashx| code will call ``Driver/Driver_abortFlash`` and stamp an
error to the ``logfile``. The other RuntimeParameter routines in the API
are not generally called by user routines. They exist because various
other units within |flashx| need to access parts of the RuntimeParameters
interface. For example, the input/output unit ``IO`` needs
``RuntimeParameters/RuntimeParameters_set``. There are no user-defined
parameters which affect the ``RuntimeParameters`` unit.

Example Usage
-------------

An implementation example from the ``IO/IO_init`` is straightforward.
First, use the module containing definitions for the unit (for ``_init``
subroutines, the usual ``use Unit_data, ONLY:`` structure is waived).
Next, use the module containing interface definitions of the
``RuntimeParameters`` unit, *i.e.*,
``use RuntimeParameters_interface, ONLY:``. Finally, read the runtime
parameters and store them in unit-specific variables.

.. container:: codeseg

   subroutine IO_init()

   use IO_data use RuntimeParamters_interface, ONLY :
   RuntimeParameters_get implicit none

   call RuntimeParameters_get(’plotFileNumber’,io_plotFileNumber) call
   RuntimeParameters_get(’checkpointFileNumber’,io_checkpointFileNumber)

   call
   RuntimeParameters_get(’plotFileIntervalTime’,io_plotFileIntervalTime)
   call
   RuntimeParameters_get(’plotFileIntervalStep’,io_plotFileIntervalStep)
   call
   RuntimeParameters_get(’checkpointFileIntervalTime’,io_checkpointFileIntervalTime)
   call
   RuntimeParameters_get(’checkpointFileIntervalStep’,io_checkpointFileIntervalStep)

   !! etc ...

Note that the parameters found in the ``flash.par`` or in the ``Config``
files, for example ``IO/plotFileNumber``, are generally stored in a
variable of the same name with a unit prefix prepended, for example
``io_plotFileNumber``. In this way, a program segment clearly indicates
the origin of variables. Variables with a unit prefix (*e.g.*, ``io_``
for ``IO``, ``pt_`` for particles) have been initialized from the
``RuntimeParameters`` database, and other variables are locally
declared. When creating new simulations, runtime parameters used as
variables should be prefixed with ``sim_``.
