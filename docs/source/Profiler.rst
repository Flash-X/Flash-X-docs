.. _`Chp:Profiler and Timer Units`:

Timer and Profiler Units
========================

.. _`Sec:Timers Unit`:

Timers
------

.. _`Sec:MPINative`:

MPINative
~~~~~~~~~

Flash-X includes an interface to a set of stopwatch-like timing routines
for monitoring performance. The interface is defined in the unit, and an
implementation that uses the timing functionality provided by MPI is
provided in . Future implementations might use the PAPI framework to
track hardware counter details.

The performance routines start or stop a timer at the beginning or end
of a section of code to be monitored, and accumulate performance
information in dynamically assigned accounting segments. The code also
has an interface to write the timing summary to the Flash-X logfile.
These routines are not recommended for timing very short segments of
code due to the overhead in accounting.

There are two ways of using the routines in your code. One mode is to
simply pass timer names as strings to the start and stop routines. In
this first way, a timer with the given name will be created if it
doesn’t exist, or otherwise reference the one already in existence. The
second mode of using the timers references them not by name but by an
integer key. This technique offers potentially faster access if a timer
is to be started and stopped many times (although still not recommended
because of the overhead). The integer key is obtained by calling with a
string name which will only create the timer if it doesn’t exist and
will return the integer key. This key can then be passed to the start
and stop routines.

The typical usage pattern for the timers is implemented in the default
implementation. This pattern is: call once at the beginning of a run,
call and around sections of code, and call at the end of the run to
report the timing summary at the end of the logfile. However, it is
possible to call in the middle of a run to reset all timing information.
This could be done along with writing the summary once per-timestep to
report code times on a per-timestep basis, which might be relevant, for
instance, for certain non-fixed operation count solvers. Since does not
reset the integer key mappings, it is safe to obtain a key through once
in a saved variable, and continue to use it after calling .

Two runtime parameters control the unit and are described below.

.. container:: center

   .. container::
      :name: Tab:timers parameters

      .. table:: Timers parameters (continued).

         +-----------+------+---------------+--------------------------------+
         | Parameter | Type | Default value | Description                    |
         +===========+======+===============+================================+
         |           | Type | Default value | Description                    |
         +-----------+------+---------------+--------------------------------+
         |           |      |               | Should each process write its  |
         |           |      |               | summary to its own file? If    |
         |           |      |               | true, each process will write  |
         |           |      |               | its summary to a file named    |
         |           |      |               | timer                          |
         |           |      |               | _summary\_\ :math:`<`\ process |
         |           |      |               | id\ :math:`>`                  |
         +-----------+------+---------------+--------------------------------+
         |           |      |               | Should timers write the        |
         |           |      |               | max/min/avg values for timers  |
         |           |      |               | to the logfile?                |
         +-----------+------+---------------+--------------------------------+

writes two summaries to the logfile: the first gives the timer execution
of the master processor, and the second gives the statistics of max,
min, and avg times for timers on all processors. The secondary max, min,
and avg times will not be written if some process executed timers
differently than another. For example, this anomaly happens if not all
processors contain at least one block. In this case, the Hydro timers
only execute on the processors that possess blocks. See for an example
of this type of output. The max, min, and avg summary can be disabled by
setting the runtime parameter to false. In addition, each process can
write its summary to its own file named . To prohibit each process from
writing its summary to its own file, set the runtime parameter to false.

Tau
~~~

In we add an alternative implementation which is designed to be used
with the framework (http://acts.nersc.gov/tau/). Here, we use API calls
to time the labeled code sections (marked by and ). After running the
simulation, the profile contains timing information for both labeled
code sections and all individual subroutines / functions. This is useful
because fine grained subroutine / function level data can be
overwhelming in a huge code like . Also, the callpaths are preserved,
meaning we can see how long is spent in individual subroutines /
functions when they are called from within a particular labeled code
section. Another reason to use the version is that the version (See ) is
implemented using recursion, and so incurs significant overhead for fine
grain measurements.

To use this implementation we must compile the source code with the
compiler wrapper scripts. These are set as the default compilers
automatically whenever we specify the option (see ) to the setup script.
In addition to the option we must specify as this implementation is not
the default.

.. _`Sec:Profiler Unit`:

Profiler
--------

In addition to an interface for simple timers, Flash-X includes a
generic interface for third-party profiling or tracing libraries. This
interface is defined in the unit.

In we created an interface to the IBM profiling libraries libmpihpm.a
and libmpihpm_smp.a and also to HPCToolkit http://hpctoolkit.org/ (Rice
University). We make use of this interface to profile Flash-X evolution
only, i.e. not initialization. To use this style of profiling add or to
your setup line and also set the Flash-X runtime parameter = .true.

For the IBM profiling library (mpihpm) you need to add LIB_MPIHPM and
LIB_MPIHPM_SMP macros to your Makefile.h to link Flash-X to the
profiling libraries. The actual macro used in the link line depends on
whether you setup Flash-X with multithreading support (LIB_MPIHPM for
MPI-only Flash-X and LIB_MPIHPM_SMP for multithreaded Flash-X). Example
values from sites/miralac1/Makefile.h follow

.. container:: codeseg

   LIB_MPI = HPM_COUNTERS = /bgsys/drivers/ppcfloor/bgpm/lib/libbgpm.a
   LIB_MPIHPM = -L/soft/perftools/hpctw -lmpihpm
   :math:`(HPM_COUNTERS)`\ (LIB_MPI) LIB_MPIHPM_SMP =
   -L/soft/perftools/hpctw -lmpihpm_smp
   :math:`(HPM_COUNTERS)`\ (LIB_MPI)

For HPCToolkit you need to set the environmental variable
HPCRUN_DELAY_SAMPLING=1 at job launch to enable selective profiling (see
the HPCToolkit user guide).
