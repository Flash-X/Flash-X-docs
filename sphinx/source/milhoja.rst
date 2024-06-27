.. include:: defs.h

.. _`Chp:milhoja`:


|milhoja|

======================

.. _`Sec:milhojaIntroduction`:

Overview
--------



.. _`Sec:mdesign`:

Milhoja Design Features
-------------------

The design goal of |milhoja| is to develop a base set of runtime
elements, such as thread teams, that can be composed at runtime into
**thread team configurations** in ways that maximize the efficient use
of the node. For a given simulation, |milhoja| operates with :math:`N`
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
designed to be robust enough that |milhoja| can also execute complex
task graphs. By disassociating the executor of the graph from the
generator, we have ensured that in the future, we will be able to use
automation in task graph generation without having to alter the
mechanics of the |milhoja|. All the heavy lifting of analysis and
scheduling can be done separately in the generator which never needs to
interface with the executor. We believe that this model of combination
of program synthesis with orthogonal composition concepts will have
longevity because of its ability to adapt to new systems incrementally.


comprised of three sets of tools:
the **configuration tools** (CFT), **code translators** (CT) and
the **runtime orchestrator**  (|milhoja|).

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


|orcha| exposes the hierarchy of data motion and allocations deep
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





