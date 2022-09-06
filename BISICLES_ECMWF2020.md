# ECMWF HPC2020

To use the ECMWF servers you need a token, I am assuming here that all of that is sorted and you know how to log on. To log into the new server there are the following resources:

- [Teleport](https://confluence.ecmwf.int/display/UDOC/Teleport+SSH+Access)
- [HPC2020](https://confluence.ecmwf.int/display/UDOC/HPC2020%3A+How+to+connect)

If you type quota on the command line you will get your quota for $HOME (small but backup), $PERM (larger, not erased, no backup) and $SCRATCH (very large, but files are deleted after 2-3 weeks). 
Typically you will install models on $PERM, and run them in $SCRATCH. 

To change into the $PERM directory:

`cd $PERM`

## Environment

To compile BISICLES, you need to load the right modules in the environment of your choosing. In this case I am going to use GNU. 

```
module load prgenv/gnu
module load git
module load python3/3.8.8-01
module load gcc/11.2.0
module load openmpi/4.1.1.1
module load hdf5-parallel/1.10.6
module load netcdf4-parallel/4.7.4
```

More in-depth instructions on compiling BISICLES can be found: [generic build instructions](http://davis.lbl.gov/Manuals/BISICLES-DOCS/readme.html)
 
### Checking out the source code

To get Chombo you need an [ANAG repository account](https://anag-repo.lbl.gov/).
You need to choose a username and password, that will be asked when you check out the source code. (You might want to write it down if you may need to compile BISICLES more than once and keep forgetting like I do. That being said, you can just make a new account). 

* Export the path to this directory so you can refer to it easily:
  
  `export BISICLES_HOME=[/path/to/bisicles]`

* Create a new directory for BISICLES to live in, e.g.:
  
  `mkdir -p $BISICLES_HOME`

  This will mean that you can use `$BISICLES_HOME` to refer to that path without having to type it out each time (and also you can copy and paste instructions).
  
* Change into the new directory:
  
  `cd $BISICLES_HOME`
  
* Then get the source code:

  `svn co https://anag-repo.lbl.gov/svn/Chombo/release/3.2 Chombo`

  `svn co https://anag-repo.lbl.gov/svn/BISICLES/public/trunk BISICLES`
  
  
### Chombo and BISICLES Configuration
  
* For the Chombo configuration, copy the template into `$BISICLES_HOME`

  `cp $BISICLES_HOME/BISICLES/docs/Make.defs.local $BISICLES_HOME`
  
* Edit the `Make.defs.local` file so that it looks something like this:

```
## Configuration variables
#DIM           =
#DEBUG         =
#OPT           =
PRECISION     = DOUBLE
#PROFILE       =
CXX           = g++
FC            = gfortran
MPI           = TRUE
## Note: don't set the MPICXX variable if you don't have MPI installed
MPICXX        = mpiCC
#OBJMODEL      =
#XTRACONFIG    =
## Optional features
USE_64        = TRUE
#USE_COMPLEX   =
#USE_EB        =
#USE_CCSE      =
USE_HDF       = TRUE
#make sure BISICLES_HOME is correct
BISICLES_HOME=[PATH TO $BISICLES_HOME]
#note that HDFINCFLAGS and HDFLIBFLAGS must also currently be defined 
# for parallel builds (due to a Chombo build-system ideosyncracy which 
# will be fixed in an upcoming Chombo release). If no serial hdf5 build 
# is present, duplicate the values in HDFMPIINCFLAGS and HDFMPILIBFLAGS

HDFINCFLAGS   = -I[PATH TO HDF5]/include
HDFLIBFLAGS   = -L[PATH TO HDF5]/lib -lhdf5 -lz
## Note: don't set the HDFMPI* variables if you don't have parallel HDF installed #PATH TO HDF5
HDFMPIINCFLAGS= -I[PATH TO HDF5]/include
HDFMPILIBFLAGS= -L[PATH TO HDF5]/lib -lhdf5  -lz
#USE_MF        =
#USE_MT        =
#USE_SETVAL    =
#AR            =
#CPP           =
#DOXYGEN       =
#LD            =
#PERL          =
#RANLIB        =
#cppdbgflags   =
#cppoptflags   =
#cxxcppflags   = -fPIC
cxxdbgflags    = -g -fPIC
cxxoptflags    = -fPIC -O2
#cxxprofflags  =
#fcppflags     = -fPIC
fdbgflags     =  -g -fPIC
foptflags     = -fPIC -O3 -ffast-math -funroll-loops
#fprofflags    =
#flibflags     =
#lddbgflags    =
#ldoptflags    =
#ldprofflags   =
#syslibflags   = 

unexport BISICLES_HOME
#end  -- dont change this line
```
You can find where your hdf5 module is located by running `$PATH`. If using the same version as here it might be:

`/usr/local/apps/hdf5-parallel/1.10.6/GNU/11.2/OMPI/4.1`

* Link the modified `Make.defs.local` file to the correct place

  `ln -s $BISICLES_HOME/Make.defs.local $BISICLES_HOME/Chombo/lib/mk/Make.defs.local`

* Make sure the BISICLES make file is correct

Copy the file `$BISICLES_HOME/BISICLES/code/mk/Make.defs.archer` to a new file with your machine name (or `Make.defs.none`), where the machine name is the output from `uname -n`:

    cd $BISICLES_HOME/BISICLES/code/mk/
    uname -n
    -> [mymachine]
    cp Make.defs.archer Make.defs.[mymachine]

Then modify `Make.defs.[mymachine]` to have the correct python path. In my case this was:

```
PYTHON_DIR=/usr/local/apps/python3/3.8.8-01/
PYTHON_INC=-I$(PYTHON_DIR)/include/python3.8/
PYTHON_LIBS=-L$(PYTHON_DIR)/lib -lpython3.8 -lm -lpthread -ldl -lutil

ifneq ($(NETCDF_DIR),)
NETCDF_INC=-I$(NETCDF_DIR)/include
NETCDF_LIBS=-L$(NETCDF_DIR)/lib -lnetcdf
endif
```

You can find the path to your python module by running `$PATH`

### Compiling BISICLES

  `cd $BISICLES_HOME/BISICLES/code`
  
  `make all OPT=TRUE MPI=TRUE`
  
  or to use PETSc (but this did not work for me yet):
  
  `make all OPT=TRUE MPI=TRUE USE_PETSC=TRUE`

Then you should have a version of BISICLES that can run problems!
(Don't try to compile the model in serial, it won't work). 

### Running an Example Problem

Instructions on running jobs on cca can be found [here](https://confluence.ecmwf.int/display/UDOC/Batch+environment%3A++PBS). 
To run jobs in parallel you need to use `aprun`. For BISICLES to run, you may need to set `LD_LIBRARY_PATH` to where the python libraries are located. 

`export LD_LIBRARY_PATH=[python path]/lib`

If you run into a "file locking issue" you may want to set file locking to false. 

`export HDF5_USE_FILE_LOCKING=FALSE`

Here is an example of a simple submission script. 

    #PBS -N HelloMPI
    #PBS -q np
    #PBS -l EC_total_tasks=72
    #PBS -l EC_hyperthreads=1
    #PBS -l walltime=48:00:00
    #-------------------------------------------

    # on compute node, change directory to 'submission directory':
    cd $PBS_O_WORKDIR

    # Define input files
    export INFILEBASE="[input file]"
    export INFILE=[input file].$PBS_JOBID
    echo "INFILE = $INFILE"
    cp $INFILEBASE $INFILE

    export PYTHONPATH=`pwd`
    export LD_LIBRARY_PATH=[python path]/lib
    export HDF5_USE_FILE_LOCKING=FALSE

    aprun -N $EC_tasks_per_node -n $EC_total_tasks -j $EC_hyperthreads $PERM/bisicles/BISICLES/code/exec2D/driver2d.Linux.64.CC.ftn.OPT.MPI.GNU.ex $INFILE > sout.0 2>err.0
