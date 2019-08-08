A list of the future tasks to be implemented on HPC2, may be brief or somewhat in detail. *Not-so-urgent types of tasks are listed in italic way.*

## Hardware Level

### Network related

- [x] Enable **jumboframe** in LAN
    - [x] config on MTU of servers
    - [x] config on MTU of switches, Jumbo
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
  - [x] pam in slurm
- [x] sshfs
- [x] *rclone*
- [x] *rsyslog forward* (by ELK stack)
- [ ] *gitlab*
- [ ] *syncthing*

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
- [x] *distributed mathematica*
  - [x] automatically activation
  - [x] launch remote kernel via cli
- [x] *SLEPc PETSc and external linear and eigen solver packages*
- [ ] *Julia*
- [x] ipyparallel
- [x] *Matlab*

### Modern cloud and distributed system tools

- [ ] *OpenStack*
- [x] *Spark*
- [ ] *Kubernetes*
- [x] *Elasticsearch*
- [ ] *?ceph*

## Miscs

- [x] opensource the ansible playbooks in this HPC
- [x] ansible authorized key double check
- [x] possible setup one more v2ray inbound which has only outbounds to 176 network, used for other users to access jupyter notebook
- [x] spack install hpl for linpack benchmark(just use mkl one)
- [x] backup mysql database a
- [ ] ? backup elastic database
- [x] bootstrap setup for new machines by curl scripts
- [ ] more careful division on playbooks, new roles comes in! gpu partition and shared storage on compute nodes, backup manage node (backup of slurmctld slurmdbd and possible elastic node)
- [ ] change mount logic to more robust and support on [sn]
- [ ] add a debug partition queue for slurm
- [ ] replication of ES
- [ ] authetication of ES
- [x] make hostname consistent by ansible on ubuntu18.04, (a detailed study on cloud init subsystem)
- [ ] ganglia incomplete metric collection
- [x] ganglia gpu plugin
- [x] elastalert for mail alerting
- [ ] MPI benchmark

### tasks to explore

- [x] unifomity of user
- [x] uniformity of environment variables (propagate by sbatch)

### warning

- [x] name system with version number
- [x] ?try install all useful things in opt or home which is going to be export

### Future directions

- [ ] combination of k8s and slurm
- [ ] hybrid HPC/cloud setup and elastic scaling

### design principle

* more on master, less on slave, everything in slave should be included in ansible workflows
* more on ansible playbooks, less by hand

