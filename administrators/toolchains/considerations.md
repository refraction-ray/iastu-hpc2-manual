In this part, I would keep some notes on tools or softwares that I haven't fully determined to merge into the main workflow on HPC2, it is kept for possible future technology selections.

List of typical HPCs:

*  [uchicago](https://rcc.uchicago.edu/docs/software/modules/index.html)
*  [princeton](https://researchcomputing.princeton.edu/software)
*  [ulhpc](https://ulhpc-tutorials.readthedocs.io/en/latest/)
*  [qmul](https://docs.hpc.qmul.ac.uk/)
*  [pku-math](http://bicmr.pku.edu.cn/~wenzw/pages/softwares.html)
*  [CWRU](https://sites.google.com/a/case.edu/hpc-upgraded-cluster/available-software)
*  [sherlock-standford](https://www.sherlock.stanford.edu/)
*  [UMBC](https://hpcf.umbc.edu/)
*  [PKU](http://hpc.pku.edu.cn/)
*  [UIOWA](https://wiki.uiowa.edu/display/hpcdocs)
*  [Yale](https://docs.ycrc.yale.edu/clusters-at-yale/)
*  [Sheffield](http://docs.hpc.shef.ac.uk/)

## OpenHPC

*Not included, using spack instead*

[Official repo](https://github.com/openhpc/ohpc/), *note it has a github wiki page*

[Cool wiki list on what tools they contained](https://github.com/openhpc/ohpc/wiki/Component-List-v1.3.7), and possible an important list of softwares that might be useful for HPC in general.

Not a valid option for debian?

## Easybuild

*Not included, using spack instead*

Alternatives to spack.

Some comparison posts: [1](https://groups.io/g/OpenHPC-users/topic/transistion_ohpc_to/265274?p=,,,20,0,0,0::recentpostdate%2Fsticky,,,20,1,0,265274), [2](https://github.com/spack/spack/issues/2115)

## Modules

*Not included, using lmod by spack*

[site](http://modules.sourceforge.net/)

Not a fan of this due to the language it utilized.

## Lmod

alternatives to modules, merged to the main workflow under spack

## Lustre or BeeGFS

file system for HPC

## Dask

python module for parallel computing, basically a parallel version of pandas. Maybe the scheme to utilize distributed computation source with one jupyter notebook frontend. And seems to be more attractive than spark stack in general.

[site](https://dask.org/)

[use dask on HPC style clusters](https://docs.dask.org/en/latest/setup/hpc.html)

## Desktop 

### ThinLinc

A remote server specially desinged for clusters: [site](https://www.cendio.com/thinlinc/what-is-thinlinc)

### X11 forward

See [here](http://bicmr.pku.edu.cn/~wenzw/pages/gui.html). Merged to the main workflow

## Globus

*large file transfer service*

## Scheme beyond MPI

* [Nice blog reflects the drawbacks of MPI style and compare it to spark or chapel](https://www.dursi.ca/post/hpc-is-dying-and-mpi-is-killing-it.html)

### hadoop

### spark

### chapel

## CGroup

* [cgroup on ubuntu user basis limit](http://www.litrin.net/2016/11/18/ubuntu%E5%9F%BA%E4%BA%8E%E7%94%A8%E6%88%B7%E7%9A%84cgroup%E8%AE%BE%E7%BD%AE/)
* [cgroup on ubuntu 18.04](https://www.paranoids.at/cgroup-ubuntu-18-04-howto/), seems not so built in, need to add systemd service by hands….