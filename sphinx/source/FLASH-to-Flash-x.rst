.. include:: defs.h

.. _`Sec:converting`:

Converting from FLASH to Flash-X
============

Converting |flash| source to |flashx| source so that it can build with
basic functionality is relatively straightforward if advanced features
of the software system are not to be used. Many low level routines
will compile without any modifications. However, several commonly used
interfaces have changed, and some units have been reorganized. The
*Driver* unit no longer implements time integration, there is a new
unit *TimeAdvance* where the integrators reside. In addition to the
Strang-split integrator inherited from |flash| there is a new
implementation of this new unit that uses the  method of lines
*Mol*. With full integration of |cgkit| into the code this unit is
likely to be converted into a collection of recipes. The *Driver* unit
only handles the book keeping such as checking for and doing
refinement, checkpointing and other I/O related actions.  

The *Grid* unit has had fairly extensive changes to its API. The
biggest change from the perspective of physics units is that  bounds
(typically stored in blkLimits and blkLimitsGC) 
for a block are based on global view instead of local view that
|flash| used.   Therefore, even with fixed block sizes, the values of
in the above two arrays are not identical for all blocks. The second
major change is in looping over blocks. Instead of obtaining a list of
blocks from the grid unit and looping over them through *block_id* we
now have iterators for looping over blocks. Several quantities that
could only be obtained through explicit interfaces such as
*Grid_getBlkPtr* are now accessible through the functions provided by
the iterators.  

For example, in |flash|, following code lines appear in a file that has looping over blocks:


.. container:: codeseg

	       use Grid_interface, ONLY: Grid_getListofBlocks, Grid_getBlkLimits
	       !! some more use statements

	       integer :: blockCount, blockList(MAXBLOCKS)
	       integer, dimension(LOW:HIGH,MDIM) :: blkLimits, blkLimitsGC
	       real, dimension(MDIM):: deltas
	       real, dimension(:,:,:,:), pointer || Uin

	       call Grid_getListofBlocks(blockCount,blockList,LEAF)
	       do i = 1,blockcount
	           blockid=blockList(i)
		   call Grid_getBlkLimits(blockid,blkLimits,blkLimitsGC)
		   call Grid_getDeltas(blockid,deltas)
		   call Grid_getBlkPtr(blockid,Uin,CENTER)
		   
		   !! some code for operating on the block
	       end do


In |flashx| Grid_getListofBlocks is deprecated, and does not work with
|amrex|. The code lines that provide the same functionality are 


.. container:: codeseg
	       
	       use Grid_interface, ONLY : Grid_getTileIterator, Grid_releaseTileIterator
	       use Grid_tile,         ONLY : Grid_tile_t
	       use Grid_iterator,     ONLY : Grid_iterator_t
	       

	       type(Grid_iterator_t) :: itor  
               type(Grid_tile_t) :: tileDesc
               integer, dimension(LOW:HIGH,MDIM) :: blkLimits, blkLimitsGC,grownLimits
	       real, dimension(MDIM):: deltas
	       real, dimension(:,:,:,:), pointer || Uin	       

	       call Grid_getTileIterator(itor,LEAF,level)
               do while(itor%%isValid())
                        call itor%%currentTile(tileDesc)
                        blkLimits(:,:)=tileDesc%%limits
                        blkLimitsGC=tileDesc%%blkLimitsGC
                        call tileDesc%getDataPtr(Uin, CENTER)
	                call tileDesc%deltas(del)

			!! some code for operating on the block

		       call tileDesc%releaseDataPtr(Uin,CENTER)
                       call itor%next()
                end do !!block loop
                call Grid_releaseTileIterator(itor)

Using general purpose macros provided with the code the same code can
be reduced to

.. container:: codeseg
	       
	      @M iter_use
	      @M iter_declare(blkLimits,blkLimitsGC, grownLimits)	       
              @M iter_all_begin(LEAF,.false., blkLimits,blkLimitsGC, grownLimits)

	              !! some code for operating on the block

              @M iter_end
		       
Note that @M iter_declare would expand to

.. container:: codeseg

           type(Grid_iterator_t) :: itor
           real,   pointer    :: Uin(:,:,:,:)
           type(Grid_tile_t)     :: tileDesc
           integer :: level
           integer, dimension(LOW:HIGH,MDIM) :: blkLimits,blkLimitsGC, grownLimits
           real,dimension(MDIM) :: deltas

In this macro, the declarations of arrays containing bounds can be
changed by changing the argument list, for example instead of
blkLimits,blkLimitsGC, grownLimits as arguments one could use
limit1, limit2, limit3 as the argument list and the corresponding
expansion would change to 

.. container:: codeseg

           type(Grid_iterator_t) :: itor
           real,   pointer    :: Uin(:,:,:,:)
           type(Grid_tile_t)     :: tileDesc
           integer :: level
           integer, dimension(LOW:HIGH,MDIM) :: limit1, limit2, limit3 
           real,dimension(MDIM) :: deltas

However, the declarations of itor, Uin, deltas, level and tileDesc are
unchangeable. Users wishing to use different variable names for these
quantities can define their own macros either with a larger argument
list or different explicit variable names. It would be best to keep
the newly defined macros local to the user's own code component and
give it a different name to avoid causing any ambiguity in expansion.
The final version of |flashx| is envisioned to
have iteration over block occur only at the level of the *TimeAdvance*
unit. 

Several new interfaces in the *Grid* API are introduced for greater
flexibility in handling fluxes and their reconciliation at fine-coarse
boundaries. These are needed for new time integrater, whether Mol or
higher order methods. Currently many interfaces do not have any real
implentations included. They may be eliminated or implemented in
future. The commonly used interfaces that have changed from |flash| to
|flash-x| are listed below.

.. container:: codeseg
	       Grid_getCellCoords
	       Grid_getCellFaceAreas
	       Grid_getCellVolumes
