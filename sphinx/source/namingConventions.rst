.. include:: defs.h

.. _`Chp:NamingConventions`:

Naming Conventions
=============


**Units**
    Unit names have their first letter capitalized, for example
    Grid, Eos, Hydro etc

**Subunits** 
    Subunits have composite names that include a full unit name
    followed by a capitalized word describing the subunit, for example  
    GridMain, GridParticles, HydroMain etc.
    Every unit has at least one subunit named UnitMain.

**Subunits Implementations** 
    The directories containing implementations of sub units have the 
    following convention: 
    If there are no implementations of the unit API in any of its child 
    directories, then the current directory name is capilaized, otherwise 
    it is all smallcase. 

**Organizational directories**
    These directory are normal unix directories, used for organizational
    purposes. For example <physics> is an organizational directory
    for all the physics units. All helper and utilities directories are 
    lowercase too.

**Unit API functions**
    Unit_routineName is the typical format of API function names.
    The unit name is followed by an underscore. The first word in the 
    remaining part of the name is lowercase and all subsequent words are
    capitalized. For example, Grid_fillGuardcells, Driver_getDt} etc

**Private UnitMain functions**
    The private functions are named un_routineName, where *un* stands for
    a short string (usually 2 or 3 letters) derived from the unit name.
    For example gr_createDomain is a private function in the GridMain
    subunit.

**Private Subunit functions**
   For every subunit other than *UnitMain*, functions are named
   un_suRoutineName, where *un* has the same meaning as for
   *UnitMain*, and *su* is another short string derived from the
   subunit name. For example gr_ptMoveData is a private function of
   GridParticles subunit.

**Third party functions**
    Functions imported from elsewhere are not required to, but encouraged to follow the Private unit
    function convention, as long as they are private to the unit.

**Unit Data modules**
    The main data modules are named Unit_main.
    For additional data modules, the naming convention is similar
    to private functions, the unit name is replaced by the short string.
    There are no such data modules in the current release, but an
    example would be hy_ppmData, if we needed a data module for the PPM
    kernel in the Hydro unit.

**Constants**
    The Constants are mostly defined in the pre-processor format 
    #define CONSTANT_NAME value. They are all uppercase, sometimes
    multiple words are separated by an underscore. They are avaiable 
    in "name.h" header files

**Variables**
    The unit-scope variables, defined in Unit_data, begin
    with *un_*, similar to the convention for private function.
    For example, gr_myPe and gr_procGrid are variables in the Grid unit.
    Variables local to a function are not constrained to be 
    named in any specific way.

**Macros**
   Macros are new in |flashx| and follow the naming convention of
   private subunit functions if they are private to a subunit. Unit
   scope macros follow the naming convention of *UnitMain* subunit.
   However, some macros are pre-defined for universal use. Those macros effectively
   become reserved keywords for the code, and may not be reused
   elsewhere in the code. Universal macros will reside in their own
   directory in the *Simulations* unit.


