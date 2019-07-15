In this page, you will be introduced to the standard workflow on job written with Fortran, C or C++, since they share similar workflows.

## Compiling

### References

[Basic usage on gcc](https://wiki.ubuntu.org.cn/Gcchowto)

[Compiling C programs](https://wiki.ubuntu.org.cn/Compiling_C)

[Compiling C and Fortran mixed](https://wiki.ubuntu.org.cn/Mix_C_Fortran)

### Possible warning by ld

`ld: warning: libmpi.so.20, needed by //usr/lib/x86_64-linux-gnu/libmpi_mpifh.so.20, may conflict with libmpi.so.12` such warning info when compiling linking might be neutral. There may not be any issues for the final binary executable. One should further confirm whether there is some conflict on mpi dependence by directly running the program `mpiexec`.

### Linking to MKL

`spack load intel-parallel-studio %intel`, `icc -mkl src.c` is in general enough (However, if you are trying to deal with some large array with number of elements larger than $$2^{32}$$, then more special attention should be paid to ensure that the program is linked against ilp64 interface, see [intel doc on 64 bit integer support of mkl](https://software.intel.com/en-us/mkl-macos-developer-guide-using-the-ilp64-interface-vs-lp64-interface)). 

By default, `-mkl` flag for icc, automatically link to various specific mkl library including lp64, instead of linking to the one for all library mkl_rt.so. So runtime enviroment variables setting has no effect on the byte size of MKL_INT.

Also notice that the stack size set by ulimit has a very low soft limit which may lead to segfault for lapack routine in MKL. Baiscally, 8192kb is too small for large array in stack (variable definition in main function as the matrix). Therefore, it would be better to reset `ulimit -s unlimited` when running some program with intel MKL. For executable to be submmited by sbatch, such setting is not necessary since slurm won't inherit the ulimit limitations from local host by configuration.

### Using armadillo

Please use icc compiler for correctly linking to armadillo lib, which can be loaded by `spack load armadillo %intel`. Compiling program by `icc -larmadillo a.cpp`. However, such simple compiling may have issues for int arguments on lapack routine. Namely, the work space length couldn't go further than $$2^{31}-1$$. 

If you want to do some serious linear algebra tasks on very large matrix, you need 64 bit long type for integers and the prototype of lapack routines. To achieve this, there are two ways, both try to linking to ilp64 lib provided by intel. Before doing that, one need to hack the armadillo include files config.hpp to uncomment the BLAS_LONG macro (which is already done by `armadillo%intel` spack module). Besides, you need either: 

1. Try compiling as `icc -larmadillo  -DMKL_ILP64 -I${MKLROOT}/include a.cpp  -L${MKLROOT}/lib/intel64 -lmkl_intel_ilp64 -lmkl_intel_thread -lmkl_core -liomp5 -lpthread -lm -ldl` to make sure ilp64 library is included.
2. Or by default, single library mkl_rt is linked by armadillo, you could specify the ipl64 interface at run time. Namely a simple `export MKL_INTERFACE_LAYER=ILP64` is enough. [Intel reference](https://software.intel.com/en-us/mkl-linux-developer-guide-dynamically-selecting-the-interface-and-threading-layer).

## Running


### Possible Intel MPI issues

Have merged into spack module files, the following info is here for historical reason and a reminder of administrators.

Somehow, `spack load intel-parallel-studio%intel` may be not enough to correctly `mpiexec` some programs (using `MPI_Init`), one may need further `source /opt/intel/bin/compilervars.sh intel64`. Especially when you meet the error as `MPIDI_NM_mpi_init_hook(705): OFI addrinfo() failed (ofi_init.h:705:MPIDI_NM_mpi_init_hook:No data available)`. This may originates from Intel 2019.1 bug. (see this [post](https://software.intel.com/en-us/forums/intel-clusters-and-hpc-technology/topic/799716) or this [issue](https://github.com/QMCPACK/qmcpack/issues/1158)). (But it may still amazingly slow to start up mpi program after correctly enviroment settings?)

Or actually the two missing env vars is `export FI_PROVIDER=sockets`, `export FI_PROVIDER_PATH="/opt/intel/compilers_and_libraries_2019.3.199/linux/mpi/intel64/libfabric/lib/prov"`.