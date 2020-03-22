---
layout: post
title:  "Kernel ret2dir"
date:   2020-3-19
desc: ""
keywords: ""
categories: [Binary]
tags: []
icon: icon-html
---

# Kernel ret2dir

> ~~d i r 溢 出~~

先看神仙：https://www.usenix.org/conference/usenixsecurity14/technical-sessions/presentation/kemerlis

再学洋文

ret2dir return-to-direct-mapped memory

算了不学了

👴这就去重修带学洋文

## 👴为什么能ret2usr

linux中用户和内核共享虚拟地址，划分为用户空间和内核空间，x86和AArch32中，3G/1G划分，0xC0000000到0xffffffff 1GB为内核空间/0x00000000到0xbfffffff 3GB为用户空间。

x86-64和AArch64中，0xffff800000000000到0xffffffffffffffff 128TB为内核空间，0x0000000000000000到0x00007ffffffff000 128TB为用户空间

内核空间和用户空间的隔离⑧行，弱的隔离有利于交互但是容易被日

内核有权限访问任意地址，👴能控制用户空间，又能通过白给漏洞控制内核中的返回地址，函数指针等数据，就可以控制内核执行用户空间中👴构造的代码

![1](https://raw.githubusercontent.com/AiDaiP/images/master/ret2dir/1.jpg)



## 👴为什么不能ret2dir

因为添了保护措施

### PaX

据说是最牛逼的防御系统级别0day的方案，防止控制流转移并解除内核空间到用户空间的引用

KERNEXEC和UDEREF两种特性，依赖于将内核空间映射到一个1G的的段中，当特权代码尝试去解引用非内核空间的指针或者从非内核空间提取指令时返回一个内存错误

x86-64中，缺少分段支持，执行进入内核空间时，KERNEXEC/amd64将用户空间用户空间映射到一个不可执行的区域，从内核空间退出后恢复

### SMEP/SMAP/PXN

SMAP

Supervisor Mode Access Prevention，管理模式访问保护，禁止内核访问用户空间的数据

SMEP

Supervisor Mode Execution Prevention，管理模式执行保护，禁止内核执行用户空间的代码

PXN

Privileged Execute Never 和SMEP差不多一个意思

### kGuard

跨平台的编译器扩展

通过在编译时增强动态控制流断言(CFA)，防止在运行时特权执行路径到用户空间的无限制转换

运行时，在每个分支，注入的CFA检查目标地址是在内核空间还是用户空间

![2](https://raw.githubusercontent.com/AiDaiP/images/master/ret2dir/2.jpg)

## ~~👴为什么还能ret2dir~~

~~因为👴bypass~~

那咋办🐎



## physmap

![4](https://raw.githubusercontent.com/AiDaiP/images/master/ret2dir/6.jpg)



physmap是内核空间中一个大的，连续的虚拟内存空间，直接映射部分或所有(取决于具体架构)的物理内存，物理地址可以在physmap中找到其对应的虚拟内存地址

物理内存可以同时分配给用户空间和内核空间，有可能存在两个虚拟地址映射到同一个物理地址，其中一个虚拟地址在physmap，一个在用户空间。当两块以上虚拟内存映射到同一物理地址时，就会产生同义词。

x86所有内核版本中physmap可读可写。x86-64中，3.8.13之前的内核，physmap可读可写可执行，3.9之后没有执行权限。AArch32和AArch64中physmap可读可写可执行



linux内核使用伙伴系统+slub分配器分配内存，内核内存分配有kmalloc和vmalloc两种方式

kmalloc在字节级分配，需要保证虚拟地址和物理地址都连续

vmalloc请求页的倍数大小的内存，需要保证虚拟地址连续，不需要物理地址连续

vmalloc⑧行，主要用kmalloc







## ret2dir

ret2dir的核心原理是physmap地址同义词，👴可以通过physmap直接调用在用户空间布置的shellcode，绕过所有防止ret2usr的保护措施

![5](https://raw.githubusercontent.com/AiDaiP/images/master/ret2dir/5.jpg)



### bypassing SMAP and UDEREF

在一个存在SMAP或UDEREF的x86系统中，假设👴可以重写内核中的一个指针kdptr，并且可以解引用

存在SMAP或UDEREF，👴搞不成，👴选择ret2dir

👴控制的用户进程在用户空间0xBEEF000有一个页(4kb)，映射到某个物理地址，👴在0xBEEF000布置payload，👴又在内核中申请内存并和0xBEEF000映射到同一个物理地址，假设分配了1904页框，physmaps起始地址为0xC0000000，这个页框映射地址为0xC0000000+(4096*1904)=0xC0770000

0xBEEF000和0xC0770000是同义词关系，任何在0xBEEF000~0xBEEFFFF的数据（原文写的是0xBEEF000~0xBEEFFFFF，👴觉得应该是0xBEEFFFF⑧）都可以通过0xC0770000~0xC0770FFF来访问，保护措施认为0xC0000000以上地址解引用都是合法的

👴把kdptr覆盖为0xC0770000，通过kdptr解引用可以访问用户空间数据，绕过了SMAP和UDEREF

有龙鸣翻译把by overwriteing kdkdptr with 0xC0770000翻译成通过重写0xC0770000来覆盖kdptr，👴吐了，建议重修带学洋文

### bypassing SMEP, PXN, KERNEXEC, and kGuard

在一个存在SMEP, PXN, KERNEXEC, kGuard的x86-64系统中，假设👴可以重写内核中的一个函数指针kfptr，👴控制的用户进程在用户空间0xBEEF000有一个页(4kb)，映射到某个物理地址，👴在0xBEEF000安排上payload，👴又在内核中申请内存并和0xBEEF000映射到同一个物理地址，假设分配了190402页框，physmaps起始地址为0xFFFF8800000000000，这个页框映射地址为0xFFFF88000000000+(4096*190402)=0xFFFF87FF9F0800000

0xBEEF000和0xFFFF87FF9F0800000是同义词关系，保护措施认为0xFFFF8800000000000以上地址解引用都是合法的

👴把kdptr覆盖为0xFFFF87FF9F0800000，通过这个函数指针调用用户空间代码

同义词关系并不影响权限，0xFFFF87FF9F0800000不可执行，不会因为0xBEEF000可执行而变成可执行，但是对于physmap可执行的龙鸣系统，👴就是可以直接跳shellcode，那咋办🐎

对于physmap不可执行的系统，👴可以rop

👴在0xFFFF87FF9F0800000布置好rop链，把kdptr覆盖为一个神奇的gadget的地址，利用kdptr对着rsp一顿操作，把内核栈劫持到0xFFFF87FF9F0800000，跑rop链，成了



## Locating Synonyms

ret2dir的一个带问题，如何拿到physmap中同义词的地址

### 泄露PFN

/proc/(pid)/pagemap白给

pagemap是一个64位的值

```c
    * Bits 0-54  page frame number (PFN) if present
    * Bits 0-4   swap type if swapped
    * Bits 5-54  swap offset if swapped
    * Bit  55    pte is soft-dirty (see Documentation/vm/soft-dirty.txt)
    * Bit  56    page exclusively mapped (since 4.2)
    * Bits 57-60 zero
    * Bit  61    page is file-page or shared-anon (since 3.5)
    * Bit  62    page swapped
    * Bit  63    page present
```

假设一个页4kb

SYN(uaddr) = PHYS_OFFSET + 4096*(PFN(uaddr) - PFN_MIN)

PHYS_OFFSET是physmap起始地址

PFN_MIN是第一个页框数，因为很多架构中物理内存都不是从0开始的

#### Sizeof(RAM) > sizeof(physmap)

在4G RAM的x86系统中，PFN范围0x0-0x100000，physmap仅能映射RAM的前891M，PFN范围0x0-0x37B00(PFN_MAX)

在用户空间不断请求大块RW内存，并触发write fault，强迫内核分配物理内存，直到PFN小于PFN_MAX，保证同义词出现在physmap中

#### 连续同义词地址

👴的payload需要多个页面

页在用户空间连续，映射到物理内存不一定连续，所以同义词地址也不一定连续。

具有连续PFN的页在物理内存连续，同义词地址也连续

用户空间中0xBEEF000和0xFEEB000映射到物理内存的页PFN为0x2E7C2和0x2E7C3，他们在用户空间中相差64M，但在物理内存中连续，在physmap也连续



### physmap喷射

> ~~众所周知，喷子是版本答案~~

pagemap没有白给，无法直接确定physmap中同义词的位置

不断使用mmap申请内存，并将payload放入申请的内存中