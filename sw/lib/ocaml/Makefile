# Hey Emacs, this is a -*- makefile -*-
#
#   Copyright (C) 2003 Pascal Brisset, Antoine Drouin
#
# This file is part of paparazzi.
#
# paparazzi is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 2, or (at your option)
# any later version.
#
# paparazzi is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with paparazzi; see the file COPYING.  If not, write to
# the Free Software Foundation, 59 Temple Place - Suite 330,
# Boston, MA 02111-1307, USA.
#


Q=@


#OCAMLC=ocamlfind ocamlc
#OCAMLOPT=ocamlfind ocamlopt
#OCAMLDEP=ocamlfind ocamldep
#OCAMLLEX=ocamllex
#OCAMLYACC=ocamlyacc
#OCAMLMKLIB=ocamlmklib
#OCAMLLIBDIR=$(shell $(OCAMLC) -where)
include ../../Makefile.ocaml

# verbose ocamlmklib: Print commands before executing them
#VERBOSITY = -verbose
VERBOSITY =

UNAME = $(shell uname -s)
ifeq ("$(UNAME)","Darwin")
	MKTEMP = gmktemp
else
	MKTEMP = mktemp
endif

LABLGTK2GNOMECANVAS = $(shell ocamlfind query -p-format lablgtk2-gnome.gnomecanvas 2>/dev/null)
ifeq ($(LABLGTK2GNOMECANVAS),)
LABLGTK2GNOMECANVAS = $(shell ocamlfind query -p-format lablgtk2.gnomecanvas 2>/dev/null)
endif

CAMLP4_DEFS ?=
NETCLIENT_VER := $(shell ocamlfind query -format '%v' netclient)
NETCLIENT_MAJOR := $(shell echo $(NETCLIENT_VER) | cut -f1 -d.)
NETCLIENT_MINOR1 := $(shell echo $(NETCLIENT_VER) | cut -f2 -d.)
NETCLIENT_MINOR2 := $(shell echo $(NETCLIENT_VER) | cut -f3 -d.)
ifeq ($(shell test $(NETCLIENT_MAJOR) -ge 4; echo $$?),0)
CAMLP4_DEFS += -DNETCLIENT_V_4
NETTLS_GNUTLS = nettls-gnutls
COMMA_NETTLS_GNUTLS = ,$(NETTLS_GNUTLS)
ifeq ($(shell test $(NETCLIENT_MINOR2) -ge 4; echo $$?),0)
CAMLP4_DEFS += -DNETCLIENT_V_404
else ifeq ($(shell test $(NETCLIENT_MINOR1) -g 0; echo $$?),0)
CAMLP4_DEFS += -DNETCLIENT_V_404
endif
endif
OCAMLC_VER := $(shell ocamlc -version)
OCAMLC_MAJOR := $(shell echo $(OCAMLC_VER) | cut -f1 -d.)
OCAMLC_MINOR := $(shell echo $(OCAMLC_VER) | cut -f2 -d.)
ifeq ($(shell test $(OCAMLC_MAJOR) -ge 4; echo $$?),0)
ifeq ($(shell test $(OCAMLC_MINOR) -ge 2; echo $$?),0)
# the Bytes module is available since OCaml 4.02.0
CAMLP4_DEFS += -DHAS_BYTES_MODULE
endif
endif
PP_OPTS = -pp "camlp4o pa_macro.cmo $(CAMLP4_DEFS)"

# which source files to run through caml4p
PP_SRC =  http.ml compat.ml

INCLUDES=
PKGCOMMON=pprzlink,xml-light,netclient,glibivy,lablgtk2,$(NETTLS_GNUTLS)
XINCLUDES=
XPKGCOMMON=pprzlink,xml-light,glibivy,$(LABLGTK2GNOMECANVAS),lablgtk2.glade

SRC = compat.ml fig.ml debug.ml base64.ml serial.ml ocaml_tools.ml expr_syntax.ml expr_parser.ml expr_lexer.ml extXml.ml env.ml xml2h.ml latlong.ml egm96.ml srtm.ml http.ml maps_support.ml gm.ml iGN.ml geometry_2d.ml cserial.o ubx.ml xmlCom.ml os_calls.ml editAirframe.ml defivybus.ml fp_proc.ml gen_common.ml
CMO = $(SRC:.ml=.cmo)
CMX = $(SRC:.ml=.cmx)

XSRC = gtk_tools.ml platform.ml contrastLabel.ml acIcon.ml wind_sock.ml gtk_papget_editor.ml gtk_papget_text_editor.ml gtk_papget_gauge_editor.ml gtk_papget_led_editor.ml papget_common.ml papget_renderer.ml papget.ml mapCanvas.ml mapWaypoints.ml mapTrack.ml mapGoogle.ml mapIGN.ml ml_gtk_drag.o xmlEdit.ml mapFP.ml
XCMO = $(XSRC:.ml=.cmo)
XCMX = $(XSRC:.ml=.cmx)

# files to generate before running ocamldep
GEN_DEP = expr_parser.ml expr_lexer.ml

TESTS_SRC = test/test_latlong.ml
TESTS_CMO = $(TESTS_SRC:.ml=.cmo)

all : byte native
byte : lib-pprz.cma xlib-pprz.cma xml_get.out META.pprz
native : lib-pprz.cmxa xlib-pprz.cmxa META.pprz


lib-pprz.cma liblib-pprz.a: $(CMO)
	@echo OL $@
	$(Q)$(OCAMLMKLIB) $(VERBOSITY) $(INCLUDES) -o lib-pprz $^

lib-pprz.cmxa dlllib-pprz.so: $(CMX)
	@echo OOL $@
	$(Q)$(OCAMLMKLIB) $(VERBOSITY) $(INCLUDES) -o lib-pprz $^

xlib-pprz.cma libxlib-pprz.a: $(XCMO)
	@echo OL $@
	$(Q)$(OCAMLMKLIB) $(VERBOSITY) $(XINCLUDES) -o xlib-pprz $^

xlib-pprz.cmxa dllxlib-pprz.so: $(XCMX)
	@echo OOL $@
	$(Q)$(OCAMLMKLIB) $(VERBOSITY) $(XINCLUDES) -o xlib-pprz $^

# trying to set correct dependencies for parallel build
# these are order only depedencies
lib-pprz.cmxa: | liblib-pprz.a dlllib-pprz.so

xlib-pprz.cmxa: | libxlib-pprz.a dllxlib-pprz.so

xml_get.cmo : xml_get.ml lib-pprz.cma
	@echo OC $@
	$(Q)$(OCAMLC) $(INCLUDES) -package $(PKGCOMMON) -I . lib-pprz.cma -c $<

xml_get.out : lib-pprz.cma xml_get.cmo
	@echo OL $@
	$(Q)$(OCAMLC) $(INCLUDES) -o $@ -package str,xml-light -linkpkg -I . $^

tests : lib-pprz.cma $(TESTS_CMO)
	$(Q)$(OCAMLC) $(INCLUDES) -o $@ -package unix,str,xml-light,ivy -linkpkg -I . -dllpath . $^

%.o : %.c
	@echo OC $<
	$(Q)$(OCAMLC) -ccopt -fPIC $(INCLUDES) -package $(PKGCOMMON) -c $<

$(XCMO) $(XCMX): PKGCOMMON=$(XPKGCOMMON)


GTKCFLAGS := $(shell pkg-config --cflags gtk+-2.0)
ml_gtk_drag.o : ml_gtk_drag.c
	@echo OC $<
	$(Q)$(OCAMLC) $(INCLUDES) -package $(PKGCOMMON) -c -ccopt "$(GTKCFLAGS)" $<

%.cmo : %.ml
	@echo OC $<
	@if test $(findstring $<,$(PP_SRC)); then \
		$(OCAMLC) $(INCLUDES) $(PP_OPTS) -package $(PKGCOMMON) -c $<; \
	else \
		$(OCAMLC) $(INCLUDES) -package $(PKGCOMMON) -c $<; \
	fi;

%.cmx : %.ml
	@echo OOC $<
	@if test $(findstring $<,$(PP_SRC)); then \
		$(OCAMLOPT) $(INCLUDES) $(PP_OPTS) -package $(PKGCOMMON) -c $<; \
	else \
		$(OCAMLOPT) $(INCLUDES) -package $(PKGCOMMON) -c $<; \
	fi;

%.cmi : %.mli
	@echo OC $<
	$(Q)$(OCAMLC) $(XINCLUDES) $(INCLUDES) -package $(PKGCOMMON),$(XPKGCOMMON) $<

%.ml : %.mll
	@echo OCL $<
	$(Q)$(OCAMLLEX) $<

%.ml %.mli : %.mly
	@echo OCY $<
	$(Q)$(OCAMLYACC) $<

gtk_papget_editor.ml : widgets.glade
	@echo GLADE $@
	$(eval $@_TMP := $(shell $(MKTEMP)))
	$(Q)grep -v invisible_char $< > $($@_TMP)
	$(Q)lablgladecc2 -root papget_editor -hide-default $($@_TMP) | grep -B 1000000 "  end" > $@
	$(Q)rm -f $($@_TMP)

gtk_papget_text_editor.ml : widgets.glade
	@echo GLADE $@
	$(eval $@_TMP := $(shell $(MKTEMP)))
	$(Q)grep -v invisible_char $< > $($@_TMP)
	$(Q)lablgladecc2 -root table_text_editor -hide-default $($@_TMP) | grep -B 1000000 "  end" > $@
	$(Q)rm -f $($@_TMP)

gtk_papget_gauge_editor.ml : widgets.glade
	@echo GLADE $@
	$(eval $@_TMP := $(shell $(MKTEMP)))
	$(Q)grep -v invisible_char $< > $($@_TMP)
	$(Q)lablgladecc2 -root table_gauge_editor -hide-default $($@_TMP) | grep -B 1000000 "  end" > $@
	$(Q)rm -f $($@_TMP)

gtk_papget_led_editor.ml : widgets.glade
	@echo GLADE $@
	$(eval $@_TMP := $(shell $(MKTEMP)))
	$(Q)grep -v invisible_char $< > $($@_TMP)
	$(Q)$(Q)lablgladecc2 -root table_led_editor -hide-default $($@_TMP) | grep -B 1000000 "  end" > $@
	$(Q)rm -f $($@_TMP)

META.pprz: META.pprz.template
	@echo COPY $<
	$(shell sed -e 's/LABLGTK2GNOMECANVAS/$(LABLGTK2GNOMECANVAS)/g' -e 's/,NETTLS_GNUTLS/$(COMMA_NETTLS_GNUTLS)/g' $< > $@)

clean :
	$(Q)rm -f *~ *.cm* *.out *.opt .depend *.a *.o *.so tests gtk_papget_*.ml expr_parser.ml expr_parser.mli expr_lexer.ml expr_lexer.mli META.pprz

.PHONY: all byte native clean

#
# Dependencies
#

.depend: Makefile $(GEN_DEP)
	@echo DEPEND $@
	$(Q)$(OCAMLDEP) $(PP_OPTS) $(PP_SRC) > .depend
	$(Q)$(OCAMLDEP) $(filter-out $(PP_SRC), $(SRC) $(XSRC)) *.mli >> .depend

ifneq ($(MAKECMDGOALS),clean)
-include .depend
endif
