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

-C /lib/modules/\$(KVERS)/build 指明**跳转到内核源码目录下读取那里的Makefile**；M=\$(PWD) 表明然后返回到当前目录继续读入、执行当前的Makefile。其中M不是makefile的选项，是内核根目录下的Makefile中使用的变量。

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

内核模块由以下几部分组成：

- 模块加载函数

  当通过insmod或modprob命令加载内核时，模块加载函数会自动被内核执行，完成模块相关初始化工作。

- 模块卸载函数

  当通过rmmod命令卸载某模块时，模块卸载函数会被内核自动执行，完成模块卸载。

- 模块许可证声明

  许可证（LICENSE）声明描述内核模块的许可权限，如果不声明LICENSE，模块被加载时，将收到内核被污染（Kernel Tainted）警告。

  Linux内核模块领域，可接受的LICENSE包括：括“GPL”、“GPL v2”、“GPL and additional rights”、“Dual BSD/GPL”、“Dual MPL/GPL”和“Proprietary”（关于模块是否可以采用非GPL许可权，如“Proprietary”，这个在学术界和法律界都有争议）。

  大多数情况下，内核模块遵循GPL兼容许可权。Linux内核模块最常见是以MODULE_LICENSE（“GPL v2”）语句声明模块采用GPL v2.

- 模块参数（可选）

  模块参数是模块被加载时候可以传递给它的值，对应模块内部的全局变量。

- 模块导出符号（可选）

  内核模块可以导出的符号（symbol，对应于函数或变量），若导出，则其他模块可以使用本模块中的变量或函数。

- 模块作者信息声明（可选）

## 模块加载函数

Linux内核模块加载函数一般以__init标识声明，典型加载函数如

```c
static int __init initialization_function(void)
{
    /*初始化代码*/
}
module_init(initialization_function);
```

模块加载函数以"module_init(函数名)"形式被指定。它返回整型值，初始化成功，返回0，初始化失败时应该返回错误代码。在Linux内核中，**错误编码**是一个**接近0的负值**，在<linux/errno.h>中定义，包含-ENODEV,-ENOMEM之类的符号值。总是返回相应的错误编码是一种非常好的编程习惯，只有这样才能用**perror**等方法转换成有意义的错误信息字符串。

Linux内核中，可以使用request_module(const char*fmt,...)函数加载内核模块

```c
request_module(module_name);
```

Linux中所有标识为\_\_init的函数如果直接编译进入内核，称为内核镜像的一部分，在连接的时候都会放在.init.text这个区段内。（Linux内核初始化\_\_init用函数指针）

```c
#define __init __attribute__((__section__(".init.text")))
```

所有的\_\_init函数在区段.initcall.init中保存了一份函数式指针，在初始化时内核会通过这些函数式指针调用这些\_\_init函数，并在初始化成功后释放init区段(.init.text,.initcall.init等)的内存。

除了函数以外，数据也可以被定义为\_\_initdata,对于只是初始化阶段所需要的数据，内核在初始化完成后，也可以释放它们占用的内存。例如，下面代码将hello_data定义为\_\_initdata:

```c
static int hello_data __initdata = 1;
static int __init hello_init(void)
{
    printk(KERN_INFO "Hello, world %d\n",hello_data);
    return 0;
}
module_init(hello_init);
static void __exit hello_exit(void)
{
    printk(KERN_INFO "Goodbye, world\n");
}
module_exit(hello_exit);
```

## 模块卸载函数

Linux内核模块加载函数一般以__exit标识声明，典型的模块卸载函数的形式：

```c
static void __exit cleanup_function(void)
{
    /*释放代码*/
}
module_exit(cleanup_function);
```

模块卸载函数在模块卸载的时候自动执行，不返回任何值，且必须以"module_exit"的形式来指定。通常来说，模块卸载实现和模块加载函数相反的功能。

我们使用\_\_exit来修饰模块卸载函数，可以告诉内核，如果相关的模块被直接编译进内核(built-in)，则cleanup_function()函数会被省略，直接不链接进最后的镜像。既然模块内置了，就不可能被卸载，因此卸载函数没有存在的必要。退出阶段的数据用__exitdata说明。

## 模块参数

我们可以用"module_param（参数名，参数类型，参数读/写权限）"为模块定义一个参数，例如下列代码定义了一个整型参数和一个字符指针参数：

```c
static char *book_name = "dissecting Linux Device Driver";
module_param(book_name, charp, S_IRUGO);
static int book_num = 4000;
module_param(book_num, int, S_IRUGO);
```

在装载内核模块时，用户可以想模块传递参数，形式为`insmod(modprobe) 模块名 参数名=参数值`，如果不传递，则使用模块内定义的缺省值。若内置，则bootloader可以通过bootargs设置`模块名.参数名=值`的形式给内置模块传递参数。

参数类型可以是byte、short、ushort、int、uint、long、ulong、charp（字符指针）、bool或invbool（布尔的反），在模块被编译时会将module_param中声明的类型与变量定义的类型进行比较，判断是否一致。

除此之外，模块也可以拥有参数数组，形式为“module_param_array（数组名，数组类型，数组长，参数读/写权限）”。

模块被加载后，在/sys/module/目录下将出现以此模块名命名的目录。

- 当“参数读/写权限”为0时，表示此参数不存在sysfs文件系统下对应的文件节点
- 当“参数读/写权限”不为0时，在此模块的目录下还将出现parameters目录，其中包含一系列以参数名命名的文件节点，这些文件的权限值就是传入module_param（）的“参数读/写权限”，而文件的内容为参数的值。

运行insmod或modprobe命令时，应使用逗号分隔输入的数组元素。

```c
//带参数的内核模块
#include <linux/init.h>
#include <linux/module.h>

static char *book_name = "dissecting Linux Device Driver";
module_param(book_name, charp, S_IRUGO);

static int book_num = 4000;
module_param(book_num, int, S_IRUGO);

static int __init book_init(void)
{
    printk(KERN_INFO "book name:%s\n", book_name);
    printk(KERN_INFO "book num:%d\n", book_num);
}
module_init(book_init);

static void __exit book_exit(void)
{
    printk(KERN_INFO "book module exit\n");
}
module_exit(book_exit);

MODULE_AUTHOR("Cao Zhuowen");
MODULE_LICENSE("GPL v2");
MODULE_DESCRIPTION("A simple Module for testing module params");
MODULE_VERSION("V1.0");
```

使用`dmesg`查看输出。

用户运行`insmod book.ko book_name='GoodBook' book_num=5000`可以自定义参数。

另外在/sys目录下，可以看到book模块的参数,并且可以通过`cat book_name`和`cat book_num`查看值

```bash
barry@barry-VirtualBox:/sys/module/book/parameters$ tree
.├── book_name└── book_num
```

## 导出符号

Linux的**`/proc/kallsyms`文件对应着内核的符号表**，记录了符号和符号所在的内存地址。

模块可以使用宏导出符号到内核符号表中

```c
EXPORT_SYMBOL(符号名);
EXPORT_SYMBOL_GPL(符号名);
```

**导出的符号可以被其他模块使用**，只需使用前声明一下即可。EXPORT_SYMBOL_GPL()只适用于包含GPL许可权的模块。

```c
//内核模块符号导出
#include <linux/init.h>
#include <linux/module.h>

int add_integar(int a, int b)
{
    return a + b;
}
EXPORT_SYMBOL_GPL(add_integar);

int sub_integar(int a, int b)
{
    return a - b;
}
EXPORT_SYMBOL_GPL(sub_integar);

MODULE_LICENSE("GPL v2");
```

可以从从`/proc/kallsyms`文件中找出add_integar,sub_integar的相关信息。

### 绕过GPL

内核用EXPORT_SYMBOL_GPL（）导出的符号是不可以被非GPL模块引用的。

历史上曾经有一些公司把内核的EXPORT_SYMBOL_GPL（）直接改为EXPORT_SYMBOL（），然后将修改后的内核以GPL形式发布。这样修改内核之后，模块不再使用内核的EXPORT_SYMBOL_GPL（）符号，因此模块不再需要GPL。这种做法可能构成“蓄意侵权（willful infringement）

另外一种做法是写一个wrapper内核模块（这个模块遵循GPL），把EXPORT_SYMBOL_GPL（）导出的符号封装一次后再以EXPORT_SYMBOL（）形式导出，而其他的模块不直接调用内核而是调用wrapper函数

```c
xxx_func() //非GPL模块
{
    wrapper_funca()
}
wrapper_funca() //wrapper模块 GPL v2
{
    funca()
}
EXPORT_SYMBOL(wrapper_funca)
funca() //Linux内核 GPL v2
{
}
EXPORT_SYMBOL_GPL(funca)
```

一般认为，保守的做法是Linux内核不能使用非GPL许可权。

## 模块声明与描述

在Linux内核模块中，我们可以用MODULE_AUTHOR, MODULE_DESCRIPTION, MODULE_VERSION, MODULE_DEVICE_TABLE, MODULE_ALIAS分别声明模块的作者，描述，版本，设备表和别名。

对于USB,PCI等设备驱动，通常会创建一个MODULE_DEVICE_TABLE，以表明该驱动模块所支持的设备：

```c
/* 驱动支持设备列表 */
static struct usb_device_id skel_table [] = {
{ USB_DEVICE(USB_SKEL_VENDOR_ID,
            USB_SKEL_PRODUCT_ID)},
    		{	} /*terminating enttry*/
};
MODULE_DEVICE_TABLE(usb,skel_table);
```

## 模块使用计数（一般不由驱动完成）

Linux 2.4内核中，模块通过自身MOD_INC_USE_COUNT, MOD_DEC_USE_COUNT宏来管理自己被使用的计数。

Linux 2.6以后内核提供了模块计数管理接口`try_module_get(&module)`和`module_put(&module)`，取代2.4内核中模块使用计数管理宏。模块计数一般不比由模块自身管理，且模块计数管理还考虑了SMP与PREEEMPT机制影像。

```c
int try_module_get(struct module *module);
```

该函数用于增加模块使用计数，若返回0，代表调用失败：使用的模块没有被加载或正在被卸载中。

```c
void module_put(struct module *module);
```

该函数减少模块使用计数。

这两个函数的引入和使用与Linux2.6以后的内核下设备模型密切相关。2.6以后的内核为不同类型设备定义了struct module*owner域，用来指向管理该设备的模块。当开始使用某个设备时，内核使用try_module_get(dev->owner)去增加使用计数。设备不再使用时，使用module_put(dev->owner)减少对管理此设备的管理模块的使用计数。这样设备在使用时，管理设备的模块将不能被卸载。

对于驱动而言，很少需要亲自调用try_module_get()和module_put()，因为驱动通常是为了支持某个具体设备的管理模块，对设备的计数管理由内核更底层的代码（总线驱动或此类设备共用的核心模块）实现，简化开发。

## 模块编译

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

该Makefile文件应该与源代码hello.c位于同一目录，**开启其中的EXTRA_CFLAGS=-g-O0，可以得到包含调试信息的hello.ko模块**。运行make命令得到的模块可直接在PC上运行。 

如果一个模块包括多个.c文件（如file1.c、file2.c），则应该以如下方式编写Makefile

```makefile
obj-m := modulename.o
modulename-objs := file1.o file2.o
```

