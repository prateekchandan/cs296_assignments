#variables
TARGET=cs296_17_exe
TARGETNEW=cs296_17_exelib
SHARED_LIB=TRUE

#programs
ECHO=/bin/echo
CAT=cat

# Project Paths
PROJECT_ROOT=.
EXTERNAL_ROOT=$(PROJECT_ROOT)/external
SRCDIR = $(PROJECT_ROOT)/src
OBJDIR = $(PROJECT_ROOT)/myobjs
BINDIR = $(PROJECT_ROOT)/mybins
INCLUDEDIR= $(EXTERNAL_ROOT)/include
LIBDIR= $(EXTERNAL_ROOT)/lib
LIBSDIR=$(PROJECT_ROOT)/mylibs
SCRIPTDIR=$(PROJECT_ROOT)/scripts
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
CPPFLAGS =-g -Wall -fno-strict-aliasing
CPPFLAGS+=-I $(BOX2D_ROOT)/include -I $(GLUI_ROOT)/include
RELEASEFLAGS = $(CPPFLAGS) -O3
LDFLAGS+=-L $(BOX2D_ROOT)/lib -L $(GLUI_ROOT)/lib

#dynamic variable
SRCS := $(wildcard $(SRCDIR)/*.cpp)
INCS := $(wildcard $(SRCDIR)/*.hpp)
OBJS := $(SRCS:$(SRCDIR)/%.cpp=$(OBJDIR)/%.o)
FILTERED = $(filter-out $(OBJDIR)/main.o,$(OBJS))

.PHONY: all setup $(OBJS) exe data plots report

all : exe

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

exe1: CPPFLAGS+= -O3
exe1: $(BINDIR)/$(TARGET)
exe:setup $(BINDIR)/$(TARGET)

$(BINDIR)/$(TARGET):$(OBJS)
	@$(CC) -p -o $(BINDIR)/$(TARGET) $(LDFLAGS) $(OBJS) $(LIBS) 2> temp.log || touch temp.err
	@if test -e temp.err; \
		then $(PRINTF) $(ERR_FMT) $(ERR_STRING) && $(CAT) temp.log; \
		elif test -s temp.log; \
		then $(PRINTF) $(WARN_FMT) $(WARN_STRING) && $(CAT) temp.log; \
		else $(PRINTF) $(OK_FMT) $(OK_STRING); \
		fi;
		@rm -f temp.log temp.err

$(OBJS): $(OBJDIR)/%.o : $(SRCDIR)/%.cpp
	@$(PRINTF) "Compiling:%25s" "$(notdir $<)"
	@$(CC) $(CPPFLAGS) -c $< -o $@ -MD 2> temp.log || touch temp.err
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
cleandat:
	@rm -rf *.dat
clean:
	@rm -f $(BINDIR)/* $(OBJDIR)/* $(LIBSDIR)/*
	@rm -rf ./plots 
	@rm -rf ./doc/*.pdf ./doc/*.log ./doc/*.aux ./doc/*.html
	@rm -rf  *.out
	@rm -rf ./data/g* ./data/pnp_austen_cs296.txt
	@rm -rf *.log  0* 1* 2* 3* 4* 5* 6* 7* 8* 9*

distclean1: clean 
	@rm -rf $(EXTERNAL_ROOT)/src/Box2D $(EXTERNAL_ROOT)/lib $(EXTERNAL_ROOT)/lib/Box2D $(EXTERNAL_ROOT)/include/Box2D
	@rm -rf $(BINDIR) $(OBJDIR) $(LIBSDIR)
	@rm -rf $(PROJECT_ROOT)/plots 
	
distclean:cleandat distclean1
plot:
	@mkdir -p plots
	@gnuplot $(SCRIPTDIR)/g17_plot01.gpt
	@gnuplot $(SCRIPTDIR)/g17_plot02.gpt
	@$(SCRIPTDIR)/helpscript3.sh
	@gnuplot $(SCRIPTDIR)/g17_plot03.gpt
	@rm -rf ./data/temp.csv 
	@gnuplot $(SCRIPTDIR)/g17_plot04.gpt
	@gnuplot $(SCRIPTDIR)/g17_plot05.gpt
	@rm -rf 0* 1* 2* 3* 4* 5* 6* 7* 8* 9* *.log

data:
	@mkdir -p data
	@chmod 777 $(SCRIPTDIR)/*
	@$(SCRIPTDIR)/g17_gen_data.sh
	@$(SCRIPTDIR)/g17_gen_csv.sh
	@.$(SCRIPTDIR)/g17_gen_data_csv.sh
	@$(SCRIPTDIR)/g17_gen_random_csv.sh

debug_prof:distclean1
	@mkdir -p myobjs mybins mylibs
	@if test -e $(INCLUDEDIR)/Box2D -a -e $(LIBDIR)/Box2D; \
		then $(ECHO) "Already present file .. ";\
		else tar -xzf $(EXTERNAL_ROOT)/src/Box2D.tgz -C $(EXTERNAL_ROOT)/src\
		&& cd $(EXTERNAL_ROOT)/src/Box2D \
		&& mkdir -p build296 \
		&& cd build296\
		&& cmake -DCMAKE_BUILD_TYPE=Debug ../ \
		&& make \
		&& make install \
		&& cd ../../../../; \
	fi;
	@make exe
	@perf record -g -- ./mybins/cs296_17_exe
	@perf report > g17_debug_prof.dat
	@perf script | python gprof2dot.py -f perf| dot -Tpng -o ./doc/debug.png
	@rm perf.data
	

release_prof:distclean1
	@mkdir -p myobjs mybins mylibs
	@if test -e $(INCLUDEDIR)/Box2D -a -e $(LIBDIR)/Box2D; \
		then $(ECHO) "Already present file .. ";\
		else tar -xzf $(EXTERNAL_ROOT)/src/Box2D.tgz -C $(EXTERNAL_ROOT)/src\
		&& cd $(EXTERNAL_ROOT)/src/Box2D \
		&& mkdir -p build296 \
		&& cd build296\
		&& cmake -DCMAKE_BUILD_TYPE=Release ../ \
		&& make \
		&& make install \
		&& cd ../../../../; \
	fi;
	@make exe1
	@perf record -g -- ./mybins/cs296_17_exe
	@perf report > g17_release_prof.dat
	@perf script | python gprof2dot.py -f perf | dot -Tpng -o ./doc/release.png
	@rm perf.data
	
report:
	@if test -e $(BINDIR)/$(TARGET);\
		then echo "source present" ;\
		else make exe;\
	fi;
	@if test -e ./data/g17_lab09data_01.csv;\
		then echo "data already present .. run make clean to clear data";\
		else python3 ./scripts/g17_gen_csv.py ; \
	fi;
	@mkdir -p plots 
	@ipython ./scripts/g17_gen_plots.py
	@python3 ./scripts/g17_gen_html.py
	@xdg-open ./doc/g17_lab09_report.html
	
