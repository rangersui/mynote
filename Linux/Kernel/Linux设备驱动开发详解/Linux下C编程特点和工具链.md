[toc]

# Linux内核引导

嵌入式Linux领域最著名的bootloader是[U-Boot](http://git.denx.de/u-boot.git/)。

早前， bootloader需要将启动信息以ATAG的形式封装，并且把ATAG的地址填充在r2寄存器中，机型号填充在r1寄存器中，详见内核文档Documentation/arm/booting。

在ARM Linux支持设备树（Device Tree）后，bootloader 则需要把dtb的地址放入r2寄存器中。

当然，ARM Linux也支持直接把dtb和zImage绑定在一起的模式（内核ARM_APPENDED_DTB选项“Use appended device tree blob to zImage”），这样r2寄存器就不再需要填充dtb地址了。

类似zImage的内核镜像实际上是由没有压缩的解压算法和被压缩的内核组成，所以在bootloader跳入zImage以后，它自身的解压缩逻辑就把内核的镜像解压缩出来了。

关于内核启动，与我们关系比较大的部分是每个平台的设备回调函数和设备属性信息，它们通常包装在DT_MACHINE_START和MACHINE_END之间，包含reserve（）、map_io（）、init_machine（）、init_late（）、smp等回调函数或者属性。这些回调函数会在内核启动过程中被调用。

## 设备树

**设备树（Device Tree）** 是描述计算机的特定硬件设备信息的数据结构，以便于操作系统的内核可以管理和使用这些硬件，包括 CPU 或 CPU，内存，总线和其他一些外设。

设备树是通过`Open Firmware`项目从基于 SPARC 的工作站和服务器派生而来的。当前的 Devicetree 一般针对嵌入式系统，但仍然与某些服务器级系统一起使用（例如，Power Architecture Platform Reference 所描述的系统）。

然而x86架构的个人计算机通常不使用设备树，而是依靠各种自动配置协议来识别硬件。使用设备树的系统通常将静态设备树（可能存储在 ROM 中）传递给操作系统，但也可以在引导的早期阶段生成设备树。

例如，U-Boot 和 kexec 可以在启动新操作系统时传递设备树。一些系统使用的引导加载程序可能不支持设备树，但是可以与操作系统一起安装静态设备树，Linux 内核支持这种方法。Device Tree 规范目前由名为devicetree.org的社区管理，该社区与 Linaro 和 Arm 等相关联。

### 设备树的优势

Linux 内核从 3.x 版本之后开始支持使用设备树，这样做的意义重大，可以实现驱动代码与设备的硬件信息相互的隔离，减少了代码中的耦合性，在此之前，一些与硬件设备相关的具体信息都要写在驱动代码中，如果外设发生相应的变化，那么驱动代码就需要改动。

但是在引入了设备树之后，这种尴尬的情况找到了解决的办法，通过设备树对硬件信息的抽象，驱动代码只要负责处理逻辑，而关于设备的具体信息存放到设备树文件中，这样，如果只是硬件接口信息的变化而没有驱动逻辑的变化，开发者只需要修改设备树文件信息，不需要改写驱动代码。

### dts

硬件的相应信息都会写在`.dts`为后缀的文件中，每一款硬件可以单独写一份`xxxx.dts`，一般在`Linux`源码中存在大量的`dts`文件，对于 arm 架构可以在`arch/arm/boot/dts`找到相应的`dts`，另外`mips`则在`arch/mips/boot/dts`，`powerpc`在`arch/powerpc/boot/dts`。

### dtb

`dtb(Device Tree Blob)`，`dts`经过`dtc`编译之后会得到`dtb`文件，`dtb`通过`Bootloader`引导程序加载到内核。所以`Bootloader`需要支持设备树才行；Kernel 也需要加入设备树的支持；

来源：[不懂Linux Device Tree，被新人嘲笑之后，含泪写完](https://www.eefocus.com/mcu-dsp/474573)

# Linux下C编程特点

## 编码风格

Linux有独特的编码风格，在Documentation/CodingStyle有详细描述。

```c
#define PI 3.1415926
int min_value, max_value;
void send_data(void);
```

Linux中，使用“TAB”缩进。

### 括号{}使用原则

1. 对结构体，if/for/while/switch语句，"{"不另起一行：

   ```c
   struct var_data{
       int len;
       char data[0];
   };
   if (a == b){
       a = c;
       d = a;
   }
   for (i = 0; i < 10; i++){
       a = c;
       d = a;
   }
   ```

2. 如果if，for循环后只有一行，不加{}

   ```
   for ( i = 0; i < 10; i++)
   	a = c;
   ```

3. if和else混用情况下，else语句不另起一行

   ```c
   if (x == y){
       
   } else if (x>y) {
       
   } else {
       
   }
   ```

4. 对于函数，{另起一行

   ```c
   int add(int a, int b)
   {
       return a + b;
   }
   ```

### switch和case对齐

```c
switch (suffix) {
case 'G':
case 'g':
	mem <<= 30;
    break;
case 'M':
case 'm':
    mem <<= 20;
    break;
case 'K':
case 'k':
    mem <<=10;
default:
    break;
}
```

内核下的scripts/checkpatch.pl 提供了1个检查代码风格的脚本。

在工程阶段，一般可以在SCM软件的服务器端使能pre-commit hook，自动检查工程师提交的代码是否符合Linux的编码风格，如果不符合，则自动拦截。git的pre-commit hook可以运行在本地代码仓库中，如Ben Dooks完成的一个版本：

```shell
#!/bin/sh 
#
# pre-commit hook to run check-patch on the output and stop any commits 
# that do not pass. Note, only for git-commit, and not for any of the 
# other scenarios 
#
# Copyright 2010 Ben Dooks, <ben-linux@fluff.org> 
if git rev-parse --verify HEAD 2>/dev/null >/dev/null 
then
	against=HEAD 
else
	# Initial commit: diff against an empty tree object 
	against=4b825dc642cb6eb9a060e54bf8d69288fbee4904 
fi
git diff --cached $against -- | ./scripts/checkpatch.pl --no-signoff -
```

## GNU C 与 ANSI C

Linux上可用的C编译器是GNU C编译器，它建立在自由软件基金会的编程许可证的基础上，因此可以自由发布。GNU C对标准C进行一系列扩展，以增强标准C的功能。

### 零长度和变量长度数组

GNU C允许使用零长度数组，定义变长对象的头结构时，这个特性非常有用：

```c
struct var_data {
    int len;
    char data[0];
};
```

> char data[0]意味着程序通过var_data结构体实例的data[index]成员可以访问len之后的第index个地址，没有给data[]数组分配内存，因此sizeof(struct var_data) =  sizeof (int)。

假设struct var_data的数据与就保存在struct var_data紧接着的内存区域中，通过如下代码可以遍历数据

```c
struct var_data s;
for (i = 0; i < s.len; i++)
    printf("%02x", s.data[i]);
```

GNU C中也可以使用1个变量定义数组

```c
int main (int argc, char *argv[])
{
    int i, n = argc;
    double x[n];
    for (i = 0; i < n; i++)
    	x[i] = i;
    return 0;
}
```

### case范围

GNU C支持case x...y写法，区间[x,y]中的数都满足case的条件

```c
switch (ch){
case '0'...'9': c -= '0';
    break;
case 'a'...'f': c -= 'a' - 10;
    break;
case 'A'...'F': c -= 'A' - 10;
    break;
}
```

代码中case '0'...'4'等价于case '0': case '1': case '2': case '3': case '4' :

### 宏

#### 语句表达式

GNU C把包含在括号中的符合语句看成一个表达式，称为语句表达式，可以出现在任何允许表达式的地方。可以在语句表达式中使用原本只能在复合语句中使用的循环，局部变量等。

```c
#define min_t(type,x,y) \
({type __x =(x);type __y = (y); __x<__y ? __x: __y;})
int ia,ib,mini;
float fa,fb,minf;
mini = min_t(int,ia,ib);
minf = min_t(float,fa,fb);
```

#### typeof关键字

```c
#define min(x,y) ({       \
const typeof(x) _x = (x); \
const typeof(y) _y = (y); \
(void) (&_x == &_y);      \
_x < _y ? _x:_y})
```

typeof获得传入参数的类型，(void)  (&\_x == &\_y)检查\_x和\_y类型是否一致。

#### 可变参数宏

标准C支持可变参数函数，如printf()

```c
int printf( const char *format [, argument]...);
```

GNU C中，宏也可以接受可变参数

```c
#define pr_debug(fmt,arg...)\
			printk(fmt,##arg) //printk是kernel打印日志
```

使用##是为了处理arg不代表任何参数的情况，这个时候前面的逗号就多余了，使用##后GNU C预处理器会丢弃前面的逗号。

### 标号元素

标准C要求数组或结构体的初始化必须以固定的顺序出现，在GNU C中，通过指定索引或结构体成员名，允许初始化值以任意顺序出现。

指定数组索引的方法是初始化值前添加"[INDEX]="，当然也可以用"[FIRST...LAST]="的形式指定一个范围。通过以下方式定义数组并赋值0：

```c
unsigned char data[MAX] = {[0...MAX-1] = 0};
```

下面代码借助结构体成员名初始化结构体：

```c
struct file_operations ext2_file_operations = {
    llseek: generic_file_llseek,
    read: generic_file_read,
    write: generic_file_write,
    ioctl: ext2_ioctl,
    mmap: generic_file_mmap,
    open: generic_file_open,
    release: ext2_release_file,
    fsync: ext2_sync_file,
};
```

Linux 2.6推荐使用标准C的方式：

```c
struct file_operations ext2_file_operations = {
    .llseek		= generic_file_llseak,
    .read		= generic_file_read,
    .write 		= generic_file_write,
    .aio_read	= generic_file_aio_read,
    .aio_write	= generic_file_aio_write,
    .ioct		= ext2_ioctl,
    .mmap		= generic_file_mmap,
    .open		= generic_file_open,
    .release	= ext2_release_file,
    .fsync		= ext2_sync_file,
    .writev		= generic_file_writev,
    .sendfile	= generic_file_sendfile,
}
```

### 当前函数名

GNU C预定义了两个标识符保存当前函数的名字，\_\_FUNCTION\_\_保存函数在源码中的名字，\_\_PRETTY\_FUNCTION\_\_保存带语言特色的名字。在C函数中，两个名字相同。

```c
void example()
{
	printf("%This is function:%s",__FUNCTION__);
}
```

代码中\_\_FUNCTION\_\_意味着字符串“example”，C99已经支持\_\_func\_\_，因此建议转用\_\_func\_\_。

### 特殊函数声明

GNU C允许声明函数，变量和类型的特殊属性，一边手动优化代码和定制代码检查的方法。要指定一个声明的属性，只需要在声明后面添加\_\_attribute\_\_，其中ATTRIBUTE为属性说明，如果存在多个属性，则以都好风格。GNU C支持noreturn，format，section，aligned，packed等十多个属性。

#### noreturn属性

作用于函数，代表该函数从不返回，这会让编译器优化代码，并消除不必要的警告信息。如：

```c
#define ATTRIB_NORET __attribute__((noreturn))
#define NORET_TYPE /**/
#define asmlinkage __attribute__((regparm(0)))
asmlinkage NORET_TYPE void do_exit(long error_code) ATTRIB_NORET
```

#### format属性

也用于函数，表示该函数使用printf，scanf或strftime风格的参数，指定format属性可以让编译器根据格式串检查参数类型。

```c
asmlinkage int printk(const char * fmt, ...) __attribute__((format(printf, 1, 2)));
```

上述代码第一个参数是格式串，从第二个参数开始会根据printf函数格式串规则检查参数。

#### unused属性

作用于函数和变量，表示该函数或变量可能不会用到，避免编译器产生警告信息。

#### aligned属性

用于变量，结构体或联合体，指定对齐方式，以字节为单位：

```c
struct example_struct{
    char a;
    int b;
    long c;
}__attribute__((aligned(4)));
```

表示该结构体以4字节对齐。

##### Data structure alignment

**[Data structure alignment](https://stackoverflow.com/questions/11770451/what-is-the-meaning-of-attribute-packed-aligned4)** is the way data is arranged and accessed in computer memory. It consists of two separate but related issues: *data alignment* and *data structure padding*.

When a modern computer reads from or writes to a memory address, it will do this in word sized chunks (e.g. 4 byte chunks on a 32-bit system). ***Data alignment\*** means putting the data at a memory offset equal to some multiple of the word size, which increases the system's performance due to the way the CPU handles memory.

To align the data, it may be necessary to insert some meaningless bytes between the end of the last data structure and the start of the next, which is ***data structure padding\***.

#### packed属性

用于变量和类型，用于变量或结构体成员时表示使用最小可能的对齐，用于枚举，结构体或联合体代表该类型使用最小的内存，例如：

```c
stuct example_struct{
    char a;
    int b;
    long c __attribute__((packed));
}
```

##### 对齐的目的

编译器对结构体成员及变量对齐的目的是为了更快地访问结构体成员及变量占据的内存。例如，对于一个32位的整型变量，若以4字节方式存放（即低两位地址为00），则CPU在一个总线周期内就可以读取32位；否则，CPU需要两个总线周期才能读取32位（一次只能取一部分）

## 内建函数

GNU C提供大量内建函数，其中大部分是标准C库函数的GNU C编译器内建版本，例如memecpy等，于对应的标准C库函数功能相同。

不属于库函数的其他内建函数通常以__builtin开始，如下所示。

- 内建函数__builtin_return_address(LEVEL)返回当前函数或其调用者的返回地址， 参数LEVEL指定的调用栈级数，如0表示当前函数的返回地址，1表示当前函数的调用者的返回地址。

- 内建函数__builtin_constant_p(EXP)用于判断一个值是否为编译时常熟，如果参数EXP的值是常数返回1，否则返回0.

  例如，下面代码可检查第一个参数是否为编译时常数以确定采用参数版本还是非参数版本：

  ```c
  #define test_bit(nr,addr)\
  (__builtin_constant_p(nr)\
  constant_test_bit((nr),(addr)) :\
  variable_test_bit((nr),(addr)))
  ```

- 内建函数__builtin_expect(EXP,C)用于为编译器提供分支预测信息，其返回值是表达式EXP的值，C的值必须是编译时常数

  Linux内核编程时常用的likely()和unlikely()底层调用的likely_nontrace(),unlikely_notrace()就是基于__builtin_expect(EXP,C)实现的。

  ```c
  #define likely_notrace(x)	__builtin_expect(!!(x),1)
  #define unlikely_notrace(x)	__builtin_expect(!!(x),0)
  ```

  若代码中出现分支，则可能中断流水线，我们可以通过likely和unlikely暗示分支容易成立还是不容易成立例如

  ```c
  if(likely(!IN_DEV_ROUTE_LOCALNET(in_dev)))
      if(ipv4_is_loopback(saddr))
          goto e_inval;
  ```

使用gcc编译C程序的时候，如果使用“-ansi-pedantic”编译选项，则会告诉编译器不使用GNU扩展语法。

```bash
gcc -ansi -pedantic -c test.c
```

## do{}while(0)语句

这一语句通常用于宏定义中。

```c
#define SAFE_FREE(p) do{free((p));(p)=NULL;}while(0)
```

do{}while(0)的使用完全是为了保证宏定义的使用者能完全无编译错误地使用宏，不对使用者做任何假设。

## goto

Linux内核源代码中对goto应用非常广泛，但是只限于错误处理，其结构如：

```c
if(register_a()!=0)
    goto err;
if(register_b()!=0)
    goto err1;
if(register_c()!=0)
    goto err2;
if(register_d()!=0)
    goto err3;
//...
err3:
	unregister_c();
err2:
	unregister_b();
err1:
	unregister_a();
err:
	return ret;
```

这种goto用于错误处理的用法是简单高效的，秩序保证在错误处理时注销，释放资源等，于正常的注册，资源申请顺序相反。

# 工具链

在Linux的编程中，通常使用GNU工具链编译Bootloader、内核和应用程序。GNU组织维护了GCC、GDB、glibc、Binutils等

建立交叉工具链的过程相当烦琐，一般可以通过类似crosstool-ng这样的工具来做。crosstool-ng也采用 了与内核相似的menuconfig配置方法。在[官网](http://www.crosstool-ng.org/)上下载crosstool-ng的源代码并编译 安装后，运行ct-ng menuconfig。在里面我们可以选择目标机处理器型号，支持的内核版本号等。 

也可以直接下载第三方编译好的、开放的、针对目标处理器的交叉工具链:

- [针对ARM、MIPS、高通Hexagon、Altera Nios II、Intel、AMD64等处理器的工具链](http://www.mentor.com/embedded-software/sourcery-tools/sourcery-codebench/editions/lite-edition/)

- [针对ARM的工具链](http://www.linaro.org/downloads/)

一个典型的ARM Linux工具链包含arm-linux-gnueabihf-gcc（后续工具省略前缀）、strip、gcc、objdump、ld、gprof、nm、readelf、addr2line等。

- **strip**可以删除可执行文件中的符号表和调试信息等来实现**缩减程序体积**的目的。
- **gprof**在编译过程中在函数入口处插入计数器以**收集每个函数的被调用情况和被调用次数**，检查程序计数器并在分析时找出与程序计数器对应的函数来**统计函数占用的时间**。
- **objdump**是反汇编工具。
- **nm**则用于显示关于对象文件、可执行文件以及对象文件库里的符号信息。