---
layout: post
title:  "_int_malloc"
date:   2019-9-12
desc: ""
keywords: ""
categories: [Binary]
tags: [pwn]
icon: icon-html
---

# \_int\_malloc

* ### 执行流程

  1. 若请求大小在fast bin范围内，在fast bin中查找是否有chunk可以使用

  2. 若请求大小是否在small bin范围内，在small bin中查找是否有chunk可以使用

  3. 调用malloc_consolidate对fastbins进行整理，然后在unsorted bin中查找是否有chunk可以使用

  4. 在large bin中查找是否有chunk可以使用

  5. 在更大的bin链中查找是否有chunk可以使用

  6. 切割top chunk

  7. 再次调用malloc_consolidate整理fastbins 

  8. 调用sysmalloc申请内存

     

* ### 定义相关变量

  ```c
  static void *
  _int_malloc (mstate av, size_t bytes)
  {
    INTERNAL_SIZE_T nb;               /* normalized request size */
    unsigned int idx;                 /* associated bin index */
    mbinptr bin;                      /* associated bin */
  
    mchunkptr victim;                 /* inspected/selected chunk */
    INTERNAL_SIZE_T size;             /* its size */
    int victim_index;                 /* its bin index */
  
    mchunkptr remainder;              /* remainder from a split */
    unsigned long remainder_size;     /* its size */
  
    unsigned int block;               /* bit map traverser */
    unsigned int bit;                 /* bit map traverser */
    unsigned int map;                 /* current word of binmap */
  
    mchunkptr fwd;                    /* misc temp for linking */
    mchunkptr bck;                    /* misc temp for linking */
  
  #if USE_TCACHE
    size_t tcache_unsorted_count;	    /* count of unsorted chunks processed */
  #endif
  ```

* ### 将请求的大小转换为chunk的大小

  ```c
    /*
       Convert request size to internal form by adding SIZE_SZ bytes
       overhead plus possibly more to obtain necessary alignment and/or
       to obtain a size of at least MINSIZE, the smallest allocatable
       size. Also, checked_request2size traps (returning 0) request sizes
       that are so large that they wrap around zero when padded and
       aligned.
     */
  
    checked_request2size (bytes, nb);
  ```

  

* ### 检查arena

  ```c
    /* There are no usable arenas.  Fall back to sysmalloc to get a chunk from
       mmap.  */
    if (__glibc_unlikely (av == NULL))
      {
        void *p = sysmalloc (nb, av);
        if (p != NULL)
  	alloc_perturb (p, bytes);
        return p;
      }
  
  ```

  若arena为空，调用sysmalloc申请内存

  分配成功后，调用alloc_perturb对分配的内存进行初始化 

  ```c
  static void
  alloc_perturb (char *p, size_t n)
  {
    if (__glibc_unlikely (perturb_byte))
      memset (p, perturb_byte ^ 0xff, n);
  }
  ```

  

* ### 检查fast bin

  ```c
    /*
       If the size qualifies as a fastbin, first check corresponding bin.
       This code is safe to execute even if av is not yet initialized, so we
       can try it without checking, which saves some time on this fast path.
     */
  
  #define REMOVE_FB(fb, victim, pp)			\
    do							\
      {							\
        victim = pp;					\
        if (victim == NULL)				\
  	break;						\
      }							\
    while ((pp = catomic_compare_and_exchange_val_acq (fb, victim->fd, victim)) \
  	 != victim);					\
  
    if ((unsigned long) (nb) <= (unsigned long) (get_max_fast ()))
      {
        idx = fastbin_index (nb);
        mfastbinptr *fb = &fastbin (av, idx);
        mchunkptr pp;
        victim = *fb;
  
        if (victim != NULL)
  	{
  	  if (SINGLE_THREAD_P)
  	    *fb = victim->fd;
  	  else
  	    REMOVE_FB (fb, pp, victim);
  	  if (__glibc_likely (victim != NULL))
  	    {
  	      size_t victim_idx = fastbin_index (chunksize (victim));
  	      if (__builtin_expect (victim_idx != idx, 0))
  		malloc_printerr ("malloc(): memory corruption (fast)");
  	      check_remalloced_chunk (av, victim, nb);
  #if USE_TCACHE
  '''
  #endif
  	      void *p = chunk2mem (victim);
  	      alloc_perturb (p, bytes);
  	      return p;
  	    }
  	}
      }
  ```

  先判断请求大小是否在fast bin范围内，若在范围内，就在fast bin查找是否有可用的free chunk，根据大小得到索引，取出chunk

  ```c
    if ((unsigned long) (nb) <= (unsigned long) (get_max_fast ()))
      {
        idx = fastbin_index (nb);
        mfastbinptr *fb = &fastbin (av, idx);
        mchunkptr pp;
        victim = *fb;
        if (victim != NULL)
  	{
  	  if (SINGLE_THREAD_P)
  	    *fb = victim->fd;
  	  else
  	    REMOVE_FB (fb, pp, victim);
  ```

  取出后进行检测

  ```c
  	  if (__glibc_likely (victim != NULL))
  	    {
  	      size_t victim_idx = fastbin_index (chunksize (victim));
  	      if (__builtin_expect (victim_idx != idx, 0))
  		malloc_printerr ("malloc(): memory corruption (fast)");
  	      check_remalloced_chunk (av, victim, nb);
  ```

  返回指针

  如果设置了perturb_type，则将获取到的chunk初始化为`perturb_type ^ 0xff`

  ```c
  	      void *p = chunk2mem (victim);
  	      alloc_perturb (p, bytes);
  	      return p;
  ```

  chunk2mem

  ```c
  /* conversion from malloc headers to user pointers, and back */
  
  #define chunk2mem(p)   ((void*)((char*)(p) + 2*SIZE_SZ))
  #define mem2chunk(mem) ((mchunkptr)((char*)(mem) - 2*SIZE_SZ))
  ```

  

* ### 检查small bin

  ```c
    /*
       If a small request, check regular bin.  Since these "smallbins"
       hold one size each, no searching within bins is necessary.
       (For a large request, we need to wait until unsorted chunks are
       processed to find best fit. But for small ones, fits are exact
       anyway, so we can check now, which is faster.)
     */
  
    if (in_smallbin_range (nb))
      {
        idx = smallbin_index (nb);
        bin = bin_at (av, idx);
  
        if ((victim = last (bin)) != bin)
          {
            bck = victim->bk;
  	  if (__glibc_unlikely (bck->fd != victim))
  	    malloc_printerr ("malloc(): smallbin double linked list corrupted");
            set_inuse_bit_at_offset (victim, nb);
            bin->bk = bck;
            bck->fd = bin;
  
            if (av != &main_arena)
  	    set_non_main_arena (victim);
            check_malloced_chunk (av, victim, nb);
  #if USE_TCACHE
  '''
  #endif
            void *p = chunk2mem (victim);
            alloc_perturb (p, bytes);
            return p;
          }
      }
  ```

  判断请求大小是否在fast bin范围内，若在范围内就根据大小取出bin链，然后判断bin链是否为空

  ```c
    if (in_smallbin_range (nb))
      {
        idx = smallbin_index (nb);
        bin = bin_at (av, idx);
  
        if ((victim = last (bin)) != bin)
          {
  ```

  small bin双向链表，先进先出，last返回最先free的chunk

  ```c
  /* Reminders about list directionality within bins */
  #define first(b)     ((b)->fd)
  #define last(b)      ((b)->bk)
  ```

  取出free chunk前进行检查

  ```c
            bck = victim->bk;
  	  if (__glibc_unlikely (bck->fd != victim))
  	    malloc_printerr ("malloc(): smallbin double linked list corrupted");
  ```

  chunk被malloc使用后，后面两个chunk的prev_inuse位要设置为1

  ```c
  set_inuse_bit_at_offset (victim, nb);
  ```

  返回指针

  如果设置了perturb_type，则将获取到的chunk初始化为`perturb_type ^ 0xff`

  ```c
            void *p = chunk2mem (victim);
            alloc_perturb (p, bytes);
            return p;`
  ```

  

* ### malloc_consolidate

  ```c
    /*
       If this is a large request, consolidate fastbins before continuing.
       While it might look excessive to kill all fastbins before
       even seeing if there is space available, this avoids
       fragmentation problems normally associated with fastbins.
       Also, in practice, programs tend to have runs of either small or
       large requests, but less often mixtures, so consolidation is not
       invoked all that often in most programs. And the programs that
       it is called frequently in otherwise tend to fragment.
     */
  
    else
      {
        idx = largebin_index (nb);
        if (atomic_load_relaxed (&av->have_fastchunks))
          malloc_consolidate (av);
      }
  ```

  如果申请的chunk大小超出small bin的范围，执行malloc_consolidate

  fast bin中有free chunk时才会触发malloc_consolidate

  将fast bin的chunk整理到unsorted bin上

  

* ### for循环

  1. 尝试从unsorted bin中分配所需内存，并进行consolidate 
  2. 尝试从large bin中分配所需内存
  3. 尝试寻找更大chunk
  4. 尝试从top chunk分配所需内存

  

  * ### unsortedbin的while循环

    ```c
      /*
         Process recently freed or remaindered chunks, taking one only if
         it is exact fit, or, if this a small request, the chunk is remainder from
         the most recent non-exact fit.  Place other traversed chunks in
         bins.  Note that this step is the only place in any routine where
         chunks are placed in bins.
    
         The outer loop here is needed because we might not realize until
         near the end of malloc that we should have consolidated, so must
         do so and retry. This happens at most once, and only when we would
         otherwise need to expand memory to service a "small" request.
       */
    
    #if USE_TCACHE
    '''
    #endif
    
      for (;; )
        {
          int iters = 0;
          while ((victim = unsorted_chunks (av)->bk) != unsorted_chunks (av))
            {
              bck = victim->bk;
              if (__builtin_expect (chunksize_nomask (victim) <= 2 * SIZE_SZ, 0)
                  || __builtin_expect (chunksize_nomask (victim)
    				   > av->system_mem, 0))
                malloc_printerr ("malloc(): memory corruption");
              size = chunksize (victim);
    
              /*
                 If a small request, try to use last remainder if it is the
                 only chunk in unsorted bin.  This helps promote locality for
                 runs of consecutive small requests. This is the only
                 exception to best-fit, and applies only when there is
                 no exact fit for a small chunk.
               */
    
              if (in_smallbin_range (nb) &&
                  bck == unsorted_chunks (av) &&
                  victim == av->last_remainder &&
                  (unsigned long) (size) > (unsigned long) (nb + MINSIZE))
                {
                  /* split and reattach remainder */
                  remainder_size = size - nb;
                  remainder = chunk_at_offset (victim, nb);
                  unsorted_chunks (av)->bk = unsorted_chunks (av)->fd = remainder;
                  av->last_remainder = remainder;
                  remainder->bk = remainder->fd = unsorted_chunks (av);
                  if (!in_smallbin_range (remainder_size))
                    {
                      remainder->fd_nextsize = NULL;
                      remainder->bk_nextsize = NULL;
                    }
    
                  set_head (victim, nb | PREV_INUSE |
                            (av != &main_arena ? NON_MAIN_ARENA : 0));
                  set_head (remainder, remainder_size | PREV_INUSE);
                  set_foot (remainder, remainder_size);
    
                  check_malloced_chunk (av, victim, nb);
                  void *p = chunk2mem (victim);
                  alloc_perturb (p, bytes);
                  return p;
                }
    
              /* remove from unsorted list */
              unsorted_chunks (av)->bk = bck;
              bck->fd = unsorted_chunks (av);
    
              /* Take now instead of binning if exact fit */
    
              if (size == nb)
                {
                  set_inuse_bit_at_offset (victim, size);
                  if (av != &main_arena)
    		set_non_main_arena (victim);
    #if USE_TCACHE
    '''
    #endif
                  check_malloced_chunk (av, victim, nb);
                  void *p = chunk2mem (victim);
                  alloc_perturb (p, bytes);
                  return p;
    #if USE_TCACHE
    '''
    #endif
                }
    
              /* place chunk in bin */
    
              if (in_smallbin_range (size))
                {
                  victim_index = smallbin_index (size);
                  bck = bin_at (av, victim_index);
                  fwd = bck->fd;
                }
              else
                {
                  victim_index = largebin_index (size);
                  bck = bin_at (av, victim_index);
                  fwd = bck->fd;
    
                  /* maintain large bins in sorted order */
                  if (fwd != bck)
                    {
                      /* Or with inuse bit to speed comparisons */
                      size |= PREV_INUSE;
                      /* if smaller than smallest, bypass loop below */
                      assert (chunk_main_arena (bck->bk));
                      if ((unsigned long) (size)
    		      < (unsigned long) chunksize_nomask (bck->bk))
                        {
                          fwd = bck;
                          bck = bck->bk;
    
                          victim->fd_nextsize = fwd->fd;
                          victim->bk_nextsize = fwd->fd->bk_nextsize;
                          fwd->fd->bk_nextsize = victim->bk_nextsize->fd_nextsize = victim;
                        }
                      else
                        {
                          assert (chunk_main_arena (fwd));
                          while ((unsigned long) size < chunksize_nomask (fwd))
                            {
                              fwd = fwd->fd_nextsize;
    			  assert (chunk_main_arena (fwd));
                            }
    
                          if ((unsigned long) size
    			  == (unsigned long) chunksize_nomask (fwd))
                            /* Always insert in the second position.  */
                            fwd = fwd->fd;
                          else
                            {
                              victim->fd_nextsize = fwd;
                              victim->bk_nextsize = fwd->bk_nextsize;
                              fwd->bk_nextsize = victim;
                              victim->bk_nextsize->fd_nextsize = victim;
                            }
                          bck = fwd->bk;
                        }
                    }
                  else
                    victim->fd_nextsize = victim->bk_nextsize = victim;
                }
    
              mark_bin (av, victim_index);
              victim->bk = bck;
              victim->fd = fwd;
              fwd->bk = victim;
              bck->fd = victim;
    
    #if USE_TCACHE
    '''
    #endif
    
    #define MAX_ITERS       10000
              if (++iters >= MAX_ITERS)
                break;
            }
    
    ```

    

  

  

     
