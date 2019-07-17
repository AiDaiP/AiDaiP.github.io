---
layout: post
title:  "Off-By-One in the heap"
date:   2019-7-17
desc: ""
keywords: ""
categories: [Binary]
tags: [pwn,heap]
icon: icon-html
---

# Off-By-One in the heap

![11](https://raw.githubusercontent.com/AiDaiP/images/master/pwn/11.png)

* ### 原理

  边界验证不严或字符串操作不合适导致出现单字节溢出

  1. 溢出字节为可控制任意字节：通过修改大小造成块结构之间出现重叠，从而泄露其他块数据，或是覆盖其他块数据 

  2. 溢出字节为 NULL 字节

     在 size 为 0x100 的时候，溢出 NULL 字节`prev_in_use`被清，这样前块会被认为是 free 块，可以触发unlink

     此时 `prev_size`  域启用，可以伪造 `prev_size`  ，使块之间发生重叠

* ### 利用方式

  * ### chunk overlapping

    使目标堆块被重新分配到某个可控的新的堆块中，实现对目标堆块任意读写。 

    * off-by-one overwrite allocated 
    * off-by-one overwrite freed 
    * off-by-one null byte 

  * ### unlink 

    触发unlink

    * off-by-one small bin 

    * off-by-one large bin 

      

* ### off-by-one overwrite allocated

  ```c
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                         A(off-by-one)                         |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                         B(allocated)                          |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                         C(target,allocated)                   |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  ```

  1. off-by-one 改B的size，使其包含C，`prev_in_use` 位保持为1否则会触发unlink导致崩溃
  2. free B
  3. malloc一个大小为B+C的块
  4. 通过新块对C任意读写

  

* ### off-by-one overwrite freed 

  ```
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                         A(off-by-one)                         |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                         B(freed)                              |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                         C(target,allocated)                   |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  ```

  1. off-by-one 改B的size，使其包含C，`prev_in_use`  位保持为1

  2. malloc一个大小为B+C的块

  3. 通过新块对C任意读写

     

* ### off-by-one null byte 

  ```
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                         A(allocated)                          |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                         B(allocated)                          |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                         C(allocated)                          |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  ```

  A发生off-by-one，溢出一个`'\x00'` 覆盖B的`prev_in_use` 位
