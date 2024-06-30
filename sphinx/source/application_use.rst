.. include:: defs.h


.. _`Chp:Application Use`:

Application's Use of Orcha
======================

In this section we desribe the end-to-end pipeline of how an
application can use |orcha|. At first glance it may seem intimidating
because there are several steps involved in the configurations
process, however, the steps are many fewer in terms of what the users
have to do to make use of |orcha|. The onus on the user/domain science
developer are summarized as below.

#. Ensure that all the dependencies are present. These include
   Flash-X_recipe_tools, Milhoja_pypkg, and |cgkit| in addition to the
   |flashx| distribution which includes setup tool and
   macroprocessor. The installation of |flashx|_recipe Tools will
   resolve all the other dependencies 
 
#. Ensure that all data that needs to move to the device either shows
   up in the argument list of one of the interfaces, or is moved using
   explicit offloading to the device. The data packet generator only
   looks at the annotations on the interface file. It assumes that
   this is all the data it needs to move to the device. If there is
   any additional data that needs to be there at the device, the unit
   must include the directives for moving in the code. An example of
   the kind of data that users may prefer to move themselves would be
   scalars that are common across all blocks and do not change during
   the processing of the blocks. 

#. Add comment blocks into the unit's interface file. The comment
   block starts with !! milhjoa begin, and ends with !! milhoja end.
   The first block in the section describes what is common among all
   interfaces. For example, blkLimits or the bounds of the block that
   is being operated on is common to several interfaces in all
   physics. So the extent, the size and the bounds are listed in the
   block for describing common properties. Comments above each
   interface then describe the size of arguments in that interface,
   referring to the common quantities where appropriate. This is
   information that is parsed and converted into json files by one of
   the code generators in the tool-chain

#. Learn to write the recipes in the format that the code generators
   can understand. Examples of recipes are provided in the code
   segments below.  

 .. container:: codeseg

	def load_recipe()
	recipe=flashx.TimeStepRecipe()
	# assuming [Modulename]_[Routine] format
	# JSON files will be populated by reading
	 [Modulename]_interface.F90
	hydro_begin = recipe.begin_orchestration(after=recipe.root)
	hydro_prepblock=recipe.add.work("Hydro_prepBlock",
	after=hydro_begin, map_to=gpu)
	hydro_advance = recipe.add.work("Hydro_advance",
	after=hydro_prepblock, map_to=gpu)
	hydro_end = recipe.end_orchestration(hydro_begin,
	after=hydro_advance)
	return recipe


The recipe described above is the simplest time-stepper for the Hydro
unit when no other physics in called by the time-stepper. Here the
first action is to start the recipe, the interface *Hydro_prepBlock"
allocates all the scratch space needed for computing one block, and it
also queries the *Grid* unit for all the quantities such as physical
coordinates, or cell areas etc that it might need. The interface
*Hydro_advance* computes the fluxes and advances the result in
time. In this instance, because both the tasks are mapped to the GPU,
the task function generated simply invokes these interfaces one after
the other, and the data packet is generated for the union of data used
by the two interfaces. Note that the code will work perfectly fine
without doing any of the steps described above, however, the
performance advantages from latency hiding for data movement will not
be possible. 


	
