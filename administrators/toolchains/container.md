## Containers in general

[Container meets HPC](https://medium.com/@ople/containers-meet-hpc-2aab7aa2d54a)

[k8s, HPC, slurm and MPI](https://www.stackhpc.com/k8s-mpi.html)

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