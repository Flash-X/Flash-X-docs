#  Makefile for FLASH user's guide

USERS_GUIDE_MAIN = flashx_ug

LATEX=env TEXINPUTS=../texmf//: latex --src
PDFLATEX=env TEXINPUTS=../texmf//: pdflatex
MKIDX=makeindex

DVIPS_FLAGS=-t letter -Ppdf -G0 -z 
L2H_FLAGS=-split 4 -toc_depth 3 -html_version 4.01 -show_section_numbers -local_icons -dir ../flash4_ug -mkdir

TEX_FILES = arch.tex \
            Driver.tex \
            Gravity.tex PhysicalConstants.tex \
            Hydro.tex \
            intro.tex \
            IO.tex \
	    Eos.tex Multispecies.tex\
            Grid.tex Logfile.tex Profiler.tex \
	    Particles.tex \
            quickstart.tex \
            RuntimeParameters.tex \
            setup.tex flashH.tex \
            sfocu.tex \
            GridSolvers.tex \
            sourceTerms.tex \
            Simulation.tex 

# The following list does not include .png files.
# The filenames below should only include the basename, not the original directory
# because symbolic links are made in the proc directory

pdf $(USERS_GUIDE_MAIN).pdf : $(USERS_GUIDE_MAIN).tex $(TEX_FILES)
#$(FIGURES) $(FIGURES:.eps=.png) $(PS_ONLY_FIGURES:.ps=.png)
	$(PDFLATEX) $(USERS_GUIDE_MAIN).tex
#	$(MKIDX) -l $(USERS_GUIDE_MAIN).idx
#	$(MKIDX) -l $(USERS_GUIDE_MAIN).rdx -o $(USERS_GUIDE_MAIN).rnd
#	$(MKIDX) -l $(USERS_GUIDE_MAIN).adx -o $(USERS_GUIDE_MAIN).and
#	$(PDFLATEX) $(USERS_GUIDE_MAIN).tex
#	$(PDFLATEX) $(USERS_GUIDE_MAIN).tex


clean :
	rm -f *.log *.aux *.ilg *.ind *.idx *~ *.toc *.bbl *.blg *.and *.rnd *.adx *.rdx $(USERS_GUIDE_MAIN).out
	rm -f *_pic.tex *_pic.png *_pic.eps

realclean : clean
	rm -f $(USERS_GUIDE_MAIN).{dvi,ps,pdf}

