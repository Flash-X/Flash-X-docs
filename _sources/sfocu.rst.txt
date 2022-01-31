.. include:: defs.h

.. _`Sec:visit`:

VisIt
=====

The developers of ``|flashx|`` also highly recommend VisIt, a free
parallel interactive visualization package provided by Lawrence
Livermore National Laboratory (see https://wci.llnl.gov/codes/visit/).
VisIt runs on Unix and PC platforms, and can handle small desktop-size
datasets as well as very large parallel datasets in the terascale range.
VisIt provides a native reader to import ``|flashx|2.5`` and |flashx|.
Version 1.10 and higher natively support |flashx|. For VisIt versions 1.8
or less, |flashx| support can be obtained by installing a tarball patch
available at
http://flash.uchicago.edu/site/flashcode/user_support/visit/. Full
instructions are also available at that site.

.. _`Chp:sfocu`:

Serial |flashx| Output Comparison Utility (``sfocu``)
====================================================

``Sfocu`` (Serial Flash Output Comparison Utility) is mainly used as
part of an automated testing suite called ``flashTest`` and was
introduced in |flashx| version 2.0 as a replacement for focu.

``Sfocu`` is a serial utility which examines two |flashx| checkpoint
files and decides whether or not they are “equal” to ensure that any
changes made to |flashx| do not adversely affect subsequent simulation
output. By “equal”, we mean that

-  The leaf-block structure matches – each leaf block must have the same
   position and size in both datasets.

-  The data arrays in the leaf blocks (``dens``, ``pres``...) are
   identical.

-  The number of particles are the same, and all floating point particle
   attributes are identical.

Thus, ``sfocu`` ignores information such as the particular numbering of
the blocks and particles, the timestamp, the build information, and so
on.

``Sfocu`` can read ``HDF5`` and ``PnetCDF`` |flashx| checkpoint files.
Although ``sfocu`` is a serial program, it is able to do comparisons on
the output of large parallel simulations. ``Sfocu`` has been used on
irix, linux, AIX and OSF1.

.. _`Sec:building sfocu`:

Building ``sfocu``
------------------

The process is entirely manual, although Makefiles for certain machines
have been provided. There are a few compile-time options which you set
via the following preprocessor definitions in the Makefile (in the
``CDEFINES`` macro):

``NO_HDF5``
   build without HDF5 support

``NO_NCDF``
   build without PnetCDF support

``NEED_MPI``
   certain parallel versions of HDF5 and all versions of PnetCDF need to
   be linked with the MPI library. This adds the necessary ``MPI_Init``
   and ``MPI_Finalize`` calls to ``sfocu``. There is no advantage to
   running ``sfocu`` on more than one processor; it will only give you
   multiple copies of the same report.

.. _`Sec:using sfocu`:

Using ``sfocu``
---------------

The basic and most common usage is to run the command
``sfocu <file1> <file2>``. The option ``-t <dist>`` allows a distance
tolerance in comparing bounding boxes of blocks in two different files
to determine which are the same (which have data to compare to one
another). You might need to widen your terminal to view the output,
since it can be over 80 columns. Sample output follows:

   .. container:: tiny

      .. container:: samepage

         ::

            A: 2006-04-25/sod_2d_45deg_4lev_ncmpi_chk_0001
            B: 2005-12-14/sod_2d_45deg_4lev_ncmpi_chk_0001
            Min Error: inf(2|a-b| / max(|a+b|, 1e-99) )
            Max Error: sup(2|a-b| / max(|a+b|, 1e-99) )
            Abs Error: sup|a-b|
            Mag Error: sup|a-b| / max(sup|a|, sup|b|, 1e-99)
            Block shapes for both files are: [8,8,1]
            Mag-error tolerance: 1e-12
            Total leaf blocks compared: 541 (all other blocks are ignored)
            -----+------------+-----------++-----------+-----------+-----------++-----------+-----------+-----------+
            Var  | Bad Blocks | Min Error ||             Max Error             ||             Abs Error             |
            -----+------------+-----------++-----------+-----------+-----------++-----------+-----------+-----------+
                 |            |           ||   Error   |     A     |     B     ||   Error   |     A     |     B     |
            -----+------------+-----------++-----------+-----------+-----------++-----------+-----------+-----------+
            dens | 502        | 0         || 1.098e-11 |  0.424    |  0.424    || 4.661e-12 |  0.424    |  0.424    |
            eint | 502        | 0         || 1.1e-11   |  1.78     |  1.78     || 1.956e-11 |  1.78     |  1.78     |
            ener | 502        | 0         || 8.847e-12 |  2.21     |  2.21     || 1.956e-11 |  2.21     |  2.21     |
            gamc | 0          | 0         || 0         |  0        |  0        || 0         |  0        |  0        |
            game | 0          | 0         || 0         |  0        |  0        || 0         |  0        |  0        |
            pres | 502        | 0         || 1.838e-14 |  0.302    |  0.302    || 1.221e-14 |  0.982    |  0.982    |
            temp | 502        | 0         || 1.1e-11   |  8.56e-09 |  8.56e-09 || 9.41e-20  |  8.56e-09 |  8.56e-09 |
            velx | 516        | 0         || 5.985     |  5.62e-17 | -1.13e-16 || 2.887e-14 |  0.657    |  0.657    |
            vely | 516        | 0         || 2         |  1e-89    | -4.27e-73 || 1.814e-14 |  0.102    |  0.102    |
            velz | 0          | 0         || 0         |  0        |  0        || 0         |  0        |  0        |
            mfrc | 0          | 0         || 0         |  0        |  0        || 0         |  0        |  0        |
            -----+------------+-----------++-----------+-----------+-----------++-----------+-----------+-----------+
            -----+------------+-----------++-----------+-----------+-----------++-----------+-----------+-----------+
            Var  | Bad Blocks | Mag Error ||                  A                ||                  B                |
            -----+------------+-----------++-----------+-----------+-----------++-----------+-----------+-----------+
                 |            |           ||    Sum    |    Max    |    Min    ||    Sum    |    Max    |    Min    |
            -----+------------+-----------++-----------+-----------+-----------++-----------+-----------+-----------+
            dens | 502        | 4.661e-12 ||  1.36e+04 |  1        |  0.125    ||  1.36e+04 |  1        |  0.125    |
            eint | 502        | 6.678e-12 ||  7.3e+04  |  2.93     |  1.61     ||  7.3e+04  |  2.93     |  1.61     |
            ener | 502        | 5.858e-12 ||  8.43e+04 |  3.34     |  2        ||  8.43e+04 |  3.34     |  2        |
            gamc | 0          | 0         ||  4.85e+04 |  1.4      |  1.4      ||  4.85e+04 |  1.4      |  1.4      |
            game | 0          | 0         ||  4.85e+04 |  1.4      |  1.4      ||  4.85e+04 |  1.4      |  1.4      |
            pres | 502        | 1.221e-14 ||  1.13e+04 |  1        |  0.1      ||  1.13e+04 |  1        |  0.1      |
            temp | 502        | 6.678e-12 ||  0.000351 |  1.41e-08 |  7.75e-09 ||  0.000351 |  1.41e-08 |  7.75e-09 |
            velx | 516        | 3.45e-14  ||  1.79e+04 |  0.837    | -6.09e-06 ||  1.79e+04 |  0.837    | -6.09e-06 |
            vely | 516        | 2.166e-14 ||  1.79e+04 |  0.838    | -1.96e-06 ||  1.79e+04 |  0.838    | -1.96e-06 |
            velz | 0          | 0         ||  0        |  0        |  0        ||  0        |  0        |  0        |
            mfrc | 0          | 0         ||  3.46e+04 |  1        |  1        ||  3.46e+04 |  1        |  1        |
            -----+------------+-----------++-----------+-----------+-----------++-----------+-----------+-----------+
            FAILURE

“Bad Blocks” is the number of leaf blocks where the data was found to
differ between datasets. Four different error measures (min/max/abs/mag)
are defined in the output above. In addition, the last six columns
report the sum, maximum and minimum of the variables in the two files.
Note that the sum is physically meaningless, since it is not
volume-weighted. Finally, the last line permits other programs to parse
the ``sfocu`` output easily: when the files are identical, the line will
instead read ``SUCCESS``.

It is possible for ``sfocu`` to miss machine-precision variations in the
data on certain machines because of compiler or library issues, although
this has only been observed on one platform, where the compiler produced
code that ignored IEEE rules until the right flag was found.

