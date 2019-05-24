**Note:** User files have no backup and no guaranteen unless special request to the administrator. So never ever store valuable files only in the cluster.

## /home

All user directories in home dir are in the ssd of master node. And home directory is shared via nfs to all computation nodes. The SSD is in size of **500GB** in total which should be shared with OS and all users, so only small tasks are recommeded to IO within home dir. Never ever store large datafiles in home dir! The home dir is under control of quota which automatically limit users' disk usage. The current soft limit and hard limit of home disk usage is 60GB and 80GB. When the disk usage is reaching the soft limit, it is ok to continue to use for another 7 days.za

## /DATA

/DATA dir is in the hdd of master node, which has size of **2T** in total. Every users has read and write permission in this dir. The best practice is to firstly create a subdir in /DATA with the username for later use. Large data file should be stored in this dir.

## /tmp

In general, the IO data files are recommended to save in `/DATA/<user>` dirs. However, these dirs are living in the master node, and other computation nodes access them via nfs, with a speed of order 100MByte/s, slower than local IO. Therefore, if IO speed is critical for your task, you can use /tmp dirs in computation nodes. These /tmp dir are living in the local SSD of each computation nodes individually which has size **500GB**. Note that /tmp in different nodes are different, they are not shared via LAN. If you choose /tmp as output file path, then after finish of the task, you need to trasfer the output files back to the master nodes. Or equivalently, copy files from /tmp to /DATA.

## /BACKUP

This dir is in another hdd of master node, which also has **2T** size in total. This dir is for important backups and normal users only have read permissions without write permission.