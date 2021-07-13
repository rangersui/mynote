[toc]

# 原文地址

[Kernels 201 - Let’s write a Kernel with keyboard and screen support](https://arjunsreedharan.org/post/99370248137/kernels-201-lets-write-a-kernel-with-keyboard)

# 译者注

太好了，这是我连续第三天翻译和操作系统相关的文章，依旧依旧来自[build-your-own-x](https://link.zhihu.com/?target=https%3A//github.com/danistefanovic/build-your-own-x)。翻译的不好的地方望赐教。

这篇是真的很长，知识点也很多，帮我复习了一遍计组的总线部分了属于是。这篇三篇文章看下来，虽然说和真正的操作系统差的很远，但确实让我对操作系统的样貌有了一个感性的认识。

16号就去军训了，可能就停更一段时间。也许下一篇文章是一个粗略的shell？

# 开始

在之前的[Kernel101(翻译版)](https://zhuanlan.zhihu.com/p/388895056)当中我们构造了一个简陋的使用GRUB进行boot up的x86核心，运行在CPU保护模式下并且在屏幕上打印一行文字。

今天，我们会扩展这一内核，包含键盘驱动以从键盘读取字符'a-z'和'0-9'然后打印到屏幕。

源代码可以在GitHub仓库找到[Github repository - mkeykernel](https://github.com/arjun024/mkeykernel)

我们通过I/O端口和I/O设备交流，这些端口是x86的I/O总线上的特定地址，仅此而已。这些端口的读/写操作通过处理器内置的特定指令完成。

## 读取并向端口写

```assembly
read_port:
	mov edx,[esp+4]
	in al,dx
	ret
	
write_port:
	mov edx,[esp+4]
	mov al,[esp+4+4]
	out dx,al
	ret
```

I/O端口通过`in`和`out`指令访问，这两个指令是x86指令的一部分。

在`read_port`，这个端口号作为参数读取。当编译器调用你的函数，它把所有的参数push进栈。参数通过栈指针被复制到寄存器`edx`。寄存器`dx`是`edx`寄存器的低16位。`in`指令读取端口（`dx`提供端口号）并且把结果送入`al`。寄存器`al`是寄存器`eax`的低8位。如果你还记得你大学时的课程，函数返回值通过`eax`寄存器接收。通过这样的方法`read_port`使得我们读取I/O端口。

`write_port`非常类似，我们用两个参数，端口号和书写的数据。`out`指令把数据写到端口。

## 中断

现在，在我们继续编写设备驱动之前，我们应该了解处理器是如何知晓设备进行执行了一个事件。

最简单的解决方法是**轮询(polling)**永远保持检查设备状态。这样很明显不实用也不高效。这就是为什么我们要使用**中断(interrupts)**，中断是通过硬件或软件传输给处理器的一个信号，表明发生了某个事件。通过中断，我们可以避免轮询并且只当我们感兴趣的那个中断被触发时做出反应。

一个设备或芯片称为**可编程中断控制器（Programmable Interrupt Controller, PIC）**使得x86成为一个中断驱动架构。这个设备管理硬件中断并给他们传输恰当的系统中断。

当特定的操作在硬件上发生，硬件会传送一个脉冲叫做**中断请求（Interrupt Request, IRQ）**通过它的中断线传输到PIC芯片。PIC把中断请求翻译为一个系统中断，然后发送一条消息让CPU从当前工作中断。然后内核就负责去处理这些中断。

如果没有PIC，我们需要去轮询所有系统中的设备来知晓是否有事件发生。

那么让我们来看看键盘，键盘通过I/O端口`0x60`和`0x64`工作。端口`0x60`提供数据（按下键）然后端口`0x64`提供状态。但是你得知道你什么时候去读这些端口。

中断在这里就很适合了，当按键按下，键盘给PIC发送信号通过中断线路IRQ1。PIC有一个初始化时存在的`offset`值。把这个值加上输入线的数值构成**中断编号（Interrupt number）**。然后处理器寻找一个特定的数据结构称为**中断描述符表（Interrupt Descriptor Table, IDT)**给出和中断编号相应的中断处理程序地址。然后这个地址上的程序进行运行来处理这一事件。

## 构建中断描述符表（IDT）

我们将IDT实现为由一个IDT_entry构成的数组。我们将会讨论键盘中断是如何映射到处理程序的，但是首先让我们看看PICs是如何工作的。

### PICs

现代x86处理器由两个PIC芯片，每个都有8根输入线，我们称这两个PIC芯片位PIC1和PIC2。PIC1接收IRQ0到IRQ7的信号，PIC2接收IRQ8到IRQ15的信号。PIC1使用端口`0x20`接收命令，`0x21`接收数据。PIC2使用端口`0xA0`接收指令，`0xA1`接收数据。

PICs使用8位**初始化指令字（Initialization command words，ICW）**初始化。查看[这条链接](https://href.li/?//stanislavs.org/helppc/8259.html)获取这些指令的详细信息。

在保护模式下，你需要给这两个PICs传输初始化指令ICW1 `0x11`  ，这条指令使得PIC在数据端口等待三条初始化指令字。

三条数据指令分别告诉PICs：

- 向量偏移（vector offset）（ICW2）
- PICs如何作为主/从接线的（How the PIC’s wired as master/slaves）（ICW3）
- 给予关于环境的附加信息（ICW4）

下一个初始化指令是ICW2，写入到每个PIC的数据端口。设置PIC的偏移值。这个偏移值就是我们加在输入线数字来构成中断编号的值。

PICs允许它们的输出在彼此之间串联成输入（PICs allow cascading of their outputs to inputs between each other.）。这一设置通过ICW3然后每一位代表对应IRQ的级联状态（cascading status）。就现在来说，我们不使用这个（We won't use cascading）所以全部设为0.

ICW4设置附加环境参数，我们只设置大多数的低位来告诉PICs我们运行在80x86模式上。

```c
 	/* ICW1 - 开始初始化 */
    write_port(0x20,0x11);
    write_port(0xA0,0x11);
    
    /* ICW2 - 重映射IDT偏移地址 */
    /*
    * x86保护模式下，我们需要重映射PICs到0x20外因为
    * Intel把前32个中断保存给了cpu异常
    */
    write_port(0x21,0x20);
    write_port(0xA1,0x28);
    
    /* ICW3 - 不建立级联（cascading） */
    write_port(0x21,0x00);
    write_port(0xA1,0X00);
    
    /* ICW4 - 环境信息 */
    write_port(0x21,0x01);
    write_port(0xA1,0x01);
    /* 初始化完成 */
```

到此为止，PIC被初始化。

### IMR

每个PIC有一个内部的8位寄存器称为**中断屏蔽寄存器（Interrupt Mask Register，IMR）**。这个寄存器存储一个IRQ数据线到PIC的位映射。当某一位被设置时，PIC将会忽略这个请求。这代表着我们能够启用和停用第n条IRQ数据线通过让第n位的IMR设为0或1通过从数据端口读取IMR寄存器的返回值，向其中写入来设置这个寄存器。

在我们的代码中，初始化PICs后，我们设置所有位为1，这样就屏蔽了所有中断。我们之后会启用键盘的中断响应，但是目前来讲，我们停用所有中断。

既然现在所有中断线被启用，我们的PICs能获取所有来自IRQ的中断请求然后和偏移值相加转换为中断编号。那么现在，我们需要填充IDT让键盘的中断被映射到我们写的键盘处理程序。

那么我们应该给键盘处理程序的地址和IDT的哪个中断进行映射呢？

键盘使用IRQ1，这是PIC1的1号线。我们初始化PIC1的偏移值0x20。那么我们的中断编号就应该是1+`0x20` = `0x21`.所以键盘处理程序的地址应该和IDT的0x21相映射。

所以我们下一步就是去填充中断`0x21`给IDT

我们将会映射这个中断到一个程序称为`keyboard_handler`，这个程序我们稍后会编写。

每个IDT条目由64位组成。在IDT中断入口，我们不存储处理程序的整个地址。我们分割为两个16位的部分。低位存储在前16位的IDT条目然后高16位存储在最后16位。这是为了保持和286的兼容性。你可以看到英特尔在很多地方都采取了这样的精明的笨办法（shrewd kludges）！！

在IDT条目中，我们同样需要设置类型——这么做是为了捕捉中断。我们同样需要给内核代码段偏移。GRUB bootloader给我们设置了一个[全局描述符表 (GDT)](https://en.wikipedia.org/wiki/Global_Descriptor_Table) （译者注：提供段式存储机制）每个GDT 8位长度，内核代码描述符在第二段。所以偏移量是`0x08`（如果做更多解释那么这篇文章就太长了）。中断门由`0x8e`表示。中间剩下的8位（译者注：似乎应该是16位？）就由0来填充。这样我们就填充了与键盘中断对应的IDT条目。

```c
    /* 填充键盘中断的IDT条目(entry) */
    keyboard_address = (unsigned long)keyboard_handler;
    IDT[0x21].offset_lowerbits = keboard_address & 0xffff;
    IDT[0x21].selector = 0x08; /* KERNEL_CODE_SEGMENT_OFFSET */
    IDT[0x21].zero = 0;
    IDT[0x21].type_attri = 0x8e; /* INTERRUPT_GATE */
    IDT[0x21].offset_higherbits = (keyboard_address & 0xffff0000) >> 16;
```

当所需的映射在IDT当中写好了，我们得告诉CPU IDT究竟在哪里。

这个过程通过`lidt`汇编指令,`lidt`有一个操作数。这个操作数必须是一个指向描述IDT描述符结构的指针。

这一描述符是比较直白的，包含了IDT的大小（字节）和它的地址。我使用一个数组来打包这些值，当然你也可以用结构体填充。

我们由指针变量`idt_ptr`然后把他传递给`lidt`通过函数`load_idt()`

```assembly
load_idt:
	mov edx,[esp+4]
	lidt [edx]
	sti
	ret
```

另外，`load_idt()`函数通过sti指令打开中断

一旦IDT被建立和加载，我们可以通过我们之前讲过的中断屏蔽打开键盘中断请求线。

``` c
void kb_init(void)
{
    /* 0xFD为1111 1101 - 只启用IRQ1（键盘） */
    write_port(0x21,0xFD);
}
```

完整代码如下：

```c
struct IDT_entry{
    unsigned short int offset_lowerbits;
    unsigned short int selector;
    unsigned char zero;
    unsigned char type_attr;
    unsigned short int offset_higherbits;
};

struct IDT_entry IDT[IDT_SIZE];

void idt_init(void)
{
    unsigned long keyboard_address;
    unsigned long idt_address;
    unsigned long idt_ptr[2];
    
    /* 填充键盘中断的IDT条目(entry) */
    keyboard_address = (unsigned long)keyboard_handler;
    IDT[0x21].offset_lowerbits = keyboard_address & 0xffff;
    IDT[0x21].selector = 0x08; /* KERNEL_CODE_SEGMENT_OFFSET */
    IDT[0x21].zero = 0;
    IDT[0x21].type_attr = 0x8e; /* INTERRUPT_GATE */
    IDT[0x21].offset_higherbits = (keyboard_address & 0xffff0000) >> 16;
    
    /*       端口
    *    PIC1   PIC2
    *指令 0x20   0xA0
    *数据 0x21   0xA1
    */
    
    /* ICW代表初始化指令字 */
    
    /* ICW1 - 开始初始化 */
    write_port(0x20,0x11);
    write_port(0xA0,0x11);
    
    /* ICW2 - 重映射IDT偏移地址 */
    /*
    * x86保护模式下，我们需要重映射PICs到0x20外因为
    * Intel把前32个中断保存给了cpu异常
    */
    write_port(0x21,0x20);
    write_port(0xA1,0x28);
    
    /* ICW3 - 建立串联（cascading） */
    write_port(0x21,0x00);
    write_port(0xA1,0X00);
    
    /* ICW4 - 环境信息 */
    write_port(0x21,0x01);
    write_port(0xA1,0x01);
    /* 初始化完成 */
    
    /* 屏蔽中断 */
	write_port(0x21 , 0xff);
	write_port(0xA1 , 0xff);

	/* 填充IDT描述符 */
	idt_address = (unsigned long)IDT ;
	idt_ptr[0] = (sizeof (struct IDT_entry) * IDT_SIZE) + ((idt_address & 0xffff) << 16);
	idt_ptr[1] = idt_address >> 16 ;
    
    load_idt(idt_ptr);
}
```

## 键盘中断处理程序

好了，现在我们成功映射键盘中断到函数`keyboard_handler`通过IDT条目的中断`0x21`

所以，每次你按下键盘你都能确保这个函数被呼叫。

```assembly
keyboard_handler:
	call keyboard_handler_main
	iretd
```

这个函数只是调用我们用C语言写的处理函数然后使用`iret`类指令返回。我们可以把整个中断处理过程写在这里但是C代码比汇编要简单。

`iret/iretd`应该在这里被使用而不是`ret`当从中断处理程序返回到被中断打断的程序中时。这类指令会弹出在中断调用时push进堆栈的标志寄存器。

### C语言程序

我们最初的信号中断结束确认（End Of Interrupt acknowledgement，EOI）向PIC的命令端口写入。只有在这之后，PIC才会进一步接收中断请求。我们需要读取两个端口：

- 数据端口`0x60`
- 命令/状态端口`0x64`

我们首先读取状态端口`0x64`，如果最低为的状态是0，这代表着缓冲是空的并且没有任何数据需要被读取。若是非空，我们能读取数据端口`0x60`，这个端口会给我们按下键的键盘码，每个键盘代码代表着在键盘上的按键。我们使用一个简单的字符数组定义在`keyboard_map.h`来映射键盘编号到对应字符。这个字符在接下来会通过和上一篇文章同样的技术被映射到屏幕上。

```c
void keyboard_handler_main(void)
{
    unsigned char status;
    char keycode;
    
    /* 写EOI */
    write_port(0x20,0x20);
    
    status = read_port(KEYBOARD_STATUS_PORT);
    /* 如果缓存非空那么低位状态会被设置 */
    if(status&0x01){
        keycode = read_port(KEYBOARD_DATA_PORT);
        if(keycode < 0)
            return;
        vidptr[current_loc++] = keyboard_map[keycode];
        vidptr[current_loc++] = 0x02;
    }
}
```

在这篇文章里，为了简洁起见，我只处理了小写a-z和数字0-9，你可以把这些扩展到特殊字符如`ALT`,`SHIFT`等等。你也可以了解案件是否按下或释放通过status口然后实施你想要实现的功能。你也可以映射任何组合键来实现你想要的功能比如说关机等。

你可以构建内核，运行在真机或模拟器（QEMU）等，方式和[之前的文章](https://zhuanlan.zhihu.com/p/388895056)是一样的。

(原文完)

# 补充

为了方便读者直接运行文件，我把代码放到  [百度云盘 ](https://pan.baidu.com/s/1_FQKrBKHRWcJFIeWW1GLbw )  提取码：0rkk 

只需要有一台linux虚拟机，安装有nasm和qemu，运行`./run.sh`就可以直接看到效果。

代码的著作权归原作者所有。

