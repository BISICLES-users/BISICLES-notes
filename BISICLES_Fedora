# Fedora 35

These instructions are primarily based on [generic build instructions](http://davis.lbl.gov/Manuals/BISICLES-DOCS/readme.html) 
but with some modifications to fit the system KNMI workstation, which run on Fedora 35. I have so far only compiled BISICLES in serial on the workstation as the model 
is mainly used for the filetools rather than simulations. 

### 1. Checking out the source code

To get Chombo you need an [ANAG repository account](https://anag-repo.lbl.gov/).
You need to choose a username and password, that will be asked when you check out the source code. 
(You might want to write it down if you may need to compile BISICLES more than once and keep forgetting like I do. 
That being said, you can just make a new account). 

* Create a new directory for BISICLES to live in, e.g.:
  
  `mkdir -p [/path/to/bisicles]`

* Export the path to this directory so you can refer to it easily:
  
  `export BISICLES_HOME=[/path/to/bisicles]`

  This will mean that you can use `$BISICLES_HOME` to refer to that path without having to type it out each time 
  (and also you can copy and paste instructions).
  
* Change into the new directory:
  
  `cd $BISICLES_HOME`
  
* Then get the source code:

  `svn co https://anag-repo.lbl.gov/svn/Chombo/release/3.2 Chombo`

  `svn co https://anag-repo.lbl.gov/svn/BISICLES/public/trunk BISICLES`

* Modifications to Chombo for Fedora35:

  The compiler may be newer than Chombo expects and therefore a new line needs to be added:

1. Find the file `Chombo/lib/src/BoxTools/LoadBalance.cpp`
2. Open it in a text editor
3. find the line near the top that reads: 
`#include <set>`
4. Add a line below that: 
`#include <limits>`

### 2. Dependencies

Chombo needs HDF5 to function, if you have sudo rights on you machine you may be able to skip this step and run dnf/yum command to install mpi, 
hdf5 or netcdf. Otherwise:

You can download the dependencies by:

`cd $BISICLES_HOME/BISICLES/docs`

`chmod +x download_dependencies.sh`

`./download_dependencies.sh`

This will download and unpack both HDF5 and NetCDF into `$BISICLES_HOME`.

When building HDF5, you can choose whether you want to have a serial or parallel (using mpi) build. Here I am only building the serial build.  

* Serial Build

`cd $BISICLES_HOME/hdf5/serial/src/hdf5-1.8.21/`

`CC=gcc CFLAGS=-fPIC ./configure --prefix=$BISICLES_HOME/hdf5/serial/ --enable-shared=no`

It'll then configure a bunch of things and come out with a summary. Then do:

`make install`

Netcdf can be found on the KNMI workstation and therefore do not need to built independently, the location will be set in the BISICLES configuration. 

### 3. Chombo Configuration
  
* Copy the `Make.defs.local`file to `$BISICLES_HOME`

  `cp $BISICLES_HOME/BISICLES/docs/Make.defs.local $BISICLES_HOME`

* Edit the file so that:

  `BISICLES_HOME=[path/to/bisicles]`

  To reflect be the path to where BISICLES lives ( what `$BISICLES_HOME` is set to), 
  if you are only building serial then comment out the sections relevant to a parallel build.
  
  Make sure that it contains the following information:

        PRECISION     = DOUBLE
        CXX           = g++
        FC            = gfortran
        #MPI           = TRUE
        #MPICXX        = mpiicc
        USE_64        = TRUE
        USE_HDF       = TRUE
        #make sure BISICLES_HOME is correct
        BISICLES_HOME=[path/to/bisicles]
        HDFINCFLAGS   = -I$(BISICLES_HOME)/hdf5/serial/include
        HDFLIBFLAGS   = -L$(BISICLES_HOME)/hdf5/serial/lib -lhdf5 -lz
        ## Note: don't set the HDFMPI* variables if you don't have parallel HDF installed
        #HDFMPIINCFLAGS= -I$(BISICLES_HOME)/hdf5/parallel/include
        #HDFMPILIBFLAGS= -L$(BISICLES_HOME)/hdf5/parallel/lib -lhdf5  -lz
        cxxdbgflags    = -g -fPIC
        cxxoptflags    = -fPIC -O2
        fdbgflags     =  -g -fPIC
        foptflags     = -fPIC -O3 -ffast-math -funroll-loops

* Make a link to the `Make.defs.local`in the place that Chombo expects it:

  `ln -s $BISICLES_HOME/Make.defs.local $BISICLES_HOME/Chombo/lib/mk/Make.defs.local`
  
### 4. BISICLES Configuration
  
* Next, create a makefile for BISICLES with the name of your machine:

  `cd $BISICLES_HOME/BISICLES/code/mk/`
  
  `uname -n`
  
  `[mymachine]`
  
  `cp Make.defs.ubuntu_21.4 Make.defs.[mymachine]`

Here we are copying the ubuntu_21.4 makefile rather than the template as that will work with Fedora35 with little or no change.
You may want to edit the PYTHON_LIBS line to:

`PYTHON_LIBS=-lpython3.10 $(shell python3-config --ldflags)`

or to reflect the version of python that you are using. 

### 5. Compiling BISICLES

* To compile a standalone BISICLES (so no filetools etc) you can run this:

  `cd $BISICLES_HOME/BISICLES/code/exec2D`
  
  `make all`

* To compile all of BISICLES:

  `cd $BISICLES_HOME/BISICLES/code`
  
  `make all`

Then you should have a version of BISICLES that can run problems and be able to use the filetools!
