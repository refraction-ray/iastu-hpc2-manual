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