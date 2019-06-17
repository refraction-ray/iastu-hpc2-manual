## Containers in general

[Container meets HPC](https://medium.com/@ople/containers-meet-hpc-2aab7aa2d54a)

[k8s, HPC, slurm and MPI](https://www.stackhpc.com/k8s-mpi.html)

## Cgroup

- check supported cgroup subsystem: `cat /proc/cgroups`
- check process in which cgroup: `cat /proc/777/cgroup`
- [cgroup on ubuntu user basis limit](http://www.litrin.net/2016/11/18/ubuntu%E5%9F%BA%E4%BA%8E%E7%94%A8%E6%88%B7%E7%9A%84cgroup%E8%AE%BE%E7%BD%AE/)
- [cgroup on ubuntu 18.04](https://www.paranoids.at/cgroup-ubuntu-18-04-howto/), seems not so built in on ubuntu, need to add systemd service by hands if you wanna auto start with boot….
- [cpu set on numa architecture](https://www.cnblogs.com/shishaochen/p/9735114.html). When using cpuset.cpus, cpuset.mems must also be specified to make it work. The value for mems, is the node number of the cpu binding mems.
- [introduction to cgroup in fs level: series](https://segmentfault.com/a/1190000006917884)
- my experience: say if you want to apply cgroup policy on existing proc, especially these service, then you need restart these service to make the policy work
- [syntax of cgrules](https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/resource_management_guide/sec-moving_a_process_to_a_control_group), `@group`.
- [cpu subsystem parameters and default value](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/resource_management_guide/sec-cpu), by some test, the default value is around 1000 for cpu.share.
- Possible issue: confilict between external cgroup on users and slurm cgroup. Syndrome: task submit failure on cgroup enabled node. `slurmstepd-master: error: Failed to invoke task plugins: task_p_pre_launch error`. More specifically, the error is happen due to external cpuset subsystem from cgroup. My guess is that the cpu core binding external to slurm is unknown to slurmctld, and the assigned cpu cores would be failed by sbatch. Possible solution: 1 easy way (which I took), use cpu instead of cpuset subsystem to limit user cpu usage in cgroup. 2 hard way (which I guess itt should work), in principle, we can assign cpu binding in slurm.conf to avoid assigenment failure. Note this is not a big issue in general, since it is rare when slurmd (which usually only on compute nodes) and cgroup (which usually only configured on login nodes) are both exist in the same machine.

## Singularity

a runtime container suitable for HPC

[site](http://singularity.lbl.gov/)

[singularity image library](https://cloud.sylabs.io/library)

[Manual from HPC user aspect](https://www.nsc.liu.se/support/singularity/)

Docker vs sigularity vs shifter in HPC: [1](https://geekyap.blogspot.com/2016/11/docker-vs-singularity-vs-shifter-in-hpc.html), [2](https://tin6150.github.io/psg/blogger_container_hpc.html), [3](https://www.sylabs.io/2018/04/singularity-compatibility-with-docker-containers/). No sudo dameon, less namespace separation, automount of home, share the same network by default, support MPI, env vars pass to container, user stay the same, more like a local process instead of a VM. And it is much easier than docker in general.

Process - Singularity - Docker -VM: the order of resource separation. Actually singularity is more like AppImage.

Singularity cached images layer are in `~/.singularity`. Namely, they cannot be shared across different user by default. But it seems to be tuned by en vars `$SINGULARITY_CACHEDIR`.

### basic commands

* `singularity exec container.img cat /etc/os-release`. `––contain` flag for a better separation, no pass of certain env vars, say local python packages. `––cleanenv`. `––containall` for more strict namespace separation. `--bind /host:/con`
* `singularity inspect -l --json ubuntu.img`
* `singularity shell centos7.img`, if `writable` is add, then img may be changed.
* `sigularity pull docker://`, not only pull but also transform it to sif format img.
* running as background service using `sigularity instance` commands, see [doc](https://www.sylabs.io/guides/3.2/user-guide/running_services.html). (In this workflow, singularity is more like docker.)

### build image

`.sif` is the standard format for singularity 3+. For image building, you must have root access, which cannot be done on HPC. But instead one can choose remote builder system [here](https://cloud.sylabs.io/builder).

#### direct way by operating in the container

`sudo singularity build --sandbox ubuntu/ library://ubuntu`

`sudo singularity exec --writable ubuntu touch /foo`

` singularity build new-sif sandbox`

#### definition files (recommended)

Below is a demo. Detailed syntax on def files [here](https://www.sylabs.io/guides/3.2/user-guide/definition_files.html).

```bash
BootStrap: library
From: ubuntu:16.04

%post
    apt-get -y update
    apt-get -y install fortune cowsay lolcat

%environment
    export LC_ALL=C
    export PATH=/usr/games:$PATH

%runscript
    fortune | cowsay | lolcat

%labels
    Author GodloveD
```

`sudo singularity build lolcow.sif lolcow.def`

### singularity hub

### common images

clearlinux family?

### Integrated with Kubernetes

reference: <https://www.sylabs.io/2019/04/the-singularity-kubernetes-integration-from-a-deep-learning-use-case-to-the-technical-specifics/>

## k8s

[k8s meets HPC](https://kubernetes.io/blog/2017/08/kubernetes-meets-high-performance/)

For current status, no open source solution that can integerate k8s and slurm well in HPC setup. You can have slurm over container or k8s over mpi, but you have to choose one work load manager.

### Helm