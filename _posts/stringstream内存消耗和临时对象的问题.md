---
title: stringstream内存消耗和临时对象的问题
date: 2019-07-08 16:23:30
toc: true
comments: true
categories: 
- C/C++
tags: 
- stringstream
---

记录stringstream在使用上的2个坑。

## 反复读写大量数据时的内存消耗

由于stringstream构造函数会特别消耗内存，似乎不打算主动释放内存(或许是为了提高效率)，但如果你要在程序中用同一个流，反复读写大量的数据，将会造成大量的内存消耗，因些这时候，需要适时地清除一下缓冲 (用 stream.str("") )。
比如，如下代码：
```cpp
#include<iostream>
#include<sstream>
#include<string>
using namespace std;
int main()
{
    std::stringstream ss;
    char buffer[16];
    int index = 0;
    while(index < 10)
    {
        ss.clear();
        //ss.str("");
        sprintf(buffer, "round%d;", index);
        string str_input(buffer);
        ss << str_input;
        cout << "ss.str():" << ss.str() << endl;
        string str_output;
        ss >> str_output;
        cout << "str_output:" << str_output << endl;
        ++index;
    }
    return 0;
}
```
其输出为：
```bash
ss.str():round0;
str_output:round0;
ss.str():round0;round1;
str_output:round1;
ss.str():round0;round1;round2;
str_output:round2;
ss.str():round0;round1;round2;round3;
str_output:round3;
ss.str():round0;round1;round2;round3;round4;
str_output:round4;
ss.str():round0;round1;round2;round3;round4;round5;
str_output:round5;
ss.str():round0;round1;round2;round3;round4;round5;round6;
str_output:round6;
ss.str():round0;round1;round2;round3;round4;round5;round6;round7;
str_output:round7;
ss.str():round0;round1;round2;round3;round4;round5;round6;round7;round8;
str_output:round8;
ss.str():round0;round1;round2;round3;round4;round5;round6;round7;round8;round9;
str_output:round9;
```
可以看出``stringstream``是以追加的方式读取输入数据，反复读写大量数据时，会消耗大量内存（然而str_output是符合预期的）。

改进方法：使用``ss.str("")``清空数据。
取消注释代码中的``ss.str("");``后，输出如下：
```bash
ss.str():round0;
str_output:round0;
ss.str():round1;
str_output:round1;
ss.str():round2;
str_output:round2;
ss.str():round3;
str_output:round3;
ss.str():round4;
str_output:round4;
ss.str():round5;
str_output:round5;
ss.str():round6;
str_output:round6;
ss.str():round7;
str_output:round7;
ss.str():round8;
str_output:round8;
ss.str():round9;
str_output:round9;
```

## stringstream.str()返回临时对象的坑

指针指向一个临时对象时，引发的问题。
```cpp
#include<iostream>
#include<sstream>
#include<string>
using namespace std;
int main()
{
    stringstream ss("0123456789");
    stringstream t_ss("abcdefg");
    string str1(ss.str());

    const char* cstr1 = str1.c_str();
    const char* cstr2 = ss.str().c_str();    //创建了一个临时对象，并释放
    const char* t_cstr = t_ss.str().c_str(); //在被释放的空间上，重新创建了一个临时对象，并释放

    //cout << ss.str().c_str() << endl;

    cout << cstr1 << endl;
    cout << cstr2 << endl;
    cout << t_cstr << endl;

    return 0;
}                       
```
输出为：
```bash
0123456789
abcdefg
abcdefg
```
第12行创建了一个临时对象并随即销毁，``cstr2``指向被销毁的地址空间，第13行在前面被销毁的空间上，又创建了一个临时对象并销毁，``t_cstr``同样指向被销毁的地址空间，即``cstr2``和``t_cstr``指向同一个被销毁的地址空间，而这个被销毁的地址空间最后一次赋值为``"abcdefg"``，所以，``cstr2``和``t_cstr``输出均为``"abcdefg"``。

如果取消第15行的注释，输出将变成：
```bash
0123456789
0123456789
0123456789
0123456789
```
原因雷同，因为第15行也创建了一个临时对象并随即销毁，该临时地址空间最后一次赋值变为``"0123456789"``。

怎么解决这个问题呢？使用引用来引用临时对象。
上述问题代码可以修改为：
```cpp
int main()
{
    stringstream ss("0123456789");
    stringstream t_ss("abcdefg");
    string str1(ss.str());

    const char* cstr1 = str1.c_str();
    const string &str2 = ss.str();
    const char* cstr2 = str2.c_str();
    const string &t_str = t_ss.str();
    const char* t_cstr = t_str.c_str();

    cout << ss.str().c_str() << endl;
    cout << cstr1 << endl;
    cout << cstr2 << endl;
    cout << t_cstr << endl;

    return 0;
}
```
此时输出符合预期：
```bash
0123456789
0123456789
0123456789
abcdefg
```

## 参考文献
[关于stringstream 的一些坑](http://blog.wangcaiyong.com/2016/10/16/stringstream/)
[小心stringstream.str()字符串用法的陷阱](https://www.ibm.com/developerworks/community/blogs/12bb75c9-dfec-42f5-8b55-b669cc56ad76/entry/_e5_b0_8f_e5_bf_83stringstream_str__e5_ad_97_e7_ac_a6_e4_b8_b2_e7_94_a8_e6_b3_95_e7_9a_84_e9_99_b7_e9_98_b13?lang=en)
