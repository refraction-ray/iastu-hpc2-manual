We strongly suggest all users use intelpython for better speed and builtin conda enviroment. Just `spack load intel-parallel-studio %intel`.

## Pip

We don't recommand to use pip in the bare enviroment. instead, you should use conda to manage your python workflow as below.

## Conda

You can actually manage your own python environment and install python packages as you like as follows which conda enviroment is utilized.

```bash
$ spack load intel-parallel-studio % intel
$ conda create -n <env_name> <packages_list>
$ conda init bash  # after this, intel python is by default when log in
# you may need a relogin to proceed correctly
(base)$ conda activate test 
(test)$ conda deactivate
(base)$
```

The python3 intepreter in this conda env is still provided by Intel.

One can install packages in conda env by `conda install --file requirements.txt`.

For packages only available to pip, please refer to [this article](https://www.anaconda.com/using-pip-in-a-conda-environment/) on possible conflict issues and solution workflows.

### Advanced usage on conda

* Conda channels explained: [so](https://stackoverflow.com/questions/42309333/explanation-of-different-conda-channels). Better use default channel, where intel builtin packages are installed by default.
* Fix package to certain version in conda enviroment: [so](https://stackoverflow.com/questions/48726621/is-it-possible-to-lock-versions-of-packages-in-anaconda). Add a requirement style like file as `~/.conda/envs/test/conda-meta/pinned`.

## Jupyter notebooks

```bash
$ spack load intel-parallel-studio %intel`
$ tmux
$ jupyter notebook --ip=`hostname` --no-browser
### ctrl-b d to leave
### after using
$ tmux attach
### ctrl-c to kill jupyter
```
General way to set up tunnels to compute nodes in your local desktop (MacOS or Linux)

```bash
$ ssh -N -L <local_port>:<remote_hostname>:<remote_port> <ssh_user>@<ssh_server_ip>
```

Such an ssh tunnel can make us visit both jupyter server in master node and in compute nodes.

## sbatch

The use of sbatch script for python is similar for usual scripts, just use `python some.py` is enough (and remember `source /etc/spack-load` and prepare the environment).

## Python Packages

### Tensorflow

If you want to use tensorflow-gpu in python, just add `/home/ubuntu/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.4.0/python-3.6.5-63x2grpokc4ax6mgyfilhyjnm5ersc3w/lib/python3.6/site-packages` this path into `sys.path`. And this path is auto included for default python base enviroment provided by intel. It is worth noting that this tensorflow is not compatible with CPU-only machines. To try tf on cpu-only machines, you can create your own conda env, and `conda install tensorflow`.

It is highly recommended that you add the following lines before you import tensorflow with gpu backend, unless you know exactly what you are attempting (without this line you will eat up all GPU memories).

```python
import os
os.environ['TF_FORCE_GPU_ALLOW_GROWTH'] = "true"
```

To sum up, the prepration lines for using gpu tensorflow would be

```python
import sys
sys.path.append("/home/ubuntu/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.4.0/python-3.6.5-63x2grpokc4ax6mgyfilhyjnm5ersc3w/lib/python3.6/site-packages")
import os
os.environ['TF_FORCE_GPU_ALLOW_GROWTH'] = "true"
import numpy as np
import tensorflow as tf
tf.enable_eager_execution()
```

