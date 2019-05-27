In this section, I will discuss on slurm and its ecosystem, including how can it interacts with mpi, python, mathematica and so on.

In a word, slurm is very powerful but has really bad presentation of documentation.

## MPI

### PMI

PMI is a key interface to understand how slurm work with mpi. Both of them has a lower level implementation which allocate resource called PMI. For `srun`, PMI of slurm is used, which can be tuned by `--mpi=<pmi>`, and all supported pmis can be shown by `srun --mpi=list`.

Note the program is compiled by `mpicxx`, which depends on the PMI implementation of corresponding MPIs. The PMI for compiling (provided by mpi) and for running (`srun` provided by slurm or `mpirun` provided by mpi ) should be the same. If not, the common error is each process think itself as a standalone rank 0 process.

To make them consistent, there are two approaches. Firstly, compile slurm with more PMI support or secondly, compile corresponding mpi implementation with slurm PMI support. Note the supported PMI is very limited for `apt install`ed slurm and the pmi header file is also missing for a standard package installation indicating compiling mpi implmentations with slurm PMI support is also subtle. 

Therefore, the best practice with minimal maintence effort here is always using `mpirun` within sbatch script and avoid `srun`. But sbatch script is still highly recommended way to submit tasks instead of directly using `mpirun -host <hostname,list>`. Firstly, the sbatch task management is under the control and accounting of slurm, and the task would be running even after logout. Secondly, the enviroment variables in the master node would broadcast to the computing nodes before the task is beginning which is very handy.

### MPI and OPENMP hybrid

Remember the `-fopenmp` flag for `mpicc`,(use `-openmp` for the Intel compiler and `-mp` for the PGI compiler) the others are similar to mpi workflow. See [here](https://rcc.uchicago.edu/docs/running-jobs/hybrid/index.html) for a demo.

### cron like job

See [here](https://rcc.uchicago.edu/docs/running-jobs/cron/index.html), and example sbatch script below.

```bash
#!/bin/bash
#SBATCH --job-name=cron
#SBATCH --begin=now+7days
#SBATCH --dependency=singleton
#SBATCH --time=00:02:00
#SBATCH --mail-type=FAIL


## Insert the command to run below. Here, we're just storing the date in a
## cron.log file
date -R >> $HOME/cron.log

## Resubmit the job for the next execution
sbatch $0
```

### parallel job array submission

See [here](https://rcc.uchicago.edu/docs/running-jobs/srun-parallel/index.html) and [here](https://rcc.uchicago.edu/docs/running-jobs/array/index.html) and [here in Mandarin](http://bicmr.pku.edu.cn/~wenzw/pages/slurm.html)

For job arrays, some import notes. Use `# SBATCH --array=1-5` to name the job as `jobid_1` and so on. In the sbatch script, use env var `${SLURM_ARRAY_TASK_ID}` to call the id for specific tast. And for `-N` or `-n`  in slurm, just use the value for one task. %A in the #SBATCH line becomes the job ID%a in the #SBATCH line becomes the array index. See [here](https://crc.pitt.edu/multiplejobs) for more user case demo.

### allocate computation node interactively

See [this blog](https://yunmingzhang.wordpress.com/2015/06/29/how-to-use-srun-to-get-an-interactive-node/) for details. For short, `srun -N 1 -n 1 -w node1 --pty bash -i`. A slurm way of ssh. Or `salloc -n2 -N1 -t 1:00:00` to allocate a node with give time and resource (2CPU cores). Then ssh to it (-X is support in this case).

? Still have no clear idea on `-n` and the real number of cpu cores the job can use.

## Management and Accounting

Use `Weight` option in NodeName line in `slurm.conf` to change the priority of assinged nodes, see [this post](https://stackoverflow.com/questions/28035631/how-do-i-set-the-order-of-nodes-for-a-slurm-job).

Details and roles on `sacctmgr` family commands: [ref](https://wiki.fysik.dtu.dk/niflheim/Slurm_accounting) (better than the official doc)

### QOS

QOS settings by saccmgr: [ref](https://slurm.schedmd.com/qos.html). 

User system hierachy: cluster-account-user, the triple system is called association.

`AccountingStorageEnforce=limits,qos` This line is important for qos to work in slurm.conf. limits implies association option, indicating user who are not added to slurm cannot use slurm.

**Note**: A user's account can not be changed directly. A new association needs to be created for the user with the new account. Then the association with the old account can be deleted.

```bash
sacctmgr show tres
sacctmgr add qos limited
sacctmgr modify qos limited set MaxTRESPerUser=node=1
sacctmgr modify user test set qos=limited
sacctmgr show assoc format=cluster,account,user,qos
```

Note usename is the same for OS and slurm.

### PAM module

*merged into ansible workflow*

[reference](https://slurm.schedmd.com/pam_slurm_adopt.html)

*

* `sudo apt install libpam-slurm`

* vim /etc/pam.d/sshd, add line `account    required      pam_slurm.so`. The order of plugins is very important. pam_slurm_adopt.so should be the last PAM module in the account stack.  And line `account    required      pam_access.so` following.

* Edit /etc/security/access.conf, add the following

  ```bash
  +:sudo:ALL
  -:ALL:ALL
  ```

Issue: seems always fail to ssh even if there is some task on the corresponding nodes. Though it is not a big issue that there is always a slurm version of ssh works. **Solved:** this issue is due to the mismatch between pam slurm and pam slurm adopt.

## Working with other tools

### Python

#### Jupyter

Directly access to jupyter server in compute nodes. [doc](https://docs.ycrc.yale.edu/clusters-at-yale/guides/jupyter/)

SSH forward with four machines: [post](https://www.ibm.com/developerworks/cn/linux/l-cn-sshforward/index.html). Solution for jupyter in computation nodes

#### mpi4py

`mpirun -n 2 python3 pympi.py`, see [here](https://www.rc.fas.harvard.edu/resources/documentation/software-development-on-odyssey/mpi-for-python-mpi4py-on-odyssey/) for a slurm working example.

#### ipyparallel

*To be detailed studied*

### Mathematica

**References:**

* <https://community.wolfram.com/groups/-/m/t/984003>
* <https://rcc.uchicago.edu/docs/software/environments/mathematica/index.html> mathematica usage as some hpc manual
* <https://www.wolfram.com/products/applications/sem/manual.pdf>
* <https://taoofmac.com/space/blog/2016/08/10/0830> mathematica on rasberry cluster
### spark

The slurm sbatch script to utlize spark cluster: [demo script](https://www.sherlock.stanford.edu/docs/software/using/spark/)