---
layout: post
title:  "RSA Parity Oracle Attack"
date:   2019-8-19
desc: ""
keywords: ""
categories: [Crypto]
tags: [Crypto,RSA]
icon: icon-html
---

# RSA Parity Oracle Attack

* ### RSA Parity Oracle

   Oracle会对一个给定的密文进行解密，并且会检查解密的明文的奇偶性，并根据奇偶性返回相应的值 

* ### RSA Parity Oracle Attack

  $C=P^e  \% N$ 

  $[C*(2^e  \% N)] \% N=(2P)^e \% N$ 

  2P可能大于N，向 Oracle 发送$[C*(2^e  \% N)] \% N$ 解密结果为：

  $2P  \% N​$ 

  2P是偶数，N是奇数

  若返回奇数，则2P大于N，且减去了奇数个N

  $P<N$ ，$2P<2N$ ，所以减去了一个N，$\frac{N}{2} \leq P < N$ 

  若返回偶数，则$2P<N$，$0\leq P < \frac{N}{2}$ 

  

  设第 i 次

  $\frac{xN}{2^{i}} \leq P < \frac{xN+N}{2^{i}}$ 

  即$\frac{2xN}{2^{i+1}} \leq P < \frac{2xN+2N}{2^{i+1}}$ 

  第 i+1 次时

  $[C*(2^e \% N)^{i+1}] \% N=(2^{i+1}P)^e  \% N$

   Oracle 解密结果为：$2^{i+1}P  \% N$ 

  $2^{i+1}P  \% N = P - kN$

  $0 \leq 2^{i+1}P-kN<N$ 

   $\frac{kN}{2^{i+1}} \leq P < \frac{kN+N}{2^{i+1}}$ 

  

  若返回奇数，则k必为奇数，$k=2y+1$，$\frac{2yN+N}{2^{i+1}} \leq P < \frac{2yN+2N}{2^{i+1}}$ 

  P必然存在，所以第 i+1 得到的这个范围和第 i 次得到的范围存在交集，所以x=y

  $\frac{2xN+N}{2^{i+1}} \leq P < \frac{2xN+2N}{2^{i+1}}$ 

  若返回偶数，则k必为偶数，k=2y，此时x=y

  $\frac{2xN}{2^{i+1}} \leq P < \frac{2xN+N}{2^{i+1}}$ 

  

* ### Python 实现

  ```python
  import decimal
  
  def oracle(c):
      res = 
      if 'odd' in res:
          return 1-
      elif 'even' in res:
          return 0
      
  def partial():
      global c_of_2
      n = 
      e = 
      c = 
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
      return int(upper)
  
  ```

  测试

  ```python
  def test():
      m = 73904820447083971267764906753665956116805164064084252932601728902283378026455902777427033972246117072981537951251680291937307449076406485024169361513248547737418408638796693001182844624306163889491586642320757658159933400582011510745854851756198271438794297727258100420272310822955739463807948806460888635711
      q = 10100914392104787390982036171384752749613793454325038443552251400597822359641840924237959230680261515583219427782992336733970800066559155097120290428855873
      p = 9515372490491300154109691469519317779706959918786075851113761293623315179805737125514146395225758080589972044505992392674891367216666249056145209710924649
      n = 96113962935441547934939223641864934667329279620673552654468819296006381484879481903361997272827032771012844198527930972593816083419344456579806295288932860787996647731761554662573734214843486835765188394938363407235010868782395265778991702514580891323440714788597176910231491318172343955993004374877984113577
      e = 65537
      d = 52174349533717877066899580699192622727999549136901022464186378919918870679250994830309693989320452841319452297279852179394809054178961478054887907002137338564852626828611907235491716368220375307903326983800373369817433465929374103000087138986358584573135380195299067898098205371848513672451118712593286082561
      m = random.randint(0, n-1)
      c = pow(m,e,n)    
      k = n.bit_length()
      decimal.getcontext().prec = k
      lower = decimal.Decimal(0)
      upper = decimal.Decimal(n)
      c_of_2 = pow(2, e, n)
      c = (c * c_of_2) % n
      for i in range(k):
          possible_plaintext = (lower + upper) / 2
          cc = c
          mm = pow(c,d,n)
          if mm & 1 == 1:
              print 'odd'
              flag = 1
          else:
              print 'even'
              flag = 0
  
          if not flag:
              upper = possible_plaintext
          else:
              lower = possible_plaintext
          c = (c * c_of_2) % n
          print i, flag, int(upper - lower)
      print(int(upper))
      print(m)
      print(m==int(upper))
  ```

  

