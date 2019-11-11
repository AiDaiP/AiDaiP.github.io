---
layout: post
title:  "malloc_printerr"
date:   2019-9-24
desc: ""
keywords: ""
categories: [Binary]
tags: [pwn]
icon: icon-html
---

# malloc_printerr

* ### corrupted size vs. prev_size

  ```c
      if (__builtin_expect (chunksize(P) != prev_size (next_chunk(P)), 0))      \
        malloc_printerr ("corrupted size vs. prev_size");           \
  ```

  下一个chunk的prev_size和当前chunk的size不同

* ### corrupted double-linked list

  ```c
      if (__builtin_expect (FD->bk != P || BK->fd != P, 0))         \
        malloc_printerr ("corrupted double-linked list");           \
  ```

  前一个chunk的bk不指向当前chunk或后一个chunk的fd不指向当前chunk

* ### corrupted double-linked list (not small)

  ```c
        if (__builtin_expect (P->fd_nextsize->bk_nextsize != P, 0)        \
      || __builtin_expect (P->bk_nextsize->fd_nextsize != P, 0))    \
          malloc_printerr ("corrupted double-linked list (not small)");   \
  ```

* ### break adjusted to free malloc space

  ```c
            if (brk == old_end && snd_brk == (char *) (MORECORE_FAILURE))
              set_head (old_top, (size + old_size) | PREV_INUSE);
  
            else if (contiguous (av) && old_size && brk < old_end)
        /* Oops!  Someone else killed our space..  Can't touch anything.  */
        malloc_printerr ("break adjusted to free malloc space");
  ```

  

* ### munmap_chunk(): invalid pointer

  ```c
    /* Unfortunately we have to do the compilers job by hand here.  Normally
       we would test BLOCK and TOTAL-SIZE separately for compliance with the
       page size.  But gcc does not recognize the optimization possibility
       (in the moment at least) so we combine the two values into one before
       the bit test.  */
    if (__builtin_expect (((block | total_size) & (GLRO (dl_pagesize) - 1)) != 0, 0))
      malloc_printerr ("munmap_chunk(): invalid pointer");
  ```

  地址不合法

* ### realloc(): invalid pointer

  ```
    if ((__builtin_expect ((uintptr_t) oldp > (uintptr_t) -oldsize, 0)
         || __builtin_expect (misaligned_chunk (oldp), 0))
        && !DUMPED_MAIN_ARENA_CHUNK (oldp))
        malloc_printerr ("realloc(): invalid pointer");
  ```

  地址不合法

* ### malloc(): memory corruption (fast)

  ```c
          size_t victim_idx = fastbin_index (chunksize (victim));
          if (__builtin_expect (victim_idx != idx, 0))
      malloc_printerr ("malloc(): memory corruption (fast)");
  ```

  size在fastbin范围时，fastbin 取出的chunk的size不属于该fastbin

* ### malloc(): smallbin double linked list corrupted

  ```c
      if (__glibc_unlikely (bck->fd != victim))
        malloc_printerr ("malloc(): smallbin double linked list corrupted");
  ```

  size在smallbin范围时，smallbin的最后一个被取出的时候发现不为double linked list，即不满足

  `victim -> bk -> fd == victim`

* ### malloc(): memory corruption

  ```c
            if (__builtin_expect (chunksize_nomask (victim) <= 2 * SIZE_SZ, 0)
                || __builtin_expect (chunksize_nomask (victim)
             > av->system_mem, 0))
              malloc_printerr ("malloc(): memory corruption");
  ```

  unsorted bin中有chunk，取unsorted bin 最后一个chunk时，size不满足 大于 2 * SIZE_SZ，且小于system_mem

* ### malloc(): corrupted unsorted chunks

  ```c
        if (__glibc_unlikely (fwd->bk != bck))
          malloc_printerr ("malloc(): corrupted unsorted chunks");
  ```

  size在largebin范围时，切割chunk，剩下的放入last remainder然后放入unsorted bin，fd不等于unsorted bin的位置

* ### malloc(): corrupted unsorted chunks 2

  ```c
        if (__glibc_unlikely (fwd->bk != bck))
          malloc_printerr ("malloc(): corrupted unsorted chunks 2");
  ```

  发生在for循环中

  

* ### free(): invalid pointer

  ```c
    if (__builtin_expect ((uintptr_t) p > (uintptr_t) -size, 0)
        || __builtin_expect (misaligned_chunk (p), 0))
      malloc_printerr ("free(): invalid pointer");
  ```

  free的地址不合法

* ### free(): invalid size

  ```c
    if (__glibc_unlikely (size < MINSIZE || !aligned_OK (size)))
      malloc_printerr ("free(): invalid size");
  ```

  free的chunk size不合法

* ### free(): invalid next size (fast)

  ```c
    bool fail = true;
    /* We might not have a lock at this point and concurrent modifications
       of system_mem might result in a false positive.  Redo the test after
       getting the lock.  */
    if (!have_lock)
      {
        __libc_lock_lock (av->mutex);
        fail = (chunksize_nomask (chunk_at_offset (p, size)) <= 2 * SIZE_SZ
          || chunksize (chunk_at_offset (p, size)) >= av->system_mem);
        __libc_lock_unlock (av->mutex);
      }
  
    if (fail)
      malloc_printerr ("free(): invalid next size (fast)");
        }
  ```

  size在fastbin范围时，下一个chunk的size不合法

* ### double free or corruption (fasttop)

  ```c
      /* Atomically link P to its fastbin: P->FD = *FB; *FB = P;  */
      mchunkptr old = *fb, old2;
  
      if (SINGLE_THREAD_P)
        {
    /* Check that the top of the bin is not the record we are going to
       add (i.e., double free).  */
    if (__builtin_expect (old == p, 0))
      malloc_printerr ("double free or corruption (fasttop)");
    p->fd = old;
    *fb = p;
        }
      else
        do
    {
      /* Check that the top of the bin is not the record we are going to
         add (i.e., double free).  */
      if (__builtin_expect (old == p, 0))
        malloc_printerr ("double free or corruption (fasttop)");
      p->fd = old2 = old;
    }
        while ((old = catomic_compare_and_exchange_val_rel (fb, p, old2))
         != old2);
  
  ```

  fastbin的第一个chunk是被free的chunk（old == p）

* ### invalid fastbin entry (free)

  ```c
      /* Check that size of fastbin chunk at the top is the same as
         size of the chunk that we are adding.  We can dereference OLD
         only if we have the lock, otherwise it might have already been
         allocated again.  */
      if (have_lock && old != NULL
    && __builtin_expect (fastbin_index (chunksize (old)) != idx, 0))
        malloc_printerr ("invalid fastbin entry (free)");
    }
  ```

  fastbin头部的chunk的size与被free的chunk的size不同

* ### double free or corruption (top)

  ```c
      /* Lightweight tests: check whether the block is already the
         top block.  */
      if (__glibc_unlikely (p == av->top))
        malloc_printerr ("double free or corruption (top)");
  ```

  size在small bin或large bin范围时，被free的chunk与topchunk位置相同（p == av->top）

* ### double free or corruption (out)

  ```c
      /* Or whether the next chunk is beyond the boundaries of the arena.  */
      if (__builtin_expect (contiguous (av)
          && (char *) nextchunk
          >= ((char *) av->top + chunksize(av->top)), 0))
    malloc_printerr ("double free or corruption (out)");
  ```

  下一个chunk的位置超出heap边界

* ### double free or corruption (!prev)

  ```c
      /* Or whether the block is actually not marked used.  */
      if (__glibc_unlikely (!prev_inuse(nextchunk)))
        malloc_printerr ("double free or corruption (!prev)");
  ```

  下一个chunk的prev_inuse位为0

* ### free(): invalid next size (normal)

  ```c
      nextsize = chunksize(nextchunk);
      if (__builtin_expect (chunksize_nomask (nextchunk) <= 2 * SIZE_SZ, 0)
    || __builtin_expect (nextsize >= av->system_mem, 0))
        malloc_printerr ("free(): invalid next size (normal)");
  ```

  size在small bin或large bin范围时，下一个chunk的size不合法

* ### free(): corrupted unsorted chunks

  ```c
        /*
    Place the chunk in unsorted chunk list. Chunks are
    not placed into regular bins until after they have
    been given one chance to be used in malloc.
        */
  
        bck = unsorted_chunks(av);
        fwd = bck->fd;
        if (__glibc_unlikely (fwd->bk != bck))
    malloc_printerr ("free(): corrupted unsorted chunks");
  ```

  small bin 或 large bin在unlink后，放入unsorted bin时，取第一个chunk，bk不是unsoted bin

  

* ### malloc_consolidate(): invalid chunk size

  ```c
      if ((&fastbin (av, idx)) != fb)
        malloc_printerr ("malloc_consolidate(): invalid chunk size");
  ```

  size不合法

* ### realloc(): invalid old size

  ```c
    /* oldmem size */
    if (__builtin_expect (chunksize_nomask (oldp) <= 2 * SIZE_SZ, 0)
        || __builtin_expect (oldsize >= av->system_mem, 0))
      malloc_printerr ("realloc(): invalid old size");
  ```

  old size不合法

* ### realloc(): invalid next size

  ```c
    if (__builtin_expect (chunksize_nomask (next) <= 2 * SIZE_SZ, 0)
        || __builtin_expect (nextsize >= av->system_mem, 0))
      malloc_printerr ("realloc(): invalid next size");
  ```

  next size不合法

* 