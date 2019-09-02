In this section, I will review some aspects on the log and monitoring related softwares such as syslog, nagios and ganglia.

## rsyslog

## logrotate

## Logstash

*Merged into elk stack*

Pipeline from log to elastic search, see [this blog](https://www.cnblogs.com/yincheng/p/logstash.html) for introduction.

## Nagios

### Installation

* See [this post](https://websiteforstudents.com/install-nagios-server-on-ubuntu-16-04-17-10-18-04-lts-server/) for its installation on ubuntu (one node).
* Install on ubuntu 18.04 cluster and basic configuration, [post](https://help.ubuntu.com/lts/serverguide/nagios.html.en)

## Ganglia

### Installation

* [ubuntu16.04](https://hostpresto.com/community/tutorials/how-to-install-and-configure-ganglia-monitor-on-ubuntu-16-04/)

### Backend

rrdtool: round robin database

## ELK

~~Elasticsearch + logstash+kibana, to be dployed soon, firstly install on VM environment~~

initial install finished on the cluster

[simple intro](https://www.ibm.com/developerworks/cn/opensource/os-cn-elk/index.html)

[Installation of ELK on ubuntu18.04 by digital ocean](https://www.digitalocean.com/community/tutorials/how-to-install-elasticsearch-logstash-and-kibana-elastic-stack-on-ubuntu-18-04)

ganglia input plugin for logstash: [doc](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-ganglia.html)

multiple input for logstash and type label: [so](https://stackoverflow.com/questions/18330541/how-to-handle-multiple-heterogeneous-inputs-with-logstash)

```bash
ganglia {
    port => 28649
    host => "{{ master_ip }}"
    type => "ganglia"
    
  }
  
  udp_send_channel {
  host = {{ master_name }}
  port = 28649
  ttl = 1
}
```

experiment conclusion: ganglia doesn't work well with logstash

[elasticalert](https://elastalert.readthedocs.io/en/latest/elastalert.html): combine ELK stack and email alert

Cluster of ES [post](https://cloud.tencent.com/developer/article/1189282), to be deployed. One also need to configure ssl for es cluster if authetication is enabled. see [this doc](https://www.elastic.co/guide/en/elasticsearch/reference/current/configuring-tls.html).

## Benchmark

### Linpack (hpl)

[post on linpack](https://saintaardvarkthecarpeted.com/blog/2011/06/10/linpack-_a_newbies_view/)

Theorectial Flop value for CPU: [post](https://software.intel.com/en-us/forums/software-tuning-performance-optimization-platform-monitoring/topic/761046)

> All of the Gold 6000 processors have two AVX512 units, so they are capable of 32 DP FLOPS/cycle.  The Gold 5000 processors have one AVX512 unit (except for the Gold 5122, which has two), so they are capable of 16 DP FLOPS/cycle.

Simply linpack directly from intel parallel studio: [blog](http://blog.chonor.cn/index.php/hplhigh-performance-linpack/)

Results: 580Gflop on single machine in normal env without fine tuning

Intel mkl linpack documentation: [intel](https://software.intel.com/en-us/mkl-windows-developer-guide-intel-distribution-for-linpack-benchmark)

### MPI benchmarks

* An example using spack installed stack for mpi bandwidth benchmark: [blog](https://jiaweizhuang.github.io/blog/mpi-tcp-ec2/#mpi-bandwidth-test-with-osu-mirco-benchmarks)

### IO benchmarks

* An example using spack installed stack for io benchmark: [blog](https://jiaweizhuang.github.io/blog/fsx-experiments/#i-o-benchmark-by-ior)