[toc]
# 原文地址
[Writing a Tiny x86 Bootloader](https://www.joe-bergeron.com/posts/Writing%20a%20Tiny%20x86%20Bootloader/)

## 译者注
其实这个程序不能算严格意义上的bootloader，实现的功能主要就是让BIOS唤起这个程序然后在屏幕上打印字符串，没有内核解压的功能，属于玩具。

本文没有完全遵从原文进行翻译，翻译的也比较生硬，见谅。

比较疑惑的地方是获取第一个参数使用的是[bp + 4]而不是[bp + 2*4]，和我查到的资料不一致，但译者没有系统学习过汇编，所以不甚了解。

在运行起来这个程序的过程中遇到了非常大的困难，暂时还没有解决，如果有人知道的话望赐教，非常感谢。

译者的运行环境为Ubuntu18 LTS和Ubuntu16 LTS，尝试过x和sdl两个display_library。原作者运行环境为Arch但没有注明版本号。

# Bootloader

在BIOS启动后，会查询任何存在的boot sector（前512字节可读并且以0x55AA结束，构成介质的引导签名）那么就会把这512字节的内容用跳转指令将程序控制转移到内存地址**0x007C00**给处理器。

当处理器处于实模式而不是保护模式时，执行被传递给引导程序代码，因此我们能直接访问底层的BIOS的中断调用。

## 开始

首先写一个小的栈提供给bootloader使用。

x86处理器有很多 段寄存器（[*segment registers*](http://wiki.osdev.org/Segmentation)），这些段寄存器用来存储开始的64k内存段。在实模式下，内存使用逻辑地址而不是物理地址。 一块内存的逻辑地址包括它所在的64k段，以及它从该段开始的偏移。64k逻辑内存段应该用16去分割，因此给定一个逻辑地址开始于64k段A，偏移为B，那么物理地址应该是A*0x10+B。

比如，处理器有一个数据段的DS寄存器，我们的代码在0x7C00，数据段应该开始于0x7C0

```assembly
mov ax, 0x7C0
mov ds, ax
```

我们得首先从其他寄存器（这里是ax）加载段，不能直接直接把他放在段寄存器中。让我们在512字节的bootloader后面直接开始我们的栈存储。启动引导程序从0x7C00到0x7E00，因此栈段，ss，起始于0x7E0

```assembly
mov ax,0x7E0
mov ss,ax
```

在x86架构上，堆栈指针会减少，所以我们必须将初始堆栈指针设置为超过堆栈段的字节数，等于堆栈的期望大小。由于堆栈段可以寻址64k的内存，让我们做一个8k的堆栈，将SP设置为0x2000。

```assembly
mov sp,0x2000
```

当完成了这一步后，我们就有了一个栈让我们实现安全的在不同的函数之间转交控制权。我们能够使用push来将调用者保存的寄存器存入栈中，用push将参数再次传递给被调用者，然后使用call将当前程序的计数器保存到栈中，并且实行一个无条件跳转到给定的标签。

那么接下来实现：清屏，移动光标并且写下文字。通过某些参数存储在某些寄存器中然后传送特定操作码作为中断发送给BIOS，我们能实现以上目标。

## 清空屏幕

比如说，存储0x07在AH寄存器然后传送中断代码0x10到BIOS，我们能将窗口向下滚动若干行。应该注意到寄存器AH和AL指的是16位寄存器AX最高位和最低位的字节。因此我们可以直接push一个16位的值到AX，然而，我们选择更清晰的方法，一次更新一个1字节的子寄存器。

通过查阅标准，可以看到，我们应该设置AH为0x07设置AL为0x00 (00h=clear entire window) ，其中BH寄存器（attribute used to write blank lines at top of window）根据BIOS颜色属性，根据我们的目的应该是黑底（0x0）淡灰色字体（0x7），因此我们必须把BH设置为0x07。寄存器CX和DX指的是我们要清除的屏幕子区域。这里标准的字符行/列是25/80，因此我们设置CH和CL设置为0x00以设置要清除的屏幕左上方，DH为0x18=24, DL为0x4f = 79。把这些整合到一起作为函数，我们得到以下片段：

```assembly
clearscreen:
	push bp
	mov bp, sp
	pusha ; 把通用寄存器压栈,当操作数是16位时，入栈顺序依次是：AX,CX,DX,BX,SP(初始值)，BP,SI,DI.
	; 注意：指令执行后，对标志寄存器无影响。
	
	mov ah,0x07 ; 告诉BIOS滚动窗口
	mov al,0x00 ; 清屏
	mov bh,0x07 ; 背景为黑色，字体为灰色
	mov cx,0x00 ; 指定屏幕左上方为0,0
	mov dh,0x18 ; 18h = 24行字符
	mov dl,0x4f ; 4fh = 79行字符
	int 0x10 ; 调用视频中断
	
	popa ; 把栈中的值弹出到通用寄存器
	mov sp,bp
	pop bp
	ret
```

```assembly
; bp为基址寄存器，一般在函数中用来保存进入函数时的sp的栈顶基址
; 每次子函数调用时，系统在开始时都会保存这个两个指针并在函数结束时恢复sp和bp的值。
; 在函数进入时：
push bp     ; 保存bp指针
mov bp,sp   ; 将sp指针传给bp，此时bp指向sp的基地址。
; 这个时候，如果该函数有参数，则[bp + 2*4]则是该子函数的第一个参数，[bp+3*4]则是该子函数的第二个参数，以此类推，有多少个参数则[bp+(n-1)*4]。
 
; 函数结束时：
mov sp,bp  ; 将原sp指针传回给sp
pop bp     ; 恢复原bp的值。
ret        ; 退出子函数
```

子程序开始和结尾处遵守调用和被调用函数之间的标准调用惯例。pusha和popa push和pop所有堆栈中的一般寄存器。我们保存调用者的基指针（4字节），然后以新的栈指针更新基指针。在最后我们镜像之前的行为。

## 光标移动

视频中断[Int 10/AH=02h](http://www.ctyme.com/intr/rb-0087.htm) 能帮我们完成这一操作。这个子程序和之前稍有不同，因为我们需要给他传递一个参数。根据标准，我们必须设置寄存器DX为一个两字节的值，第一个字节代表目标行，第二个是目标列。AH应该为0x02，BH代表我们想要移动光标到的页数。这个参数的意义是BIOS允许绘制屏幕外的内容，以便通过向用户展示屏幕以外的内容之前渲染内容使得视觉过度更平滑。这被称为多或双缓存。这里不需要用到，因此我们采取缺省值

```assembly
movecursor:
	push bp
	mov bp,sp
	pusha
	
	mov dx,[bp+4] ; 从栈中获取第一个参数
	mov ah,0x02 ; 设定光标位置
	mov bh,0x00 ; 设定第零页
	int 0x10
	
	popa
	mov sp,bp
	pop bp
	ret
```

唯一一个看起来不一般的操作就是mov dx,[bp+4] 这一操作将此参数传入DX寄存器。我们以4为偏移是因为基指针的内容占据2字节，参数占据2字节，因此我们需要从bp的实际地址总共偏移4字节。同样注意调用者有责任在被调用函数返回时清理堆栈，通过移动堆栈指针将参数从堆栈顶部移除。

## 打印文字

通过[AH=0Eh](http://www.ctyme.com/intr/rb-0106.htm) 实现打印字符串到光标所在的位置，首先我们能够定义一些数据并且存放一个指针在开始位置

```assembly
msg:	db "Hello world!",0
```

结尾处的0用一个空字符结尾字符串这样就知道字符串已经中止。我们能通过msg来引用这个地址。

```assembly
print:
	push bp
	mov bp,sp
	pusha
	
	mov si,[bp+4] ; 获取字符串指针
	mov bh,0x00	  ; 第0页
	mov bl,0x00   ; 前台颜色，文字模式下无关紧要
	mov ah,0x0E   ; 打印字符到TTY
.char:
	mov al,[si]   ; 从指针获取当前字符
	add si,1	  ; 移动指针到下一位
	or al,0
	je .return    ; 如果字符串结束则中止
	int 0x10      ; 如果字符串还没结束继续打印
	jmp .char     ; 继续循环
	
.return:
	popa
	mov sp,bp
	pop bp
	ret
```

## 最终程序

程序的第一行告诉汇编程序，我们在16位实模式下工作。

在我们完成打印后的cli和hlt两行告诉处理器不接受中断并停止处理。

最后，记得bootsector中的代码必须正好是512字节，以0xAA55结尾？最后两行将二进制文件填充到510字节的长度，并确保文件以适当的引导签名结束。

```assembly
bits 16

mov ax, 0x07C0
mov ds, ax
mov ax, 0x07E0      ; 07E0h = (07C00h+200h)/10h, 栈段的开始
mov ss, ax
mov sp, 0x2000      ; 8k栈空间

call clearscreen

push 0x0000 ; 参数
call movecursor
add sp, 2 ; 出栈
; 入栈是减法，出栈是加法

push msg ; 压入的是指针不是数据
call print
add sp, 2

cli
hlt

clearscreen:
	push bp
	mov bp, sp
	pusha ; 把通用寄存器压栈,当操作数是16位时，入栈顺序依次是：AX,CX,DX,BX,SP(初始值)，BP,SI,DI.
	; 注意：指令执行后，对标志寄存器无影响。
	
	mov ah,0x07 ; 告诉BIOS滚动窗口
	mov al,0x00 ; 清屏
	mov bh,0x07 ; 背景为黑色，字体为灰色
	mov cx,0x00 ; 指定屏幕左上方为0,0
	mov dh,0x18 ; 18h = 24行字符
	mov dl,0x4f ; 4fh = 79行字符
	int 0x10 ; 调用视频中断
	
	popa ; 把栈中的值弹出到通用寄存器
	mov sp,bp
	pop bp
	ret

movecursor:
	push bp
	mov bp,sp
	pusha
	
	mov dx,[bp+4] ; 从栈中获取第一个参数
	mov ah,0x02 ; 设定光标位置
	mov bh,0x00 ; 设定第零页
	int 0x10
	
	popa
	mov sp,bp
	pop bp
	ret

print:
	push bp
	mov bp,sp
	pusha
	
	mov si,[bp+4] ; 获取字符串指针
	mov bh,0x00	  ; 第0页
	mov bl,0x00   ; 前台颜色，文字模式下无关紧要
	mov ah,0x0E   ; 打印字符到TTY
.char:
	mov al,[si]   ; 从指针获取当前字符
	add si,1	  ; 移动指针到下一位
	or al,0
	je .return    ; 如果字符串结束则中止
	int 0x10      ; 如果字符串还没结束继续打印
	jmp .char     ; 继续循环
	
.return:
	popa
	mov sp,bp
	pop bp
	ret


msg:    db "Hello world!", 0

times 510-($-$$) db 0 ; 将二进制文件填充到510字节的长度
dw 0xAA55 ; 磁盘引导记录的重要标志，DW定义16位数据，每个数据需两个单元存放
```

# 运行代码

安装环境（Ubuntu）
```bash
sudo apt-get install nasm bochs bochs-sdl bochs-x
```

编译文件
```
nasm -f bin boot.asm -o boot.com
```

bochs配置文件
```
    megs: 32
    romimage: file=/usr/share/bochs/BIOS-bochs-latest, address=0xfffe0000
    vgaromimage: file=/usr/share/bochs/VGABIOS-lgpl-latest
    floppya: 1_44=boot.com, status=inserted
    boot: a
    log: bochsout.txt
    mouse: enabled=0
    display_library: x, options="gui_debug"
```
运行bochs
```bash
bochs -f bochsrc.txt
```

bochs模拟遇到错误

```bash
(.:3077): Gtk-CRITICAL **: IA__gtk_widget_show: assertion 'GTK_IS_WIDGET (widget)' failed
```

没有找到解决方法。



原博客还提供了

```bash
sudo dd if=boot.com of=/dev/sdb bs=512 count=1
```

这一方法使用U盘启动，依旧未果。



U盘还原方式：

```bash
sudo mkfs.vfat -L labelname /dev/sdb
# -L命令是可选，为你的u盘重新命名
# 最后的/dev/sdb一定要是设备名
```

