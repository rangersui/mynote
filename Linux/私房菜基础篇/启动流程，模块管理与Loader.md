

[toc]

# 查看启动流程

```bash
dmesg
```

# Linux启动流程

启动的过程中启动引导程序（Boot loader）使用的软件可能不一样，目前各大发行版采用grub2，早期使用grub或LILO。

以个人电脑使用的Linux为例，当按下开机后，计算机硬件主动读取BIOS或UEFI BIOS加载硬件信息以及进行硬件系统自我测试，之后系统会主动读取第一个可启动的设备（由BIOS设置），此时可以读入启动引导程序了。启动引导程序可以指定使用哪个内核文件，并实际加载内核到内存当中解压缩与执行。此时内核就能够开始在内存中活动，并检测所有硬件信息与加载适当的驱动程序来使整台主机开始运行。此时内核就能够开始在内存中活动并检测所有硬件信息与加载适当的驱动程序来使整台主机运行，等到内核检测硬件与加载驱动程序完毕后，操作系统就i开始运行了。

- 加载BIOS的硬件信息与进行自我检测（自检），并根据设置取得第一个可启动的设备
- 读取并执行第一个启动设备内MBR的启动引导程序（grub2，spfdisk等程序）
- 根据启动引导程序的设置加载Kernel，Kernel会开始检测硬件与加载驱动程序
- 在硬件驱动成功后，Kernel会主动调用systemd程序，并以default.target流程启动：
  - systemd执行sysinit.target初始化系统及basic.target准备操作系统
  - systemd启动multi-user.target下的本机与服务器服务
  - systemd执行multi-user.target下的/etc/rc.d/rc.local文件
  - systemd执行multi-user.target下的getty.target及登陆服务
  - systemd执行graphical需要的服务

> rc代表run commands，详见[What does the “rc” stand for in /etc/rc.d?](https://unix.stackexchange.com/questions/111611/what-does-the-rc-stand-for-in-etc-rc-d)

# Systemd的单元

参考文献：[linux的服务器单元与目标](https://blog.csdn.net/qq_25744595/article/details/84949417)

扩展阅读：[systemd for Administrators](http://0pointer.net/blog/projects/systemd-for-admins-1.html)

## 单元类型

| 单元类型  | 单元后缀   | 说明                                              |
| --------- | ---------- | ------------------------------------------------- |
| service   | .service   | 描述一个服务（运行由systemd控制的进程的基本单元） |
| socket    | .socket    | 描述一个套接字（每个socket都有个对应的服务）      |
| device    | .device    | 描述一个内核标识设备文件                          |
| mount     | .mount     | 描述一个文件系统挂载点                            |
| automount | .automount | 描述一个文件系统自动挂载点                        |
| swap      | .swap      | 描述一个内存交换设备或文件                        |
| path      | .path      | 描述一个系统中的文件或目录                        |
| timer     | .timer     | 描述一个定时器                                    |
| snapshot  | .snapshot  | 用于保存一个systemd的状态                         |
| scope     | .scope     | 使用systemd的总线接口以编程方式创建外部进程       |
| slice     | .slice     | 描述基于Cgroup的一组通过层次阻止的管理系统进程    |
| target    | .target    | 描述一组systemd的单元                             |

## 单元配置文件的位置

| 目录                     | 描述                            | 同配置文件的应用优先级 |
| ------------------------ | ------------------------------- | ---------------------- |
| /usr/lib/systemd/system/ | 由安装RPM包发布的Systemd单元    | 最低                   |
| /run/systemd/system/     | 在运行时创建的单元              | 高                     |
| /etc/systemd/system/     | 由管理员创建和管理的Systemd单元 | 最高                   |

## 依赖关系

- 需求依赖：用Requires或Wants配置语句描述
- 顺序依赖：使用Before或After配置语句描述
- 冲突依赖：使用Conflicts配置语句描述



# BIOS、boot loader与kernel加载

## BIOS

系统启动时首先加载BIOS，并通过BIOS加载CMOS（CMOS是主板上的一块可读写的并行或串行FLASH芯片，保存BIOS的硬件配置和用户对某些参数的设定，一般通过拿掉主板上的电池，进行CMOS放电操作，从而还原BIOS设置）的信息，借助CMOS内的设置获取主机的各项硬件配置，例如CPU与接口设备的沟通频率，启动设备的查找顺序，硬盘的大小和与类型，系统时间，各周边总线是否启动PnP（即插即用设备），个接口设备的I/O地址，以及与CPU沟通的IRQ中断等的信息。

取得以上信息后，BIOS会进行启动自我检测（Power-on Self Test，POST）。然后开始执行硬件检测初始化，设置PnP（热插拔）设备，在定义出可启动的设备顺序，接下来就会开始进行启动设备的数据读取。

BIOS会指定启动的设备好让我们可以读取磁盘中的操作系统内核文件，因为不同操作系统的文件格式不相同，因此我们必须要以一个启动引导程序来处理内核加载的问题，称为boot loader。该程序一般安装在启动设备的第一个扇区，也就是MBR。

BIOS通过硬件的INT 13中断功能来读取MBR，也就是说只要BIOS能够检测到磁盘，就有办法通过INT 13这条通道来读取该磁盘的第一个扇区的MBR。（有多个硬盘可以设置第一启动设备）

## boot loader

每个文件系统（filesystem或partition）都会保留一块启动扇区（boot sector）提供操作系统安装boot loader，而通常操作系统默认都会安装一份loader到它的根目录所在的文件系统的boot sector上。因为boot loader可以转交其他loader的功能，因此可以加载其他boot sector内的sector。但是windows的loader默认不具有控制权转交功能，因此不能使用windows的loader加载linux的loader。

## 加载内核检测硬件与initramfs的功能

当boot loader开始读取内核文件后，接下来Linux就会将内核文件解压到内存中，并利用内核的功能，开始测试各个周围设备，包括存储设备，CPU，网卡，声卡等。此时Linux内核会以自己的功能重新检测一次硬件，而不一定会使用BIOS检测到的硬件信息。也就是说，内核此时才开始接管BIOS后的工作。内核文件一般放置在/boot里面并取名/boot/vmlinuz

```bash
ls --format=single-column -F /boot
```

其中vmlinuz-xxx就是内核文件，System.map-xxx为内核功能放置到内存地址的对应表。伴随着系统升级，有可能在/boot目录下存在多个kernel文件，建议最少保存两个以防需要recovery。

Linux内核时可以通过动态加载内核模块的，这些模块被存放在/lib/modules/目录内。由于模块放置到此盘根目录内（/lib不可以与/放在不同的硬盘分区），因此在启动过程中内核必须要被挂载到根目录，这样才能够读取内核模块提供的加载驱动的功能。启动过程中根目录是以只读的方式挂载的。

一般来说，Linux发行版会将非必要的功能可以编译成为模块的内核功能，编译成为模块。因此USB,SATA,SCSI等磁盘设备的驱动程序通常是以模块的方式来存在的。内核接管系统并检测硬件及尝试挂载根目录获取额外的驱动程序。但是内核需要加载磁盘的驱动程序才能挂在根目录，但是磁盘驱动程序存在于/lib/modules内，因此通过虚拟文件系统来处理这一问题。

### 虚拟文件系统

虚拟文件系统（Initial RAM 或 Initial RAM Filesystem）一般使用的文件名为/boot/initrd或/boot/initramfs，这个文件可以通过boot loader来加载到内存中，然后这个文件会被解压缩并且在内存当中模拟成一个根目录，且此模拟在内存当中的文件系统能够提供一个可执行的程序，通过该程序来加载启动过程中最需要的内核模块，通常时USB,RAID,LVM,SCSI等文件系统与磁盘接口的驱动程序。等待加载完成后，会帮助内核重新调用systemd来开始后续的正常启动流程。

boot loader可以加载kernel与initramfs，然后在内存中让initramfs解压缩成为根目录，内核就可以借此加载适当的驱动程序，最终释放虚拟文件系统，并挂在实际的根目录或文件系统，从而开始后续的正常启动流程。更详细的initramfs说明，使用man initrd查看。

initramfs就是一个小型的根目录，这个小型根目录里面也是通过systemd进行管理，同时查看default.target的链接，会发现这个小型系统也是通过basic.target,sysinit.target等硬件检测，内核功能启用的流程，然后开始让系统顺利运行。最终才卸载initramfs的小型文件系统，实际挂在系统的根目录。initramfs仅时带入启动过程会用到的内核模块而已。如果在initramfs寻找modules关键词，就会发现主要的内核模块大概是SCSI,virtio,RAID等和磁盘相关性比较高的模块。除非SATA的模块被直接编译到内核中，没有initramfs，Linux是不能顺利启动的。

# 第一个程序systemd

当内核加载完毕，进行完硬件检测与驱动程序加载后，硬件已经准备就绪，内核会主动地调用第一个程序systemd。systemd最主要的功能就是准备软件的执行环境，包括系统的主机名，网络设置，语言设置，文件系统格式以及其他服务的启动等。而所有的操作都会通过systemd的默认启动服务集合，亦即是 /etc/systemd/system/default.target 来规划。另外systemd以及舍弃了沿用多年的System V的运行级别（run level）。

runlevel和systemd目前是同时存在的。使用init会调用rc.d脚本，里面是link文件指向/etc/init.d中的脚本。

> rc.d代表run commands的目录，.d是相对于单文件的而言的，用于消除歧义

## 使用default.target进入启动程序分析

> 以下目录中/usr/lib/systemd/system/目录并不能被访问到而是通过/lib/systemd/system访问。在/usr/lib/systemd/user/目录中\*.target软链接到/lib/systemd/system/\*.target

可以是作为默认的操作环境（default.target）的主要项目有：multi-user.target以及graphical.target这两个。当然还有某些比较特殊的操作环境，包括rescue.target,emergency.target,shutdown.target以及在initramfs里的initrd.target

但是过去的System V使用的是一个称为runlevel的概念年启动系统，systemd为了兼容旧式System V的操作行为，将runlevel与操作系统环境结合，可以使用以下命令查询。

```bash
ll -d /lib/systemd/system/runlevel*.target | cut -c 28-
```

对应关系为：

| System V   | systemd                             |
| ---------- | ----------------------------------- |
| init 0     | systemctl poweroff                  |
| init 1     | systemctl rescue                    |
| init [2-4] | systemctl isolate multi-user.target |
| init 5     | systemctl isolate graphical.target  |
| init 6     | systemctl reboot                    |

### systemd处理流程

在取得了/etc/systemd/system/default.target这一默认操作界面的设置之后链接到/usr/lib/systemd/system/这个目录下去取得multi-user.target或graphical.target这两个其中的一个（正常进入linux操作环境的情况下），假设使用graphical.target，接下来systemd会去找两个地方的设置：

- /etc/systemd/system/graphical.target.wants/ : 用户设置加载的unit
- /lib/systemd/system/graphical.target.wants/ : 系统默认加载的unit

查看/lib/systemd/system/graphical.target文件可以看到graphical.target必须要完成multi-user.target之后才能够进行，而进行完graphical.target之后，还得启动display-manager.service才能运行同样的可以查看multi-user.target要执行哪一些项目。

通过以下命令获得graphical.target所有的启动依赖

``` bash
systemctl list-dependencies graphical.target
```

可以看到以下启动顺序：

- local-fs.target和swap.target：挂载本机/etc/fstab 里面所规范的文件系统与相关的内存交换分区
- sysinit.target: 检测外围硬件驱动程序与防火墙相关任务
- basic.target: 加载主要的外围硬件驱动程序与防火墙相关任务
- multi-user.target: 其他一般系统或网络服务的加载
- 图形界面相关服务如gdm.service等其他服务的加载

## 执行sysinit.target初始化系统，basic.target准备系统

### sysinit.target

```bash
systemctl list-dependencies sysinit.target
```

服务可以归类为以下：

- 特殊文件系统设备的挂载：

  - dev-hugepages.mount

  - dev-mqueue.mount

    主要挂在和内存分页使用与消息队列的功能，挂在成功后会在/dev下建立/dev/hugepages/,/dev/mqueue/等目录

- 特殊文件系统的启用：包括磁盘阵列，网络驱动器（iscsi），LVM文件系统，文件系统对照服务（multipath）等，也会在这里被检测和使用到；

- 启动过程中的信息传递与动画执行：使用plymouthd服务搭配plymouth命令来传递动画与信息

- 日志式日志文件的使用：就是systemd-journald这个服务的启用

- 加载额外的内核模块：通过/etc/modules-load.d/*.conf文件的设置，让内核额外加载管理员需要的内核模块

- 加载额外的内核参数设置：包括/etc/sysctl.conf以及/etc/sysctl.d/*.conf内部设置

- 启动系统的随机数生成器：帮助系统进行密码加密演算

- 设置终端的字体

- 启动动态设备管理器：udevd。用在动态对于实际设备读写与设备文件名的一个服务。

### basic.target

```bash
systemctl list-dependencies basic.target
```

- 加载alsa音效驱动程序
- 加载firewalled防火墙（CentOS）
- 加载CPU的微指令功能
- 启动预设值SELinux的安全上下文
- 将目前启动过程产生的启动信息写入到/var/log/dmesg中
- 由/etc/sysconfig/modules/*.modules及/etc/rc.modules加载管理员指定的模块
- 加载systemd支持的timer功能

## 启动multi-user.target下的服务

经过上面两个步骤，系统从可读写到可使用，接下来开始启动主机服务。这些服务则大多附属于multi-user.target环境下，可以通过/etc/systemd/system/ulti-user.target.wants/查看默认要被启动的服务

一般服务启动的脚本存放在:

- /lib/systemd/system/ (系统默认的服务启动脚本设置)

- /etc/systemd/system/ (管理员自己开发与设置的脚本设置)

而用户针对主机的本地服务与服务器网络服务的各项unit若要enable的话，就是将它存放在/etc/systemd/system/multi-user.target.wants/目录下进行链接，可以使用以下命令实行同等操作：

```bash
systemctl disable *.service
systemctl enable *.service
```

实质上就是删除或建立软链接

### 兼容System V的rc-local.service

在过去，系统启动成功后还要额外执行某些程序，可以将程序命令或脚本的绝对路径写入到/etc/rc.d/rc.local文件。新的systemd机制中，建议直接写一个systemd的启动脚本配置到/etc/systemd/system下面，然后使用systemctl enable的方式来设置启用它，而不直接使用rc.local

在Ubuntu中目录/etc/存在/etc/rc[0-5].d六个目录，分别对应不同的runlevel，目录下直接放置服务的软链接，每次运行该运行级时会运行目录下链接

- 目录下文件名称带K代表该服务在此runlevel下会被kill，而S代表该服务应当被启动！

### 提供tty界面与登陆服务

由于服务是同步运行的，如果getty服务先启动完成，有可用的终端让你尝试登陆系统，但是systemd-logind.service或者systemd-user-sessions.service服务尚未执行完毕的话，还是无法登录系统。

## 启动graphical.target下面的服务

在这里，主要是加载图形界面的管理程序例如window display manager（WDM）等，实际上让用户可以登陆的服务是gdm.service如果要回到命令行界面运行

```bash
init 3
```

需要注意的是，Ubuntu16.04输入```init 3```后进入黑屏仅有光标闪烁，此时需要```ctrl + alt + f[1-6]```才能进入到用户登陆界面。

## 启动过程中用到的主要配置文件

在sysinit.target系统初始化当中，有两个地方可以处理模块加载的问题：

- /etc/modules-load.d/*.conf: 单纯要内核加载模块的位置
- /etc/modprobe.d/*.conf: 可以加上模块参数的位置

### modprobe.d

以修改vsftpd服务的端口为例，可以修改防火墙设置nf_conntrack_ftp。因此可以将这个模块写入到系统启动流程中

```bash 
vim etc/modules-load.d/vbird.conf
nf_conntrack_ftp
```

一个模块（驱动程序）写一行，上述模块针对默认FTP端口即21，如果要设置到555，要外带参数，模块外加参数的设置方式要写入另一个地方。

### modules-load.d

```bash
vim /etc/modprobe.d/vbird.conf
options nf_conntrack_ftp ports=555
```

之后重新启动就可以顺利加载并处理这个模块了。如果不想启动测试，现在处理，可以通过以下方式查看

```bash
lsmod | grep nf conntrack ftp
systemctl restart systemd-modules-load.service
lsmod | grep nf conntrack_ftp
```

通过上述方式，就可以在启动的时候将你所需要的驱动程序加载或者调整模块的外加参数

### /etc/sysconfig

Redhat系有该文件，Debain系没有，所以Ubuntu上找不到这个文件夹

# 内核与内核模块

在整个启动的过程中，是否能能够成功地驱动我们主机的硬件设备是内核的工作。而内核一般都是压缩文件，因此在使用内核之前，需要先解压缩后才能加载到内存中。

为了应付日新月异的硬件，目前内核都是具有**可读取模块化驱动程序**的功能，即所谓的**模块化**功能。这些模块可以是由硬件开发商提供，也可能是内核本来就支持，不过新的硬件一般需要硬件开发商提供驱动程序模块。

## 内核与内核模块的存放位置

- 内核： /boot/vmlinuz 或 /boot/vmlinuz-version
- 内核解压所需 RAM Disk：/boot/initramfs (/boot/initramfs-version)
- 内核模块：/lib/modules/version/kernel 或 /lib/modules/$ (uname -r) /kernel
- 内核源代码：/usr/src/linux 或 /usr/src/kernels

当内核被顺利加载系统当中，以下信息会被记录：

- 内核版本：/proc/version
- 系统内核功能：/proc/sys/kernel/

如果有新硬件而操作系统不支持，有以下方法：

- 重新编译内核，并加入最新的硬件驱动程序源代码
- 将该硬件的驱动程序编译成为模块，在启动时加载该模块

以下介绍加载一个已经存在的模块的方法

## 内核模块与依赖性

| 目录    | 功能                                      |
| ------- | ----------------------------------------- |
| arch    | 与硬件平台有关的选项，例如CPU的等级等     |
| crypto  | 内核所支持的加密技术                      |
| drivers | 硬件驱动程序                              |
| fs      | 支持的文件系统，例如vfat，reiserfs，nfs等 |
| lib     | 函数库                                    |
| net     | 网络有关的各项协议数据，还有防火墙模块等  |
| sound   | 音效相关模块                              |

通过查看/lib/modules/(linux version)/modules.dep可以查看内核支持的模块的各项依赖性

可以通过depmod文件建立该文件

## 查看内核模块

通过lsmod命令可以查看加载的模块，内容包括：

- 模块名称
- 模块大小
- 此模块是否被其他模块使用

也就是说，模块其实有依赖性，可以通过modinfo命令了解模块的详细信息。

## 内核模块的加载与删除

如果要手动加载模块，应该使用modprobe命令，因为modprobe会主动查找modules.dep的内容，解决模块依赖性后决定需要加载哪些模块。insmod完全由用户决定加载一个完整文件名的模块，不检查依赖性。

```bash
modprobe cifs #加载模块
modprobe -c #列出所有模块
modprobe -r cifs #删除模块
```

## 内核模块的额外参数设置: /etc/modprobe.d/*conf

如果需要给内核模块加上某些参数，可以建立.conf文件，通过options带入内核模块参数

# Boot loader: Grub2

boot loader是加载内核的重要工具，没有boot loader那么内核无法被系统加载。本节介绍grub2

## 两个stage

MBR的功能有限，为了解决这个问题，Linux将boot loader的程序代码执行与设置值加载分成两个阶段完成

### stage1 执行boot loader主程序

第一阶段为执行boot loader的主程序，这个主程序必须要被安装在启动区，即是MBR或启动扇区。因为MBR实在太小了，一般只安装最小主程序而没有安装loader相关的配置文件

### stage2 主程序加载配置文件

第二阶段为通过boot loader加载所有配置文件与相关的环境参数文件（包括文件系统定义与主要配置文件grub.cfg）一般来说配置文件都在/boot下面。
