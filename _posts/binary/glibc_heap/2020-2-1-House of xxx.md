---
clayout: post
title:  "House of xxx"
date:   2020-2-1
desc: ""
keywords: ""
categories: [Binary]
tags: [pwn]
icon: icon-html
---

# House of xxx

👴之前没记录过House of xxx，现在随便写写

### Heap Overflow

没验size，👴能填到后面的chunk

### Use After Free

free之后还能用

### Double Free

也算是个uaf，多次free，多次进链

### Unlink

fd=ptr - 0x18，bk=ptr - 0x10，unlink时ptr的指针变为ptr - 0x18

###Unlink overlap

free1 used2 used3

通过used2伪造used3的prev_size为free1和used2size和，prev_inuse为0，free3，unlink，再取出来，得到两个used2

### Fastbin Attack

伪造chunk，改fd打过去

### Tcache Attack

没验证，改fd直接打

### Unsortedbin Attack

👴能控制bk，就能把unsorted_chunks (av)写到bk->fd(bk指向的地址+0x10)，unsorted_chunks (av)是个libc地址

### House of Force

* 原理

  把top chunk的size改大，malloc时从top chunk切出来，然后main_arena中的top chunk指针根据size改变，如果malloc的size👴能控制，👴就能让top chunk去👴想让它去的地方，下一次malloc就到了👴想控制的地方

* 条件

  1. 👴能控制top chunk的size

     直接改成-1，验size的时候-1转换成无符号数，所以👴申请的size必能过验证

  2. 👴能自由控制分配的size

### house of einherjar

* 原理

  把最后一个chunk的pre_inuse置零，在👴想控制的地方伪造一个chunk，根据伪造的chunk和最后一个chunk的距离设置最后一个chunk的pre_size，free最后一个chunk，free认为上一个chunk已经free，根据pre_size找上一个chunk，然后伪造的chunk，最后一个chunk，top chunk unlink，top chunk就到了👴想让他去的地方

* 条件

  1. 👴能控制最后一个chunk的pre_size和pre_inuse

     两个物理相邻的 chunk 会共享 prev_size字段

  2. 最后一个chunk的size必须是0x100的倍数，否则过不了check

  3. 👴能伪造chunk

### house of spirit

* 原理

  在👴想控制的地方伪造一个fake fastbin，把它free，它就进了fastbin链，下次malloc就到了这个fake fastbin

  

* 条件

  1. 👴能伪造fake fastbin，目标区域的前后都是可控的，前面伪造size，后面伪造下一个chunk的size
  2. 👴能控制free的指针



### House of Roman

* 原理

  fastbin attack + unsortedbin attack

  通过unsortedbin搞出来libc地址(main_arena+88)，写到fastbin的fd，然后👴改一字节指向malloc_hook-0x23，因为这有个0x7f👴fastbin attack能打过去，👴就可以控制malloc_hook了

* 条件

  1. 👴能搞出来unsortbin
  2. 👴能控制fastbin的fd