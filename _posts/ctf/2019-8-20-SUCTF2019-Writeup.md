---
layout: post
title:  "SUCTF2019-Writeup"
date:   2019-8-20
desc: ""
keywords: ""
categories: [Ctf]
tags: [CTF]
icon: icon-html
---

# SUCTF2019-Writeup

* ### 签到题

  base64转图片![2](https://raw.githubusercontent.com/AiDaiP/images/master/suctf2019/2.png)

  按f进入坦克

* ### SignIn 

  ![1](https://raw.githubusercontent.com/AiDaiP/images/master/suctf2019/1.png)

  扫一眼就知道是白给rsa，直接解

  ```python
  import gmpy2
  p =  282164587459512124844245113950593348271
  q = 366669102002966856876605669837014229419
  c = 0xad939ff59f6e70bcbfad406f2494993757eee98b91bc244184a377520d06fc35
  e = 65537
  n = 103461035900816914121390101299049044413950405173712170434161686539878160984549
  phin = (p-1)*(q-1)
  d = gmpy2.invert(e,phin)
  m = pow(c,d,n)
  print(hex(m))
  ```



- ### hardCPP

  ![10](https://raw.githubusercontent.com/AiDaiP/images/master/suctf2019/10.png)

  ```python
  import string
  chars = string.printable
  enc = [  0xF3, 0x2E, 0x18, 0x36, 0xE1, 0x4C, 0x22, 0xD1, 0xF9, 0x8C, 0x40, 0x76, 0xF4, 0x0E, 0x00, 0x05, 0xA3, 0x90, 0x0E, 0xA5]
  for i in chars:
      a = [i]
      for j in range(20):
          a.append(chr(((enc[j] ^ ((ord(a[j])^18)*3+2))-(ord(a[j])%7))&0xff))
      s = "".join(a)
      if "flag" in s:
          print(s)
          break
  # #flag{mY-CurR1ed_Fns}
  ```

- ### CheckIn

  .user.ini后门

  ![3](https://raw.githubusercontent.com/AiDaiP/images/master/suctf2019/3.png)

  ![9](https://raw.githubusercontent.com/AiDaiP/images/master/suctf2019/9.png)

  ![4](https://raw.githubusercontent.com/AiDaiP/images/master/suctf2019/4.png)

  

* ### DSA

  DSA伪造签名

  求出x就完事了

  给出的几组数据中有r相同，存在共享k

  ```python
  from Crypto.PublicKey import DSA
  from hashlib import md5
  import gmpy2
  from Crypto.PublicKey import DSA
  key = DSA.generate(1024)
  
  def sign(m, k):
          inv_k = gmpy2.invert(k, q)
          r = pow(g, k, p) % q 
          s = (inv_k * (m + x * r)) % q
          return (r, int(s))
  
  p=89884656743115828531119184982796894663903477583059402619493107477701531448691777824805084580798468225554890927976916075470580037678339751860677039838524731471375089833773655274504660369369475511516039814042510089662618017515507661507423631391876847011336744142703525446772249114439095297538367680904122682163
  
  q=1083749197603514894592443021039982410258039824781
  
  g=19209255113109211287182003686336478382926591769410637118332162770232576550612837590140766948293798337911479071873750116529623755546575719467322207960901844857052068820875068819133458407537168202026447857684825810884084857815335831863851646098902970637458902267893093324494327651907517071660266730103869209198
  
  y=63855621553257553356471498867327897325399671664649190666029860850698259029192393459568867273267130887510106973938466063853354683404457978062463835928740975136972713067800589135585426284793875572930008629099902609983901843694733915531153990889746016426778036286422871654180294150416727660639507106302263504038
  
  
  fuck1 = 136494280340438758462640370068750063956
  m3 = fuck1
  
  fuck2 = 76447611971473350019028042637993930502
  
  m4 = fuck2
  s3 = 731852527883754918324666897777941019957384624211L
  s4 = 1029200344303632428382821043296603997470179874353L
  r = 395376858693035890919932439242396135161343196224L
  ds = s4 - s3
  dm = m4 - m3
  k = gmpy2.mul(dm, gmpy2.invert(ds, q))
  k = gmpy2.f_mod(k, q)
  print(k)
  tmp = gmpy2.mul(k, s3) - m3
  x = tmp * gmpy2.invert(r, q)
  x = gmpy2.f_mod(x, q)
  print(int(x))
  key.x = int(x)
  h = 334436397493699539473999398012751306876
  sig = sign(h,int(k))
  print(sig)
  
  (1053866066578981563752856210782507701312673283378L, 185546454661748073259065074929615976951712831621L)
  ```

  

  

  

* ### MT

  对明文重复加密最终可以得到明文，所以对密文重复加密，就可以得到明文

  ```python
from Crypto.Random import random
  from Crypto.Util import number
  
  def convert(m):
      m = m ^ m >> 13
      m = m ^ m << 9 & 2029229568
      m = m ^ m << 17 & 2245263360
      m = m ^ m >> 19
      return m
  
  def transform(message):
      new_message = ''
      for i in range(len(message) / 4):
          block = message[i * 4 : i * 4 +4]
          block = number.bytes_to_long(block)
          block = convert(block)
          block = number.long_to_bytes(block, 4)
          new_message += block
      return new_message
  c1 = '641460a9'
  c2 = 'e3953b1a'
  c3 = 'aa21f3a2'
  def decode(c):
  	fuck = c
  	while True:
  	    nmsl = fuck
  	    fuck = transform(fuck.decode('hex')).encode('hex')
  	    if fuck == c:
  	        return nmsl
  
  print(decode(c1)+decode(c2)+decode(c3))
  #flag{84b45f89af22ce7e67275bdc}
  ```





* ### Prime

  变形rsa，给出四组(n,c)，e=n

  gcd试一波发现模不互素

  n是四个质数乘积，可以通过gcd得到这四个素数，然后算phin求d解密

  ```python
  from hashlib import md5
  import string
  chars = string.letters+string.digits
  from pwn import *
  import gmpy2
  r = remote('47.111.59.243',8003)
  fuck = r.recvline()
  print(fuck)
  gg = fuck[40:44]
  print(gg)
  nmsl = fuck[54:-1]
  print(nmsl)
  def fuck():
  	for a in chars:
  		for b in chars:
  			for c in chars:
  				for d in chars:
  					shit = a+b+c+d+gg
  					if md5(shit).hexdigest()[0:5] == nmsl:
  						print(md5(shit).hexdigest())
  						print(shit)
  						return a+b+c+d
  r.sendline(fuck())
  
  cs0 = int(r.recvline()[10:].strip('L\n'),16)
  ns0 = int(r.recvline()[10:].strip('L\n'),16)
  cs1 = int(r.recvline()[10:].strip('L\n'),16)
  ns1 = int(r.recvline()[10:].strip('L\n'),16)
  cs2 = int(r.recvline()[10:].strip('L\n'),16)
  ns2 = int(r.recvline()[10:].strip('L\n'),16)
  cs3 = int(r.recvline()[10:].strip('L\n'),16)
  ns3 = int(r.recvline()[10:].strip('L\n'),16)
  p0 = int(gmpy2.gcd(ns0,ns1))
  p1 = int(gmpy2.gcd(ns0,ns2))
  p2 = int(gmpy2.gcd(ns0,ns3))
  nmsl = p0*p1*p2
  p3 = ns0//nmsl
  e0 = ns0
  phin = (p0-1)*(p1-1)*(p2-1)*(p3-1)
  d0 = gmpy2.invert(e0,phin)
  m0 = pow(cs0,d0,ns0)
  
  p0 = int(gmpy2.gcd(ns1,ns0))
  p1 = int(gmpy2.gcd(ns1,ns2))
  p2 = int(gmpy2.gcd(ns1,ns3))
  nmsl = p0*p1*p2
  p3 = ns1//nmsl
  e0 = ns1
  phin = (p0-1)*(p1-1)*(p2-1)*(p3-1)
  d0 = gmpy2.invert(e0,phin)
  m1 = pow(cs1,d0,ns1)
  
  p0 = int(gmpy2.gcd(ns2,ns0))
  p1 = int(gmpy2.gcd(ns2,ns1))
  p2 = int(gmpy2.gcd(ns2,ns3))
  nmsl = p0*p1*p2
  p3 = ns2//nmsl
  e0 = ns2
  phin = (p0-1)*(p1-1)*(p2-1)*(p3-1)
  d0 = gmpy2.invert(e0,phin)
  m2 = pow(cs2,d0,ns2)
  
  p0 = int(gmpy2.gcd(ns3,ns0))
  p1 = int(gmpy2.gcd(ns3,ns1))
  p2 = int(gmpy2.gcd(ns3,ns2))
  nmsl = p0*p1*p2
  p3 = ns3//nmsl
  e0 = ns3
  phin = (p0-1)*(p1-1)*(p2-1)*(p3-1)
  d0 = gmpy2.invert(e0,phin)
  m3 = pow(cs3,d0,ns3)
  
  
  print(hex(m0))
  print(hex(m1))
  print(hex(m2))
  print(hex(m3))
  r.interactive()
  ```

  





* ### RSA

  RSA Parity Oracle Attack

  ```python
  # -*- coding: utf-8 -*-
  import libnum, gmpy2, socket, time, decimal
  from hashlib import md5
  import string
  chars = string.letters+string.digits
  from pwn import *
  import gmpy2
  import re
  import libnum
  import decimal
  
  r = remote('47.111.59.243',9421)
  fuck = r.recvline()
  print(fuck)
  gg = fuck[40:44]
  print(gg)
  nmsl = fuck[54:-1]
  print(nmsl)
  
  
  def fuck():
      for a in chars:
          for b in chars:
              for c in chars:
                  for d in chars:
                      shit = a+b+c+d+gg
                      if md5(shit).hexdigest()[0:5] == nmsl:
                          print(md5(shit).hexdigest())
                          print(shit)
                          return a+b+c+d
  
  def oracle(c):
      r.sendline('D')
  
      r.sendline(str(c))
      sh = r.recvuntil('option')
      if 'odd' in sh:
          return 1
      elif 'even' in sh:
          return 0
  
  
  
  r.sendline(fuck())
  ss = r.recvuntil('option')
  n = int(re.findall("n\s*=\s*(\d+)",ss)[0])
  e = int(re.findall("e\s*=\s*(\d+)",ss)[0])
  c = int(re.findall("c\s*=\s*(\d+)",ss)[0])
  global c_of_2
  k = n.bit_length()
  decimal.getcontext().prec = k
  lower = decimal.Decimal(0)
  upper = decimal.Decimal(n)
  c_of_2 = pow(2, e, n)
  c = (c * c_of_2) % n
  for i in range(k):
      possible_plaintext = (lower + upper) / 2
      flag = oracle(c)
      if not flag:
          upper = possible_plaintext
      else:
          lower = possible_plaintext
      c = (c * c_of_2) % n
      print i, flag, int(upper - lower)
  
  print(int(upper))
  r.sendline('G')
  r.sendline(str(int(upper)))
  
  
  r.recvuntil('Congratulations!')
  ss = r.recvuntil('option')
  n = int(re.findall("n\s*=\s*(\d+)",ss)[0])
  e = int(re.findall("e\s*=\s*(\d+)",ss)[0])
  c = int(re.findall("c\s*=\s*(\d+)",ss)[0])
  global c_of_2
  k = n.bit_length()
  decimal.getcontext().prec = k 
  lower = decimal.Decimal(0)
  upper = decimal.Decimal(n)
  c_of_2 = pow(2, e, n)
  c = (c * c_of_2) % n
  for i in range(k):
      possible_plaintext = (lower + upper) / 2
      flag = oracle(c)
      if not flag:
          upper = possible_plaintext
      else:
          lower = possible_plaintext
      c = (c * c_of_2) % n  
      print i, flag, int(upper - lower)
  
  print(int(upper))
  r.sendline('G')
  r.sendline(str(int(upper)))
  
  r.recvuntil('Congratulations!')
  ss = r.recvuntil('option')
  n = int(re.findall("n\s*=\s*(\d+)",ss)[0])
  e = int(re.findall("e\s*=\s*(\d+)",ss)[0])
  c = int(re.findall("c\s*=\s*(\d+)",ss)[0])
  global c_of_2
  k = n.bit_length()
  decimal.getcontext().prec = k 
  lower = decimal.Decimal(0)
  upper = decimal.Decimal(n)
  c_of_2 = pow(2, e, n)
  c = (c * c_of_2) % n
  for i in range(k):
      possible_plaintext = (lower + upper) / 2
      flag = oracle(c)
      if not flag:
          upper = possible_plaintext 
      else:
          lower = possible_plaintext  
      c = (c * c_of_2) % n  
      print i, flag, int(upper - lower)
  
  print(int(upper))
  
  r.interactive()
  #flag{aabe6ba4245f3685db029b126068b7ab393187ab}
  ```

  

* ### Game

  ![5](https://raw.githubusercontent.com/AiDaiP/images/master/suctf2019/5.png)

  这个魔方没什么卵用，但俺刚开始还是想转魔方，录像倒放懒得搞，外挂也找不到，计时器也关不掉，gg

  看源码能发现一个假flag

  ```html
      <div class="text text--best-time">
        <icon trophy></icon>
        <span>Well done!，
  	  here is your flag:ON2WG5DGPNUECSDBNBQV6RTBNMZV6RRRMFTX2===	  </span>
      </div>
  ```

  base32解开

  ```
  suctf{hAHaha_Fak3_F1ag}
  ```

  根据这个洋文可以判断它是个假的flag，但是还是要交一发gtmd魔方

  然后俺就不会做了，准备弃赛的时候看到了secret这个洋文单词

  ![6](https://raw.githubusercontent.com/AiDaiP/images/master/suctf2019/6.png)

  然后找到了这个，得到了一张图片

  ![iZwz9i9xnerwj6o7h40eauZ](https://raw.githubusercontent.com/AiDaiP/images/master/suctf2019/iZwz9i9xnerwj6o7h40eauZ.png)

  然后按图片隐写走，zsteg看一眼

  ![7](https://raw.githubusercontent.com/AiDaiP/images/master/suctf2019/7.png)

  ```python
  U2FsdGVkX1+zHjSBeYPtWQVSwXzcVFZLu6Qm0To/KeuHg8vKAxFrVQ==
  
  >>> base64.b64decode('U2FsdGVkX1+zHjSBeYPtWQVSwXzcVFZLu6Qm0To/KeuHg8vKAxFrVQ==')
  'Salted__\xb3\x1e4\x81y\x83\xedY\x05R\xc1|\xdcTVK\xbb\xa4&\xd1:?)\xeb\x87\x83\xcb\xca\x03\x11kU'
  ```

  wdnmd，应该是某种加密方法的密文用base64编码

  密钥应该是假flag

  最后试出来是3DES

  ![8](https://raw.githubusercontent.com/AiDaiP/images/master/suctf2019/8.png)

  