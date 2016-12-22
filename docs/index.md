local install on ubuntu 16.04:

first install cuda toolkit from https://developer.nvidia.com/cuda-downloads 

or:
~~~~
sudo apt install cudatoolkit
~~~~

Install petsc from master branch
~~~~
git clone -depth 1 -b master https://bitbucket.org/petsc/petsc petsc
cd petsc

# run these and add to bashrc
export PETSC_DIR=`pwd` # for bashrc need to give real path!
export PETSC_ARCH=arch-linux2-c-debug

# may need to change cuda path based on install/version
export PATH=/usr/local/cuda-8.0/bin:$PATH 
export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64:$LD_LIBRARY_PATH

./configure --download-f2cblaslapack --download-mpich --with-cuda=1 --download-cusp --with-thrust=1 COPTFLAGS='-O3' CXXOPTFLAGS='-O3' FOPTFLAGS='-O3' --with-debugging=0
make all

# add this to path for easy petsc commands
~~~~
examples running locally:
~~~~
# to run on GPU:
cd src/ksp/ksp/examples/tutorials
make ex50

# run sequentially
./ex50 -dm_mat_type seqaijcusp -dm_vec_type seqcusp -mat_type seqaijcusp -vec_type seqcusp

# run with 2 mpi processes
mpiexec -n 2 ./ex50 -dm_mat_type mpiaijcusp -dm_vec_type mpicusp -mat_type mpiaijcusp -vec_type mpicusp

# run 6x refined problem with useful logging enabled, specific ksp and pc types
mpiexec -n 2 ./ex50 -log_view -mat_view ::ascii_info -ksp_monitor -ksp_final_residual -ksp_converged_reason -ksp_type bcgsl -pc_type asm -pc_asm_overlap 3 -sub_pc_type ilu -sub_pc_factor_levels 12 -da_refine 6 -dm_mat_type mpiaijcusp -dm_vec_type mpicusp -mat_type mpiaijcusp -vec_type mpicusp
~~~~

blue waters petsc cuda install:
~~~~
ssh traXXX@bwbay.ncsa.illinois.edu


git clone -depth 1 -b master https://bitbucket.org/petsc/petsc petsc
cd petsc

# run these and add to bashrc
export PETSC_DIR=`pwd` # for bashrc need to give real path!
export PETSC_ARCH=BW-gnu

module unload PrgEnv-cray
module load PrgEnv-gnu
module load cmake
module load cudatoolkit

./configure --with-cuda=1 --with-thrust=1 --download-f2cblaslapack --download-mpich=1 --download-cusp=1  COPTFLAGS=-O3 CXXOPTFLAGS=-O3 FOPTFLAGS=-O3 --with-mpiexec=aprun --with-debugging=0
make all
~~~~
examples running on blue waters:
~~~~
# start an interactive session on node with Kepler GPU:
qsub -I -l nodes=1:ppn=16:xk -l walltime=1:00:00

# use aprun instead of mpiexec
aprun -n 1 -N 1 ./ex50 -dm_mat_type seqaijcusp -dm_vec_type seqcusp -mat_type seqaijcusp -vec_type seqcusp
# run with 2 mpi processes
aprun -n 2 -N 2 ./ex50 -dm_mat_type mpiaijcusp -dm_vec_type mpicusp -mat_type mpiaijcusp -vec_type mpicusp

# run on multiple nodes example:
qsub -I -l nodes=2:ppn=16:xk -l walltime=1:00:00
aprun -n 32 -N 16 ./ex50 -dm_mat_type mpiaijcusp -dm_vec_type mpicusp -mat_type mpiaijcusp -vec_type mpicusp
~~~~

example job.pbs file: note that you put multiple aprun commands
in a single job to test a range of settings at once

~~~~
#!/bin/bash

#PBS -A bagx
#PBS -l nodes=1:ppn=16:xk
#PBS -l walltime=01:00:00
#PBS -N gpu-scale
#PBS -e $PBS_JOBID.err
#PBS -o $PBS_JOBID.out

cd $PBS_O_WORKDIR
aprun -n 1 -N 1 ./ex50 -dm_mat_type seqaijcusp -dm_vec_type seqcusp -mat_type seqaijcusp -vec_type seqcusp -da_refine 7 
aprun -n 1 -N 1 ./ex50 -dm_mat_type seqaijcusp -dm_vec_type seqcusp -mat_type seqaijcusp -vec_type seqcusp -da_refine 8
aprun -n 1 -N 1 ./ex50 -dm_mat_type seqaijcusp -dm_vec_type seqcusp -mat_type seqaijcusp -vec_type seqcusp -da_refine 9
~~~~
