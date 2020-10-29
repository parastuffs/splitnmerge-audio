# Split'n'Merge Audio
The aim of this project is to find a way to merge different sources of audio into a virtual mic.

## Jack
First step: try to install and run a JACK server.

https://wiki.archlinux.fr/Jack


If you find the message
```Could not open component .so '/usr/lib/jack/jack_firewire.so': libffado.so.2: cannot open shared object file: No such file or directory```
when starting JACK daemon, you might need to install a firewire driver (even if you don't have a firewire input): `libffado`.
