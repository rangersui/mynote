[toc]

# Bootrom

Bootrom是一小片掩膜ROM或写保护闪存嵌入于处理器芯片当中。其中包含当处理器通电/重置时最早被执行的代码。

- iPhone boot ROM： 嵌入于掩膜ROM并且不能被更改。从flash或者USB（DFU模式下）加载下一阶段的boot loader并且通过内置的RSA实现验证签名。
- 德州仪器OMAP4 boot ROM：能从flash，外部存储，SD/MMC，USB或UART加载用户代码。提供为后一阶段使用的功能（cache/TLB管理等）。

# Bootloader

Bootloader负责寻找并加载最终在芯片上运行的OS或固件。和bootrom的主要区别是它一般储存在可写flash上并且能被升级或替换。

有时候Bootrom会行使bootloader的工作。比如OMAP的bootrom足够复杂所以可以直接加载并启动一个linux内核。

然而，在很多情况下需要使用一个独立的bootloader，既因为bootrom并不能够这么做，也因为需要额外的自由度。因为这个过程可以是简单的（从RAM的一个固定的flash位置加载内核并跳转到那里）或者更复杂。比如U-Boot是一个mini-OS，有console，一些命令，允许中断boot进程并且修改内核命令行参数甚至从其他位置加载内核等。

当你有一个在启动前需要一些设置的操作系统时，你需要使用bootloaders。更小的微控制器比如NXP的LPC系列经常使用单片固件所以不需要bootloader（然而也可能有他们定制的bootloader）

更简单的芯片也许同时不存在bootrom和bootloader——直接从固定的地址执行指令。事实上大多数x86芯片直到今天也是这么做的——他们直接在0xFFFFFFF0处执行代码，期望芯片组已经把BIOS闪存芯片映射到那个位置。在这里你可以认为BIOS作为bootloader，尽管BIOS也向OS提供服务，类似于bootrom。

参考资料：[What's the difference between a Bootrom vs bootloader on ARM systems](https://stackoverflow.com/questions/15665052/what-is-the-difference-between-a-bootrom-vs-bootloader-on-arm-systems)

## TrustedFirmware



## u-boot



## Optee



# Busybox

精简版的shell，常用于嵌入式linux领域，构建根文件系统。

# Buildroot

Buildroot可以完成Bootloader的配置编译、内核的配置编译、根文件系统的配置编译、用户空间所需的软件组件及库的配置编译等工作。

# 根文件系统

根文件系统首先是内核启动时所mount的第一个文件系统，内核代码映像文件保存在根文件系统中，而系统引导启动程序会在根文件系统挂载之后从中把一些基本的初始化脚本和服务等加载到内存中去运行。