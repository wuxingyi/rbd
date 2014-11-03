## This repo is used to track my work with ceph/rbd and qemu/rbd.

According to the ceph website, ceph is a unified sotrage system, providing block storage, object sotrage and filesystem interfaces. This repo is mainly concentred on block storage interfaces. Qemu is an emulator and virtualizer. Qemu use a ordinary file as the virtual machine's hard disk, a rbd disk is used by qemu to connect to ceph cluster. Ceph provides rbd interfaces which can be used by qemu's block layer through librbd.
See more on:

1. [http://ceph.com/](http://ceph.com/ "ceph")
2. [http://wiki.qemu.org/Main_Page](http://wiki.qemu.org/Main_Page "qemu")
