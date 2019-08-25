We will talk about how to connect to IASTU-HPC2 and how to upload/dowload files in this part.

## SSH client

To connet the cluster for tasks and computation, users should use SSH clients. Such clients are shipped with MacOS and Linux distributions by default (**terminal**). Therefore, linux and mac users need do nothing.

If users use Windows, we strongly recommended WSL (Windows Subsystem for Linux prrovided by Windows 10) for developing environment as well as ssh clients. Besides, there are various ssh client softwares in windows, like PuTTY or XSHELL.

## SSH connect

Using the user account and password the administrator gave to connect. For the ip and port part, please also refer to the administrator. The general syntax for *nix system is like

```bash
$ ssh <user>@<server_ip> -p <port>
```

Actually there are **two pairs** of ip-port to login. One (inner ip) is only accessible within Tsinghua network (or together with Tsinghua VPN such that it can be accessed from all networks) and the other one (outer ip) is accessible in all networks. The latter one is only suitable for ssh related connection, such as ssh, sftp, rsync, sshfs and so on. To utilize web service, such as jupyter, always use the former one (inner ip).

You may meet some warning prompt as below when you firstly login, just type y and continue.

```bash
The authenticity of host can't be established.
ECDSA key fingerprint is SHA256:
Are you sure you want to continue connecting (yes/no)?
```

When you firstly login the cluster, you are expected to change the password by `passwd` command. And more importantly, you are recommended to use public key instead of password to login the system later. This settings make the ssh login safer, please google "public key ssh authetication settings".

## File transfer

You can transfer files via **scp** or **rsync** which can be called via CLI enviroment or **sftp** via GUI softwares. 

* SCP to copy file from your clients to HPC server. (For copy file from HPC server to your local desktop, just reverse the order of the two path.) For copy directorys, `-r` option should set for scp.

   ```bash
   $ scp -P <port> <source_file_path> <user>@<server_ip>:<destination_path>
   ```

* Rsync is a similar file transfer tool but it is better at compare the difference between files and *sync* two pathes in different machines. It works well for directory. If the port is not the default one (22), then `-e "ssh -p <port>"` option should be set for rsync.

  ```bash
  $ rsync -avz <source_file_path> <user>@<server_ip>:<destination_path>
  ```

* There are various SFTP clients softwares for different OS. Like transmit in MacOS or WinSCP in Windows. Such sofwares have nice GUI, and easy to use just as normal FTP.

* sshfs is another CLI tool in *nix system, which is very handy to mount the remote directory locally, and do file tasks just as in local filesystem.

   ```bash
   $ sshfs <user>@<server_ip>:/remote/path /local/path
   ```

   I cannot help Windows users, maybe you could find similar things...

## Remote desktop support

The cluster comes with X11 forward support. If you want to use some softwares with GUI, you should ssh into the server with `ssh -X` option. From client side, such X11 forward is supported natively in linux. For mac, it is recommended to download **Xquatrz** software. There are similar softwares on Windows, too.

It is worthing noting such X11 forward is chained, i.e. you can also use GUI tools in compute nodes. Below is an example to use mathematica GUI from HPC.

```bash
# in your desktop
(local)$ ssh -X <user>@<server_ip>
(master)$ salloc -N1 -n1 -t 1:00:00 # alloc resource first to enable ssh to cn
(master)$ squeue # find the job in now in c1 node
(master)$ ssh -X c1
(c1)$ spack load mathematica
(c1)$ mathematica # mathematica will open in your local desktop
```



