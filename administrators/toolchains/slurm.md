In this section, I will discuss on slurm and its ecosystem, including how can it interacts with mpi, python, mathematica and so on.

In a word, slurm is very powerful but honestly speaking, it has a really bad presentation of documentation and not so good and large community.

## MPI

### PMI

PMI is a key interface to understand how slurm work with mpi. Both of them has a lower level implementation which allocate resource called PMI. For `srun`, PMI of slurm is used, which can be tuned by `--mpi=<pmi>`, and all supported pmis can be shown by `srun --mpi=list`.

Note the program is compiled by `mpicxx`, which depends on the PMI implementation of corresponding MPIs. The PMI for compiling (provided by mpi) and for running (`srun` provided by slurm or `mpirun` provided by mpi ) should be the same. If not, the common error is each process think itself as a standalone rank 0 process.

To make them consistent, there are two approaches. Firstly, compile slurm with more PMI support or secondly, compile corresponding mpi implementation with slurm PMI support. Note the supported PMI is very limited for `apt install`ed slurm and the pmi header file is also missing for a standard package installation indicating compiling mpi implmentations with slurm PMI support is also subtle. 

Therefore, the best practice with minimal maintence effort here is always using `mpirun` within sbatch script and avoid `srun`. But sbatch script is still highly recommended way to submit tasks instead of directly using `mpirun -host <hostname,list>`. Firstly, the sbatch task management is under the control and accounting of slurm, and the task would be running even after logout. Secondly, the enviroment variables in the master node would broadcast to the computing nodes before the task is beginning which is very handy. Besides, for mpirun to be available on compute node, ORTE need ssh connection, which is closed by pam plugin of slurm, so the only way to run jobs on compute nodes is via slurm interface.

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

`x11` forward option for srun: [doc](https://portal.supercomputing.wales/index.php/index/slurm/interactive-use-job-arrays/x11-gui-forwarding/)

? Still have no clear idea on `-n` and the real number of cpu cores the job can use (seems one for one thread).

## Management and Accounting

Use `Weight` option in NodeName line in `slurm.conf` to change the priority of assinged nodes, see [this post](https://stackoverflow.com/questions/28035631/how-do-i-set-the-order-of-nodes-for-a-slurm-job). Also, the cpu cores, sockets must by given in slurm.conf, slurm has no ability to auto detect.

Details and roles on `sacctmgr` family commands: [ref](https://wiki.fysik.dtu.dk/niflheim/Slurm_accounting) (better than the official doc)

### billing

[Tres doc](https://slurm.schedmd.com/tres.html)

In slurm.conf, `AccountingStorageTRES=gres/gpu,gres/gpu:tesla`. And in partitial line, `TRESBillingWeights="CPU=1.0,Mem=0.25G,GRES/gpu=2.0"`. If TRESBillingWeights is not defined then the job is billed against the total number of allocated CPUs.

`shares` in user of `sacctmgr`

Share in user's perspective: [doc](https://www.rc.fas.harvard.edu/fairshare/) 

`GrpTRESMins` in sacct modify user's total running time, and multifactor priority plugin to check the raw usage by sshare.

### partition

Nodes may be in more than one partition.

`scontrol show partition`

`sacctmgr add user name partition=gpu`

or in slurm.conf: `PartitionName=MONTHLY1 AllowAccounts=diamond Nodes=compute-0-0`. More conf paramters include `AllocNodes` (no idea the difference between `Nodes`), `Default=YES`,`Hidden`,`State`,`PreemptMode`,`TRESBillingWeights`

A blank list of nodes (i.e. "Nodes= ") can be used if one wants a partition to exist, but have no resources (possibly on a temporary basis).

Specify partition in user's pespective.

- environment variable

```
export SBATCH_PARTITION=<partitionname>
```

- batch script option

```
#SBATCH [-p|--partition=]<partitionname>
```

- command line option

```
sbatch [-p|--partition=]<partitionname>
```

premmpted between queues. Also need global context `PreemptType=preempt/partition_prio`

```
PartitionName=DEFAULT Nodes=tux[0-9]
PartitionName=high Default=NO Shared=FORCE:1 Priority=5 PreemptMode=off
PartitionName=med Default=NO Shared=FORCE:1 Priority=3 PreemptMode=suspend
PartitionName=low Default=YES Shared=NO Priority=1 PreemptMode=requeue
```

### resource manage

`sacctmgr show tres`

In `slurm.conf`, we have `GresTypes` as a comma delimited list of generic resources to be managed (e.g.*GresTypes=gpu,mps*). These resources may have an associated GRES plugin of the same name providing additional functionality. No generic resources are managed by default. Ensure this parameter is consistent across all nodes in the cluster for proper operation. The slurmctld daemon must be restarted for changes to this parameter to become effective.

And also `gres` in node line conf in `slurm.conf` as a comma delimited list of generic resources specifications for a node. The format is:` "<name>[:<type>][:no_consume]:<number>[K|M|G]"`. The first field is the resource name, which matches the `GresType` configuration parameter name. The optional type field might be used to identify a model of that generic resource. A generic resource can also be specified as non-consumable (i.e. multiple jobs can use the same generic resource) with the optional field ":no_consume". The final field must specify a generic resources count. A suffix of "K", "M", "G", "T" or "P" may be used to multiply the number by 1024, 1048576, 1073741824, etc. respectively. (e.g."Gres=gpu:tesla:1,gpu:kepler:1,bandwidth:lustre:no_consume:4G"). By default a node has no generic resources and its maximum count is that of an unsigned 64bit integer. 

More on [gres.conf man page](https://slurm.schedmd.com/gres.conf.html). The gres.conf file may be omitted completely if the configuration information in the slurm.conf file fully describes all GRES.  The file will always be located in the same directory as the **slurm.conf** file.

Jobs will not be allocated any generic resources unless specifically requested at job submit time using the options:

*—gres*: Generic resources required per node, recommended way, general snytax for job submission  `--gres=gpu:kepler:2`

*—gpu*: GPUs required per job

### QOS

QOS settings by saccmgr: [ref](https://slurm.schedmd.com/qos.html). 

User system hierachy: cluster-account-user, the triple system is called association.

`AccountingStorageEnforce=limits,qos` in **slurm.conf**. This line is important for qos to work in slurm.conf. limits also  implies association option, indicating user who are not added to slurm cannot use slurm.

**Note**: A user's account can not be changed directly. A new association needs to be created for the user with the new account. Then the association with the old account can be deleted.

```bash
sacctmgr show tres
sacctmgr add qos limited
sacctmgr modify qos limited set MaxTRESPerUser=node=1
sacctmgr modify user test set qos=limited
sacctmgr show assoc format=cluster,account,user,qos
```

Note usename is the same for OS and slurm.

GrpTRESMins=cpu=10,mem=20 would make 2 different limits 1 for 10 cpu minutes and 1 for 20 MB memory minutes. This is the case for each limit that deals with TRES. To remove the limit -1 is used i.e. GrpTRESMins=cpu=-1 would remove only the cpu TRES limit. When dealing with Memory as a TRES all limits are in MB.

Node QOS is weird. It seems the system takes each task as a node. Even more tasks are shared with the same node, the node count increases. Instead, try limit by cpu. Is cpu for cores or threades? Experiments: cpu in qos context is by cores, say `cpu=28` may limit to 56 threads. It is worth noting however, `--cpus-per-task` is given by threads (not sure now, there are conflicting evidence...)! Slurm seems to have a mix conception of cpu.

QOS limit is more flexible to fine tune than account which cannot be changed unless deleting the user. Or by `sacctmgr modify user where user=example set defaultaccount=groupb`, to at least change the default group.

`sacctmgr add coord names=blah`, users that can modify other users qos and so on.

### PAM module

*merged into ansible workflow*

[reference](https://slurm.schedmd.com/pam_slurm_adopt.html)


* `sudo apt install libpam-slurm`

* vim /etc/pam.d/sshd, add line `account    required      pam_slurm.so`. The order of plugins is very important. pam_slurm_adopt.so should be the last PAM module in the account stack.  And line `account    required      pam_access.so` following.

* Edit /etc/security/access.conf, add the following

  ```bash
  +:sudo:ALL
  -:ALL:ALL
  ```

Issue: seems always fail to ssh even if there is some task on the corresponding nodes. Though it is not a big issue that there is always a slurm version of ssh works. **Solved:** this issue is due to the mismatch between pam slurm and pam slurm adopt.

### scontrol

`scontrol show node c1`, to see how many cores are really allocated.

`scontrol show job 237`, to see the status of give job.

`scontrol show assoc`, to check all details of users, accounts and qos.

`sacct -a`, see all users' jobs, past and current

bring node from down: back to service, restart slurmctld or slurmd wont work, see [this post](https://bitsanddragons.wordpress.com/2016/08/25/slurm-node-state-control/). `scontrol update nodename=c2 state=IDLE`.

`scontrol ping` check the status of master and backup node for slurmctld

### misc

* No conf to randomize node assignment: [post](https://serverfault.com/questions/881099/randomize-slurm-node-allocation), somewhat hard to believe

* `sdiag` to check scheduling relevant info

* `sattach jobid` directly see stdout and stderr of the running job

* `sshare`

* `strigger`:  event trigger. eg. `strigger --set --node --down --program=/usr/sbin/slurm_admin_notify`

* `sacctmgr show stats` on RPC call statistics on sacctmgr

* `scontrol reconfigure` can let slurmctld to reload the conf, thought some parameters may be valid only after restart. `scontrol show config`

* asterik * in status of sinfo indicates the node is unreachable.

* In some system, squeue is aliased to `alias squeue="squeue -u <user>"`, therefore you cannot directly view others' jobs. But you can `unalias squeue`, and then `squeue` can check all jobs by all users.

* singularity plugin [readme](https://github.com/sylabs/singularity/blob/master/docs/2.x-slurm/README.md)

* srun -n parameter seems working well, mathematica `LaunchKernels`  or numpy with mkl multi thread speed both cannot excced the limit by -n.

* A job's expected start time can be seen using the `squeue —start` command.

* PrivateData controls whether some info is accessible to normal users

  > **PrivateData**
  >
  >  This controls what type of information is hidden from regular users. By default, all information is visible to all users. User **SlurmUser** and **root** can always view all information. Multiple values may be specified with a comma separator. Acceptable values include: 
  >
  > accounts, jobs, nodes, users and so on.

* Test legal nodelist syntax

  ```bash
  $ yhcontrol show hostlist cn0,cn1,cn2,cn3,cn6,cn7
  cn[0-3,6-7]
  $ yhcontrol show hostnames cn[0-3,6-7]
  cn0
  cn1
  cn2
  cn3
  cn6
  cn7
  ```

* `sinfo -R` reasons for node status, completing for long time is a indication that something is wrong, most probable case is disconnected between master and the node somehow.

* If `SelectType=select/linear` is configured, all resources on the selected nodes will be allocated to the job/step. If SelectType=select/cons_res is configured, individual sockets, cores and threads may be allocated from the selected nodes

* `sbatch --mem` limit seems not work. The task can easily go over the memory limit without any problem.

* `PropagateResourceLimits` or `PropagateResourceLimitsExcept` parameters are configured in slurm.conf and avoid propagation of specified limits. Configure on these two parameters unless you want to ulimit be effective on compute nodes.

### More reference

* Network topology in slurm: [doc](https://slurm.schedmd.com/topology.html)
* Schduling configuration: [doc](https://slurm.schedmd.com/sched_config.html)
* Job premmption guide: [doc](http://slurm.schedmd.com/preempt.html)
* High throughput guide: namely fine tuning on burst of short jobs for slurm: [doc](https://slurm.schedmd.com/high_throughput.html)
* Gres: [doc](https://slurm.schedmd.com/gres.html)
* Shuguang doc on slurm: [doc](https://www.hpccube.com/wiki/index.php/SLURM%E4%BD%BF%E7%94%A8%E5%9F%BA%E7%A1%80%E6%95%99%E7%A8%8B)
* Tianhe doc on slurm in admin's perspective: [doc](https://www.csrc.ac.cn/upload/file/20151014/1444804067282511.pdf)
* Heterogeneous job  submission: [doc](https://slurm.schedmd.com/heterogeneous_jobs.html) (Including block, plane and cylic allocation)
* Job status and info to elasticsearch: [doc](https://slurm.schedmd.com/elasticsearch.html)

## Working with other tools

### Python

#### Jupyter

Directly access to jupyter server in compute nodes. [doc](https://docs.ycrc.yale.edu/clusters-at-yale/guides/jupyter/)

SSH forward with four machines: [post](https://www.ibm.com/developerworks/cn/linux/l-cn-sshforward/index.html). Solution for jupyter in computation nodes.

#### mpi4py

`mpirun -n 2 python3 pympi.py`, see [here](https://www.rc.fas.harvard.edu/resources/documentation/software-development-on-odyssey/mpi-for-python-mpi4py-on-odyssey/) for a slurm working example.

#### ipyparallel

*To be detailed studied*

[ipcluster command line](https://ipyparallel.readthedocs.io/en/latest/process.html)

[Combine slurm, ipyparallel and celery](https://ulhpc-tutorials.readthedocs.io/en/latest/python/advanced/jupyter-celery/)

### Mathematica

#### remote kernel with the help of  Tunnel

See [manual](https://github.com/sakra/Tunnel/blob/master/MANUAL.md) of tunnel tool. Or more general reference of mma doc ParallelTools/tutorial/ConnectionMethods.

Some recaps: copy `tunnel.m` into `/opt/mathematica/11.0.1/Kernels/Packages`, and add both `tunnel.sh` and `tunnel_sub.sh` into `~/.Mathematica/FrondEnd`, `chmod +x` for the two scripts. (It may be possible to configure the tunnel sh in system wide context?)

tunnel.sh is for remote control kernel, with GUI to launch, tunnel_sub.sh is for compute kernels, launched indirectly by control kernel.

Use local GUI call mathematica remote kernels in master node, 

```bash
# Arguments to MLOpen
-LinkMode Listen -LinkProtocol TCPIP -LinkOptions MLDontInteract -LinkHost 127.0.0.1

# Lauch command
"`userbaseDirectory`/FrontEnd/tunnel.sh" "<user>@<ip>" "/opt/mathematica/11.0.1/Executables/WolframKernel" "`linkname`"
```

Of course you need to configure passwordless ssh login for user or add password above. The two files in launch command are for tunnel.sh and math kernel.

On master node, further call more remote kernels on compute nodes. You can achieve a 224 threads parallel table in our cluster!

```mathematica
Needs["SubKernels`RemoteKernels`"]

$RemoteCommand = 
 "\"" <> $UserBaseDirectory <> 
  "/FrontEnd/tunnel_sub.sh\" \"`1`\" \
\"/opt/mathematica/11.0.1/Executables/MathKernel\" \"`2`\""

kernel = RemoteMachine["<user>@<hostname>", 2, LinkHost -> "127.0.0.1"]
(* 2 is the kernel number on hostname, if the user name is the same, which is the case in our HPC, <user>@ part can be omitted, a cn for host is enough *)

LaunchKernels[kernel]

ParallelTable[$MachineName, {i, 1, Length[Kernels[]]}] (* A test on real parallel fashion *)

CloseKernels[{19, 20, 21}] (*close kernels 19,20,21 *)
```

 The standard mathematica parallel and remot kernel workflow is decided as above.

**References:**

* <https://community.wolfram.com/groups/-/m/t/984003>
* <https://rcc.uchicago.edu/docs/software/environments/mathematica/index.html> mathematica usage as some hpc manual
* <https://www.wolfram.com/products/applications/sem/manual.pdf>
* <https://taoofmac.com/space/blog/2016/08/10/0830> mathematica on rasberry cluster
* <https://mathematica.stackexchange.com/questions/31518/how-to-change-front-end-setting-parallel-kernel-configuration-in-the-code/31834#31834> remote kernel launching
### spark

The slurm sbatch script to utlize spark cluster: [demo script](https://www.sherlock.stanford.edu/docs/software/using/spark/), [so](https://serverfault.com/questions/776687/how-can-i-run-spark-on-a-cluster-using-slurm).

spark enabled jupyter: [doc](https://researchcomputing.princeton.edu/faq/spark-via-slurm), [blog](https://blog.sicara.com/get-started-pyspark-jupyter-guide-tutorial-ae2fe84f594f).

```bash
export PYSPARK_DRIVER_PYTHON=/path/to/your/jupyter
export PYSPARK_DRIVER_PYTHON_OPTS="notebook"
export PYSPARK_PYTHON=`which python`
```

### database

Use database instance in HPC: [doc](https://www.sherlock.stanford.edu/docs/software/using/mariadb/)