In this section, I will give some reviews and setups on the upfront machine. This machine is somehow an integrated part of the cluster but a standalone machine at the same time. There are no computation tasks running on this machine, but it provided several features for the cluster as a whole which is better to separate from the cluster master to enhance security. 

In the below, we may call this machine r1.

## Hardware specs

Actually it is not important how powerful the hardware is for such a machine. Anyway, the specs are below.

* Intel(R) Core(TM) i7-3770 CPU @ 3.40GHz
* 16G Memory
* 256G+1T storage

## Software setups

### V2RAY

Web proxy is very vital for the cluster to run. Since master of the cluster only has Internet access to IPs in the campus which protects the whole cluster from most of the attacks, we need some ways to make the cluster access the Internet in both direction. For out direction, we implement the http proxy by v2ray in r1, and for the in direction, we implement the port forwarding by v2ray dokodemo-door in r1.  Such port forwarding works for ssh, but how about jupyter notebooks and other web service in the cluster which we may wanna access outside campus? Well, there are also two approaches. Firstly, one can use VPN provided by University, so that the cluster is reachable. Secondly, we also implement a shadowsocks proxy in r1 (also by v2ray) whose outbounds are restriced to the subnet of IASTU. One can use this shadowsocks service to access web resource on the cluster if the user is outside campus (also the user cannot abuse the shadowsocks service since it is only open to subnet in the campus).

### Misc

* ipv6

  Unlike the cluster utillizing purely ipv4 stacks, this relay host has ipv6 address. Actually, there is no ipv6 access in IASTU network. But one can setup ipv6 by following [this tutorial](https://github.com/tuna/ipv6.tsinghua.edu.cn/blob/master/isatap.md), which create a 6in4 tunnel to the server on campus. To ensure the tunnel is persistent, I added one line in root's crontab as `@reboot /bin/bash .../.ipv6.sh`. I think this approach is more elegant and simpler than hacking rc.local thing.

* ddns

  A ddns crontab task is also configured, which has the flask server on my digital ocean VPS. The ddns infrastructure in implemented by me as in [this repo](https://github.com/refraction-ray/simple-ddns).

### Mail

`apt install mailutils`

Issue: cannot tune the from address beyond hostname, postfix cannot control `mail` in this. One need to tune the hostname in the form of domains, with dot between to make mails acceptable by other smtp servers. Even though, only campus mail server accept the mail, because the domain is of course illegal from the beginning. Another route is to install `ssmtp` instead of postfix, which can configured easily to send mails by other smtp server at first place (bu using pasword and account on other mail service). And ssmtp is also compatible with `mail` frontend. However, to keep things on r1 the most simple, we still take the former route: using postfix with hostname set to `hostname.localdomain` form. Then `mail` can send alert mail to university emails. In this approach, no personal mail credential need to be stored on r1.

### Monitoring

Just use a homemade super lightweighted monitoring script (add into crontab), instead of any sophisticate monitor systems. See a demo script [on the gist](https://gist.github.com/refraction-ray/4904205f157bd79cd23b4a11c5ac2428). In case, r1 is disconnected to the campus network such that the email cannot be sent, we also implementa curl to the cluster nginx, as `curl -XWARN` to record these warnings in nginx log and can be later viewed on kibana and the whole ELK stack on the cluster.

## Design principles

The ultimate design goal or r1 is security. The aim is that even r1 is hacked, the hacker cannot get enough useful informations to threatened the security of the real cluster.