In this page, I will record all the operations in the VM environment as a pretest to deploy something on HPC2.

## KVM settings

I chose KVM scheme to do the vitrualization. Test host enviroment Ubuntu 16.04.

### VM create

Install relevant package `sudo apt-get install qemu-kvm libvirt-bin virtinst bridge-utils cpu-checker`. 

Check the installation and configuration is succeed by `kvm-ok`.

VM environment Ubuntu 18.04 LTS server.

`qemu-img create -f qcow2 master.img 5G`

*Note: one can also omit the qemu-img create command, but use `—disk size=5` as args in the following line. The default path of the images is `/var/lib/libvirt/images/`*.

`sudo virt-install --name=master --memory=2048 --vcpus=1 --cdrom=ubuntu-18.04.2-live-server-amd64.iso -f master.img --graphics vnc,listen=0.0.0.0,password=<key>`

Use screen sharing in mac to access the VNC desktop to finish the installation, the default port is 5900.

*Note: for multiple VM, if VNC port is not specified, the following port will be 5901 for the second one and so on.*

`sudo virsh list` to check the VM running now.

**VM delete:** `virsh destroy domain && virsh undefine domain` to drop the VM, but still one need to delete the image. Try `virsh vol-list default` and `virsh vol-delete --pool default name`. The pool is find by `virsh pool-list`.

**VM volume resize**: `sudo virsh vol-resize master.qcow2 --pool default 10G`, `sudo virsh vol-info master.qcow2 --pool default`. Never resize for a online VM, it is easy to corrupt the disk!!! For the resize operation on VM, see my blog [here](https://re-ra.xyz/KVM-%E7%A1%AC%E7%9B%98%E6%89%A9%E5%AE%B9/).

### Network configuration

To simulate the cluster network topology in VM clusters, see my blog [here](https://re-ra.xyz/KVM-%E8%99%9A%E6%8B%9F%E9%9B%86%E7%BE%A4%E7%BD%91%E7%BB%9C%E6%8B%93%E6%89%91%E8%AE%BE%E7%BD%AE/).

## Basic

### Python3 preparation

The operations below only on master VM. *Since ansible can be installed directly via apt, the python part might be moved later which controled by ansible* 

*The following workflows on python is somehow deprecated for our cluster real settings*

Final decision: change python under spack manage.

* apt source [change to tuna](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

* `sudo apt update && sudo apt install python3-pip`

* There is a pip3 upgrade issue on ubuntu18.04, the revert command is `sudo python3 -m pip uninstall pip`, see [here](https://stackoverflow.com/questions/49836676/error-after-upgrading-pip-cannot-import-name-main) for pip upgrade issue. 

* change the default path of dist package, create file as `/etc/pip.conf`, and add the following in the file

  ```bash
  [global]
  target=/opt/python3/dist-packages
  index-url=https://pypi.tuna.tsinghua.edu.cn/simple
  ```

  and then `sudo pip3 install` for every packages you want. `export PYTHONPATH=/opt/python3/dist-packages` for python3 to auto include it in sys.path.

* One can upgrade pip by `sudo pip3 install pip -U`, and in bashrc `alias pip3="python3 -m pip"`.

* After such settings, user cannot install package in their own home dir unless create a file `~/.pip/pip.conf` to overwrite the default installation path as 

  ```bash
  [global]
  target=/home/user/.local/lib/python3.6/site-packages
  ```

  It is worth noting that, after this user specific config, the installation of package is home dir even if we use sudo. Only `sudo su` to root can the installation dir change back to opt.

### NFS and NTP

*NTP part can be controled by ansible later in production*

* on master, `sudo apt install nfs-kernel-server`.
* config `/etc/exports`, note there should not be any space after comma. `/opt 192.168.48.0/24 (rw,sync)`.
* on node1, `sudo apt install nfs-common`, and `sudo showmount -e master`
* `sudo mount -t nfs master:/opt /opt`
* remember to export PYTHONPATH in this node, and use python3, the external package works.
* on master, `sudo install ntp ntpstat`, note there is a new toolset called timedatectl in ubuntu, be cautious.
* edit `/etc/ntp.conf`, `sudo service ntp restart`

```bash
restrict ntp.tuna.tsinghua.edu.cn
restrict 192.168.48.0 mask 255.255.255.0 nomodify
server ntp.tuna.tsinghua.edu.cn prefer
```

* ` ntpq -p` or `ntpstat` to check the current state of ntp service and upstream
* for node1, also config ntp service, but with server as master
* Caution: for NFS disconnect, you may experien hang on ls and df -H, try solution in [this post](https://blog.csdn.net/NULL_LLUN/article/details/80169107). Basically combination of umount and nfs service restart. [Reference](https://blog.csdn.net/lanranguidao/article/details/81484421)
* Possible issue on NFS: [gitlab post](https://gitlab.com/gitlab-org/gitlab-ce/issues/52017)

### SSH

* export home and mount for node1 as nfs
* `ssh-keygen -b 4096`, and vim `authorized_keys` to add the publick key line, chmod 600 for the key file. And now one can ssh hostname freely without password.

### Ansible

* on master, `sudo apt install ansible`

* ~~edit `/etc/ansible/hosts` for a temporary ping test `ansible all -m ping`~~ (Just follow an all in one roles package)

  ```bash
  [ln]
  master
  [cn]
  node1
  node2
  [all:vars]
  ansible_python_interpreter=/usr/bin/python3
  ```

  For the vars part, actually you must specify python3, otherwise python is consulted while it doesn't exist on ubuntu18.04 by default. See [this issue](https://github.com/ansible/ansible/issues/19605) for reference.

* create a standalone dir for ansible roles and config management.

* `ansible-playbook site.yml -vv --ask-become-pass`

* backup the ansible roles to other server: `rsync -avz -e "ssh -p port" hpc/ user@ip:/home/user/`

* parallel ad-hoc commands `ansible all -a "bash /blah/proxy.cfg"`

### Remote Desktop

* Edit sshd_config in server side by turning on X11Forwading option.
* check there is xauth binary in the server, `which xauth`
* For client, use `ssh -X`, or add `ForwardX11 yes ` in `.ssh/config`
* For mac client, install XQuatrz for X11 display support
* Such desktop forwarding can be even forwarded more than once, say from A ssh -X to B and from B ssh -X to C, then `xclock` in C will redirect the display of clock on A's screen.

### quota

* apt install quota 
* edit /etc/fstab, use usrquota, grpquota instead of defaults for root /. being sure to separate all options with a comma and no spaces.
* `sudo mount -o remount /` to remount root
* check the effect by `cat /proc/mounts | grep ' / '`
* `sudo quotacheck -ugm /`, This command creates the files `/aquota.user` and `/aquota.group`. 
* `sudo quotaon -v /` turn on quota
* `sudo setquota -u test 200M 220M 0 0 /` block s h node s h fs.
* `sudo repquota -s /` generate reports for user usage on the disk.

## slurm

### munge

come with slurm on ubuntu apt, change the file `/etc/munge/munge.key` to the same and restart service munge. Test see [here](https://github.com/dun/munge/wiki/Installation-Guide).

Error: munged: Error: Logfile is insecure: world-writable permissions without sticky bit. `chmod o+t /var`

### debug

`slurmd -Dcvv`

Somehow the service slurmctld and slurmd may get very messy, one may first `ps aux|grep slurm` and kill them totally by pid, and then try start the service again.

### integrate with mpi

Note intel-mpi is a little different to use! A direct `srun` doesn't work. See [here](https://slurm.schedmd.com/mpi_guide.html). Instead use sbatch script as follows

```bash
#!/bin/sh
export I_MPI_PROCESS_MANAGER=mpd
mpirun -n 3 ./a.out
```

For intel mpi, the path should be something like `./a.out` though for openmpi, `a.out` is enough.

Note the different implementation of PMI interface, which is the key for srun to work as expected. But it is ok srun itself doesn't work. We can combine sbatch and mpirun to achieve the same effect.

Note slurm can also interact with containers such as docker or sigularity in principle.

### account slurmdbd

* `sudo apt install slurmdbd mysql-server python-mysqldb`
* `sudo service mysql status`
* Ubuntu mysql has default root user with empty password, but can only log in by sudo
* `CREATE USER 'slurm'@'localhost' IDENTIFIED BY 'pw';`
* `GRANT ALL PRIVILEGES ON slurm_acct_db.* TO 'slurm'@'localhost';` in mysql
* possible mysql table missing issue: [issue](https://github.com/giovtorres/docker-centos7-slurm/issues/3)
* slurmdbd behavior is somewhat very weird and require explicit acctmgr cluster add as well as restart service for slrumdbd and slrumctld many times in some sorts of order.
* Anyways, slurm together with its eco and doc, is… totally a mess, Good luck
* Note that pid is configured twice (one in etc one in service), make them consistent for all three services. slurmd, slurmdbd, slurmctld.


## Experiments on hadoop & spark

`spack install spark +hadoop ^jdk@1.8.0_212`. Cautions: 1. spark is not well supported by jdk 9+. There maybe `java.lang.IllegalArgumentException` in pyspark running for java >= 9. See [this blog](https://towardsdatascience.com/how-to-get-started-with-pyspark-1adc142456ec). 2 It make things much worse by the fact that spack couldn't handle jdk install well due to some invalid url or cookies or licences issue for oracle java (the newest version jdk seems ok to install by spack though). 

My workaround: install jdk 8 by `sudo apt install openjdk-8-jdk`. Then add jdk as external package by editing packages.yaml for spack. And remember to `spack install jdk@1.8.0_212` to add it to the spack db. Then try spack install command as shown at the beginning.

To use pyspark, `spack load hadoop spark jdk`. Then directly use the python interpreter provided by spark as `pyspark`. Or `pip install pyspark` for some python, and use pyspark in the python by `import pyspark`. Remember to set both env vars `PYSPARK_PYTHON=python3` and `PYSPARK_DRIVER_PYTHON=python3`, to avoid python version mix of spark worker if you choose original python.

### cluster mode settings

Reference on this [post](http://dblab.xmu.edu.cn/blog/1714-2/)

In `spark/conf`, `cp slave.templates slave`, and add slave hostname into slave file. `cp spark-env.sh.template spark-env.sh` and add following lines.

```bash
source /home/sxz/spack/share/spack/setup-env.sh
export SPARK_DIST_CLASSPATH=$(`spack location -i hadoop`/bin/hadoop classpath)
export HADOOP_CONF_DIR=`spack location -i hadoop`/etc/hadoop
export SPARK_MASTER_IP=<master ip>
```

Make sure the conf dir is sync with all nodes (which is by default if spack install in home and home is shared within cluster).

Start hadoop first  \`spack location -i hadoop\`/sbin/start-all.sh.

Then start spark master and slaves. \`spack location -i spark\`/sbin/start-master.sh and start-slaves.sh

Task submission `pyspark --master spark://master:7077`. (Note hadoop might have some issue in the above setting and running scheme.) Actually, one should edit `etc/yarn-site.xml` to make `pyspark --master yarn` work as expected. It may be due to memory limit of VMs. See [post here](https://dongkelun.com/2018/04/16/sparkOnYarnConf/). 

```bash
<property>
    <name>yarn.nodemanager.pmem-check-enabled</name>
    <value>false</value>
</property>

<property>
    <name>yarn.nodemanager.vmem-check-enabled</name>
    <value>false</value>
</property>
```

Moreover, one still need a slaves file in hadoop/etc, though not exist for the installation?

* standalone mode [doc](https://spark.apache.org/docs/latest/spark-standalone.html), prefer this approach now.
* env vars `SPARK_MASTER_WEBUI_PORT` for webui port
* `[Pyspark: Exception: Java gateway process exited before sending the driver its port number](https://stackoverflow.com/questions/31841509/pyspark-exception-java-gateway-process-exited-before-sending-the-driver-its-po)`: spack load jdk to set `JAVA+HOME`, otherwise this error when create spark sc.

## CGROUP

setup on Ubuntu16.04 for a test. [ref](http://www.fernandoalmeida.net/blog/how-to-limit-cpu-and-memory-usage-with-cgroups-on-debian-ubuntu/), [ref](https://www.paranoids.at/cgroup-ubuntu-18-04-howto/). [cgroup: cpu setting](https://segmentfault.com/a/1190000008323952).

* `sudo apt install cgroup-tools`

* sudo vim `/etc/cgconfig.conf`

  ```bash
  group app/calculation {
    cpu {
      cpu.shares = 600;
    }
    memory {
      memory.limit_in_bytes = 20000000000;
    }
  }
  ```

* `sudo vim /etc/cgrules`

  ```bash
  user:process cpu,memory app/calculation/
  ```

* ```bash
  sudo cgconfigparser -l /etc/cgconfig.conf
  sudo cgrulesengd
  ```

* check cgroup working status: ` cat /sys/fs/cgroup/cpu/app/calculation/tasks`

### Misc

* To limit the cpu usage, combine cpu.cfs_quota_us and cpu.cfs_period_us. The ratio is the number of kernels one can maxilmally utilize. This is somewhat a hard limit comparing to cpu.shares which is just a soft limit and a nice like primitive.

## Hopefully workflow

How to achieve minimal steps.

* For master node
  * OS installation
  * create user account with sudo
  * two ethernet cable plugin and enable the wan network
  * manually `ip addr add ip dev lan_nic` 
  * sshd enable and run and generate ssh-keygen
  * apt install ansible
  * config inventory and vars for ansible roles
  * run ansible on master only
* For slave node
  * OS installation
  * create user account with the same name and password
  * ethernet cable to master switch
  * sshd enable and run
  * paste the pub key to authorized keys
* For master node
  * run ansible again for all nodes