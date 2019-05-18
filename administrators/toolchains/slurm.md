In this section, I will discuss on slurm and its ecosystem, including how can it interacts with mpi, python, mathematica and so on.

## MPI

### PMI

PMI is a key interface to understand how slurm work with mpi. Both of them has a lower level implementation which allocate resource called PMI. For `srun`, PMI of slurm is used, which can be tuned by `--mpi=<pmi>`, and all supported pmis can be shown by `srun --mpi=list`.

Note the program is compiled by `mpicxx`, which depends on the PMI implementation of corresponding MPIs. The PMI for compiling (provided by mpi) and for running (`srun` provided by slurm or `mpirun` provided by mpi ) should be the same. If not, the common error is each process think itself as a standalone rank 0 process.

To make them consistent, there are two approaches. Firstly, compile slurm with more PMI support or secondly, compile corresponding mpi implementation with slurm PMI support. Note the supported PMI is very limited for `apt install`ed slurm and the pmi header file is also missing for a standard package installation indicating compiling mpi implmentations with slurm PMI support is also subtle. 

Therefore, the best practice with minimal maintence effort here is always using `mpirun` within sbatch script and avoid `srun`. But sbatch script is still highly recommended way to submit tasks instead of directly using `mpirun -host <hostname,list>`. Firstly, the sbatch task management is under the control and accounting of slurm, and the task would be running even after logout. Secondly, the enviroment variables in the master node would broadcast to the computing nodes before the task is beginning which is very handy.

### MPI and OPENMP hybrid

Remember the `-fopenmp` flag for `mpicc`,(use `-openmp` for the Intel compiler and `-mp` for the PGI compiler) the others are similar to mpi workflow. See [here](https://rcc.uchicago.edu/docs/running-jobs/hybrid/index.html) for a demo.

### cron like job

See [here](https://rcc.uchicago.edu/docs/running-jobs/cron/index.html)

### parallel job array submission

See [here](https://rcc.uchicago.edu/docs/running-jobs/srun-parallel/index.html) and [here](https://rcc.uchicago.edu/docs/running-jobs/array/index.html) and [here in Mandarin](http://bicmr.pku.edu.cn/~wenzw/pages/slurm.html)

For job arrays, some import notes. Use `# SBATCH --array=1-5` to name the job as `jobid_1` and so on. In the sbatch script, use env var `${SLURM_ARRAY_TASK_ID}` to call the id for specific tast. And for `-N` or `-n`  in slurm, just use the value for one task. %A in the #SBATCH line becomes the job ID%a in the #SBATCH line becomes the array index. See [here](https://crc.pitt.edu/multiplejobs) for more user case demo.

### allocate computation node interactively

See [this blog](https://yunmingzhang.wordpress.com/2015/06/29/how-to-use-srun-to-get-an-interactive-node/) for details. For short, `srun -N 1 -n 1 -w node1 --pty bash -i`.

## Management and Accounting

Use `Weight` option in NodeName line in `slurm.conf` to change the priority of assinged nodes, see [this post](https://stackoverflow.com/questions/28035631/how-do-i-set-the-order-of-nodes-for-a-slurm-job).

Details and roles on `sacctmgr` family commands: [ref](https://wiki.fysik.dtu.dk/niflheim/Slurm_accounting) (better than the official doc)

## Working with other tools

### Python

#### mpi4py

`mpirun -n 2 python3 pympi.py`, see [here](https://www.rc.fas.harvard.edu/resources/documentation/software-development-on-odyssey/mpi-for-python-mpi4py-on-odyssey/) for a slurm working example.

### Mathematica

**References:**

* <https://community.wolfram.com/groups/-/m/t/984003>
* <https://rcc.uchicago.edu/docs/software/environments/mathematica/index.html> mathematica usage as some hpc manual
* <https://www.wolfram.com/products/applications/sem/manual.pdf>