.. include:: defs.h

.. _`Chp:mp`:


|MP|
======================


With the advent of accelerators we need to have
many more alternative implementations to accommodate differences
in hardware characteristics. Combined with the already existing
alternatives, in many situations we would have had a combinatorial
explosion of maintained alternatives to continue to have the same level of
flexibility in application configurability on different
platforms. Often, the alternatives for
different devices need either a different data layout or a different
control flow while the arithmetic of the algorithm remains the
same. Hence, without some form of unification of the code base there
would be rampant duplication of arithmetic code leading to maintenance
difficulties. This is the primary challenge that C++ based tools
addresses through template meta-programming, which is not available in C or
Fortran. Another obvious solution is to use macros to encode the
invariant arithmetic code. We have taken this approach, but instead of
using pre-processor macros, we built a more capable and versatile
macroprocessor described below.

.. _`Sec:mpuse`:

Using Macros
------------

A function in a physics unit typically has some
arithmetic interspersed with some control flow logic. Often the
features of the implementation that need to be modified for
optimization are data layouts and control flow without changing the arithmetic.
We begin by casting a non-trivial function in the physics unit
of the code as a collection of code blocks. Some code blocks may be
declarations, some may implement the control logic, and some will
implement the numerics of the function. Sometimes arithmetic and logic
blocks cannot be separated out so some code blocks have both. These code
blocks become **components** in a hierarchical composability through the use
of macros in the form of key-value dictionary where values are code
snippets of arbitrary length and complexity. Keys, or macros are user-defined
with a provision for multiple alternative definitions, including null
definitions, which lets them mimic specializations similar to those
provided by C++ template meta-programming.


The macros supported in |flashx| are similar to C preprocessor
macros in how they are defined. They permit arguments, are allowed to
be inlined in a regular programming language statement,  and one macro name
can be embedded in another macro's definition as long as self-reference
is avoided. Where are tool differs from the preprocessor is in
permitting alternative definitions, including null definitions for
macros, and an arbitration mechanism to select a specific definition.
The macroprocessor is written in python and the definitions are stored
in ``.ini'' files, according to the Python ConfigParser format. The
source files needing translation are given the extension ``.F90-mc''
to differentiate them from the source files that do not need
expansion.


Provision for alternative definitions combined with the ability to
embed a macro anywhere in the code permits arbitrary granularity in
code variants. The example shown below exhibits use of macros at
various granularities. In this example the CPU version of the code
uses 1D scratch arrays operating on one spatial data point at a time,
while the GPU version uses  4D scratch arrays operating on all spatial
data points simultaneously. Both versions otherwise have identical
logic for copying data into the target scratches. The macro *indices*
is the finest grain macro, with its null definition for the CPU
version allows unification of the scratch space reference in the
code. Similarly, macros *loop* and *loop_end* have null definitions
for the CPU version. Also, *hy_flux* is itself a macro which has other
other macros embedded in it. This approach can be particularly useful
when the involved arithmetic is complex but identical in both versions
and the only difference is in the data layout and access patterns. 

. container:: center

   .. figure:: macro-example.png
      :alt: macro-use
      :name: Fig:macro-use
      :width: 3in

	      
An additional feature is required if more than one variant needs
to be included in an application instance. It must be possible to
invoke each variant under different identifiers. For example, if a function
foo can generate two variants, var1 and var2, both of which need to be included
in the application,  then it must be possible to call them as
foo_var1 and foo_var2, but if they are exclusive in another
application instance then it should be possible to simply call
foo. Furthermore, with multiple alternative definitions existing for
macros, a mechanism is needed to arbitrate on which
variants are to be built with which definitions. The setup tool of
Flash-X already had arbitration built-in upto the granularity of
file. For macros arbitration is needed upto the level of a single
macro definition. The setup tool is enhance to keep track of macro
definitions in addition to files that it has encountered. The
inheritance rules remain the same as those for files, a local level
directory inherits all definitions and replaces those that are
present it any of its own ".ini" files. As with files, the definitions
placed in the Simulation directory of the application override all
definitions encountered earlier.



