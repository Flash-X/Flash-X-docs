.. include:: defs.h

.. _`Chp:IO`:

IO Unit
=======

|flashx| uses parallel input/output (IO) libraries to simplify and manage
the output of the large amounts of data usually produced. In addition to
keeping the data output in a standard format, the parallel IO libraries
also ensure that files will be portable across various platforms. The
mapping of |flashx| data-structures to records in these files is
controlled by the |flashx| IO unit. |flashx| can output data with HDF5
parallel IO library. Various techniques can be used to write the data to
disk when running a parallel simulation. The first is to move all the
data to a single processor for output; this technique is known as serial
IO. Secondly, each processor can write to a separate file, known as
direct IO. As a third option, each processor can use parallel access to
write to a single file in a technique known as parallel IO. Finally, a
hybrid method can be used where clusters of processors write to the same
file, though different clusters of processors output to different files.
In general, parallel access to a single file will provide the best
parallel IO performance unless the number of processors is very large.
On some platforms, such as Linux clusters, there may not be a parallel
file system, so moving all the data to a single processor is the only
solution. Therefore |flashx| supports HDF5 libraries in both serial and
parallel forms, where the serial version collects data to one processor
before writing it, while the parallel version has every processor
writing its data to the same file.

.. _`Sec:|flashx| output formats`:

IO Implementations
------------------

|flashx| supports multiple IO implementations: direct, serial and
parallel implementations as well as support for different parallel
libraries. In addition, |flashx| also supports multiple () ``Grid``
implementations. As a consequence, there are many permutations of the IO
API implementation, and the selected implementation must match not only
the correct IO library, but also the correct grid. Although there are
many IO options, the ``setup`` script in |flashx| is quite ‘smart’ and
will not let the user setup a problem with incompatible ``IO`` and
``Grid`` unit implementations. summarizes the different implementation
of the |flashx| IO unit in the current release.

.. container:: longtable

   p2.5inp3.5in

   | 
   | Implementation Path & Description

   | Implementation path & Description ``IO/IOMain/HDF5/parallel/PM``
     &Hierarchical Data Format (HDF) 5 output. A single HDF5 file is
     created, with each processor writing its data to the same file
     simultaneously. This particular implementation only works with the
     ``PARAMESH`` grid package.
   | ``IO/IOMain/HDF5/parallel/AM`` &Hierarchical Data Format (HDF) 5
     output. A single HDF5 file is created, with each processor writing
     its data to the same file simultaneously. This particular
     implementation only works with the ``|amrex|``\ grid package.
   | ``IO/IOMain/hdf5/parallel/UG`` &Hierarchical Data Format (HDF) 5
     output. A single HDF5 file is created, with each processor writing
     its data to the same file simultaneously. This implementation only
     works with the Uniform Grid.
   | ``IO/IOMain/hdf5/parallel/NoFbs`` &Hierarchical Data Format (HDF) 5
     output. A single HDF5 file is created, with each processor writing
     its data to the same file simultaneously. All data is written out
     as one block. This implementation only works with the Uniform Grid.
   | ``IO/IOMain/hdf5/serial/PM`` & Hierarchical Data Format (HDF) 5
     output. Each processor passes its data to processor 0 through
     explicit MPI sends and receives. Processor 0 does all of the
     writing. The resulting file format is identical to the parallel
     version; the only difference is how the data is moved during the
     writing. This implementation only works with the ``PARAMESH`` grid
     package.
   | ``IO/IOMain/hdf5/serial/AM`` & Hierarchical Data Format (HDF) 5
     output. Each processor passes its data to processor 0 through
     explicit MPI sends and receives. Processor 0 does all of the
     writing. The resulting file format is identical to the parallel
     version; the only difference is how the data is moved during the
     writing. This implementation only works with the ``|amrex|``\ grid
     package.
   | ``IO/IOMain/hdf5/serial/UG`` & Hierarchical Data Format (HDF) 5
     output. Each processor passes its data to processor 0 through
     explicit MPI sends and receives. Processor 0 does all of the
     writing. The resulting file format is identical to the parallel
     version; the only difference is how the data is moved during the
     writing. This particular implementation only works with the Uniform
     Grid.

|flashx| also comes with some predefined setup **shortcuts** which make
choosing the correct IO significantly easier; see for more details about
shortcuts. In |flashx| HDF5 serial IO is included by default. Since
``PARAMESH`` 4.0 is the default grid, the included IO implementations
will be compatible with ``PARAMESH`` 4.0. For clarity, a number or
examples are shown below.

An example of a basic setup with HDF5 serial IO and the ``PARAMESH``
grid, (both defaults) is:

.. container:: codeseg

   ./setup Sod -2d -auto

To include a parallel implementation of HDF5 for a ``PARAMESH`` grid the
``setup`` syntax is:

.. container:: codeseg

   ./setup Sod -2d -auto -unit=IO/IOMain/hdf5/parallel/PM

using the already defined shortcuts the ``setup`` line can be shortened
to

.. container:: codeseg

   ./setup Sod -2d -auto +parallelio

To set up a problem with the Uniform Grid and HDF5 serial IO, the
``setup`` line is:

.. container:: codeseg

   ./setup Sod -2d -auto -unit=Grid/GridMain/UG
   -unit=IO/IOMain/hdf5/serial/UG

using the already defined shortcuts the ``setup`` line can be shortened
to

.. container:: codeseg

   ./setup Sod -2d -auto +ug

To set up a problem with the Uniform Grid and HDF5 parallel IO, the
complete ``setup`` line is:

.. container:: codeseg

   ./setup Sod -2d -auto -unit=Grid/GridMain/UG
   -unit=IO/IOMain/hdf5/parallel/UG

using the already defined shortcuts the ``setup`` line can be shortened
to

.. container:: codeseg

   ./setup Sod -2d -auto +ug +parallelio

If you do *not* want to use IO, you need to *explicitly* specify on the
``setup`` line that it should not be included, as in this example:

.. container:: codeseg

   ./setup Sod -2d -auto +noio

To setup a problem using the Parallel-NetCDF library the user should
include either

.. container:: codeseg

   -unit=IO/IOMain/pnetcdf/PM or -unit=IO/IOMain/pnetcdf/UG

to the setup line. The predefined shortcut for including the
Parallel-NetCDF library is

.. container:: codeseg

   +pnetcdf

Note that Parallel-NetCDF IO unit does not have a serial implementation.

If you are using non-fixedblocksize the shortcut

.. container:: codeseg

   +nofbs

will bring in both Uniform Grid,set the mode to nonfixed blocksize, and
choose the appropriate IO.

In keeping with the |flashx| code architecture, the F90 module
``IO_data`` stores all the data with ``IO`` unit scope. The routine
``IO/IO_init`` is called once by ``Driver/Driver_initFlash`` and
initializes ``IO`` data and stores any runtime parameters. See .

Output Files
------------

The IO unit can output 4 different types of files: checkpoint files,
plotfiles, particle files and flash.dat, a text file holding the
integrated grid quantities. |flashx| also outputs a logfile, but this
file is controlled by the Logfile Unit; see for a description of that
format.

There are a number of runtime parameters that are used to control the
output and frequency of IO files. A list of all the runtime parameters
and their descriptions for the ``IO`` unit can be found online
````\ IO/all of them. Additional description is located in for
checkpoint parameters, for plotfile parameters, for particle file
parameters, for flash.dat parameters, and for genereal IO parameters.

Checkpoint files - Restarting a Simulation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Checkpoint files are used to restart a simulation. In a typical
production run, a simulation can be interrupted for a number of reasons—
*e.g.*, if the machine crashes, the present queue window closes, the
machine runs out of disk space, or perhaps (gasp) there is a bug in
|flashx|. Once the problem is fixed, a simulation can be restarted from
the last checkpoint file rather than the beginning of the run. A
checkpoint file contains all the information needed to restart the
simulation. The data is stored at full precision of the code (8-byte
reals) and includes all of the variables, species, grid reconstruction
data, scalar values, as well as meta-data about the run.

The API routine for writing a checkpoint file is
``IO/IO_writeCheckpoint``. Users usually will not need to call this
routine directly because the |flashx| IO unit calls
``IO_writeCheckpoint`` from the routine ``IO/IO_output`` which checks
the runtime parameters to see if it is appropriate to write a checkpoint
file at this time. There are a number of ways to get |flashx| to produce
a checkpoint file for restarting. Within the flash.par, runtime
parameters can be set to dump output. A checkpoint file can be dumped
based on elapsed simulation time, elapsed wall clock time or the number
of timesteps advanced. A checkpoint file is also produced when the
simulation ends, when the max simulation time ``Driver/tmax``, the
minimum cosmological redshift, or the total number of steps
``Driver/nend`` has been reached. A user can force a dump to a
checkpoint file at another time by creating a file named
``.dump_checkpoint`` in the output directory of the master processor.
This manual action causes |flashx| to write a checkpoint in the next
timestep. Checkpoint files will continue to be dumped after every
timestep as long as the code finds a ``.dump_checkpoint`` file in the
output directory, so the user must remember to remove the file once all
the desired checkpoint files have been dumped. Creating a file named
``.dump_restart`` in the output directory will cause |flashx| to output a
checkpoint file and then stop the simulation. This technique is useful
for producing one last checkpoint file to save time evolution since the
last checkpoint, if the machine is going down or a queue window is about
to end. These different methods can be combined without problems. Each
counter (number of timesteps between last checkpoint, amount of
simulation time single last checkpoint, the change in cosmological
redshift, and the amount of wall clock time elapsed since the last
checkpoint) is independent of the others, and are not influenced by the
use of a ``.dump_checkpoint`` or ``.dump_restart``.

Runtime Parameters used to control checkpoint file output include:

.. container:: center

   .. container::
      :name: Tab:checkpoint parameters

      .. table:: Checkpoint IO parameters (continued).

         +------------------+-------------+---------------+------------------+
         | Parameter        | Type        | Default value | Description      |
         +==================+=============+===============+==================+
         |                  | Type        | Default value | Description      |
         +------------------+-------------+---------------+------------------+
         | ``checkp         | ``INTEGER`` | ``0``         | The number of    |
         | ointFileNumber`` |             |               | the initial      |
         |                  |             |               | checkpoint file. |
         |                  |             |               | This number is   |
         |                  |             |               | appended to the  |
         |                  |             |               | end of the       |
         |                  |             |               | filename and     |
         |                  |             |               | incremented at   |
         |                  |             |               | each subsequent  |
         |                  |             |               | output. When     |
         |                  |             |               | restarting a     |
         |                  |             |               | simulation, this |
         |                  |             |               | indicates which  |
         |                  |             |               | checkpoint file  |
         |                  |             |               | to use.          |
         +------------------+-------------+---------------+------------------+
         | ``checkpointFi   | ``INTEGER`` | ``0``         | The number of    |
         | leIntervalStep`` |             |               | timesteps        |
         |                  |             |               | desired between  |
         |                  |             |               | subsequent       |
         |                  |             |               | checkpoint       |
         |                  |             |               | files.           |
         +------------------+-------------+---------------+------------------+
         |                  |             |               |                  |
         +------------------+-------------+---------------+------------------+
         | ``checkpointFi   | ``REAL``    | ``1.``        | The amount of    |
         | leIntervalTime`` |             |               | simulation time  |
         |                  |             |               | desired between  |
         |                  |             |               | subsequent       |
         |                  |             |               | checkpoint       |
         |                  |             |               | files.           |
         +------------------+-------------+---------------+------------------+
         | ``checkpoin      | ``REAL``    | ``HUGE(1.)``  | The amount of    |
         | tFileIntervalZ`` |             |               | cosmological     |
         |                  |             |               | redshift change  |
         |                  |             |               | that is desired  |
         |                  |             |               | between          |
         |                  |             |               | subsequent       |
         |                  |             |               | checkpoint       |
         |                  |             |               | files.           |
         +------------------+-------------+---------------+------------------+
         |                  |             |               |                  |
         +------------------+-------------+---------------+------------------+
         | ``roll           | ``INTEGER`` | 10000         | The number of    |
         | ing_checkpoint`` |             |               | checkpoint files |
         |                  |             |               | to keep          |
         |                  |             |               | available at any |
         |                  |             |               | point in the     |
         |                  |             |               | simulation. If a |
         |                  |             |               | checkpoint       |
         |                  |             |               | number is        |
         |                  |             |               | greater than     |
         |                  |             |               | ``rolli          |
         |                  |             |               | ng_checkpoint``, |
         |                  |             |               | then the         |
         |                  |             |               | checkpoint       |
         |                  |             |               | number is reset  |
         |                  |             |               | to 0. There will |
         |                  |             |               | be at most       |
         |                  |             |               | ``roll           |
         |                  |             |               | ing_checkpoint`` |
         |                  |             |               | checkpoint files |
         |                  |             |               | kept. This       |
         |                  |             |               | parameter is     |
         |                  |             |               | intended to be   |
         |                  |             |               | used when disk   |
         |                  |             |               | space is at a    |
         |                  |             |               | premium.         |
         +------------------+-------------+---------------+------------------+
         |                  |             |               |                  |
         +------------------+-------------+---------------+------------------+
         | ``wall_cl        | ``REAL``    | 43200.        | The maximum      |
         | ock_checkpoint`` |             |               | amount of wall   |
         |                  |             |               | clock time       |
         |                  |             |               | (seconds) to     |
         |                  |             |               | elapse between   |
         |                  |             |               | checkpoints.     |
         |                  |             |               | When the         |
         |                  |             |               | simulation is    |
         |                  |             |               | started, the     |
         |                  |             |               | current time is  |
         |                  |             |               | stored. If       |
         |                  |             |               | ``wall_cl        |
         |                  |             |               | ock_checkpoint`` |
         |                  |             |               | seconds elapse   |
         |                  |             |               | over the course  |
         |                  |             |               | of the           |
         |                  |             |               | simulation, a    |
         |                  |             |               | checkpoint file  |
         |                  |             |               | is stored. This  |
         |                  |             |               | is useful for    |
         |                  |             |               | ensuring that a  |
         |                  |             |               | checkpoint file  |
         |                  |             |               | is produced      |
         |                  |             |               | before a queue   |
         |                  |             |               | closes.          |
         +------------------+-------------+---------------+------------------+
         |                  |             |               |                  |
         +------------------+-------------+---------------+------------------+
         | ``restart``      | ``BOOLEAN`` | ``.false.``   | A logical        |
         |                  |             |               | variable         |
         |                  |             |               | indicating       |
         |                  |             |               | whether the      |
         |                  |             |               | simulation is    |
         |                  |             |               | restarting from  |
         |                  |             |               | a checkpoint     |
         |                  |             |               | file             |
         |                  |             |               | (``.true.``) or  |
         |                  |             |               | starting from    |
         |                  |             |               | scratch          |
         |                  |             |               | (``.false.``).   |
         +------------------+-------------+---------------+------------------+

|flashx| is capable of restarting from any of the checkpoint files it
produces. The user should make sure that the checkpoint file is valid
(*e.g.*, the code did not stop while outputting). To tell |flashx| to
restart, set the ``Driver/restart`` runtime parameter to ``.true.`` in
the ``flash.par``. Also, set ``IO/checkpointFileNumber`` to the number
of the file from which you wish to restart. If plotfiles or particle
files are being produced set ``IO/plotfileNumber`` and
``IO/particleFileNumber`` to the number of the *next* plotfile and
particle file you want |flashx| to output. In |flashx| plotfiles and
particle file outputs are forced whenever a checkpoint file is written.
Sometimes several plotfiles may be produced after the last valid
checkpoint file. Resetting ``plotfileNumber`` to the first plotfile
produced after the checkpoint from which you are restarting will ensure
that there are no gaps in the output. See

for more details on plotfiles.

.. _`Sec:Plotfiles`:

Plotfiles
~~~~~~~~~

A plotfile contains all the information needed to interpret the grid
data maintained by |flashx|. The data in plotfiles, including the grid
metadata such as coordinates and block sizes, are stored at single
precision to preserve space. This can, however, be overridden by setting
the runtime parameters ``plotfileMetadataDP`` and/or
``plotfileGridQuantityDP`` to true to set the grid metadata and the
quantities stored on the grid (dens, pres, temp, etc.) to use double
precision, respectively. Users must choose which variables to output
with the runtime parameters ``IO/plot_var_1``, ``IO/plot_var_2``,
*etc.*, by setting them in the ``flash.par`` file. For example:

.. container:: codeseg

   plot_var_1 = "dens" plot_var_2 = "pres"

Currently, we support a number of plotvars named ``plot_var_n`` up to
the number of ``UNKVARS`` in a given simulation. Similarly, scratch
variables may be output to plot files . At this time, the plotting of
face centered quantities is not supported.

.. container:: flashtip

   In |flashx| a few variables like density and pressure were output to
   the plotfiles by default. Because |flashx| supports a wider range of
   simulations, it makes no assumptions that density or pressure
   variables are even included in the simulation. In |flashx| a user
   *must* define plotfile variables in the ``flash.par`` file, otherwise
   the plotfiles will not contain any variables.

| The interface for writing a plotfile is the routine
  ``IO/IO_writePlotfile``. As with checkpoint files, the user will not
  need to call this routine directly because it is invoked indirectly
  through calling ``IO/IO_output`` when, based on runtime parameters,
  |flashx| needs to write a plotfile. |flashx| can produce plotfiles in
  much the same manner as it does with checkpoint files. They can be
  dumped based on elapsed simulation time, on steps since the last
  plotfile dump or by forcing a plotfile to be written by hand by
  creating a\ ``.dump_plotfile`` in the output directory. A plotfile
  will also be written at the termination of a simulation as well.
| If plotfiles are being kept at particular intervals (such as time
  intervals) for purposes such as visualization or analysis, it is also
  possible to have |flashx| denote a plotfile as “forced". This
  designation places the word forced between the basename and the file
  format type identifier (or the split number if splitting is used).
  These files are numbered separately from normal plotfiles. By default,
  plotfiles are considered forced if output for any reason other than
  the change in simulation time, change in cosmological redshift, change
  in step number, or the termination of a simulation from reaching
  ``nend`` , ``zFinal``, or ``tmax``. This option can be disabled by
  setting ``ignoreForcedPlot`` to true in a simulations ``flash.par``
  file. The following runtime parameters pertain to controlling
  plotfiles:

.. container:: center

   .. container::
      :name: Tab:plotfile parameters

      .. table:: Plotfile IO parameters (continued).

         +------------------+-------------+---------------+------------------+
         | Parameter        | Type        | Default value | Description      |
         +==================+=============+===============+==================+
         |                  | Type        | Default value | Description      |
         +------------------+-------------+---------------+------------------+
         | ``               | ``INTEGER`` | ``0``         | The number of    |
         | plotFileNumber`` |             |               | the starting (or |
         |                  |             |               | restarting)      |
         |                  |             |               | plotfile. This   |
         |                  |             |               | number is        |
         |                  |             |               | appended to the  |
         |                  |             |               | filename.        |
         +------------------+-------------+---------------+------------------+
         |                  |             |               |                  |
         +------------------+-------------+---------------+------------------+
         | ``plotFi         | ``REAL``    | ``1.``        | The amount of    |
         | leIntervalTime`` |             |               | simulation time  |
         |                  |             |               | desired between  |
         |                  |             |               | subsequent       |
         |                  |             |               | plotfiles.       |
         +------------------+-------------+---------------+------------------+
         | ``plotFi         | ``INTEGER`` | ``0``         | The number of    |
         | leIntervalStep`` |             |               | timesteps        |
         |                  |             |               | desired between  |
         |                  |             |               | subsequent       |
         |                  |             |               | plotfiles.       |
         +------------------+-------------+---------------+------------------+
         |                  |             |               |                  |
         +------------------+-------------+---------------+------------------+
         | ``plo            | ``INTEGER`` | ``HUGE(1.)``  | The change in    |
         | tFileIntervalZ`` |             |               | cosmological     |
         |                  |             |               | redshift desired |
         |                  |             |               | between          |
         |                  |             |               | subsequent       |
         |                  |             |               | plotfiles.       |
         +------------------+-------------+---------------+------------------+
         |                  |             |               |                  |
         +------------------+-------------+---------------+------------------+
         |                  | ``BOOLEAN`` | ``.false.``   | A logical        |
         |                  |             |               | variable         |
         |                  |             |               | indicating       |
         |                  |             |               | whether to       |
         |                  |             |               | interpolate the  |
         |                  |             |               | data to cell     |
         |                  |             |               | corners before   |
         |                  |             |               | outputting. This |
         |                  |             |               | option only      |
         |                  |             |               | applies to       |
         |                  |             |               | plotfiles.       |
         +------------------+-------------+---------------+------------------+
         |                  |             |               |                  |
         +------------------+-------------+---------------+------------------+
         |                  | ``STRING``  | ``"none"``    | Name of the      |
         |                  |             |               | variables to     |
         |                  |             |               | store in a       |
         |                  |             |               | plotfile. Up to  |
         |                  |             |               | 12 variables can |
         |                  |             |               | be selected for  |
         |                  |             |               | storage, and the |
         |                  |             |               | standard         |
         |                  |             |               | 4-character      |
         |                  |             |               | variable name    |
         |                  |             |               | can be used to   |
         |                  |             |               | select them.     |
         +------------------+-------------+---------------+------------------+
         | ``ig             | ``BOOLEAN`` | ``.false.``   | A logical        |
         | noreForcedPlot`` |             |               | variable         |
         |                  |             |               | indicating       |
         |                  |             |               | whether or not   |
         |                  |             |               | to denote        |
         |                  |             |               | certain          |
         |                  |             |               | plotfiles as     |
         |                  |             |               | forced.          |
         +------------------+-------------+---------------+------------------+
         | ``forced         | ``INTEGER`` | ``0``         | An integer that  |
         | PlotfileNumber`` |             |               | sets the         |
         |                  |             |               | starting number  |
         |                  |             |               | for a forced     |
         |                  |             |               | plotfile.        |
         +------------------+-------------+---------------+------------------+
         | ``plot           | ``BOOLEAN`` | ``.false.``   | A logical        |
         | fileMetadataDP`` |             |               | variable         |
         |                  |             |               | indicating       |
         |                  |             |               | whether or or    |
         |                  |             |               | not to output    |
         |                  |             |               | the normally     |
         |                  |             |               | single-precision |
         |                  |             |               | grid metadata    |
         |                  |             |               | fields as double |
         |                  |             |               | precision in     |
         |                  |             |               | plotfiles. This  |
         |                  |             |               | specifically     |
         |                  |             |               | affects          |
         |                  |             |               | ``coordinates``, |
         |                  |             |               | ``block size``,  |
         |                  |             |               | and              |
         |                  |             |               | `                |
         |                  |             |               | `bounding box``. |
         +------------------+-------------+---------------+------------------+
         | ``plotfile       | ``BOOLEAN`` | ``.false.``   | A logical        |
         | GridQuantityDP`` |             |               | variable that    |
         |                  |             |               | sets whether or  |
         |                  |             |               | not quantities   |
         |                  |             |               | stored on the    |
         |                  |             |               | grid, such as    |
         |                  |             |               | those stored in  |
         |                  |             |               | ``unk``, are     |
         |                  |             |               | output in single |
         |                  |             |               | precision or     |
         |                  |             |               | double precision |
         |                  |             |               | in plotfiles.    |
         +------------------+-------------+---------------+------------------+
         |                  |             |               |                  |
         +------------------+-------------+---------------+------------------+
         |                  |             |               |                  |
         +------------------+-------------+---------------+------------------+

.. _`Sec:Particle files`:

Particle files
~~~~~~~~~~~~~~

When Lagrangian particles are included in a simulation, the ParticleIO
subunit controls input and output of the particle information. The
particle files are stored in double precision. Particle data is written
to the checkpoint file in order to restart the simulation, but is not
written to plotfiles. Hence analysis and metadata about particles is
also written to the particle files. The particle files are intended for
more frequent dumps. The interface for writing the particle file is
``IO/IO_writeParticles``. Again the user will not usually call this
function directly because the routine ``IO_output`` controls particle
output based on the runtime parameters controlling particle files. They
are controlled in much of the same way as the plotfiles or checkpoint
files and can be dumped based on elapsed simulation time, on steps since
the last particle dump or by forcing a particle file to be written by
hand by creating a ``.dump_particle_file`` in the output directory. The
following runtime parameters pertain to controlling particle files:

.. container:: center

   .. container::
      :name: Tab:particle file parameters

      .. table:: Particle File IO runtime parameters.

         +------------------+-------------+---------------+------------------+
         | Parameter        | Type        | Default value | Description      |
         +==================+=============+===============+==================+
         |                  | Type        | Default value | Description      |
         +------------------+-------------+---------------+------------------+
         | ``part           | ``INTEGER`` | ``0``         | The number of    |
         | icleFileNumber`` |             |               | the starting (or |
         |                  |             |               | restarting)      |
         |                  |             |               | particle file.   |
         |                  |             |               | This number is   |
         |                  |             |               | appended to the  |
         |                  |             |               | end of the       |
         |                  |             |               | filename.        |
         +------------------+-------------+---------------+------------------+
         |                  |             |               |                  |
         +------------------+-------------+---------------+------------------+
         | ``particleFi     | ``REAL``    | ``1.``        | The amount of    |
         | leIntervalTime`` |             |               | simulation time  |
         |                  |             |               | desired between  |
         |                  |             |               | subsequent       |
         |                  |             |               | particle file    |
         |                  |             |               | dumps.           |
         +------------------+-------------+---------------+------------------+
         |                  |             |               |                  |
         +------------------+-------------+---------------+------------------+
         |                  |             |               |                  |
         +------------------+-------------+---------------+------------------+
         | ``particleFi     | ``INTEGER`` | ``0``         | The number of    |
         | leIntervalStep`` |             |               | timesteps        |
         |                  |             |               | desired between  |
         |                  |             |               | subsequent       |
         |                  |             |               | particle file    |
         |                  |             |               | dumps.           |
         +------------------+-------------+---------------+------------------+
         |                  |             |               |                  |
         +------------------+-------------+---------------+------------------+
         | ``particl        | ``REAL``    | ``HUGE(1.)``  | The change in    |
         | eFileIntervalZ`` |             |               | cosmological     |
         |                  |             |               | redshift desired |
         |                  |             |               | between          |
         |                  |             |               | subsequent       |
         |                  |             |               | particle file    |
         |                  |             |               | dumps.           |
         +------------------+-------------+---------------+------------------+
         |                  |             |               |                  |
         +------------------+-------------+---------------+------------------+

All the code necessary to output particle data is contained in the
``IO`` subunit called ``IOParticles``. Whenever the ``Particles`` unit
is included in a simulation the correct ``IOParticles`` subunit will
also be included. For example as setup:

.. container:: codeseg

   ./setup IsentropicVortex -2d -auto -unit=Particles +ug

will include the ``IO`` unit ``IO/IOMain/hdf5/serial/UG`` and the
correct ``IOParticles`` subunit ``IO/IOParticles/hdf5/serial/UG``. The
shortcuts ``+parallelio``, ``+pnetcdf``, ``+ug`` will also cause the
setup script to pick up the correct ``IOParticles`` subunit as long as a
``Particles`` unit is included in the simulation.

Integrated Grid Quantities – flash.dat
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

At each simulation time step, values which represent the overall state
(*e.g.*, total energy and momentum) are computed by calculating over all
cells in the computations domain. These integral quantities are written
to the ASCI file ``flash.dat``. A default routine
``IO/IO_writeIntegralQuantities`` is provided to output standard
measures for hydrodynamic simulations. The user should copy and modify
the routine ``IO_writeIntegralQuantities`` into a given simulation
directory to store any quantities other than the default values. Two
runtime parameters pertaining to the ``flash.dat`` file are listed in
the table below.

.. container:: center

   .. container::
      :name: Tab:flash.dat parameters

      .. table:: flash.dat runtime parameters.

         +-----------------+-------------+-----------------+-----------------+
         | Parameter       | Type        | Default value   | Description     |
         +=================+=============+=================+=================+
         |                 | Type        | Default value   | Description     |
         +-----------------+-------------+-----------------+-----------------+
         | ``stats_file``  | ``STRING``  | ``"flash.dat"`` | Name of the     |
         |                 |             |                 | file to which   |
         |                 |             |                 | the integral    |
         |                 |             |                 | quantities are  |
         |                 |             |                 | written.        |
         +-----------------+-------------+-----------------+-----------------+
         |                 |             |                 |                 |
         +-----------------+-------------+-----------------+-----------------+
         | ``wr_i          | ``INTEGER`` | ``1``           | The number of   |
         | ntegrals_freq`` |             |                 | timesteps to    |
         |                 |             |                 | elapse between  |
         |                 |             |                 | outputs to the  |
         |                 |             |                 | scalar/integral |
         |                 |             |                 | data file       |
         |                 |             |                 | (``flash.dat``) |
         +-----------------+-------------+-----------------+-----------------+

General Runtime Parameters
~~~~~~~~~~~~~~~~~~~~~~~~~~

There are several runtime parameters that pertain to the general IO unit
or multiple output files rather than one particular output file. They
are listed in the table below.

.. container:: center

   .. container:: longtable

      p1.7inllp2.7in

      | 
      |  Parameter & Type & Default value & Description
      | Parameter & Type & Default value & Description

      | ``basenm`` & ``STRING`` & ``"flash_"`` & The main part of the
        output filenames. The full filename consists of the base name, a
        series of three-character abbreviations indicating whether it is
        a plotfile, particle file or checkpoint file, the file format,
        and a 4-digit file number. See for a description of how |flashx|
        output files are named.

      | ``output_directory`` & ``STRING`` & ``""`` & Output directory
        for plotfiles, particle files and checkpoint files. The default
        is the directory in which the executable sits.
        ``output_directory`` can be an absolute or relative path.
      | ``memory_stat_freq`` & ``INTEGER`` & ``100000`` & The number of
        timesteps to elapse between memory statistic dumps to the log
        file (``flash.log``).
      | ``useCollectiveHDF5`` &\ ``BOOLEAN``\ &\ ``.true.`` & When using
        the parallel HDF5 implementation of IO, will enable collective
        mode for HDF5.
      | ``summaryOutputOnly`` &\ ``BOOLEAN``\ &\ ``.false.`` & When set
        to .true. write an integrated grid quantities file only.
        Checkpoint, plot and particle files are not written unless the
        user creates a .dump_plotfile, .dump_checkpoint, .dump_restart
        or .dump_particle file.

.. _`Sec:runtime parameters`:

Restarts and Runtime Parameters
-------------------------------

|flashx| outputs the runtime parameters of a simulation to all checkpoint
files. When a simulation is restarted, these values are known by the
``RuntimeParameters`` unit while the code is running. On a restart, all
values from the checkpoint used in the restart are stored as previous
values in the lists kept by the ``RuntimeParameters`` unit. All current
values are taken from the defaults used by |flashx| and any simulation
parameter files (*e.g.*, ``flash.par``). If needed, the previous values
from the checkpoint file can be obtained using the routines
``RuntimeParameters/RuntimeParameters_getPrev``.

.. _`Sec:output scalars`:

Output Scalars
--------------

In |flashx|, each unit has the opportunity to request scalar data to be
output to checkpoint or plotfiles. Because there is no central database,
each unit “owns" different data in the simulation. For example, the
``Driver`` unit owns the timestep variable ``dt``, the simulation
variable ``simTime``, and the simulation step number ``nStep``. The
``Grid`` unit owns the sizes of each block, ``nxb``, ``nyb``, and
``nzb``. The ``IO`` unit owns the variable ``checkpointFileNumber``.
Each of these quantities are output into checkpoint files. Instead of
hard coding the values into checkpoint routines, |flashx| offers a more
flexible interface whereby each unit sends its data to the ``IO`` unit.
The ``IO`` unit then stores these values in a linked list and writes
them to the checkpoint file or plotfile. Each unit has a routine called
“\ ``Unit_sendOutputData``", *e.g.*, ``Driver/Driver_sendOutputData``
and ``Grid/Grid_sendOutputData``. These routines in turn call
``IO/IO_setScalar``. For example, the routine
``Grid/Grid_sendOutputData`` calls

.. container:: codeseg

   IO_setScalar("nxb", NXB) IO_setScalar("nyb", NYB) IO_setScalar("nzb",
   NZB)

To output additional simulation scalars in a checkpoint file, the user
should override one of the “\ ``Unit_sendOutputData``" or
``Simulation_sendOutputData``.

After restarting a simulation from a checkpoint file, a unit might call
``IO/IO_getScalar`` to reset a variable value. For example, the
``Driver`` unit calls ``IO_getScalar("dt", dr_dt)`` to get the value of
the timestep ``dt`` reinitialized from the checkpoint file. A value from
the checkpoint file can be obtained by calling ``IO/IO_getPrevScalar``.
This call can take an optional argument to find out if an error has
occurred in finding the previous value, most commonly because the value
was not found in the checkpoint file. By using this argument, the user
can then decide what to do if the value is not found. If the scalar
value is not found and the optional argument is not used, then the
subroutine will call ``Driver/Driver_abortFlash`` and terminate the run.

.. _`Sec: Output user defined arrays`:

Output User-defined Arrays
--------------------------

Often in a simulation the user needs to output additional information to
a checkpoint or plotfile which is not a grid scope variable. In |flashx|
any additional information had to be hard coded into the simulation. In
|flashx|, we have provided a general interface ``IO/IO_writeUserArray``
and ``IO/IO_readUserArray`` which allows the user to write and read any
generic array needed to be stored. The above two functions do not have
any implementation and it is up to the user to fill them in with the
needed calls to the HDF5 or pnetCDF C routines. We provide
implementation for reading and writing integer and double precision
arrays with the helper routines ``io_h5write_generic_iarr``,
``io_h5write_generic_rarr``, ``io_ncmpi_write_generic_iarr``, and
``io_ncmpi_write_generic_rarr``. Data is written out as a 1-dimensional
array, but the user can write multidimensional arrays simply by passing
a reference to the data and the total number of elements to write. See
these routines and the simulation ``StirTurb`` for details on their
usage.

.. _`lbl:OutputScratchVariables`:

Output Scratch Variables
------------------------

In |flashx| a user can allocate space for a scratch or temporary variable
with grid scope using one of the ``Config`` keywords ``SCRATCHVAR``,
``SCRATCHCENTERVAR``, ``SCRATCHFACEXVAR``,\ ``SCRATCHFACEYVAR`` or
``SCRATCHFACEZVAR`` (see ). To output these scratch variables, the user
only needs to set the values of the runtime parameters
``IO/plot_grid_var_1``, ``IO/plot_grid_var_2``, *etc.*, by setting them
in the ``flash.par`` file. For example to output the magnitude of
vorticity with a declaration in a ``Config`` file of
``SCRATCHVAR mvrt``:

.. container:: codeseg

   plot_grid_var_1 = "mvrt"

Note that the post-processing routines like ``fidlr`` do not display
these variables, although they are present in the output file. Future
implementations may support this visualization.

Face-Centered Data
------------------

Face-centered variables are now output to checkpoint files, when they
are declared in a configuration file. Presently, up to nine
face-centered variables are supported in checkpoint files. Plotfile
output of face-centered data is not yet supported.

.. _`Sec:Output file names`:

Output Filenames
----------------

|flashx| constructs the output filenames based on the user-supplied
basename, (runtime parameter ``basenm``) and the file counter that is
incremented after each output. Additionally, information about the file
type and data storage is included in the filename. The general
checkpoint filename is:

``basename_s0000_\left\{\begin{array}{c}\mathtt{hdf5}\\ \mathtt{ncmpi}\\
             \end{array}\right\}_chk_0000``,

where ``hdf5`` or ``ncmpi`` (prefix for PnetCDF) is picked depending on
the particular IO implementation, the number following the “s” is the
split file number, if split file IO is in use, and the number at the end
of the filename is the current checkpointFileNumber. (The PnetCDF
function prefix "``ncmpi``" derived from the serial NetCDF calls
beginning with "``nc``")

The general plotfile filename is:

``basename_s0000_\left\{\begin{array}{c}
                                       \mathtt{hdf5}\\
                                       \mathtt{ncmpi}\\
                                       \end{array}\right\}_plt_\left\{
                                       \begin{array}{c}\mathtt{crn}\\
                                       \mathtt{cnt}\\
                                       \end{array}\right\}_0000``,

where ``hdf5`` or ``ncmpi`` is picked depending on the IO implementation
used, ``crn`` and ``cnt`` indicate data stored at the cell corners or
centers respectively, the number following “s” is the split file number,
if used, and the number at the end of the filename is the current value
of ``plotfileNumber``. ``crn`` is reserved, even though corner data
output is not presently supported by |flashx|\ ’s IO.

.. _`Sec:Output formats`:

Output Formats
--------------

HDF5 is our most most widely used IO library although Parallel-NetCDF is
rapidly gaining acceptance among the high performance computing
community. In |flashx| we also offer a serial direct FORTRAN IO which is
currently only implemented for the uniform grid. This option is intended
to provide users a way to output data if they do not have access to HDF5
or PnetCDF. Additionally, if HDF5 or PnetCDF are not performing well on
a given platform the direct IO implementation can be used as a last
resort. Our tools, fidlr and sfocu (), do not currently support the
direct IO implementation, and the output files from this mode are not
portable across platforms.

.. _`Sec:HDF5`:

HDF5
~~~~

HDF5 is supported on a large variety of platforms and offers large file
support and parallel IO via MPI-IO. Information about the different
versions of HDF can be found at
https://support.hdfgroup.org/documentation/. The IO in |flashx|
implementations require HDF5 1.4.0 or later. Please note that HDF5 1.6.2
requires IDL 1.6 or higher in order to use fidlr3.0 for post processing.

Implementations of the ``HDF5`` ``IO`` unit use the HDF application
programming interface (API) for organizing data in a database fashion.
In addition to the raw data, information about the data type and byte
ordering (little- or big-endian), rank, and dimensions of the dataset is
stored. This makes the HDF format extremely portable across platforms.
Different packages can query the file for its contents without knowing
the details of the routine that generated the data.

|flashx| provides different HDF5 IO unit implementations – the serial and
parallel versions for each supported grid, Uniform Grid and
``PARAMESH``. It is important to remember to match the IO implementation
with the correct grid, although the ``setup`` script generally takes
care of this matching. ``PARAMESH`` 2, ``PARAMESH`` 4.0, and
``PARAMESH`` 4dev all work with the ``PARAMESH``\ (PM) implementation of
IO. Nonfixed blocksize IO has its own implementation in parallel, and is
presently not supported in serial mode. Examples are given below for the
five different HDF5 IO implementations.

.. container:: codeseg

   ./setup Sod -2d -auto -unit=IO/IOMain/hdf5/serial/PM (included by
   default) ./setup Sod -2d -auto -unit=IO/IOMain/hdf5/parallel/PM
   ./setup Sod -2d -auto -unit=Grid/GridMain/UG
   -unit=IO/IOMain/hdf5/serial/UG ./setup Sod -2d -auto
   -unit=Grid/GridMain/UG -unit=IO/IOMain/hdf5/parallel/UG ./setup Sod
   -2d -auto -nofbs -unit=Grid/GridMain/UG
   -unit=IO/IOMain/hdf5/parallel/NoFbs

The default IO implementation is ``IO/IOMain/hdf5/serial/PM``. It can be
included simply by adding ``-unit=IO`` to the ``setup`` line. In
|flashx|, the user can set up shortcuts See for more information about
creating shortcuts.

The format of the HDF5 output files produced by these various IO
implementations is identical; only the method by which they are written
differs. It is possible to create a checkpoint file with the parallel
routines and restart |flashx| from that file using the serial routines or
vice-versa. (This switch would require resetting up and compiling a code
to get an executable with the serial version of IO.) When outputting
with the Uniform Grid, some data is stored that isn’t explicitly
necessary for data analysis or visualization, but is retained to keep
the output format of ``PARAMESH`` the same as with the Uniform Grid. See
for more information on output data formats. For example, the refinement
level in the Uniform Grid case is always equal to 1, as is the nodetype
array. A tree structure for the Uniform Grid is ‘faked’ for
visualization purposes. In a similar way, the non-fixedblocksize mode
outputs all of the data stored by the grid as though it is one large
block. This allows restarting with differing numbers of processors and
decomposing the domain in an arbitrary fashion in Uniform Grid.

Parallel HDF5 mode has two runtime parameters useful for debugging:
``IO/chkGuardCellsInput`` and ``IO/chkGuardCellsOutput``. When these
runtime parameters are true, the |flashx| input and output routines read
and/or output the guard cells in addition to the normal interior cells.
Note that the HDF5 files produced are *not* compatible with the
visualization and analysis tools provided with |flashx|.

.. _`sec:IOCollectiveMode`:

Collective Mode
^^^^^^^^^^^^^^^

By default, the parallel mode of HDF5 uses an independent access pattern
for writing datasets and performs IO without aggregating the disk access
for writing. Parallel HDF5 can also be run so that the writes to the
file’s datasets are aggregated, allowing the data from multiple
processors to be written to disk in fewer operations. This can greatly
increase the performance of IO on filesystems that support this
behavior. |flashx| can make use of this mode by setting the runtime
parameter ``useCollectiveHDF5`` to true.

Machine Compatibility
^^^^^^^^^^^^^^^^^^^^^

The HDF5 modules have been tested successfully on the ASC platforms and
on a Linux clusters. Performance varies widely across the platforms, but
the parallel version is usually faster than the serial version.
Experience on performing parallel IO on a Linux Cluster using PVFS is
reported in Ross *et al.* (2001). Note that for clusters without a
parallel filesystem, you should not use the parallel HDF5 IO module with
an NFS mounted filesystem. In this case, all of the information will
still have to pass through the node from which the disk is hanging,
resulting in contention. It is recommended that a serial version of the
HDF5 unit be used instead.

.. _`Sec:Data Format`:

HDF5 Data Format
^^^^^^^^^^^^^^^^

The HDF5 data format for |flashx| is identical to |flashx| for all grid
variables and datastructures used to recreate the tree and neighbor data
with the exception that ``bounding box``, ``coordinates``, and
``block size`` are now sized as ``mdim``, or the maximum dimensions
supported by |flashx|’s grids, which is three, rather than ``ndim``.
``PARAMESH`` 4.0 and ``PARAMESH`` 4dev, however, do requires a few
additional tree data structures to be output which are described below.
The format of the metadata stored in the HDF5 files has changed to
reduce the number of ‘writes’ required. Additionally, scalar data, like
``time, dt, nstep``, *etc.*, are now stored in a linked list and written
all at one time. Any unit can add scalar data to the checkpoint file by
calling the routine ``IO/IO_setScalar``. See for more details. The
|flashx| HDF5 format is summarized in .

.. container::
   :name: Tab:hdf5

   .. table:: HDF5 format (continued).

      +----------------------------------+----------------------------------+
      | Record label                     | Description of the record        |
      +==================================+==================================+
      |                                  | Description of the record        |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | sim info                         | Stores simulation meta data in a |
      |                                  | user defined C structure.        |
      |                                  | Structure datatype and           |
      |                                  | attributes of the structure are  |
      |                                  | described below.                 |
      +----------------------------------+----------------------------------+
      | .. container:: center            |                                  |
      |                                  |                                  |
      |    .. container:: codeseg        |                                  |
      |                                  |                                  |
      |       typedef struct sim_info_t  |                                  |
      |       int file_format_version;   |                                  |
      |       char setup_call[400]; char |                                  |
      |       file_c                     |                                  |
      | reation_time[MAX_STRING_LENGTH]; |                                  |
      |       char                       |                                  |
      |       f                          |                                  |
      | lash_version[MAX_STRING_LENGTH]; |                                  |
      |       char                       |                                  |
      |                                  |                                  |
      |   build_date[MAX_STRING_LENGTH]; |                                  |
      |       char                       |                                  |
      |                                  |                                  |
      |    build_dir[MAX_STRING_LENGTH]; |                                  |
      |       char                       |                                  |
      |       b                          |                                  |
      | uild_machine[MAX_STRING_LENGTH]; |                                  |
      |       char cflags[400]; char     |                                  |
      |       fflags[400]; char          |                                  |
      |       setu                       |                                  |
      | p_time_stamp[MAX_STRING_LENGTH]; |                                  |
      |       char                       |                                  |
      |       buil                       |                                  |
      | d_time_stamp[MAX_STRING_LENGTH]; |                                  |
      |       sim_info_t;                |                                  |
      |                                  |                                  |
      |       sim_info_t sim_info;       |                                  |
      +----------------------------------+----------------------------------+
      | `                                | An integer giving the version    |
      | `sim_info.file_format_version``: | number of the HDF5 file format.  |
      |                                  | This is incremented anytime      |
      |                                  | changes are made to the layout   |
      |                                  | of the file.                     |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | ``sim_info.setup_call``:         | The complete syntax of the       |
      |                                  | ``setup`` command used when      |
      |                                  | creating the current |flashx|     |
      |                                  | executable.                      |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | ``sim_info.file_creation_time``: | The time and date that the file  |
      |                                  | was created.                     |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | ``sim_info.flash_version``:      | The version of |flashx| used for  |
      |                                  | the current simulation. This is  |
      |                                  | returned by routine              |
      |                                  | ``setup_flashVersion``.          |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | ``sim_info.build_date``:         | The date and time that the       |
      |                                  | |flashx| executable was compiled. |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | ``sim_info.build_dir``:          | The complete path to the |flashx| |
      |                                  | root directory of the source     |
      |                                  | tree used when compiling the     |
      |                                  | |flashx| executable. This is      |
      |                                  | generated by the subroutine      |
      |                                  | ``setup_buildstats`` which is    |
      |                                  | created at compile time by the   |
      |                                  | Makefile.                        |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | ``sim_info.build_machine``:      | The name of the machine (and     |
      |                                  | anything else returned from      |
      |                                  | ``uname -a``) on which |flashx|   |
      |                                  | was compiled.                    |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | ``sim_info.cflags``:             | The c compiler flags used in the |
      |                                  | given simulation. The routine    |
      |                                  | ``setup_buildstats`` is written  |
      |                                  | by the ``setup`` script at       |
      |                                  | compile time and also includes   |
      |                                  | the ``fflags`` below.            |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | ``sim_info.fflags``:             | The f compiler flags used in the |
      |                                  | given simulation.                |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | ``sim_info.setup_time_stamp``:   | The date and time the given      |
      |                                  | simulation was setup. The        |
      |                                  | routine ``setup_buildstamp`` is  |
      |                                  | created by the ``setup`` script  |
      |                                  | at compile time.                 |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | ``sim_info.build_time_stamp``:   | The date and time the given      |
      |                                  | simulation was built. The        |
      |                                  | routine ``setup_buildstamp`` is  |
      |                                  | created by the ``setup`` script  |
      |                                  | at compile time.                 |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | *RuntimeParameter and Scalar     |                                  |
      | data*                            |                                  |
      +----------------------------------+----------------------------------+
      | Data are stored in linked lists  |                                  |
      | with the nodes of each entry for |                                  |
      | each type listed below.          |                                  |
      +----------------------------------+----------------------------------+
      | .. container:: center            |                                  |
      |                                  |                                  |
      |    .. container:: codeseg        |                                  |
      |                                  |                                  |
      |       typedef struct int_list_t  |                                  |
      |       char                       |                                  |
      |       name[MAX_STRING_LENGTH];   |                                  |
      |       int value; int_list_t;     |                                  |
      |                                  |                                  |
      |       typedef struct real_list_t |                                  |
      |       char                       |                                  |
      |       name[MAX_STRING_LENGTH];   |                                  |
      |       double value; real_list_t; |                                  |
      |                                  |                                  |
      |       typedef struct str_list_t  |                                  |
      |       char                       |                                  |
      |       name[MAX_STRING_LENGTH];   |                                  |
      |       char                       |                                  |
      |       value[MAX_STRING_LENGTH];  |                                  |
      |       str_list_t;                |                                  |
      |                                  |                                  |
      |       typedef struct log_list_t  |                                  |
      |       char                       |                                  |
      |       name[MAX_STRING_LENGTH];   |                                  |
      |       int value; log_list_t;     |                                  |
      |                                  |                                  |
      |       int_list_t \*int_list;     |                                  |
      |       real_list_t \*real_list;   |                                  |
      |       str_list_t \*str_list;     |                                  |
      |       log_list_t \*log_list;     |                                  |
      +----------------------------------+----------------------------------+
      | integer runtime parameters       | ``int                            |
      |                                  | _list_t int_list(numIntParams)`` |
      +----------------------------------+----------------------------------+
      |                                  | A linked list holding the names  |
      |                                  | and values of all the integer    |
      |                                  | runtime parameters.              |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | real runtime parameters          | ``real_l                         |
      |                                  | ist_t real_list(numRealParams)`` |
      +----------------------------------+----------------------------------+
      |                                  | A linked list holding the names  |
      |                                  | and values of all the real       |
      |                                  | runtime parameters.              |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | string runtime parameters        | ``str                            |
      |                                  | _list_t str_list(numStrParams)`` |
      +----------------------------------+----------------------------------+
      |                                  | A linked list holding the names  |
      |                                  | and values of all the string     |
      |                                  | runtime parameters.              |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | logical runtime parameters       | ``log                            |
      |                                  | _list_t log_list(numLogParams)`` |
      +----------------------------------+----------------------------------+
      |                                  | A linked list holding the names  |
      |                                  | and values of all the logical    |
      |                                  | runtime parameters.              |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | integer scalars                  | ``int_                           |
      |                                  | list_t int_list(numIntScalars)`` |
      +----------------------------------+----------------------------------+
      |                                  | A linked list holding the names  |
      |                                  | and values of all the integer    |
      |                                  | scalars.                         |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | real scalars                     | ``real_li                        |
      |                                  | st_t real_list(numRealScalars)`` |
      +----------------------------------+----------------------------------+
      |                                  | A linked list holding the names  |
      |                                  | and values of all the real       |
      |                                  | scalars.                         |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | string scalars                   | ``str_                           |
      |                                  | list_t str_list(numStrScalars)`` |
      +----------------------------------+----------------------------------+
      |                                  | A linked list holding the names  |
      |                                  | and values of all the string     |
      |                                  | scalars.                         |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | logical scalars                  | ``log_                           |
      |                                  | list_t log_list(numLogScalars)`` |
      +----------------------------------+----------------------------------+
      |                                  | A linked list holding the names  |
      |                                  | and values of all the logical    |
      |                                  | scalars.                         |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | *Grid data: included only in     |                                  |
      | checkpoint files and plotfiles*  |                                  |
      +----------------------------------+----------------------------------+
      | unknown names                    | ``character*4 unk_names(nvar)``  |
      +----------------------------------+----------------------------------+
      |                                  | This array contains              |
      |                                  | four-character names             |
      |                                  | corresponding to the first index |
      |                                  | of the ``unk`` array. They serve |
      |                                  | to identify the variables stored |
      |                                  | in the ‘unknowns’ records.       |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | refine level                     | ``in                             |
      |                                  | teger lrefine(globalNumBlocks)`` |
      +----------------------------------+----------------------------------+
      |                                  | This array stores the refinement |
      |                                  | level for each block.            |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | node type                        | ``int                            |
      |                                  | eger nodetype(globalNumBlocks)`` |
      +----------------------------------+----------------------------------+
      |                                  | This array stores the node type  |
      |                                  | for a block. Blocks with node    |
      |                                  | type 1 are leaf nodes, and their |
      |                                  | data will always be valid. The   |
      |                                  | leaf blocks contain the data     |
      |                                  | which is to be used for plotting |
      |                                  | purposes.                        |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | gid                              | ``integer gid(nf                 |
      |                                  | aces+1+nchild,globalNumBlocks)`` |
      +----------------------------------+----------------------------------+
      |                                  | This is the global               |
      |                                  | identification array. For a      |
      |                                  | given block, this array gives    |
      |                                  | the block number of the blocks   |
      |                                  | that neighbor it and the block   |
      |                                  | numbers of its parent and        |
      |                                  | children.                        |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | coordinates                      | ``re                             |
      |                                  | al coord(mdim,globalNumBlocks)`` |
      +----------------------------------+----------------------------------+
      |                                  | This array stores the            |
      |                                  | coordinates of the center of the |
      |                                  | block.                           |
      +----------------------------------+----------------------------------+
      |                                  | = *x*-coordinate                 |
      +----------------------------------+----------------------------------+
      |                                  | = *y*-coordinate                 |
      +----------------------------------+----------------------------------+
      |                                  | ``coord(3,blockID)`` =           |
      |                                  | *z*-coordinate                   |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | block size                       | ``r                              |
      |                                  | eal size(mdim,globalNumBlocks)`` |
      +----------------------------------+----------------------------------+
      |                                  | This array stores the dimensions |
      |                                  | of the current block.            |
      +----------------------------------+----------------------------------+
      |                                  | = *x* size                       |
      +----------------------------------+----------------------------------+
      |                                  | = *y* size                       |
      +----------------------------------+----------------------------------+
      |                                  | = *z* size                       |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | bounding box                     | ``real b                         |
      |                                  | nd_box(2,mdim,globalNumBlocks)`` |
      +----------------------------------+----------------------------------+
      |                                  | This array stores the minimum    |
      |                                  | (``bnd_box(1,:,:)``) and maximum |
      |                                  | (``bnd_box(2,:,:)``) coordinate  |
      |                                  | of a block in each spatial       |
      |                                  | direction.                       |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | which child (*|paramesh|4.0 and    | ``intege                         |
      | |paramesh|4dev only!*)             | r which_child(globalNumBlocks)`` |
      +----------------------------------+----------------------------------+
      |                                  | An integer array identifying     |
      |                                  | which part of the parents’       |
      |                                  | volume this child corresponds    |
      |                                  | to.                              |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | *variable*                       | ``real un                        |
      |                                  | k(nxb,nyb,nzb,globalNumBlocks)`` |
      +----------------------------------+----------------------------------+
      |                                  | = number of cells/block in *x*   |
      +----------------------------------+----------------------------------+
      |                                  | = number of cells/block in *y*   |
      +----------------------------------+----------------------------------+
      |                                  | = number of cells/block in *z*   |
      +----------------------------------+----------------------------------+
      |                                  | This array holds the data for a  |
      |                                  | single variable. The record      |
      |                                  | label is identical to the        |
      |                                  | four-character variable name     |
      |                                  | stored in the record *unknown    |
      |                                  | names*. Note that, for a plot    |
      |                                  | file with ``CORNERS=.true.`` in  |
      |                                  | the parameter file, the          |
      |                                  | information is interpolated to   |
      |                                  | the cell corners and stored.     |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | *Particle Data: included in      |                                  |
      | checkpoint files and particle    |                                  |
      | files*                           |                                  |
      +----------------------------------+----------------------------------+
      | localnp                          | ``in                             |
      |                                  | teger localnp(globalNumBlocks)`` |
      +----------------------------------+----------------------------------+
      |                                  | This array holds the number of   |
      |                                  | particles on each processor.     |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | particle names                   | ``character*2                    |
      |                                  | 4 particle_labels(NPART_PROPS)`` |
      +----------------------------------+----------------------------------+
      |                                  | This array contains twenty       |
      |                                  | four-character names             |
      |                                  | corresponding to the attributes  |
      |                                  | in the particles array. They     |
      |                                  | serve to identify the variables  |
      |                                  | stored in the ’particles’        |
      |                                  | record.                          |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+
      | tracer particles                 | ``real particles(N               |
      |                                  | PART_PROPS, globalNumParticles`` |
      +----------------------------------+----------------------------------+
      |                                  | Real array holding the particles |
      |                                  | data structure. The first        |
      |                                  | dimension holds the various      |
      |                                  | particle properties like,        |
      |                                  | velocity, tag etc. The second    |
      |                                  | dimension is sized as the total  |
      |                                  | number of particles in the       |
      |                                  | simulation. Note that all the    |
      |                                  | particle properties are real     |
      |                                  | values.                          |
      +----------------------------------+----------------------------------+
      |                                  |                                  |
      +----------------------------------+----------------------------------+

Split File IO
^^^^^^^^^^^^^

On machines with large numbers of processors, IO may perform better if,
all processors write to a limited number of separate files rather than
one single file. This technique can help mitigate IO bottlenecks and
contention issues on these large machines better than even parallel-mode
IO can. In addition this technique has the benefit of keeping the number
of output files much lower than if every processor writes its own file.
Split file IO can be enabled by setting the ``IO/outputSplitNum``
parameter to the number of files desired (i.e. if ``outputSplitNum`` is
set to 4, every checkpoint, plotfile and particle file will be broken
into 4 files, by processor number). This feature is only available with
the HDF5 parallel IO mode, and is still experimental. Users should use
this at their own risk.

.. _`Sec:PnetCDF IO`:

Parallel-NetCDF
~~~~~~~~~~~~~~~

Another implementation of the IO unit uses the Parallel-NetCDF library
available at http://www.mcs.anl.gov/parallel-netcdf/. At this time, the
|flashx| code requires version 1.1.0 or higher. Our testing shows
performance of PNetCDF library to be very similar to HDF5 library when
using collective I/O optimizations in parallel I/O mode.

There are two different PnetCDF IO unit implementations. Both are
parallel implementations, one for each supported grid, the Uniform Grid
and ``PARAMESH``. It is important to remember to match the IO
implementation with the correct grid. To include PnetCDF IO in a
simulation the user should add ``-unit=IO/IOMain/pnetcdf.....`` to the
``setup`` line. See examples below for the two different PnetCDF IO
implementations.

.. container:: codeseg

   ./setup Sod -2d -auto -unit=IO/IOMain/pnetcdf/PM ./setup Sod -2d
   -auto -unit=Grid/GridMain/UG -unit=IO/IOMain/pnetcdf/UG

The paths to these IO implementations can be long and tedious to type,
users are advised to set up shortcuts for various implementations. See
for information about creating shortcuts.

To the end-user, the PnetCDF data format is very similar to the HDF5
format. (Under the hood the data storage is quite different.) In HDF5
there are datasets and dataspaces, in PnetCDF there are dimensions and
variables. All the same data is stored in the PnetCDF checkpoint as in
the HDF5 checkpoint file, although there are some differences in how the
data is stored. The grid data is stored in multidimensional arrays, as
it is in HDF5. These are unknown names, refine level, node type, gid,
coordinates, proc number, block size and bounding box. The particles
data structure is also stored in the same way. The simulation metadata,
like file format version, file creation time, command line, *etc.*, are
stored as global attributes. The runtime parameters and the output
scalars are also stored as attributes. The ``unk`` and particle labels
are also stored as global attributes. In PnetCDF, all global quantities
must be consistent across all processors involved in a write to a file,
or else the write will fail. All IO calls are run in a collective mode
in PnetCDF.

Direct IO
~~~~~~~~~

As mentioned above, the direct IO implementation has been added so users
can always output data even if the HDF5 or pnetCDF libraries are
unavailable. The user should examine the two helper routines
``io_writeData`` and ``io_readData``. Copy the base implementation to a
simulation directory, and modify them in order to write out specifically
what is needed. To include the direct IO implementation add the
following to your setup line:

.. container:: codeseg

   -unit=IO/IOMain/direct/UG or -unit=IO/IOMain/direct/PM

Output Side Effects
~~~~~~~~~~~~~~~~~~~

In |flashx| when plotfiles or checkpoint files are output by
``IO/IO_output``, the grid is fully restricted and user variables are
computed prior to writing the file. ``IO/IO_writeCheckpoint`` and
``IO/IO_writePlotfile`` by default, do not do this step themselves. The
restriction can be forced for all writes by setting runtime parameter
``IO/alwaysRestrictCheckpoint`` to true and the user variables can
always be computed prior to output by setting
``IO/alwaysComputeUserVars`` to true.

Working with Output Files
-------------------------

The checkpoint file output formats offer great flexibility when
visualizing the data. The visualization program does not have to know
the details of how the file was written; rather it can query the file to
find the number of dimensions, block sizes, variable data etc that it
needs to visualize the data. ``IDL`` routines for reading HDF5 and
PnetCDF formats are provided in ``tools/fidlr3/``. These can be used
interactively though the ``IDL`` command line (see ). In addition, ViSit
version 10.0 and higher (see ) can natively read |flashx| HDF5 output
files by using the command line option ``-assume_format |flashx|``.

.. _`Sec:IO Unit Test`:

Unit Test
---------

The ``IO`` unit test is provided to test IO performance on various
platforms with the different |flashx| IO implementations and parallel
libraries. The ``unitTest`` is setup like any other |flashx| simulation.
It can be run with any IO implementation as long as the correct Grid
implementation is included. This ``unitTest`` writes a checkpoint file,
a plotfile, and if particles are included, a particle file. Particles IO
can be tested simply by including particles in the simulation. Variables
needed for particles should be uncommented in the ``Config`` file.

Example setups:

.. container:: codeseg

   #setup for PARAMESH Grid and serial HDF5 io ./setup unitTest/IO -auto

   #setup for PARAMESH Grid with parallel HDF5 IO (see shortcuts docs
   for explanation) ./setup unitTest/IO -auto +parallelIO (same as)
   ./setup unitTest/IO -auto -unit=IO/IOMain/hdf5/parallel/PM

   #setup for Uniform Grid with serial HDF5 IO, 3d problem, increasing
   default number of zones ./setup unitTest/IO -3d -auto +ug -nxb=16
   -nyb=16 -nzb=16 (same as) ./setup unitTest/IO -3d -auto
   -unit=Grid/GridMain/UG -nxb=16 -nyb=16 -nzb=16

   #setup for PM3 and parallel netCDF, with particles ./setup
   unitTest/IO -auto -unit=Particles +pnetcdf

   #setup for UG and parallel netCDF ./setup unitTest/IO -auto +pnetcdf
   +ug

Run the test like any other |flashx| simulation:

.. container:: codeseg

   mpirun -np numProcs flash3

There are a few things to keep in mind when working with the IO unit
test:

-  The Config file in unitTest/IO declares some dummy grid scope
   variables which are stored in the unk array. If the user wants a more
   intensive IO test, more variables can be added. Variables are
   initialized to dummy values in ``Driver_evolveFlash``.

-  Variables will only be output to the plotfile if they are declared in
   the ``flash.par`` (see the example ``flash.par`` in the unit test).

-  The only units besides the simulation unit included in this
   simulation are
   ``Grid, IO, Driver, Timers, Logfile, RuntimeParameters`` and
   ``PhysicalConstants``.

-  If the ``PARAMESH`` Grid implementation is being used, it is
   important to note that the grid will not refine on its own. The user
   should set ``lrefine_min`` to a value :math:`>` 1 to create more
   blocks. The user could also set the runtime parameters ``nblockx``,
   ``nblocky``, ``nblockz`` to make a bigger problem.

-  Just like any other simulation, the user can change the number of
   zones in a simulation using ``-nxb=numZones`` on the setup line.

Derived data type I/O
---------------------

In |flashx| we introduced an alternative I/O implementation for both HDF5
and Parallel-NetCDF which is a slight spin on the standard parallel I/O
implementations. In this new implementation we select the data from the
mesh data structures directly using HDF5 hyperslabs (HDF5) and MPI
derived datatypes (Parallel-NetCDF) and then write the selected data to
datasets in the file. This eliminates the need for manually copying data
into a |flashx| allocated temporary buffer and then writing the data from
the temporary buffer to disk.

You can include derived data type I/O in your |flashx| application by
adding the setup shortcuts ``+hdf5TypeIO`` for HDF5 and ``+pnetTypeIO``
for Parallel-NetCDF to your setup line. If you are using the HDF5
implementation then you need a parallel installation of HDF5. All of the
runtime parameters introduced in this chapter should be compatible with
derived data type I/O.

A nice property of derived data type I/O is that it eliminates a lot of
the I/O code duplication which has been spreading in the |flashx| I/O
unit over the last decade. The same code is used for UG, NoFBS and
|paramesh| |flashx| applications and we have also shared code between the
HDF5 and Parallel-NetCDF implementations. A technical reason for using
the new I/O implementation is that we provide more information to the
I/O libraries about the exact data we want to read from / write to disk.
This allows us to take advantage of recent enhancements to I/O libraries
such as the nonblocking APIs in the Parallel-NetCDF library. We discuss
experimentation with this API and other ideas in the paper “A Case Study
for Scientific I/O: Improving the |flashx| Astrophysics Code”
`www.mcs.anl.gov/uploads/cels/papers/P1819.pdf <www.mcs.anl.gov/uploads/cels/papers/P1819.pdf>`__

The new I/O code has been tested in our internal |flashx| regression
tests from before the |flashx| release and there are no known issues,
however, it will probably be in the release following |flashx| when we
will recommend using it as the default implementation. We have made the
research ideas from our case study paper usable for all |flashx|
applications, however, the code still needs a clean up and exhaustive
testing with all the |flashx| runtime parameters introduced in this
chapter.
