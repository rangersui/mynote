[toc]

# 编译

准备tool chain，并解压到/opt/cross-arm/下，软连接arm64到tool-chain的文件夹

```bash
ln -s xxx xxx
```

解压linux源码，export编译环境

```
export CROSS_COMPILE=/opt/cross-arm/arm64/bin/aarch64-linux-gnu-
```

编译源码

```bash
make ARCH=arm64 xxx_defconfig
make ARCH=arm64 Image dtbs
```

编译结果为：

- linux-source/arch/arm64/boot/Image （内核可执行文件）
- linux-source/arch/arm64/boot/dts/xxx/xxx.dtb （设备树二进制文件）

然后把flashmastr-src解压，把编译好的Image和dtb放到Res/xxx/kernel/EMMC下，Image重命名为vmlinux.bin

其中projects/XXX_EMMC_SDK是项目配置文件，rootfs_path需要修改为...rootfs.4_1.0001_64

在该文件下，运行./gen_upd_pkg.sh -b projects/XXX_EMMC_SDK

# 替换内核

插入U盘，找到flashmastr-src/output/package/images下的boot.img，然后通过U盘拷贝。

在机器上找到插入的U盘并挂载。

使用`dd`来把boot.img覆盖到mmcblk0.boot1

```bash
dd if=boot.img of=/dev/mmcblk0.boot1
```

如果没有mmcblk0，可以通过偏移的方式写入：

```bash
dd if=boot.img of=/dev/mmcblk0 seek=4096  bs=512
# 4096代表偏移4096个blocks，block是相对于bs来说的，默认bs是512bytes
```

`dd`也能制作镜像文件

```bash
dd if=/dev/zero of=100M.img bs=1M count=100
# 创建一个空的100M的镜像文件100M.img,100M.img数据全为0
```

## 在U-boot下替换内核

uboot下直接用mmc write 替换 boot.img

新文件: new.img (22419456 Bytes)

MMC分区信息如下: 

mmcblk0:6M(reserved),1M(env),20M(mboot),180M(reserved_data),48M(recoveryA),48M(recoveryB),**48M(boot)** ...

要替换的boot image位于mmcblk0p7: 48M(boot)

计算偏移量 6+1+20+180+48+48=303MB=317718528 Bytes = 620544 Blocks(512Bytes/Block) = 97800 (hex offset)

计算Block Count (基于new.img) 22419456 Bytes = 43788 Blocks (512Bytes/Block) = AB0C (hex count)

reboot进入uboot: xxx-EMMC#

```bash
mmc dev 0 0 #准备写入user partition
dcache off
usb start
fatload usb 0 8000000 kernel.img #从USB设备0读入到内存0x8000000
mmc write 8000000 7B800 A3C5 #在offset M处写入kernel.img
```

