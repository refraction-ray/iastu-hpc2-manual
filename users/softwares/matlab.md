`spack load matlab` before using, preinstalled version is Matlab 2018b.

## X11 Forwarding

One should use `ssh -Y` instead of `ssh -X` for matlab to work over X11 frowarding. [reference](https://www.mathworks.com/matlabcentral/answers/101307-how-do-i-get-matlab-for-linux-to-display-its-x11-windows-on-a-mac).