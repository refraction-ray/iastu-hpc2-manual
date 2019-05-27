**Note:** Since users' computation tasks are usually depending on other packages and softwares, it would be better to read the [module section](./module_spack.md) first. In that section, you would be told how to load softwares you want to use or link with. If you don't load any softwares, the system has very limited default softwares like gcc and openmpi, which in general cannot meet your research needs.

In the following, you will be introduced to the general workflow of using the cluster. We assume that the users have adequate knowledge of compiling source code, linking shared library, running executable and mpi programming  in single node Linux. If not, please learn basics about Linux based programming first.

## connecting

This part include SSH to use HPC's CLI as well as file transfers to upload src code/download data files. Please see [connecting section](./connecting.md) on these tasks.

## compile jobs

Use master node to edit src code and compile your binary. You may want to `spack load` approriate packages in this step. For example, if your code depdends on MKL, then you must firstly `spack load intel-parallel-studio%intel` first.

**Some tips on compiling src code:**

* Since you may need to compile some projects again and again, we strongly recommend the users write up their own Makefile to speed up and reduce error.


* When intel mkl is utilized in the code, explicitly or implicitly via dependences, we strongly recommended to use intel compiler to avoid boring and fragile linking flags for gcc.

## submit jobs

**Warning:** Login nodes [master] are not for running computation intensive jobs in general. But since the cluster has very limited computation nodes, it is ok to run some tasks on master node without affecting others. But always be careful when you decided to run some tasks on master, it is taken as the last sort. Use computation nodes first if available.

Before arranging your jobs on the cluster, you should first learn about hardware specs (CPU and memory info in particular) for each nodes to better utilize the computation resource. See hardware specs [here](../../administrators/hardwares/README.md). A very quick summary, for each nodes in the HPC available to normal users, there are 56 CPU threads and a minimum of 128G memory.

In our cluster, we use SLURM as resource and job managers. Ideally, all jobs should be submitted via slurm.

### sbatch basics

The most common way to submit jobs is by `sbatch`. First you need to create some bash scripts, for example `run.sh`.

```bash
#! /bin/bash
#SBATCH -N 1
source /etc/spack-load
spack load intel-parallel-studio %intel
mpiexec -n 56 ./helloworld /DATA/<user>/task/input.txt
```

Let's read the sbtach script line by line.

The first line is just shebang, indication the intepreter of the script. You don't need to change it for most of the time.

The second line is begin with `#SBATCH`, which indicates parameters of sbatch, there are more parameters, which can be written down line by line. But `-N` is the must-have one, it indicates how manty nodes are required for this task. Please `man sbatch` for other parameters. Some more parameters on compute resource request are recommended, such as `--cpus-per-task`, `—ntasks`, `--mem-per-cpu`.

The following line is to activate spack, it is required when you want to load some modules. Since it is so common, so **it would be better to always keep this line in the beginning of the script:** `source /etc/spack-load`.

The fourth line implies that we want to load intel module, which includes MKL, ICC, IFORT, MPI and so on.

Finally the fifth line tells slurm to run a mpi task with 56 threads.

You can submit the task by `sbatch run.sh`, and check the status of the task by `squeue` or `squeue -o %all` if you really enjoy lots of info.

You can check the stdout of the job by `slurm-<jobid>.out`.

You can cancel the task by `scancel <task_id>`, task id can be obtained by squeue.

**Warn:** 

1. **Don't use srun in sbatch scripts for mpi tasks**. Instead **use mpiexec directly**. Since slurm in our cluster has no support for pmix modules, srun doesn't work as the alias of mpiexec as expected.


2. Better not use srun directly for submitting jobs. Since such jobs would be killed as long as the ssh connection is off.

### job arrays

Use the `--array` option in sbatch and the env vars `SLURM_ARRAY_TASK_ID`. Noe in such a script, both N and n are specified for one task.

```bash
#! /bin/bash
#SBATCH -N 1
#SBATCH --array=1-8
#SBATCH --cpus-per-task=14

source /etc/spack-load

spack load intel-parallel-studio %intel

python calculate.py output-${SLURM_ARRAY_TASK_ID}.txt
```

### interactive sessions on compute nodes

1. `srun -N 1 -n 1 -w c3 --pty bash -i`，run a bash shell on c3 node with one node and one cpu core.
2. `salloc -n2 -N1 -t 1:00:00`, and then ssh to the assigned node `ssh [-X] cn`.

```
 Optional arguments for salloc:
        -n      number of CPU cores to request 
        -N      number of nodes to request
        -m      memory amount to request
        -t      time limit, format hh:mm:ss
```

**Warning:** If you try to ssh to a compute node without any active job allocation, the connection wil be denied. By connection closed prompt.