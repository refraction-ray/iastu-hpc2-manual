This section reviews the real setups on the cluster.

## hardware

### switch

port 23 24 are changed into VLAN2 for S1720-28GWR-4P.

### AP

The reseverd ip is `192.168.1.250/24`. The dhcp is turned off and it sit in AP mode. When connecting to it, the ip is assigned by master node DHCP sevice in 48 subnet.

### master server

LAN upper RJ45 port 10/24 and WAN lower RJ45 port 44/24.

maste node is currently set as static ip in WAN.

Harddisk(? position may not be accurate now): left down for 2.5 SSD, right down for sdb, old 2T HDD, left up for sdc, new 3.5 2T HDD.

### computation server

The leftmost RJ45 port is utilized. (not the one for iDRAC) The nic name is eno1 in ubuntu OS.

### outdated server

centos 6.6, name as d1. For operations on this server, please refer [this section](./oldtimes.md).

**Note:** d1 is not ready to open to users. It currently went offline.

## software

### basic on master

First use fdisk make one partition on each disk of sdb and sdc, and then use `mkfs.ext4 /dev/sdb1` to format the two disks. Mount two 2T hdds `/dev/sdb1` and `/dev/sdc1` to /DATA and /BACKUP, where /DATA has similar permission with tmp and shared across nfs. Namely, `chmod a+w /DATA`, `chmod a+t /DATA`. Meanwhile, /BACKUP is only written by root. There are several backup crontab tasks by root managed by `rsync -az` from /home/ubuntu and /opt to /BACKUP. Besides, the config file `/etc/fstab` is also configured such that /DATA and /BACKUP can be mounted automatically when reboot.

```bash
## add two following lines in /etc/fstab
/dev/sdb1  /DATA ext4 defaults 0 0
/dev/sdc1  /BACKUP ext4 defaults 0 0
```


The backup crontab and fstab mount config have not included into ansible workflow due to flexibility consideration.

In terms of network and nfs, ntp, apt setups, please see relevant section in [Virtual Machine](./VM.md) part.

**Note**: For proxy part, note there are softwares not following http_proxy and need to set proxy in their own way. Such apps include apt, and git and [crontab](https://unix.stackexchange.com/questions/390974/how-should-i-set-http-proxy-variable-for-cron-jobs) and [docker](https://elegantinfrastructure.com/docker/ultimate-guide-to-docker-http-proxy-configuration/) (Note there are four different types of proxies you may want to configure in terms of docker!). (Maybe /etc/enviroment is a better place for http proxy variables)

### ansible

`sudo apt install ansible` on master node.

Please see the ansible playbooks in HPC2. The playbooks went opensource, see [here](https://github.com/refraction-ray/hpc-build-ansible-playbooks).

Test command: `ansible-playbook site_test.yml -i hosts_test -vv`, remembering change the group role in hosts_test, this test will be conducted on a remote VM server.

### gpu drivers

```bash
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt-get updatE
## check drivers
ubuntu-drivers devices
sudo apt-get install nvidia-driver-415
## reboot is needed
apt list --installed|grep nvidia
nvidia-smi
```

### spack

*into ansible workflow*

Just see the practice on [spack](../toolchains/spack.md). Combine spack with lmod to provide consitent interface on package management.

#### some spack things to note

* spack install rclone, and there is a go folder on the home directory, outside the spack folder!! Seems because `GOPATH` is by default instead of set within spack folder. See [this blog](https://blog.csdn.net/zwqjoy/article/details/78788918) for more info on go project and package organization in fs level. **Solved** by [this commit](https://github.com/tldahlgren/spack/commit/d2e22d95e7b50a9e94b4ebc0f9bc25fb61ca3cdc).

### python

*into ansible workflow and intel parallel studio installation, always use intel python and its conda for users*

prefered way: ~~intel python+spack pip~~. `spack load intel-parallel-studio`, `spack load py-setuptools`, `spack load py-pip`. **Intel python+conda create enviroment**.

Never use admin account's global pip. Reason: the package would be installed in ~/.pip. However, if later spack-pip install some packages, the dependece is automatically used if it is already in ~/.pip. But this folder is not accessible by other user which may lead to a chaos on python packages. However, for normal user, global pip3 is the recommended way to download packages.

### jupyter

Use intel python and pip as root, `pip install jupyter ipyparallel jupyter_contrib_nbextensions`. Somehow the cluster tab works after several trials. Dont know exact solution though.

### mathematica

Installed by the bash script on /opt/mathematica/verno. And the script is in the bin subdirectory of the above path. Add it as a package in spack override repo, and `spack load mathematica` to use it.

**Possible issue:** The activation need to be carried out per user per node basis. ~~Maybe have a look at MathML in the future.~~ Already written a script to activate all nodes at one time.

To utilize remote kernel launching with a better interface, add tunnel.m at Kernels/packages. Besides, one should also add tunnel.sh and tunnel_sub.sh in their home directory `~/.Mathematica/FrontEnd`.

One liner for user to be accessible `ansible -i /home/ubuntu/hpc/hosts all -m command -a "/usr/bin/python3 /home/ubuntu/softwares/mmaact/automma.py '/opt/mathematica/11.0.1/bin/math'" --become-user=<user> -Kvv --become`.

### matlab

mount iso to a dir, and use `ssh -X` to install by X11 forwarding. Remember umount and mount iso2.

It is worth noting, that matlab installer supports -X for forwarding. While for matlab itself, only `ssh -Y` works for remote desktop scheme.

### singularity

`spack install singularity`, remember to check `spack edit singularity`, there is an after install warn prompt asking you run a script which would change the permission of some files with s bit. It is crucial for singularity run by normal users.


### ganglia

*into ansible workflow*

`apt-get install ganglia-monitor ganglia-monitor-python gmetad ganglia-webfrontend`

ganglia-monitor is client side gmond. 

On client, only `ganglia-monitor` and `ganglia-monitor-python` should be installed. The second one is necessary as modules to watch spec of nodes.

Please reference [this post](https://hostpresto.com/community/tutorials/how-to-install-and-configure-ganglia-monitor-on-ubuntu-16-04/) on the change of configuration files.

The workflow of ganglia configuration and installation have been merged to ansible playbooks.

gmetric can be customized to report certain spec. See temperature example [here](http://cutler.io/2011/11/lm-sensors-on-ganglia/).

Apache password protected sites: [digitalocean](https://www.digitalocean.com/community/tutorials/how-to-set-up-password-authentication-with-apache-on-ubuntu-14-04)

gmetric expire invalid metric: [mailist](https://sourceforge.net/p/ganglia/mailman/message/23458211/): add a finite int for -d flag in gmetric cli. Also see [this post](https://qingwen-chen.blogspot.com/2011/03/remove-unused-metrics-from-ganglia.html)

GPU monitoring part, from beginning, I was thinking about incoporate nvidia plugin for ganglia [github](https://github.com/ganglia/gmond_python_modules/tree/master/gpu/nvidia). But the solution is too invasive and no guranteen on the sucess based on search results in the internet. So finally I decided to write a small script to collect gpu data given by nvidia-smi and send them to gmond by gmetric command. Just as what I have done on cpu temperature.

### ELK

*into ansible workflow*

* add elastic repo and key

* apt install elasticsearch, es binding to localhost instead of master

* apt install kibana and configure nginx reverse proxy

* apt install logstash and configure the pipe, note the ip binding of beat input configure (there are `""` for string ip)

* apt install filebeat (on all nodes)

* `sudo filebeat setup --template -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["localhost:9200"]'`

* `sudo filebeat setup -e -E output.logstash.enabled=false -E output.elasticsearch.hosts=['localhost:9200'] -E setup.kibana.host=localhost:5601`

* `sudo filebeat setup --pipelines --modules system nginx mysql  -E output.logstash.enabled=false -E output.elasticsearch.hosts=['localhost:9200'] -E setup.kibana.host=localhost:5601` [ref](https://www.elastic.co/guide/en/logstash/master/use-ingest-pipelines.html)

* Summary of the above three points to init filebeat, the following is enough

  ```bash
  $ filebeat modules enable system nginx mysql iptables apache2 # space instead of ,

  $ sudo filebeat setup -e -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["localhost:9200"]'

  $ sudo filebeat setup --pipelines --modules nginx,mysql,iptables,apache2,system  -E output.logstash.enabled=false -E output.elasticsearch.hosts=['localhost:9200']  -M system.auth.var.convert_timezone=true -M system.syslog.var.convert_timezone=true -M nginx.error.var.convert_timezone=true ## the arg for modules: , instead of space
  ```

  The `-E` flag is for temporary output to es database to write in some templates and pipelines. Since the filebeat is configured by output to logstash.

* Note filebeat setup need temporary linking to es database, which is a must.

* work test `curl -X GET "localhost:9200/_cat/indices?v"`

* disable logstash ssl (seems to be disable by default)

* edit the index of logstash output (add beat.hostname in index name)

* filebeat system module, timezone problem: [post](https://discuss.elastic.co/t/filebeat-system-module-auth-log-timezone-conversion-problem/140192/7), [post with logstash setup](https://discuss.elastic.co/t/filebeat-pipeline-setup-for-system-module-ignores-var-convert-timezone-true/181456/2)

  *Detailed explanation on the tiemstamp mismatch for system module from filebeat: there are two types of log files, one with time zone information or direct claim on UTC timestamp, which the parser of filebeat can input the data into ES confidently with UTC timestamps. That is because ES always accept UTC timestamps and @timestamp is time zone agnostic. The reason that we feel the timestamps are just right in kibana is because the default setting of kibana is render the timestamp with timezones defined by broswer, aka. local OS system.*

  *Coming back to the second type of log files, like syslog and authlog in ubuntu, they have timestamps but have no indication whether such timestamps is UTC or localtime. Actually rsyslog can run on UTC even if OS is set to some other timezones. This could persists to the restart of rsyslog service. So to parse these logs and write UTC timestamps to ES, filebeat must has a way to specify whetehr we need to convert the lietral time strings in syslog by some timezone and write another literal timestamps to ES. This is in principle configured by /etc/filbeat/modules.d/system.yml. There is a variable in the file called var.convert_timezone, turn it to be true (seems default false), and in principle, you can get the correct time view in kibana now.*

  *The mechanism behind convert_timezone in a ingest pipeline ([pipeline basic](https://www.elastic.co/cn/blog/new-way-to-ingest-part-1)) of ES. Namely the conversion is happening in ES side instead of handling by filebeat itself.*

  *But reconfiguring filebeat turns out not that easy. There are two totally different cases. The first one is the output of filebeat is directly some ES, which seems to be the default support case from the doc. In this case, one should first stop filebeat service, and delete all previou pipeline by curl, and then start filebeat service again, everything should be fine now, easy. In this case, restaring filebeat will automatically generate new pipelines in ES, which you dont need to care.*

  *The second use case of filebeat, make the output of filebeat to logstash. I dont think this setup is very meaningful nowadays, since filebeat seems very powerful by itself. However, if you insist this approach and try to fix time zone problem, it would be a little subtle. A simple change on system.yml won't work, though it may be a bug instead of a feature. There are several differences between this case and the ES direct case. In the second case, the pipeline in ES cannot be autogenerated when start filebeat, which is fair since filebeat has no chance to communicate with the ES directly on the normal runtime. So after stop filebeat service and delete all previous pipelines in ES, you need to generate pipelines by hand using filebeat setup tools as indicated by the above bash block. Apart from setup —pipelines, setup -e is also suggested in case. Things behinds setup -e is to configure both index template in ES and dashbord in Kibana. And setup —pipelines, as shown by this option, is to add pipelines in ES. One can restart filebeat after there two lines of commands. Extra options for these commands with -E flags is for temporary config on config time which overwrites the default config in filebeat.yml. This is necessary because filebeat must connect to ES directly at configuring time to write there pipelines and index templates back to the ES. It also worth noting these -M options for generating pipelines. It turns out hacking convert_timezone in yml files in modules.d doesn't work for logstash output. Instead, you MUST specify them by -M options explicitly when generating pipelines. E is for configuration overwirte while M is for module configuration overwrite.*

  For certain pipeline in ES, one can check by ``curl -XGET 'http://localhost:9200/_ingest/pipeline/filebeat-6.8.0-nginx-e*'``, one should make sure there is a timezone key field in it if the convert_timezone function is enabled.

  Also tips for debug, turn kibana time range as this week, so that you can realize that something may written into the furture by some misconfigure.

  *In sum, time is a big topic and a subtle issue in ELK stack and even in development in general. Pay attention to them and be careful!*

* to change config of filebeat

  * service filebeat stop
  * `curl -XDELETE 'http://localhost:9200/_ingest/pipeline/filebeat*'`
  * run the two setup steps in the summray
  * service filebeat start

* `sudo /usr/share/elasticsearch/bin/elasticsearch-setup-passwords interactive` (Thanks to elastic guys, xpack security now comes to free for 6.8.0+).

* configure on password in logstash need quote.

* To query es with user authetication, just add `-u esuser:espass` option in curl commands.

**Misc note:**

* For debug test on es, curl will go proxy!!
* no specified JAVA_HOME warning in es service log doesn't matter
* logstash config [intro](https://www.elastic.co/guide/en/logstash/current/advanced-pipeline.html#configuring-geoip-plugin), grok [official guide](https://www.elastic.co/guide/en/logstash/7.1/plugins-filters-grok.html)
* actually it is ok for missing hostname, but the log from compute node is just too small compared to master…. It is not an issue due to ELK stack, but issue of non uptodate syslog. (time mismatch)
* [timezone issue of syslog](https://stackoverflow.com/questions/22853026/ubuntu-change-timezone-to-utc-does-not-affect-the-time-of-syslog): every damon can see the timezone issue only solved by service restart! **case solved**
* ES basic query syntax: [doc](https://lucene.apache.org/core/2_9_4/queryparsersyntax.html)
* pipline doesn't exsit error when modules is enabled for filebeat: [post](https://discuss.elastic.co/t/fitebeat-6-3-1-system-module-issue/141587/2). Run `curl -XDELETE "http://localhost:9200/_ingest/pipeline/filebeat-*"`, to resolve conflict with possible old pipelines?
* Actual problem: one should load pipeline for all modules in one line, otherwise, the latter one would overwrite the former one.
* Timestamp in ES is always UTC, but kibana show them with broswer default timezone.

#### elastalert

*into ansible workflow*

In general, a cool, reasonable and east-to-follow tools. The logical flow is better compared to the tools as middlewares in the logstash. Here we just query the ES database periodically, and sent alert accordingly based on some predefined rules.

~~`pip3 install elastalert`~~

~~`pip3 install "elasticsearch>=5.0.0"`~~

`apt install elastalert`

`elastalert-create-index`

For mail configuration, see [this issue](https://github.com/Yelp/elastalert/issues/627). It is better to use campus mail as From, since the cluster is air gapped with the Internet. But it could also be done for other smtp servers, just setup a port forwarding on the proxy server.

**Note:** to use `elastalert-test-rule`, first `unset http_proxy` to make it accessible to localhost ES.

### quota

See reference on ubuntu 18.04 quota command: [digital ocean](https://www.digitalocean.com/community/tutorials/how-to-set-filesystem-quotas-on-ubuntu-18-04), it is very well write up.

`sudo apt install quota`

*need further experiments on VM cluster first before apply it, always be carful for disk stuff*

See the VM corresponding part for operations.

Not included in ansible due to flexibility consideration. Only use in master node.

### ulimit

*merged into ansible workflow*

user level vs shell level: [ref](https://www.ibm.com/developerworks/cn/linux/l-cn-ulimit/index.html).

hard limit can only be changed by root and soft limit is something that anyone can change, but firstly, you need to change it.

ulimit config file in `/etc/security/limit.d`, see [demo](https://access.redhat.com/solutions/61334)

check by `ulimit -a` for individual users.

### numa

`apt install numactl`

```bash
$ numactl -H
available: 2 nodes (0-1)
node 0 cpus: 0 1 2 3 4 5 6 7 8 9 10 11 12 13 28 29 30 31 32 33 34 35 36 37 38 39 40 41
node 0 size: 128601 MB
node 0 free: 14004 MB
node 1 cpus: 14 15 16 17 18 19 20 21 22 23 24 25 26 27 42 43 44 45 46 47 48 49 50 51 52 53 54 55
node 1 size: 128996 MB
node 1 free: 27058 MB
node distances:
node   0   1
  0:  10  21
  1:  21  10
```

`apt install hwloc`

### cgroup

`sudo apt install cgroup-tools`

```bash
# /etc/cgconfig.conf
group service {
  cpuset {
    cpuset.cpus="0-13";
    cpuset.mems=0;
  }
}
group user {
  cpuset {
    cpuset.cpus="14-27,42-55";
    cpuset.mems=1;
  }
}

# /etc/cgrules.conf
elasticsearch: cpuset service/
kibana:        cpuset service/
logstash:      cpuset service/
<user>:        cpuset user/

# make new cgroup policy come to live
sudo cgconfigparser -l /etc/cgconfig.conf
sudo cgrulesengd
```

### tinc

`sudo apt install tinc`

combine tinc vpn with http proxy, such that http proxy ip is not public to everyone. Make http proxy only listen to the tinc ip interface.

```bash
## /etc/netname/tinc.conf
Name = wst
Interface = tinc
ConnectTo = of
ConnectTo = smart
AddressFamily = any
GraphDumpFile = /var/log/tinc-peer.log

## /etc/netname/tinc-up
#! /bin/bash
ip link set $INTERFACE up
ip addr add {{ tinc_ip }}/16 dev $INTERFACE

## /etc/netname/tinc-down
#! /bin/bash
ip link set $INTERFACE down

## /etc/netname/hosts/wst
Subnet =  10.x.y.2/32

-----BEGIN RSA PUBLIC KEY-----
pubkey-wst
-----END RSA PUBLIC KEY-----

## /etc/netname/hosts/of
Subnet =  10.x.y.1/32
Address = public ip

-----BEGIN RSA PUBLIC KEY-----
pubkey-of
-----END RSA PUBLIC KEY-----
```

`tincd -n netname -K` to generate key pairs, and `tincd -n netname` to start the daemon. For debug usage, try `tincd -n netname -d5 -D`  for a foregroud d with verbose output. On each Tinc daemon debug window, quit the daemon by pressing `CTRL-\`.

`sudo iptables -t nat -I POSTROUTING 1 -o tinc -s 192.168.48.0/24 ! -d 192.168.48.0/24 -j SNAT --to-source 10.26.11.1` on master node, make compute nodes available without any modification on them. (this new SNAT line is hopefully also managed by ansible playbooks). `sudo iptables -t nat -nLv` check current iptables.

### jumbo frame

`ip link set eth0 mtu 9000`

Test: `ping -M do -s 8972 master`, do: fragmentation forbiden, there is head bytes auto added, so -s 9000 is unaccessible, see [this post](https://zhuanlan.zhihu.com/p/30020463).

MTU setting in netplan has issues in ubuntu18.04, so basically it doesn't work by netplan apply. See one [possible solution by add mac address match in netplan](https://hoppsjots.org/?p=229)

The benchmarks shows little gain in enabling jumbo frames.

### docker

* Installation [reference](https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/) (somehow docker's apt source is added into the main source.list file instead of ppa file in source.d dir)
* Add relevant user to docker group: [ref](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user)
* proxy settings on docker server: [ref](https://elegantinfrastructure.com/docker/ultimate-guide-to-docker-http-proxy-configuration/)
* change default docker image path to /DATA: [ref](https://linuxconfig.org/how-to-move-docker-s-default-var-lib-docker-to-another-directory-on-ubuntu-debian-linux)

**Warning**: only trusted used can be add to docker group to directly communicate with docker deamon. It is not desinged for normal users but only resever for the administrator to debug. See [security issues of docker](https://docs.docker.com/engine/security/security/#/docker-daemon-attack-surface) and [also this post](https://fosterelli.co/privilege-escalation-via-docker.html). For normal use of containers, please try sigularity instead.

### mail

**currently no configuration right now**. mailutils seem to use hostname as fromto, no matter what myhostname is configured by postfix, ~~it instead use `-aFrom` in commad line~~. Workable example ` echo "hello"|mail --debug-level 3 -s "subject" -aFrom user@some.localdomain receiver@mails.tsinghua.edu.cn`. Tested on r1, no configuration on the cluster.

```bash
$ sudo apt install mailutils
Postfix (main.cf) was not set up.  Start with
  cp /usr/share/postfix/main.cf.debian /etc/postfix/main.cf
.  If you need to make changes, edit /etc/postfix/main.cf (and others) as
needed.  To view Postfix configuration values, see postconf(1).
After modifying main.cf, be sure to run 'service postfix reload'.
$ sudo apt remove mailutils
```

### backup 

* legacy approch (deprecated)

```bash
crontab -l
# user home dirs, more are omitted
20 2 * * * /usr/bin/rsync -avz /home/ubuntu/ /BACKUP/ubuntu/
# softwares
20 5 * * 2 /usr/bin/rsync -avz /opt/ /BACKUP/opt/
# ganglia rrd files
20 5 * * 4 /usr/bin/rsync -avz /var/lib/ganglia/rrds/ /BACKUP/database/rrds/
# configs
20 4 * * * /usr/bin/rsync -avz /etc/ /BACKUP/etc/
# database files for slurmdbd
10 5 * * 3 /usr/bin/rsync -avz /var/lib/mysql/ /BACKUP/database/mysql/
```

* new approach based on restic

`apt install restic`

```bash
$ sudo restic init --repo /BACKUP/restic
$ sudo restic -r /BACKUP/restic backup /home
$ sudo RESTIC_PASSWORD= restic -r /BACKUP/restic snapshots
$ sudo RESTIC_PASSWORD="" restic -r /BACKUP/restic check
$ sudo RESTIC_PASSWORD="" restic -r /BACKUP/restic mount ./tmpmnt
# RESTIC_PASSWORD="" restic -r /BACKUP/restic forget --keep-daily 5  --prune
# RESTIC_PASSWORD="" /usr/bin/restic -r /BACKUP/restic backup --one-file-system / --exclude-file=/BACKUP/ignorefile
```

ignorefile

```
/tmp/*
/dev/*
/DATA*
/BACKUP
/run/*
/proc/*
/swap.img
/lost+found/*
/mnt/*
/home/*
/opt/*
```

Approach to recover the whole OS in hard disk level: [post](https://forum.restic.net/t/lack-of-documentation-on-how-to-do-a-full-system-back-up/659/18)

## some benchmarks

### network

* iperf, the master to compuation node bandwidth is around 940Mbit/s, which is near to the limit of the Gigabit nic.

### cpu

* Tried linpack in MKL, see results in [here](../toolchains/log.md)

### memory

Available frequency is 2400, though the param is 2666, the speed is limited by CPU 5120.

### disk

* by brute force `dd if=/dev/zero of=/tmp/output bs=8k count=50k`
  * cn3 ssd: 1.4GB/s
  * cn3 write to home folder, which is share by NFS and stored in master's SSD, 89.1 MB/s
  * master ssd:  1.3 GB/s
  * master /DATA, sdb1: 1.3GB/s (? cannot understand), similar results for sdc in master, it is weird though. (maybe due to disk controller or cache therein?)
  * dn1: sdb2 557MB/s, similar result for sdb, sda (under lvm): 471MB/s