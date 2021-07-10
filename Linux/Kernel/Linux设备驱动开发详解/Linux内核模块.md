[toc]

# 内核模块

内核模块具有以下特点：

- 本身不被编译入内核映像，控制内核大小
- 一旦被加载，就和内核其他部分完全一样

## Hello World

```c
#include<linux/init.h>
#include<linux/module.h>

static int __init hello_init(void)
{
    printk(KERN_INFO "Hello World enter\n");
    return 0;
}
module_init(hello_init);
static void __exit hello_exit(void)
{
    printk(KERN_INFO "Hello World exit\n");
}
module_exit(hello_exit);
MODULE_AUTHOR("Cao Zhuowen");
MODULE_LICENSE("GPL v2");
MODULE_DESCRIPTION("A Simple Hello World Module");
MODULE_ALIAS("a simplest module");
```

### 编译模块

```makefile
KVERS = $(shell uname -r)

# Kernel modules
obj-m += hello.o

# Specify flags for the module compilation.
#EXTRA_CFLAGS=-g -O0

build: kernel_modules

kernel_modules:
	make -C /lib/modules/$(KVERS)/build M=$(CURDIR) modules

clean:
	make -C /lib/modules/$(KVERS)/build M=$(CURDIR) clean

```

-C /lib/modules/$(KVERS)/build 指明**跳转到内核源码目录下读取那里的Makefile**；M=$(PWD) 表明然后返回到当前目录继续读入、执行当前的Makefile。其中M不是makefile的选项，是内核根目录下的Makefile中使用的变量。

#### 参考

[make -C M=](https://www.cnblogs.com/liulipeng/p/3408238.html)

[make -C M选项](https://blog.csdn.net/shenwansangz/article/details/47041651)

### 加载模块

```bash
sudo insmod hello.ko #加载模块
lsmod #查看所有在运行的模块
dmesg #查看内核日志
sudo rmmod hello #卸载模块
modinfo hello.ko #获得模块的基本信息
```

同样的也可以使用modprobe加载和卸载模块。但是需要更新modules.dep文件。先将.ko文件复制到/lib/module/'uname-r'/下，然后使用`depmod`命令更新modules.dep文件。

```bash
sudo modprobe hello
lsmod
sudo modprobe -r hello
```

## Linux内核模块程序结构

