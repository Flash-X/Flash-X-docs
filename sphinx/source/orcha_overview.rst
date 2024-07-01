.. include:: defs.h


.. _`Chp:ORCHA Overview`:

Overview of ORCHA
======================

Applications can effectively utilize heterogeneous platforms if they
have (1) data structures and algorithms suitable for target
devices, (2) can conceptualize a map of computation to target devices,
and (3) can execute the map by moving data and computation to devices
efficiently. In general, attempting to meet these requirements naively
can result in several implementation variants of the same
computation. That in turn could lead to a maintenance nightmare. Tools
in |orcha| avoid this nightmare through abstractions and
code generation. The tools are designed such that each tool focuses on
a small subset of abstractions and code generation that have similar
requirements, but are substantially different from those addressed by
the other tools. Through this approach of divide and conquer the tools
have been kept relatively simple and customizable, but their
combination provides a powerful performance portability solution. Four
main tools are |setup|, |cgkit|, |MP|, and  |milhoja|. They have
helper code generation tools that enable daisy-chaining the actions of
the tools in different ways needed by different applications. The
tools are described individually in separate sections followed by an
explanation of how are they combined for an end-to-end solution.


|orcha| tools expose the hierarchy of data motion and allocations deep
within the components of the applications and separate them from the
arithmetic and logic implementing the numerical method. On the platform
hardware side, they expose the hierarchical parallelism of a given
platform’s hardware through appropriately designed interfaces. The
objective of this exercise is to enable the software to take maximal
advantage of the exposed parallelism in hardware without having to alter
the arithmetic. Some program synthesis is involved in the process that can take
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



