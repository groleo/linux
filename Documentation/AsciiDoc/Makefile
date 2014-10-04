###
# This makefile is used to generate the kernel documentation,
# primarily based on in-line comments in various source files.
# See Documentation/kernel-doc-nano-HOWTO.txt for instruction in how
# to document the SRC - and how to read it.
# To add a new book the only step required is to add the book to the
# list of ASCIIDOCS.

ASCIIDOCS := z8530book.tmpl article.txt drivers.9.txt
GENDIR:=AsciiDoc
srctree?=../..
obj?=.

-include $(srctree)/Documentation/$(GENDIR)/media/Makefile

###
# The build process is as follows (targets):
#              (xmldocs) [by docproc]
# file.tmpl --> file.xml +--> file.ps   (psdocs)   [by db2ps or xmlto]
#                        +--> file.pdf  (pdfdocs)  [by db2pdf or xmlto]
#                        +--> DIR=file  (htmldocs) [by xmlto]
#                        +--> man/      (mandocs)  [by xmlto]


# for PDF and PS output you can choose between xmlto and docbook-utils tools
PDF_METHOD	= $(prefer-db2x)
PS_METHOD	= $(prefer-db2x)


###
# The targets that may be used.
PHONY += xmldocs sgmldocs psdocs pdfdocs htmldocs mandocs installmandocs cleandocs LaTeX docbook dvi chunked epub

BOOKS := $(addsuffix .txt,$(addprefix $(obj)/,$(ASCIIDOCS)))
xmldocs: $(BOOKS)
sgmldocs: xmldocs

installmandocs: mandocs
	mkdir -p /usr/local/man/man9/
	install Documentation/$(GENDIR)/man/*.9.gz /usr/local/man/man9/

###
#External programs used
KERNELDOC = $(srctree)/scripts/kernel-doc

DOCPROC   = $(objtree)/scripts/docproc

XMLTOFLAGS = -m $(srctree)/Documentation/$(GENDIR)/stylesheet.xsl
XMLTOFLAGS += --skip-validation

###
# DOCPROC is used for two purposes:
# 1) To generate a dependency list for a .tmpl file
# 2) To preprocess a .tmpl file and call kernel-doc with
#     appropriate parameters.
# The following rules are used to generate the .txt documentation
# required to generate the final targets. (ps, pdf, html).
quiet_cmd_docproc = DOCPROC $@
      cmd_docproc = SRCTREE=$(srctree)/ $(DOCPROC) doc $< >$@
define rule_docproc
	set -e;								\
        $(if $($(quiet)cmd_$(1)),echo '  $($(quiet)cmd_$(1))';) 	\
        $(cmd_$(1)); 							\
        ( 								\
          echo 'cmd_$@ := $(cmd_$(1))'; 				\
          echo $@: `SRCTREE=$(srctree) $(DOCPROC) depend $<`; 		\
        ) > $(dir $@).$(notdir $@).cmd
endef

%.txt: %.tmpl FORCE
	$(call if_changed_rule,docproc)

ifneq ($(cmd_files),)
  include $(cmd_files)
endif

###
# Changes in kernel-doc force a rebuild of all documentation
$(BOOKS): $(KERNELDOC)

# Tell kbuild to always build the programs
always := $(hostprogs-y)

notfoundtemplate = echo "*** You have to install docbook-utils or xmlto ***"; \
		   exit 1

quiet_cmd_db2html = HTML    $@
      cmd_db2html = xmlto xhtml $(XMLTOFLAGS) -o $(patsubst %.html,%,$@) $< && \
		echo '<a HREF="$(patsubst %.html,%,$(notdir $@))/index.html"> \
        $(patsubst %.html,%,$(notdir $@))</a><p>' > $@

quiet_cmd_a2man = MAN     $@
      cmd_a2man = a2x --doctype=manpage --format=manpage $(obj)/*.9; gzip -f $(obj)/man/*.9

mandocs :
	@(which a2x > /dev/null 2>&1) || \
	 (echo "*** You need to install a2x ***"; \
	  exit 1)
	for file in $(ASCIIDOCS); do \
		a2x --doctype=manpage --format=manpage $$file; \
	done	 		
	@touch $@

#article format (PDF)
quiet_cmd_a2art = PDF     $@
	cmd_a2art = a2x --doctype=article --format=pdf $(ASCIIDOCS); gzip -f $(obj)/pdfdocs/*.pdf

pdfdocs :
	@(which a2x > /dev/null 2>&1) || \
	 (echo "*** You need to install a2x ***"; \
	  exit 1)
	for file in $(ASCIIDOCS); do \
		a2x --doctype=article --format=pdf $$file; \
	done
	@touch $@

#article format (PS)
quiet_cmd_a2psdoc = PS     $@
	cmd_a2art = a2x --doctype=article --format=ps $(ASCIIDOCS); gzip -f $(obj)/psdocs/*.pdf

psdocs :
	@(which a2x > /dev/null 2>&1) || \
	 (echo "*** You need to install a2x ***"; \
	  exit 1)
	for file in $(ASCIIDOCS); do \
		a2x --doctype=article --format=ps $$file; \
	done
	@touch $@

#html format
quiet_cmd_a2html = HTML     $@
	cmd_a2html = a2x --doctype=article --format=xhtml $(ASCIIDOCS); gzip -f $(obj)/html/*.html

htmldocs :
	@(which a2x > /dev/null 2>&1) || \
	 (echo "*** You need to install a2x ***"; \
	  exit 1)
	for file in $(ASCIIDOCS); do \
		a2x --doctype=article --format=xhtml $$file; \
	done
	@touch $@

#latex format
quiet_cmd_a2tex = TEX     $@
	cmd_a2tex = a2x --doctype=article --format=tex $(ASCIIDOCS); gzip -f $(obj)/tex/*.tex

LaTeX : 
	@(which a2x > /dev/null 2>&1) || \
	 (echo "*** You need to install a2x ***"; \
	  exit 1)
	for file in $(ASCIIDOCS); do \
		a2x --doctype=article --format=tex $$file; \
	done
	@touch $@

#docbook format 
quiet_cmd_a2docbook = TEX     $@
	cmd_a2docbook = a2x --doctype=article --format=docbook $(ASCIIDOCS); gzip -f $(obj)/xml/*.xml
	
docbook : 
	@(which a2x > /dev/null 2>&1) || \
	 (echo "*** You need to install a2x ***"; \
	  exit 1)
	for file in $(ASCIIDOCS); do \
		a2x --doctype=article --format=docbook $$file; \
	done
	@touch $@

#dvi format
quiet_cmd_a2docbook = DVI     $@
	cmd_a2docbook = a2x --doctype=article --format=dvi $(ASCIIDOCS); gzip -f $(obj)/dvi/*.dvi
	
dvi : 
	@(which a2x > /dev/null 2>&1) || \
	 (echo "*** You need to install a2x ***"; \
	  exit 1)
	for file in $(ASCIIDOCS); do \
		a2x --doctype=article --format=dvi $$file; \
	done
	@touch $@

#chunked format
quiet_cmd_a2chunked = CHUNK     $@
	cmd_a2chunked  = a2x --doctype=article --format=chunked $(ASCIIDOCS);


chunked : 
	@(which a2x > /dev/null 2>&1) || \
	 (echo "*** You need to install a2x ***"; \
	  exit 1)
	for file in $(ASCIIDOCS); do \
		a2x --doctype=article --format=chunked $$file; \
	done
	@touch $@

#epub format
quiet_cmd_a2epub = EPUB     $@
	cmd_a2epub = a2x --doctype=article --format=epub $(ASCIIDOCS)

epub : 	
	@(which a2x > /dev/null 2>&1) || \
	 (echo "*** You need to install a2x ***"; \
	  exit 1)
	for file in $(ASCIIDOCS); do \
		a2x --doctype=article --format=epub $$file; \
	done 
	@touch $@
###
# Rules to generate postscripts and PNG images from .fig format files
quiet_cmd_fig2eps = FIG2EPS $@
      cmd_fig2eps = fig2dev -Leps $< $@

%.eps: %.fig
	@(which fig2dev > /dev/null 2>&1) || \
	 (echo "*** You need to install transfig ***"; \
	  exit 1)
	$(call cmd,fig2eps)

quiet_cmd_fig2png = FIG2PNG $@
      cmd_fig2png = fig2dev -Lpng $< $@

%.png: %.fig
	@(which fig2dev > /dev/null 2>&1) || \
	 (echo "*** You need to install transfig ***"; \
	  exit 1)
	$(call cmd,fig2png)

###
# Rule to convert a .c file to inline XML documentation
       gen_xml = : 
 quiet_gen_xml = echo '  GEN     $@'
silent_gen_xml = :
%.txt: %.c
	@$($(quiet)gen_xml)
	@(                            \
	   echo "<programlisting>";   \
	   expand --tabs=8 < $< |     \
	   sed -e "s/&/\\&amp;/g"     \
	       -e "s/</\\&lt;/g"      \
	       -e "s/>/\\&gt;/g";     \
	   echo "</programlisting>")  > $@

###
# Help targets as used by the top-level makefile
dochelp:
	@echo  '  Linux kernel internal documentation in different formats:'
	@echo  '  htmldocs        - HTML'
	@echo  '  LaTeX		  - TEX'
	@echo  '  pdfdocs         - PDF'
	@echo  '  psdocs          - Postscript'
	@echo  '  xmldocs         - XML asciidoc'
	@echo  '  mandocs         - man pages'
	@echo  '  installmandocs  - install man pages generated by mandocs'
	@echo  '  docbook         - xml'
	@echo  '  chunked         - CHUNKED'
	@echo  '  epub 		  - EPUB'	
	@echo  '  cleandocs       - clean all generated asciidoc files'

###
# Temporary files left by various tools
clean-files := $(ASCIIDOCS) \
	$(patsubst %.txt, %.dvi,  $(ASCIIDOCS)) \
	$(patsubst %.txt, %.aux,  $(ASCIIDOCS)) \
	$(patsubst %.txt, %.tex,  $(ASCIIDOCS)) \
	$(patsubst %.txt, %.log,  $(ASCIIDOCS)) \
	$(patsubst %.txt, %.out,  $(ASCIIDOCS)) \
	$(patsubst %.txt, %.ps,   $(ASCIIDOCS)) \
	$(patsubst %.txt, %.pdf,  $(ASCIIDOCS)) \
	$(patsubst %.txt, %.html, $(ASCIIDOCS)) \
	$(patsubst %.txt, %.9,    $(ASCIIDOCS)) \
	$(patsubst %.txt, %.xml,  $(ASCIIDOCS)) \
	$(index)

clean-dirs := $(patsubst %.txt,%,$(ASCIIDOCS)) man

cleandocs:
	mkdir $(srctree)/asciidoc_aux
	mv $(ASCIIDOCS) media Makefile $(srctree)/asciidoc_aux
	rm -rf * 
	mv $(srctree)/asciidoc_aux/* $(obj)
	rm -rf $(srctree)/asciidoc_aux

# Declare the contents of the .PHONY variable as phony.  We keep that
# information in a variable se we can use it in if_changed and friends.
.PHONY: $(PHONY)
