.. include:: defs.h

.. _`Chp:Architecture`:

Overview of |flashx| architecture
================================

|flashx| is a component based software system where different
permutations and combinations of various components generate different
applications. Some aspects of |flashx| architecture are adapted from
|flash| , but it is fundamentally a new
software with an architecture designed for use with heterogeneous
platforms. Portability on heterogeneous platforms is achieved through
a **language-agnostic performance portability layer** (LAPPL) comprised of three sets of tools:
the **configuration tools** (CFT), **code translators** (CT) and
the **runtime orchestrator**  (RO).

The CFT include a domain specific language configuration (DSCL) for
program synthesis through assembly, and a key-value dictionary with a
correspoding translator *macroprocessor*. The configuration toolchain
takes its inspiration from the *setup tool* of |flash| which has been used
to implement composability at the level of major functionalities in the
code. The setup tool interprets the configuration (DSCL), which encodes
metadata about each component and subcomponent of the code in a
completely distributed fashion in *Config files*. The metadata about one
component is completely oblivious of the metadata about other
components, though it has some knowledge of the components that are
available in the code. The setup tool recursively parses the Config file
of each mentioned component, assimilates the information, and assembles
a consistent, fully-configured instance of an application along with its
build system and a static runtime environment. The key-value dictionary
serves dual purpose of lowering the granularity of components for
assembly by the setup tool, and enables single expression of maintained
source code. The keys are language-agnostic macros with enhancements
such as inlining, recursion and arguments. Combined with the setup tool,
the macroprocessor brings the functionality of C++ template
metaprogramming that permits maintenance
of single source code that can be specialized for target devices as
needed. In addition to being able to generate variants for different
target architecture, the setup tool is also able to include multiple
variants of same API by appropriately appending the variant name to
the routine/function name during configuration. This feature is
useful, for example, if different equations of state need to be used
in different parts of the domain.

Another DSL is being built for expressing *recipies* for overall
control flow of the simulatio, which are translated into a FORTRAN
by the CT. The recipes replace the timestepper
functionality because control flow during evolution of solution now
may involve offloading work and data to different devices, therefore,
simple drivers are no longer adequate. The CT have two tasks: (1)
parse the |flashx| directives for optimization within the physics
operators; and (2) to translate the recipe into a data flow
description of the execution suitable for the RO. The RO then
uses this generated information to orchestrate computation and data
movement between devices as needed. The overarching design principle
of the performance portability layer is to enable both domain-specific
knowledge and platform knowledge to be utilized in application
configuration without placing undue burden on either the tool
developers or the domain experts. 

The LAPPL exposes the hierarchy of data motion and allocations deep
within the components of the applications and separates them from the
arithmetic and logic implementing the numerical method. On the platform
hardware side, they expose the hierarchical parallelism of a given
platform’s hardware through appropriately designed interfaces. The
objective of this exercise is to enable the software to take maximal
advantage of the exposed parallelism in hardware without having to alter
the arithmetic. The directives serve the purpose of letting the code
translators know which kind of transformations are safe for specific
devices. Some program synthesis is involved in the process that can take
several passes of code generation. Similar to the code expression, the
offline tools themselves subscribe to separation of concerns in the
sense that each tool addresses itself to a very focused program
synthesis step.

We begin by casting every non-trivial function in the physics operators
of the code as a collection of code blocks. Some code blocks may be
declarations, some may implement the control logic, and some will
implement the numerics of the function. Sometimes arithmetic and logic
blocks cannot be separated out; some code blocks have both. These code
blocks become **components** in a hierarchical composability through the use
of macros in the form of s key-value dictionary where values are code
snippets of arbitrary length and complexity. Keys are user-defined
with a provision for multiple alternative definitions, including null
definitions, which lets them mimic the template meta-programming in
C++ mentioned earlier.

The information encoded in the directives is to be utilized by the CT to
determine which code blocks can be treated as kernels on devices such as
accelerators, which code blocks can have overlapped computation with
other code blocks, which code blocks can be queued sequentially so that
data movement is minimized, etc. Note that the absence of such metadata
has no implication for correct execution. Instead, what this
decomposition aims for is better optimization with richer metadata.
This division of labor between multiple tools is made not only to achieve
good encapsulation and modularity, but also to ensure that no single
tool is too complex or requires too much intelligence. This design
approach enables extraction of all useful information for orchestrating
computations during runtime when the application is being configured.
This is in contrast to general solutions that delay the orchestration
decision due to the conservative assumption that information about the
tasks and their dependencies is only fully known at runtime. While this
approach may not squeeze every last bit of performance from the
hardware, it has the virtue of being simple and maintainable with no
runtime overheads. Also, by design, every step in program synthesis is
human-readable to facilitate both correctness-debugging and
performance-debugging.

The design goal of the RO is to develop a base set of runtime
elements, such as thread teams, that can be composed at runtime into
**thread team configurations** in ways that maximize the efficient use
of the node. For a given simulation, the RO operates with :math:`N`
thread teams that are created when the simulation starts and persist
until the simulation terminates, where each team is allowed to
simultaneously use at most :math:`M` threads at any given time. The
thread teams are run in cycles such that for each cycle a team is
assigned an action routine and a data type. Examples of data types are
tiles, blocks, and data packets of blocks. Upon starting a cycle, the
team is given data items of the appropriate type and the threads of the
team work in a coordinated fashion so that the given action is applied
to each data item once. The cycle ends once the team has been informed
that no more data items will be given and the action has been applied to
all given data items. In our design, the pairing of the action with data
items is viewed as a task so that thread teams implement task-based
parallelism *via* thread-level parallelism. While the threads in the
team execute code in the host, they can also be used to launch kernels
in accelerators and therefore make use of finer-grained parallelism.
Ideally, the data type assigned to a thread team for a given action will
be chosen based on the hardware that the action routine has been written
to use. For instance, a tile would be a sensible data type for executing
an action routine that uses the CPU for heavy computation or a data
packet of blocks for a routine that launches kernels on a remote device
with its own memory system. 

Note that no part of this design limits itself to using manually built
task graphs nor requires extreme simplicity in them. The interfaces are
designed to be robust enough that the RO can also execute complex
task graphs. By disassociating the executor of the graph from the
generator, we have ensured that in the future, we will be able to use
automation in task graph generation without having to alter the
mechanics of the RO. All the heavy lifting of analysis and
scheduling can be done separately in the generator which never needs to
interface with the executor. We believe that this model of combination
of program synthesis with orthogonal composition concepts will have
longevity because of its ability to adapt to new systems incrementally.


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
