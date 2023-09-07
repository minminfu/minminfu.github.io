---
permalink: /misc/
title: "Code and Miscellaneous"
author_profile: false
redirect_from: 
  - /code/
  - /misc.html
---
## How to Build Dedalus on NCAR Cheyenne

[Dedalus](https://dedalus-project.org/) is a great spectral-based PDE solver that is fast and easy to use. I didn't see any instructions available online for how to build it on NCAR's Cheyenne computing cluster, so I'll share how I did it here. The instructions here worked as of August 2, 2021. 

Go to your work directory and clone dedalus.

    cd /glade/work/USERNAME/
    git clone https://github.com/DedalusProject/dedalus

Now load the following modules:

    module load intel/19.0.5
    module load openmpi/4.0.5
    module load python/3.7.9
    module load hdf5/1.10.7
    module load ncarcompilers/0.5.0
    module load ncarenv/1.3

Make sure ncarcompilers and ncarenv are loaded. NCAR does not use conda so we should clone ncar_pylib to our own work directory such that we can use pip. 

    ncar_pylib -c 20201220 /glade/work/USERNAME/dedalus_2021

Now load our clone of NCAR's python environment.

    source /glade/work/USERNAME/dedalus_2021/bin/activate

Make sure all python packages are installed.

    cd /glade/work/USERNAME/dedalus
    pip3 install --no-cache-dir -r requirements.txt

In my experience, some potential errors can be resolved by uninstalling and reinstalling the requirements using pip (such as mpi4py). Sometimes it can take a bit of trial and error. My package versions are:

    Cython                        0.29.24
    docopt                        0.6.2
    h5py                          3.3.0
    matplotlib                    3.4.2
    mpi4py                        3.0.3
    numpy                         1.21.1
    scipy                         1.7.0
    setuptools                    57.4.0

Unfortunately the fftw3 installed by CISL does not have mpi enabled. I recommend building from the source. 

    # FFTW built from source
    export FFTW_PATH=/glade/work/USERNAME/fftw3
    export FFTW_VERSION=3.3.6-pl2
    mkdir -p ${FFTW_PATH}
    cd ${FFTW_PATH}	
    wget http://www.fftw.org/fftw-${FFTW_VERSION}.tar.gz
    tar -xvzf fftw-${FFTW_VERSION}.tar.gz
    cd fftw-${FFTW_VERSION}
    ./configure --prefix=${FFTW_PATH} \
        CC=mpicc \
        CXX=mpicxx \
        F77=mpif90 \
        MPICC=mpicc \
        MPICXX=mpicxx \
        --enable-shared \
        --enable-mpi \
        --enable-openmp \
        --enable-threads
    make
    make install

Make sure these two lines are in your .bashrc

    export PYTHONPATH=/glade/work/USERNAME/dedalus:$PYTHONPATH 
    export JUPYTER_PATH=/glade/work/USERNAME/dedalus:$JUPYTER_PATH
    # FFTW built from source
	export FFTW_PATH=/glade/work/USERNAME/fftw3
    export FFTW_VERSION=3.3.6-pl2
    export PATH=${FFTW_PATH}/bin:${PATH}
    export LD_LIBRARY_PATH=${FFTW_PATH}/lib:${LD_LIBRARY_PATH}
    export LD_LIBRARY_PATH=/glade/u/apps/opt/intel/mkl/lib/intel64/:${LD_LIBRARY_PATH}
    # Dedalus internal
    export MPI_LIBRARY_PATH=/glade/u/apps/ch/opt/openmpi/4.0.5/intel/19.0.5/lib
    export MPI_INCLUDE_PATH=/glade/u/apps/ch/opt/openmpi/4.0.5/intel/19.0.5/include
    export FFTW_LIBRARY_PATH=/glade/work/USERNAME/fftw3test/lib
    export FFTW_INCLUDE_PATH=/glade/work/USERNAME/fftw3test/include

Run the setup script

    cd /glade/work/USERNAME/dedalus
    python3 setup.py build_ext --inplace

You should be done. I add the following to my .bashrc for easily loading what I need to run dedalus. 

	loaddedalus2021() {
        module purge
        module load intel
        module load openmpi
        module load python
        module load hdf5
        module load ncarcompilers
        module load ncarenv
        
        # Load my packages
        source /glade/work/USERNAME/dedalus_2021/bin/activate
        # FFTW built from source
        export FFTW_PATH=/glade/work/USERNAME/fftw3
        export FFTW_VERSION=3.3.6-pl2
        export PATH=${FFTW_PATH}/bin:${PATH}
        export LD_LIBRARY_PATH=${FFTW_PATH}/lib:${LD_LIBRARY_PATH}
        export LD_LIBRARY_PATH=/glade/u/apps/opt/intel/mkl/lib/intel64/:${LD_LIBRARY_PATH}
        # MPI FFTW paths (I think only needed for initial build)
        export MPI_LIBRARY_PATH=/glade/u/apps/ch/opt/openmpi/4.0.5/intel/19.0.5/lib
        export MPI_INCLUDE_PATH=/glade/u/apps/ch/opt/openmpi/4.0.5/intel/19.0.5/include
        export FFTW_LIBRARY_PATH=/glade/work/USERNAME/fftw3test/lib
        export FFTW_INCLUDE_PATH=/glade/work/USERNAME/fftw3test/include
        # Dedalus internal path
        export PYTHONPATH=/glade/work/USERNAME/dedalus:$PYTHONPATH 
        export JUPYTER_PATH=/glade/work/USERNAME/dedalus:$JUPYTER_PATH
	}

Hope this works for you. Good luck!

## More about me
I grew up in Fremont, California in the San Francisco Bay Area. Outside of research, I love classical music and have an interest in violin and guitar.  

