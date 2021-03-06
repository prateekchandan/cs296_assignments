#variables
TARGET=cs296_17_exe
TARGETNEW=cs296_17_exelib
SHARED_LIB=TRUE

#programs
ECHO=/bin/echo
CAT=cat

# Project Paths
PROJECT_ROOT=./
EXTERNAL_ROOT=$(PROJECT_ROOT)/external
SRCDIR = $(PROJECT_ROOT)/src
OBJDIR = $(PROJECT_ROOT)/myobjs
BINDIR = $(PROJECT_ROOT)/mybins
INCLUDEDIR= $(EXTERNAL_ROOT)/include
LIBDIR= $(EXTERNAL_ROOT)/lib
LIBSDIR=$(PROJECT_ROOT)/mylibs

#compiler options
CC=g++
PRINTF=printf

#####################################
OK_STRING="[OK]"
ERR_STRING="[ERRORS]"
WARN_STRING="[WARNINGS]"
OK_FMT="%30s\n"
ERR_FMT="%30s\n"
WARN_FMT="%30s\n"
######################################


# Library Paths
BOX2D_ROOT=$(EXTERNAL_ROOT)
GLUI_ROOT=/usr
GL_ROOT=/usr/include/

#Libraries
LIBS = -lBox2D -lglui -lglut -lGLU -lGL

# Compiler and Linker flags
CPPFLAGS =-g -O3 -Wall -fno-strict-aliasing
CPPFLAGS+=-I $(BOX2D_ROOT)/include -I $(GLUI_ROOT)/include
LDFLAGS+=-L $(BOX2D_ROOT)/lib -L $(GLUI_ROOT)/lib

#dynamic variable
SRCS := $(wildcard $(SRCDIR)/*.cpp)
INCS := $(wildcard $(SRCDIR)/*.hpp)
OBJS := $(SRCS:$(SRCDIR)/%.cpp=$(OBJDIR)/%.o)
FILTERED = $(filter-out $(OBJDIR)/main.o,$(OBJS))

.PHONY: all setup $(OBJS) exe

all : setup

setup:
	@mkdir -p myobjs mybins mylibs
	@if test -e $(INCLUDEDIR)/Box2D -a -e $(LIBDIR)/Box2D; \
		then $(ECHO) "Already present file .. ";\
		else tar -xzf $(EXTERNAL_ROOT)/src/Box2D.tgz -C $(EXTERNAL_ROOT)/src\
		&& cd $(EXTERNAL_ROOT)/src/Box2D \
		&& mkdir -p build296 \
		&& cd build296\
		&& cmake ../ \
		&& make \
		&& make install \
		&& cd ../../../../; \
	fi;

exe: $(BINDIR)/$(TARGET)

$(BINDIR)/$(TARGET): $(OBJS)
	@$(CC) -o $@ $(LDFLAGS) $(OBJS) $(LIBS) 2> temp.log || touch temp.err
	@if test -e temp.err; \
		then $(PRINTF) $(ERR_FMT) $(ERR_STRING) && $(CAT) temp.log; \
		elif test -s temp.log; \
		then $(PRINTF) $(WARN_FMT) $(WARN_STRING) && $(CAT) temp.log; \
		else $(PRINTF) $(OK_FMT) $(OK_STRING); \
		fi;
		@rm -f temp.log temp.err

$(OBJS): $(OBJDIR)/%.o : $(SRCDIR)/%.cpp
	@$(PRINTF) "Compiling:%25s" "$(notdir $<)"
	@$(CC) $(CPPFLAGS) -fPIC -c $< -o $@ -MD 2> temp.log || touch temp.err
	@if test -e temp.err; \
		then $(PRINTF) $(ERR_FMT) $(ERR_STRING) && $(CAT) temp.log; \
		elif test -s temp.log; \
		then $(PRINTF) $(WARN_FMT) $(WARN_STRING) && $(CAT) temp.log; \
		else printf "%30s\n" "[OK]"; \
		fi;
		@rm -f temp.log temp.err

target_lib:
	@if test $(SHARED_LIB) = FALSE;\
	then make target_static;\
	else make target_dynamic;\
	fi;

target_static:
	@$(ECHO) "BUILDING the static library .. "
	@ar cq $(LIBSDIR)/libCS296test.a $(FILTERED)

target_dynamic:
	@$(ECHO) "creating the dynamic libraries .. "
	@$(CC) -shared -Wl,-soname,$(LIBSDIR)/libCS296test.so \
	    -o $(LIBSDIR)/libCS296test.so $(FILTERED)

exelib:$(BINDIR)/$(TARGETNEW)

$(BINDIR)/$(TARGETNEW):$(OBJS) target_lib
	@$(CC) -o $@ $(LDFLAGS) $(LDPATH) -L $(LIBSDIR) $(OBJDIR)/main.o -lCS296test $(LIBS)

clean:
	@rm $(BINDIR)/* $(OBJDIR)/* $(LIBSDIR)/*

distclean:
	@rm -rf $(EXTERNAL_ROOT)/src/Box2D $(EXTERNAL_ROOT)/lib/Box2D $(EXTERNAL_ROOT)/include/Box2D
	@rm -rf $(BINDIR) $(OBJDIR) $(LIBSDIR)
