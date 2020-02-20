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

先学洋文

```
corrosion	英[kəˈrəʊʒn]  美[kəˈroʊʒn]
n. 腐蚀; 侵蚀;
[例句] Zinc is used to protect other metals from corrosion.
锌被用来保护其他金属不受腐蚀。
[其他] 复数：corrosions-
```

## 原理

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

### House of Corrosion

如果能改写global_max_fast，就可以申请超出fastbin范围的chunk

free时根据size得到index，把free chunk写到fastbinsY[idx]，实现在fastbinsY后的目标地址写入一个堆地址

如果存在uaf，改free chunk的fd再取回来，fd写到fastbinsY[idx]，实现在fastbinsY后的目标地址任意写

size计算：size = offset*2 + 0x20，offset为目标地址到fastbinsY的偏移

* Transplant

  向目标地址写目标libc地址，并且把chunk申请到libc

  1. 改global_max_fast

  2. 根据目标地址(addr1)计算size1，找一个有目标libc地址的地址(addr2)，计算size2

  3. A=malloc(size1)

  4. 使A->fd指向自己

     如果有double free就double free

     如果有uaf，B=malloc(size1)，free(B)，free(A)，uaf改A的fd指向自己

  5. malloc(size1)回到A，改Asize为size2，再freeA，此时A地址进addr2，目标libc地址进A->fd

  6. 把A的size改回去，malloc(size1)，A->fd进addr1也就是目标libc地址进addr1

  7. malloc(size1)到目标libc地址

## 基本操作

### glibc2.27

1. 改global_max_fast
2. 

