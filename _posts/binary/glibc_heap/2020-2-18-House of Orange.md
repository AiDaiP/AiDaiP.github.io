---
clayout: post
title:  "House of Orange"
date:   2020-2-18
desc: ""
keywords: ""
categories: [Binary]
tags: [pwn]
icon: icon-html
---

# House of Orange

> 🍊屋

### 原理

申请的size大于当前top chunk的size时，原来的top chunk会被释放并放入unsorted bin，可以在没有free函数的情况下得到一个unsorted bin

在`_int_malloc`中，fastbin、small bins、unsorted bin、large bins都没有满足要求的，就会使用top chunk，如果top chunk也不够，就需要执行sysmalloc申请更多空间

```c
      /*
         Otherwise, relay to handle system-dependent cases
       */
      else
        {
          void *p = sysmalloc (nb, av);
          if (p != NULL)
            alloc_perturb (p, bytes);
          return p;
        }
```

在sysmalloc中有mmap()和brk()两种拓展方式，以brk()方式拓展后top chunk进unsorted bin

* brk()

  将数据段(.data)的最高地址指针_edata往高地址推

* mmap()

  在进程的虚拟地址空间中（堆和栈中间，称为文件映射区域的地方）找一块空闲的虚拟内存

两种方式分配的都是虚拟内存

```c
  /*
     If have mmap, and the request size meets the mmap threshold, and
     the system supports mmap, and there are few enough currently
     allocated mmapped regions, try to directly map this request
     rather than expanding top.
   */

  if (av == NULL
      || ((unsigned long) (nb) >= (unsigned long) (mp_.mmap_threshold)
	  && (mp_.n_mmaps < mp_.n_mmaps_max)))
    {
      char *mm;           /* return value from mmap call*/

    try_mmap:
```

如果分配的size大于mmap分配阈值`mp_.mmap_threshold`，就会使用mmap()系统调用直接向操作系统申请内存

所以size需要小于`mp_.mmap_threshold`(默认128k)

跑brk前，有对top chunk size的检查

```
  assert ((old_top == initial_top (av) && old_size == 0) ||
          ((unsigned long) (old_size) >= MINSIZE &&
           prev_inuse (old_top) &&
           ((unsigned long) old_end & (pagesize - 1)) == 0));
```

old就是当前的top chunk

top chunk可能没有初始化，size可能为0

如果top chunk已经初始化，size必须大于等于MINSIZE

top chunk的前一个chunk一定是已分配的，所以prev_inuse必须为1

top chunk的结束地址必须页对齐，topchunk地址+size满足0x1000对齐



### 基本操作

1. 存在堆溢出等可以控制top chunk size的漏洞，伪造top chunk size，使top chunk size满足：
   * 大于等于MINSIZE
   * prev_inuse位为1
   * 页对齐
   * 小于下一次申请的size+MINSIZE
2. 申请size大于fake top chunk size且小于`mp_.mmap_threshold`的chunk，原来的top chunk进 unsorted bin





### Hitcon2016-House of Orange

🍊屋祖宗，Hitcon🌶🔟💉💦🐮🍺

