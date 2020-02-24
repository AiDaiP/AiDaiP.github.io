---
clayout: post
title:  "House of Strom"
date:   2020-2-20
desc: ""
keywords: ""
categories: [Binary]
tags: [pwn]
icon: icon-html
---

# House of Strom

> 风暴要火

## 原理

unsorted bin + large bin

glibc-2.23

### unsorted bin

```c
          /* remove from unsorted list */
          unsorted_chunks (av)->bk = bck;
          bck->fd = unsorted_chunks (av);

          /* Take now instead of binning if exact fit */

          if (size == nb)
            {
              set_inuse_bit_at_offset (victim, size);
              if (av != &main_arena)
                victim->size |= NON_MAIN_ARENA;
              check_malloced_chunk (av, victim, nb);
              void *p = chunk2mem (victim);
              alloc_perturb (p, bytes);
              return p;
            }
```

在unsorted bin解链的时候没有检查当前取出的chunk，通过控制bk可以把fakechunk塞进去

p->fd=0，p->bk=fake_chunk

下次malloc把p取出来，fake_chunk进链 

### large bin

```c
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
                  assert ((bck->bk->size & NON_MAIN_ARENA) == 0);
                  if ((unsigned long) (size) < (unsigned long) (bck->bk->size))
                    {
                      fwd = bck;
                      bck = bck->bk;

                      victim->fd_nextsize = fwd->fd;
                      victim->bk_nextsize = fwd->fd->bk_nextsize;
                      fwd->fd->bk_nextsize = victim->bk_nextsize->fd_nextsize = victim;
                    }
                  else
                    {
                      assert ((fwd->size & NON_MAIN_ARENA) == 0);
                      while ((unsigned long) size < fwd->size)
                        {
                          fwd = fwd->fd_nextsize;
                          assert ((fwd->size & NON_MAIN_ARENA) == 0);
                        }

                      if ((unsigned long) size == (unsigned long) fwd->size)
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
```

malloc时，在unsorted bin中找，如果size不符合malloc的需求，就把这个chunk放到small bin或者large bin中

进large bin时

```c
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
```

fwd是large bin中第一个chunk，fwd可控可以搞事

```c
                      else
                        {
                          victim->fd_nextsize = fwd;
                          victim->bk_nextsize = fwd->bk_nextsize;
                          fwd->bk_nextsize = victim;
                          victim->bk_nextsize->fd_nextsize = victim;
                        }
```

victim是unsorted bin，fwd是large bin，把unsorted bin插入large bin

`fwd->bk_nextsize = victim;`

如果可以控制fwd->bk_nextsize，就可以任意地址写入一个堆地址

```
          victim->bk = bck;
          victim->fd = fwd;
          fwd->bk = victim;
          bck->fd = victim;
```

`fwd->bk`赋给bck，后面有` bck->fd = victim;`，bck必须是可写的地址

### House of Strom

#### 条件

1. 可以控制unsorted bin和large bin

2. 任意地址chunk的size的低四位要为0

#### 基本操作

利用unsorted bin把fake chunk塞进unsorted bin 链，利用lage bin伪造fake chunk size，实现任意地址分配chunk

1. 搞出来一个unsorted bin A和一个large bin B

2. 把fake chunk塞进unsorted bin 链

   ```
   A->fd = 0
   A->bk = fake_chunk
   ```

3. lage bin伪造fake chunk size

   ```
   B->fd = 0
   B->bk = fake_chunk+8//fake_chunk+8可写
   B->fd_nextsiz = 0
   B->bk_nextsize = fake_chunk - 0x18 - 5
   ```

   fake_chunk - 0x18 - 5地址没有对齐，把一个chunk地址写入后，对齐来看size的位置刚好为一个合法size

   在开PIE的情况下为0x555555756000附近，size为0x55

   没开PIE时size可能为0x610000-0x25d0000

4. malloc到fake chunk

   无PIE时，malloc(0x48)，1/3概率成功

   有PIE时，1/32概率成功