---
clayout: post
title:  "tcache_perthread_struct"
date:   2019-11-5
desc: ""
keywords: ""
categories: [Binary]
tags: [pwn]
icon: icon-html
---

# tcache_perthread_struct

## 源码

glibc-2.27

```c
/* We overlay this structure on the user-data portion of a chunk when
   the chunk is stored in the per-thread cache.  */
typedef struct tcache_entry
{
  struct tcache_entry *next;
} tcache_entry;

/* There is one of these for each thread, which contains the
   per-thread cache (hence "tcache_perthread_struct").  Keeping
   overall size low is mildly important.  Note that COUNTS and ENTRIES
   are redundant (we could have just counted the linked list each
   time), this is for performance reasons.  */
typedef struct tcache_perthread_struct
{
  char counts[TCACHE_MAX_BINS];
  tcache_entry *entries[TCACHE_MAX_BINS];
} tcache_perthread_struct;
```

TCACHE_MAX_BINS

```
# define TCACHE_MAX_BINS		64
```

tcache_entry中next指针指向链表中下一个chunk的fd

有64个entries，entries记录每条链第一个chunk的fd

counts记录每条链上的chunk数量，每条链上最多有7个chunk

tcache_perthread_struct大小为size_chunkhead + size_counts + size_entries = 16 + 64 + 64*8 = 592 = 0x250

```c
pwndbg> x/10gx 0x8403000
0x8403000:      0x0000000000000000      0x0000000000000251
0x8403010:      0x0100000002000000      0x0100000000000000
0x8403020:      0x0000000000000000      0x0000000000000000
0x8403030:      0x0000000000000000      0x0000000000000000
0x8403040:      0x0000000000000000      0x0000000000000000
pwndbg> bin
tcachebins
0x50 [  2]: 0x8403260 ◂— 0x8403260
0x90 [  1]: 0x84032b0 ◂— 0x0
0x110 [  1]: 0x8403340 ◂— 0x0
```

tcache关于index的宏定义

```c
/* Only used to pre-fill the tunables.  */
# define tidx2usize(idx)	(((size_t) idx) * MALLOC_ALIGNMENT + MINSIZE - SIZE_SZ)

/* When "x" is from chunksize().  */
# define csize2tidx(x) (((x) - MINSIZE + MALLOC_ALIGNMENT - 1) / MALLOC_ALIGNMENT)
/* When "x" is a user-provided size.  */
# define usize2tidx(x) csize2tidx (request2size (x))
```



## 利用

`tcache_perthread_struct`是一块先于用户申请分配的堆空间，在堆起始处。=

若能泄露一个堆地址，就可以获得堆基址和`tcache_perthread_struct`的地址

若能控制`tcache_perthread_struct`，可以把counts改为7，下一次free对应size的chunk就会进入unsorted bin

