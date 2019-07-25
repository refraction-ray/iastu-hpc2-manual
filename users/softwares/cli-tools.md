In this part, we will give brief tutorial on some useful CLI tools.

## tmux

Tmux is installed on all nodes. It can keep some interactive sessions running even the ssh connection is off.

The very basics of tmux (for more information and commands, please google or `man tmux`).

`tmux`: open a new session of shell

`Ctrl-B D`: detach the tmux session and go back to the ssh bash shell

`Ctrl-D`: kill the tmux shell session

`tmux ls`: list all active tmux sessions

`tmux attach [-t no or name]`: go back to tmux session given no.

`Ctrl+B [`: move the screen in tmux session, enter vim mode

`q`: quit the above vim mode

`Ctrl+B $`: rename current session