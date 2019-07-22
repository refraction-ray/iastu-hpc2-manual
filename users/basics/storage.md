**Note:** User files have no backup and no guaranteen unless special request to the administrator. So never ever store valuable files only in the cluster.

**Update:** Currently all files stored in `/home/<user>` folder would have backup in daily basis by default. **However, we still have no promise on the data in the cluster, they might be lost.** Backup important file by your own.

No matter where your files stored, it is highly recommend that all of your files are save in a directory named after your user name.

**TL;DR**

**Rule of thumb**:

* Source code and small scripts -> `/home/<user>`
* Data files as output of program -> `/DATA`
* Data files as output of program but IO speed is critical -> `/tmp`

## /home

All user directories in home dir are in the ssd of master node. And home directory is shared via nfs to all computation nodes. The SSD is in size of **500GB** in total which should be shared with OS and all users, so only small tasks are recommeded to IO within home dir. Never ever store large datafiles in home dir! The home dir is under control of quota which automatically limit users' disk usage. The current soft limit and hard limit of home disk usage is 45GB and **60GB**. When the disk usage is reaching the soft limit, it is ok to continue using for another 7 days. Use `quota -u $USER` to check the status of your disk quota.

The default permission on `/home/<user>` is 700, therefore, others cannot view the user's file, code and data.

## /DATA

/DATA dir is in the hdd of master node, which has size of **2T** in total. Every users has read and write permission in this dir.  This dir is also shared via NFS and accessible to all computation nodes. The best practice is to firstly create a subdir in /DATA with the username for later use. Large data file should be stored in this dir. Currently there is no quota limit on the usage for /DATA.

Note that the default permission on `/DATA/<user>` is 755, which means that other users have permission to read your files in this directory. This feature is designed to facilitate the data share within research group, whose data can directly acquired from DATA folder. If you don't want to share your data with others, you can mkdir within `/DATA/<user>`, such as `/DATA/<user>/private` and `chmod 700` on such folder. Then all data files stored within such private folder is not available to other users.

## /tmp

In general, the data files for task IO are recommended to save in `/DATA/<user>` dirs. However, these dirs are living in the master node, and other computation nodes access them via nfs, with a speed of order 100MByte/s, slower than local IO. Therefore, if IO speed is critical for your task, you can use /tmp dirs in computation nodes. These /tmp dir are living in the local SSD of each computation nodes individually which has size **500GB**. Note that /tmp in different nodes are different, they are not shared via LAN. If you choose /tmp as output file path, then after finish of the task, you need to transfer the output files back to the master nodes. Or equivalently, copy files from /tmp to /DATA.

## /BACKUP

This dir is in another hdd of master node, which also has **2T** size in total. This dir is for important backups and normal users only have read permissions without write permission. For normal users, they shouldn't care about this dir. And if users have some need to backup locally, they can contact the administrator for the backup, and the backup files will go here.

## /DATA2

*Currently no disk for DATA2, below is here for historical reasons.*

This directory is shared by nfs, and lives in an old machine with **9T** space in total (by hardware raid controller). Therefore, all IO in this dir would go through NFS no matter in master or computation nodes. So it is slow. Besides, the disk is aged and not under good maintenance, so the risk of data lost is high.  Moreover, the account system is different in /DATA2, so all files would lose owner attributes, and the permission control is somewaht chaotic.

Therefore, unless you have lots of unimportant data file to save, we strongly recommend you choose /DATA as the IO dir. However if the data file of your project is beyond the size of /DATA dir, /DATA2 would be the only choice.