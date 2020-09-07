---
title: 侯捷C++手把手教学(上)
---
# 头文件与类的声明

## C和C++的区别
在C当中数据和函数是分离的，所有函数都能访问其他数据

在C++当中，变量和函数包装在类当中，通过private和public来控制访问

## C++ 关于数据和函数
### complex类
数据：
实部，虚部

操作：
加，减，乘，除，共轭，正弦

### string类
数据：
字符（一个指针指向一串字符）

操作：
拷贝，输出，添加，插入

## 头文件引用
```C++
#include <iostream>
//标准库
#inlcude "test.h"
//头文件
```

## 头文件防卫式声明
```C++
#ifndef __COMPLEX__
#define __COMPLEX__
#include <cmath>

class ostream;
class complex;
complex& __doapl(complex* ths,const complex& r);
//前置声明

class complex
{
//类声明
public:
    complex (double r = 0,double i = 0): re (r), im (i){}
    complex& operator += (const complex&);
    double real () const {return re;}
    //const 代表不改变类的成员变量
    double imag () const {return im;}
    //自动认为是inline
private:
    double re, im;
    friend complex& __doapl (complex*,const complex&);
};

inline double
imag(const complex& x)
{
    return x.imag();
}
//函数重载
#endif
```

# 构造函数

## 什么时候会产生inline
函数在classbody内定义完成，自动成为inline候选人。

## 访问级别(access level)
希望被外部访问到public，不希望在外部被访问到private，可以交错写。

## 构造函数
```C++
complex (double r = 0, double i = 0): re(r), im(i) {}
```
没有返回类型，函数名称就是类名称。re(r)和im(i)叫做初值列(initialization list)，这样写比赋值操作更好。

### 重载
```C++
complex () : re(0), im(0) {} //invalid
void real(double r) {re = r;} //valid
```
允许同名函数(在编译器看来不同名)，不允许歧义调用。

## constructor(ctor，构造函数)被放在private区域
```C++
Class A{
public:
    static A& getInstance();
    setup(){}
private:
    A();
    A(const A& rhs);
};

A& A::getInstance()
{
    static A a;
    return a;
}
```
# 参数传递与返回值
未完待续