.. include:: defs.h

.. _`Chp:Profiler and Timer Units`:

Timer and Profiler Units
========================

.. _`Sec:Timers Unit`:

Timers
------

.. _`Sec:MPINative`:

MPINative
~~~~~~~~~

|flashx| includes an interface to a set of stopwatch-like timing routines
for monitoring performance. The interface is defined in the
``monitors/Timers`` unit, and an implementation that uses the timing
functionality provided by MPI is provided in
``monitors/Timers/TimersMain/MPINative``. Future implementations might
use the PAPI framework to track hardware counter details.

The performance routines start or stop a timer at the beginning or end
of a section of code to be monitored, and accumulate performance
information in dynamically assigned accounting segments. The code also
has an interface to write the timing summary to the |flashx| logfile.
These routines are not recommended for timing very short segments of
code due to the overhead in accounting.

There are two ways of using the ``Timers`` routines in your code. One
mode is to simply pass timer names as strings to the start and stop
routines. In this first way, a timer with the given name will be created
if it doesn’t exist, or otherwise reference the one already in
existence. The second mode of using the timers references them not by
name but by an integer key. This technique offers potentially faster
access if a timer is to be started and stopped many times (although
still not recommended because of the overhead). The integer key is
obtained by calling with a string name ``monitors/Timers/Timers_create``
which will only create the timer if it doesn’t exist and will return the
integer key. This key can then be passed to the start and stop routines.

The typical usage pattern for the timers is implemented in the default
``Driver`` implementation. This pattern is: call
``monitors/Timers/Timers_init`` once at the beginning of a run, call
``monitors/Timers/Timers_start`` and ``monitors/Timers/Timers_stop``
around sections of code, and call ``monitors/Timers/Timers_getSummary``
at the end of the run to report the timing summary at the end of the
logfile. However, it is possible to call
``monitors/Timers/Timers_reset`` in the middle of a run to reset all
timing information. This could be done along with writing the summary
once per-timestep to report code times on a per-timestep basis, which
might be relevant, for instance, for certain non-fixed operation count
solvers. Since ``monitors/Timers/Timers_reset`` does not reset the
integer key mappings, it is safe to obtain a key through
``monitors/Timers/Timers_create`` once in a saved variable, and continue
to use it after calling ``monitors/Timers/Timers_reset``.

Two runtime parameters control the ``Timer`` unit and are described
below.

.. container:: center

   .. container::
      :name: Tab:timers parameters

      .. table:: Timers parameters (continued).

         +------------------+-------------+---------------+------------------+
         | Parameter        | Type        | Default value | Description      |
         +==================+=============+===============+==================+
         |                  | Type        | Default value | Description      |
         +------------------+-------------+---------------+------------------+
         | ``eachPro        | ``LOGICAL`` | ``TRUE``      | Should each      |
         | cWritesSummary`` |             |               | process write    |
         |                  |             |               | its summary to   |
         |                  |             |               | its own file? If |
         |                  |             |               | true, each       |
         |                  |             |               | process will     |
         |                  |             |               | write its        |
         |                  |             |               | summary to a     |
         |                  |             |               | file named       |
         |                  |             |               | tim              |
         |                  |             |               | er_summary\_\ :m |
         |                  |             |               | ath:`<`\ process |
         |                  |             |               | id\ :math:`>`    |
         +------------------+-------------+---------------+------------------+
         | ``wr             | ``LOGICAL`` | ``TRUE``      | Should timers    |
         | iteStatSummary`` |             |               | write the        |
         |                  |             |               | max/min/avg      |
         |                  |             |               | values for       |
         |                  |             |               | timers to the    |
         |                  |             |               | logfile?         |
         +------------------+-------------+---------------+------------------+

``monitors/Timers/TimersMain/MPINative`` writes two summaries to the
logfile: the first gives the timer execution of the master processor,
and the second gives the statistics of max, min, and avg times for
timers on all processors. The secondary max, min, and avg times will not
be written if some process executed timers differently than another. For
example, this anomaly happens if not all processors contain at least one
block. In this case, the ``Hydro`` timers only execute on the processors
that possess blocks. See for an example of this type of output. The max,
min, and avg summary can be disabled by setting the runtime parameter
``Timers/writeStatSummary`` to false. In addition, each process can
write its summary to its own file named ``timer_summary_<process id>``.
To prohibit each process from writing its summary to its own file, set
the runtime parameter ``Timers/eachProcWritesSummary`` to false.

Tau
~~~

In ``|flashx|3.1`` we add an alternative ``Timers`` implementation which
is designed to be used with the ``Tau`` framework
(http://acts.nersc.gov/tau/). Here, we use ``Tau`` API calls to time the
``|flashx|`` labeled code sections (marked by ``Timers_start`` and
``Timers_stop``). After running the simulation, the ``Tau`` profile
contains timing information for both ``|flashx|`` labeled code sections
and all individual subroutines / functions. This is useful because fine
grained subroutine / function level data can be overwhelming in a huge
code like ``|flashx|``. Also, the callpaths are preserved, meaning we can
see how long is spent in individual subroutines / functions when they
are called from within a particular ``|flashx|`` labeled code section.
Another reason to use the ``Tau`` version is that the ``MPINative``
version (See ) is implemented using recursion, and so incurs significant
overhead for fine grain measurements.

To use this implementation we must compile the ``|flashx|`` source code
with the ``Tau`` compiler wrapper scripts. These are set as the default
compilers automatically whenever we specify the ``-tau`` option (see )
to the setup script. In addition to the ``-tau`` option we must specify
``–with-unit=monitors/Timers/TimersMain/Tau`` as this ``Timers``
implementation is not the default.

.. _`Sec:Profiler Unit`:

Profiler
--------

In addition to an interface for simple timers, |flashx| includes a
generic interface for third-party profiling or tracing libraries. This
interface is defined in the ``monitors/Profiler`` unit.

In |flashx| we created an interface to the IBM profiling libraries
libmpihpm.a and libmpihpm_smp.a and also to HPCToolkit
http://hpctoolkit.org/ (Rice University). We make use of this interface
to profile |flashx| evolution only, i.e. not initialization. To use this
style of profiling add ``-unit=monitors/Profiler/ProfilerMain/mpihpm``
or ``-unit=monitors/Profiler/ProfilerMain/hpctoolkit`` to your setup
line and also set the |flashx| runtime parameter ``profileEvolutionOnly``
= .true.

For the IBM profiling library (mpihpm) you need to add LIB_MPIHPM and
LIB_MPIHPM_SMP macros to your Makefile.h to link |flashx| to the
profiling libraries. The actual macro used in the link line depends on
whether you setup |flashx| with multithreading support (LIB_MPIHPM for
MPI-only |flashx| and LIB_MPIHPM_SMP for multithreaded |flashx|). Example
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
