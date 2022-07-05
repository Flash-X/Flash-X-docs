.. include:: defs.h

.. _`Chp:flashxtest`:

Flash-X Testing Framework (FlashXTest)
====================================================

Testing and maintainence of Flash-X source code is implemented using the ``FlashXTest`` python and command line utility. The utility can be installed directly from its `repository <https://github.com/Flash-X/Flash-X-Test>`_. 

After cloning the repository type,

::

   ./setup develop

in the root folder and it will install ``FlashXTest`` in your python environment and copy the command line script to your ``~/.local/bin``. ``FlashXTest`` won't mainpulate your environment ``PATH`` variable so you may have to manually update your ``PATH`` in ``~/.bashrc`` by adding

::

   export PATH="$PATH:~/.local/bin"

Once this is done you can type ``flashxtest --help`` in your terminal and should be able to see following

::

   Usage: flashxtest [OPTIONS] COMMAND [ARGS]...

     Python CLI for Flash-X Testing Utility

   Options:
     --help  Show this message and exit.

   Commands:
     add     Add a test from simulation directory
     init    Initialize test configuration
     remove  Remove a test from test suite
     run     Run a list of tests from xml file
     view    Launch webviewer


To setup your local testing infrastructure either create an empty folder in your desired location or go directly to your ``Flash-X`` root directory and run 

::

   flashxtest init -z <path-to-Flash-X> -s <site>

This will create a ``config`` file and a ``/.fxt`` folder which will be used to manage your test configuration.


Test information should be provided in individual ``Simulation/SimulationMain`` directories. Each simulation directory should contain a ``tests`` folder which contains a ``test.toml`` configuration file and ``*.par`` files pertaining to individual tests. Here is an example of a sample ``test.toml`` located in ``Simulation/SimulationMain/incompFlow/DeformingBubble/tests``

::

   # TOML file for DeformingBubble test
   # TODO: handle string replacement feature used in xml files

   [default]
       testNode = "UnitTest/incompFlow/DeformingBubble/2D/AMReX"
       setupOptions = "-auto -2d -nxb=16 -nyb=16 +amrex +parallelIO"
       numProcs = 1

   [user-test]
       testNode = "UnitTest/incompFlow/DeformingBubble/2D/pm4dev"
       setupOptions = "-auto -2d -nxb=16 -nyb=16 --index-reorder +pm4dev -gridinterpolation=native +serialIO"
       numProcs = 2
       parFile = "test_user.par"


The keys ``default`` and ``user-test`` provide configuration of two seprate tests for this particular simulation. To add one of the tests in your local testing suite type

::

   flashxtest add incompFlow/DeformingBubble -t <test-key>

This will create a ``testInfo.xml`` which will translate the information under ``<test-key>`` into an ``xml`` node. A ``jobFile`` will also be created which will contain the value of ``testNode`` provided in ``test.toml``. Add more tests by repeating the ``flashxtest add`` command. An already populated set of ``testInfo.xml`` and ``jobFile`` is available in the ``Tests`` folder of ``FlashXTest`` `repository <https://github.com/Flash-X/Flash-X-Test>`_, and contains all the tests that currently run by the maintainers.


To deploy a test run, type


::

   flashxtest run 

in the directory containing your ``config``, ``testInfo.xml``, and ``jobFile``. The ``flashxtest run`` command will query all the nodes provided in ``jobFile`` against ``testInfo.xml`` and will run tests based on listed configuration.

There maybe situations where breaking up the ``jobFile`` is desired to organize test deployment. This is easily handled by the ``flashxtest run`` command. Simply breakup the ``jobFile`` to mulitple files with your desired naming convention and type

::

   flashxtest run <filename1> <filename2> <filename3> ....


*This project is currently under development. Please file bugs/issues at: https://github.com/Flash-X/Flash-X-Test* 
