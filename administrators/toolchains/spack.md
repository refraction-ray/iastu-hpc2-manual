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
* `spack install <pack>`
* `spack versions <pack>`: see available versions of pack
* `spack find`: look for installed packs
* `spack uninstall <pack>`: `-R` remove dependents
* `spack env create <proj>`: virt spack env

### Syntax for version

* `%` for compiler
* `@` for versions
* `^` for dependency packages
* cppflags, cflags, cxxflags, fflags, ldflags, ldlibs are also accepted as parameters of install

## Configuration

* Path: `/etc/spack`, `~/.spack`
* extended yaml: `::` for replace while `:` for update
* `spec` in compiler dict is the name used by `spack install pack @spec`
* `flags` in compiler dict, `environment` for env vars
* `packages`.`providers`: default implementation of some proto, like mpi

## Integration of modules

Using lsmod.

`spack install lsmod`

` . $(spack location -i lmod)/lmod/lmod/init/bash`

`. share/spack/setup-env.sh`

then you can use `spack load` and module system. Somehow spack system require python2.

## Issues with specific packages

* Intel family [doc](https://spack.readthedocs.io/en/latest/build_systems/intelpackage.html)