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