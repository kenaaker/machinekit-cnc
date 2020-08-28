
CROSS_COMPILE ?= arm-linux-gnueabihf-
export CROSS_COMPILE
SYSTEM_BUILD_ROOT ?= /home/kdaaker/nfsmounts/beaglebone-ai
SYSROOT_FLAGS ?= --sysroot=$(SYSTEM_BUILD_ROOT)
export SYSTEM_BUILD_ROOT
export SYSROOT_FLAGS

# see http://www.cmcrossroads.com/ask-mr-make/6535-tracing-rule-execution-in-gnu-make
# to trace make execution of make in more detail:
#     make VV=1
ifeq ("$(origin VV)", "command line")
    OLD_SHELL := $(SHELL)
    SHELL = $(warning Building $@$(if $<, (from $<))$(if $?, ($? newer)))$(OLD_SHELL)
endif

TOP_DIR?=$(shell pwd)

# Delete the default suffix rules
.SUFFIXES:
.PHONY: all userspace modules clean modclean depclean install python pythonclean userspace

ECHO=echo

BUILD_KBUILD = no

LDFLAGS += $(CROSS_COMPILE_FLAGS)

BASEPWD := $(shell pwd)
export BASEPWD

ifeq ($(origin PYTHONPATH),undefined)
	PYTHONPATH := $(BASEPWD)/lib/python
else
	PYTHONPATH := $(BASEPWD)/lib/python:${PYTHONPATH}
endif
export PYTHONPATH

# Configuration values.

# General stuff
CPPFLAGS :=
CC = $(CROSS_COMPILE)gcc
CC_FOR_BUILD = $(CROSS_COMPILE)gcc
OBJCOPY = $(CROSS_COMPILE)objcopy
LD = $(CROSS_COMPILE)ld

MSGFMT = /usr/bin/msgfmt
XGETTEXT = /usr/bin/xgettext

OPT := -O2

DEBUG := -g

CXXFLAGS := $(OPT) $(DEBUG)

ULFLAGS = -D_GNU_SOURCE -DLOCALE_DIR=\"$(localedir)\" -DPACKAGE=\"$(package)\"

# deps for xemc
XLIBS := -lX11 -lXaw -lXt
HAVE_XAW := yes

# General library directory
LIB_DIR=/afs/aaker.org/home/kdaaker/src/CAM_stuff/machinekit/machinekit-cnc/lib

# ncurses support for keystick
KEYSTICKLIBS := -lncurses
HAVE_NCURSES := yes

# emcweb UI
BUILD_EMCWEB := no

# Python-related
BUILD_PYTHON=yes
PYTHON_CPPFLAGS=-I/usr/include/python2.7
PYTHON_LIBS=-lpython2.7
BOOST_PYTHON_LIB=boost_python
SITEPY=/usr/lib/python2.7/dist-packages
CYTHON=/usr/bin/cython

# Install related stuff.
DESTDIR := /usr
localedir := cnc_msgs

# tcl tk cflags, includes and libs
TCL_DBGX=
TK_DBGX=
TCL_CFLAGS=-I=/usr/include/tcl8.6 -DUSE_TCL_STUBS
TCL_LIBS=-L=/usr/lib/arm-linux-gnueabihf -ltclstub8.6 -lX11 -lXss -lXext -lXft -lfontconfig -lfreetype -lfontconfig -lpthread -ldl -lz -lpthread -lm -L/usr/lib/arm-linux-gnueabihf -ltk8.6 -lGL -lXinerama

EXTRA_CFLAGS :=

# default target:  put before everything else
all: configs userspace modules


#############################################################################################

OBJDIR := objects
RTLIBDIR := ../rtlib

DEP = $(1) $(CPPFLAGS) $(SYSROOT_FLAGS) -MM -MG -MT "$(2)" $(4) -o $(3).tmp && \
	mv -f "$(3)".tmp "$(3)"

cc-option = $(shell \
    if $(CC) $(CPPFLAGS) $(SYSROOT_FLAGS) $(CFLAGS) $(1) -S -o /dev/null -xc /dev/null \
	    > /dev/null 2>&1; then \
	echo "$(1)"; \
    else \
	echo "$(2)"; \
    fi ;)

#############################################################################################


# Print 'entering' all the time
MAKEFLAGS += w

INCLUDE_PREFIX = .

# Create the variables with := so that subsequent += alterations keep it
# as a "substitute at assignment time" variable
TARGETS :=
PYTARGETS :=
# Add to this list directories to be created during 'make install'
INSTALL_DIRS :=

# Submakefiles from each of these directories will be included if they exist
SUBDIRS := \
	libnml \
	libnml/linklist \
	libnml/cms \
	libnml/os_intf \
	libnml/rcs \
	libnml/nml \
	libnml/buffer \
	libnml/posemath \
	emc/usr_intf/axis \
	emc/usr_intf/touchy \
	emc/usr_intf/stepconf \
	emc/usr_intf/pncconf \
	emc/usr_intf/gremlin \
	emc/usr_intf/gscreen \
	emc/usr_intf/gmoccapy \
	emc/usr_intf \
	emc/nml_intf \
	emc/task \
	emc/iotask \
	emc/kinematics \
	emc/motion \
	emc/tp \
	emc/canterp \
	emc/ini \
	emc/rs274ngc \
	emc/sai \
	emc \
	emc/pythonplugin \
	po

OBJ_DIRS := $(patsubst %,objects/%,$(SUBDIRS))
OBJ_DIRS += objects/emc/motion
OBJ_DIRS += objects/emc/usr_intf/axis/extensions
	
../lib/:
	mkdir $@

$(OBJDIR):
	mkdir $@
	
objects/libnml: | objects
	mkdir $@

objects/libnml/linklist: | objects/libnml
	mkdir $@

objects/libnml/cms: | objects/libnml
	mkdir $@

objects/libnml/os_intf: | objects/libnml
	mkdir $@

objects/libnml/rcs: | objects/libnml
	mkdir $@
	
objects/libnml/nml: | objects/libnml
	mkdir $@

objects/libnml/buffer: | objects/libnml
	mkdir $@

objects/libnml/posemath: | objects/libnml
	mkdir $@

objects/emc/usr_intf/axis: | objects/emc/usr_intf
	mkdir $@

objects/emc/usr_intf/touchy: | objects/emc/usr_intf
	mkdir $@

objects/emc/usr_intf/stepconf: | objects/emc/usr_intf
	mkdir $@

objects/emc/usr_intf/pncconf: | objects/emc/usr_intf
	mkdir $@

objects/emc/usr_intf/gremlin: | objects/emc/usr_intf
	mkdir $@

objects/emc/usr_intf/gscreen: | objects/emc/usr_intf
	mkdir $@

objects/emc/usr_intf/gmoccapy: | objects/emc/usr_intf
	mkdir $@

objects/emc/usr_intf/axis/extensions: | objects/emc/usr_intf/axis
	mkdir $@

objects/emc/usr_intf: | objects/emc
	mkdir $@

objects/emc/nml_intf: | objects/emc
	mkdir $@

objects/emc/task: | objects/emc
	mkdir $@

objects/emc/iotask: | objects/emc
	mkdir $@

objects/emc/kinematics: | objects/emc
	mkdir $@

objects/emc/tp: | objects/emc
	mkdir $@
	
objects/emc/canterp: | objects/emc
	mkdir $@\

objects/emc/ini: | objects/emc
	mkdir $@
	
objects/emc/rs274ngc: | objects/emc
	mkdir $@

objects/emc/sai: | objects/emc
	mkdir $@

objects/emc/motion: | objects/emc
	mkdir $@
	
objects/emc: | objects
	mkdir $@

objects/emc/pythonplugin: | objects/emc
	mkdir $@

objects/po: | objects
	mkdir $@

../share:
	mkdir $@
	
../share/locale: | ../share
	mkdir $@

../share/locale/%: po/%.po | ../share/locale
	mkdir $@

../share/locale/%: po/gmoccapy/%.po | ../share/locale
	mkdir $@
	
../share/locale/%/LC_MESSAGES: po/%.po | ../share/locale/%
	mkdir $@

../share/locale/%/LC_MESSAGES: po/gmoccapy/%.po | ../share/locale/%
	mkdir $@

#../share/locale/%/LC_MESSAGES: | ../share/locale/%
#	mkdir $@

.PRECIOUS: ../share/locale/% ../share/locale/%/LC_MESSAGES
	
# Each item in INCLUDES is transformed into a -I directive later on
# The top directory is always included
INCLUDES := .
INCLUDES += ../include
INCLUDES += ../../machinekit-hal/include
INCLUDES += ../../machinekit-hal/src/rtapi
INCLUDES += ../../machinekit-hal/src/inifile
INCLUDES += ../../machinekit-hal/src/hal/lib

USERSRCS :=
PROGRAMS :=

HAL_LIB := -L../../machinekit-hal/lib


# When used like $(call TOxxx, ...) these turn a list of source files
# into the corresponding list of object files, dependency files,
# or both.  When a source file has to be compiled with special flags,
# TOOBJSDEPS is used.  Confusingly, TOOBJSDEPS includes preprocessed source
# file names, but this is what allows 'make src.i' to produce proper
# preprocessed source when src.c needs a customized compile flag.
# See Submakefile.skel for an example.
TOOBJS = $(patsubst %.cc,objects/%$(2).o,$(patsubst %.c,objects/%$(2).o,$(1)))
TODEPS = $(patsubst %.cc,objects/%$(2).d,$(patsubst %.c,objects/%$(2).d,$(1)))
#-not sure, something fishy here -mah
#TODEPS = $(patsubst %.cc,objects/%.d,$(patsubst %.c,objects/%.d,$(1)))

TOOBJSDEPS = $(call TOOBJS,$(1),$(2)) $(call TODEPS, $(1),$(2))

# Same, but for RT objects in OBJDIR, compiled with -DRTAPI
TORTOBJS = $(patsubst %.cc,$(OBJDIR)/%$(2).o,$(patsubst %.c,$(OBJDIR)/%$(2).o,$(1)))

# This macro generates build rules for C components
# Example call & resulting rules:
# $$(eval $(call c_comp_build_rules,hal/components/encoder.o,hal/components/othersrc.o))
# obj-m += encoder.o
# encoder-objs := hal/components/encoder.o hal/components/othersrc.o
# $(RTLIBDIR)/encoder.so: \
#     hal/components/encoder.o hal/components/othersrc.o
define c_comp_build_rules
obj-m += $$(patsubst $$(dir $(1))%.o,%.o,$(1))
$$(patsubst $$(dir $(1))%.o,%-objs,$(1)) += $$(addprefix $(OBJDIR)/,$(1) $(2))
$$(patsubst $$(dir $(1))%.c,$(OBJDIR)/%.o,$$(addprefix $(OBJDIR)/,$(1) $(2))): 
$$(patsubst $$(dir $(1))%.o,$(RTLIBDIR)/%.so,$(1)): $$(addprefix $(OBJDIR)/,$(1) $(2))

endef

###############################################################################

SUBMAKEFILES := $(patsubst %,%/Submakefile,$(SUBDIRS))
#$(info SUBMAKEFILES=$(SUBMAKEFILES))
-include $(wildcard $(SUBMAKEFILES))
$(info All Submakefiles included)

# This checks that all the things listed in USERSRCS are either C files
# or C++ files
ASSERT_EMPTY = $(if $(1), $(error "Should be empty but is not: $(1)"))
$(call ASSERT_EMPTY,$(filter-out %.c %.cc %.js %.proto, $(USERSRCS)))

ifeq ($(BUILD_PYTHON),yes)
$(call TOOBJS,$(PYSRCS)) : EXTRA_CFLAGS += -fPIC -fno-strict-aliasing
USERSRCS += $(PYSRCS)
endif

# Find the list of object files for each type of source file
CUSERSRCS := $(filter %.c,$(USERSRCS))
CXXUSERSRCS := $(filter %.cc,$(USERSRCS))
CUSEROBJS = $(call TOOBJS,$(CUSERSRCS))
CXXUSEROBJS += $(call TOOBJS,$(CXXUSERSRCS))

# Find the list of build-arch object files for C sources
USERSRCS_BUILD :=
CUSERSRCS_BUILD := $(filter %.c,$(USERSRCS_BUILD))
CUSEROBJS_BUILD := $(call TOOBJS,$(CUSERSRCS_BUILD),_build)

# Find the dependency filenames, then include them all
DEPS := $(patsubst %.o,%.d,$(CUSEROBJS) $(CXXUSEROBJS) $(CUSEROBJS_BUILD))
READ_DEPS = $(wildcard $(DEPS))
$(shell echo 1>&2 Reading $(words $(READ_DEPS))/$(words $(DEPS)) dependency files)
-include $(READ_DEPS)
UNREAD_DEPS = $(filter-out $(READ_DEPS), $(DEPS))
$(shell echo 1>&2 Done reading dependencies)

# Each directory in $(INCLUDES) is passed as a -I directory when compiling.
INCLUDE := $(patsubst %,-I%, $(INCLUDES))
ifeq ($(BUILD_PYTHON),yes)
INCLUDE += $(PYTHON_CPPFLAGS)
endif

# trap build errors by making it possible to compare configure SHA vs build SHA
# see configure.ac GIT_CONFIG_SHA for the configure-time string
GIT_BUILD_SHA := $(shell ../scripts/get-git-sha)

# Compilation options.	Perhaps some of these should come from Makefile.inc? (CXXFLAGS now does)
INTEGER_OVERFLOW_FLAGS := -fwrapv -fno-strict-overflow
OPT +=  $(INTEGER_OVERFLOW_FLAGS)
DEBUG += -g -Wall -funwind-tables

CFLAGS := $(INCLUDE) -I$(BASEPWD)/include $(OPT) $(DEBUG) -DULAPI -std=gnu99 -fgnu89-inline

# enable runtime comparison of GIT_CONFIG_SHA (config.h) and GIT_BUILD_SHA
CFLAGS += -DGIT_BUILD_SHA=\"'$(GIT_BUILD_SHA)'\"

CFLAGS += $(CROSS_COMPILE_FLAGS)

BASE_CXXFLAGS:= $(INCLUDE) \
		$(CROSS_COMPILE_FLAGS) \
		$(CXXFLAGS) \
		$(OPT) \
		$(DEBUG) \
		-DGIT_BUILD_SHA=\"'$(GIT_BUILD_SHA)'\"

CXXFLAGS :=   $(BASE_CXXFLAGS) -DULAPI
RT_CXXFLAGS :=   $(BASE_CXXFLAGS)

CXX := $(CROSS_COMPILE)g++

RUN_IN_PLACE := yes
ifeq ($(RUN_IN_PLACE),yes)
LDFLAGS += -L$(LIB_DIR) -Wl,-rpath,../lib:../../machinekit-hal/lib
else
LDFLAGS += -Wl,-rpath-link,../lib
endif
LDFLAGS += -Wl,--no-as-needed

# Rules to make .o (object) files
$(CUSEROBJS) : objects/%.o: %.c | $(OBJ_DIRS)
	$(CC) -c $(CPPFLAGS) $(SYSROOT_FLAGS) $(CFLAGS) $(EXTRA_CFLAGS) -MMD -MF "${@:.o=.d}" -MT "$@" $< -o $@

# Rules to make .o (object) files, build arch
$(CUSEROBJS_BUILD) : objects/%_build.o: %.c | $(OBJ_DIRS)
	$(CC_FOR_BUILD) -c $(CFLAGS) $(EXTRA_CFLAGS) -MMD -MF "${@:.o=.d}" -MT "$@" $< -o $@

$(CXXUSEROBJS) : objects/%.o: %.cc | $(OBJ_DIRS)
	$(CXX) -c $(CPPFLAGS) $(SYSROOT_FLAGS) $(CXXFLAGS) $(EXTRA_CFLAGS) -MMD -MF "${@:.o=.d}" -MT "$@" $< -o $@

all:

# For each file to be copied to ../include, its location in the source tree
# is listed here.  Note that due to $(INCLUDE), defined above, the include
# files in the source tree are the ones used when building linuxcnc.  The copy
# in ../include is used when building external components of linuxcnc.
HEADERS := \
    config.h \
    emc/ini/emcIniFile.hh \
    emc/ini/iniaxis.hh \
    emc/ini/initool.hh \
    emc/ini/initraj.hh \
    emc/ini/inihal.hh \
    emc/kinematics/cubic.h \
    emc/kinematics/kinematics.h \
    emc/kinematics/genhexkins.h \
    emc/kinematics/genserkins.h \
    emc/kinematics/pumakins.h \
    emc/tp/tc.h \
    emc/tp/tc_types.h \
    emc/tp/tcq.h \
    emc/tp/tp.h \
    emc/tp/tp_types.h \
    emc/tp/spherical_arc.h \
    emc/tp/blendmath.h \
    emc/tp/tp_shared.h \
    emc/tp/tp_private.h \
    emc/motion/emcmotcfg.h \
    emc/motion/emcmotglb.h \
    emc/motion/motion.h \
    emc/motion/motion_id.h \
    emc/motion/usrmotintf.h \
    emc/motion/state_tag.h \
    emc/nml_intf/canon.hh \
    emc/nml_intf/canon_position.hh \
    emc/nml_intf/emctool.h \
    emc/nml_intf/emc.hh \
    emc/nml_intf/emc_nml.hh \
    emc/nml_intf/emccfg.h \
    emc/nml_intf/emcglb.h \
    emc/nml_intf/emcpos.h \
    emc/nml_intf/emcpose.h \
    emc/nml_intf/interp_return.hh \
    emc/nml_intf/interpl.hh \
    emc/nml_intf/motion_types.h \
    emc/nml_intf/debugflags.h \
    emc/rs274ngc/interp_internal.hh \
    emc/rs274ngc/interp_base.hh \
    emc/rs274ngc/modal_state.hh \
    emc/rs274ngc/rs274ngc.hh \
    libnml/buffer/locmem.hh \
    libnml/buffer/memsem.hh \
    libnml/buffer/phantom.hh \
    libnml/buffer/physmem.hh \
    libnml/buffer/recvn.h \
    libnml/buffer/rem_msg.hh \
    libnml/buffer/sendn.h \
    libnml/buffer/shmem.hh \
    libnml/buffer/tcpmem.hh \
    libnml/cms/cms.hh \
    libnml/cms/cms_aup.hh \
    libnml/cms/cms_cfg.hh \
    libnml/cms/cms_dup.hh \
    libnml/cms/cms_srv.hh \
    libnml/cms/cms_up.hh \
    libnml/cms/cms_user.hh \
    libnml/cms/cms_xup.hh \
    libnml/cms/cmsdiag.hh \
    libnml/cms/tcp_opts.hh \
    libnml/cms/tcp_srv.hh \
    libnml/linklist/linklist.hh \
    libnml/nml/cmd_msg.hh \
    libnml/nml/nml.hh \
    libnml/nml/nml_mod.hh \
    libnml/nml/nml_oi.hh \
    libnml/nml/nml_srv.hh \
    libnml/nml/nml_type.hh \
    libnml/nml/nmldiag.hh \
    libnml/nml/nmlmsg.hh \
    libnml/nml/stat_msg.hh \
    libnml/os_intf/_sem.h \
    libnml/os_intf/sem.hh \
    libnml/os_intf/_shm.h \
    libnml/os_intf/shm.hh \
    libnml/os_intf/_timer.h \
    libnml/os_intf/timer.hh \

## the "headers" target installs all the header files in ../include
.PHONY: headers
SUBDIRECTORY_HEADERS := $(patsubst %,../include/%,$(foreach h,$(HEADERS),$(notdir $h)))

# install header files as part of the build
TARGETS += headers

# Add headers to this list that need to go into subdirectories; their
# respective Submakefiles will need to provide rules to copy them into
# ../install.
headers: $(SUBDIRECTORY_HEADERS)

# Add converting of %.po files
TARGETS += $(patsubst po/%.po, ../share/locale/%/LC_MESSAGES/linuxcnc.mo, $(wildcard po/*.po))
TARGETS += $(patsubst po/gmoccapy/%.po, ../share/locale/%/LC_MESSAGES/gmoccapy.mo, $(wildcard po/gmoccapy/*.po))
TARGETS += $(patsubst po/%.po, objects/%.msg, $(wildcard po/*.po))

# tooledit as standalone app:
TARGETS += ../bin/tooledit
../bin/tooledit: Makefile
	rm -f $@
	(echo "#!$(TCLSH)")>|$@
	(echo "source $(EMC2_TCL_LIB_DIR)/tooledit.tcl")>>$@
	(echo "standalone_tooledit")>>$@
	chmod +x $@

# ngcgui as standalone app:
TARGETS += ../bin/ngcgui
../bin/ngcgui: Makefile
	rm -f $@
	(echo "#!$(TCLSH)")>|$@
	(echo "source $(EMC2_TCL_LIB_DIR)/ngcgui.tcl")>>$@
	(echo "::ngcgui::standalone_ngcgui")>>$@
	chmod +x $@

# pyngcgui as standalone app:
TARGETS += ../bin/pyngcgui
../bin/pyngcgui: Makefile
	rm -f $@
	(echo "#!$(PYTHON)")>|$@
	(echo "import pyngcgui")>>$@
	(echo "pyngcgui.standalone_pyngcgui()")>>$@
	chmod +x $@

# gremlin_view as standalone app:
TARGETS += ../bin/gremlin_view
../bin/gremlin_view: Makefile
	rm -f $@
	(echo "#!$(PYTHON)")>|$@
	(echo "import gremlin_view")>>$@
	(echo "gremlin_view.standalone_gremlin_view()")>>$@
	chmod +x $@


# And make userspace depend on $(TARGETS)
userspace: $(TARGETS)

ifeq ($(BUILD_PYTHON),yes)

pythonclean:
	rm -f $(PYTARGETS)
	find ../lib/python -name '*.so' -exec rm {} +

python: $(PYTARGETS)

userspace: python

clean: pythonclean

endif

# These rules clean things up.	'modclean' cleans files generated by 'modules'
# (except that it doesn't remove the modules that were copied to rtlib)
# 'clean' cleans everything but dependency files, and 'depclean' cleans them
# too.
modclean:
	-rm -f $(RTLIBDIR)/*.so

depclean:
	-rm -rf objects/*.d

docclean:
#	-rm -f ../man/man9/*.9icomp
#	-rm -f ../man/doc/man9/*.asciidoc

clean: depclean modclean docclean
	find . -name '*.o' |xargs rm -f
	rm -rf objects
	rm -f $(TARGETS)
	rm -rf ../rtlib ../libexec
	rm -f ../etc/linuxcnc/rtapi.ini
	rm -f $(COPY_CONFIGS)
	rm -f $(RTLIBDIR)/*.so
	rm -rf ../share/linuxcnc/stepconf
	rm -rf ../libexec
	rm -rf ../include/*
	rm -rf ../lib/*
	
# So that nothing is built as root, this rule does not depend on the touched
# files 
#fix_perms = test -f $(1) && chown root $(1) && chmod 4750 $(1) || true
#
#ifeq ($(BUILD_DRIVERS),yes)
#ifeq ($(HAVE_KERNEL_THREADS),yes)
#CHECK_KFLAV_SWITCH=-k
#endif
#setuid:
#ifeq ($(HAVE_KERNEL_THREADS),yes)
#	$(call fix_perms,../libexec/linuxcnc_module_helper)
#endif
#	$(call fix_perms,../libexec/pci_read)
#	$(call fix_perms,../libexec/pci_write)
#	$(foreach f,$(BUILD_THREAD_FLAVORS),\
#	    $(call fix_perms,../libexec/rtapi_app_$(f));)
# check system configuration:  logging, ulimits, udev
#	@-../scripts/check-system-configuration.sh $(CHECK_KFLAV_SWITCH)
#else
setuid:
	@echo "'make setuid' is not needed if hardware drivers are not used"


# These rules allows a header file from this directory to be installed into
# ../include.  A pair of rules like these will exist in the Submakefile
# of each file that contains headers.
../include/%.h: %.h
	$(ECHO) Copying header file $@
	-cp $^ $@
../include/%.hh: %.hh
	$(ECHO) Copying header file $@
	-cp $^ $@


DIR=install -d -m 0755 -o root
FILE=install -m 0644 -o root
TREE=cp -dR
CONFIGFILE=install -m 0644
EXE=install -m 0755 -o root
SETUID=install -m 4755 -o root
GLOB=$(wildcard $(1))

MENUS = ../share/menus/CNC.menu \
    ../share/desktop-directories/cnc.directory \
    ../share/applications/linuxcnc.desktop \

install-menus install-menu: $(MENUS)
	cp $< $(HOME)/.config/menus/applications-merged

DOCS_HELP=$(call GLOB,../help/*)
NC_FILES=$(filter-out %/butterfly.ngc,$(call GLOB,../nc_files/*))
TCL=$(call GLOB,../tcl/*.tcl)
TCL_BIN=$(call GLOB,../tcl/bin/*.tcl) ../tcl/bin/popimage

install-test:
	@if type -path dpkg-query > /dev/null 2>&1 ; then  \
		if dpkg-query -S $(DESTDIR)/usr/bin/linuxcnc > /dev/null 2>&1 || dpkg-query -S $(DESTDIR)/usr/bin/emc > /dev/null 2>&1 ; then \
			echo "*** Error: Package version installed in $(DESTDIR)/usr"; \
			echo "Use './configure --enable-run-in-place' or uninstall the linuxcnc package"; \
			echo "before installing."; \
			exit 1; \
		fi \
	fi

install: install-test install-kernel-indep
	$(ECHO) "Installed in $(DESTDIR) with prefix $(prefix)"

install-dirs:
	$(DIR)  $(DESTDIR)$(EMC2_RTLIB_DIR) \
		$(DESTDIR)$(sysconfdir)/linuxcnc $(DESTDIR)$(bindir) \
		$(DESTDIR)$(includedir)/linuxcnc \
		$(DESTDIR)$(docdir) $(DESTDIR)$(ncfilesdir) \
		$(DESTDIR)$(EMC2_LIBEXEC_DIR) \
		$(DESTDIR)/etc/X11/app-defaults $(DESTDIR)$(tcldir)/bin \
		$(DESTDIR)$(tcldir)/scripts \
		$(DESTDIR)$(tcldir)/msgs \
		$(DESTDIR)$(localedir)/de/LC_MESSAGES \
		$(DESTDIR)$(datadir)/axis/images \
		$(DESTDIR)$(datadir)/axis/tcl \
		$(DESTDIR)$(datadir)/gscreen/images \
		$(DESTDIR)$(datadir)/gscreen/skins \
		$(DESTDIR)$(datadir)/gmoccapy/images \
		$(DESTDIR)$(datadir)/glade3/catalogs \
		$(DESTDIR)$(datadir)/glade3/pixmaps \
		$(DESTDIR)$(datadir)/gtksourceview-2.0/language-specs \
		$(DESTDIR)$(datadir)/linuxcnc/stepconf \
		$(DESTDIR)$(datadir)/linuxcnc/pncconf/pncconf-help \
		$(DESTDIR)$(sysconfdir)/rsyslog.d \
		$(DESTDIR)$(sysconfdir)/security/limits.d \
		$(DESTDIR)$(sysconfdir)/udev/rules.d \
		$(DESTDIR)$(sampleconfsdir) \
		$(addprefix $(DESTDIR), $(INSTALL_DIRS))
ifeq ($(BUILD_EMCWEB),yes)
	$(DIR)	$(DESTDIR)$(datadir)/linuxcnc/doc-root/css/images \
		$(DESTDIR)$(datadir)/linuxcnc/doc-root/js \
		$(DESTDIR)$(datadir)/linuxcnc/doc-root/res
endif
#ifeq ($(USERMODE_PCI),yes)
#	$(DIR)	$(DESTDIR)$(includedir)/linuxcnc/userpci
#endif

install-kernel-indep: install-dirs
	$(FILE) objects/*.msg $(DESTDIR)$(tcldir)/msgs
	$(FILE) ../*.png ../*.gif $(DESTDIR)$(datadir)/linuxcnc

	# install all the sample configs, including subdirs (tar is required on debian systems, and common on others)
	((cd ../configs && tar --exclude CVS --exclude .cvsignore --exclude .gitignore -cf - .) | (cd $(DESTDIR)$(sampleconfsdir) && tar -xf -))

	$(EXE) $(filter ../bin/%,$(TARGETS)) $(DESTDIR)$(bindir)
	$(EXE) ../scripts/linuxcnc $(DESTDIR)$(bindir)
	$(EXE) ../scripts/machinekit $(DESTDIR)$(bindir)
	$(EXE) ../scripts/linuxcnc_info $(DESTDIR)$(bindir)
# 	$(EXE) ../scripts/linuxcncmkdesktop $(DESTDIR)$(bindir)
	$(EXE) ../scripts/latency-test $(DESTDIR)$(bindir)
	$(FILE) $(filter ../lib/%.a ../lib/%.so.0,$(TARGETS)) $(DESTDIR)$(libdir)
	cp --no-dereference $(filter ../lib/%.so, $(TARGETS)) $(DESTDIR)$(libdir)
#	# don't run ldconfig under fakeroot (silence dpkg-build warning)
	-test -n "$$FAKED_MODE" || ldconfig $(DESTDIR)$(libdir)
	$(FILE) $(HEADERS) $(DESTDIR)$(includedir)/linuxcnc/
	$(FILE) $(DOCS_HELP) $(DESTDIR)$(docdir)
	$(TREE) $(NC_FILES) $(DESTDIR)$(ncfilesdir)
	$(EXE) ../nc_files/M101 $(DESTDIR)$(ncfilesdir)
	$(FILE) ../tcl/TkLinuxCNC $(DESTDIR)/etc/X11/app-defaults
	$(FILE) ../app-defaults/XEmc $(DESTDIR)/etc/X11/app-defaults
	$(FILE) Makefile.modinc $(DESTDIR)$(datadir)/linuxcnc
	$(FILE) Makefile.inc $(DESTDIR)$(datadir)/linuxcnc
	$(EXE) $(TCL) $(DESTDIR)$(tcldir)
	$(FILE) ../tcl/linuxcnc.so $(DESTDIR)$(tcldir)
	$(FILE) ../tcl/pkgIndex.tcl $(DESTDIR)$(tcldir)
	$(EXE) $(TCL_BIN) $(DESTDIR)$(tcldir)/bin
	$(FILE) ../tcl/scripts/balloon.tcl ../tcl/scripts/emchelp.tcl $(DESTDIR)$(tcldir)/scripts
	$(EXE) ../tcl/scripts/Set_Coordinates.tcl $(DESTDIR)$(tcldir)/scripts
	$(FILE) ../share/linuxcnc/*.glade $(DESTDIR)$(prefix)/share/linuxcnc
	$(FILE) ../share/linuxcnc/stepconf/*.glade $(DESTDIR)$(prefix)/share/linuxcnc/stepconf
	$(FILE) ../share/linuxcnc/touchy.glade $(DESTDIR)$(prefix)/share/linuxcnc
	$(FILE) ../share/linuxcnc/gscreen.glade $(DESTDIR)$(prefix)/share/linuxcnc
	$(FILE) ../share/linuxcnc/gscreen2.glade $(DESTDIR)$(prefix)/share/linuxcnc
	$(FILE) ../share/gmoccapy/gmoccapy.glade $(DESTDIR)$(prefix)/share/gmoccapy
	$(FILE) ../share/linuxcnc/pncconf.glade $(DESTDIR)$(prefix)/share/linuxcnc
	$(FILE) ../share/linuxcnc/gremlin_view.ui $(DESTDIR)$(prefix)/share/linuxcnc
	$(FILE) ../share/linuxcnc/popupkeyboard.ui $(DESTDIR)$(prefix)/share/linuxcnc
	$(FILE) ../configs/common/linuxcnc.nml $(DESTDIR)$(prefix)/share/linuxcnc
	$(FILE) ../src/emc/usr_intf/pncconf/pncconf-help/*.txt $(DESTDIR)$(prefix)/share/linuxcnc/pncconf/pncconf-help
	$(FILE) ../src/emc/usr_intf/pncconf/pncconf-help/*.png $(DESTDIR)$(prefix)/share/linuxcnc/pncconf/pncconf-help

	$(EXE) ../bin/profile_axis $(DESTDIR)$(bindir)

ifeq ($(BUILD_PYTHON),yes)
install-kernel-indep: install-python
install-python: install-dirs
	$(DIR) $(DESTDIR)$(SITEPY) $(DESTDIR)$(SITEPY)/rs274
	$(DIR) $(DESTDIR)$(SITEPY)/touchy
	$(DIR) $(DESTDIR)$(SITEPY)/gscreen
	$(DIR) $(DESTDIR)$(SITEPY)/gmoccapy
	$(DIR) $(DESTDIR)$(SITEPY)/stepconf
	$(DIR) $(DESTDIR)$(SITEPY)/machinekit
	$(DIR) $(DESTDIR)$(SITEPY)/fdm
	$(DIR) $(DESTDIR)$(SITEPY)/fdm/config
	$(DIR) $(DESTDIR)$(datadir)/fdm/thermistor_tables
	$(FILE) ../lib/python/*.py ../lib/python/*.so $(DESTDIR)$(SITEPY)
	$(FILE) ../lib/python/rs274/*.py $(DESTDIR)$(SITEPY)/rs274
	$(FILE) ../lib/python/touchy/*.py $(DESTDIR)$(SITEPY)/touchy
	$(FILE) ../lib/python/gscreen/*.py $(DESTDIR)$(SITEPY)/gscreen
	$(FILE) ../lib/python/gmoccapy/*.py $(DESTDIR)$(SITEPY)/gmoccapy
	$(FILE) ../lib/python/stepconf/*.py $(DESTDIR)$(SITEPY)/stepconf
	$(FILE) ../lib/python/fdm/*.py $(DESTDIR)$(SITEPY)/fdm
	$(FILE) ../lib/python/fdm/config/*.py $(DESTDIR)$(SITEPY)/fdm/config
	$(EXE) ../bin/axis $(DESTDIR)$(bindir)
	$(EXE) ../bin/axis-remote $(DESTDIR)$(bindir)
	$(EXE) ../bin/debuglevel $(DESTDIR)$(bindir)
	$(EXE) ../bin/linuxcnctop $(DESTDIR)$(bindir)
	$(EXE) ../bin/mdi $(DESTDIR)$(bindir)
	$(EXE) ../bin/hal_manualtoolchange $(DESTDIR)$(bindir)
	$(EXE) ../bin/image-to-gcode $(DESTDIR)$(bindir)
	$(EXE) ../bin/touchy $(DESTDIR)$(bindir)
	$(EXE) ../bin/gscreen $(DESTDIR)$(bindir)
	$(EXE) ../bin/gmoccapy $(DESTDIR)$(bindir)
	$(EXE) ../bin/stepconf $(DESTDIR)$(bindir)
	$(EXE) ../bin/pncconf $(DESTDIR)$(bindir)	
	$(FILE) ../share/linuxcnc/machinekit-wizard.gif $(DESTDIR)$(prefix)/share/linuxcnc
	$(FILE) emc/usr_intf/axis/etc/axis_light_background $(DESTDIR)$(docdir)
	$(FILE) emc/usr_intf/axis/README $(DESTDIR)$(docdir)/README.axis
	$(FILE) ../share/axis/images/*.png ../share/axis/images/*.gif ../share/axis/images/*.xbm ../share/axis/images/*.ngc $(DESTDIR)$(datadir)/axis/images
	$(FILE) ../share/axis/tcl/*.tcl $(DESTDIR)$(datadir)/axis/tcl
	$(FILE) ../share/gscreen/images/*.gif $(DESTDIR)$(datadir)/gscreen/images
	$(TREE) ../share/gscreen/skins/* $(DESTDIR)$(datadir)/gscreen/skins
	$(FILE) ../share/gmoccapy/images/*.png ../share/gmoccapy/images/*.gif ../share/gmoccapy/images/*.svg $(DESTDIR)$(datadir)/gmoccapy/images
	$(FILE) ../share/fdm/thermistor_tables/*.txt $(DESTDIR)$(datadir)/fdm/thermistor_tables
	$(EXE) ../bin/gremlin $(DESTDIR)$(bindir)
	$(EXE) ../bin/lintini $(DESTDIR)$(bindir)
	$(EXE) ../bin/teach-in $(DESTDIR)$(bindir)
	$(EXE) ../bin/tracking-test $(DESTDIR)$(bindir)
	$(EXE) ../scripts/gcode-to-ngc $(DESTDIR)$(bindir)
	$(EXE) ../scripts/g1-to-g23 $(DESTDIR)$(bindir)
endif


	# RTAPI modules:  install userland flavor .so modules into
	# e.g. /usr/lib/linuxcnc/modules

	echo Installing modules
	$(DIR) $(DESTDIR)$(EMC2_RTLIB_BASE_DIR)/modules
	$(FILE) ../rtlib/modules/* $(DESTDIR)$(EMC2_RTLIB_BASE_DIR)/modules

CONF=../configs
COMMON=$(CONF)/common
CONFILES=$(addsuffix /$(1), $(filter-out $(COMMON) $(CONF),\
                                         ${shell find ${CONF} -type d -print}))

.PHONY: configs
COPY_CONFIGS = $(patsubst %,../configs/%/core_stepper.hal, \
                sim \
                by_interface/parport/stepper \
                by_machine/sherline/Sherline3Axis \
                by_machine/sherline/SherlineLathe \
                by_machine/cooltool \
                by_interface/parport/classicladder/demo_step_cl)

COPY_CONFIGS += $(patsubst %,../configs/%/core_servo.hal, \
                sim \
                by_interface/vitalsystems \
                by_interface/servotogo \
                by_interface/vigilant)

COPY_CONFIGS += $(patsubst %,../configs/%/core_sim.hal, \
                sim)

COPY_CONFIGS += $(patsubst %,../configs/%/core_sim9.hal, \
                sim)

COPY_CONFIGS += $(patsubst %,../configs/%/axis_manualtoolchange.hal, \
                sim \
                by_interface/pluto/lathe-pluto\
                by_interface/parport/plasma-thc-sim)

show_configs_copy_stuff:
	echo CONFILES=$(CONFILES)
	echo CALL:$(call CONFILES,axis_manualtoolchange.hal)
	echo CALL:$(call CONFILES,core_stepper.hal)
	echo CALL:$(call CONFILES,core_servo.hal)
	echo CALL:$(call CONFILES,core_sim.hal)
	echo CALL:$(call CONFILES,core_sim9.hal)
	echo COPY_CONFIGS=$(COPY_CONFIGS)

configs: $(COPY_CONFIGS)

$(call CONFILES,axis_manualtoolchange.hal): %/axis_manualtoolchange.hal: ../configs/common/axis_manualtoolchange.hal
	-cp $< $@
$(call CONFILES,core_stepper.hal): %/core_stepper.hal: ../configs/common/core_stepper.hal
	-cp $< $@
$(call CONFILES,core_servo.hal): %/core_servo.hal: ../configs/common/core_servo.hal
	-cp $< $@
$(call CONFILES,core_sim.hal): %/core_sim.hal: ../configs/common/core_sim.hal
	-cp $< $@
$(call CONFILES,core_sim9.hal): %/core_sim9.hal: ../configs/common/core_sim9.hal
	-cp $< $@

#################################################################################################################

EXTRA_CFLAGS = \
	-D__MODULE__ \
	-I$(INCLUDE_PREFIX) \
	-I$(INCLUDE_PREFIX)/libnml/linklist \
	-I$(INCLUDE_PREFIX)/libnml/cms \
	-I$(INCLUDE_PREFIX)/libnml/os_intf \
	-I$(INCLUDE_PREFIX)/libnml/rcs \
	-I$(INCLUDE_PREFIX)/libnml/nml \
	-I$(INCLUDE_PREFIX)/libnml/buffer \
	-I$(INCLUDE_PREFIX)/libnml/posemath \
	-I$(INCLUDE_PREFIX)/emc/nml_intf \
	-I$(INCLUDE_PREFIX)/emc/kinematics \
	-I$(INCLUDE_PREFIX)/emc/motion \
	-I$(INCLUDE_PREFIX)/emc/tp \
	-DSEQUENTIAL_SUPPORT -DHAL_SUPPORT -DDYNAMIC_PLCSIZE -DRT_SUPPORT \
	-DOLD_TIMERS_MONOS_SUPPORT -DMODBUS_IO_MASTER

TOCOMPRTOBJS = $(foreach file,$($(patsubst %.o,%,$(1))-objs), $(OBJDIR)/$(file))

###############################################################################################

# Link shared libs
../lib/%.so: ../lib/%.so.0
	$(ECHO) Symlinking $(notdir $<) to $(notdir $@)
	ln -sf $(notdir $<) $@
# - Be sure the setuid message comes last
ALL_SHLIBS=$(filter ../%.so.0,$(TARGETS)) $(filter ../%.so,$(TARGETS))
modules: $(ALL_SHLIBS)

