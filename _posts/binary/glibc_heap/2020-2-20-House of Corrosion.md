---
clayout: post
title:  "House of Corrosion"
date:   2020-2-20
desc: ""
keywords: ""
categories: [Binary]
tags: [pwn]
icon: icon-html
---

# House of Corrosion

无内鬼，来个链接

https://github.com/CptGibbon/House-of-Corrosion

先学洋文

```
corrosion	英[kəˈrəʊʒn]  美[kəˈroʊʒn]
n. 腐蚀; 侵蚀;
[例句] Zinc is used to protect other metals from corrosion.
锌被用来保护其他金属不受腐蚀。
[其他] 复数：corrosions-
```

## 源码

glibc2.27

### global_max_fast

```c
/* Maximum size of memory handled in fastbins.  */
static INTERNAL_SIZE_T global_max_fast;

/*
   Set value of max_fast.
   Use impossibly small value if 0.
   Precondition: there are no existing fastbin chunks in the main arena.
   Since do_check_malloc_state () checks this, we call malloc_consolidate ()
   before changing max_fast.  Note other arenas will leak their fast bin
   entries if max_fast is reduced.
 */

#define set_max_fast(s) \
  global_max_fast = (((s) == 0)						      \
                     ? SMALLBIN_WIDTH : ((s + SIZE_SZ) & ~MALLOC_ALIGN_MASK))
```

global_max_fast用来标志fastbin的大小的阈值，小于这个值的堆块会被认为是fastbin

global_max_fast由set_max_fast()初始化，set_max_fast()第一次调用在malloc_init_state()中

get_max_fast()可以获取global_max_fast的值

global_max_fast默认为0x80

```c
/*
   Initialize a malloc_state struct.

   This is called from ptmalloc_init () or from _int_new_arena ()
   when creating a new arena.
 */

static void
malloc_init_state (mstate av)
{
  int i;
  mbinptr bin;

  /* Establish circular links for normal bins */
  for (i = 1; i < NBINS; ++i)
    {
      bin = bin_at (av, i);
      bin->fd = bin->bk = bin;
    }

#if MORECORE_CONTIGUOUS
  if (av != &main_arena)
#endif
  set_noncontiguous (av);
  if (av == &main_arena)
    set_max_fast (DEFAULT_MXFAST);
  atomic_store_relaxed (&av->have_fastchunks, false);

  av->top = initial_top (av);
}
```

### fastbinsY

```c
  /* Fastbins */
  mfastbinptr fastbinsY[NFASTBINS];
  
  typedef struct malloc_chunk *mfastbinptr;
#define fastbin(ar_ptr, idx) ((ar_ptr)->fastbinsY[idx])
```

fastbinsY存放fastbin top，index由size决定

## House of Corrosion

如果能改写global_max_fast，就可以申请超出fastbin范围的chunk

* 写原语

  fastbinsY后任意地址写

  free时根据size得到index，把free chunk写到fastbinsY[idx]，实现在fastbinsY后的目标地址写入一个堆地址

  如果存在uaf，改free chunk的fd再取回来，fd写到fastbinsY[idx]，实现在fastbinsY后的目标地址任意写

  size计算：size = offset*2 + 0x20，offset为目标地址到fastbinsY的偏移

* transplant原语

  向目标地址写目标libc地址（把目标libc地址转移到目标地址）

  1. 改global_max_fast

  2. 根据目标地址(addr1)计算size1，找一个有目标libc地址的地址(addr2)，计算size2

  3. A=malloc(size1)

  4. 使A->fd指向自己

     如果有double free就double free

     如果有uaf，B=malloc(size1)，free(B)，free(A)，uaf改A的fd指向自己

  5. malloc(size1)回到A，改Asize为size2，再freeA，此时A地址进addr2，目标libc地址进A->fd

  6. 把A的size改回去，malloc(size1)，A->fd进addr1也就是目标libc地址进addr1

* get shell（glibc2.27）

  1. 改global_max_fast

     * unsorted bin attack 直接打global_max_fast，需要爆破2字节（glibc2.29不好使）

     * UAF改tcache fd使其指向一个unsorted bin，再取回来，main_arena地址进tcache，再uaf改fd指向global_max_fast，需要爆破2字节（glibc2.29好使）

  2. transplant原语把`__default_morecore`地址转移到stderr的`_IO_buf_end`

  3. 写原语写`_IO_buf_base`，`_IO_buf_base+_IO_buf_end=onegadget`

  4. 写原语把`_flags`写为0，过`_IO_str_overflow`check，`_IO_write_ptr`写为0x7fffffff，过`_IO_write_ptr-_IO_write_base>_IO_buf_base+_IO_buf_end`

  5. 写原语把stdout的`_mode`写为0

  6. 找一个call eax，不能直接找到可以用transplant原语找，写原语写到`stderr+0xe0(tdout的_flags)`

  7. 写原语部分覆写vtable，指向`IO_str_jumps-0x10`

  8. 写原语把`global_max_fast`改回去

  9. 触发stderr，执行call rax

* get shell（glibc2.29）

  👴选择死亡
