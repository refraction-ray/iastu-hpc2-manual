Module systems and its integration to the package manager spack is the key part in our HPC, and it is necessay to understand the basics about spack and lmod to utilize softwares in our HPC. Otherwise, no softwares can be used with the bare system apart from gcc and openmpi. 

When you first log into the cluster, you'll be presented with a default, bare bone environment with minimal software available. The module system is used to manage the user environment and to activate software packages on demand. In order to use software installed, you must first load the corresponding software module.

We highly recommend you to read through the section and find packages you need by spack instead of installing them again in your own home directory, which is both a waste of space and time. If the softwares and modules you need is not listed in `spack find`, and you believe the package is somewhat universal to the users, please contact the adminstrator to install.

## spack

Spack is a flexible package manager for modern HPC. The module systems is shipped with spack. Therefore spack plays central role in our HPC systems. For interested readers, I recommend [this blog](https://re-ra.xyz/Spack-%E5%85%A5%E9%97%A8%E6%8C%87%E5%8D%97/) as a good introduction to spack usage in the administrator's perspective.

### syntax about the package

We use `<package>` for the syntax of a package.

* name: `armadillo`
* version: `@8.9.1`
* dependent: `^intel-parallel-studio`
* compiler: `%gcc`
* hash: `/xyzabc`
* option: `+mpi`, `~mpi`

To specify a package in spack, you must provide a legal syntax as described above. You don't need to write down all of them for asking a package, instead as long as there is no ambiguous, the package would uniquely recognized.

### Basics on spack

* `spack find` to see all packages abailable in the system.
* `spack load <package>` to load the package into environment. In other words, to be able to use it.
* `spack unload <package>` to unload the package from enviroment.
* `module list` show all loaded packages now.
* `module purge` unload all packages.

**Rule of thumb for loading package:**

* Want to use icc, ifort or intel mpi, intel mkl or optimized intel python? `spack load intel-parallel-studio%intel`.
* Want to use some advanced linear algebra wrapper instead of bare lapack API? Try `spack load eigen` or see following on `spack env activate arma` .
* Want to use mathematica? `spack load mathematica`.
* Want to use a very fast python3 with scientific packages? See following `spack env activate pyml`.
* [Advanced] Want to use a container solution to make the deploy of tasks simpler? `spack load singularity`.

## prepared environments

The users can use prepared enviroments to develop their projects. The benefits of using spack environments is 1) there will be no conflict when `spack load ` packages, 2) the enviroments are tested and are guaranteed to work for certain tasks.

The general workflow to use prdefined enviroments.

```bash
$ module purge ## make sure no pre loaded package
$ spack env list  ## list all envs
==> 3 environments
    arma  dised  pyml
$ spack env activate arma ## activate one of the env
$ spack env status
==> In environment arma
$ spack find ## only packages in arma are shown
==> In environment arma
==> Root specs
armadillo  intel-parallel-studio

==> 6 installed packages
-- linux-ubuntu18.04-x86_64 / gcc@7.4.0 -------------------------
armadillo@8.100.1  arpack-ng@3.7.0  intel-mpi@2019  openblas@0.3.6  superlu@5.2.1

-- linux-ubuntu18.04-x86_64 / intel@2019 ------------------------
intel-parallel-studio@2019
$ spack load armadillo ## only root specs packages need to be load
$ spack load intel-parallel-studio ## and no worry on package syntax, name is enough in environments
$ module list ## see currently loaded package

Currently Loaded Modules:
  1) armadillo-8.100.1-gcc-7.4.0-openblas-le   2) intel-parallel-studio-2019-intel-2019-26
  
### do tasks depend on this enviroment here

$ spack env deacivate ## exit the environment
$ spack env status
==> No active environment
```

Currently we have the following spack environments.

* dised

  Enviroments that supports distributed exact diagonalization, slepc and petsc package are included. The mpi and compiler should all be from intel family (icc and mpiicc).

* pyml

  Enviroments for python and machine learning. Python 3.6.8 optimized by intel with tensorflow, pytorch and keras. All comes with GPU support. Other packages include pandas, numpy, scipy, scikit-learn, mpi4py, quspin (ED calculation), jupyter environment and more.

* arma

  Enviroments that supports the usage on armadillo package for matrix calculation in C++.

A shortcut for activate certian env with modules auto loaded. To load this environment, type:
`source /home/ubuntu/spack/var/spack/environments/{env_name}/loads`

## Using module directly

It is in general possible to avoid using spack and use `module` command directly. The only problem is that the module name is not so neat and hard to memorize compared to package syntax for `spack load`. However you can still do this.

```bash
$ module avail ## show all avilable software modules in the system
$ module spider slepc ## see the module name related to slepc
----------------------------------------------------------------------------------------------------------------------------------------------------
  slepc-3.11.1-intel-2019-2x: slepc-3.11.1-intel-2019-2x
----------------------------------------------------------------------------------------------------------------------------------------------------

    This module can be loaded directly: module load slepc-3.11.1-intel-2019-2x
$ module load slepc-3.11.1-intel-2019-2x ## here you go
$ module whatis slepc-3.11.1-intel-2019-2x ## see the one-line information of the module
slepc-3.11.1-intel-2019-2x: Scalable Library for Eigenvalue Problem Computations.
```

**Note:** spack package syntax doesn't work for `module load`. You  must specify the full module name.

## used within scripts

**Must read:** Spack is only activated when you log in. It indicates that spack is not accessible by default in scripts such as sbtach scripts. Namely, the following scripts would fail when submitting jobs.

```bash
#! /bin/bash
# A wrong version of script
spack load armadillo
mpirun ...
```

The correct way to call spack in scripts is firstly activate spack itself by `source /etc/spack-load`. So the correct version should looks like

```bash
#! /bin/bash
# A correct version of script
source /etc/spack-load # this is very important, always add this line in sbatch script in case of error
spack load armadillo
mpirun ...
```

**Warn:** Even if you are trying to use module system directly in sbatch script, like `module load slepc-3.11.1-intel-2019-2x`. `source /etc/spack-load` line should still be there to make `module` work. In sum, always write down  `source /etc/spack-load`  in your sbatch script whatever the case is. It has no side effects anyway.