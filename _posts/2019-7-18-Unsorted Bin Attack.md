---
layout: post
title:  "Unsorted Bin Attack"
date:   2019-7-18
desc: ""
keywords: ""
categories: [Binary]
tags: [pwn,heap]
icon: icon-html
---

# Unsorted Bin Attack

![11](https://raw.githubusercontent.com/AiDaiP/images/master/pwn/11.png)

* ### Unsorted bin

  * #### 来源

    1. 当一个较大的 chunk 被分割成两半后，如果剩下的部分大于 MINSIZE，就会被放到 unsorted bin 中。

       ```c
       /* The smallest size we can malloc is an aligned minimal chunk */
       
       #define MINSIZE                                                                
           (unsigned long) (((MIN_CHUNK_SIZE + MALLOC_ALIGN_MASK) &                   
                             ~MALLOC_ALIGN_MASK))
       ```

       

    2. 释放一个不属于 fast bin 的 chunk，并且该 chunk 不和 top chunk 紧邻时，该 chunk 会被首先放到 unsorted bin 中。 

    3. 当进行 malloc_consolidate 时，如果不是和 top chunk 近邻，可能会把合并后的 chunk 放到 unsorted bin 中

  * #### 使用

    1. 先进先出，插入的时候插入到 unsorted bin 的头部，取出的时候从链表尾获取
    2. malloc 时，如果在 fastbin，small bin 中找不到对应大小的 chunk，就会尝试从 Unsorted Bin 中寻找 chunk。如果取出来的 chunk 大小刚好满足，就会直接返回给用户，否则就会把这些 chunk 分别插入到对应的 bin 中 

* ### 原理

  ```c
            /* remove from unsorted list */
            if (__glibc_unlikely (bck->fd != victim))
              malloc_printerr ("malloc(): corrupted unsorted chunks 3");
            unsorted_chunks (av)->bk = bck;
            bck->fd = unsorted_chunks (av);
  ```

  将一个unsorted bin取出时， bck->fd变为取出的unsorted bin的位置

  如果可以控制bk，就能将`unsorted_chunks (av) `写到任意地址

