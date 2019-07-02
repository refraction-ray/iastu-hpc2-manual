In this page, you will be introduced to the standard workflow on job written with Fortran, C or C++, since they share similar workflows.

## Compiling

### Possible warning by ld

`ld: warning: libmpi.so.20, needed by //usr/lib/x86_64-linux-gnu/libmpi_mpifh.so.20, may conflict with libmpi.so.12` such warning info when compiling linking might be neutral. There may not be any issues for the final binary executable. One should further confirm whether there is some conflict on mpi dependence by directly running the program `mpiexec`.

## Running

### Possible Intel MPI issues

Have merged into spack module files, the following info is here for historical reason and administrators.

Somehow, `spack load intel-parallel-studio%intel` may be not enough to correctly `mpiexec` some programs (using `MPI_Init`), one may need further `source /opt/intel/bin/compilervars.sh intel64`. Especially when you meet the error as `MPIDI_NM_mpi_init_hook(705): OFI addrinfo() failed (ofi_init.h:705:MPIDI_NM_mpi_init_hook:No data available)`. This may originates from Intel 2019.1 bug. (see this [post](https://software.intel.com/en-us/forums/intel-clusters-and-hpc-technology/topic/799716) or this [issue](https://github.com/QMCPACK/qmcpack/issues/1158)). (But it may still amazingly slow to start up mpi program?)

Or actually the two missing env vars is `export FI_PROVIDER=sockets`, `export FI_PROVIDER_PATH="/opt/intel/compilers_and_libraries_2019.3.199/linux/mpi/intel64/libfabric/lib/prov"`.