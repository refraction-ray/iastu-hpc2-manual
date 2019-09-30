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

*A more specific step by step example for using jupyter on compute nodes:*

Say you want to run your jupyter notebook on c9 and access it by your laptop.

* SSH log into hpc2 master, and `salloc -p gpu -w c9` to allocate c9.

* `ssh c9` from master.

* In c9, `tmux` and prepare the conda env as well as `jupyter notebook --ip="0.0.0.0"`, say the allocated port is 8888.

* `ctrl-B d` to detach tmux and `ctrl-d` to log out c9 (remember cancel the allocate job after finishing using jupyter).

* In your own laptop terminal, use `ssh -N -L 9000:c9:8888 ias`, then you can visit jupyter in your own laptop broswer as `http://127.0.0.1:9000`. Note ias here is register in your `~/.ssh/config` on your laptop, something as 

  ```bash
  # this file is in ~/.ssh/config on your laptop
  Host ias
      Hostname <hpc2 ip addr>
  	User <user name @ hpc2>
  	Port <hpc2 server port>
  ```

## sbatch

The use of sbatch script for python is similar for usual scripts, just use `python some.py` is enough (and remember `source /etc/spack-load` and prepare the environment).

## Python Packages

### Tensorflow

See [this section](/users/softwares/tensorflow.md)