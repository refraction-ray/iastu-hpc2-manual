## compile jobs

When intel mkl is utilized in the code, explicitly or implicitly, we strongly recommended to use intel compiler to compile the program to avoid boring and fragile linking flags for gcc.

Login nodes master are not for running computation intensive jobs in general. But since the cluster has very limited computation nodes, it is ok to run some tasks on master node without affecting others. But always be careful when you decided to run some tasks on master, it is taken as the last sort. Use computation nodes first if available.

## submit jobs

Before arranging your jobs on the cluster, you should first learn about hardware specs (CPU and memory info in particular) for each nodes to better utilize the computation resource. See hardware specs [here](../../administrators/hardwares/README.md).

In our cluster, we use SLURM as resource and job managers. Ideally, all jobs should be submitted via slurm.

## use slurm to submit jobs

```bash
#! /bin/bash
#SBATCH -N 1
#SBATCH --array=1-8
#SBATCH --cpus-per-task=14

source /etc/spack-load

spack load intel-parallel-studio %intel

python calculate.py output-${SLURM_ARRAY_TASK_ID}.txt
```

`srun -w master -n 1 hostname` specify the node to use