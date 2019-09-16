We strongly suggest all users use intelpython for better speed and builtin conda enviroment. Just `spack load intel-parallel-studio %intel`.

## Pip

We don't recommand to use pip in the bare enviroment. instead, you should use conda to manage your python workflow as below. When using pip, always check by `which pip`, make sure the pip is provided by some conda env (`/home/<user>/.conda/envs/<env-name>/bin/pip`) instead of local pip `/opt/intel/intelpython3/bin/pip` or `/usr/bin/pip`.

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

It is recommended that pip is included in the default `<package_list>` for conda env create.

One can install packages in conda env by `conda install --file requirements.txt`.

For packages only available to pip, please refer to [this article](https://www.anaconda.com/using-pip-in-a-conda-environment/) on possible conflict issues and solution workflows.

To use conda env in sbatch, one only need call python binary as `/home/<user>/.conda/envs/<conda-env>/bin/python`, this is enough for activate the conda env.

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

* preinstalled gpu tensorflow

If you want to use tensorflow-gpu in python, just add `/home/ubuntu/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.4.0/python-3.6.5-63x2grpokc4ax6mgyfilhyjnm5ersc3w/lib/python3.6/site-packages` this path into `sys.path`. And this path is auto included for default python base enviroment provided by intel. Also remember spack loading cuda and cudnn for gpu tf to work.

It is worth noting that this tensorflow is not compatible with CPU-only machines. To try tf on cpu-only machines, you can create your own conda env, and `conda install tensorflow`.

* GPU memory allocate

It is highly recommended that you add the following lines before you import tensorflow with gpu backend, unless you know exactly what you are attempting (without this line you will eat up all GPU memories).

```python
import os
os.environ['TF_FORCE_GPU_ALLOW_GROWTH'] = "true"
```

To sum up, the prepration lines in jupyter for using gpu tensorflow would be

```python
import sys
sys.path.append("/home/ubuntu/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.4.0/python-3.6.5-63x2grpokc4ax6mgyfilhyjnm5ersc3w/lib/python3.6/site-packages")
import os
os.environ['TF_FORCE_GPU_ALLOW_GROWTH'] = "true"
import numpy as np
import tensorflow as tf
tf.enable_eager_execution()
```

* verbose log output

In sbatch and py scirpt mode, a possible header for tensorflow cpu would be something like below

```python
import os
os.environ['TF_CPP_MIN_LOG_LEVEL'] = '3'  # or any {'0', '1', '2'}
os.environ['MKL_VERBOSE'] = ''
import numpy as np
import tensorflow as tf
tf.enable_eager_execution()
```

Note the enviroment variables controlling log level. Somehow, sbatch python task has very verbose ouput for mkl and tensorflow by default (these stuff will eat all of your disk space in output file!). One should turn off them for a small slurm.out file. See [this post](https://stackoverflow.com/questions/38073432/how-to-suppress-verbose-tensorflow-logging) for turning off tf logs.

* ulimit -u limit

Also note that tensorflow may need very large number of processes (actually threads, linux made no big difference on these two things, see [so](https://stackoverflow.com/questions/344203/maximum-number-of-threads-per-process-in-linux) for more info). So if one want to utilize tensorflow with multiprocessing for some naive parallel programming, it might be necessary to turn up the ulimit on max user process, which has soft limit ~~2048~~ by default (default value already changed to 8192) to avoid fork bomb. Namely run `ulimit -u 16384` on terminal (or in sbatch script for a slurm job), so that one can run more tf instances at the same time. `cat /proc/<pid>/limits` see whether the new ulimit is valid for pid program.

However even by doing this, you cannot re login to the shell, since the default ulimit isn't changed. You may need to add corresponding line into user's bash_profile, otherwise you may NEVER to login the cluster unless the heavy tf task is finished.