.. include:: defs.h

.. _`Sec:disclaimers`:

Current Status and Disclaimers 
============

The *Developer's Guide* included at the end is work in progress. It is
incomplete, but the information that is included is valid. Please check back
frequently for updates if you are developing code for |flashx|

Maintaining a large and complex code like |flashx| is
challenging. This is especially true because several components of
|orcha| have not been exercised sufficiently in production setting so
far. It is still possible that some of the interfaces may need
modifications.  Therefore, it is extremely important for the developers to
read through the section on **Good Practices for Maintenance** to
ensure that their contributions will not need large amounts of
refactoring later. 

Additionally, some portions of |flashx| are undergoing rapid development because of
which their description in the users's guide may become out of date
periodically. The status of all major components, and persons who
can give most up-to-date information about the corresponding component
are listed in below.

**Infrastructure**

+---------------+--------------------------+----------------+
| Setup too     |  Klaus Weide/Younjun Lee | mostly current |  
+---------------+--------------------------+----------------+
| Grid          | Klaus Weide/Anshu Dubey  | mostly current |
+---------------+--------------------------+----------------+
| I/O           | Klaus Weide/Anshu Dubey  | current        |
+---------------+--------------------------+----------------+
| Orchestration | Youngjun Lee/Klaus Weide | In flux        |
+---------------+--------------------------+----------------+
| MOL           | Steve Fromm              | In flux        |
+---------------+--------------------------+----------------+


**Physics**

+---------------+------------------------------+-------------------------------+
| Unsplit Hydro | Klaus Weide                  | current                       |
+---------------+------------------------------+-------------------------------+
| INS           | Akash Dhruv                  | mostly current                |
+---------------+------------------------------+-------------------------------+
| Spark         | Youngjun Lee/Anshu Dubey     | mostly current                |
+---------------+------------------------------+-------------------------------+
| Gravity       | Austin Harris/Sean Couch     | current                       |
+---------------+------------------------------+-------------------------------+
|  Burn         | Austin Harris                | current                       |
+---------------+------------------------------+-------------------------------+
| Eos           | Austin Harris/Steve Fromm    | current, except Hybrid        |
+---------------+------------------------------+-------------------------------+
| Thornado      | Eirik Endeve                 | submodule, not described here |
+---------------+------------------------------+-------------------------------+
| Particles     | Anshu Dubey                  |  tracers only                 |
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
|  Klaus Weide   | kweide@uchicago.edu |
+----------------+---------------------+
|  Youngjun Lee  | leey@anl.gov        |
+----------------+---------------------+








	 




	 



	 




	 
