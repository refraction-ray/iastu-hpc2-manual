## TensorFlow versions

* preinstalled gpu tensorflow

*tf-gpu cannot be imported on cpu only machines though sharing the same python namespace*

If you want to use tensorflow-gpu in python, just add `/home/ubuntu/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.4.0/python-3.6.5-63x2grpokc4ax6mgyfilhyjnm5ersc3w/lib/python3.6/site-packages` this path into `sys.path`. And this path is auto included for default python base enviroment provided by intel. Also remember spack loading cuda and cudnn for gpu tf to work.

It is worth noting that this tensorflow is not compatible with CPU-only machines. To try tf on cpu-only machines, you can create your own conda env, and `conda install tensorflow`.

* pick GPU device

```python
os.environ["CUDA_DEVICE_ORDER"]="PCI_BUS_ID"   # see issue #152
os.environ["CUDA_VISIBLE_DEVICES"]="0"
```

[ref](https://stackoverflow.com/questions/37893755/tensorflow-set-cuda-visible-devices-within-jupyter)

## TensorFlow issues

* GPU memory allocate

*tf may eat up all your GPU memory*

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

*tf may eat up all your disks*

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

*tf may eat up all your processes*

Also note that tensorflow may need very large number of processes (actually threads, linux made no big difference on these two things, see [so](https://stackoverflow.com/questions/344203/maximum-number-of-threads-per-process-in-linux) for more info). So if one want to utilize tensorflow with multiprocessing for some naive parallel programming, it might be necessary to turn up the ulimit on max user process, which has soft limit ~~2048~~ by default (default value already changed to 8192) to avoid fork bomb. Namely run `ulimit -u 16384` on terminal (or in sbatch script for a slurm job), so that one can run more tf instances at the same time. `cat /proc/<pid>/limits` see whether the new ulimit is valid for pid program.

However even by doing this, you cannot re login to the shell, since the default ulimit isn't changed. You may need to add corresponding line into user's bash_profile, otherwise you may NEVER to login the cluster unless the heavy tf task is finished.

* memory leak in tf2.0

*tf may eat up all your memories*

Somehow, there is memory leak issue in tf2.0 nightly release. The same code has no problem when running on tf1.13. So be careful about tf2.0, not recommend to use it for now. If you insist attempt tf2.0, be super careful about the possible memory usage!

## Tensorboard

Preinstalled GPU version tf, tensorboard bin path is `/home/ubuntu/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.4.0/python-3.6.5-63x2grpokc4ax6mgyfilhyjnm5ersc3w/bin/tensorboard`.

To open a server remotely to view tensorboard, try `tensorboard --host 0.0.0.0 --logdir ./`. Then you can visit it by `http://<cluster_ip>:<tb_port>` from your laptop's broswer.

In the code side, if what you want is just the static graph, then one line `train_writer = tf.summary.FileWriter( './logs/train ', sess.graph)` is enough.