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

  7. 调用malloc_consolidate整理fastbins 

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

  

  * ### 尝试从unsorted bin中分配所需内存

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

    循环执行consolidate整理chunk，并检查是否有满足需求的chunk

    循环次数上限为10000

    ```c
    #define MAX_ITERS       10000
              if (++iters >= MAX_ITERS)
                break;
    ```

    检查chunk是否对齐

    ```c
              if (__builtin_expect (chunksize_nomask (victim) <= 2 * SIZE_SZ, 0)
                  || __builtin_expect (chunksize_nomask (victim)
    				   > av->system_mem, 0))
                malloc_printerr ("malloc(): memory corruption");
    ```

    重新申请small chunk

    若大小范围在small chunk的范围内，进入循环后会再一次申请small chunk

    如果unsorted bin中有last remainder，且last remainder可以切割给malloc使用，就切割该last remainder

    ```c
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
    ```

    last remainder被改变后，重新设置标志位

    ```c
    /* Set size/use field */
    #define set_head(p, s)       ((p)->mchunk_size = (s))
    
    /* Set size at footer (only when chunk is not in use) */
    #define set_foot(p, s)       (((mchunkptr) ((char *) (p) + (s)))->mchunk_prev_size = (s))
    ```

    若unsorted bin中有free chunk的大小和请求大小相同，取出这个free chunk

    ```c
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
    ```

    consolidate，将unsorted bin中的free chunk放入small bin或large bin

    大小在small bin范围内，放入small bin

    ```c
              /* place chunk in bin */
    
              if (in_smallbin_range (size))
                {
                  victim_index = smallbin_index (size);
                  bck = bin_at (av, victim_index);
                  fwd = bck->fd;
                }
    ```

    大小在large bin范围内，放入large bin

    ```c
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
    ```

    最后

    ```c
              mark_bin (av, victim_index);
              victim->bk = bck;
              victim->fd = fwd;
              fwd->bk = victim;
              bck->fd = victim;
    ```

    

  * ### 尝试从large bin中分配所需内存

    尝试从large bin中分配所需内存

    ```c
          /*
             If a large request, scan through the chunks of current bin in
             sorted order to find smallest that fits.  Use the skip list for this.
           */
    
          if (!in_smallbin_range (nb))
            {
              bin = bin_at (av, idx);
    
              /* skip scan if empty or largest chunk is too small */
              if ((victim = first (bin)) != bin
    	      && (unsigned long) chunksize_nomask (victim)
    	        >= (unsigned long) (nb))
                {
                  victim = victim->bk_nextsize;
                  while (((unsigned long) (size = chunksize (victim)) <
                          (unsigned long) (nb)))
                    victim = victim->bk_nextsize;
    
                  /* Avoid removing the first entry for a size so that the skip
                     list does not have to be rerouted.  */
                  if (victim != last (bin)
    		  && chunksize_nomask (victim)
    		    == chunksize_nomask (victim->fd))
                    victim = victim->fd;
    
                  remainder_size = size - nb;
                  unlink (av, victim, bck, fwd);
    
                  /* Exhaust */
                  if (remainder_size < MINSIZE)
                    {
                      set_inuse_bit_at_offset (victim, size);
                      if (av != &main_arena)
    		    set_non_main_arena (victim);
                    }
                  /* Split */
                  else
                    {
                      remainder = chunk_at_offset (victim, nb);
                      /* We cannot assume the unsorted list is empty and therefore
                         have to perform a complete insert here.  */
                      bck = unsorted_chunks (av);
                      fwd = bck->fd;
    		  if (__glibc_unlikely (fwd->bk != bck))
    		    malloc_printerr ("malloc(): corrupted unsorted chunks");
                      remainder->bk = bck;
                      remainder->fd = fwd;
                      bck->fd = remainder;
                      fwd->bk = remainder;
                      if (!in_smallbin_range (remainder_size))
                        {
                          remainder->fd_nextsize = NULL;
                          remainder->bk_nextsize = NULL;
                        }
                      set_head (victim, nb | PREV_INUSE |
                                (av != &main_arena ? NON_MAIN_ARENA : 0));
                      set_head (remainder, remainder_size | PREV_INUSE);
                      set_foot (remainder, remainder_size);
                    }
                  check_malloced_chunk (av, victim, nb);
                  void *p = chunk2mem (victim);
                  alloc_perturb (p, bytes);
                  return p;
                }
            }
    ```

    若large bin中有满足要求的free chunk，就切割free chunk给malloc

    ```c
          if (!in_smallbin_range (nb))
            {
              bin = bin_at (av, idx);
    
              /* skip scan if empty or largest chunk is too small */
              if ((victim = first (bin)) != bin
    	      && (unsigned long) chunksize_nomask (victim)
    	        >= (unsigned long) (nb))
                {
    ```

    last remainder

    对于unsorted bin中切割free chunk产生的last remainder，即使大小小于MINISIZE，还会留在unsorted bin

    对于large bin中切割free chunk产生的last remainder，若大小小于MINISIZE，last remainder给malloc使用，最终申请的大小大于请求的大小

    ```c
                  if (remainder_size < MINSIZE)
                    {
                      set_inuse_bit_at_offset (victim, size);
                      if (av != &main_arena)
    		    set_non_main_arena (victim);
                    }
    ```

    然后开始切割

    ```c
                  /* Split */
                  else
                    {
                      remainder = chunk_at_offset (victim, nb);
                      /* We cannot assume the unsorted list is empty and therefore
                         have to perform a complete insert here.  */
                      bck = unsorted_chunks (av);
                      fwd = bck->fd;
    		  if (__glibc_unlikely (fwd->bk != bck))
    		    malloc_printerr ("malloc(): corrupted unsorted chunks");
                      remainder->bk = bck;
                      remainder->fd = fwd;
                      bck->fd = remainder;
                      fwd->bk = remainder;
                      if (!in_smallbin_range (remainder_size))
                        {
                          remainder->fd_nextsize = NULL;
                          remainder->bk_nextsize = NULL;
                        }
                      set_head (victim, nb | PREV_INUSE |
                                (av != &main_arena ? NON_MAIN_ARENA : 0));
                      set_head (remainder, remainder_size | PREV_INUSE);
                      set_foot (remainder, remainder_size);
                    }
                  check_malloced_chunk (av, victim, nb);
                  void *p = chunk2mem (victim);
                  alloc_perturb (p, bytes);
                  return p;
                }
    ```

  * ### 尝试寻找更大chunk

    ```c
          /*
             Search for a chunk by scanning bins, starting with next largest
             bin. This search is strictly by best-fit; i.e., the smallest
             (with ties going to approximately the least recently used) chunk
             that fits is selected.
    
             The bitmap avoids needing to check that most blocks are nonempty.
             The particular case of skipping all bins during warm-up phases
             when no chunks have been returned yet is faster than it might look.
           */
    
          ++idx;
          bin = bin_at (av, idx);
          block = idx2block (idx);
          map = av->binmap[block];
          bit = idx2bit (idx);
    
          for (;; )
            {
              /* Skip rest of block if there are no more set bits in this block.  */
              if (bit > map || bit == 0)
                {
                  do
                    {
                      if (++block >= BINMAPSIZE) /* out of bins */
                        goto use_top;
                    }
                  while ((map = av->binmap[block]) == 0);
    
                  bin = bin_at (av, (block << BINMAPSHIFT));
                  bit = 1;
                }
    
              /* Advance to bin with set bit. There must be one. */
              while ((bit & map) == 0)
                {
                  bin = next_bin (bin);
                  bit <<= 1;
                  assert (bit != 0);
                }
    
              /* Inspect the bin. It is likely to be non-empty */
              victim = last (bin);
    
              /*  If a false alarm (empty bin), clear the bit. */
              if (victim == bin)
                {
                  av->binmap[block] = map &= ~bit; /* Write through */
                  bin = next_bin (bin);
                  bit <<= 1;
                }
    
              else
                {
                  size = chunksize (victim);
    
                  /*  We know the first chunk in this bin is big enough to use. */
                  assert ((unsigned long) (size) >= (unsigned long) (nb));
    
                  remainder_size = size - nb;
    
                  /* unlink */
                  unlink (av, victim, bck, fwd);
    
                  /* Exhaust */
                  if (remainder_size < MINSIZE)
                    {
                      set_inuse_bit_at_offset (victim, size);
                      if (av != &main_arena)
    		    set_non_main_arena (victim);
                    }
    
                  /* Split */
                  else
                    {
                      remainder = chunk_at_offset (victim, nb);
    
                      /* We cannot assume the unsorted list is empty and therefore
                         have to perform a complete insert here.  */
                      bck = unsorted_chunks (av);
                      fwd = bck->fd;
    		  if (__glibc_unlikely (fwd->bk != bck))
    		    malloc_printerr ("malloc(): corrupted unsorted chunks 2");
                      remainder->bk = bck;
                      remainder->fd = fwd;
                      bck->fd = remainder;
                      fwd->bk = remainder;
    
                      /* advertise as last remainder */
                      if (in_smallbin_range (nb))
                        av->last_remainder = remainder;
                      if (!in_smallbin_range (remainder_size))
                        {
                          remainder->fd_nextsize = NULL;
                          remainder->bk_nextsize = NULL;
                        }
                      set_head (victim, nb | PREV_INUSE |
                                (av != &main_arena ? NON_MAIN_ARENA : 0));
                      set_head (remainder, remainder_size | PREV_INUSE);
                      set_foot (remainder, remainder_size);
                    }
                  check_malloced_chunk (av, victim, nb);
                  void *p = chunk2mem (victim);
                  alloc_perturb (p, bytes);
                  return p;
                }
            }
    ```

    当所有的bins链都没有合适的chunk时，会尝试寻找一个更大的chunk，而不是直接使用top chunk

    切割free chunk后产生last remainder，若大小小于MINSIZE，last remainder给malloc使用

    ```c
                  /* Exhaust */
                  if (remainder_size < MINSIZE)
                    {
                      set_inuse_bit_at_offset (victim, size);
                      if (av != &main_arena)
    		    set_non_main_arena (victim);
    ```

    

  * ### 使用top chunk

    ```c
        use_top:
          /*
             If large enough, split off the chunk bordering the end of memory
             (held in av->top). Note that this is in accord with the best-fit
             search rule.  In effect, av->top is treated as larger (and thus
             less well fitting) than any other available chunk since it can
             be extended to be as large as necessary (up to system
             limitations).
    
             We require that av->top always exists (i.e., has size >=
             MINSIZE) after initialization, so if it would otherwise be
             exhausted by current request, it is replenished. (The main
             reason for ensuring it exists is that we may need MINSIZE space
             to put in fenceposts in sysmalloc.)
           */
    
          victim = av->top;
          size = chunksize (victim);
    
          if ((unsigned long) (size) >= (unsigned long) (nb + MINSIZE))
            {
              remainder_size = size - nb;
              remainder = chunk_at_offset (victim, nb);
              av->top = remainder;
              set_head (victim, nb | PREV_INUSE |
                        (av != &main_arena ? NON_MAIN_ARENA : 0));
              set_head (remainder, remainder_size | PREV_INUSE);
    
              check_malloced_chunk (av, victim, nb);
              void *p = chunk2mem (victim);
              alloc_perturb (p, bytes);
              return p;
            }
    
          /* When we are using atomic ops to free fast chunks we can get
             here for all block sizes.  */
          else if (atomic_load_relaxed (&av->have_fastchunks))
            {
              malloc_consolidate (av);
              /* restore original bin index */
              if (in_smallbin_range (nb))
                idx = smallbin_index (nb);
              else
                idx = largebin_index (nb);
            }
    
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
        }
    }
    ```

    若top chunk不够用，先调用malloc_consolidate整理fastbins 

    若还不够用，执行sysmalloc申请内存

    ```c
          /* When we are using atomic ops to free fast chunks we can get
             here for all block sizes.  */
          else if (atomic_load_relaxed (&av->have_fastchunks))
            {
              malloc_consolidate (av);
              /* restore original bin index */
              if (in_smallbin_range (nb))
                idx = smallbin_index (nb);
              else
                idx = largebin_index (nb);
            }
    
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

    
