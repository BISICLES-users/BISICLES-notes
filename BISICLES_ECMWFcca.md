# ECMWF

To use the ECMWF servers you need a token, I am assuming here that all of that is sorted and you know how to log on.

When you log on to ECMWF there are 3 choices: ecgate, cca and ccb. Most people seem to use cca. 
Once you choose which server you want to use, you will automatically be in your home directory. If you type quota on the command line you will get your quota for $HOME (small but backup), $PERM (larger, not erased, no backup) and $SCRATCH (very large, but files are deleted after 2-3 weeks). 
Typically you will install models on $PERM, and run them in $SCRATCH. 

To change into the $PERM directory:

`cd $PERM`

## Environment

To compile BISICLES, switch to the GNU environment, you could also use intel if you wish. 

`prgenvswitchto gnu`

Then load the relevant modules to compile BISICLES. 

`module load subversion git openssl cray-hdf5-parallel cray-netcdf-hdf5parallel cray-petsc python`

Furthermore, it is necessary to edit the proxy settings in your $HOME/.subversion/servers file. First check if the following file exists:

`vim $HOME/.subversion/servers`

If it doesn't exist, you can create one by running `svn help`. Then add the following lines to the `[global]` section:

    [global]
    http-proxy-host = proxy.ecmwf.int
    http-proxy-port = 3333

You may also need to export the following paths:

    export https_proxy=proxy:2222
    export http_proxy=proxy:2222
    export ftp_proxy=proxy:2222

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
  
I am using the main development branch here instead of the current release, as the current release doesn't support Python3. However, unlike for the Debian/Ubuntu build, it doesn't matter which version you use. 
  
### Chombo and BISICLES Configuration
  
For the Chombo configuration, you can use the `Make.defs.archer` example as that uses a fairly similar set-up. 

* Link the `Make.defs.archer`file to `Make.defs.local`

  `ln -s $BISICLES_HOME/Chombo/lib/mk/local/Make.defs.archer $BISICLES_HOME/Chombo/lib/mk/Make.defs.local`

* Make sure the BISICLES make file is correct

`$BISICLES_HOME/BISICLES/code/mk/Make.defs.archer` includes the following details for the python interface and netcdf. 
  
    PYTHON_DIR=/work/y07/y07/cse/python/2.7.6-static/
    PYTHON_INC=-I$(PYTHON_DIR)/include/python2.7/
    PYTHON_LIBS=-L$(PYTHON_DIR)/lib -lpython2.7 -lpthread -ldl -lutil
    
    NETCDF_INC=-I$(NETCDF_DIR)/include
    NETCDF_LIBS=-L$(NETCDF_DIR)/lib -lnetcdf

Copy the file to a new file with your machine name (or `Make.defs.none`), where the machine name is the output from `uname -n`:

    cd $BISICLES_HOME/BISICLES/code/mk/
    uname -n
    -> [mymachine]
    cp Make.defs.archer Make.defs.[mymachine]

Then modify `Make.defs.[mymachine]` to have the correct python path. In my case this was:

    PYTHON_DIR=/usr/local/apps/python/2.7.17-01/
    PYTHON_INC=-I$(PYTHON_DIR)/include/python2.7/
    PYTHON_LIBS=-L$(PYTHON_DIR)/lib -lpython2.7 -lm -lpthread -ldl -lutil

    ifneq ($(NETCDF_DIR),)
    NETCDF_INC=-I$(NETCDF_DIR)/include
    NETCDF_LIBS=-L$(NETCDF_DIR)/lib -lnetcdf
    endif

You can find the path to your python module by running `$PATH`

### Compiling BISICLES

  `cd $BISICLES_HOME/BISICLES/code`
  
  `make all OPT=TRUE MPI=TRUE DEBUG=FALSE`
  
  or to use PETSc (but this did not work for me yet):
  
  `make all OPT=TRUE MPI=TRUE DEBUG=FLASE USE_PETSC=TRUE`

Then you should have a version of BISICLES that can run problems!

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
