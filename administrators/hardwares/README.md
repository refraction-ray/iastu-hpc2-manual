This section deals with hardwares utilized or considered on building HPC2.

## Servers

### DELL T7920

*Tower workstation*

### DELL PowerEdge R740 

*2U rack server*

product no., the second digit is for the generation of Dell servers. 4 here is for 14 generation. As for the first digit, it seems that the larger number the more powerful computer it is.

* [spec sheet](https://i.dell.com/sites/csdocuments/Product_Docs/en/poweredge-r740-spec-sheet.pdf)
* [technical guide](https://www.dell.com/en-us/work/shop/povw/poweredge-r740)

**Notes**:

Quicksync is optional for this version of server.

## Others

### Rack

### Switch

* Huawei S1720-28GWR-4P

[Web configure guide](https://support.huawei.com/enterprise/zh/doc/EDOC1000169678)

* Huawei S1700-16G-AC

### Ethernet cable

### Wireless AP

## Hardware knowledges

### Jargons

* SFP: (small form factor pluggable transceiver), port may be seen on advanced switch or routers. Support 10Gb and Fibre.
* Backplane capacity(throughput):  [some interesting math on switch throughput](https://serverfault.com/questions/505125/calculating-backplane-capacity-of-a-switch), sometimes throughput may count in the unit of packets instead of traffic. In general backplane capacity should be larger than 2 times ports number times bandwidth of each ports on switch.
* POE: power supply by ethernet cable. see [here](https://kb.netgear.com/zh_CN/209/%E4%BB%80%E4%B9%88%E6%98%AF-PoE-%E4%BB%A5%E5%A4%AA%E7%BD%91%E4%BE%9B%E7%94%B5). PSE: power provider, PD: power consumer.
* KVM switch: one IO set control multiple nodes
* SAS vs. SATA: [see the comparison](https://www.diffen.com/difference/SATA_vs_Serial_Attached_SCSI)
* Link aggregation: one logical channel with more than one physical channel of ethernet

## General References

* [Basics on network: practical perspective](https://www.dell.com/community/%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8%E8%AE%A8%E8%AE%BA%E5%8C%BA/%E7%BD%91%E7%BB%9C%E5%9F%BA%E6%9C%AC%E5%8A%9F%E7%B3%BB%E5%88%97-%E7%BB%86%E8%AF%B4%E7%BD%91%E7%BB%9C%E9%82%A3%E4%BA%9B%E4%BA%8B%E5%84%BF-3%E6%9C%8826%E6%97%A5%E6%9B%B4%E6%96%B0/td-p/7045185)

