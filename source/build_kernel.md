# Build the Popcorn Linux Kernel

## Running Popcorn Linux on Virtual Machines (QEMU)
We can run Popcorn Linux on real x86_64 and arm64 machines connected using ConnectX-4 InfiniBand. However, it's easier to set up the environment with QEMU virtual machines. This guide describes how to set up a testing environment for Popcorn Linux using QEMU VMs.

```
--------------     --------------
|   Node 0   |     |   Node 1   |
|            |     |            |
| linux-x86  |     | linux-arm  |
|  10.2.0.2  |     |  10.2.1.2  |
--------------     --------------
|  qemu-x86  |     |  qemu-arm  |
---------------------------------
| tap0:10.2.0.1   tap1:10.2.1.1 |
|                               |   ---------------
|      host (x86, iptables)     |   |   Gateway   |
|  <Your NIC> e.g.,192.168.1.6  |---| 192.168.1.1 |--- Internet
---------------------------------   ---------------
```

Basically, we will run two VMs - one for x86 and one for arm - on a x86 host, where the host and VMs are connected over Ethernet. To ease the testing, we will compile the kernel at the host (not inside the VMs) and boot VMs using the kernels on the host's file system. The examples and commands are based on Ubuntu 18.04.

### Pre-requisites
i) Install dependency packages:
```
❯ sudo apt-get update
❯ sudo apt-get install build-essential libssl-dev libncursesw5-dev git curl bc bridge-utils libelf-dev
❯ sudo apt-get install qemu-system-x86 qemu-system-arm gcc-aarch64-linux-gnu
```
ii) Download QEMU images:

[popcorn-VMs-ubuntu2004-4G.tar.gz](https://drive.google.com/file/d/1C7_h5K_fasDcPGQhZvMxjEcqapxNOUFF/view?usp=sharing) (807MB downloaded; 4GB x86.img, 4GB arm.img after extract the tarball).

iii) Configure host network

Use the [init_tap_network.sh](https://raw.githubusercontent.com/xjtuwxg/HeterSec/master/scripts/init_tap_network.sh) script to set up the `tap0`/`tap1 `interfaces and enable IP forwarding on the host machine (`eth0` is your ethernet NIC name):
```
❯ ./init_tap_network.sh eth0
```
Now you should be able to see `tap0` and `tap1` on your host machine:
```
❯ ip a
... ...
4: tap0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN group default qlen 1000
    link/ether 7a:2f:e6:f7:39:a9 brd ff:ff:ff:ff:ff:ff
    inet 10.2.0.1/24 scope global tap0
       valid_lft forever preferred_lft forever
5: tap1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN group default qlen 1000
    link/ether 1a:32:e6:c4:34:c4 brd ff:ff:ff:ff:ff:ff
    inet 10.2.1.1/24 scope global tap1
       valid_lft forever preferred_lft forever
```
More detail can be found here to set up the host network.

## Build the Popcorn Linux kernel from the source
Download the source code:
```
❯ git clone --depth=1 -b main --single-branch https://github.com/ssrg-vt/popcorn-kernel.git linux-x86
❯ cp -r linux-x86 linux-arm/
❯ ls
arm.img  x86.img  build  init_tap_network.sh  linux-arm  linux-x86  popcorn-compiler
```

Build the x86_64 kernel and arm64 kernel respectively:
```
❯ cp linux-x86/kernel/popcorn/configs/config-x86_64-qemu linux-x86/.config
❯ make -C linux-x86 -j8
```

```
❯ cp linux-arm/kernel/popcorn/configs/config-arm64-qemu linux-arm/.config
❯ CROSS_COMPILE=aarch64-linux-gnu- ARCH="arm64" make -C linux-arm -j8
```

## Boot the VMs
Boot the VMs with newly built kernels and the `tap` network:
```
sudo qemu-system-x86_64 \
    -enable-kvm -cpu host -smp 2 -m 4096 -nographic \
    -drive id=root,media=disk,file=x86.img \
    -net nic,macaddr=00:da:bc:de:00:13 -net tap,ifname=tap0,script=no \
    -kernel linux-x86/arch/x86/boot/bzImage \
    -append "root=/dev/sda rw console=ttyS0" \
    -pidfile vm0.pid 2>&1 | tee vm0.log
```
```
sudo qemu-system-aarch64 \
    -machine virt -cpu cortex-a57 -smp 2 -m 4096 -nographic \
    -drive id=root,if=none,media=disk,file=arm.img \
    -device virtio-blk-device,drive=root \
    -netdev type=tap,id=net0,ifname=tap1 \
    -device virtio-net-device,netdev=net0,mac=00:da:bc:de:02:11 \
    -kernel linux-arm/arch/arm64/boot/Image \
    -append "root=/dev/vda rw console=ttyAMA0" \
    -pidfile vm1.pid 2>&1 | tee vm1.log
```
### Setup the VMs' network and the message layer
The Popcorn Linux kernel relies on a message layer for fast communication among Popcorn Nodes. The message layer is implemented as a loadable kernel module and has to be loaded before running Popcorn applications. To run that, first configure the IP addresses for the nodes:
```
popcorn@x86:~$ cat /etc/popcorn/nodes
10.2.0.2
10.2.1.2
```
```
popcorn@arm:~$ cat /etc/popcorn/nodes
10.2.0.2
10.2.1.2
```
Next, copy the kernel module to the nodes:
```
linux-x86 ❯ scp msg_layer/msg_socket.ko popcorn@10.2.0.2:
linux-arm ❯ scp msg_layer/msg_socket.ko popcorn@10.2.1.2:
```
Install the kernel modules on the x86 VM first and then the arm VM:
```
popcorn@x86:~$ sudo insmod ./msg_socket.ko
[   22.155240]  *  0: 10.2.0.2
[   22.156029]     1: 10.2.1.2
[   22.156744]
```
```
popcorn@arm:~$ sudo insmod ./msg_socket.ko
[   48.433928]     0: 10.2.0.2
[   48.434146]  *  1: 10.2.1.2
[   48.434278]
[   48.465160]   1 identified as arm
[   48.465382]   0 identified as x86
popcorn@arm:~$
```
You could see the similar message from x86 console:
```
[  132.347344]   0 identified as x86
[  132.348349]   1 identified as arm
```
Congratulation! Now, the VMs are connected over the messaging layer, and ready to migrate threads!
