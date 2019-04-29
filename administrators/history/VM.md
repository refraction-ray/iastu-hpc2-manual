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

### Network configuration

To simulate the cluster network topology in VM clusters, see my blog [here](https://re-ra.xyz/KVM-%E8%99%9A%E6%8B%9F%E9%9B%86%E7%BE%A4%E7%BD%91%E7%BB%9C%E6%8B%93%E6%89%91%E8%AE%BE%E7%BD%AE/).

## Basic on master

### Python3 preparation

The operations below only on master VM. *Since ansible can be installed directly via apt, the python part might be moved later which controled by ansible*

* apt source [change to tuna](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

* `sudo apt update && sudo apt install python3-pip`

* There is a pip3 upgrade issue on ubuntu18.04, the revert command is `sudo python3 -m pip uninstall pip`, see [here](https://stackoverflow.com/questions/49836676/error-after-upgrading-pip-cannot-import-name-main) for pip upgrade issue. 

* change the default path of dist package, create file as `/etc/pip.conf`, and add the following in the file

  ```bash
  [global]
  target=/opt/python3/dist-packages
  index-url=https://pypi.tuna.tsinghua.edu.cn/simple
  ```

  and then `sudo pip3 install` for every packagecd

   you want. `export PYTHONPATH=/opt/python3/dist-packages` for python3 to auto include it in sys.path.

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