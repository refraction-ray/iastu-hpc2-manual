## Jax on cpu

To utilize cpu version of jax is pretty easy, just 

```python
pip install -U jaxlib
pip install -U jax
```

is ok. Remember using ``pip`` provided in your conda env to avoid package clashes and other nasty stuff.

Jax has many sharp bits you need to pay attention to, like how to enable 64bit floats and how to utilize random number generators, please refer to jax documentation for these details.

## Jax on GPU

To enable jax on gpu is another story, it is somehow involved and error prone, but I will elaborate the exact way that you can enjoy gpu version of jax in our HPC below.

### TL;DR

```python3
## installation of gpu jax
$ conda create -n jaxgpu python pip
$ conda activate jaxgpu
$ PYTHON_VERSION=cp36
$ CUDA_VERSION=cuda101
$ PLATFORM=linux_x86_64
$ BASE_URL='https://storage.googleapis.com/jax-releases'
$ pip install --upgrade $BASE_URL/$CUDA_VERSION/jaxlib-0.1.47-$PYTHON_VERSION-none-$PLATFORM.whl
$ pip install --upgrade jax  # install jax

## using of gpu jax
$ conda activate jaxgpu
$ spack load cuda@10.1
$ spack load cudnn@7.6
$ export XLA_FLAGS=--xla_gpu_cuda_data_dir=/home/ubuntu/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.4.0/cuda-10.1.243-ohxd3xdnjd3ayvjdi2ku7dtam643l7vd
$ python test.py
```

### Details

Nivdia is notorious for various versions of GPU drivers, cuda, and cudnn and how they are conflict or compatible with each other. See [this page](https://docs.nvidia.com/deeplearning/sdk/cudnn-support-matrix/index.html) for some compatible trios. But the list is definitely not the whole story since many driver versions are omitted and obviously some trios not in the list also works. So…, it is a headache anyway.

In our HPC setup, GPU driver in master is 418 and in c9 is 430. And as I tests with jaxlib, cuda 10.1+ cudnn 7.6 is a workable combination for GPUs in both machines. Note cuda 10.0+ cudnn 7.5 fails when ``import jax`` with the error complaining that ``ImportError: .../jaxlib/xla_extension.so: symbol cudnnSetCTCLossDescriptorEx version libcudnn.so.7 not defined in file libcudnn.so.7 with link time reference``. See [this issue](https://github.com/google/jax/issues/2494). Such error are often indicating that cuda and cudnn combinations fail to interact with current GPU driver versions and XLA versions. It is always hard to figure out which exactly combinations of drivers, cuda, cudnn, and jaxlib works. I only give the solution in our HPC:  GPU driver version 418 and 430, jaxlib 0.1.47 + cuda 10.1.243 + cudnn 7.6.5.32 works. Other combinations? sorry, I dont know, try on your own risk.

Even after you have configured the right combination of cuda, cudnn (provided by spack) and jaxlib (which is installed via instruction [here](https://github.com/google/jax#installation)), you may meet another error when actually running some code.

```python
# test demo for gpu jax, test.py
import jax
key = jax.random.PRNGKey(42)
print(jax.random.normal(key,[10]))
```

The error is ``RuntimeError: Internal: libdevice not found at ./libdevice.10.bc``, see details in [this issue](https://github.com/google/jax/issues/989). The origin of this error is that XLA design is not so smart, it cannot find cuda installation beyond default path when not specified. So one need to ``export XLA_FLAGS=--xla_gpu_cuda_data_dir=/path/to/cuda`` before actually running jax. In our case, the exact command is ``export XLA_FLAGS=--xla_gpu_cuda_data_dir=/home/ubuntu/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.4.0/cuda-10.1.243-ohxd3xdnjd3ayvjdi2ku7dtam643l7vd``.