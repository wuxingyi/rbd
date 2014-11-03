1. install ceph and copy ceph.conf and keyring(eg.ceph.client.admin.keyring) to /etc/ceph
2. create a rbd disk in your ceph cluster:

    	qemu-img create -f raw rbd:wuxingyi/firstdisk 10G
3. 如果需要联通网络，则需要首先在host上建一个网桥，并将一块物理网卡加入其中，另外需要加入虚拟网卡的启动和删除脚本，创建/etc/qemu-ifup和/etc/qemu-ifdown两个脚本，形如：
 
    	[root@ceph-admin for_rbd]# cat /etc/qemu-ifup 
    	#!/bin/sh 
    	switch=br1001 
    	ifconfig $1 up 
    	brctl addif ${switch} $1
    	[root@ceph-admin for_rbd]# cat /etc/qemu-ifdown 
    	#!/bin/sh 
    	switch=br1001 
    	brctl delif ${switch} $1 
    	ifconfig $1 down 
    	#ip link set $1 down 
    	#tunctl -d $1

4. 请千万注意，此处需要支持rbd版本的qemu，centos6.5源自带的版本不支持，请找我们已经发布的qemu版本。centos6.4_20G.qcow2镜像我也做好了，有需要可以找我要。

	    /usr/libexec/qemu-kvm -m 1024 -smp 4 --enable-kvm -vnc :0 -drive 
	    if=none,id=drive0,cache=none,format=qcow2,file=./centos6.4_20G.qcow2 -device virtio-blk,drive=drive0,bootindex=1 -drive format=raw,file=rbd:xingyiwu/firstdisk,id=drive1,if=none,cache=none -device virtio-blk,drive=drive1 -net nic,netdev=foo -netdev tap,vhost=on,id=foo &
    
5. 进入系统后首先应该好配置网络，一个版本是：

		[root@CentOS vdb]# vim /etc/sysconfig/network-scripts/ifcfg-eth0 
		DEVICE=eth0 
		ONBOOT=yes 
		BOOTPROTO=static 
		TYPE=Ethernet 
		IPADDR=10.0.0.100 
		GATEWAY=10.0.0.1

6. 此时fdisk -l可以看到之前创建的ceph rbd块设备(/dev/vdb)：

		Disk /dev/vdb: 21.5 GB, 21474836480 bytes
		16 heads, 63 sectors/track, 41610 cylinders
		Units = cylinders of 1008 * 512 = 516096 bytes
		Sector size (logical/physical): 512 bytes / 512 bytes
		I/O size (minimum/optimal): 512 bytes / 512 bytes
		Disk identifier: 0x00000000
 
7. 进行fdisk格式化，mkfs和mount后，可以看到：

		[root@CentOS ~]# mount 
		/dev/vda2 on / type ext4 (rw) 
		proc on /proc type proc (rw) 
		sysfs on /sys type sysfs (rw) 
		devpts on /dev/pts type devpts (rw,gid=5,mode=620) 
		tmpfs on /dev/shm type tmpfs (rw) 
		/dev/vda1 on /boot type ext4 (rw) 
		/dev/vdb on /vdb type ext4 (rw) 
		none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)

8. 修改/etc/fstab,加入以下一行,以后每次启动都将自动mount：

		/dev/vdb /vdb ext4 defaults 0 0
9. 如果需要更多disk，修改qemu命令行即可。
