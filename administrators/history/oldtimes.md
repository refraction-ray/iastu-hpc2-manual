In this section, the command and settings in old machines are given.

## d1

### specs

```bash
$ df -TH
Filesystem           Type   Size  Used Avail Use% Mounted on
/dev/mapper/vg_tu176064-lv_root
                     ext4    53G   28G   26G  52% /
tmpfs                tmpfs  136G     0  136G   0% /dev/shm
/dev/sda1            ext4   500M  140M  334M  30% /boot
/dev/mapper/vg_tu176064-lv_home
                     ext4   233G  140G   82G  64% /home
/dev/sdb1            ext4   985G  4.1G  931G   1% /mnt/home
/dev/sdb2            ext4   9.0T   48M  8.5T   1% /mnt/data1
$ lsblk
NAME                           MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda                              8:0    0 278.9G  0 disk
├─sda1                           8:1    0   500M  0 part /boot
└─sda2                           8:2    0 278.4G  0 part
  ├─vg_tu176064-lv_root (dm-0) 253:0    0    50G  0 lvm  /
  ├─vg_tu176064-lv_swap (dm-1) 253:1    0     8G  0 lvm  [SWAP]
  └─vg_tu176064-lv_home (dm-2) 253:2    0 220.4G  0 lvm  /home
sdb                              8:16   0   9.1T  0 disk
├─sdb1                           8:17   0 931.3G  0 part /mnt/home
└─sdb2                           8:18   0   8.2T  0 part /mnt/data1
$ lspci -knn | grep 'RAID bus controller'
01:00.0 RAID bus controller [0104]: LSI Logic / Symbios Logic MegaRAID SAS 2108 
```

```bash
$ sudo vgdisplay
  --- Volume group ---
  VG Name               vg_tu176064
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               3
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               278.38 GiB
  PE Size               4.00 MiB
  Total PE              71266
  Alloc PE / Size       71266 / 278.38 GiB
  Free  PE / Size       0 / 0
$ sudo pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda2
  VG Name               vg_tu176064
  PV Size               278.39 GiB / not usable 3.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              71266
  Free PE               0
  Allocated PE          71266
```

4U rack server but seems not so standard. location: currently right to the 2U .46 server

* Disk: sda 300G, sdb 10T. 
* Memory: 16G*16 1066MHz [M393B2K70CM0-YF8](https://www.samsung.com/semiconductor/dram/module/M393B2K70CM0-YF8/)
* CPU: Intel(R) Xeon(R) CPU E7- 4820  @ 2.00GHz * 4
* GPU(?): MGA G200, see [this post](https://superuser.com/questions/1372289/what-is-a-matrox-gpu-and-why-does-my-universitys-unix-server-have-one)
* Baseboard: DELL 0HV8Y2 (possibly R910? [R910 spec sheet](https://www.dell.com/downloads/global/products/pedge/spc_r910_new.pdf))
* MIC: BMC5709 [tech specs](https://www1.la.dell.com/hn/en/corp/Networking-Solutions/nic-broadcom-bcm5709/pd.aspx?refid=nic-broadcom-bcm5709&s=corp) 1000MBits*4 ports
* hardware raid: LSI, seems there is CLI tools `storcli` [how to install](https://blog.51cto.com/hsuehwee/1633515), but actually no controller is report.

