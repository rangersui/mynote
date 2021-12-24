[toc]

# 使用vscode对C/C++项目进行编译和调试

## 单文件

[安装MinGW](https://blog.csdn.net/wxh0000mm/article/details/100666329)，并将你的MinGW加入到系统的环境变量当中。

打开命令行，输入：

```bash
gcc -v
```

检查是否能正常读取gcc版本号，完成此步**代表环境变量已经生效**。

进入vscode并安装**code runner**插件。

编写C/C++代码并按下F5，会自动进行编译和调试，会自动生成一个.vscode目录，其中保存launch和task两个json配置文件。task主要管理编译，launch负责管理调试和执行。

## 多文件

配置tasks和launch文件时真的很烦，所以换一种方法，**使用CMake来进行多文件的编译和调试**。

安装两个插件:**CMake**和**CMake Tools**,其中CMake用于语法高亮和自动补全，CMake Tools生成CMake项目、构建CMake项目。

简单来讲CMakeFile就和各种IDE里的Project差不多，而且优点是所有IDE都可以读取CMake项目。不过CMake file也不是那么好写，但是有了生成工具，没有什么特殊要求的话就还好。

安装完两个插件以后，按下Ctrl+Shift+p输入cmake:q会自动跳出CMake: Quick Start选项，选择自己使用的编译器，GCC/Clang都可以，我现在用的是GCC。然后输入项目名称，创建可执行文件（Executable）然后就会自动生成一个CMake项目。其中：

- build文件夹是`cmake`指令的输出文件夹
- 默认生成的`CMakeLists.txt`文件
- 默认生成的`main.cpp`文件

编译、调试和执行按钮都在最底下栏，图片来自[VSCode与CMake搭配使用之基本配置](https://blog.csdn.net/jiasike/article/details/107474368)

![](https://img-blog.csdnimg.cn/20200720213518920.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ppYXNpa2U=,size_16,color_FFFFFF,t_70#pic_center)

至于如何写CMakeFile，可能需要找一篇专门的文章进行学习，这里不进行过多赘述了。也许之后会直接加进这个文档里面。

以我的测试项目为例：

```cpp
//test.h
#include<string>
#ifndef TEST
#define TEST
class student{
public:
    int number;
    std::string name;
    student():number(0),name(""){}
    student(int num,std::string Name):number(num),name(Name){}
};
#endif
```

```cpp
//main.cpp
#include <iostream>
#include "test.h"
int main(int, char**) {
    student stu1(1,"Cao");
    std::cout << stu1.name;
}
```

```cmake
cmake_minimum_required(VERSION 3.0.0)
project(myCmakeTest VERSION 0.1.0)

include(CTest)
enable_testing()

add_executable(myCmakeTest main.cpp test.h)

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
include(CPack)
```



所以怎么使用vscode代替vs进行单文件或者多文件的编译和调试就讲到这里了，以下是答疑时间：

## 什么是MinGW?

*MinGW*，是Minimalist GNUfor Windows的缩写。

# Clang与LLVM的关系

LLVM是low level virtual machine的简称，是一个编译器框架。LLVM的主要作用是作为多种语言的后端，提供可编程语言无关的优化和针对多种CPU的代码生成功能。包含了多种*子项目*，**其中包含最著名的Clang**。

## 什么是编译器前端和后端？

编译器粗略分为

- 词法分析

- 语法分析

- 类型检查

- 中间代码生成

  ------分割线------

- 目标代码生成

- 目标代码优化

我们把中间代码生成及其以前称为编译器前端，后端与前端是独立两部分。

- **前端是编译器对程序代码的分析和理解过程与语法有关。**

- **后端是生成目标代码的过程，和目标机器有关**

# Clang和GCC

GCC（GNU Compiler Collection，GNU 编译器套装），是一套由 GNU 开发的编程语言编译器。GCC 原名为 GNU C 语言编译器，因为它原本只能处理 C语言。

LLVM由UIUC的大神Chris Lattner在攻读博士学位时完善，使用**GCC 作为前端来对用户程序进行语义分析**产生 IF（Intermidiate Format, 中间格式），然后 **LLVM 使用分析结果完成代码优化和生成**。这项研究让他在 2005 年毕业时就成为了业界小有名气的编译器专家，他也因此早早地被 Apple 盯上，最终成为其编译器项目的骨干。

Apple 招募了 Chris Lattner ，打算从零开始写 C、C++、Objective-C **语言的前端 Clang，以完全替代掉 GCC**。

# 编译和调试

## 编译过程

源程序经过**预处理器**，把存放在独立文件中的多个模块聚合在一起，并且将宏语句转化为源语言的语句。经过预处理器的源程序输入**编译器**（应该是前端和后端都包含了），产生*汇编语言*作为输出。汇编语言由**汇编器**进行处理，生成*可重定位的机器代码*。大型程序经常被分成多个部分进行编译，可重定位的机器代码有必要和其他可重定位的目标文件以及库文件连接到一起，形成可运行的目标机器代码。一个文件中的代码可能指向的是另一个文件中的位置，**链接器(linker)**能够解决外部内存地址的问题，**加载器(loader)**把所有的可执行目标文件放到内存中执行。

## 调试工具

GDB（GNU symbolic debugger）是一个**由GNU开源组织发布**的、UNIX/LINUX操作系统下的、基于命令行的、功能强大的程序调试工具。可以通过命令行手动对程序进行断点调试，详见[linux下gdb调试方法与技巧整理](https://blog.csdn.net/niyaozuozuihao/article/details/91802994)。

GDB和GCC的关系就是都是由GNU开源组织发布的，**两者不形成绑定**。顺带一提GNU是一个笑点奇怪的梗，因为GNU的全称为GNU's Not Unix，嗯，它是递归缩写……

除了GDB以外也有别的调试工具，不做详细介绍。

## 可重定位是什么意思？

代码里的各种跳转代码/指令，比如C语言里的goto，汇编里的jmp、jz等等，它们都是跳到某一地址，然后在该地址继续往下执行代码 的意思，我们写的代码时用的内存空间是逻辑空间，但是代码在实际运行时，用到的却是货真价实的物理地址空间。

因为操作系统给进程分配的内存的起始位置是无法确定的，换句话说，也就是程序 在运行时实际的物理起始位置是不确定，所以不能在编译时就把地址给写死，否则，如果实际运行时物理空间起始位置与编译时写死的起始地址 不一致的话，程序就会出问题。

如果我们编译时，涉及地址跳转，某个地址对应信息的读取、写入等等之类与地址有关的操作，代码里所有地址都采用动态调整的方式，也就是可以根据操作系统实际给进程分配的 实际物理内存 的起始位置 ，而进行调整的话，那么代码就不会出错啦。

这种可以使地址平移的代码就叫做可重定位代码，它是在加载的时候，也就是**系统给进程确定了物理地址时，才生成绝对地址**

# 参考文献

[详解三大编译器：gcc、llvm 和 clang](https://zhuanlan.zhihu.com/p/357803433)

[深入研究Clang（一）Clang和LLVM的关系及整体架构](https://zhuanlan.zhihu.com/p/26223459)

[编译器前端和后端](https://www.jianshu.com/p/de1fbf4ff764)

AlfredV.Aho. 编译原理.第2版[M] 的引论(关于我买回来两个月的紫龙书终于拆封了这件事

[编译原理之可重定位代码是什么意思？](https://blog.csdn.net/yesyes120/article/details/78944991)

[VSCode与CMake搭配使用之基本配置](https://blog.csdn.net/jiasike/article/details/107474368)

