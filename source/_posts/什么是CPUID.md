---
title: 什么是CPUID
top: false
cover: false
toc: true
mathjax: true
date: 2021-05-23 18:43:47
password:
summary:
tags: 底层
categories: 
  - 项目
---



# 什么是CPUID

x86芯片结构中，CPUID是处理器提供的一个指令操作码，能够让软件利用它分析出处理器的信息。比如，一个程序可以使用CPUID查看处理器的类型和是否能够使用MMX/SSE指令集。

## 历史

在CPUID出现之前，程序员需要费劲地写机器码来分析处理器的具体信息。80386处理器出现后，EDX的重置等操作可以提供一些信息，但是这并不是程序标准的读取信息方式。其他不是x86体系的处理器，程序员也是同样需要复杂的机器码来探测cpu是否支持某个信息。

当CPUID在x86家族出现后，很多其他体系结构比如ARM也提供了芯片上的寄存器来描述处理器类型信息。

## 调用 CPUID

在嵌入式语言中，会使用`EAX`寄存器分析处理器主要的信息。在Intel的术语中，这个叫做CPUID leaf。在使用CPUID之前，首先应该将EAX设置为0：`EAX=0`，然后CPU会将值放入EAX寄存器中。例如下图：

![image-20230104140116624](C:\Users\xuxiaohu\AppData\Roaming\Typora\typora-user-images\image-20230104140116624.png)

查阅寄存器中不同的位置来决定不同的cpu信息。CPU的stepping，model和family 信息在EAX中，feature flags在寄存器EDX和ECX中，额外的信息在寄存器EBX中。

使用例子：

得到信息：

```c++
#ifndef CPUID_H
#define CPUID_H

#ifdef _WIN32
#include <limits.h>
#include <intrin.h>
typedef unsigned __int32  uint32_t;

#else
#include <stdint.h>
#endif

class CPUID {
  uint32_t regs[4];

public:
  explicit CPUID(unsigned i) {
#ifdef _WIN32
    __cpuid((int *)regs, (int)i);

#else
    asm volatile
      ("cpuid" : "=a" (regs[0]), "=b" (regs[1]), "=c" (regs[2]), "=d" (regs[3])
       : "a" (i), "c" (0));
    // ECX is set to zero for CPUID function 4
#endif
  }

  const uint32_t &EAX() const {return regs[0];}
  const uint32_t &EBX() const {return regs[1];}
  const uint32_t &ECX() const {return regs[2];}
  const uint32_t &EDX() const {return regs[3];}
};

#endif // CPUID_H
```

使用信息：

```c++
#include "CPUID.h"

#include <iostream>
#include <string>

using namespace std;

int main(int argc, char *argv[]) {
  CPUID cpuID(0); // Get CPU vendor

  string vendor;
  vendor += string((const char *)&cpuID.EBX(), 4);
  vendor += string((const char *)&cpuID.EDX(), 4);
  vendor += string((const char *)&cpuID.ECX(), 4);

  cout << "CPU vendor = " << vendor << endl;

  return 0;
}
```

## 参考

1. [CPUID - Wikipedia](https://en.wikipedia.org/wiki/CPUID)
2. [assembly - CPUID implementations in C++ - Stack Overflow](https://stackoverflow.com/questions/1666093/cpuid-implementations-in-c)







