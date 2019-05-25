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

### ansible

Please see the ansible playbooks in HPC2.

### spack

*into ansible workflow*

Just see the practice on [spack](../toolchains/spack.md). Combine spack with lmod to provide consitent interface on package management.

### intel parallel studio XE

Since the cluster is in an air-gapped enviroment, and the rpm downloading seems not work well through http and ftp proxy, so one can directly download all rpms in another computer which has web connections by the online installer provided by intel and rsync the final tar to the cluster to finish the installing. And I prefer the intelpython as the favored version of python since it is very fast.

Remember to include cluster helper for MKL library which is omitted by default.

### python

*into ansible workflow and intel parallel studio installation*

prefered way: intel python+spack pip. `spack load intel-parallel-studio`, `spack load py-setuptools`, `spack load py-pip`.

Never use admin account's global pip. Reason: the package would be installed in ~/.pip. However, if later spack-pip install some packages, the dependece is automatically used if it is already in ~/.pip. But this folder is not accessible by other user which may lead to a chaos on python packages. However, for normal user, global pip3 is the recommended way to download packages.

### mathematica

Installed by the bash script on /opt/mathematica/verno. And the script is in the bin subdirectory of the above path. Add it as a package in spack, and `spack load mathematica` to use it.

**Possible issue:** The activation need to be carried out per user per node basis. Maybe have a look at MathML in the future.

### singularity

`spack install singularity`

### ganglia

*into ansible workflow*

`apt-get install ganglia-monitor ganglia-monitor-python gmetad ganglia-webfrontend`

ganglia-monitor is client side gmond. 

On client, only `ganglia-monitor` and `ganglia-monitor-python` should be installed. The second one is necessary as modules to watch spec of nodes.

Please reference [this post](https://hostpresto.com/community/tutorials/how-to-install-and-configure-ganglia-monitor-on-ubuntu-16-04/) on the change of configuration files.

The workflow of ganglia configuration and installation have been merged to ansible playbooks.

gmetric can be customized to report certain spec. See temperature example [here](http://cutler.io/2011/11/lm-sensors-on-ganglia/).

### quota

See reference on ubuntu 18.04 quota command: [digital ocean](https://www.digitalocean.com/community/tutorials/how-to-set-filesystem-quotas-on-ubuntu-18-04), it is very well write up.

`sudo apt install quota`

*need further experiments on VM cluster first before apply it, always be carful for disk stuff*

See the VM corresponding part for operations.

Not included in ansible due to flexibility consideration.

### ulimit

*merged into ansible workflow*

user level vs shell level: [ref](https://www.ibm.com/developerworks/cn/linux/l-cn-ulimit/index.html).

hard limit can only be changed by root and soft limit is something that anyone can change, but firstly, you need to change it.

ulimit config file in `/etc/security/limit.d`, see [demo](https://access.redhat.com/solutions/61334)

check by `ulimit -a` for individual users.

## some benchmarks

### network

* iperf, the master to compuation node bandwidth is around 940Mbit/s, which is near to the limit of the Gigabit nic.

### IO

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
* `sacctmgr add user <name> account=n3` to make user available to slurm. (already merged into ansible workflow)
* `sudo setquota -u <name> 60G 80G 0 0 /`