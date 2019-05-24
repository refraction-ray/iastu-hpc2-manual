## compile jobs

When intel mkl is utilized in the code, explicitly or implicitly, we strongly recommended to use intel compiler to compile the program to avoid boring and fragile linking flags for gcc.

## use slurm to submit jobs

```bash
#! /bin/bash
#SBATCH -N 1
#SBATCH --array=1-8
#SBATCH --cpus-per-task=14

source /home/ubuntu/spack/share/spack/setup-env.sh
source $(spack location -i lmod)/lmod/lmod/init/bash

spack load intel-parallel-studio %intel

python calculate.py output-${SLURM_ARRAY_TASK_ID}
```

