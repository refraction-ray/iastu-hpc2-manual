A list of the future tasks to be implemented on HPC2, may be brief or somewhat in detail. *Not-so-urgent types of tasks are listed in italic way.*

## Hardware Level

### Network related

- [ ] Enable **jumboframe** in LAN
  - [ ] config on MTU of servers
  - [ ] config on MTU of switches, Jumbo
- [ ] Divide S1720 switch into two **VLAN**
- [x] DHCP on the master node with very long lease time

### Server related

- [ ] iDRAC configuration
- [ ] Open manage configuration
- [ ] *include more old machines into the cluster*

## Software Level

### OS related

- [x] locale to en.US for all systems and timezone
- [x] ntp server for master node
- [x] nfs settings
- [x] dhcp server for master node
- [x] apt and pip source change
- [ ] ? shutdown password login in LAN

### DevOp softwares

- [ ] cobbler
- [x] ansible
  - [ ] create playbooks for:
    - [ ] user management and their ssh key
    - [ ] apt tools install
    - [ ] service state management
    - [ ] network configuration (dhcp server with fix ip on master and dhcp auto on nodes and nat)
- [ ] nagios
- [ ] *ganglia*
- [x] *modules*

### HPC softwares

- [x] spack
- [x] slurm
- [ ] intel parallel studio (ifort icc intelmpi intelpython and mkl included)
- [x] Eigen and armadillo
- [ ] boost
- [ ] *GSL*
- [ ] *distributed mathematica*
- [ ] *SLEPc PETSc and external linear and eigen solver packages*
- [ ] *Julia*

### Modern cloud and distributed system tools

- [ ] *OpenStack*
- [ ] *Spark*
- [ ] *Kubernetes*
- [ ] *Elasticsearch*
- [ ] *ceph*

## Miscs

### tasks to explore

- [x] unifomity of user
- [x] uniformity of environment variables

### warning

- [x] name system with version number
- [x] ?try install all useful things in opt or home which is going to be export

### design principle

* more on master, less on slave
* more on ansible playbooks, less by hand

