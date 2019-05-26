## Information

IASTU-HPC2 is a mini cluster which meets high performance computation(HPC) needs in physics research.

It is free to use for any member from Yao's group. The user account shall be assigned upon request to the administrators.

The cluster provide both state-of-the-art hardwares and sophisticated modern software stacks for HPC.

## Glossary

* HPC: High Performance Computing refers to the practice of aggregating computing power to achieve higher performance for numerical computation. HPC also refers clusters organized in such a style.
* Node: A node is a physical, stand-alone computer, that can handle computing tasks and run jobs. It's connected to other compute nodes via a fast network interconnect, and contains CPUs, memory and harddisks.
* Cluster: A cluster is the collection of nodes with networking and file storage facilities. It's usually a group of independent computers connected via a switch. There are various styles of clusters nowdays in industry and academic context. The one we utilized in our cluster is the well-known HPC style cluster, in contrast with other newer styles such as MapReduce clusters for big data. In the doc, *the* cluster usually represents for our cluster IASTU-HPC2.
* Login node: Login nodes are points of access to the cluster. Users are required to connect to login nodes via SSH. In login node, users can debug, compile and test their codes and projects. But it is not recommended to do CPU computation intensive tasks in master node in general. The login node is also called master in this documentations.
* Compuation nodes: Computation nodes are nodes for calculations, which in general cannot access directly by the user. It can only be utilized by resource manager in master node.
* IO: Input/Output refer to file writing/readings and network packets sending/receivings. In our context, it is often related to file IO.
* Socket: the physical package of a CPU processor, which is composed of several CPU cores.
* CPU cores: physical cores of CPU. For example, a socket of Intel Xeon 5120 has 14 CPU cores and 28 threades with hyper-threading on.