---
title: C/C++位域结构
date: 2019-07-07 20:10:30
comments: true
categories: 
- C/C++
tags: 
- 结构体
- 位域
---

关于位域结构，[C/C++位域结构深入解析](https://jocent.me/2017/07/24/bit-field-detail.html)这篇文章进行了详细的解读，我就不一一摘抄了。在此处列举一些tips方便查阅：
1. 位域结构可以用来压缩存储；
2. 位域的成员需要为无符号类型；
3. 如果相邻位域字段的类型不同，仍然会压缩（ubuntu 16.04）。
比如，这里Test结构体占用4个字节。
```cpp
struct Test{
    unsigned int a : 20;
    unsigned char b : 3;
    unsigned int c : 2;
};
```

再解释一下参考文献中示例的输出：

```cpp
struct BitFieldDemo
{
    unsigned char a : 2;
    unsigned char b : 3;
    unsigned char c : 3;
};
// 取位域大小，字节单位
int nsize = sizeof(struct BitFieldDemo);

// 位域定义及其赋值
struct BitFieldDemo BFD;    /* or = {0x3, 0x5, 0x2}*/
BFD.a = 0x3;
BFD.b = 0x5;
BFD.c = 0x2;

printf("BFD.a = %d, BFD.b = %d, BFD.c = %d \n", BFD.a, BFD.b, BFD.c);
printf("BFD = %d \n", *((unsigned int *)(&BFD)));

// 内存拷贝
char szBuffer[100] = "abcdefghijklmnopqrstuvwxyz0123456789";

struct BitFieldDemo *pBFD = NULL;
pBFD = (struct BitFieldDemo *)malloc(sizeof(struct BitFieldDemo));
if (pBFD != NULL)
{
    memcpy(pBFD, szBuffer, sizeof(struct BitFieldDemo));

    printf("a = %d, b = %d, c = %d \n", pBFD->a, pBFD->b, pBFD->c);
    printf("*pBFD = %d \n", *((unsigned int *)(pBFD)));
    
    free(pBFD);
    pBFD = NULL;
}
```
输出为：
```cpp
BFD.a = 3, BFD.b = 5, BFD.c = 2 
BFD = 343 
a = 1, b = 0, c = 3 
*pBFD = 97
```
为什么第三行输出为``a = 1, b = 0, c = 3 ``呢？

``memcpy``函数对字符串从低位到高位拷贝，所以将字符a拷贝到了``pBFD``指向的地址空间中，字符a的ASCII码为``0110 0001``，结构体中成员地址由低到高存放，因此：
```cpp
01 : pBFD->a
000: pBFD->b
011: pBFD->c
```
即``a = 1, b = 0, c = 3 ``。

为什么第二行输出为``BFD = 343``呢？
这个我也无法理解，等待大佬解读。。。
