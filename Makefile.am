###############################################################################
## Makefile for apertium-evn
###############################################################################

LANG1=evn
BASENAME=apertium-$(LANG1)

TARGETS_COMMON =			\
	$(LANG1).automorf.hfst		\
	$(LANG1).automorf.bin		\
	$(LANG1).automorf.att.gz	\
	$(LANG1).autogen.hfst		\
	$(LANG1).autophon.hfst		\
	$(LANG1).autogen.bin		\
	$(LANG1).autogen.att.gz		\
	$(LANG1).autoseg.hfst		\
	$(LANG1).autoseg.bin		\
	$(LANG1).prob		\
	$(LANG1).rlx.bin

TARGETS_INTERMEDIATE = $(LANG1).mor.hfst $(LANG1).seg.hfst $(LANG1).gen.hfst

# This include defines goals for install-modes, .deps/.d and .mode files:
@ap_include@

###############################################################################

evn.automorf.hfst: evn.mor.hfst evn.spellrelax.hfst
	hfst-invert $< |  hfst-compose -F -1 - -2 evn.spellrelax.hfst  | hfst-invert | hfst-fst2fst -w -o $@

evn.autogen.hfst: evn.gen.hfst
	hfst-fst2fst -w $< -o $@

evn.autoseg.hfst: evn.seg.hfst 
	hfst-fst2fst -w $< -o $@

evn.mor.hfst: evn.gen.hfst evn.mor.twol.hfst
	hfst-compose-intersect -1 evn.gen.hfst -2 evn.mor.twol.hfst | hfst-realign | hfst-invert -o $@

# <impf><o_pl1><s_pl3>tʼis<v><tv>+o<mark>:k>oj>ki>tʼis>o
evn.gen.hfst: evn.lexc.hfst evn.twol.hfst
	hfst-compose-intersect -1 evn.lexc.hfst -2 evn.twol.hfst | hfst-realign -o $@

evn.twol.hfst: apertium-evn.evn.twol
	hfst-twolc $< -o $@

# <impf><o_pl1><s_pl3>tʼis<v><tv>+o<mark>:k>{o}j>k{i}>tʼis>o
evn.lexc.hfst: apertium-evn.evn.lexc
	hfst-lexc -A --Werror $< -o $@

evn.mor.twol.hfst: apertium-evn.evn.mor.twol
	hfst-twolc apertium-evn.evn.mor.twol -o evn.mor.twol.hfst

# kojkitʼiso:k>oj>ki>tʼis>o
evn.seg.hfst: evn.mor.hfst evn.gen.hfst
	hfst-compose -F -1 evn.mor.hfst -2 evn.gen.hfst -o evn.seg.hfst 

evn.phon.hfst: apertium-evn.evn.phon
	hfst-lexc $< -o $@

evn.autophon.hfst: evn.mor.hfst evn.phon.hfst
	hfst-fst2fst -w evn.phon.hfst -o evn.autophon.hfst
#	hfst-project -p upper evn.mor.hfst | hfst-compose -1 - -2 evn.phon.hfst | hfst-fst2fst -w -o evn.autophon.hfst

evn.spellrelax.hfst: apertium-evn.evn.spellrelax 
	hfst-regexp2fst -S -o $@ < $<

evn.rlx.bin: apertium-evn.evn.rlx
	cg-comp $< $@

$(LANG1).autogen.att.gz: $(LANG1).gen.hfst
	hfst-fst2txt $< | gzip -9 -c > $@

$(LANG1).automorf.att.gz: $(LANG1).mor.hfst evn.spellrelax.hfst
	hfst-invert $< |  hfst-compose -F -1 - -2 evn.spellrelax.hfst  | hfst-invert | hfst-fst2txt | gzip -9 -c > $@

$(LANG1).autoseg.att.gz: $(LANG1).seg.hfst
	hfst-fst2txt $< | gzip -9 -c > $@

$(LANG1).autogen.bin: $(LANG1).autogen.att.gz .deps/.d
	zcat < $< > .deps/$(LANG1).autogen.att
	lt-comp lr .deps/$(LANG1).autogen.att $@

$(LANG1).automorf.bin: $(LANG1).automorf.att.gz .deps/.d
	zcat < $< > .deps/$(LANG1).automorf.att
	lt-comp lr .deps/$(LANG1).automorf.att $@

$(LANG1).autoseg.bin: $(LANG1).autoseg.att.gz .deps/.d
	zcat < $< > .deps/$(LANG1).autoseg.att
	lt-comp lr .deps/$(LANG1).autoseg.att $@

$(LANG1).prob: $(BASENAME).$(LANG1).tagged
	apertium-tagger -s 0 -u 2 $@ $<

###############################################################################
## Distribution
###############################################################################

EXTRA_DIST=$(BASENAME).$(LANG1).lexc		\
		$(BASENAME).$(LANG1).twol	\
		$(BASENAME).$(LANG1).spellrelax \
		$(BASENAME).$(LANG1).rlx	\
		modes.xml

###############################################################################
## Installation stuff
###############################################################################
#
#   apertium_evn_dir: This is where the compiled binaries go
#   apertium_evn_srcdir: This is where the source files go

apertium_evndir=$(prefix)/share/apertium/$(BASENAME)/
apertium_evn_srcdir=$(prefix)/share/apertium/$(BASENAME)/

EXTRA_TARGETS=

if HAVE_HFSTOSPELL
EXTRA_TARGETS += $(LANG1).zhfst
endif # HAVE_HFSTOSPELL

apertium_evn_DATA=$(TARGETS_COMMON) $(EXTRA_TARGETS) 

pkgconfigdir = $(prefix)/share/pkgconfig
pkgconfig_DATA = $(BASENAME).pc

noinst_DATA=modes/$(LANG1)-morph.mode

install-data-local: install-modes
	$(INSTALL_DATA) $(BASENAME).$(LANG1).lexc $(DESTDIR)$(apertium_evn_srcdir)
	$(INSTALL_DATA) $(BASENAME).$(LANG1).twol $(DESTDIR)$(apertium_evn_srcdir)
	$(INSTALL_DATA) $(BASENAME).$(LANG1).rlx $(DESTDIR)$(apertium_evn_srcdir)


###############################################################################
## Cleanup
###############################################################################

CLEANFILES = $(TARGETS_COMMON) $(TARGETS_INTERMEDIATE)
clean-local:
	-rm -rf .deps modes

###############################################################################
