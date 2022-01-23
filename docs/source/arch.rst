.. _`Chp:Architecture`:

Architecture
========================

Flash-X has a component based architecture where
different permutations and combinations of various components generate
different applications. The code has hierarchy in its components. At
the top level are *Units* that implement a specific
functionality. They are equivalent of *Classes* in object oriented
models without explicitly making use of the Class construct. The
reason is both a design choice and historical. Through a somewhat
loose interpretation of classes it is possible to obtain arbitrary
granularities in components without cumbersome inheritance
structure. And it allows legacy components to become compatible with
the code architecture by attaching meta information to them without
having to modify the code itself. Units publish a high level API
through which any code outside of the Unit interacts with them. 
Units can have *subunits* where each subunit implements a subset of
the Unit's API such that the union of all subunits covers the whole
API of the Unit. Subunits cover disjointed sets of API in that the
subset of API implemented by a subunit is unique to it. Below subunit
level the code components are not given any specific name. For
convenience heretofore they will be referred to as *components* with
the understanding that they represent the unix subtree in the source starting at
their path.

The include a configuration domain specific language (DSL) for program
synthesis through assembly, and a key-value dictionary with a
correspoding translator *macroprocessor*. The configuration toolchain
takes its inspiration from the *setup tool* of FLASH which has been used
to implement composability at the level of major functionalities in the
code. The setup tool interprets the configuration (DSL), which encode
metadata about each component and subcomponent of the code in a
completely distributed fashion in *config files*. The metadata about one
component is completely oblivious of the metadata about other
components, though it has some knowledge of the components that are
available in the code. The setup tool recursively parses the config file
of each mentioned component, assimilates the information, and assembles
a consistent, fully-configured instance of an application along with its
build system and static runtime environment. The key-value dictionary
serves dual purpose of lowering the granularity of components for
assembly by the setup tool, and enables single expression of maintained
source code. The keys are language agnostic macros with enhancements
such as inlining, recursion and arguments. Combined with the setup tool,
the macroprocessor brings the functionality of C++ template
metaprogramming tools such as Kokkos and Raja that permits maintenance
of single source code that can be specialized for target devices as
needed.

The enable the maintained source code to be augmented with performance
hints in the form of native directives. Another DSL has been built for
expressing *recipies* for overall control flow of the simulation. The
recipes replace the timestepper functionality because control flow
during evolution of solution now may involve offloading work and data to
different devices, therefore, simple drivers are no longer adequate. The
have two tasks: (1) parse the directives for optimization within the
physics operators; and (2) to translate the recipe into a data flow
description of the execution suitable for the . The then uses this
generated information to orchestrate computation and data movement
between devices as needed. The overarching design principle of the is to
enable both domain-specific knowledge and platform knowledge to be
utilized in application configuration without placing undue burden on
either the tool developers or the domain experts.

The exposes the hierarchy of data motion and allocations deep within the
components of the applications and separates them from the arithmetic
and logic implementing the numerical method. On the platform hardware
side, they expose the hierarchical parallelism of a given platform’s
hardware through appropriately designed interfaces. The objective of
this exercise is to enable the software to take maximal advantage of the
exposed parallelism in hardware without having to alter the arithmetic.
The directives serve the purpose of letting the code translators know
which kind of transformations are safe for specific devices. Some
program synthesis is involved in the process that can take several
passes of code generation. Similar to the code expression, the offline
tools themselves subscribe to separation of concerns in the sense that
each tool addresses itself to a very focused program synthesis step.

We begin by casting every non-trivial function in the physics operators
of the code as a collection of code blocks. Some code blocks may be
declarations, some may implement the control logic, and some will
implement the numerics of the function. Sometimes arithmetic and logic
blocks cannot be separated out; some code blocks have both. These code
blocks become components in a hierarchical composability through the use
of key-value dictionary where values are code snippets of arbitrary
length and complexity. Keys are user defined with a provision for
multiple alternative definitions which lets them mimic the template
meta-programming in C++ where a single expression of an algorithm can
have specializations through alternative definitions of the key (see for
details).

The information encoded in the directives is utilized by the to
determine which code blocks can be treated as kernels on devices such as
accelerators, which code blocks can have overlapped computation with
other code blocks, which code blocks can be queued sequentially so that
data movement is minimized, etc. Note that the absence of such metadata
has no implication for correct execution. Instead, what this
decomposition aims for is better optimization with richer metadata.
While the present implementation primarily requires hand coding of this
information, we expect to be able to utilize third-party code analysis
tools to infer this information automatically in the future. This
division of labor between multiple tools is made not only to achieve
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

The design goal of the is to develop a base set of runtime elements,
such as thread teams, that can be composed at runtime into **thread team
configurations** in ways that maximize the efficient use of the node.
For a given simulation, the operates with :math:`N` thread teams that
are created when the simulation starts and persist until the simulation
terminates, where each team is allowed to simultaneously use at most
:math:`M` threads at any given time. The thread teams are run in cycles
such that for each cycle a team is assigned an action routine and a data
type. Examples of data types are tiles, blocks, and data packets of
blocks. Upon starting a cycle, the team is given data items of the
appropriate type and the threads of the team work in a coordinated
fashion so that the given action is applied to each data item once. The
cycle ends once the team has been informed that no more data items will
be given and the action has been applied to all given data items. In our
design, the pairing of the action with data items is viewed as a task so
that thread teams implement task-based parallelism *via* thread-level
parallelism. While the threads in the team execute code in the host,
they can also be used to launch kernels in accelerators and therefore
make use of finer-grained parallelism. Ideally, the data type assigned
to a thread team for a given action will be chosen based on the hardware
that the action routine has been written to use. For instance, a tile
would be a sensible data type for executing an action routine that uses
the CPU for heavy computation or a data packet of blocks for a routine
that launches kernels on a remote device with its own memory system.
While it might have been possible to adapt existing tools such as Legion
:raw-latex:`\cite{legion}` for some of the functionality we are aiming
for, we have opted for an end-to-end domain specific solution that has
simple enough tools that even a small team can customize and maintain
with their own code. Additionally, our orchestration system is
absolutely language agnostic, even though we are exercising it with
Fortran.

Note that no part of this design limits itself to using manually built
task graphs nor requires extreme simplicity in them. The interfaces are
designed to be robust enough that the can also execute complex task
graphs. By disassociating the executor of the graph from the generator,
we have ensured that in the future, we will be able to use automation in
task graph generation without having to alter the mechanics of the . All
the heavy lifting of analysis and scheduling can be done separately in
the generator which never needs to interface with the executor. We
believe that this model of combination of program synthesis with
orthogonal composition concepts will have longevity because of its
ability to adapt to new systems incrementally.

.. _s.trans:

Translator
~~~~~~~~~~

The translator for the keys adds a new capability to the Setup Tool that
implements the key-value dictionary feature of the program synthesis.
This tool, like the Setup Tool, is written in python and is the last
feature invoked during the configuration of an application instance.
Definitions are stored in “.ini” files, according to the Python
ConfigParser format. Style guidelines are documented for ease of use by
the developers. When a new definition file is created, the corresponding
config file is informed about its existence so that the Setup Tool
includes it during configuration. The source files needing translation
are given the extension “.F90-mc” to differentiate them from the source
files that do not need to have the translation applied on them. The
translator can also be invoked independently if a developer wishes to
inspect how keys are replaced by their definitions. The output comes out
in a “.F90” file which is a normal Fortran file and can be compiled by
any Fortran compiler. Various command-line arguments can specify which
files to process and which definitions files to read. By default, the
translator will read all ‘.ini’ files in the directory, then process all
‘.F90-mc’ files and write the modified files to ‘.F90’ format.

.. _`Sec:Inheritance`:

Inheritance
-----------

inheritance is implemented through the Unix directory structure and the
setup tool. When the tool parses the source tree, it treats each child
or subdirectory as inheriting all of the Config and Makefile files in
its parent’s directory. While source files at a given level of the
directory hierarchy override files with the same name at higher levels,
Makefiles and configuration files are cumulative. Since functions can
have multiple implementations, selection for a specific application
follows a few simple rules applied in order described in

However, we must take care that this special use of the directory
structure for inheritance does not interfere with its traditional use
for organization. We avoid any problems by means of a careful naming
convention that allows clear distinction between organizational and
namespace directories. See for naming conventions.

.. _`Sec: unitTest `:

Unit Test Framework
-------------------

In keeping with good software practice, incorporates a unit test
framework that allows for rigorous testing and easy isolation of errors.
The components of the unit test show up in two different places in the
source tree. One is a dedicated path in the unit, , where is the name of
a specific unit test. The other place is a subdirectory called ,
somewhere in the hierarchy of the corresponding unit which implements a
function and any helper functions it may need. The primary reason for
organizing unit tests in this somewhat confusing way is that unit tests
are special cases of simulation setups that also need extensive access
to internal data of the unit being tested. By splitting the unit test
into two places, it is possible to meet both requirements without
violating unit encapsulation. We illustrate the functioning of the unit
test framework with the unit test of the Eos unit. For more details
please see . The Eos unit test needs its own version of the routine
which makes a call to its routine. The initial conditions specification
and unit test specific are placed in , since the Simulation unit allows
any substitute function to be placed in the specific simulation
directory. The function resides in , and therefore has access to all
internal Eos data structures and helper functions.
