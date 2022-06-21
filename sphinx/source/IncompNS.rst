.. include:: defs.h

.. _`Chp:IncompNS Unit`:

Incompressible Navier-Stokes Unit
=================================

The ``IncompNS`` unit solves incompressible Navier-Stokes equations in
two or three spatial dimensions. The currently released implementation
allows for both constant and variable density throughout the simulation domain.

Multistep and Runge-Kutta explicit projection schemes are used for time
integration. These methods are described in Armfield & Street 2002, Yang
& Balaras 2006, and Vanella *et al.* 2010. Implementations using a
staggered grid arrangement are provided for both uniform grid (UG) and
PARAMESH adaptive mesh refinement ``Grid`` implementations. The
``MultigridMC`` and ``BiPCGStab`` Poisson solvers con be employed for
AMR cases, whereas the homogeneous trigonometric solver + PFFT can be
used in UG. Typical velocity boundary conditions for this problem are
implemented.

*More documentation to appear later.*
