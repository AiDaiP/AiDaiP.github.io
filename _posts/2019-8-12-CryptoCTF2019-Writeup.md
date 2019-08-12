---
layout: post
title:  "CryptoCTF2019-Writeup"
date:   2019-8-12
desc: ""
keywords: ""
categories: [Ctf]
tags: [CTF,Crypto]
icon: icon-html
---

# 2019-8-12-CryptoCTF2019-Writeup

* ### Decode me!

  ```
  mb xwhvxw mlnX 4X6AhPLAR4eupSRJ6FLt8AgE6JsLdBRxq57L8IeMyBRHp6IGsmgFIB5E :ztey xam lb lbaH
  ```

  凯撒，中间那一坨像base64

  大写字母在右侧，字符串估计是反的，先凯撒一波

  根据洋文判断

  ```
  ti edoced tsuE 4E6HoWSHY4lbwZYQ6MSa8HnL6QzSkIYex57S8PlTfIYOw6PNztnMPI5L :galf eht si sihO
  Ohis is the flag: L5IPMntzNP6wOYIfTlP8S75xeYIkSzQ6LnH8aSM6QYZwbl4YHSWoH6E4 Eust decode it
  ```

  根据洋文判断大小写字母偏移量不同，小写是7，第一个洋文单词应该是This，所以大写单词偏移量为12

  俺寻思数字偏移量跟大小写字母也不同，根据base64爆破一下

  ```
  This is the flag: Q0NURntzSU1wTDNfYlU3X20xeDNkXzV1QnM3aXR1VDEwbl9DMXBoM1J9 Just decode it
  CCTF{sIMpL3_bU7_m1x3d_5uBs7ituT10n_C1ph3R}
  ```

* ### Time Capsule

  ```python
  #!/usr/bin/python
  from Crypto.Util.number import *
  from secret import flag, n, t, z
  def encrypt_time_capsule(msg, n, t, z):
  	m = bytes_to_long(msg)
  	l = pow(2, pow(2, t), n)
  	c = l ^ z ^ m
  	return (c, n, t, z) 
  
  print encrypt_time_capsule(flag, n, t, z)
  ```

  算pow(2, pow(2, t), n)就完事了

  直接算时间不够

  $pow(2, x, n) = pow(2, pow(2, t, phin), n)$

  分解n算phin就行了

  http://factordb.com直接分解

  ```python
  from Crypto.Util.number import *
  fuck = (30263951492003430418944035844723976843761515320480688994488846431636782360488888188067655841720110193942081554547272176290791213962513701884837856823209432209367951673301622535940395295826053396595886942990258678430777333636450042181585837395671842878310404080487115827773100028876775230121509570227303374672524063165714509957850966189605469484201028704363052317830254920108664916139026741331552127849056897534960886647382429202269846392809641322613341548025760209280611758326300214885296175538901366986310471066687700879304860668964595202268317011117634615297226602309205086105573924029744405559823548638486054634428L, 16801166465109052984956796702219479136700692152603640001472470493600002617002298302681832215942994746974878002533318970006820414971818787350153626339308150944829424332670924459749331062287393811934457789103209090873472485865328414154574392274611574654819495894137917800304580119452390318440601827273834522783696472257727329819952363099498446006266115011271978143149347765073211516486037823196033938908784720042927986421555211961923200006343296692217770693318701970436618066568854673260978968978974409802211538011638213976732286150311971354861300195440286582255769421094876667270445809991401456443444265323573485901383L, 6039738711082505929, 13991757597132156574040593242062545731003627107933800388678432418251474177745394167528325524552592875014173967690166427876430087295180152485599151947856471802414472083299904768768434074446565880773029215057131908495627123103779932128807797869164409662146821626628200600678966223382354752280901657213357146668056525234446747959642220954294230018094612469738051942026463767172625588865125393400027831917763819584423585903587577154729283694206436985549513217882666427997109549686825235958909428605247221998366006018410026392446064720747424287400728961283471932279824049509228058334419865822774654587977497006575152095818L)
  c = fuck[0]
  n = fuck[1]
  t = fuck[2]
  z = fuck[3]
  factors = [15013, 583343756982313, 585503197547927, 609245815680559, 612567235432583, 634947980859229, 635224892351513, 639438000563939, 654170414254271, 654269804672441, 667954470985657, 706144068530309, 721443717105973, 737993471695639, 744872496387077, 746232585529679, 795581973851653, 815694637597057, 817224718609627, 841183196554507, 864339847436159, 873021823131881, 884236929660113, 899583643974479, 922745965897867, 942872831732189, 951697329369323, 971274523714349, 1017566110290559, 1018452110902339, 1025985735184171, 1027313536626551, 1059774237802229, 1067609726096989, 1070689247726159, 1079289330417443, 1098516592571807, 1107673252158281, 1108654254305327, 1110918654474373, 1111516996694389, 1112193819715441]
  phin = 1
  for i in factors:
  	phin *= (i-1)
  l =  pow(2, pow(2, t, phin), n)
  m = l ^ z ^ c
  print(m)
  print(long_to_bytes(m))
  #CCTF{_______________________________________________Happy_Birthday_LCS______________________________________________}
  ```

  

* ###Clever Girl

  ```python
  #!/usr/bin/env python
  import gmpy2
  from fractions import Fraction
  from secret import p, q, s, X, Y
  from flag import flag
  
  assert gmpy2.is_prime(p) * gmpy2.is_prime(q) > 0
  assert Fraction(p, p+1) + Fraction(q+1, q) == Fraction(2*s - X, s + Y)
  print 'Fraction(p, p+1) + Fraction(q+1, q) = Fraction(2*s - %s, s + %s)' % (X, Y)
  
  n = p * q
  c = pow(int(flag.encode('hex'), 16), 0x20002, n)
  print 'n =', n
  print 'c =', c
  
  ```

  $\frac{p}{p+1}+\frac{q+1}{1}=\frac{2s-X}{s+Y}$

  $\frac{2n+p+q+1}{n+q}=\frac{2s-X}{s+Y}$

  假设

  $2s-X=2n+p+q+1$

  $s+Y=n+q$

  解方程得到pq

  ```python
  import sympy
  X = 153801856029563198525204130558738800846256680799373350925981555360388985602786501362501554433635610131437376183630577217917787342621398264625389914280509
  Y = 8086061902465799210233863613232941060876437002894022994953293934963170056653232109405937694010696299303888742108631749969054117542816358078039478109426
  n = 161010103536746712075112156042553283066813155993777943981946663919051986586388748662616958741697621238654724628406094469789970509959159343108847331259823125490271091357244742345403096394500947202321339572876147277506789731024810289354756781901338337411136794489136638411531539112369520980466458615878975406339
  p=sympy.symbols('p')
  p=sympy.symbols('q')
  p=sympy.symbols('s')
  sympy.solve([n+q - s - Y, 2*n+p+q+1-2*s+X, p*q-n], (p,q,s))
  ```

  $c ≡  (m^{2})^{65537}\ mod\ n$

  e与φ(n)不互素

  先转换成

  $c_1≡  m^{2}\ mod\ n$

  然后爆破m

  ```python
  import gmpy
  from Crypto.Util.number import *
  p = 12604273285023995463340817959574344558787108098986028639834181397979984443923512555395852711753996829630650627741178073792454428457548575860120924352450409
  q= 12774247264858490260286489817359549241755117653791190036750069541210299769639605520977166141575653832360695781409025914510310324035255606840902393222949771
  n = 161010103536746712075112156042553283066813155993777943981946663919051986586388748662616958741697621238654724628406094469789970509959159343108847331259823125490271091357244742345403096394500947202321339572876147277506789731024810289354756781901338337411136794489136638411531539112369520980466458615878975406339
  c = 64166146958225113130966383399465462600516627646827654061505253681784027524205938322376396685421354659091159523153346321216052274404398431369574383580893610370389016662302880230566394277969479472339696624461863666891731292801506958051383432113998695237733732222591191217365300789670291769876292466495287189494
  b = 2
  a = 65537
  phin = (p-1)*(q-1)
  bd = gmpy.invert(a,phin)
  c = pow(c,bd,n)
  i = 0
  while 1:
      if gmpy.root(c+i*c,2)[1]==1:
          m= gmpy.root(c+i*c, 2)[0]
          break
      i += 1
  print(long_to_bytes(m))
  #CCTF{4Ll___G1rL5___Are__T4len73E__:P}
  ```

* ### Aron

  * ### 非预期解

      ```
      *********************************************************************************
      | hey! I have developed an efficient pseudorandom function, PRF, but it needs   |
      | deep tests for security points!! Try hard to break this PRF and get the flag! |
      | In each step I will compute the f_a(n), f_a(n + 1), f_a(n + 2), f_a(n+3), and |
      | f_a(n + 4) for secret verctor a, and for your given positive number 0 < n < p |
      *********************************************************************************
      | for n = 133405621931957608209509172726988831973, and with these PRF parameters:
      | (p, g) = (0xef1bbcfefad8e7ee73301ee8522639bd, 0x495ba1e98cab4d10e3739aa1b51d25de)
      | the five consecutive random numbers generated by our secure PRF are:
      | f_a(n + 0) = 298766663226902994782897360082395953159
      | f_a(n + 1) = 33479012578134692795345802086090312435
      | f_a(n + 2) = 115606985662707724254358253451438702104
      | f_a(n + 3) = 40785189116942677349327713781909618113
      | f_a(n + 4) = 186165789474863334614202245998513357998
      | Options:
      |    [G]uess next number!
      |    [P]RF function
      |    [N]ew numbers
      |    [Q]uit
      ```

      预测f_a(n + 5)

      俺一看可以改n，那俺把n改成n-1然后输入f_a(n + 4)就完事了

      ```
      $ N
      Do you want to provide desired integer as `n'? [Y]es [N]o
      $ Y
      enter your integer n:
      $ 133405621931957608209509172726988831972
      | the five consecutive random numbers generated by our secure PRF are:
      | f_a(n + 0) = 303296073606278266079727338726791413872
      | f_a(n + 1) = 298766663226902994782897360082395953159
      | f_a(n + 2) = 33479012578134692795345802086090312435
      | f_a(n + 3) = 115606985662707724254358253451438702104
      | f_a(n + 4) = 40785189116942677349327713781909618113
      | Options:
      |    [G]uess next number!
      |    [P]RF function
      |    [N]ew numbers
      |    [Q]uit
      $ G
      please guess and enter the next number:
      $ 186165789474863334614202245998513357998
      Congratz! :) You got the flag: CCTF{___Naor-Reingold___p5euD0r4ndOM_fuNc710N__PRF__}
      [*] Got EOF while reading in interactive
      ```

  * ### 预期解

      等wp

      有个带佬的wp就写了几句也没写清楚，但是俺从他那学会了新的proofofwork姿势

      ```python
      import re
      chal=re.match('Submit a printable string X, such that (.*)\(X\)\[-6:\] = ([0-9a-f]{6})',r.recvline())
      h=chal.group(1)
      v=chal.group(2)
      print(h+"(x) = "+v)
      i=0
      exec('from hashlib import '+h+'\nwhile True:\n if('+h+'(str(i)).hexdigest()[-6:]=="'+v+'"):break\n i+=1')
      print "x = %d"%i
      ```

      

* ### roXen

  ```
  Relationship with a cryptographer!
  The Girlfriend: All you ever care about is crypto! I am sick of it! It's me or crypto!
  The Cryptographer boyfriend: You meant to say it's you XOR cryptography.
  The Girlfriend: I am leaving you.
  ```

  ```python
  #!/usr/bin/env python
  
  from Crypto.Util.number import *
  from secret import exp, flag, nbit
  
  assert exp & (exp + 1) == 0
  
  def adlit(x):
      l = len(bin(x)[2:])
      return (2 ** l - 1) ^ x
  
  def genadlit(nbit):
      while True:
          p = getPrime(nbit)
          q = adlit(p) + 31337
          if isPrime(q):
              return p, q
  
  p, q = genadlit(nbit)
  e, n = exp, p * q
  
  c = pow(bytes_to_long(flag), e, n)
  
  print 'n =', hex(n)
  print 'c =', hex(c)
  ```

  exp为梅森素数，n2048位，pq1024位

  根据genadlit(nbit)可得

  $(2^{1024}-1) ⊕ p + 31337 = q$

  $(2^{1024}-1) ⊕ p$，p所有为0的位变为1，为1的位变为0，所以

  $(2^{1024}-1) ⊕ p+p=2^{1024}-1$

  所以

  $p+q=2^{1024}+31336$

  $p*q=n$，解方程

  ```python
  import sympy
  n = 0x3ff77ad8783e006b6a2c9857f2f13a9d896297558e7c986c491e30c1a920512a0bad9f07c5569cf998fc35a3071de9d8b0f5ada4f8767b828e35044abce5dcf88f80d1c0a0b682605cce776a184e1bcb8118790fff92dc519d24f998a9c04faf43c434bef6c0fa39a3db7452dc07ccfced9271799f37d91d56b5f21c51651d6a9a41ee5a8af17a2f945fac2b1a0ea98bc70ef0f3e37371c9c7b6f90d3d811212fc80e0abcd5bbefe0c6edb3ca6845ded90677ccd8ff4de2c747b37265fc1250ba9aa89b4fd2bdfb4b4b72a7ff5b5ee67e81fd25027b6cb49db610ec60a05016e125ce0848f2c32bff33eed415a6d227262b338b0d1f3803d83977341c0d3638fL
  p = sympy.symbols('p')
  q = sympy.symbols('q')
  sympy.solve([p*q-n, p+q-2**1024-31336], (p, q))
  ```

  ```
  p = 91934396941118575436929554782758166784623142015203107928295225306949429527662253180027648166060067602233902389535868116051536080388999480377007211745229221564969130373120800620379012435790356909945473565305296926519232706950561924532325538399351352696805684504904629096892037592742285758390953849377910498739
  q = 87834916545113015336000964296144306577174555879027549345134855850783246277838709952680829156347468418886211490335525241607253688425417142115840218894244902812798763051744684655923207165455737209507609386779708842318917975391900956941587572141475884466544826179681669143055208345737430546444402480246313669813
  ```

  然后爆破e，爆破时应考虑e与φ(n)不互素的情况

  ```python
  import gmpy2
  from Crypto.Util.number import *
  c = 0x2672cade2272f3024fd2d1984ea1b8e54809977e7a8c70a07e2560f39e6fcce0e292426e28df51492dec67d000d640f3e5b4c6c447845e70d1432a3c816a33da6a276b0baabd0111279c9f267a90333625425b1d73f1cdc254ded2ad54955914824fc99e65b3dea3e365cfb1dce6e025986b2485b6c13ca0ee73c2433cf0ca0265afe42cbf647b5c721a6e51514220bab8fcb9cff570a6922bceb12e9d61115357afe1705bda3c3f0b647ba37711c560b75841135198cc076d0a52c74f9802760c1f881887cc3e50b7e0ff36f0d9fa1bfc66dff717f032c066b555e315cb07e3df13774eaa70b18ea1bb3ea0fd1227d4bac84be2660552d3885c79815baef661L
  p = 91934396941118575436929554782758166784623142015203107928295225306949429527662253180027648166060067602233902389535868116051536080388999480377007211745229221564969130373120800620379012435790356909945473565305296926519232706950561924532325538399351352696805684504904629096892037592742285758390953849377910498739
  q = 87834916545113015336000964296144306577174555879027549345134855850783246277838709952680829156347468418886211490335525241607253688425417142115840218894244902812798763051744684655923207165455737209507609386779708842318917975391900956941587572141475884466544826179681669143055208345737430546444402480246313669813
  n = 0x3ff77ad8783e006b6a2c9857f2f13a9d896297558e7c986c491e30c1a920512a0bad9f07c5569cf998fc35a3071de9d8b0f5ada4f8767b828e35044abce5dcf88f80d1c0a0b682605cce776a184e1bcb8118790fff92dc519d24f998a9c04faf43c434bef6c0fa39a3db7452dc07ccfced9271799f37d91d56b5f21c51651d6a9a41ee5a8af17a2f945fac2b1a0ea98bc70ef0f3e37371c9c7b6f90d3d811212fc80e0abcd5bbefe0c6edb3ca6845ded90677ccd8ff4de2c747b37265fc1250ba9aa89b4fd2bdfb4b4b72a7ff5b5ee67e81fd25027b6cb49db610ec60a05016e125ce0848f2c32bff33eed415a6d227262b338b0d1f3803d83977341c0d3638fL
  phin = (p-1)*(q-1)
  i = 1
  while True:
      i += 1
      e = 2**i - 1
      a =  gmpy2.gcd(e, phin)
      b = e // a
      try:
          bd = gmpy2.invert(b, phin)
      except:
          continue
      fuck = pow(c, bd, n)
      
      p = gmpy2.iroot(fuck, a)[0]
      m = long_to_bytes(p)
      if 'CCTF{' in m:
          print(i)
          print(m)
          break
  #i = 3729
  #CCTF{it5_3a5y_l1k3_5uNd4y_MOrn1N9}
  ```

  

* ### Fast Speedy!

  ```python
  #!/usr/bin/env python
  
  import random
  from Crypto.Util.number import *
  
  def drift(R, B):
      n = len(B)
      ans, ini = R[-1], 0
      for i in B:
          ini ^= R[i-1]
      R = [ini] + R[:-1]
      return ans, R
  
  flag = open('flag.png', 'r').read()
  flag = bin(int(flag.encode('hex'), 16))[2:]
  
  r = random.randint(7, 128)
  s = random.randint(2, r)
  R = [random.randint(0, 1) for _ in xrange(r)]
  B = [i for i in xrange(s)]
  random.shuffle(B)
  
  A, enc = [], []
  for i in range(len(flag)):
      ans, R = drift(R, B)
      A = A + [ans]
      enc = enc + [int(flag[i]) ^ ans]
  
  enc = ''.join([str(b) for b in enc])
  f = open('flag.png.enc', 'w')
  f.write(long_to_bytes(int(enc, 2)))
  f.close()
  ```

  长得像lfsr

  flag的每一位和drift每一次的out异或得到密文

  random.shuffle(B)没用，B控制drift内异或顺序，怎么换都无所吊谓

  搞到s，R然后输入flag.png.enc就可以得到明文

  先通过png头爆破，out必为`bin(enc_head ^ png_head)`

  每次的ans是R的最后一位，然后去掉R的最后一位，异或结果放到R第一位，生成新的R

  所以初始R必为`out[:r][::-1]`

  B遍历(2, r)找到可以生成out的(R,B)组合

  ```python
  def drift(R, B):
      n = len(B)
      ans, ini = R[-1], 0
      for i in B:
          ini ^= R[i-1]
      R = [ini] + R[:-1]
      return ans, R
  enc_head = 0x7006d570000627b6fcf3881fbe2f1817
  png_head = 0x89504e470d0a1a0a0000000d49484452
  fuck = bin(enc_head ^ png_head)[2:]
  out = [int(i) for i in fuck]
  
  for r in range(7,128):
      for s in range(2,r):
          R = out[:r][::-1]
          B = [i for i in range(s)]
          for i in range(len(out)):
              t, R = drift(R,B)
              if t != out[i]:
                  break
          if i == len(out)-1:
              print(r)
              print(s)
              print(R)
  ```

  出来225组，我寻思爆破一下

  第一组就是答案

  ```python
  r = 13
  b = 5
  R = [0, 1, 0, 1, 0, 1, 0, 0, 1, 1, 1, 1, 1]
  ```

  ```python
  import random
  from Crypto.Util.number import *
  
  def drift(R, B):
      n = len(B)
      ans, ini = R[-1], 0
      for i in B:
          ini ^= R[i-1]
      R = [ini] + R[:-1]
      return ans, R
  
  flag = open('flag.png.enc', 'r').read()
  flag = '0' + bin(int(flag.encode('hex'), 16))[2:]
  
  R = [0, 1, 0, 1, 0, 1, 0, 0, 1, 1, 1, 1, 1]
  s = 5
  B = [i for i in xrange(s)]
  
  A, enc = [], []
  for i in range(len(flag)):
      ans, R = drift(R, B)
      A = A + [ans]
      enc = enc + [int(flag[i]) ^ ans]
  
  enc = ''.join([str(b) for b in enc])
  f = open('flag.png', 'w')
  f.write(long_to_bytes(int(enc, 2)))
  f.close()
  #CCTF{LFSR__In___51mPL3___w0rD5}
  ```

  ![12](https://raw.githubusercontent.com/AiDaiP/images/master/crypto/12.png)

  

  

  

  

  

  