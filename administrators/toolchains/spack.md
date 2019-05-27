In this section, I will give some information on the package manager designed for HPC. It would be ideal that all computation related software is under the control of spack.

## Install spack

```bash
git clone https://github.com/spack/spack
cd spack
git checkout releases/v0.12
# add to env var
 . share/spack/setup-env.sh
```

## Commands

* `spack list`
* `spack add <pack>`: to be installed but not now
* `spack install <pack>`
* `spack versions <pack>`: see available versions of pack
* `spack find`: look for installed packs, `--debs` for detail dependents
* `spack uninstall <pack>`: `-R` remove dependents
* `spack env create <proj>`: virt spack env
* `spack extention <pack>`: for example, numpy is the extension of python [doc](https://spack.readthedocs.io/en/latest/basic_usage.html#extensions-python-support)
* `spack config get `
* `spack edit <pack>`
* `spack module tcl `: module management, `refresh`, eg `spack module tcl refresh --delete-tree` to regenerate al module files.

###  Syntax for version

* `%` for compiler
* `@` for versions
* `^` for dependency packages
* cppflags, cflags, cxxflags, fflags, ldflags, ldlibs are also accepted as parameters of install
* `/` hash

## Configuration

* Path: `/etc/spack`, `~/.spack`
* extended yaml: `::` for replace while `:` for update
* `spec` in compiler dict is the name used by `spack install pack @spec`
* `flags` in compiler dict, `environment` for env vars
* `packages`.`providers`: default implementation of some proto, like mpi

## Integration of modules

Using lsmod.

`spack install lmod`

` . $(spack location -i lmod)/lmod/lmod/init/bash`

`. share/spack/setup-env.sh`

then you can use `spack load` and module system. Somehow spack system require python2.

Default path of module files of installed softwares: `$PREFIX/share/spack/modules/`

## Customize repo and package file recipe

[doc](https://spack.readthedocs.io/en/latest/tutorial_packaging.html) **recommended for careful reading**

`spack repo add $SPACK_ROOT/var/spack/repos/tutorial/`

`spack create` to create package

`spack edit <pack>`: change the package recipe py

`spack info <pack>`: some info on the package

spec obj to access internal info

by this mechanism, basically you can put all software under control of spack

## Misc

### interaction

* singularity
* docker

### Python

After spack installing python, to get other python packages, there are two approaches.

The first one in to just install packages provided by `spack extensions python`, thought the packages are limited compared to pypi repos, but you can customize them using variants and compiling them as you like. To use them, remember loading them by `spack load py-numpy` before running python.

For other packages, one can install by spack local pip. First you should `spack install py-pip ^python@ver`. Then to use this pip, you should load `py-pip` AND `py-setuptools`. The second one is necessary for pip to work. Then `pip install` can work as expected and all packages are in spack local site-packages directory, which can be used directly by spack python without load (on contrast of the packages managed in the first approach).

**Updates:** Just use intel python and intel conda enviroment as the standard workflow.

### Intel

See [doc](https://spack.readthedocs.io/en/latest/build_systems/intelpackage.html) for workflow to manage intel family, especially parallel studio.

A demo for compilers of intel (see [here](https://spack.readthedocs.io/en/latest/getting_started.html#compiler-config))

```bash
compilers:
- compiler:
    modules: [gcc-4.9.3, intel-15.0.24]
    operating_system: centos7
    paths:
      cc: /opt/intel-15.0.24/bin/icc-15.0.24-beta
      cxx: /opt/intel-15.0.24/bin/icpc-15.0.24-beta
      f77: /opt/intel-15.0.24/bin/ifort-15.0.24-beta
      fc: /opt/intel-15.0.24/bin/ifort-15.0.24-beta
    spec: intel@15.0.24.4.9.3
```

A demo for packages of intel

```bash
packages:
  all:
  providers:
    mpi:       [intel-parallel-studio+mpi]
    # Note: +mpi vs. +mkl
    blas:      [intel-parallel-studio+mkl]
    lapack:    [intel-parallel-studio+mkl]
    scalapack: [intel-parallel-studio+mkl]
    
  intel-parallel-studio:
    paths:
      intel-parallel-studio@cluster.2018.2.199 +mkl+mpi+ipp+tbb+daal: /opt/intel
      intel-parallel-studio@cluster.2018.3.222 +mkl+mpi+ipp+tbb+daal: /opt/intel
    buildable: False
```

Further summary: better also add intel-mpi and intel-mkl interface as external packages. intel-parallel-studio will automatically activate intel python. The PS1 is still missing as the activation script coming with the vendor (still dont know why this line exist). Add `tcl:intel-parallel-studio:filter: environment_blacklist:: [PS1]` to avoid this change in loading.

## Issues with specific packages

* Intel family [doc](https://spack.readthedocs.io/en/latest/build_systems/intelpackage.html), better to also set up intel-mpi and intel-mkl for flexibility
* Possible issue for some packages in air gapped computers. [issue](https://github.com/spack/spack/issues/7848). Solution: `git config --global url.https://github.com/.insteadOf git://github.com/`
* virtual management on external non-existing mathematica module
* jdk: spack seems to have issue for installing jdk for certain old versions. Dont want to comment on Oracle...
* m4: install issue with non gcc compiler: [issue](https://github.com/spack/spack/issues/4883), workaround: always use `^m4%gcc` in spack syntax
* armadillo by default detect whether you have installed mkl and firstly link to it, it has nothing to do with whether intel-mkl is a dependent of spack syntax or not. Though mkl is default, armadillo installation fails somehow when provided intel-parallel-studio as mkl.
* To use mkl, it is better to stay with icc instead of gcc, which can make the world simpler.
* numactl seems to be only supported by gcc
* an seemingly promising spec for slepc install: `spack install -v slepc ~arpack ^petsc+mpi~int64 ^intel-parallel-studio`, but the installation always stop in pestc configure part with mpi check. Actually lead to zombie process.
* PETSC install should load intel-parallel-studio first? and external mumps not works well. Actually that should be the case? since with-mpms option wants a tar instead of an installed path of mumps. A successful spec for petsc: ` spack install -v petsc@3.11.0  ^intel-parallel-studio`, `spack  install -v  slepc ~arpack ^intel-parallel-studio ^petsc@3.11.0`. But no modules auto generated for this two packages, dont know why. The pair of softwares cannot be used, there is segmentfaut at runtime. 
* singularity: after install, you need to run a script in its bin manually to make singularity work with nonsudo user. `sudo bash bin/spack_perms_fix.sh` in singularity install path.