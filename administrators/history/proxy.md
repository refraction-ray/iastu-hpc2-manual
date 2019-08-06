In this section, I will give some reviews and setups on the upfront machine. This machine is somehow an integrated part of the cluster but a standalone machine at the same time. There are no computation tasks running on this machine, but it provided several features for the cluster as a whole which is better to separate from the cluster master to enhance security. 

In the below, we may call this machine r1.

## Software setups

### V2RAY

Web proxy is very vital for the cluster to run. Since master of the cluster only has Internet access to IPs in the campus which protects the whole cluster from most of the attacks, we need some ways to make the cluster access the Internet in both direction. For out direction, we implement the http proxy by v2ray in r1, and for the in direction, we implement the port forwarding by v2ray dokodemo-door in r1.  Such port forwarding works for ssh, but how about jupyter notebooks and other web service in the cluster which we may wanna access outside campus? Well, there are also two approaches. Firstly, one can use VPN provided by University, so that the cluster is reachable. Secondly, we also implement a shadowsocks proxy in r1 (also by v2ray) whose outbounds are restriced to the subnet of IASTU. One can use this shadowsocks service to access web resource on the cluster if the user is outside campus (also the user cannot abuse the shadowsocks service since it is only open to subnet in the campus).

### Misc

Unlike the cluster utillizing purely ipv4 stacks, this relay host has ipv6 address. Actually, there is no ipv6 access in IASTU network. But one can setup ipv6 by following [this tutorial](https://github.com/tuna/ipv6.tsinghua.edu.cn/blob/master/isatap.md), which create a 6in4 tunnel to the server on campus.

A ddns crontab task is also configured, which has the flask server on my digital ocean VPS. The ddns infrastructure in implemented by me as in [this repo](https://github.com/refraction-ray/simple-ddns).