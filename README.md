# Linux Kernel 调试


# 调试驱动模块

```
cat /proc/modules


add-symbol-file path/to/mymodule.ko 0xfffffffa00000000    
```

## KGDB 

内核配置参数

```
# CONFIG_STRICT_KERNEL_RWX is not set

CONFIG_FRAME_POINTER=y

CONFIG_KGDB=y

CONFIG_KGDB_SERIAL_CONSOLE=y

```
busybox 根文件系统
```
$ dd if=/dev/zero of=./busybox.img bs=1M count=100
$ mkfs.ext3 busybox.img

$ mkdir /tmp/disk
$ sudo mount -o loop busybox.img /tmp/disk
$ sudo cp -rf _install.* /tmp/disk

$ cd /tmp/disk
$ sudu mkdir dev sys proc etc lib mnt
$ sudo cp -a ~/workspace/busybox-1.27.2/examples/bootfloppy/etc/*  etc
                                                                   $ sudo vi etc/init.d/rcS
                                                                   “
#! /bin/sh

/bin/mount -a
/bin/mount -t  sysfs sysfs /sys
/bin/mount -t tmpfs tmpfs /dev
/sbin/mdev -s
”

$ cd ~/workspace
$ sudo umount /tmp/disk

```


qemu 启动
```
qemu-system-x86_64 -kernel /mnt/disk/sec/kernelesc/compiled_linux-4.4.38/linux-4.4.38/arch/x86/boot/bzImage -append "root=/dev/sda kgdboc=ttyS0,115200 kgdbwait" -boot c -hda busybox.img -k en-us -serial tcp::4321,server
```

gdb 
```
gdb vmlinux -ex 'target remote :4321
```

系统启动完成后的内核调试:如果想再中断,则要在目标机上输入,系统同样会中断,进入假死状态,等待远程gdb的连接

```
echo g > /proc/sysrq-trigger
```

gdb 不要使用pwndbg 插件，否则无法正常使用

## Qemu 

```
-s -S 
or -gdb tcp::1234 -S
```
gdbinit
```
add-auto-load-safe-path /mnt/disk/sec/kernelesc/compiled_linux-4.4.38/linux-4.4.38
file /mnt/disk/sec/kernelesc/compiled_linux-4.4.38/linux-4.4.38/vmlinux 
directory /mnt/disk/sec/kernelesc/compiled_linux-4.4.38/linux-4.4.38
target remote:1234

```

## 使用Virtualbox + KDbg 

使用该法比使用qemu 开虚拟机时，速度要快

vbox 上使用主机管道， 

```
socat -d -d /tmp/pipe PTY
```

gdb 
```
target remote /dev/pts/2
```
[详情](http://opensourceforu.com/2011/03/kgdb-with-virtualbox-debug-live-kernel/)


人生苦短，改用此法

## 调试过程问题解决

### 在断点停下来后，gdb 跳到另外一条指令j
原因 被其他线程抢占

```
set scheduler-locking step
```
scheduler-locking 有 step, on , off 三种模式

## 遇到Remote 'g' packet reply is too long

```
(gdb) disconnect
(gdb) set arch i386:x86-64
(gdb) target remote localhost:1234
```
[详情](https://wiki.osdev.org/QEMU_and_GDB_in_long_mode)

## "No symbol "remotebaud" in current context.

** "set|show remotebaud". Use "set|show serial baud" instead.


## 调试符号源码

 linux-image-<release>-dbgsym=<release>.<version>
linux-image-4.4.0-112-generic_4.4.0-112.135_amd64.deb

https://gist.github.com/NLKNguyen/2fd920e2a50fd4b9701f
