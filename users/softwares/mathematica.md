No matter which way you would like to use mathematica, you should first load it by `spack load mathematica`. Besides, mathematica is a commercial software which need licence to access. The software need activation by default. For details, users should contact the administrator.

## GUI forward

When connecting to the cluster by `ssh -X`, and Xserver app is well prepared in the local desktop (see [remote desktop](../basics/connecting.md)), you can just type `mathematica` on the node, and a mathematica app will automatically open in your desktop.

This X11 forward work for both master and compute nodes. We suggest the users use compute node to forward mathematica app. The workflow is as follow

```bash
(local)  $ ssh -X <user>@<ip>
(master) $ salloc -n1 -N1 -w c2 -t 1:00:00
(master) $ ssh -X c2
(c2)     $ tmux
(c2 tmux)$ spack load mathematica
(c2 tmux)$ mathematica # now, mathematica app is open in your local desktop
(c2 tmux) # ctrl+b, d
(c2) # ctrl-d ctrl-d
```

## Remote kernel launch

In this workflow, you can use your own native mathematica app in your desktop, while using math kernel on our cluster. This approach is elegant and suitable for daily interactive use.

First, copy tunnel.sh and tunnel_sub.sh [here](https://github.com/sakra/Tunnel/tree/master/scripts) to `~/.Mathematica/FrontEnd/` on master cluster, and remember `chown +x` for these two scripts. Now configure your native desktop mathematica, by Evaluation-Kernel Configuration Options-Add. Give a kernel name and choose advanced options. Fill the following two blank with the following two lines:

Arguments to MLOpen: `-LinkMode Listen -LinkProtocol TCPIP -LinkOptions MLDontInteract -LinkHost 127.0.0.1` .

Launch command: 

```
"`userbaseDirectory`/FrontEnd/tunnel.sh" "<user>:<passwd>@<cluster_ip>:<ssh_port>" "/opt/mathematica/11.0.1/Executables/WolframKernel" "`linkname`"
```

Click on OK, and your are ready to go, just change Evaluation-Notebook Kernel-The new kenel name. In the command above, passwd(if the public key ssh login is enabled) and ssh_port(if it is default by 22) can be omitted.

You can call more kernels for parallel calculation in mathematica, just `LaunchKernels[6]`, where 6 is the number of kernels you want.

### Advanced

You may think that 56 cores in one machine is too few for your needs. Then you can use the remote kernel you are using and configured as above to call kernels in other machines of the cluster!

First of all, you should `salloc` new nodes on cluster, otherwise, the underlying ssh connection for kernel launching would be denied.

Below is the mathematica command for launching more compute kernels in other machines:

```mathematica
Needs["SubKernels`RemoteKernels`"]

$RemoteCommand = 
 "\"" <> $UserBaseDirectory <> 
  "/FrontEnd/tunnel_sub.sh\" \"`1`\" \
\"/opt/mathematica/11.0.1/Executables/MathKernel\" \"`2`\""

kernel = RemoteMachine["<hostname>", 2, LinkHost -> "127.0.0.1"]
(* hostname in our case, is cn depending on your salloc result. 2 is the kernel number on hostname, if the user name is the same, which is the case in our HPC, <user>@ part can be omitted, a cn for host is enough *)

LaunchKernels[kernel]

ParallelTable[$MachineName, {i, 1, Length[Kernels[]]}] (* A test on real parallel fashion, you may see a mix of master and cn *)

CloseKernels[{19, 20, 21}] (*close kernels 19,20,21 *)

CloseKernels[] (*close all kernels*)
```

For more details, please refer on this [manual](https://github.com/sakra/Tunnel/blob/master/MANUAL.md).

**Possible issue:** As long as your desktop go offline or change network, the variables in memory of mathematica seem to vanish.

## Sbatch job submission

One can use sbatch to submit mathematica tasks. The sbatch files is like below:

```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --tasks-per-node=2

source /etc/spack-load
spack load mathematica

math -noprompt -run "<<task.m"
```

The command to run mathematica is  `math -nopromt -run "<<task.m" `. task.m is a file with mathematica commands and **remember to add `Exit[]` at the end**.

```mathematica
(* task.m *)
Print[1+1];
Exit[]; (*important for gracefully end the task!*)
```

## CLI math interactive session

This is not the recommended way to use mathematica on the cluster. Anyway, you can type `math`, and use the interactive command line interface just like mathematica with GUI. Combined with tmux, it is also possible to run some tasks by this approach. But we strongly suggest you dont use mathematica in this way for time-consuming jobs.