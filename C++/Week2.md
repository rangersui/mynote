# 第一周（从C到C++ B）
## 函数重载
匹配重载函数的规则 C++ 按以下三个步骤次序寻找匹配的重载函数
1. 根据函数类型寻找严格匹配
2. 通过数据类型兼容性的隐式转换（自动转换）寻找一个匹配
3. 通过用户自定义的类型转换寻找一个匹配

若以上三种方法都不呢个匹配一个重载函数，则编译器放弃匹配并给出错误报告。

## 参数带默认值的函数
一个函数中可以有多个参数带默认值
1. 无默认值的参数不能缺省实参，实参从左边一次初始化形参，其余的形参使用默认值
2. 因此，函数设计时应将带默认值的形参全部安排在参数表的右边
   
notice:
1. 参数的默认值只应该出现在函数声明中
2. 有函数原型声明时，函数定义中不应该出现参数默认值
3. 若无函数原型声明函数，允许在函数定义时声明参数默认值。

## 内联函数
允许一些小巧的，常用的函数在编译时直接内嵌到函数体内，在调用点“就地展开”

不允许含有复杂的流程控制语句（switch，while）。递归函数也不行。

inline对编译器提出一个建议，是否采纳建议由编译系统决定。

## 函数模板
函数模板只是描述了对某种假想的数据类型的数据将要进行的操作
1. 编译系统不会编译函数模板本身，因此，我们称模板为声明（它仅描述了可能进行的操作），而非定义。
   
当编译系统遇到要用具体的实际参数使用函数模板时
1. 用实际参数的数据类型取代形式数据类型
2. 根据函数模板的执行语句，自动定义一个函数
3. 对生成的模板函数进行编译（编译系统只检查模板函数的语法）

```C++
#include <iostream>
 using namespace std;
 int main()
 {
    int a=3, b=5;
    char c=’3’, d=’5’;
    double x=3.5, y = 5.5;
    char s[10]=”ABCD”, t[10]=”abcd”;

    cout << Max(a, b) << endl;
    cout << Max(c, d) << endl;
    cout << Max(x, y) << endl;
    cout << Max(s, t) << endl;
    return 0;
 }
/*output
5
5
5.5
ABCD <-显然A是小于a的，为什么输出的是ABCD呢？
*/
//系统生成的模板如下
char *Max(char *a, char *b) //T为 char*
{ 
    return (a>b) ? a : b; //比较两个地址值
}

```
因此，在上面的例子中，比较函数的地址大小是没有意义的，因此需要重载函数。**C++允许重载模板函数**并且规定：编译系统首先匹配重载模板函数，而不会生成之前的错误的函数。
```C++
char * Max(char *a,char *b)
{
    return strcmp(a,b)>0?a:b;
}
```
## 动态变量和动态数组
堆(heap)是程序数据区的一种。作为一种系统资源，可以实现
1. 按需申请使用
2. 用后归还

## 申请以及释放动态变量（堆变量）
语法
```C++
new typename
delete typename
```
$\color{#87CEFA}{new}$是运算符，C++保留字

执行new操作，将在对空间中切割出($\color{#87CEFA}{sizeof}$(数据类型))字节的内存空间
1. 该运算返回切割的堆空间首地址。该地址应该被保存，以便delete
2. 操作不成功，返回NULL

堆变量没有原名，常用间接名
```C++
double *p;
p = new double;
*p = 3.14;
cout<<*p<<endl;
delete p;
```

## 申请以及释放动态变量（堆变量）
语法
```C++
new 数据类型[元素个数]
delete [] 堆地址
```
在堆空间切下 元素个数*$\color{#87CEFA}{sizeof}$(数据类型)字节的内存空间，元素个数可以不是常量。
1. 操作成功，返回首地址，需要用$\color{#87CEFA}{delete}$ []释放空间。
2. 操作不成功，返回NULL

$\color{#87CEFA}{new}$返回的地址为堆数组的首地址（相当于堆数组名），加上数据类型，元素个数，成为堆数组三要素。
```C++
void f()
{
    int n;
    double *array;
    cin>>n;
    array = new double[n];
    if(array==NULL)return;
    for(int i=0;i<n;i++)
    {
        array[i] = (i+1)*3.1415926;
        cout<<array[i]<<" ";
    }
    delete [] array;
}
```
## 动态二维数组
定义数组时，数组的元素个数必须是常量

动态数组（堆数组）能在程序运行时根据需要创建数组。即数组的元素个数可以成为变量或表达式的值。
1. C++的new操作运算成功的结果是返回堆中的某个变量值，一般为一个一级地址，对应一维数组。

建立动态二维数组方案
1. 所有元素连续存放
2. 同一行的元素连续存放（不同行不一定连续）

关键点
1. 储存每一行的首地址。需建立一个地址的索引——指针数组
2. 用完后释放

```C++
template <typename T>
T **&New2DArray(T **&array,int row,int col)
{
    array = (T**)new char[row*sizeof(void*)];
    if(array==NULL)return array;
    array[0] = new T [row*col];
    if(array[0]==NULL)
    {
        delete [] array;
        return array=NULL;
    }
    for(int i=1; i<row; i++)
        array[i] = array[0]+i*col;
    return array;
}
template<typename T>
void Delete2DArray(T**&array)
{
    if(array[0]) delete [] array[0];
    if(array) delete [] array;
    array = NULL;
}
```