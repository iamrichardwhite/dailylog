#Summary

##Linux Tools
###ls只显示目录

	ls -l | grep ^d

###RPM
	rpm -qa|grep -i <package name>
	rpm -e <package name>

###Disk Usage
	df -h

###Directory Size
	du -sh ./*

###fstab
[Archlinux Fstab Instruction](https://wiki.archlinux.org/index.php/Fstab_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

	# <file system>        <dir>         <type>    <options>             <dump> <pass>
	tmpfs                  /tmp          tmpfs     nodev,nosuid          0      0
	/dev/sda1              /             ext4      defaults,noatime      0      1
	/dev/sda2              none          swap      defaults              0      0
	/dev/sda3              /home         ext4      defaults,noatime      0      2

**options**: defautls

**dump**: dump 工具通过它决定何时作备份. dump 会检查其内容，并用数字来决定是否对这个文件系统进行备份。 允许的数字是 0 和 1 。0 表示忽略， 1 则进行备份。大部分的用户是没有安装 dump 的 ，对他们而言 `<dump>` 应设为 0

**pass** fsck 读取 `<pass>` 的数值来决定需要检查的文件系统的检查顺序。允许的数字是0, 1, 和2。 根目录应当获得最高的优先权 1, 其它所有需要被检查的设备设置为 2. 0 表示设备不会被 fsck 所检查。

###screen
	screen //create a new session
	screen -S <session name>
	screen -ls
	screen -r <session id>
	screen kill <session id>	
**C+a d** Detach

###.xz file
	tar xvpf test.tar.xz

###GRUB2 default entry

[GRUB2](https://fedoraproject.org/wiki/GRUB_2/zh-cn)

* Edit `/etc/default/grub` and set `grub_default=saved`
* Update `grub.cfg` with `grub2-mkconfig -o /boot/grub2/grub.cfg`
* Use `grub2-editenv list` to get current default entry
* Use `grep menuentry /boot/grub2/grub.cfg` to get all menuentry
* Use `grub2-set-default <标题或名称>` to change saved entry


###GCC-4.4

`Cannot use CONFIG_CC_STACKPROTECTOR_STRONG: -fstack-protector-strong not supported by compiler`

Because the gcc compiler is too old.

Compile gcc-4.9.3 frome source and install.

	cd gcc-4.9.3

	./contrib/download_prerequisites

	cd ..

	mkdir gcc-build-4.9.3

	cd gcc-build-4.9.3
	
	../gcc-4.9.3/configure --enable-checking=release --enable-languages=c,c++ --disable-multilib

	make -j3
	
	make install

	gcc --version
###Git and GitHub
Finished GitHub course on [WheelHouse](https://learn.wheelhouse.io/events/early-access).

Git:

	config
	init
	branch,checkout,-b,merge,-d
	add,mv,rm
	commit
	log --oneline
	reset,--soft,--mixed,--hard
	reverse
	pull,push
GitHub:

[Fork a repo](https://help.github.com/articles/fork-a-repo/)

[Syncing a fork](https://help.github.com/articles/syncing-a-fork/)

###Vim

* 显示当前文件名和目录

	:file
	Ctrl+G

* 取消高亮

	:nohl

* [将你的Vim 打造成轻巧强大的IDE](http://yuez.me/jiang-ni-de-vim-da-zao-cheng-qing-qiao-qiang-da-de-ide/)

## Linux Kernel

###Kernel Version
	uname -r

###could not find module

kernel在“make install”时“ERROR: modinfo: could not find module power_meter”

[could not find module](http://smilejay.com/2013/11/kernel-install-error-could-not-find-module/)

###Kernel Balloon
	init_vqs():Initialize VirtQueue
	balloon():Calc diff and invoke leak or fill
	fill_balloon():
		balloon_page_enqueue():allocate a new page and inserts it into the balloon page list.
			alloc_page():alloc_pages_current()
			__alloc_pages_nodemask()
		balloon_page_dequeue()

	drivers/virtio/virtio_balloon.c
		balloon()
			diff=towards_target(vb)
		towards_target()
			virtio_cread(vb->vdev, struct virtio_balloon_config, num_pages,&num_pages);
			return num_pages-vb->num_pages
	include/linux/virtio_config.h
		virtio_cread()
			*(num_pages)=virtio_cread64(vdev,offsetof(structname,member));
		virtio_cread64()
			__virtio_cread_many(vdev, offset, &ret, 1, sizeof(ret));
		__virtio_cread_many()
		vdev->config->get(vdev, offset + bytes * i,
                                          buf + i * bytes, bytes);
###virtio

PCI (Peripheral Component Interconnect) 外部设备互连总线

Support PCI devices:

* virtio-net-pci
* virtio-scsi-pci 
* virtio-balloon-pci
* virtio-serial-pci
* virtio-rng-pci

## QEMU
###Increase Space For IMG File

	qemu-img resize centos6.img +20G

Then enter guest os

	fdisk /dev/sda

Create new partition in fdisk

	mkfs.ext4 /dev/sda3
	vim /ect/fstab

Mount new partition in fstab and restart guest os
###QEMU Network Setting

需求：首先要清楚，因为我要修改QEMU代码，所以不能用virsh来自动配置，因为它采用的是qemu-kvm，进行了特殊的订制，可能会有不必要的麻烦。我需要自己`make install`之后，用`qemu-system-x86_64`来手动启动虚拟机。

结构：在Host中，创建桥接网络virbr0，并且将eth0连入此桥接网络。然后在Guest中，修改`network-scripts`，正确识别并设置QEMU的网卡e1000。最后，编写Host中的QEMU启动脚本，将Guest连入virbr0。

1. Host Setting
	1. REMEMBER TO CHECK NETWORK CABLE CONNECTION FIRST!!!我浪费了好多时间在配置脚本上，总是无法正确设置网络，结果发现网线松动了。
	2. Enter `/etc/sysconfig/network-scripts/`
	3. Add ifcfg-virbr0
	4. Edit ifcfg-eth0(maybe enp3s0,enp2s0),add it to virbr0
	5. `service network restart`
	6. `brctl show	//To ensure correctness`
2. QEMU scripts
	1. Edit runkvm for correct img path
	2. Edit qemu-ifup for correct bridge name
3. Guest Setting
	1. Run `runkvm` to start guest and enter it with root
	2. `cd /etc/udev/rules.d/`
	3. `cat 70-persistent-net.rules`
	4. PCI device e1000 is QEMU network adapter,remember its NAME(eth0/1/2) and ATTR
	5. Add eth0/1/2 scripts under network-scripts:
		1. `cd /etc/sysconfig/network-scripts/`
		2. `cp ifcfg-eth0 ifcfg-eth1(If qemu network adapter is eth0,normal setting should work,and we don't need to edit,just ifconfig to check network connection.)`
		3. `vim ifcfg-eth1`
		4. Change HWADDR to ATTR in `70-persistent-net.rules`
        5. `service network restart`
	6. Explanation: 因为之前都是在做Xen的虚拟化，所以这个镜像中配置的`network-scripts`是指定的Xen的网卡作为eth0，现在采用QEMU了，缺少对应的网络配置脚本，那么直接在`network-scripts`中加入新的脚本配置QEMU网卡就可以了。
### Setting files
	
ifcfg-virbr0

	TYPE=Bridge
	STP=yes
	BOOTPROTO=none
	NAME=virbr0
	UUID=cc49c39a-8713-4c88-b877-557a462ab738
	DEVICE=virbr0
	ONBOOT=yes
	DNS1=162.105.129.26
	DEFROUTE=yes
	IPV4_FAILURE_FATAL=no
	IPV6INIT=no
	DELAY=0
	BRIDGINS_OPTS=priority=32768
	IPADDR=172.17.2.131
	PREFIX=16
	GATEWAY=172.17.1.1
	NM_CONTROLLED=no

ifcfg-enp3s0

	BRIDGE=virbr0
	BOOTPROTO=none
	NAME=enp3s0
	UUID=cc49c39a-8713-4c88-b877-557a462ab738
	DEVICE=enp3s0
	ONBOOT=yes
	NM_CONTROLLED=no

runkvm

	#!/bin/bash
	
	qemu-system-x86_64 -enable-kvm -nographic -m 8192 -smp 3,sockets=3,cores=1,threads=1 -device e1000,netdev=net0 -netdev tap,id=net0,script=./qemu-ifup -device virtio-balloon-pci,id=balloon0,bus=pci.0 /root/richard/test/centos6.img

[QEMU network scripts](http://blog.csdn.net/cnsword/article/details/8659868)

qemu-ifup
	
	#! /bin/sh
	switch=br0
	ifconfig $1 up 
	 #ip link set $1 up
	brctl addif ${switch} $1

qemu-ifdown

	#! /bin/sh
	switch=br0
	brctl delif ${switch} $1
	ifconfig $1 down 
	#ip link set $1 down
	#tunctl -d $1
###Balloon
Virtio Balloon Device

	balloon_page(addr,defalte)
	balloon_stat_names[]
	reset_stats
	virtio_balloon_handle_output(vdev,vq)
	
	hw/virtio/virtio-balloon.c
		virtio_balloon_set_config()
			 qapi_event_send_balloon_change(vm_ram_size -
                        ((ram_addr_t) dev->actual << VIRTIO_BALLOON_PFN_SHIFT),&error_abort);

Guest Kernel通过vdev->config->get()获取config信息读取页面数目，这部分的信息由QEMU在virtio_balloon_set_config()设置

问题在于，QEMU中是从哪里调用的这个set_config()，参数中的页面数目是从哪里拿到的？

balloon set_config invoke

	qemu/build/x86_64-softmmu/hmp-commands.h
		这种全是{}，的头文件是用来做什么的？里面的赋值很像是json
		{
		.name       = "balloon",
		.args_type  = "value:M",//此处value依然为MB数
		.params     = "target",
		.help       = "request VM to change its memory allocation (in MB)",
		.mhandler.cmd = hmp_balloon,
		},
	qemu/hmp.c
		hmp_balloon()
			qmp_balloon(value, &err);//此处的value应该已经转换为字节数为单位了
	qemu/balloon.c
		qmp_balloon()
			balloon_event_fn(balloon_opaque,target);
		QEMUBalloonEvent *balloon_event_fn
		qemu_add_balloon_handler()
			balloon_event_fn=event_func;
	qemu/hw/virtio/virtio-balloon.c
		virtio_balloon_device_realize()
			ret = qemu_add_balloon_handler(virtio_balloon_to_target,virtio_balloon_stat, s);
		virtio_balloon_to_target()
			ram_addr_t vm_ram_size = get_current_ram_size();

		    if (target > vm_ram_size) {
		        target = vm_ram_size;
		    }
		    if (target) {
		        dev->num_pages = (vm_ram_size - target) >> VIRTIO_BALLOON_PFN_SHIFT;
		        virtio_notify_config(vdev);
		    }
###QEMU Documentations


Enter QEMU console
	Ctrl+a c

[不朽传奇-云计算技术背后的那些天才程序员:Qemu的作者法布里斯贝拉](http://bbs.csdn.net/topics/390736251)

[QEMU Emulator User Documentation](http://wiki.qemu.org/download/qemu-doc.html)

	"Computers as Components, Second Edition: Principles of Embedded Computing System Design", Wayne Wolf, ISBN-13: 978-0123743978
* QDev = QEMU Device Model
* QOM = QEMU Object Model
* QMP = QEMU Mechine Protocol
* QAPI: Introduce a number of changes to the internal implementation of QMP to simplify maintainability and enable features such as asynchronous command completion and rich error reporting. 
* Tiny Code Generator (TCG)

###QEMU src directories
* docs/ 包含了一些文档，说实话，对初学者来说，读这些文档压根没有头绪 
* hw/   包含了所有支持的硬件设备 
* include/  包含了一些头文件 
* linux-user/  包含了linux下的用户模式的代码 
* target-XXX/   包含了QEMU目前所支持guset端的处理器架构。包含的代码的主要功能是将该guest架构的指令翻译成TCG OP代码。也就是target-arm下的代码就是将arm架构的指令翻译成TCG OP。这些目录占了源码目录的很大一部分。
* tcg/   包含了动态翻译工具tcg的源码部分，主要是将TCG OP转化为host binary的部分。这个目录下也包含了多个架构名字命名的目录，每个目录下存放着针对该架构的代码。后续会详细介绍。 
* test/ 从名字上可以看出，应该是存放测试部分的代码。
###QEMU的机制

##Other
###NIC Advanced Setting

Enter NIC Advanced Setting by:
设备管理器->对应网卡->属性->高级

All settings following should be disabled.

> **Interrupt Moderation**
> 
> To reduce the number of interrupts, many NICs use interrupt moderation. With interrupt moderation, the NIC hardware will not generate an interrupt immediately after it receives a packet. Instead, the hardware waits for more packets to arrive, or for a time-out to expire, before generating an interrupt. The hardware vendor specifies the maximum number of packets, time-out interval, or other interrupt moderation algorithm.
> 
> The measured round-trip time for a packet is one of the most commonly used techniques to determine the network bandwidth between two endpoints. However, when interrupt moderation is enabled, receiving a packet does not generate an immediate interrupt and therefore the perceived round-trip time for a particular packet becomes larger than the average time. To allow accurate measurement of round trip time for a packet, NDIS provides the ability to disable and enable interrupt moderation on demand.

>**IEEE 802.11d**
>
>IEEE 802.11d was developed out of necessity, as initially, IEEE 802.11 was limited to only a few geographical domains. IEEE 802.11d was approved to provide recommendations for wireless communications, whereby geographical information is added to transmitted frames. This specification was applied to the frame format of beacons, probes and probe requests. 
>
>IEEE 802.11d allows a device to self-configure and operate according to the regulations of its operating country and includes parameters like country name, channel quantity and maximum transmission level.

>**TCP/UDP Checksum Offload (IPv4)**
>
>The TCP/UDP Checksum Offload (IPv4) function enables the adapter port to compute the checksum of transmitting IPv4 packets and verify the checksum of receiving IPv4 packets, taking load off from the CPU.

###SPEC2006 setting

After run `./run mcf`, got error: `ERROR: specperl: bad interpreter: No such file or directory`

Use install.sh to set up env:`./install.sh`

Restore default run script:
`mv ./run.backup ./run`

###MPI and Pthread

* 并行计算
并行可以用openmpi，也可以用mpich，二者应该是并列的。但是由于二者提供了几乎一样的命令，所以二者可以同时安装，但是不可以同时处于使用状态。

* openmpi

安装openmpi:

	sudo yum install openmpi openmpi-devel
安装后，二进制文件位于 /usr/lib64/openmpi/bin 下，动态库文件位于 /usr/lib64/openmpi/lib 下，因而实际使用的话还需要额外的配置，在 .bashrc 中加入如下语句:

	export PATH=/usr/lib64/openmpi/bin/:${PATH}
	module load mpi/openmpi-x86_64
PS：要使用 module 命令需要先安装 environment-modules 包。

* mpich

安装mpich:

	sudo yum install mpich mpich-devel
安装后，二进制文件位于 /usr/lib64/mpich/bin 下，动态库文件位于 /usr/lib64/mpich/lib 下，因而实际使用的话还需要额外的配置，在 .bashrc 中加入如下语句:

	export PATH=/usr/lib64/mpich/bin/:${PATH}
	module load mpi/mpich-x86_64
* Note
只能使用一个，我选择openmpi
C和C++编译命令不同，一个是mpicc，一个是mpicxx

* pthread

编译命令
	g++ -lrt -lpthread PI.cpp -o PI

###Hyper-V

自己做虚拟化研究，却对Hyper-V毫无了解，真是羞愧= =

Hyper-V是Windows的一个组件，开启该功能后，系统会重启进行模块安装，安装完毕后，相当于在内核中加了一个中间层：Hyper-V层。随后所有的，包括原host在内的系统都会变成guest。不同于Xen中domain0还是hypervisor，Hyper-V上原host不过是一个有着特殊权限的guest而已。

综上，Hyper-V会和VirtualBox和VMwareWorkstation冲突，因为Windows变成了guest，无法再支持VT-x了。

###Modern Operating System: Memory Management
####No Memory Abastraction
* 单进程	
* 多进程：页机制，PSW，保护位，静态重定位
* 在嵌入式系统中，依然有很多没有内存抽象
####A Memory Abstraction:Address Spaces
Two problems:protection and multi-process

* The Notion of an Adress Space:Base and Limit Registers,dynamic relocation
* Swapping:memory compaction
* Managing Free Memory
	* bitmaps
	* linked lists:first fit,next fit,best fit,worst fit,quick fit
* Virtual Memory
	* Paging:MMU,pages,page frames,present/absent bit,page fault,page table
	* Page Tables:Empty,Caching disabled,Referenced,Modified,Protection,Present/absent,Page frame number
	* Speeding Up Paging:fast,and page table is small
	
		* Translation lookaside buffers:TLB 
		* Software TLB Management:less miss,and quick deal miss;software cache
		* soft miss,hard miss
	* Page Tables for Lage Memories:multi-level page tables,inverted page tables
* Page Swapping Algorithm
	* optimal
	* NRU,FIFO,Second Chance,Clock
	* LRU,NFU,Aging
	* Working set,WSClock

###Memcached
forked memcached, and wget & compiled memcached-1.4.5
###KVM
影子页表，EPT
[Best practices for KVM](http://download.csdn.net/detail/u013159078/6715195)
通过一篇文章大致理解了KVM的内存管理机制，从GVA->GHA->HVA->HHA，先是影子页表，然后Intel加入了EPT，也就是两级MMU，硬件上解决了这一问题。
###Python教程
[廖雪峰的网站](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)
###Sublime Text
* 高分屏支持比较好，所以不用2,VS Code/Atom也是同样的问题
* Ctrl+Shift+P command plate
* install pacakge control
* Package Install: Install Package
* 输入法光标跟随:IMEsupport，并不能完美解决，但已经好了很多了。