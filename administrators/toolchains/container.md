## Containers in general

[Container meets HPC](https://medium.com/@ople/containers-meet-hpc-2aab7aa2d54a)

## Singularity

a runtime container suitable for HPC

[site](http://singularity.lbl.gov/)

It seems quite interesting for this container scheme and share strong similarity with docker.

[Manual from HPC user aspect](https://www.nsc.liu.se/support/singularity/)

Docker vs sigularity vs shifter in HPC: [1](https://geekyap.blogspot.com/2016/11/docker-vs-singularity-vs-shifter-in-hpc.html), [2](https://tin6150.github.io/psg/blogger_container_hpc.html). No sudo dameon, less namespace separation, automount of home, share the same network by default, support MPI, more like a local process instead of a VM.

Process - Singularity - Docker -VM: the order of resource separation. 

Singularity cached images layer are in `~/.singularity`. Namely, they cannot be shared across different user by default. But it seems to be tuned by en vars `$SINGULARITY_CACHEDIR`.

### basic commands

* `singularity exec container.img cat /etc/os-release`
* `singularity inspect -l --json ubuntu.img`
* `singularity shell centos7.img`, if `writable` is add, then img may be changed

### singularity hub

### common images

clearlinux family?

### Integrated with Kubernetes

reference: <https://www.sylabs.io/2019/04/the-singularity-kubernetes-integration-from-a-deep-learning-use-case-to-the-technical-specifics/>