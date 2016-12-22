local install on ubuntu 16.04:

first install cuda toolkit from https://developer.nvidia.com/cuda-downloads 

or:

```
sudo apt install cudatoolkit
```


Install petsc from master branch
```sh
git clone -depth 1 -b master https://bitbucket.org/petsc/petsc petsc
cd petsc

# run these and add to bashrc
export PETSC_DIR=# put path to petsc here!
export PETSC_ARCH=arch-linux2-c-debug

# may need to change cuda path based on install/version
export PATH=/usr/local/cuda-8.0/bin:$PATH 
export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64:$LD_LIBRARY_PATH

./configure --download-f2cblaslapack --download-mpich --with-cuda=1 --download-cusp --with-thrust=1 COPTFLAGS='-O3' CXXOPTFLAGS='-O3' FOPTFLAGS='-O3' --with-debugging=0
make all

# add this to path for easy petsc commands
```
