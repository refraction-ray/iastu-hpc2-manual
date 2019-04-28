A list of the future tasks to be implemented on HPC2, may be brief or somewhat in detail. *Not-so-urgent types of tasks are listed in italic way.*

## Hardware Level

### Network related

- [ ] Enable **jumboframe** in LAN
  - [ ] config on MTU of servers
  - [ ] config on MTU of switches, Jumbo
- [ ] Divide S1720 switch into two **VLAN**
- [ ] DHCP on the master node with very long lease time

### Server related

- [ ] iDRAC configuration
- [ ] Open manage configuration
- [ ] *include more old machines into the cluster*

## Software Level

### OS related

- [ ] locale to en.US for all systems
- [ ] ntp server for master node
- [ ] nfs settings
- [ ] dhcp server for master node
- [ ] apt and pip source change

### DevOp softwares

- [ ] crobbler
- [ ] ansible
  - [ ] create playbooks for:
    - [ ] user management and their ssh key
    - [ ] apt tools install
    - [ ] service state management
    - [ ] network configuration (dhcp server with fix ip on master and dhcp auto on nodes)
- [ ] nagios
- [ ] *ganglia*
- [ ] *modules*

### HPC softwares

- [ ] slurm
- [ ] intel parallel studio (ifort icc intelmpi intelpython and mkl included)
- [ ] Eigen and armadillo
- [ ] boost
- [ ] *GSL*
- [ ] *distributed mathematica*
- [ ] *SLEPc PETSc and external linear and eigen solver packages*
- [ ] *Julia*

### Modern cloud and distributed system tools

- [ ] *OpenStack*
- [ ] *Spark*
- [ ] *Kubernetes*

## Main problems to address

- [ ] unifomity of user
- [ ] uniformity of environment variables

