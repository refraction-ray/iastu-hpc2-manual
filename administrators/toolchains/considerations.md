In this part, I would keep some notes on tools or softwares that I haven't fully determined to merge into the main workflow on HPC2, it is kept for possible future technology selections.

List of softwares in a typical HPC:

*  [uchicago](https://rcc.uchicago.edu/docs/software/modules/index.html)

## OpenHPC

[Official repo](https://github.com/openhpc/ohpc/), *note it has a github wiki page*

[Cool wiki list on what tools they contained](https://github.com/openhpc/ohpc/wiki/Component-List-v1.3.7), and possible an important list of softwares that might be useful for HPC in general.

Not a valid option for debian?

## Singularity

*a runtime container*

[site](http://singularity.lbl.gov/)

It seems quite interesting for this container scheme and share strong similarity with docker.

## Modules

[site](http://modules.sourceforge.net/)

Not a fan of this due to the language it utilized.

## Lmod

alternatives to modules, merged to the main workflow under spack

## Lustre or BeeGFS

lustre file system for HPC

## Dask

python module for parallel computing. Maybe the scheme to utilize distributed computation source with one jupyter notebook frontend.

[site](https://dask.org/)

[use dask on HPC style clusters](https://docs.dask.org/en/latest/setup/hpc.html)

## CGGROUP

## ThinLinc

A remote server specially desinged for clusters: [site](https://www.cendio.com/thinlinc/what-is-thinlinc)