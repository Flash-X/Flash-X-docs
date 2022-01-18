.. _`Sec:Introduction`:

Introduction
============

The Flash-X code is a component-based software system for simulation of
multiphysics applications formulated largely as a collection of partial-
and ordinary- differential equations as well as algebraic equations. The
maintained code components are written in a combination of high level
languages such as Fortran, C and C++ with an embedded domain-specific
macro language implemented in the form of key-value dictionaries. The
accompanying configuration tool-chain, written in Python and .... can
translate and assemble different permutations and combinations of the
components to configure a diverse set of applications. Also included in
the distribution is a accompanying domain-specific runtime system that
can orchestrate data movement between devices (CPU, accelerators, and
other specialized devices that might exist) on a compute node of a high
performance computing (HPC) platform. It uses the Message-Passing
Interface (MPI) library communication between nodes, though more than
one MPI rank can also be placed on a node. HDF5 is the default mode for
IO. has three interchangeable discretization grids: a Uniform Grid, a
oct-tree based adaptive grid using the library, and a block-structured
adaptive grid using AMReX library, which also mimics an oct-tree-like
layout for use in .

