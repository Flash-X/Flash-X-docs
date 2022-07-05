.. include:: defs.h

.. _`Sec:converting`:

Converting from FLASH to Flash-X
============

Converting |flash| source to |flashx| source so that it can build with basic functionality is relatively straightforward if advanced features of the software system are not to be used. Many low level routines will compile without any modifications. However, several commonly used interfaces have changed, and iterators are used for looping over blocks. At the end of the chapter we list most commonly used Grid accessor functions that have changed. Some commonly repeated patterns have macros defined for them that are also listed at the end, with the description of functionality they provide.

In |flash|, following code lines appear in a file that has looping over blocks:


.. container:: codeseg

	       use Grid_interface, ONLY: Grid_getListofBlocks, Grid_getBlkLimits
	       !! some more use statements

	       integer :: blockCount, blockList(MAXBLOCKS), blockType
	       integer, dimension(LOW:HIGH,MDIM) :: blkLimits, blkLimitsGC
	       real, dimension(MDIM):: deltas
	       !! some more declarations

	       call Grid_getListofBlocks(blockCount,blockList,blockType)
	       do i = 1,blockcount
	           blockid=blockList(i)
		   call Grid_getBlkLimits(blockid,blkLimits,blkLimitsGC)
		   call Grid_getDeltas(blockid,deltas)
		   
		   !! some code for operating on the block
	       end do


In |flashx| Grid_getListofBlocks does not exist. The code lines that provide the same functionality are 


.. container:: codeseg
	       
	       use Grid_interface, ONLY : Grid_getTileIterator, Grid_releaseTileIterator
	       use Grid_tile,         ONLY : Grid_tile_t
	       use Grid_iterator,     ONLY : Grid_iterator_t
	       !! use statements unchanged

	       type(Grid_iterator_t) :: itor  
               type(Grid_tile_t) :: tileDesc
	       !! declarations unchaged 

Additionally, bounds for a block are based on global view instead of local view that |flash| used.  
