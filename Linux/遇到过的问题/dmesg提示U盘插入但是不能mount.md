# 问题复现

给系统换了一个新的rootfs，重启后dmesg显示有U盘插入

```bash
[    2.234865] usb 1-1: New USB device found, idVendor=30de, idProduct=6545
[    2.241594] usb 1-1: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[    2.248726] usb 1-1: Product: TransMemory
[    2.252738] usb 1-1: Manufacturer: KIOXIA
[    2.256745] usb 1-1: SerialNumber: C03FD5F7A876E4B1A31108B5
[    2.263391] usb-storage 1-1:1.0: USB Mass Storage device detected
[    2.270078] scsi host0: usb-storage 1-1:1.0
```

但是进入/dev后不能找到任何设备

```bash
/dev # ls
null  pts
```

挂载设备失败

```bash
/ # mkdir /mnt/sda
/ # cd /mnt/
/mnt # ls
sda
/mnt # cd ..
/ # mount /dev/sda1 /mnt/sda
mount: mounting /dev/sda1 on /mnt/sda failed: No such file or directory
```

# 解决方案

Support for the devtmpfs filesystem is controlled by kernel configuration variable: CONFIG_DEVTMPFS. It is not enabled in the default configuration of the ARM Versatile PB, so if you want to try out the following using this target, you will have to go back and enable this option. Trying out devtmpfs is as simple as entering this command:

```
# mount -t devtmpfs devtmpfs /dev
```

You will notice that afterward, there are many more device nodes in /dev. For a permanent fix, add this to /etc/init.d/rcS:

```
#!/bin/shmount -t proc proc /procmount -t sysfs sysfs /sysmount -t devtmpfs devtmpfs /dev
```

If you enable CONFIG_DEVTMPFS_MOUNT in your kernel configuration, the kernel will automatically mount devtmpfs just after mounting ...
