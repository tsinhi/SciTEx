# Make file for SciTE on Linux or compatible OS
# Copyright 1998-2010 by Neil Hodgson <neilh@scintilla.org>
# The License.txt file describes the conditions under which this software may be distributed.
# This makefile assumes GCC 4.x is used and changes will be needed to use other compilers.
# GNU make does not like \r\n line endings so should be saved to CVS in binary form.

srcdir ?= .

.SUFFIXES: .cxx .o .h .a .c
ifdef CLANG
# Can choose aspect to sanitize: address and undefined can simply change SANITIZE but for
# thread also need to create Position Independent Executable -> search online documentation
SANITIZE = address
#SANITIZE = undefined
CXX = clang++ -fsanitize=$(SANITIZE) -Wno-deprecated-register
CC = clang -fsanitize=$(SANITIZE)
CCWARNINGS = -Wno-empty-body -Wno-string-plus-int
else
MISLEADING=-Wno-misleading-indentation
endif

ifdef GTK3
GTKVERSION=gtk+-3.0
else
GTKVERSION=gtk+-2.0
endif

# For the Gnome desktop stuff to work, prefix must point to where Gnome thinks it is.
CONFIGFLAGS:=$(shell pkg-config --cflags $(GTKVERSION))
CONFIGLIB:=$(shell pkg-config --libs $(GTKVERSION) gthread-2.0 gmodule-no-export-2.0)
CONFIGTHREADS:=
gnomeprefix:=$(shell pkg-config --variable=prefix $(GTKVERSION) 2>/dev/null)
ifndef prefix
ifdef gnomeprefix
  prefix=$(gnomeprefix)
else
  prefix=/usr
endif
endif
datadir=$(prefix)/share
pixmapdir=$(datadir)/pixmaps
bindir=$(prefix)/bin
SYSCONF_PATH=$(prefix)/share/scite

INSTALL=install

PROG	= $(srcdir)/../bin/SciTE

all: $(PROG)

vpath %.h $(srcdir) $(srcdir)/../src $(srcdir)/../../scintilla/include
vpath %.cxx $(srcdir) $(srcdir)/../src

INCLUDEDIRS=-I $(srcdir)/../../scintilla/include -I $(srcdir)/../src
CXXBASEFLAGS=-W -Wall -pedantic -DGTK -DSCI_LEXER -DPIXMAP_PATH=\"$(pixmapdir)\" -DSYSCONF_PATH=\"$(SYSCONF_PATH)\" $(INCLUDEDIRS)

ifdef DEBUG
CXXTFLAGS=-DDEBUG -g $(CXXBASEFLAGS)
else
CXXTFLAGS=-DNDEBUG -Os $(CXXBASEFLAGS)
endif

ifndef NO_LUA
LUA_CORE_OBJS = lapi.o lcode.o lctype.o ldebug.o ldo.o ldump.o lfunc.o lgc.o llex.o \
		lmem.o lobject.o lopcodes.o lparser.o lstate.o lstring.o \
		ltable.o ltm.o lundump.o lvm.o lzio.o

LUA_LIB_OBJS =	lauxlib.o lbaselib.o lbitlib.o lcorolib.o ldblib.o liolib.o lmathlib.o ltablib.o \
		lstrlib.o loadlib.o loslib.o linit.o lutf8lib.o

LUA_OBJS = LuaExtension.o $(LUA_CORE_OBJS) $(LUA_LIB_OBJS)

vpath %.c $(srcdir)/../lua/src

INCLUDEDIRS=-I $(srcdir)/../../scintilla/include -I $(srcdir)/../src -I$(srcdir)/../lua/src

.c.o:
	$(CC) $(INCLUDEDIRS) $(MISLEADING) -DLUA_USE_POSIX $(LUACFLAGS) $(CXXTFLAGS) $(CFLAGS) $(CCWARNINGS) -c $< -o $@

else
CXXTFLAGS+=-DNO_LUA
endif

CXXTFLAGS:=$(CXXTFLAGS)

UNAME:=$(shell uname -s)
ifeq ($(UNAME), Linux)
LIBDL:=-ldl
LUACFLAGS:=-DLUA_USE_DLOPEN
endif
ifneq (,$(findstring GNU,$(UNAME)))
LIBDL:=-ldl
LUACFLAGS:=-DLUA_USE_DLOPEN
endif
ifeq ($(UNAME), Darwin)
CXXTFLAGS:=$(CXXTFLAGS) -Dunix
LUACFLAGS:=-DLUA_USE_DLOPEN
endif

.cxx.o:
	$(CXX) --std=c++17 $(CONFIGFLAGS) $(CXXTFLAGS) $(CXXFLAGS) -c $< -o $@

clean:
	rm -f *.o *.plist $(PROG)

analyze:
	clang --analyze --std=c++17 $(CONFIGFLAGS) $(CXXTFLAGS) $(CXXFLAGS) $(srcdir)/*.cxx $(srcdir)/../src/*.cxx
	# Could analyze Lua implementation as well but won't be fixing the warnings so its just noise
	#clang --analyze $(CONFIGFLAGS) $(CXXTFLAGS) $(CXXFLAGS) *.cxx ../src/*.cxx ../lua/src/*.c

depend deps.mak:
	python AppDepGen.py

# make should be run in ../../scintilla/gtk to compile all the lexers.
COMPLIB=$(srcdir)/../../scintilla/bin/scintilla.a

# To almost make lua.vers (needs header and footer added) which is only needed after updating Lua:
# nm -g ../bin/SciTE | grep lua | awk '{print "\t\t" $3 ";"}' >lua2.vers

$(PROG): SciTEGTK.o GUIGTK.o ScintillaCall.o ScintillaWindow.o Widget.o \
FilePath.o EditorConfig.o SciTEBase.o FileWorker.o Cookie.o Credits.o SciTEBuffers.o SciTEIO.o StringList.o \
ExportHTML.o ExportPDF.o ExportRTF.o ExportTEX.o ExportXML.o \
MatchMarker.o StringHelpers.o \
PropSetFile.o MultiplexExtension.o DirectorExtension.o SciTEProps.o StyleDefinition.o StyleWriter.o Utf8_16.o \
	JobQueue.o GTKMutex.o IFaceTable.o $(COMPLIB) $(LUA_OBJS)
	$(CXX) `$(CONFIGTHREADS)` -rdynamic -Wl,--as-needed -Wl,--version-script $(srcdir)/lua.vers -DGTK $^ -o $@ $(CONFIGLIB) $(LIBDL) $(LDLIBS) -lm -lstdc++

# Automatically generate header dependencies with "make deps"
include deps.mak

# The two last install commands will fail if Gnome is not installed or is not at $(prefix).
# This is OK - just means no SciTE in the Gnome Applications menu
# Dead:	install -D SciTEGTK.properties $(SYSCONF_PATH)/SciTEGlobal.properties
install:
	$(INSTALL) -m 755 -d $(DESTDIR)$(bindir) $(DESTDIR)$(SYSCONF_PATH)

	$(INSTALL) -m 755 $(PROG) $(DESTDIR)$(bindir)

	for files in $(srcdir)/../src/*.properties $(srcdir)/../doc/*.html $(srcdir)/../doc/SciTEIco.png $(srcdir)/../doc/PrintHi.png $(srcdir)/../doc/SciTEIndicators.png; \
	do \
		$(INSTALL) -m 644 $$files $(DESTDIR)$(SYSCONF_PATH); \
	done

ifdef gnomeprefix
	$(INSTALL) -m 755 -d $(DESTDIR)$(datadir)/applications $(DESTDIR)$(pixmapdir)
	$(INSTALL) -m 644 $(srcdir)/SciTE.desktop $(DESTDIR)$(datadir)/applications/SciTE.desktop
	$(INSTALL) -m 644 $(srcdir)/Sci48M.png $(DESTDIR)$(pixmapdir)/Sci48M.png
endif

uninstall:
	rm -f $(DESTDIR)$(bindir)/SciTE
	rm -rf $(DESTDIR)$(SYSCONF_PATH)
ifdef gnomeprefix
	rm -f $(DESTDIR)$(datadir)/applications/SciTE.desktop
	rm -f $(DESTDIR)$(pixmapdir)/Sci48M.png
endif
