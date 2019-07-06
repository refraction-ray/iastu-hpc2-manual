## hardware

### switch

port 23 24 are changed into VLAN2 for S1720-28GWR-4P.

### AP

The reseverd ip is `192.168.1.250/24`. The dhcp is turned off and it sit in AP mode. When connecting to it, the ip is assigned by master node DHCP sevice in 48 subnet.

### master server

LAN upper RJ45 port 10/24 and WAN lower RJ45 port 44/24.

maste node is currently set as static ip in WAN.

Harddisk(? position may not be accurate now): left down for 2.5 SSD, right down for sdb, old 2T HDD, left up for sdc, new 3.5 2T HDD.

### computation server

The leftmost RJ45 port is utilized. (not the one for iDRAC) The nic name is eno1 in ubuntu OS.

### outdated server

centos 6.6, name as d1. For operations on this server, please refer [this section](./oldtimes.md).

**Note:** d1 is not ready to open to users.

## software

### basic on master

First use fdisk make one partition on each disk of sdb and sdc, and then use `mkfs.ext4 /dev/sdb1` to format the two disks. Mount two 2T hdds `/dev/sdb1` and `/dev/sdc1` to /DATA and /BACKUP, where /DATA has similar permission with tmp and shared across nfs. Namely, `chmod a+w /DATA`, `chmod a+t /DATA`. Meanwhile, /BACKUP is only written by root. There are several backup crontab tasks by root managed by `rsync -az` from /home/ubuntu and /opt to /BACKUP. Besides, the config file `/etc/fstab` is also configured such that /DATA and /BACKUP can be mounted automatically when reboot.

```bash
## add two following lines in /etc/fstab
/dev/sdb1  /DATA ext4 defaults 0 0
/dev/sdc1  /BACKUP ext4 defaults 0 0
```

```bash
$ sudo crontab -l

20 2 * * * /usr/bin/rsync -avz /home/ubuntu/ /BACKUP/ubuntu/
20 5 * * 2 /usr/bin/rsync -avz /opt/ /BACKUP/opt/
```

And add old machines (currently d1) into ~/.ssh/config.

The backup crontab and fstab mount config have not included into ansible workflow due to flexibility consideration.

In terms of network and nfs, ntp,apt setups, please see relevant section in [Virtual Machine](./VM.md) part.

### ansible

`sudo apt install ansible` on master node.

Please see the ansible playbooks in HPC2. The playbooks went opensource, see [here](https://github.com/refraction-ray/hpc-build-ansible-playbooks).

### spack

*into ansible workflow*

Just see the practice on [spack](../toolchains/spack.md). Combine spack with lmod to provide consitent interface on package management.

#### some spack things to note

* spack install rclone, and there is a go folder on the home directory, outside the spack folder!! Seems because `GOPATH` is by default instead of set within spack folder. See [this blog](https://blog.csdn.net/zwqjoy/article/details/78788918) for more info on go project and package organization in fs level. **Solved** by [this commit](https://github.com/tldahlgren/spack/commit/d2e22d95e7b50a9e94b4ebc0f9bc25fb61ca3cdc).

### intel parallel studio XE

Since the cluster is in an air-gapped enviroment, and the rpm downloading seems not work well through http and ftp proxy, so one can directly download all rpms in another computer which has web connections by the online installer provided by intel and rsync the final tar to the cluster to finish the installing. And I prefer the intelpython as the favored version of python since it is very fast.

**Warn:** Remember to include cluster helper for MKL library which is omitted by default.

### PETSC+SLEPC

configure for petsc: dont set compiler when set with-mpi-dir 

Installed externally to spack.

`export PETSC_DIR= &&export PETSC_ARCH=`

```
./configure  --download-superlu_dist --download-mumps --download-hypre --download-scalapack --download-metis --with-blaslapack-dir=/opt/intel/mkl CFLAGS=-fPIC CXXFLAGS=-fPIC FFLAGS=-fPIC FCFLAGS=-fPIC F90FLAGS=-fPIC F77FLAGS=-fPIC --with-debugging=0 --with-mpi-dir=/opt/intel/impi/2019.3.199/intel64 --with-cxx-dialect=C++11
```

for sinvert project, `cmake -DMACHINE=linux -DCMAKE_C_COMPILER=icc -DCMAKE_CXX_COMPILER=icpc ../src`. It turns out as a cmake error where default openmpi has been utilized.

### python

*into ansible workflow and intel parallel studio installation, always use intel python and its conda for users*

prefered way: ~~intel python+spack pip~~. `spack load intel-parallel-studio`, `spack load py-setuptools`, `spack load py-pip`. **Intel python+conda create enviroment**.

Never use admin account's global pip. Reason: the package would be installed in ~/.pip. However, if later spack-pip install some packages, the dependece is automatically used if it is already in ~/.pip. But this folder is not accessible by other user which may lead to a chaos on python packages. However, for normal user, global pip3 is the recommended way to download packages.

### jupyter

Use intel python and pip as root, `pip install jupyter ipyparallel jupyter_contrib_nbextensions`. Somehow the cluster tab works after several trials. Dont know exact solution though.

### mathematica

Installed by the bash script on /opt/mathematica/verno. And the script is in the bin subdirectory of the above path. Add it as a package in spack override repo, and `spack load mathematica` to use it.

**Possible issue:** The activation need to be carried out per user per node basis. Maybe have a look at MathML in the future.

To utilize remote kernel launching with a better interface, add tunnel.m at Kernels/packages. Besides, one should also add tunnel.sh and tunnel_sub.sh in their home directory `~/.Mathematica/FrontEnd`.

One liner for user to be accessible `ansible -i /home/ubuntu/hpc/hosts all -m command -a "/usr/bin/python3 /home/ubuntu/softwares/mmaact/automma.py '/opt/mathematica/11.0.1/bin/math'" --become-user=<user> -Kvv --become`.

### matlab

mount iso to a dir, and use `ssh -X` to install by X11 forwarding. Remember umount and mount iso2.

It is worth noting, that matlab installer supports -X for forwarding. While for matlab itself, only `ssh -Y` works for remote desktop scheme.

### singularity

`spack install singularity`, remember to check `spack edit singularity`, there is an after install warn prompt asking you run a script which would change the permission of some files with s bit. It is crucial for singularity run by normal users.

### spark

`spack install` as described in VM part. 

TODO: add `SPARK_HOME` to spark module file.

*only test on user conda env*: `pip install findspark`, to call spark context in python more smoothly.

### dask

*only test on user conda env*: `conda install dask`. Fix tornado version in a pinned file at conda-meta dir for conda virtual enviroment. Otherwise, dask would upgrade tornado which break down jupyter notebook! See [this issue](https://github.com/jupyter/notebook/issues/3595) for jupyter notebook breakdown.

### ganglia

*into ansible workflow*

`apt-get install ganglia-monitor ganglia-monitor-python gmetad ganglia-webfrontend`

ganglia-monitor is client side gmond. 

On client, only `ganglia-monitor` and `ganglia-monitor-python` should be installed. The second one is necessary as modules to watch spec of nodes.

Please reference [this post](https://hostpresto.com/community/tutorials/how-to-install-and-configure-ganglia-monitor-on-ubuntu-16-04/) on the change of configuration files.

The workflow of ganglia configuration and installation have been merged to ansible playbooks.

gmetric can be customized to report certain spec. See temperature example [here](http://cutler.io/2011/11/lm-sensors-on-ganglia/).

Apache password protected sites: [digitalocean](https://www.digitalocean.com/community/tutorials/how-to-set-up-password-authentication-with-apache-on-ubuntu-14-04)

### ELK

*into ansible workflow*

* add elastic repo and key
* apt install elasticsearch
* apt install kibana and configure nginx reverse proxy
* apt install logstash and configure the pipe, note the ip binding of beat input configure (there are `""` for string ip)
* apt install filebeat (on all nodes)
* `sudo filebeat setup --template -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["localhost:9200"]'`
* `sudo filebeat setup -e -E output.logstash.enabled=false -E output.elasticsearch.hosts=['localhost:9200'] -E setup.kibana.host=localhost:5601`
* work test `curl -X GET "localhost:9200/_cat/indices?v"`
* disable logstash ssl (seems to be disable by default)
* edit the index of logstash output (add beat.hostname in index name)

**Misc note:**

* For debug test on es, curl will go proxy!!
* no specified JAVA_HOME warning in es service log doesn't matter
* logstash config [intro](https://www.elastic.co/guide/en/logstash/current/advanced-pipeline.html#configuring-geoip-plugin), grok [official guide](https://www.elastic.co/guide/en/logstash/7.1/plugins-filters-grok.html)
* actually it is ok, but the log from compute node is just too small compared to master…. It is not an issue due to ELK stack, but issue of non uptodate syslog.
* [timezone issue of syslog](https://stackoverflow.com/questions/22853026/ubuntu-change-timezone-to-utc-does-not-affect-the-time-of-syslog): every damon can see the timezone issue only solved by service restart! **case solved**

### quota

See reference on ubuntu 18.04 quota command: [digital ocean](https://www.digitalocean.com/community/tutorials/how-to-set-filesystem-quotas-on-ubuntu-18-04), it is very well write up.

`sudo apt install quota`

*need further experiments on VM cluster first before apply it, always be carful for disk stuff*

See the VM corresponding part for operations.

Not included in ansible due to flexibility consideration. Only use in master node.

### ulimit

*merged into ansible workflow*

user level vs shell level: [ref](https://www.ibm.com/developerworks/cn/linux/l-cn-ulimit/index.html).

hard limit can only be changed by root and soft limit is something that anyone can change, but firstly, you need to change it.

ulimit config file in `/etc/security/limit.d`, see [demo](https://access.redhat.com/solutions/61334)

check by `ulimit -a` for individual users.

### numa

`apt install numactl`

```bash
$ numactl -H
available: 2 nodes (0-1)
node 0 cpus: 0 1 2 3 4 5 6 7 8 9 10 11 12 13 28 29 30 31 32 33 34 35 36 37 38 39 40 41
node 0 size: 128601 MB
node 0 free: 14004 MB
node 1 cpus: 14 15 16 17 18 19 20 21 22 23 24 25 26 27 42 43 44 45 46 47 48 49 50 51 52 53 54 55
node 1 size: 128996 MB
node 1 free: 27058 MB
node distances:
node   0   1
  0:  10  21
  1:  21  10
```

`apt install hwloc`

### cgroup

`sudo apt install cgroup-tools`

```bash
# /etc/cgconfig.conf
group service {
  cpuset {
    cpuset.cpus="0-13";
    cpuset.mems=0;
  }
}
group user {
  cpuset {
    cpuset.cpus="14-27,42-55";
    cpuset.mems=1;
  }
}

# /etc/cgrules.conf
elasticsearch: cpuset service/
kibana:        cpuset service/
logstash:      cpuset service/
<user>:        cpuset user/

# make new cgroup policy come to live
sudo cgconfigparser -l /etc/cgconfig.conf
sudo cgrulesengd
```

### tinc

`sudo apt install tinc`

combine tinc vpn with http proxy, such that http proxy ip is not public to everyone. Make http proxy only listen to the tinc ip interface.

```bash
## /etc/netname/tinc.conf
Name = wst
Interface = tinc
ConnectTo = of
ConnectTo = smart
AddressFamily = any
GraphDumpFile = /var/log/tinc-peer.log

## /etc/netname/tinc-up
#! /bin/bash
ip link set $INTERFACE up
ip addr add {{ tinc_ip }}/16 dev $INTERFACE

## /etc/netname/tinc-down
#! /bin/bash
ip link set $INTERFACE down

## /etc/netname/hosts/wst
Subnet =  10.x.y.2/32

-----BEGIN RSA PUBLIC KEY-----
pubkey-wst
-----END RSA PUBLIC KEY-----

## /etc/netname/hosts/of
Subnet =  10.x.y.1/32
Address = public ip

-----BEGIN RSA PUBLIC KEY-----
pubkey-of
-----END RSA PUBLIC KEY-----
```

`tincd -n netname -K` to generate key pairs, and `tincd -n netname` to start the daemon. For debug usage, try `tincd -n netname -d5 -D`  for a foregroud d with verbose output. On each Tinc daemon debug window, quit the daemon by pressing `CTRL-\`.

`sudo iptables -t nat -I POSTROUTING 1 -o tinc -s 192.168.48.0/24 ! -d 192.168.48.0/24 -j SNAT --to-source 10.26.11.1` on master node, make compute nodes available without any modification on them. (this new SNAT line is hopefully also managed by ansible playbooks). `sudo iptables -t nat -nLv` check current iptables.

## some benchmarks

### network

* iperf, the master to compuation node bandwidth is around 940Mbit/s, which is near to the limit of the Gigabit nic.

### cpu

### memory

Available frequency is 2400, though the param is 2666, the speed is limited by CPU 5120.

### disk

* by brute force `dd if=/dev/zero of=/tmp/output bs=8k count=50k`
  * cn3 ssd: 1.4GB/s
  * cn3 write to home folder, which is share by NFS and stored in master's SSD, 89.1 MB/s
  * master ssd:  1.3 GB/s
  * master /DATA, sdb1: 1.3GB/s (? cannot understand), similar results for sdc in master, it is weird though.
  * dn1: sdb2 557MB/s, similar result for sdb, sda (under lvm): 471MB/s

## some scenarios

### add a new user

* Add user account and password info into ansible user playbook and run it.
* The password can be generated by `openssl rand -base64 32`, see [here](https://www.howtogeek.com/howto/30184/10-ways-to-generate-a-random-password-from-the-command-line/) for more approaches to generate random password.
* `sacctmgr add user <name> account=n3` to make user available to slurm. (already merged into ansible workflow)
* `sudo setquota -u <name> 60G 80G 0 0 /` (already merged into ansible workflow)
* mkdir on /DATA path. (already merged into ansible workflow)
* Probably add `<user> cpu usersoft/` line in `/etc/cgrules.conf`. And `sudo cgrulesengd`. (edit in roles/cgroup/files and run ansible cgroup role instead) **Unincluded**
* Activate mathematica by ansible one-liner. **Unincluded**
* Add backup crontab for user home directory. **Unincluded**

### reboot

* It is highly suggested that all ansible playbooks to be executed once reboot.
* config cgroup as `sudo cgconfigparser -l /etc/cgconfig.conf && sudo cgrulesengd`.
* start tinc vpn (maybe auto start as I observed).

### summary on works beyond ansible workflow

All in master nodes, keep the bottom line that all tasks on compute node should merged into ansible workflow.

* hard disk mount and fstab configure (one time forever, required before **basic roles**, actually can easily merged into basic role)
* Possible nvidia drivers install and reboot if GPU is available. (one time forever) Cuda and cudnn can be managed by spack.
* quota initial configure (one time forever, required before **user roles)**
* intel parallel studio install (one time forever) (no need to install before any roles, possible issue for python path maybe in **python roles**)
* mathematica install and add virtual mathematica packages in spack (one time forever) (no need to install before any roles)
* backup crontabs (one time forever? maybe find some more advanced tools) (no need to configure before any roles)
* python packages install and jupyter configure (continuing work) (no need to install before any roles)
* spack packages install by specs and spack env maintenance (continuing work) (no need to install before any roles)
* sacctmgr cluster, qos and account add (continuing work for advanced scenario, minimum setup required before **user roles** after slurm roles)
* two line of commands to final set up ELK stack on master (should find some more elegant way in the future)
* tinc vpn set up on master node

### install softwares or libraries beyond spack

Installation path: large size commercial softwares: `/opt/softwarename/version/`. Open source library from source: `/home/ubuntu/softwares/softwarename`, in this dir, you may have some name+ver dir for installed versions of softwares and some dir name ended with src as source files. Hopefully, one should put a self-contained information file in each software dir. The name convention is `softwarename.info`. The content includes what each dir within for, what the notes and warnings for this software configuration and installation, most importantly, the install process details for each installed version, such as options for `./configure` and so on. One may even want to capture all stdout for configure and make on each installed version dir with name `configure.out`, `make.out` and so on. To record this stdout more easily, using `script cmake.out` and then run `cmake`, remember ctrl-d or exit to stop the recording. See [more](https://www.geeksforgeeks.org/script-command-in-linux-with-examples/) on script command in linux.

Finally, you may want to include such libraries under the control of spack. This usually involves two steps. If such library already has a position in spack repo, then you only need to add the external path for this package in spack config packages.yaml. If there is some more further fine tuning on module files to load it, you need further hacking spack config modules.yaml. (All these config change should go under ansible workflow). Besides, if the softwares is not registered in spack, then you need first add a package.py in repo as a placeholder. Something as below is enough.

```python
# placeholder for external mma

from spack import *

class Mathematica(Package):

    homepage = "https://www.wolfram.com/mathematica/"

    version('11.0')
```

The insipration of standard workflow on software installation is from [this post](http://www.admin-magazine.com/HPC/Articles/Warewulf-Cluster-Manager-Development-and-Run-Time/(language)/eng-US).