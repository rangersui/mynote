[toc]

# 编译

```makefile
gcc -o target -g main.cpp
```

加上-g生成可以被gdb调试的程序

# 常用指令

## 查看变量

### print (p)

#### 指定输出格式

print/格式 变量：

- `d`：按十进制格式打印变量

- `x`：按十六进制格式打印变量

- `t`：按二进制格式打印变量

- `o`：按八进制格式打印变量

- `f`：按浮点数格式打印变量

- `c`：按字符格式打印变量

#### 指定范围

例如：`print 'main'::val` 表示查看 main 函数中的整型变量 **val** 的值。

#### 查看数组中多个元素的值

可数据名后加 *@* 并跟上期望查看的元素个数。例如，`print *array@3` 可查看数组 int_array 的前3个元素。

#### 指定查看数组特定位置的元素

```shell
set $index=7
print array[$index++]
#持续回车就会连续显示
```

#### 特殊设置

详见：https://www.cnblogs.com/rosesmall/archive/2012/04/12/2444431.html

#####  打开数组显示
`set print array`  打开后当数组显示时，每个元素占一行，如果不打开的话，每个元素则以逗号分隔。这个选项默认是关闭的。

##### 数据显示的最大长度
`set print elements`  指定一个来指定数据显示的最大长度，当到达这个长度时，GDB就不再往下显示了。如果设置为0，则表示不限制。

##### 遇到结束符则停止显示
`set print null-stop`  如果打开了这个选项，那么当显示字符串时，遇到结束符则停止显示。默认为off。

##### 美化结构体

`set print pretty` 如果打开printf pretty这个选项，那么当GDB显示结构体时会比较漂亮。

##### 自动按照虚方法调用的规则显示输出
`set print object`  在C++中，如果一个对象指针指向其派生类，如果打开这个选项，GDB会自动按照虚方法调用的规则显示输出，如果关闭这个选项的话，GDB就不管虚函数表了。这个选项默认是off。

##### 显示其中的静态数据成员
`set print static-members`  这个选项表示，当显示一个C++对象中的内容时，是否显示其中的静态数据成员。默认是on。

##### 用比较规整的格式来显示虚函数表
`set print vtbl`  当此选项打开时，GDB将用比较规整的格式来显示虚函数表时。其默认是关闭的。

### x (examine命令)

查看变量在对应内存地址中的值，其命令格式为：`x/[n][f][u] addr`。

#### 可选参数

- `n` 表示要打印的内存单元个数，默认值为1；

- `f` 表示上面介绍的各种格式控制符；
- `u` 表示要打印的内存单元长度; 常用的内存单元包括：`b` (单字节)、`h` (双字节)、`w` (四字节)、`g` (八字节)。

#### 参数

- `addr` 表示变量的**内存地址** (注意：不能是变量名)

#### 用法

如果需要以单字节为单位，以**二进制的方式打印**变量val，可以使用 GDB 命令 `x/4tb &val`；

同样地，`x/10dw array` 可以四字节的方法打印以 array 地址开始的连续10个内存单元的值。

### 其他

`display` 命令可以用于当程序被 GDB 暂停时自动打印某变量的值。

`info registers` 命令可以查看除浮点寄存器外的所有寄存器值。

## 查看代码 list

详见：https://sourceware.org/gdb/onlinedocs/gdb/List.html

### 显示行

`list num`num指定哪一行代码显示在当中。

### 显示函数

`list function`

### 设置显示的行数

```
set listsize count
set listsize unlimited
```

Make the `list` command display count source lines (unless the `list` argument explicitly specifies some other number). 

Setting count to `unlimited` or `0` means there’s no limit.

## 断点 break

### 断点设置

FMI: https://sourceware.org/gdb/onlinedocs/gdb/Set-Breaks.html#Set-Breaks

`break location`

Set a breakpoint at the given location, which can specify a function name, a line number, or an address of an instruction. 

When using source languages that permit overloading of symbols, such as C`++`, a function name may **refer to more than one possible place** to break. See [Ambiguous Expressions](https://sourceware.org/gdb/onlinedocs/gdb/Ambiguous-Expressions.html#Ambiguous-Expressions), for a discussion of that situation.

#### 条件断点

`break … if cond`

evaluate the expression cond each time the breakpoint is reached, and stop only if the value is nonzero—that is, if cond evaluates as true. ‘…’ stands for one of the possible arguments described above (or no argument) specifying where to break. See [Break Conditions](https://sourceware.org/gdb/onlinedocs/gdb/Conditions.html#Conditions), for more information on breakpoint conditions.

# 参考链接

英文详细教程：https://sourceware.org/gdb/onlinedocs/gdb

print：https://vimjc.com/gdb-print-variable.html

一些设置：https://www.cnblogs.com/rosesmall/archive/2012/04/12/2444431.html







