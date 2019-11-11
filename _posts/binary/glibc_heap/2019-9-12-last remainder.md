---
layout: post
title:  "last remainder"
date:   2019-9-12
desc: ""
keywords: ""
categories: [Binary]
tags: [pwn]
icon: icon-html
---

# last remainder

* ### last remainder

  ```c
  struct malloc_state
  {
    /* Serialize access.  */
    __libc_lock_define (, mutex);
  
    /* Flags (formerly in max_fast).  */
    int flags;
  
    /* Set if the fastbin chunks contain recently inserted free blocks.  */
    /* Note this is a bool but not all targets support atomics on booleans.  */
    int have_fastchunks;
  
    /* Fastbins */
    mfastbinptr fastbinsY[NFASTBINS];
  
    /* Base of the topmost chunk -- not otherwise kept in a bin */
    mchunkptr top;
  
    /* The remainder from the most recent split of a small request */
    mchunkptr last_remainder;
  
    /* Normal bins packed as described above */
    mchunkptr bins[NBINS * 2 - 2];
  
    /* Bitmap of bins */
    unsigned int binmap[BINMAPSIZE];
  
    /* Linked list */
    struct malloc_state *next;
  
    /* Linked list for free arenas.  Access to this field is serialized
       by free_list_lock in arena.c.  */
    struct malloc_state *next_free;
  
    /* Number of threads attached to this arena.  0 if the arena is on
       the free list.  Access to this field is serialized by
       free_list_lock in arena.c.  */
    INTERNAL_SIZE_T attached_threads;
  
    /* Memory allocated from the system in this arena.  */
    INTERNAL_SIZE_T system_mem;
    INTERNAL_SIZE_T max_system_mem;
  };
  ```

  bins链(fast bin除外)存在free chunk时，若malloc的请求大小小于free chunk，就会切割这个free chunk，剩余的chunk就是last remainder，last remainder会被放入unsorted bin中 

  产生last remainder后，malloc_state结构体中的last_remainder成员指针会被初始化，指向这个last_remainder

  

* ### last remainder的产生

  * ### 切割unsorted bin

    当unsorted bin有free chunk可以给malloc切割使用时:

    1. 将free chunk放置到对应大小的bins链
    2. 切割free chunk，产生last remainder
    3. last remainder放入unsorted bin

  * ### 切割small bin、large bin

    1. 切割free chunk，产生last remainder
    2. last remainder放入unsorted bin

  