---
title: C/C++结构体对齐
date: 2019-07-07 17:53:30
toc: true
comments: true
categories: 
- C/C++
tags: 
- 结构体
---

## 对齐规则
1. 结构体每个成员相对于结构体首地址的偏移量都是成员大小的整数倍；
2. 结构体的总大小为结构体最宽基本类型成员大小的整数倍；
3. 结构体变量的首地址是结构体中最宽数据类型的整数倍


## 示例
示例1: Test结构体占12个字节
```cpp
struct Test{
    char a;  //1 + 补3
    int b;   //4
    char c;  //1 + 补3
};  //12 Bytes
```

示例2: Test结构体占24个字节（x86_64指针占8字节）
```cpp
struct Test{
    char a;  //1 + 补3
    int b;   //4
    char c;  //1 + 补3
    int *d;  //补4 + 8
};  //24 Bytes
```

示例3: Test结构体占8个字节
```cpp
struct Test{
    int a;   //4
    char b;  //1
    char c;  //1
};  //6 + 补2 Bytes
```

示例4: Test结构体占48个字节(x86_64)
```cpp
struct Test
{
    char a;
    float b;
    int c;
    double d;
    unsigned e;
    int *f;
    double *g;
};
```
理解如下：
```cpp
struct Test {
    char a;             // 1 bytes
    char padding1[3];   // 3 bytes
    float b;            // 4 bytes
    int c;              // 4 bytes
    char padding2[4];   // 4 bytes
    double d;           // 8 bytes
    unsigned e;         // 4 bytes
    char padding3[4];   // 4 bytes
    int *f;             // 8 bytes
    double *g;          // 8 bytes
    
};
```

## 为什么要对齐
许多计算机系统对基本数据类型合法地址做出了一些限制，要求某种类型对象的地址必须是某个值K（通常是2、4或8）的倍数。这种对齐限制简化了形成处理器和存储器系统之间接口的硬件设计。
1. CPU在访问存储器的时候只能在某些地址处获取某些特定类型的数据。
2. CPU对内存的读写并非最小粒度（1byte）寻址，而是4bytes或8bytes,或者更多。如果数据不按照一定的规则存储的话，会降低读取速度，从而影响计算效率。
3. 跨平台移植：不同的硬件平台读写数据的方式不同，为了兼容性，需要做内存对齐。

## 如何定义结构体以提高存储空间利用率
1. 按照存储空间占用情况由大到小定义结构体变量；
2. 将相同数据类型的成员放到一起，可以减少编译器为了对齐而添加填充字符。

Tips
1. 每个特定平台上的编译器都有自己的默认“对齐系数”，可以通过预编译命令#pragma pack(n)。
2. 查看机器是32位还是64位用``uname -a``命令。
3. gdb查看结构体各成员内存地址。

## 参考
[知乎：如何计算结构体大小？](https://www.zhihu.com/question/28958350)
[结构体的内存分配机制](https://www.cnblogs.com/plxx/p/3382588.html)
