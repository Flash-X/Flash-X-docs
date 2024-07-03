.. include:: defs.h

.. _`Sec:Introduction`:

Introduction
============

The |flashx| code is a component-based software system for simulation of
multiphysics applications formulated largely as a collection of partial-
and ordinary- differential equations as well as algebraic equations. The
maintained code components are written in a combination of high level
languages such as Fortran, C and C++. |flashx| is accompanied by
several tools written in python or C++ that provide configurability
and performance portability to the code. The distribution includes
several existing application configurations, and tests for exercising
these configurations. A unit-test framework is also embedded in the
code. The code uses the Message-Passing Interface (MPI) library
communication between nodes, though more than 
one MPI rank can also be placed on a node. HDF5 is the default mode for
IO. |flashx| has three interchangeable discretization grids: a Uniform
Grid, a oct-tree based adaptive grid using the |paramesh|  library, and
a block-structured adaptive grid using |amrex| library, which also mimics
an oct-tree-like layout for use in |flashx|.

The precursor of  |flashx| is FLASH, which was developed at the
University of Chicago, and is now available from University of
Rochester.  |flashx| has been architected from the outset to be
compatible with increasing heterogeneity of both the platforms and the
solvers within the code. Flash-X distribution
includes solvers for compressible and incompressible fluids, and several other
solvers needed for astrophysics and fluid-structure interaction
applications. Not all physics and/or general solvers from FLASH have been
migrated to  |flashx|, however, the code itself has transitioned to open
development and community based governance. The expectation is that
the code will grow with community contributions. Additionally, some
external solvers can be included as add-ons in configuration of applications
because those solvers have been designed to be compatible with |flashx|. 
