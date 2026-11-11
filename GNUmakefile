# GNUmakefile -- build the LCS and M-atlas papers
#
# Directory structure (all repositories side by side under one parent):
#   ../texmf/tex/latex/        STYDIR: the one directory for .sty files,
#                              shared with LaTeX-Semantic-Markup
#   LaTeX-Semantic-Markup/     shmuelsemtex.sty source; its "make install"
#                              copies the package into STYDIR
#   Local-Coordinate-Spaces/   this repository: the papers, and
#                              lcs-common.sty, the preamble they share
#
# Targets:
#   all       build both papers (default); needs shmuelsemtex.sty in STYDIR
#   install   copy lcs-common.sty into STYDIR
#   clean     remove TeX build leftovers
#
# Files are copied, never linked (no symlinks on JFS or FAT32).
# TEXINPUTS uses ";" as its separator on Windows and OS/2, ":" elsewhere.

STYDIR  ?= ../texmf/tex/latex
LATEX   ?= pdflatex
BIBTEX  ?= bibtex
PAPERS  := LCS.arXiv.V2 M-atlas

ifeq ($(OS),Windows_NT)
  PATHSEP := ;
else ifdef OS2_SHELL
  PATHSEP := ;
else
  PATHSEP := :
endif

export TEXINPUTS := $(STYDIR)//$(PATHSEP)

.PHONY: all install clean

all: $(PAPERS:=.pdf)

install: $(STYDIR)/lcs-common.sty

$(STYDIR)/lcs-common.sty: lcs-common.sty
	mkdir -p $(STYDIR)
	cp lcs-common.sty $@

$(STYDIR)/shmuelsemtex.sty:
	@echo "shmuelsemtex.sty is not in $(STYDIR):"
	@echo "run make install in ../LaTeX-Semantic-Markup first."
	@exit 1

%.pdf: %.tex papers.bib $(STYDIR)/lcs-common.sty $(STYDIR)/shmuelsemtex.sty
	$(LATEX) -interaction=nonstopmode $*.tex
	$(BIBTEX) $*
	$(LATEX) -interaction=nonstopmode $*.tex
	$(LATEX) -interaction=nonstopmode $*.tex

AUX_EXTS := aux log out toc blg bbl

clean:
	$(RM) $(foreach ext,$(AUX_EXTS),$(wildcard *.$(ext)))
