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

### Network configuration

To simulate the cluster network topology in VM clusters, see my blog.

