.. include:: defs.h

.. _`Chp:Architecture`:

Overview of |flashx| architecture
================================

|flashx| is a component based software system where different
permutations and combinations of various components generate different
applications. Some aspects of |flashx| architecture are adapted from
|flash| , but it is fundamentally a new software with an architecture
designed for use with heterogeneous platforms. Portability on
heterogeneous platforms is achieved through 
a new **orchestration system for applications** (|orcha|) that is
designed to be language agnostic and adaptable for future changes in
the computing platforms. |orcha| has a collection of tools that address
three main major concerns from the applications perspective described
below.


As mentioned earlier |flashx| is not a monolithic application code;
instead, it should be viewed as a collection of components that are
selectively grouped to form various applications. Users specify which
components should be included in a simulation, define a rough
discretization/parallelization layout, and assign their own initial
conditions, boundary conditions, and problem setup to create a unique
application executable. In |flashx| terminology, a component that
implements an exclusive portion of the code’s functionality is called
a **unit**. A typical |flashx| simulation requires a proper subset of the
units available in the code. Thus, it is important to distinguish
between the entire |flashx| source code and a given |flashx|
application.

.. _`Sec:unit`:

Unit
----

A |flashx| unit provides well-defined functionality and publishes an
Application Programming Interface (API), a collection of routines
through which other units can interact with it. A unit
can have multiple alternative implementations of varying complexity
and for different purposes. A unit can have an arbitrary number of
subunits that provide subsets of the unit’s functionality, though in
practice the number of subunits remains low. Units must include a null
implementation for every routine in their API at the top level of
their hierarchy. This feature permits an application to
easily exclude a unit without the need to modify code elsewhere. For
example, the input/output unit can be easily turned on and off for
testing purposes, by using the null implementations. 

|flashx| implements its inheritance, extensibility, and
modularity  through its configuration layer. This layer
consists of a collection of text Config files that reside at various
levels of the code organization, and the setup tool which interprets
the Config files. The two primary functions of this layer are to
configure a single application from the |flashx| source tree, and to
implement inheritance and customizability in the code. 

Unit architecture abstracts the computational complexity of
the unit from its public interfaces, and controls the scope of various
data items owned by the unit. A unit’s API provides interfaces for
modifying the state of the solution and for accessing and modifying
data it owns that may be needed by other units.  
Units can have one or more subunits which are groupings of
self-contained functionality. The concept subunits
formalizes the selective use of a subset of a unit’s functionality,
and the possibility of multiple alternative implementations of the
same subset.  Subunits implement disjoint subsets of a
unit’s API, where none of the subsets can be a null set. The union of
all subsets constituting various subunits must be exactly equal to the
unit API. Every unit has at least a Main subunit that implements the
bulk of the unit’s functionality, including its initialization. The
Main subunit is also the custodian of all the unit-scope data. 


.. _`Sec:Inheritance`:

|flashx| Inheritance
-------------------

|flashx| inheritance is implemented through the Unix directory structure
and the setup tool. When the ``setup`` tool parses the source tree, it
treats each child or subdirectory as inheriting all of the Config and
Makefile files in its parent’s directory. While source files at a given
level of the directory hierarchy override files with the same name at
higher levels, Makefiles and configuration files are cumulative. Since
functions can have multiple implementations, selection for a specific
application follows a few simple rules described in the following
Figures.

.. container:: center

   .. figure:: overview_flowchart.png
      :alt: overall_flowchart
      :name: Fig:highlevel_flowchart
      :width: 4in

      Overview of the control flow during configuration


      

 .. container:: center

   .. figure:: overall_inheritance.png
      :alt: overall_inheritance
      :name: Fig:high_inheritance
      :width: 4in

      Overview of the inheritance in various components of the code



      
 .. container:: center

   .. figure:: keys_rp_inheritance.png
      :alt: Inheritance as it applied to keys defining macros and
	    runtime parameters
      :name: Fig:keys_rp
      :width: 4in

      Overview of the control flow during configuration

      


 .. container:: center

   .. figure:: file_generation.png
      :alt: Step_file_generation
      :name: Fig:file_generation
      :width: 4in

      Steps in arbitration on files to be included/generated


 However, we must take care that this special use of the directory
structure for inheritance does not interfere with its traditional use
for organization. We avoid any problems by means of a careful naming
convention that allows clear distinction between organizational and
namespace directories. See for naming conventions.




.. _`Sec: unitTest `:

Unit Test Framework
-------------------

In keeping with good software practice, |flashx| incorporates a unit test
framework that allows for rigorous testing and easy isolation of errors.
The components of the unit test show up in two different places in the
|flashx| source tree. One is a dedicated path in the ``Simulation`` unit,
``Simulation/SimulationMain/unitTest/UnitTestName``, where
*UnitTestName* is the name of a specific unit test. The other place is a
subdirectory called ``unitTest``, somewhere in the hierarchy of the
corresponding unit which implements a function ``Unit_unitTest`` and any
helper functions it may need. The primary reason for organizing unit
tests in this somewhat confusing way is that unit tests are special
cases of simulation setups that also need extensive access to internal
data of the unit being tested. By splitting the unit test into two
places, it is possible to meet both requirements without violating unit
encapsulation. We illustrate the functioning of the unit test framework
with the unit test of the ``Eos`` unit. For more details please see .
The ``Eos`` unit test needs its own version of the routine
``Driver/Driver_evolveAll`` which makes a call to its ``Eos_unitTest``
routine. The initial conditions specification and unit test specific
``Driver_evolveAll`` are placed in
``Simulation/SimulationMain/unitTest/Eos``, since the ``Simulation``
unit allows any substitute |flashx| function to be placed in the specific
simulation directory. The function ``Eos_unitTest`` resides in
``physics/Eos/unitTest``, and therefore has access to all internal
``Eos`` data structures and helper functions.
