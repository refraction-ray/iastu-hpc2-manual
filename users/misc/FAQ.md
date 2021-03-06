# Frequently asked questions

[TOC]

### Why I cannot open jupyter in browser?

You should always visit jupyter by **inner ip** of the cluster (go to `http://<inner-ip>:<jupyter-port>` in local browser). The outer ip accessible from the whole Internet is only reserved for ssh related connection (i.e. only ssh port is open to forward to the real master node). The downside of using inner ip to access jupyter web is the access range is limited in campus network.

If you are trying connecting to jupyter from the Internet, there are two choices: 1. using VPN provided by Tsinghua in your local laptop and then visit inner ip. 2. Using shadowsocks service the cluster provided which grants you access to the network at IASTU from the Internet. For ss ip, port, password, and encryption method, please contact administrator directly.

### Why mathematica is not activated?

Mathematica is not activated per user basis by default. If you want to utilize mathematica in the cluster, please directly contact the administrator for a quick activation.

### I activated mathematica by myself, but why mathematica is still not usable on compute nodes?

Mathematica activation is independent on each server. So don't try to activate mathematica by yourself. You still cannot use mathematica on compute nodes even you have attempted to activate mathematica on login node. Therefore, if you have the need to use mathematica, just contact the administrator from the begining, no DIY is needed.

### Why my job on master node has been killed?

We have no guarantee on jobs running on master node. No intensive job should be executed on master node. The job will be killed anytime if the normal workload of login node is affected. Therefore, as normal user, please always try to incoprate your workflow with slurm, and submit your jobs to compute nodes.

### Is there internet connection on the cluster?

Yes, in general, you could connect to the Internet on all nodes including compute nodes. 

It is worth noting that the Internet accessibility is enabled by predefined http_proxy. Therefore, softwares with no support on http_proxy or softwares requiring special config on http_proxy more than simple enviroment variables may fail to connect to the Internet on the cluster unless further fine tuning and some workaround setups.

If you indeed have the need to access the Internet from the cluster and the corresponding software fails to work, please contact the adminstrator for help and further investigation.

### Why I have used up my disk space?

Use `quota` to check your disk quota on home, in general, it is as small as several dozen of GBs. So never put large datafile and outputs in your home folder. Instead, put these large files into /DATA dir. Please refer to [storage](/users/basics/storage.md) documentation carefully before using the cluster.

### How can I check which file occupied too much of my space?

To avoid storage in home being used up and delete unecessary large files, find them by

`find /home/<user> -type f -size +500M`.

### Why my task is way more slower than expected

The most possible reason is memory consumptions. Only 128G memory in compute nodes and more memory need would go to swap, which is unaccepteable for most of the softwares and tasks. So be sure ssh to the nodes you are running jobs and check the memory usage by `free`.

If your job requires lots of memory, it would be better to allocate the full node for computation no matter how many cpu cores are actually utilized, in case the memory is used up by others. `#SBATCH --exclusive`

### Why I cannot ssh to compute nodes

By default the compute nodes can not be sshed unless you have some job on it. You can first `salloc -w c[n]` and then ssh into it.

If you are asked about password, it is highly possible that you have messed up things in `~/.ssh`. Try generate a new pair of ssh keys by `ssh-keygen` and copy the id_rsa_pub content to add it in authorized_keys file in the same folder.

### import numpy cause segment fault in c10-14

Somehow the Ubuntu version is slightly different in new installed nodes c10-14. You may try create a new conda env with python3.8 and the following version of numpy ``conda install -c conda-forge numpy=1.19.4``