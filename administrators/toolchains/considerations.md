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
*  [Odyssey-Harvard](https://www.rc.fas.harvard.edu/resources/odyssey-architecture/)
*  [Niflheim](https://wiki.fysik.dtu.dk/niflheim/niflheim)

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

Lustre: need patch on kernel, not considering for mini clusters.

Some configuration reference on lustre: [post](https://wangmingjun.com/2018/06/13/%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%90%AD%E5%BB%BAlustre%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F/)

Small files benchmarking on distrubuted FS [post](https://www.jdieter.net/posts/2017/08/14/benchmarking-small-file-performance-on-distributed-filesystems/)

[Analysis of Six Distributed File Systems](https://hal.inria.fr/file/index/docid/789086/filename/a_survey_of_dfs.pdf)

### GlusterFS

Seems very promising and easy to configure, also more suitable for a small cluster. But after some reading, it seems not to be a good choice as the main fs for HPC.

[Discussion on the comparison between gluster and lustre](https://lustre-discuss.lustre.narkive.com/lE17VMmK/glusterfs-and-lustre)

## Desktop 

### ThinLinc

A remote server specially desinged for clusters: [site](https://www.cendio.com/thinlinc/what-is-thinlinc)

### X11 forward

See [here](http://bicmr.pku.edu.cn/~wenzw/pages/gui.html). Merged to the main workflow

## Globus

*large file transfer service*

## Name Service Switch

`man NSSWITCH.CONF`

## LDAP

openLDAP for user management: [tutorial](https://www.ibm.com/developerworks/cn/linux/l-openldap/index.html)

openLDAP configure: [post](https://juejin.im/entry/5aec6ac46fb9a07ac3635884)

## BACKUP and sync

### Borg Backup

### restic 

*decided to try this in main workflow*

very cool and nice softwares with friendly and clear docs!

### general resources

* [Arch wiki on backup programs](https://wiki.archlinux.org/index.php/Synchronization_and_backup_programs)

## Parallel Scheme beyond MPI

* [Nice blog reflects the drawbacks of MPI style and compare it to spark or chapel](https://www.dursi.ca/post/hpc-is-dying-and-mpi-is-killing-it.html)

### hadoop

### spark

### chapel

language designed for parallel

