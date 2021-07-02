[toc]

# 认识内核和获取内核源码

内核是整个操作系统的最底层，负责了整个硬件的驱动，以及提供各种系统所需的内核功能，包括防火墙，是否支持LVM或磁盘配额等文件系统功能，这些都是内核负责的。所以我们会看到MBR的loader加载内核文件来驱动系统的硬件，也就是说如果内核不识别新硬件，那么新硬件是不能被使用的

## 什么是内核

计算机完成的所有功能，都需要有内核支持，内核其实就是系统上的一个文件，这个文件包含了驱动主机各项硬件的检测程序与驱动模块。内核文件通常被命名为/boot/vmlinuz-xxx，一台主机上可以有多个内核文件，只是启动的时候竟能选择一个来加载。甚至可以在一个linux发行版上放置多个内核，通过这些内核做成多重引导。

### 内核模块

为了应对不断更新的硬件，Linux使用模块化设置，将一些不常用的类似驱动程序独立出内核，编译为模块，然后内核就可以在系统正常运行的过程当中加载这个模块。这样一来，就不需要修改内核的前提之下编译出适当的内核模块，并且加载他。内核模块一般放在/lib/modules/$(uname -r)/kernel当中。

### 内核编译

内核通过内核源代码编译而成，所以要获得内核源代码，然后通过编译得到。

### 更新内核的目的

内核代码根据需要进行编译，根据需要修改。

- 新功能的需求

  需要新功能，而这个新功能在新的内核中才有，只能重新编译我的内核。一些新的主板芯片组也需要新的内核支持之后才能 正常有效的工作。

- 原本内核太过臃肿

  内核编译了很多不需要的功能，可以通过重新编译来取消该功能。

- 与硬件搭配的稳定性

  由于原本linux内核大多数是针对Intel CPU开发的，所以如果CPU是AMD的，有可能会让系统不太稳定。此时重新编译内核让系统获取正确的模块。

- 其他需求（如嵌入式系统）

  当需要特殊的环境需求时，需要自行设计内核。

# 内核编译前的预处理和内核功能选择

内核编译前有一些预备工作。

## 安装库

```bash
sudo apt-get install libncurses-dev flex bison openssl libssl-dev dkms libelf-dev libudev-dev libpci-dev libiberty-dev autoconf
```

## 保持干净源代码：make mrproper

如果是第一次编译，但是不清楚下载下来的源代码是否有目标文件以及相关的配置文件在，因此通过以下方式处理

```bash
cd /usr/src/kernels/linux-3.10.89/
make mrproper
```

这个操作会将以前进行过的内核功能选择文件一并删除，因此只在第一次执行内核编译前才进行这个操作，其余时刻，你想删除前一次编译过程的残留数据，只要执行：

```bash
make clean
```

make clean只会删除目标文件和编译过程中其他的中间文件，而不会删除配置文件。

## 选择内核功能

/boot/下存在一个名为config-xxx文件，该文件就是内核功能列表文件。该文件为Linux make config build file，内核功能列表文档。内核功能的选择，最后会在/usr/src/kernels/linux-3.10.89/下产生一个名为.config的颖仓文件，这个文件就是/boot/config-xxx的文件。可以通过以下方法建立文件：

- make menuconfig

  最常用是命令行模式下可以显示纯文本界面的方式，不需要启动X Window就能够选择内核功能选项

- make oldconfig

  通过使用已存在的./.config文件内容，使用该文件的设置值为默认值，只将新版本内核中的新功能列出让用户选择，可以简化内核功能的选择过程。对于作为升级内核源代码后的功能选择来说，是非常好的一个选项。

- make xconfig

  通过以Qt为图形界面基础功能的图形化接口显示，需要X Window的支持，例如KDE就是通过Qt来设计的X Window，如果使用KDE，可以使用此选项

- make gconfig

  以Gtk为图形界面基础功能的图形化接口显示，需要具有X Window的支持，例如GNOME就是通过Gtk来设计的X Window

- make config

  最原始的选择方法，每个选项都已列表方式一条一条地列出来选择，如果设置错误只能再次选择。

更多方式参考内核目录下的README文件。

## menuconfig的操作

- 上下箭头选择选项
- 左右箭头选择Select, Exit, Help, Save, Load
- 使用Enter进入进一步设置
- 空格修改选项

内核功能根据需求配置，不做介绍，详见原书。

## 内核编译与安装

``` bash
make -j4 clean bzImage modules
# bzImage为经过压缩的内核 modules是模块
```

可以找到关于make -j报错的问题，有可能是因为虚拟机内存不足导致的。

## 模块安装

``` bash
make modules_install
```

如果反复编译同一个内核，建议在make menuconfig的时候，把General setup内的local version修改一个新的名字。

## 安装新内核与多重内核选项（待更新）



# 额外模块编译

如果编译内核的时候发现遗漏了模块，那么需要通过以下方式来给内核导入新的模块

## 编译前事项

安装kernel-devel（Redhat）

   ```bash
   sudo apt-get install linux-kernel-headers kernel-package # for Ubuntu
   ```

