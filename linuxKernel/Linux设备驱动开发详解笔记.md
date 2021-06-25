[toc]

# Linux内核的组成

[源代码地址](https://elixir.bootlin.com/linux/v4.0-rc1/source)

## 目录结构

- arch：包含和硬件体系结构相关的代码，每种平台占一个相应的目录，如i386、arm、arm64、powerpc、mips等。在arch目录下，存放的是各个平台以及各个平台的芯片对Linux内核进程调度、内存管理、中断等的支持，以及每个具体的SoC和电路板的板级支持代码。
- block：块设备驱动程序I/O调度
- crypto：常用加密和散列算法，还有一些压缩和CRC检验算法
- documentation：内核各个部分的通用解释和注解
- drivers：设备驱动程序，每个不同的驱动占用一个子目录，如char、block、net、mtd、i2c等。
- fs：所支持的各种文件系统，如EXT，FAT，NTFS，JFFS2等。
- include：头文件，与系统相关的头文件放置在include/linux子目录下；内核API级别头文件？
- init：内核初始化代码。start_kernal就在init/main.c文件中。
- ipc：进程间通信的代码。
- kernel：内核最核心的部分，包括进程调度、定时器等，和平台相关的一部分代码放在arch/*/kernel目录下。
- lib：库文件代码
- mm：内存管理代码，和平台相关的一部分代码放在arch/*/mm目录下
- net：网络相关代码，实现各种常见的网络协议
- scripts：主要是一个SELinux的模块
- sound：ALSA、OSS音频设备的驱动核心代码和常用设备驱动。
- usr：实现用于打包和压缩的cpio等 //cpio用来建立，还原备份档？

内核一般要做到drivers和arch的软件架构分离，驱动中不包含板级信息，让驱动跨平台。同时内核的通用部分（如kernel、fs、ipc、net等）则与具体的硬件（arch和drivers）剥离

## 内核组成部分

Linux内核主要由进程调度（SCHED）、内存管理（MM）、虚拟文件系统（VFS）、网络接口（NET）和进程间通信（IPC）5个子系统组成。

### 进程调度



