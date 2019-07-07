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

示例2: Test结构体占24个字节
x86_64指针占8字节
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
解释如下：
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

## 如何定义结构体以提高存储空间利用率
1. 按照存储空间占用情况由大到小定义结构体变量；
2. 将相同数据类型的成员放到一起，可以减少编译器为了对齐而添加填充字符。

## 参考
[知乎：如何计算结构体大小？](https://www.zhihu.com/question/28958350)
[结构体的内存分配机制](https://www.cnblogs.com/plxx/p/3382588.html)
