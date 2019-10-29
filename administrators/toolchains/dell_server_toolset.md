This section focuses on toolkit with DELL servers provided by DELL.

## Open Manage

Two parts: sever administrator (1 to 1, server management) and essentials (1 to many, cluster management)

**Reference**:

* [Review and doc index on iDRAC and open manage](https://www.dell.com/support/article/us/en/04/sln129295/dell-poweredge-%E5%A6%82%E4%BD%95%E5%9C%A8%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E9%85%8D%E7%BD%AEidrac%E5%92%8C%E7%B3%BB%E7%BB%9F%E7%AE%A1%E7%90%86%E9%80%89%E9%A1%B9?lang=zh)
* [Open manage essentials: User's guide](https://www.dell.com/support/manuals/us/en/04/dell-openmanage-essentials-v2.1/omeug-v4/about-openmanage-essentials?guid=guid-f21468fb-0f16-4adb-8d29-41c5b1d32395&lang=en-us)

## iDRAC

The general term: BMC (Baseboard Management Controller). See [this post](https://medium.com/@lioukiki1/bmc%E6%98%AF%E4%BB%80%E9%BA%BC-%E8%83%BD%E5%90%83%E5%97%8E-bee457ea5c37)

iDRAC might be configure by LCD if the server has. [Basic on iDRAC config](https://www.dell.com/support/article/us/en/04/sln129356/start-up-page-for-dell-poweredge-server-of-12th-and-13th-generation-r620-r630?lang=en), [default username and password for iDRAC](https://www.dell.com/support/article/us/en/04/sln306783/dell-poweredge-what-is-the-default-username-and-password-for-idrac?lang=en)

It has its own ip assigned (though not a fan of this feature), (actually dhcp supported)

Share port with OS: [post](https://www.dell.com/community/PowerEdge-Hardware-General/iDRAC-8-NIC-Port-Sharing/td-p/5078061)

Just use via web, idrac port support dhcp.

Reset idrac password: [post](https://www.dell.com/community/Systems-Management-General/Reset-Lost-DRAC-Password-without-resetting-IP-Configs/m-p/5122979#M23584)

### CLI tool - racadm

 [Basic command of racadm](https://blog.51cto.com/wuyanc/1864022)

### Life Cycle Controler

Seems an integrated part of idrac.