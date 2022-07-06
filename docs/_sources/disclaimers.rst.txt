.. include:: defs.h

.. _`Sec:disclaimers`:

Current Status and Disclaimers 
============

The *Developer's Guide* included at the end is work in progress. It is
incomplete, but the information that is included is valid. Please check back
frequently for updates if you are developing code for |flashx|

Maintaining a large and complex code like |flashx| is
challenging. This is especially true because a great deal of
performance portability layer is currently in the process of being
hardened for production use and not all of its interfaces are
finalized. Therefore, it is extremely important for the developers to
read through the section on **Good Practices for Maintenance** to
ensure that their contributions will not need large amounts of
refactoring later. 

Additionally, some portions of |flashx| are undergoing rapid development because of
which their description in the users's guide may become out of date
periodically. The status of all major components, and persons who
can give most up-to-date information about the corresponding component
are listed in below.

**Infrastructure**


+------------+------------------------------+----------------------+
| Setup tool | Tom Klosterman / Klaus Weide | may be out of date   |
+------------+------------------------------+----------------------+
| Grid       | Klaus Weide / Tom Klosterman | likely to be current |
+------------+------------------------------+----------------------+
| I/O        | Rajeev Jain                  | current              |
+------------+------------------------------+----------------------+



**Physics**


+---------------+------------------------------+-------------------------------+
| Unsplit Hydro | Klaus Weide                  | current                       |
+---------------+------------------------------+-------------------------------+
| INS           | Akash Dhruv                  | may be out of date            |
+---------------+------------------------------+-------------------------------+
| Spark         | Sean Couch / Anshu Dubey     | may be out of date            |
+---------------+------------------------------+-------------------------------+
| Gravity       | Sean Couch                   | current                       |
+---------------+------------------------------+-------------------------------+
|               | Austin Harris                | current except Xnet           |
| Burn          |                              | which is not included yet     | 
+---------------+------------------------------+-------------------------------+
| Eos           | Austin Harris / Sean Couch   | current, except Weaklib       |
+---------------+------------------------------+-------------------------------+
| Thornado      | Eirik Endeve                 | submodule, not described here |
+---------------+------------------------------+-------------------------------+
| Particles     | Anshu Dubey                  | mostly current, tracers only         |
+---------------+------------------------------+-------------------------------+


**Contact**

+----------------+---------------------+
|  Akash Dhruv   | adhruv@anl.gov      |
+----------------+---------------------+
|  Anshu Dubey   | adubey@anl.gov      |
+----------------+---------------------+
|  Sean Couch    | scouch@msu.edu      |
+----------------+---------------------+
|  Eirik Endeve  | endevee@ornl.gov    |
+----------------+---------------------+
|  Austin Harris | harrisja@ornl.gov   |
+----------------+---------------------+
|  Rajeev Jain   | rajeeja@anl.gov     |
+----------------+---------------------+
|  Jared O'Neal  | joneal@anl.gov      |
+----------------+---------------------+
|  Klaus Weide   | kweide@uchicago.edu |
+----------------+---------------------+









	 




	 



	 




	 
