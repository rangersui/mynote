[toc]

# 原文地址

[Kernels 101 – Let’s write a Kernel](https://arjunsreedharan.org/post/82710718100/kernel-101-lets-write-a-kernel)

# 译者注

这应该是我翻译的第二篇和操作系统相关的文章，同样来自于github项目[build-your-own-x](https://link.zhihu.com/?target=https%3A//github.com/danistefanovic/build-your-own-x)。依旧不会完全遵照原文，有翻译的不好的地方望赐教。

和上一次的玩具bootloader一样，这个也不是严格意义的内核，不过翻译完这么一篇文章配合一下实践，让我对内核启动方面有了更深入的了解。

# 开始

实现一个由GRUB加载的x86操作系统，这个操作系统只会在屏幕上显示一条信息然后停机。

## x86计算机是如何boot的

在我们开始编写内核之前，让我们看一下计算机是如何boots up并转交控制权给内核的：

大多数x86 CPU的寄存器在通电时有定义好的数据，eip寄存器存储cpu要读取的指令地址。EIP被硬编码位0xFFFFFFF0,这样x86 CPU被硬连线到从物理地址0xFFFFFFF0开始执行。这是32位地址空间的最后16字节，这一地址也被称为复位向量（reset vector）

现在，芯片的内存映射（memory map）确保0xFFFFFFF0被映射到BIOS的特定位置，而不是映射到内存。同时，BIOS把自己复制RAM使得能被更快地访问。这被称为[shadowing](http://biosengineer.blogspot.com/2007/10/bios-shadowing-8.html)，该处的RAM称为shadow ram。地址0xFFFFFFF0只会包含一个跳转指令到BIOS复制到的内存地址。

这样BIOS代码就开始运行了，BIOS首先以boot device的顺序寻找可以boot的设备，它检查特定的数来决定某个设备是否可以boot（第511和第512区域为0xAA55)

一旦BIOS找到了可以boot的设备，他将拷贝这第一个扇区拷贝到RAM，其起始位置为0x7C00，然后跳转到该地址并执行加载的代码，这一代码称为bootloader。

bootloader在物理地址0x100000加载内核，地址0x100000被用来作为所有x86计算机的大内核（big kernel）起始地址。

所有x86处理器都是在一个简单的16位模式下开始的，称为实模式（real mode）。GRUB bootloader通过设置CR0寄存器最低位到1切换到保护模式（protected mode），这样kernel就在32位保护模式下加载了。

需要注意就linux kernel而言，GRUB检测到linux boot 协议然后在实模式加载linux内核。Linux内核自己切换到保护模式。

## 你需要的环境

- x86计算机
- Linux
- NASM汇编器
- gcc
- ld（GNU Linker）
- grub

## 源代码

原作者提供了源代码在[Github repository - mkernel](https://github.com/arjun024/mkernel)

## 汇编入口点（entry point）

我们希望一切都以C来编写，但是不能避免还是要写一点汇编，我们将会写一个很小的x86汇编程序作为内核的起始点。我们所有的汇编文件将会启用一个我们用C编写的外部函数，然后停止程序流。

那么我们如何确定这个汇编代码正常工作呢？

> 我们使用一个链接脚本来连接对象文件来创建一个最终内核执行文件（之后会详细解释）在这个链接脚本，我们将会明确说明我们想要我们的二进制文件被加载到0x100000。这一地址就是我们内核起始的地方。这样bootloader将会替我们开启内核的入口。

```assembly
;; kernel.asm
bits 32 ; nasm指令 32位
section .text

global start
extern kmain ; kmain被定义在c文件内

start:
	cli	; 屏蔽中断
	mov esp, stack_space ; 设置栈指针
	call kmain
	hlt	; 停机
	
section .bss ; block starting symbol
resb 8192 ; 8KB栈大小
stack_space:
```

指令`bits 32`并不是一个x86汇编指令。这是用来告诉NASM汇编器应该产生运行在32位模式下的代码。对我们的样例来说这句代码并不重要，但是这是一个好习惯。

下一行开始text section（也被称为code section）这是我们写代码的位置

`global`是另一个NASM指令，用于将源代码中的符号设为全局。通过这样做，链接器知道`start`符号在哪里；作为我们的入口点。

`kmain`是我们定义在kernel.c的函数。`extern`说明了该函数被定义在其他地方。

然后，我们有`start`函数，这个函数调用`kmain`函数然后通过`hlt`指令停止CPU。中断可以从停机中唤醒CPU，因此通过`cli`指令屏蔽中断

我们应该为堆栈留出一些内存，然后将栈指针（esp）指向它。但是GRUB似乎为我们做了这个，堆栈指针在这一点上已经被设置好了，但是为了以防万一我们在BBS部分分配一些空间，然后将栈指针指向所分配内存的开头。使用resb指令来保存以字节为单位的内存，在它之后留下一个标签，它将指向保存内存的边缘。在调用kmain之前，使用mov指令使堆栈指针esp指向这一空间。

> 译者注：[.bbs](https://en.wikipedia.org/wiki/.bss)被称为block starting symbol。The program loader allocates memory for the bss section when it loads the program. 

## 用C语言编写kernel

```c
/*
* kernel.c
*/
void kmain(void)
{
    const char *str = "My first kernel!";
    char *vidptr = (char*)0xb8000; //显示内存从此处开始
    unsigned int i = 0;
    unsigned int j = 0;
    
    /*这一循环清空屏幕
     *25行80列，每个元素2字节*/
    while(j<80*25*2){
        vidptr[j] = ' ';
        /* 属性字节 - 绿字黑色屏幕 */
        /* 译者注：可以从上一篇文章看这是怎么来的 */
        vidptr[j+1] = 0x02;
        j = j + 2;
    }
    
    j = 0;
    
    /*把字节写到视频内存*/
    while(str[j] != '\0'){
        vidptr[i] = str[j];
        vidptr[i+1] = 0x02;
        ++j;
        i = i + 2;
    }
    return;
}
```

这一内核会做的所有事情就是清空屏幕然后写字符串“My first kernel”

首先我们会创建一个指针`vidptr`指向地址0xb8000，这一地址是保护模式下视频内存的起点。屏幕的文字内存只是一块地址空间里的内存而已，这一内存映射输入输出从0xb8000开始支持25行80列ascii字符。

每个字符在文本内存的字符元素以16位展示而不是我们所习惯的8位。第一字节应该代表字符的ASCII码，第二字节是属性字节(attribute-byte)这描述了字符的格式比如说颜色。

为了打印绿色黑底的字符`s`，我们应该先存入字符's'然后存入0x02，以下展示可以用的颜色

```
0 - Black, 1 - Blue, 2 - Green, 3 - Cyan, 4 - Red, 5 - Magenta, 6 - Brown, 
7 - Light Grey, 8 - Dark Grey, 9 - Light Blue, 10/a - Light Green, 11/b - Light Cyan,
12/c - Light Red, 13/d - Light Magenta, 14/e - Light Brown, 15/f – White.
```

在我们的内核中，我们如果使用淡灰黑底，那么属性字节应该被设置为0x07.

第一个循环，程序书写空白字符在屏幕上来清空屏幕；第二个循环来输出"my first kernel!"。

## 链接程序

我们将会通过NASM来编译kernel.asm为一个对象文件，然后再用GCC把kernel.c变成另一个对象文件。现在，我们的工作是把这些文件连接成为一个可执行的内核文件。

因此，我们使用一个显式链接器脚本（explicit linker script），可以作为参数传递给ld（linker）。

```ld
/*
* link.ld
*/
OUTPUT_FORMAT(elf32-i386)
ENTRY(start)
SECTIONS
{
	. = 0x100000;
	.text : { *(.text) }
	.data : { *(.data) }
	.bbs  : { *(.bbs)  }
}
```

首先，我们设置我们的输出格式为32为可执行可连接格式（ELF）。ELF是标准的Unix-like系统在x86架构上的二进制文件。

**ENTRY**接受一个阐述，指定作为程序入口点的符号。

**SECTIONS**是最重要的部分。在这里，我们定义我们执行文件的布局，我们可以指定不同的部分是如何合并的还有每个部分应该放置在什么位置。

在SECTIONS语句的大括号内，句号'.'代表位置计数器。

在SECTIONS块的开始，位置计数器被初始化为0x0，可以通过给它分配新的值来修改。

还记得之前说内核的代码应该开始于0x100000。所以我们设置位置计数器到0x100000。

然后下一行的.text:{*(.text)}

星号（\*）是一个通配符，可以匹配任何文件名。因此，表达式\*(.text)意味着所有输入文件的所有.text输入部分。

因此，链接器将对象文件的所有文本部分合并到可执行文件的文本部分，地址存储在位置计数器上。因此，我们的可执行文件的代码部分开始于0x100000。

在链接器放置文本输出部分后，位置计数器的值将变成
0x1000000 + 文本输出部分的大小。

同样地，数据和bss部分也被合并，并被放置在当时的位置计数器的值上。

## Grub和多引导(Multiboot)

现在，我们已经有了两个文件，足以运行我们的kernel，不过既然我们用GRUB来boot我们的内核，那么还有一些余下的步骤。

有一个称为多引导规范（multiboot specification）的标准用于规范使用一个boot loader启动多种x86内核。

GRUB只有在以多引导规范编译时才能加载我们的内核。

根据规范，内核必须包含一个头文件（被称为多引导头文件）在前8k字节。

更进一步的，这个多引导头必须包含3个域以四字节对齐命名为：

- **magic** 域：包含 magic number 0x1BADB002，以识别头文件
- **flags** 域： 不关心这个域，设为0
- **checksum**域：checksum和magic和flags域相加必须为0

因此，`kernel.asm`需要一定的修改

```assembly
;;kernel.asm

;nasm directive - 32 bit
bits 32
section .text
        ;multiboot spec
        align 4
        dd 0x1BADB002            ;magic
        dd 0x00                  ;flags
        dd - (0x1BADB002 + 0x00) ;checksum. m+f+c should be zero

global start
extern kmain	        ;kmain is defined in the c file

start:
  cli 			;block interrupts
  mov esp, stack_space	;set stack pointer
  call kmain
  hlt		 	;halt the CPU

section .bss
resb 8192		;8KB for stack
stack_space:
```

**dd**定义了一个4字节的double word

## 构建内核

我们首先创建对象文件

```shell
nasm -f elf32 kernel.asm -o kasm.o
```

运行汇编器创建ELF-32格式的kasm.o

```shell
gcc -m32 -c kernel.c -o kc.o
```

'-c'选项确保在编译后链接不隐性发生。

```shell
ld -m elf_i386 -T link.ld -o kernel kasm.o kc.o
```

运行链接脚本生成执行文件kernel

## 设置GRUB并允许内核

GRUB需要你的kernel以`kernel-<version>`格式命名。因此，重命名kernel，我把它命名为kernel-701

现在把这个文件放置到/boot目录，你需要使用sudo命令

进入你的GRUB设置文件grub.cfg，添加一个入口，命名为

```cfg
title myKernel
	root(hd0,0)
	kernel /boot/kernel-701 ro
```

不要忘记删除指令`hiddenmenu`，如果存在的话。

### 了解你的grub版本

```shell
grub-install --version
```

### 使用grub2

To **add** a **new kernel** to grub2:

1. Move your **kernel** to /boot/
2. Run sudo update-**grub**.

然后进入管理员模式`sudo -i`手动编辑vim /boot/grub/grub.cfg

请注意手动编辑grub.cfg是非常危险的行为，请在虚拟机进行！

通过`/<kernel version>`查找你的内核所在的位置，然后删除大括号内`update-grub`自动编写的一大堆东西，保留以下两条内容
```
menuentry 'kernel 101' {
	set root='hd0,msdos1'
	multiboot /boot/kernel-701 ro
}
```

## Invalid Magic Number

如果遇到这个问题，先不要慌，打开advanced options for ubuntu，选择一个正常工作的内核即可。(你自己编写的内核可能也需要通过这个方法进入，似乎不能自动加载)

## qemu

 如果使用qemu模拟器而不是grub:

```sh
qemu-system-i386 -kernel kernel
```

下一篇文章:
[Kernels 201 - Let’s write a Kernel with keyboard and screen support](http://arjunsreedharan.org/post/99370248137/kernels-201-lets-write-a-kernel-with-keyboard-and)