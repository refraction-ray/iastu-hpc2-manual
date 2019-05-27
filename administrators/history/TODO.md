A list of the future tasks to be implemented on HPC2, may be brief or somewhat in detail. *Not-so-urgent types of tasks are listed in italic way.*

## Hardware Level

### Network related

- [ ] Enable **jumboframe** in LAN
    - [ ] config on MTU of servers
    - [ ] config on MTU of switches, Jumbo
- [x] Divide S1720 switch into two **VLAN**
- [x] DHCP on the master node with very long lease time

### Server related

- [ ] iDRAC configuration
- [ ] Open manage configuration
- [x] *include more old machines into the cluster*

## Software Level

### OS related

- [x] locale to en.US for all systems and timezone
- [x] ntp server for master node
- [x] nfs settings
- [x] dhcp server for master node
- [x] apt and pip source change
- [ ] ? shutdown password login in LAN
- [x] user resource limit
  - [x] disk limit in home
  - [x] resource limit in master
  - [x] qos in slurm
  - [x] pam in slrum
- [x] sshfs
- [ ] rclone

### DevOp softwares

- [ ] cobbler
- [x] ansible
  - [x] create playbooks for:
    - [x] user management and their ssh key
    - [x] apt tools install
    - [x] service state management
    - [x] network configuration (dhcp server with fix ip on master and dhcp auto on nodes and nat)
- [ ] *nagios*
- [x] ganglia
- [x] *modules*
- [x] backup tools

### HPC softwares

- [x] spack
- [x] slurm
- [x] intel parallel studio (ifort icc intelmpi intelpython and mkl included)
- [x] Eigen and armadillo
- [x] boost
- [x] *GSL*
- [ ] *distributed mathematica*
  - [ ] automatically activation
  - [ ] remote kernel on cluster
- [x] *SLEPc PETSc and external linear and eigen solver packages*
- [ ] *Julia*
- [x] ipyparallel

### Modern cloud and distributed system tools

- [ ] *OpenStack*
- [ ] *Spark*
- [ ] *Kubernetes*
- [ ] *Elasticsearch*
- [ ] *ceph*

## Miscs

- [ ] opensource the ansible playbooks in this HPC
- [ ] ansible authorized key double check

### tasks to explore

- [x] unifomity of user
- [x] uniformity of environment variables

### warning

- [x] name system with version number
- [x] ?try install all useful things in opt or home which is going to be export

### design principle

* more on master, less on slave, everything is slave should be included in ansible workflows
* more on ansible playbooks, less by hand

