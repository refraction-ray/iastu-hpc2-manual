In this section, I will discuss on slurm and its ecosystem, including how can it interacts with mpi, python, mathematica and so on.

## MPI

### PMI

PMI is a key interface to understand how slurm work with mpi. Both of them has a lower level implementation which allocate resource called PMI. For `srun`, PMI of slurm is used, which can be tuned by `--mpi=<pmi>`, and all supported pmis can be shown by `srun --mpi=list`.

Note the program is compiled by `mpicxx`, which depends on the PMI implementation of corresponding MPIs. The PMI for compiling (provided by mpi) and for running (`srun` provided by slurm or `mpirun` provided by mpi ) should be the same. If not, the common error is each process think itself as a standalone rank 0 process.

To make them consistent, there are two approaches. Firstly, compile slurm with more PMI support or secondly, compile corresponding mpi implementation with slurm PMI support. Note the supported PMI is very limited for `apt install`ed slurm and the pmi header file is also missing for a standard package installation indicating compiling mpi implmentations with slurm PMI support is also subtle. 

Therefore, the best practice with minimal maintence effort here is always using `mpirun` within sbatch script and avoid `srun`. But sbatch script is still highly recommended way to submit tasks instead of directly using `mpirun -host <hostname,list>`. Firstly, the sbatch task management is under the control and accounting of slurm, and the task would be running even after logout. Secondly, the enviroment variables in the master node would broadcast to the computing nodes before the task is beginning which is very handy.

## Management and Accounting

Details and roles on `sacctmgr` family commands: [ref](https://wiki.fysik.dtu.dk/niflheim/Slurm_accounting) (better than the official doc)

## Working with other tools

### Python

#### mpi4py

`mpirun -n 2 python3 pympi.py`, see [here](https://www.rc.fas.harvard.edu/resources/documentation/software-development-on-odyssey/mpi-for-python-mpi4py-on-odyssey/) for a slurm working example.

### Mathematica

**References:**

* <https://community.wolfram.com/groups/-/m/t/984003>