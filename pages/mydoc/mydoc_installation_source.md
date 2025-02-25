---
title: Compile from source (Ubuntu)
keywords: dafoam, installation, compile
summary: 
sidebar: mydoc_sidebar
permalink: mydoc_installation_source.html
folder: mydoc
---

{% include note.html content="This section assumes you want to compile the latest DAFoam optimization package from the source on a Linux system. If you use the Docker image, there is no need to compile anything and you can skip this section. For DAFoam older versions, refer to [v2.2.10-](mydoc_installation_source_2210.html), [v2.2.0-](mydoc_installation_source_220.html), and [v1.0.0](mydoc_installation_source_100.html)." %}

The DAFoam package can be compiled with various dependency versions. Here we elaborate on how to compile it on **Ubuntu 20.04** using the dependencies shown in the following table.


Ubuntu | Compiler | OpenMPI | mpi4py | PETSc  | petsc4py | CGNS  | Python | Numpy  | Scipy | Cython
| :------------------------------------------------------------------------------------------------ | 
20.04  | gcc/9.3  | 3.1.6   | 3.1.3  | 3.14.6 | 3.14.1   | 4.1.2 | 3.8    | 1.21.2 | 1.7.1 | 0.29.21

To compile, you can just copy the code blocks in the following steps and run them on the terminal. If a code block contains multiple lines, copy all the lines and run them on the terminal. Make sure each step run successfully before going to the next one. The entire compilation may take a few hours, the most time-consuming part is to compile OpenFOAM. **Make sure you DO NOT close the terminal before all the installation steps are done!**

## **Prerequisites**

Run this on terminal to install prerequisites:

<pre>
sudo apt-get update && \
sudo apt-get install -y build-essential flex bison cmake zlib1g-dev libboost-system-dev libboost-thread-dev libreadline-dev libncurses-dev libxt-dev freeglut3-dev texinfo libscotch-dev libcgal-dev gfortran swig wget git vim cmake-curses-gui libfl-dev apt-utils libibverbs-dev ca-certificates pkg-config liblapack-dev libmetis-dev --no-install-recommends
</pre>

## **Root folder**

First create a "dafoam" folder in your home directory. Then create a "loadDAFoam.sh" bash script and set up the root path $DAFOAM_ROOT_PATH. Finally, we will create the "packages", "OpenFOAM", and "repos" folders. We will compile and install everything in $DAFOAM_ROOT_PATH.

<pre>
mkdir -p $HOME/dafoam && \
mkdir -p $HOME/dafoam/packages $HOME/dafoam/OpenFOAM $HOME/dafoam/repos && \
echo '#!/bin/bash' > $HOME/dafoam/loadDAFoam.sh && \
echo '# DAFoam root path' >> $HOME/dafoam/loadDAFoam.sh && \
echo 'export DAFOAM_ROOT_PATH=$HOME/dafoam' >> $HOME/dafoam/loadDAFoam.sh && \
chmod 755 $HOME/dafoam/loadDAFoam.sh && \
. $HOME/dafoam/loadDAFoam.sh
</pre>

## **Python**

Install Miniconda3 by running this command:

<pre>
cd $HOME/dafoam/packages && \
wget https://repo.anaconda.com/miniconda/Miniconda3-py38_4.10.3-Linux-x86_64.sh && \
chmod 755 Miniconda3-py38_4.10.3-Linux-x86_64.sh && \
./Miniconda3-py38_4.10.3-Linux-x86_64.sh -b -p $HOME/dafoam/packages/miniconda3 && \
echo '# Miniconda3' >> $HOME/dafoam/loadDAFoam.sh && \
echo 'export PATH=$DAFOAM_ROOT_PATH/packages/miniconda3/bin:$PATH' >> $HOME/dafoam/loadDAFoam.sh && \
echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$DAFOAM_ROOT_PATH/packages/miniconda3/lib' >> $HOME/dafoam/loadDAFoam.sh && \
echo 'export PYTHONUSERBASE=no-local-libs' >> $HOME/dafoam/loadDAFoam.sh && \
. $HOME/dafoam/loadDAFoam.sh
</pre>

In the above, we use "export PYTHONUSERBASE=no-local-libs" to bypass the site-packages in user's .local directory because they may conflict with the DAFoam packages. Then we can upgrade the pip utility:

<pre>
pip install --upgrade pip && \
pip install numpy==1.21.2 && \
pip install scipy==1.7.1 && \
pip install cython==0.29.21 && \
pip install numpy-stl==2.16.0 && \
pip install pynastran==1.3.3
</pre>


## **OpenMPI**

First append relevant environmental variables by running:

<pre>
echo '# OpenMPI-3.1.6' >> $HOME/dafoam/loadDAFoam.sh && \
echo 'export MPI_INSTALL_DIR=$DAFOAM_ROOT_PATH/packages/openmpi-3.1.6/opt-gfortran' >> $HOME/dafoam/loadDAFoam.sh && \
echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$MPI_INSTALL_DIR/lib' >> $HOME/dafoam/loadDAFoam.sh && \
echo 'export PATH=$MPI_INSTALL_DIR/bin:$PATH' >> $HOME/dafoam/loadDAFoam.sh&& \
. $HOME/dafoam/loadDAFoam.sh
</pre>

Then, configure and build OpenMPI:

<pre>
cd $HOME/dafoam/packages && \
wget https://download.open-mpi.org/release/open-mpi/v3.1/openmpi-3.1.6.tar.gz  && \
tar -xvf openmpi-3.1.6.tar.gz && \
cd openmpi-3.1.6 && \
./configure --prefix=$MPI_INSTALL_DIR && \
make all install
</pre>

To verify the installation, run:

<pre>
mpicc -v
</pre>

You should see the version of the compiled OpenMPI.

Finally, install mpi4py-3.1.3:

<pre>
pip install mpi4py==3.1.3
</pre>

## **Petsc**

First append relevant environmental variables by running:

<pre>
echo '# Petsc-3.14.6' >> $HOME/dafoam/loadDAFoam.sh && \
echo 'export PETSC_DIR=$DAFOAM_ROOT_PATH/packages/petsc-3.14.6' >> $HOME/dafoam/loadDAFoam.sh && \
echo 'export PETSC_ARCH=real-opt' >> $HOME/dafoam/loadDAFoam.sh && \
echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$PETSC_DIR/$PETSC_ARCH/lib' >> $HOME/dafoam/loadDAFoam.sh && \
echo 'export PETSC_LIB=$PETSC_DIR/$PETSC_ARCH/lib' >> $HOME/dafoam/loadDAFoam.sh && \
. $HOME/dafoam/loadDAFoam.sh
</pre>

Then, configure and compile:

<pre>
cd $HOME/dafoam/packages && \
wget https://www.mcs.anl.gov/petsc/mirror/release-snapshots/petsc-3.14.6.tar.gz  && \
tar -xvf petsc-3.14.6.tar.gz && \
cd petsc-3.14.6 && \
./configure --PETSC_ARCH=real-opt --with-scalar-type=real --with-debugging=0 --download-metis=yes --download-parmetis=yes --download-superlu_dist=yes --download-fblaslapack=yes --with-shared-libraries=yes --with-fortran-bindings=1 --with-cxx-dialect=C++11 && \
make PETSC_DIR=$HOME/dafoam/packages/petsc-3.14.6 PETSC_ARCH=real-opt all
</pre>

Finally, install petsc4py-3.14.1:

<pre>
cd $PETSC_DIR/src/binding/petsc4py && \
pip install .
</pre>

## **CGNS**

First append relevant environmental variables by running:

<pre>
echo '# CGNS-4.1.2' >> $HOME/dafoam/loadDAFoam.sh && \
echo 'export CGNS_HOME=$DAFOAM_ROOT_PATH/packages/CGNS-4.1.2/opt-gfortran' >> $HOME/dafoam/loadDAFoam.sh && \
echo 'export PATH=$PATH:$CGNS_HOME/bin' >> $HOME/dafoam/loadDAFoam.sh && \
echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$CGNS_HOME/lib' >> $HOME/dafoam/loadDAFoam.sh && \
. $HOME/dafoam/loadDAFoam.sh
</pre>

Then, configure and compile:

<pre>
cd $HOME/dafoam/packages && \
wget https://github.com/CGNS/CGNS/archive/v4.1.2.tar.gz  && \
tar -xvaf v4.1.2.tar.gz && \
cd CGNS-4.1.2 && \
mkdir -p build && \
cd build && \
cmake .. -DCGNS_ENABLE_FORTRAN=1 -DCMAKE_INSTALL_PREFIX=$CGNS_HOME -DCGNS_BUILD_CGNSTOOLS=0 && \
make all install
</pre>

## **IPOPT**

Download Ipopt-3.13.2 and set up the relevant environmental variables to loadDAFoam.sh by running:

<pre>
echo '# Ipopt' >> $HOME/dafoam/loadDAFoam.sh && \
echo 'export IPOPT_DIR=$DAFOAM_ROOT_PATH/packages/Ipopt' >> $HOME/dafoam/loadDAFoam.sh && \
echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$IPOPT_DIR/lib' >> $HOME/dafoam/loadDAFoam.sh && \
. $HOME/dafoam/loadDAFoam.sh
</pre>

Next, compiles the ThirdParty dependencies Metis and Mumps by running:

<pre>
cd $HOME/dafoam/packages && \
git clone -b stable/3.13 https://github.com/coin-or/Ipopt.git && \
cd $IPOPT_DIR && \
git clone -b stable/2.1 https://github.com/coin-or-tools/ThirdParty-Mumps.git && \
cd ThirdParty-Mumps && \
./get.Mumps && \
./configure --prefix=$IPOPT_DIR && \
make && \
make install
</pre>

Finally, compile Ipopt by running:

<pre>
cd $IPOPT_DIR && \
mkdir build && \
cd build && \
../configure --prefix=${IPOPT_DIR} --disable-java --with-mumps --with-mumps-lflags="-L${IPOPT_DIR}/lib -lcoinmumps" --with-mumps-cflags="-I${IPOPT_DIR}/include/coin-or/mumps" && \
make && \
make install
</pre>

## **MACH-Aero framework**

The supported repo versions in the MACH-Aero framework for DAFoam-{{ site.latest_version }} is as follows

baseclasses | pySpline |  pyGeo  | multipoint | pyHyp  | cgnsUtilities | IDWarp  | pyOptSparse | pyOFM  | DAFoam
| :----------------------------------------------------------------------------------------------------------- | 
v1.6.1      | v1.5.0   | 65bd5cf | v1.4.0     | v2.5.0 | v2.6.0        | v2.6.0  | v2.8.1      | v1.2.1 | {{ site.latest_version }}

Now run this command to install all the repos for MACH-Aero:

<pre>
cd $HOME/dafoam/repos && \
wget https://github.com/mdolab/baseclasses/archive/v1.6.1.tar.gz -O baseclasses.tar.gz && \
tar -xvf baseclasses.tar.gz && cd baseclasses-1.6.1 && pip install . && \
cd $HOME/dafoam/repos && \
wget https://github.com/mdolab/pyspline/archive/v1.5.0.tar.gz -O pyspline.tar.gz && \
tar -xvf pyspline.tar.gz && cd pyspline-1.5.0 && \
cp config/defaults/config.LINUX_GFORTRAN.mk config/config.mk && \
make && pip install . && \
cd $HOME/dafoam/repos && \
wget https://github.com/mdolab/pygeo/archive/65bd5cfaeaedf3764fe53c2cd56507e047e601f5.tar.gz -O pygeo.tar.gz && \
tar -xvf pygeo.tar.gz && mv pygeo-* pygeo && cd pygeo && pip install . && \
cd $HOME/dafoam/repos && \
wget https://github.com/mdolab/multipoint/archive/v1.4.0.tar.gz -O multipoint.tar.gz && \
tar -xvf multipoint.tar.gz && cd multipoint-1.4.0 && pip install . && \
cd $HOME/dafoam/repos && \
wget https://github.com/mdolab/pyhyp/archive/v2.5.0.tar.gz -O pyhyp.tar.gz && \
tar -xvf pyhyp.tar.gz && cd pyhyp-2.5.0 && \
cp -r config/defaults/config.LINUX_GFORTRAN_OPENMPI.mk config/config.mk && \
make && pip install . && \
cd $HOME/dafoam/repos && \
wget https://github.com/mdolab/cgnsutilities/archive/v2.6.0.tar.gz -O cgnsutilities.tar.gz && \
tar -xvf cgnsutilities.tar.gz && cd cgnsutilities-2.6.0 && \
cp config/defaults/config.LINUX_GFORTRAN.mk config/config.mk && \
make && pip install . && \
cd $HOME/dafoam/repos && \
wget https://github.com/mdolab/idwarp/archive/v2.6.0.tar.gz -O idwarp.tar.gz && \
tar -xvf idwarp.tar.gz && cd idwarp-2.6.0 && \
cp -r config/defaults/config.LINUX_GFORTRAN_OPENMPI.mk config/config.mk && \
make && pip install . && \
cd $HOME/dafoam/repos && \
wget https://github.com/mdolab/pyoptsparse/archive/v2.8.1.tar.gz -O pyoptsparse.tar.gz && \
tar -xvf pyoptsparse.tar.gz && cd pyoptsparse-2.8.1 && \
pip install .
</pre>


## **OpenFOAM**

**There are three versions of OpenFOAM to compile: original, reverse-mode AD (ADR), and forward-mode AD (ADF).** The reverse-mode AD enables the JacobianFree adjoint option, and the forward-mode AD enables the brute-force AD for verifying the adjoint accuracy.

**Build Original**

Run the following:

<pre>
cd $HOME/dafoam/OpenFOAM && \
wget https://sourceforge.net/projects/openfoam/files/v1812/OpenFOAM-v1812.tgz/download -O OpenFOAM-v1812.tgz && \
wget https://sourceforge.net/projects/openfoam/files/v1812/ThirdParty-v1812.tgz/download -O ThirdParty-v1812.tgz && \
tar -xvf OpenFOAM-v1812.tgz && \
tar -xvf ThirdParty-v1812.tgz && \
cd $HOME/dafoam/OpenFOAM/OpenFOAM-v1812 && \
sed -i 's/$HOME/$DAFOAM_ROOT_PATH/g' etc/bashrc && \
wget https://github.com/DAFoam/files/releases/download/v1.0.0/UPstream.C && \
mv UPstream.C src/Pstream/mpi/UPstream.C && \
echo '# OpenFOAM-v1812' >> $HOME/dafoam/loadDAFoam.sh && \
echo 'source $DAFOAM_ROOT_PATH/OpenFOAM/OpenFOAM-v1812/etc/bashrc' >> $HOME/dafoam/loadDAFoam.sh && \
echo 'export LD_LIBRARY_PATH=$DAFOAM_ROOT_PATH/OpenFOAM/sharedLibs:$LD_LIBRARY_PATH' >> $HOME/dafoam/loadDAFoam.sh && \
echo 'export PATH=$DAFOAM_ROOT_PATH/OpenFOAM/sharedBins:$PATH' >> $HOME/dafoam/loadDAFoam.sh && \
. $HOME/dafoam/loadDAFoam.sh && \
export WM_NCOMPPROCS=4 && \
./Allwmake
</pre>

{% include note.html content="In the above command, we replaced the OpenFOAM-v1812's built-in UPstream.C file with a customized one because we need to prevent OpenFOAM from calling the MPI_Finialize function when wrapping OpenFOAM functions using Cython." %}

{% include note.html content="The above command will compile OpenFOAM using 4 CPU cores. If you want to compile OpenFOAM using more cores, change the ``WM_NCOMPPROCS`` parameter before running ``./Allwmake``" %}

Finally, verify the installation by running:

<pre>
simpleFoam -help
</pre>

It should see some basic information of OpenFOAM


**Build Reverse Mode AD**

Run the following:

<pre>
cd $HOME/dafoam/OpenFOAM && \
wget https://github.com/DAFoam/OpenFOAM-v1812-AD/archive/v1.2.9.tar.gz -O OpenFOAM-v1812-AD.tgz && \
tar -xvf OpenFOAM-v1812-AD.tgz && mv OpenFOAM-v1812-AD-* OpenFOAM-v1812-ADR && \
cd $HOME/dafoam/OpenFOAM/OpenFOAM-v1812-ADR && \
sed -i 's/WM_PROJECT_VERSION=v1812-AD/WM_PROJECT_VERSION=v1812-ADR/g' etc/bashrc && \
sed -i 's/$HOME/$DAFOAM_ROOT_PATH/g' etc/bashrc && \
sed -i 's/export WM_CODI_AD_MODE=CODI_AD_FORWARD/export WM_CODI_AD_MODE=CODI_AD_REVERSE/g' etc/bashrc && \
source etc/bashrc && \
export WM_NCOMPPROCS=4 && \
./Allwmake 2> warningLog.txt
</pre>

Then, verify the installation by running:

<pre>
DASimpleFoamReverseAD -help
</pre>

It should see some basic information of DASimpleFoamReverseAD.

{% include note.html content="We use CodiPack to differentiate the OpenFOAM libraries. During the compliation, it will generate a lot of warning messages, which are saved to the warningLog.txt file. After the compilation is done, remember to delete this warning file, which can be larger than 1 GB." %}

After OpenFOAM-v1812-ADR is compiled and verified, we need to link all the compiled AD libraries to the original OpenFOAM-v1812 folder. Note that we need to link the relative path because we want this to be portable.

<pre>
cd $HOME/dafoam/OpenFOAM/OpenFOAM-v1812/platforms/*/lib
ln -s ../../../../OpenFOAM-v1812-ADR/platforms/*/lib/*.so .
cd $HOME/dafoam/OpenFOAM/OpenFOAM-v1812/platforms/*/lib/dummy
ln -s ../../../../../OpenFOAM-v1812-ADR/platforms/*/lib/dummy/*.so .
cd $HOME/dafoam/OpenFOAM/OpenFOAM-v1812/platforms/*/lib/openmpi-system
ln -s ../../../../../OpenFOAM-v1812-ADR/platforms/*/lib/openmpi-system/*.so .
</pre>

**Build Forward Mode AD**

Run the following:

<pre>
cd $HOME/dafoam/OpenFOAM && \
wget https://github.com/DAFoam/OpenFOAM-v1812-AD/archive/v1.2.9.tar.gz -O OpenFOAM-v1812-AD.tgz && \
tar -xvf OpenFOAM-v1812-AD.tgz && mv OpenFOAM-v1812-AD-* OpenFOAM-v1812-ADF && \
cd $HOME/dafoam/OpenFOAM/OpenFOAM-v1812-ADF && \
sed -i 's/WM_PROJECT_VERSION=v1812-AD/WM_PROJECT_VERSION=v1812-ADF/g' etc/bashrc && \
sed -i 's/$HOME/$DAFOAM_ROOT_PATH/g' etc/bashrc && \
source etc/bashrc && \
export WM_NCOMPPROCS=4 && \
./Allwmake 2> warningLog.txt
</pre>

After OpenFOAM-v1812-ADF is compiled and verified, we need to link all the compiled AD libraries to the original OpenFOAM-v1812 folder. Note that we need to link the relative path because we want this to be portable.

<pre>
cd $HOME/dafoam/OpenFOAM/OpenFOAM-v1812/platforms/*/lib
ln -s ../../../../OpenFOAM-v1812-ADF/platforms/*/lib/*.so .
cd $HOME/dafoam/OpenFOAM/OpenFOAM-v1812/platforms/*/lib/dummy
ln -s ../../../../../OpenFOAM-v1812-ADF/platforms/*/lib/dummy/*.so .
cd $HOME/dafoam/OpenFOAM/OpenFOAM-v1812/platforms/*/lib/openmpi-system
ln -s ../../../../../OpenFOAM-v1812-ADF/platforms/*/lib/openmpi-system/*.so .
</pre>

## **pyOFM**

Compile pyOFM by running:

<pre>
cd $HOME/dafoam/repos && \
wget https://github.com/mdolab/pyofm/archive/v1.2.1.tar.gz -O pyofm.tar.gz && \
tar -xvf pyofm.tar.gz && cd pyofm-1.2.1 && \
. $HOME/dafoam/loadDAFoam.sh && \
make && pip install .
</pre>

## **DAFoam**

Similar to OpenFOAM, we need to compile three versions of DAFoam: original, reverse-mode AD (ADR), and forward-mode AD (ADF). It can be done by running the following commands:

<pre>
cd $HOME/dafoam/repos && \
wget https://github.com/mdolab/dafoam/archive/{{ site.latest_version }}.tar.gz -O dafoam.tar.gz && \
tar -xvf dafoam.tar.gz && cd dafoam-* && \
. $HOME/dafoam/loadDAFoam.sh && \
./Allmake 2> errorLog.txt && \
source $HOME/dafoam/OpenFOAM/OpenFOAM-v1812-ADR/etc/bashrc && \
./Allclean && ./Allmake 2> errorLog.txt && \
source $HOME/dafoam/OpenFOAM/OpenFOAM-v1812-ADF/etc/bashrc && \
./Allclean && ./Allmake 2> errorLog.txt && \
pip install .
</pre>

Once done, unset the AD environment, and re-source the original OpenFOAM-v1812. **This step is needed everytime you compile an AD versions of DAFoam!**

<pre>
unset WM_CODI_AD_MODE && \
. $HOME/dafoam/loadDAFoam.sh
</pre>


{% include note.html content="Before running any jobs, source the loadDAFoam.sh file to load DAFoam environment!" %}

## **MDO packages**

To perform multidisplinary deisgn optimization, we need to install the following packages:

[OpenMDAO](https://openmdao.org) is an open-source multidisciplinary optimization framework. 

<pre>
pip install openmdao==3.19
</pre>

[Mphys](https://github.com/OpenMDAO/mphys) is an interface that faciliate the interation between low- and high-fidelity tools within OpenMDAO.

<pre>
cd $HOME/dafoam/repos && \
wget https://github.com/OpenMDAO/mphys/archive/337347ec5325e66234e881fc99fd9c4857c16e4e.tar.gz -O mphys.tar.gz && \
tar -xvf mphys.tar.gz && mv mphys-* mphys && \
cd mphys && pip install -e .
</pre>

[FUNtoFEM](https://github.com/smdogroup/funtofem) is a generic aeroelastic analysis and adjoint-based gradient evaluation tools.

<pre>
cd $HOME/dafoam/repos && \
wget https://github.com/smdogroup/funtofem/archive/f2b39efd280ac52000ba821307cd8e3f10b3dae2.tar.gz -O funtofem.tar.gz && \
tar -xvf funtofem.tar.gz && mv funtofem-* funtofem && \
cd funtofem && cp Makefile.in.info Makefile.in && \
sed -i "s/git/dafoam\/repos/g" Makefile.in && \
make && pip install -e .
</pre>

[TACS](https://github.com/smdogroup/tacs) is a finite-element library for analysis and adjoint-based gradient evaluation

<pre>
cd $HOME/dafoam/repos && \
wget https://github.com/smdogroup/tacs/archive/refs/tags/v3.0.0.tar.gz -O tacs.tar.gz && \
tar -xvf tacs.tar.gz && mv tacs-* tacs && \
cd tacs/extern && \
wget https://github.com/DAFoam/files/releases/download/TACS_Extern/TACS_extern.tar.gz && tar -xzf TACS_extern.tar.gz && \
rm -rf metis-4.0.3* && \
wget https://github.com/DAFoam/files/releases/download/TACS_Extern/metis-5.1.0.tar.gz && \
tar -czvf TACS_extern.tar.gz metis*.tar.gz UFconfig*.tar.gz  AMD*.tar.gz &&\
tar -xzf metis*.tar.gz && \
cd metis-5.1.0 && make config prefix=$HOME/dafoam/repos/tacs/extern/metis/ CFLAGS="-fPIC" && make install && \
cd ../../ && \
cp Makefile.in.info Makefile.in && \
ls && \
sed -i "s/git/dafoam\/repos/g" Makefile.in && \
make && pip install -e . && \
cd extern/f5tovtk && make && cp f5tovtk $HOME/dafoam/OpenFOAM/sharedBins
</pre>

## **DAFoam regression tests**

To verify the DAFoam installation, you can run the regression tests:

<pre>
cd $HOME/dafoam/repos/dafoam/tests && ./Allrun
</pre>

The regression tests should take less than 30 minutes. The test progress will be printed to screen. Make sure you see this at the end:

<pre>   
************************************************************
**************** All DAFoam tests passed! ******************
************************************************************
</pre>

|

In summary, here is the folder structures for all the installed packages:

<pre>
$HOME/dafoam
  loadDAFoam.sh
  - OpenFOAM
    - OpenFOAM-v1812
    - OpenFOAM-v1812-ADF
    - OpenFOAM-v1812-ADR
    - ThirdParty-v1812
  - packages
    - Ipopt
    - miniconda3
    - CGNS-4.1.2
    - openmpi-3.1.6
    - petsc-3.14.6
  - repos
    - baseclasses
    - cgnsutilities
    - dafoam
    - funtofem
    - idwarp
    - multipoint
    - mphys
    - pygeo
    - pyhyp
    - pyofm
    - pyoptsparse
    - pyspline
    - tacs
</pre>

The loadDAFoam.sh file should look like this:

<pre>
#!/bin/bash
# DAFoam root path
export DAFOAM_ROOT_PATH=$HOME/dafoam
# Miniconda3
export PATH=$DAFOAM_ROOT_PATH/packages/miniconda3/bin:$PATH
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$DAFOAM_ROOT_PATH/packages/miniconda3/lib
export PYTHONUSERBASE=no-local-libs
# OpenMPI-3.1.6
export MPI_INSTALL_DIR=$DAFOAM_ROOT_PATH/packages/openmpi-3.1.6/opt-gfortran
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$MPI_INSTALL_DIR/lib
export PATH=$MPI_INSTALL_DIR/bin:$PATH
# PETSC
export PETSC_DIR=$DAFOAM_ROOT_PATH/packages/petsc-3.14.6
export PETSC_ARCH=real-opt
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$PETSC_DIR/$PETSC_ARCH/lib
export PETSC_LIB=$PETSC_DIR/$PETSC_ARCH/lib
# CGNS-4.1.2
export CGNS_HOME=$DAFOAM_ROOT_PATH/packages/CGNS-4.1.2/opt-gfortran
export PATH=$PATH:$CGNS_HOME/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$CGNS_HOME/lib
# Ipopt
export IPOPT_DIR=$DAFOAM_ROOT_PATH/packages/Ipopt
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$IPOPT_DIR/lib
# OpenFOAM-v1812
source $DAFOAM_ROOT_PATH/OpenFOAM/OpenFOAM-v1812/etc/bashrc
export LD_LIBRARY_PATH=$DAFOAM_ROOT_PATH/OpenFOAM/sharedLibs:$LD_LIBRARY_PATH
export PATH=$DAFOAM_ROOT_PATH/OpenFOAM/sharedBins:$PATH
</pre>

## **Compile SNOPT for pyOptSparse (optional)**

This step is needed if you want to use the SNOPT optimizer. Detailed instructions are available from [pyOptSparse Documentation](https://mdolab-pyoptsparse.readthedocs-hosted.com).

SNOPT is a commercial package, and you can purchase it from [here](http://www.sbsi-sol-optimize.com/asp/sol_snopt.htm). Once you obtain the SNOPT source code, copy all the source files (except for snopth.f) to the "$HOME/dafoam/repos/pyoptsparse-2.8.1/pyoptsparse/pySNOPT/source" folder. Then, run this command to compile pyOptSparse with SNOPT.

<pre>
cd $HOME/dafoam/repos/pyoptsparse-2.8.1 && \
pip install .
</pre>


## **Make the DAFoam package portable (optional)**

This step is only needed if you want to change the root path of your installation, e.g., copy your compiled DAFoam packages to another directory.

The only thing you need to do is to modify the interpreter lines "#!" for files in $HOME/dafoam/packages/miniconda3/. This is because Miniconda hard codes the Python path, so we need to chagne it to "#!/usr/bin/env python"

First find an example of the hard-coded interpreter line from $HOME/dafoam/packages/miniconda3/bin/conda. Run this command

<pre>
head -1 $HOME/dafoam/packages/miniconda3/bin/conda
</pre>

You may see an output like this:

<pre>
#!/home/replace_this_with_your_username/dafoam/packages/miniconda3/bin/python
</pre>

Then run this command to replace all the hard-coded interpreter lines:

<pre>
sed -i 's,^#\!/home/replace_this_with_your_username/dafoam/packages/miniconda3/bin/python,#!/usr/bin/env python,g' $HOME/dafoam/packages/miniconda3/*/*
</pre>

Finally, you can change the DAFOAM_ROOT_PATH value (in loadDAFoam.sh) to your new directory, source the "loadDAFoam.sh" script again, and run DAFoam without compiling everything again.


## **Tensorflow C API (optional)**

If you want to use Tensorflow C CPI in DAFoam, run the following commands:

<pre>
cd $HOME/dafoam/packages && \
mkdir tensorflow-1.15 && \
cd tensorflow-1.15 && \
wget https://github.com/DAFoam/files/releases/download/TensorFlow/libtensorflow-cpu-linux-x86_64-1.15.0.tar.gz && \
tar -xvf libtensorflow-cpu-linux-x86_64-1.15.0.tar.gz && \
echo '# Tensorflow' >> $HOME/dafoam/loadDAFoam.sh && \
echo 'export LIBRARY_PATH=$LIBRARY_PATH:$DAFOAM_ROOT_PATH/packages/tensorflow-1.15/lib' >> $HOME/dafoam/loadDAFoam.sh && \
echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$DAFOAM_ROOT_PATH/packages/tensorflow-1.15/lib' >> $HOME/dafoam/loadDAFoam.sh && \
echo 'export TENSOR_FLOW_INCLUDE_PATH=$DAFOAM_ROOT_PATH/packages/tensorflow-1.15/include' >> $HOME/dafoam/loadDAFoam.sh && \
. $HOME/dafoam/loadDAFoam.sh
</pre>

The Tensorflow C API is ready to use, and DAFoam will compile Tensorflow related libraries when running `./Allmake`.

{% include links.html %}
