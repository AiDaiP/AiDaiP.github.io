---
layout: post
title:  "Jarvis OJ-从Crypto到去世"
date:   2019-2-12
desc: ""
keywords: ""
categories: [Ctf]
tags: [CTF,JarvisOJ,Crypto]
icon: icon-html
---

# Jarvis OJ-从Crypto到去世

* #### Medium RSA 

  题目给了公钥和密文

  ```
  OpenSSL> rsa -pubin -text -modulus -in pubkey.pem
  Public-Key: (256 bit)
  Modulus:
      00:c2:63:6a:e5:c3:d8:e4:3f:fb:97:ab:09:02:8f:
      1a:ac:6c:0b:f6:cd:3d:70:eb:ca:28:1b:ff:e9:7f:
      be:30:dd
  Exponent: 65537 (0x10001)
  Modulus=C2636AE5C3D8E43FFB97AB09028F1AAC6C0BF6CD3D70EBCA281BFFE97FBE30DD
  writing RSA key
  -----BEGIN PUBLIC KEY-----
  MDwwDQYJKoZIhvcNAQEBBQADKwAwKAIhAMJjauXD2OQ/+5erCQKPGqxsC/bNPXDr
  yigb/+l/vjDdAgMBAAE=
  -----END PUBLIC KEY-----
  ```

  ![11](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/11.png)

  ```
  N = 87924348264132406875276140514499937145050893665602592992418171647042491658461
  E = 65537
  P = 275127860351348928173285174381581152299
  Q = 319576316814478949870590164193048041239
  d = 10866948760844599168252082612378495977388271279679231539839049698621994994673
  ```

  ```
  import math
  import sys
  from Crypto.PublicKey import RSA
  
  keypair = RSA.generate(1024)
  
  keypair.p = 275127860351348928173285174381581152299
  keypair.q = 319576316814478949870590164193048041239
  keypair.e = 65537
  
  keypair.n = keypair.p * keypair.q
  Qn = long((keypair.p-1) * (keypair.q-1))
  
  i = 1
  while (True):
      x = (Qn * i ) + 1
      if (x % keypair.e == 0):
          keypair.d = x / keypair.e
          break
      i += 1
  
  private = open('private.pem','w')
  private.write(keypair.exportKey())
  private.close()
  ```

  得到私钥，解密得到flag

  `PCTF{256b_i5_m3dium}`

* #### Extremely hard RSA

   e = 3

   小公钥指数攻击

   ```python
   import gmpy
   import rsa
   from rsa import transform,core
   
   c = open('flag.enc','rb').read()
   c = transform.bytes2int(c)
   n = 721059527572145959497866070657244746540818298735241721382435892767279354577831824618770455583435147844630635953460258329387406192598509097375098935299515255208445013180388186216473913754107215551156731413550416051385656895153798495423962750773689964815342291306243827028882267935999927349370340823239030087548468521168519725061290069094595524921012137038227208900579645041589141405674545883465785472925889948455146449614776287566375730215127615312001651111977914327170496695481547965108836595145998046638495232893568434202438172004892803105333017726958632541897741726563336871452837359564555756166187509015523771005760534037559648199915268764998183410394036820824721644946933656264441126738697663216138624571035323231711566263476403936148535644088575960271071967700560360448191493328793704136810376879662623765917690163480410089565377528947433177653458111431603202302962218312038109342064899388130688144810901340648989107010954279327738671710906115976561154622625847780945535284376248111949506936128229494332806622251145622565895781480383025403043645862516504771643210000415216199272423542871886181906457361118669629044165861299560814450960273479900717138570739601887771447529543568822851100841225147694940195217298482866496536787241L
   print(hex(c))
   def dec(c,N):
   	i=0
   	#i = 118719488
   	while 1:
   		print(i)
   		if(gmpy.root(c+i*N, 3)[1]==1):
   			print(gmpy.root(c+i*N, 3))
   			break
   		i=i+1
   dec(c,n)
   
   m = 440721643740967258786371951429849843897639673893942371730874939742481383302887786063966117819631425015196093856646526738786745933078032806737504580146717737115929461581126895844008044713461807791172016433647699394456368658396746134702627548155069403689581548233891848149612485605022294307233116137509171389596747894529765156771462793389236431942344003532140158865426896855377113878133478689191912682550117563858186
   
   m = transform.int2bytes(m)
   print(m)
   #PCTF{Sm4ll_3xpon3nt_i5_W3ak}
   ```

* #### very hard RSA

   共模攻击

   ```python
   #-*- coding: UTF-8 -*-
   import gmpy2
   import rsa
   from rsa import transform,core
   
   n = 0x00b0bee5e3e9e5a7e8d00b493355c618fc8c7d7d03b82e409951c182f398dee3104580e7ba70d383ae5311475656e8a964d380cb157f48c951adfa65db0b122ca40e42fa709189b719a4f0d746e2f6069baf11cebd650f14b93c977352fd13b1eea6d6e1da775502abff89d3a8b3615fd0db49b88a976bc20568489284e181f6f11e270891c8ef80017bad238e363039a458470f1749101bc29949d3a4f4038d463938851579c7525a69984f15b5667f34209b70eb261136947fa123e549dfff00601883afd936fe411e006e4e93d1a00b0fea541bbfc8c5186cb6220503a94b2413110d640c77ea54ba3220fc8f4cc6ce77151e29b3e06578c478bd1bebe04589ef9a197f6f806db8b3ecd826cad24f5324ccdec6e8fead2c2150068602c8dcdc59402ccac9424b790048ccdd9327068095efa010b7f196c74ba8c37b128f9e1411751633f78b7b9e56f71f77a1b4daad3fc54b5e7ef935d9a72fb176759765522b4bbc02e314d5c06b64d5054b7b096c601236e6ccf45b5e611c805d335dbab0c35d226cc208d8ce4736ba39a0354426fae006c7fe52d5267dcfb9c3884f51fddfdf4a9794bcfe0e1557113749e6c8ef421dba263aff68739ce00ed80fd0022ef92d3488f76deb62bdef7bea6026f22a1d25aa2a92d124414a8021fe0c174b9803e6bb5fad75e186a946a17280770f1243f4387446ccceb2222a965cc30b3929
   e1 = 17
   e2 = 65537
   s = gmpy2.gcdext(e1,e2)
   s1 = s[1]
   s2 = -s[2]
   file1 = open("flag.enc1" ,'rb').read()
   c1 = transform.bytes2int(file1)
   file2 = open("flag.enc2" ,'rb').read()
   c2 = transform.bytes2int(file2)
   c2 = gmpy2.invert(c2, n)#c2的s2次幂等于c2的模反元素的-s2次幂。
   m = (pow(c1,s1,n) * pow(c2,s2,n)) % n
   m = transform.int2bytes(m)
   print(m)
   #PCTF{M4st3r_oF_Number_Th3ory}
   ```

* #### [61dctf]rsappend

   共模攻击

   ```python
   #-*- coding: UTF-8 -*-
   import gmpy2
   import rsa
   from rsa import transform,core
   
   n = 130129008900473203968454456805638875182255844172836031362469765750555629223299054613072677100571707156698316733582683118539756860001556017029333867329591302318262912728008327902112481960175532302595162289611406978353816368008691640641366763939266242207191229240305820321249712345088877729541037319788659353057396178127928848886417880913823432701577855911982710310391664759040416918636673098245499680559140960154217578440590540485803953844560093151975252604098243460784073934982164384904788470380402066708313893480356219937915540825156266934523595689350157227336528136089157698775968997579723271988825396396444999743016035145444220925369592263295741687879468786947998534483539986779457827253891091252408156073413533385415338751818544323853074296042153599429749378847870780593975579477549218822682233583377677693108437331184962345568217859524495625257015837972947971787321584159575618388588687948368216479955807108888453821700067186732627409832722329355336479016104249514839541606562090752437124270651936485389358065775555250883907067083447197860848471728871909151915883316674512739238840179296263390441457949281128267215916340163366686542160467601357340644950755337706786366316621293666173843528346692669268972961669116101104865152273L
   
   e1 = 3493354673
   e2 = 340864687
   s = gmpy2.gcdext(e1,e2)
   s1 = s[1]
   s2 = -s[2]
   c1 = 95302089605615051645253770338205531172677353498946580682786822045513597212422921981567826452072575982096979591435896082106066368909398510427324124083956090397824543655853708684901332136907086372208856759943292176759073194584568350898675282883285945088425893961769183074018286761903249180704401358403273776903672507958464947244563165564687651203497198317095965140433811056890812018746508121991041040929574993486548175817290824525606551459788553765629416110310419007396912225733599205599864440826319234419035248234403040065378375700430311931418759746223148198205862641252459687694589780856855757703678024583642215076094232444641853081607984934672271461513190437757388818064739151861157236855430066735235471068167602037785718403200529481153399754491247323829122718485697100562237822159608309949585990842201041193231738706398444530233533281604482892716292766323711237917277799500317596333142843576977429802405873159965636003943698854699972663575602383960580472190300576561953143218321528070200681456278974433060654128626428761278953024384187213765974659768657721533448706022075747036347982370028705538843276631102928500802573434484354218539824751579164697748967608238067706842975984077663380114254296902060435479795741671231918448537178
   
   c2 = 105918022590727868761989308441554006325741233318901416621101439141134508212362387984949614887131575960019253866892976283979646611794365370050551871112439674346802340152058463892106629344277362169322187627579360245792142005899616101515519718660483000821415412306495286717542069436530262341500852884860324349096274655178057271529986597578695272732947460673640986877589225588415523871081101162696385279491410034057376225511693022693861779342120101749193614060384925056132593068290214170342896671210026723193650534803792328917982049779674425511275821311773130342656939142955431868128759911406827872932920704284125816103225607727270365652734742083302757644298457617564597237089509337896240249999242834787525341715546373108420197569425092674224333823552432226153066667988737348643469923827028254712179077001007265954488404167147591307425224250970874724864947175449960116685682348915647317191880538777148647712260093008241728509225817352093441924045801257015598517963598799676359095235231066752986688784477694024390356904157694178691411759003004184256950519184836209393583431328640341243629167223114681734264945594931213193459079614652400888215324779908031661350565230685273639666615465296133672907093946148188967451042301308884510424218096
   
   c2 = gmpy2.invert(c2, n)
   m = (pow(c1,s1,n) * pow(c2,s2,n)) % n
   m = transform.int2bytes(m)
   print(m)
   #flag{we_do_ctf_together_for_fun}
   ```

   

* #### hard RSA

   e = 2，Rabin算法

   ```python
   # coding=utf-8
   import gmpy2
   import string
   from rsa import transform,core
   n = 87924348264132406875276140514499937145050893665602592992418171647042491658461L
   e = 2
   p = 275127860351348928173285174381581152299
   q = 319576316814478949870590164193048041239
   c = open("flag.enc" ,'rb').read()
   c = transform.bytes2int(c)
   # 计算yp和yq
   inv_p = gmpy2.invert(p, q)
   inv_q = gmpy2.invert(q, p)
   
   # 计算mp和mq
   mp = pow(c, (p + 1) / 4, p)
   mq = pow(c, (q + 1) / 4, q)
   
   # 计算a,b,c,d
   a = (inv_p * p * mq + inv_q * q * mp) % n
   b = n - int(a)
   c = (inv_p * p * mq - inv_q * q * mp) % n
   d = n - int(c)
   
   for i in (a, b, c, d):
       print(transform.int2bytes(i))
   #PCTF{sp3ci4l_rsa}
   ```

* #### God Like RSA

   私钥修复

   - given a candidate for `(p mod 16**(t - 1))`, generate all possible candidates for `(p mod 16**t)` (check against mask for prime1)
   - calculate `q = n * invmod(p, 16**t)` (and check against mask for prime2)
   - calculate `d = invmod(e, 16**t) * (1 + k * (N - p - q + 1))` (and check against mask for private exponent)
   - calculate `d_p = invmod(e, 16**t) * (1 + k_p * (p - 1))` (and check against mask for exponent1)
   - calculate `d_q = invmod(e, 16**t) * (1 + k_q * (q - 1))` (and check against mask for exponent2)
   - if any of checks failed - check next candidate

   找个脚本改一下

   ```python
   #!/usr/bin/python
   #-*- coding:utf-8 -*-
   
   import re
   import pickle
   from itertools import product
   from libnum import invmod, gcd
   
   def solve_linear(a, b, mod):
       if a & 1 == 0 or b & 1 == 0:
           return None
       return (b * invmod(a, mod)) & (mod - 1)  # hack for mod = power of 2
   
   
   def to_n(s):
       s = re.sub(r"[^0-9a-f]", "", s)
       return int(s, 16)
   
   
   def msk(s):
       cleaned = "".join(map(lambda x: x[-2:], s.split(":")))
       return msk_ranges(cleaned), msk_mask(cleaned), msk_val(cleaned)
   
   
   def msk_ranges(s):
       return [range(16) if c == " " else [int(c, 16)] for c in s]
   
   
   def msk_mask(s):
       return int("".join("0" if c == " " else "f" for c in s), 16)
   
   
   def msk_val(s):
       return int("".join("0" if c == " " else c for c in s), 16)
   
   
   E = 0x10001
   
   N = to_n("""00:db:fa:bd:b1:49:5d:32:76:e7:62:6b:84:79:6e:
       9f:c2:0f:a1:3c:17:44:f1:0c:8c:3f:3e:3c:2c:60:
       40:c2:e7:f3:13:df:a3:d1:fe:10:d1:ae:57:7c:fe:
       ab:74:52:aa:53:10:2e:ef:7b:e0:09:9c:02:25:60:
       e5:7a:5c:30:d5:09:40:64:2d:1b:09:7d:d2:10:9a:
       e0:2f:2d:cf:f8:19:8c:d5:a3:95:fc:ac:42:66:10:
       78:48:b9:dd:63:c3:87:d2:53:8e:50:41:53:43:04:
       20:33:ea:09:c0:84:15:5e:65:2b:0f:06:23:40:d5:
       d4:71:7a:40:2a:9d:80:6a:6b""")
   
   p_ranges, pmask_msk, pmask_val = msk("""00:  :6 : 1:1 :  :b :0 : 2:c : b:2 :  : a:1 :
       c :  : 0:  :28:0 :  :cd:  : 8:  :  :20: c:  :
         : 5:  :9 : c:3 :  :  : a:b :c :3 :  :  :  :
        f:  :  : f: 1: 1:b :  : c:f : a:  :a :  :  :
        a:38:  :6 :  """)
   
   q_ranges, qmask_msk, qmask_val = msk("""00:e :  :d :2 :6 : 7:  :33:  :46:  : 4:  :  :
         :5 :  : 4:6 :  : 6:  : e:d :  :  : 9: e:1 :
         :  :  :  :  :0 :  :  :  :c : 5:  :  :a :0 :
       6 :  :  :8 :e9:f : f:7 :5 : e:1 :  :  : 1:9 :
        4:d :e9: 6:  """)
   
   _, dmask_msk, dmask_val = msk(""" f:  :  : a: a:9 :e :  : 1: 2:  :  :e :  :1 :
   3 : 1:  :  :  : a:  :2 :  :  :  :  :  :  :  :
   9 : a:  :  :  :  :  : 5:c1: 0:b : 3: 2:0 :b0:
     :c : f:  :f :  :d2:  :  : d:  :1 :  :3 :  :
     :  :  :0 : 3:  :  : 5:c :  :3 :6 :  :a4:  :
   4 :  :  :8f:  :  :  :  : a:  : c:5f: 7: 6:  :
    1:  : b:  : 5:  :84:0 :b : f: 3:  :  : 4: 6:
     :  : 5:1 :  :d :  : f:  : c:  :  : 5:  :  :
     :e :f4:b :4 :8e:  :  """)
   
   _, dpmask_msk, dpmask_val = msk(""" 9:d : 5:  :c :67:  : 9:  :  :  : d:  :  : 3:
    f:6 : 0:c :  :6 :ad:  :2 :d :d :  :  :0 :7 :
     :5 : 6:  : 5:1 :f : d:  : 2:  :  : 2: 3:  :
   9 :  :  :  :  :67: 3:  :4 : 7:c0: 4:b :c :f :
     :3 :b : 1""")
   
   _, dqmask_msk, dqmask_val = msk("""1 : 9:47:8 :  :  :  : 3:  :  :  :6 :  :  :0 :
   e :e :8 :  :  :  :  : 1:c :74:  :  :d : 9:3 :
   5 : e:  : 2:  :7 : 2:c :  :  :  :  :5 :  : 8:
     :  :c :  : 1:  :a :  : 9: 5:  : 3:  : e:c :
     :  : 6:  """)
   
   
   def search(K, Kp, Kq, check_level, break_step):
       max_step = 0
       cands = [0]
       for step in range(1, break_step + 1):
           #print " ", step, "( max =", max_step, ")"
           max_step = max(step, max_step)
   
           mod = 1 << (4 * step)
           mask = mod - 1
   
           cands_next = []
           for p, new_digit in product(cands, p_ranges[-step]):
               pval = (new_digit << ((step - 1) * 4)) | p
   
               if check_level >= 1:
                   qval = solve_linear(pval, N & mask, mod)
                   if qval is None or not check_val(qval, mask, qmask_msk, qmask_val):
                       continue
   
               if check_level >= 2:
                   val = solve_linear(E, 1 + K * (N - pval - qval + 1), mod)
                   if val is None or not check_val(val, mask, dmask_msk, dmask_val):
                       continue
   
               if check_level >= 3:
                   val = solve_linear(E, 1 + Kp * (pval - 1), mod)
                   if val is None or not check_val(val, mask, dpmask_msk, dpmask_val):
                       continue
   
               if check_level >= 4:
                   val = solve_linear(E, 1 + Kq * (qval - 1), mod)
                   if val is None or not check_val(val, mask, dqmask_msk, dqmask_val):
                       continue
   
                   if pval * qval == N:
                       print "Kq =", Kq
                       print "pwned"
                       print "p =", pval
                       print "q =", qval
                       p = pval
                       q = qval
                       d = invmod(E, (p - 1) * (q - 1))
                       coef = invmod(p, q)
   
                       from Crypto.PublicKey import RSA
                       print RSA.construct(map(long, (N, E, d, p, q, coef))).exportKey()
                       quit()
   
               cands_next.append(pval)
   
           if not cands_next:
               return False
           cands = cands_next
       return True
   
   
   
   def check_val(val, mask, mask_msk, mask_val):
       test_mask = mask_msk & mask
       test_val = mask_val & mask
       return val & test_mask == test_val
   
   
   for K in range(1, E):
       if K % 100 == 0:
           print "checking", K
       if search(K, 0, 0, check_level=2, break_step=20):
           print "K =", K
           break
   
   for Kp in range(1, E):
       if Kp % 1000 == 0:
           print "checking", Kp
       if search(K, Kp, 0, check_level=3, break_step=30):
           print "Kp =", Kp
           break
   
   for Kq in range(1, E):
       if Kq % 100 == 0:
           print "checking", Kq
       if search(K, Kp, Kq, check_level=4, break_step=9999):
           print "Kq =", Kq
           break
   
   ```

   ```
   -----BEGIN RSA PRIVATE KEY-----
   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
   -----END RSA PRIVATE KEY-----
   ```

   

* #### bbencode 

  ```python
  def str2num(s):
      return int(s.encode('hex'), 16)
  def bbencode(n):
      a = 0
      for i in bin(n)[2:]:
          a = a << 1
          if (int(i)):
              a = a ^ n
          if a >> 256:
              a = a ^ 0x10000000000000000000000000000000000000000000000000000000000000223L
      return a
  
  flag = 61406787709715709430385495960238216763226399960658358000016620560764164045692
  for i in range(10000000):
      flag = bbencode(flag)
      if '666c6167' == str(hex(flag))[2:10]:
          print (hex(flag)[2:-1].decode('hex'))   
  #result:61406787709715709430385495960238216763226399960658358000016620560764164045692
  ```

  密文继续加密就能出flag

* #### vigenere 

  出自TWCTF2016，Blue-Whale OJ上也收了这题，我寻思根据base64解密后全部为可打印字符来确定每一位的密钥，但是密钥长度不知道，于是就开始手动爆破(mdzz)

  爆破过程就不发上来了

  大佬的wp：http://73spica.tech/blog/tw_mma_ctf_2016_vigenere-cipher/

* #### **BrokenPic**  

  ```
  42 4D 38 0C 30 00 00 00 00 00 36 00 00 00 28 00 00 00 56 05 00 00 00 03 00 00 01 00 18 00 00 00 00 00 02 0C 30 00 12 0B 00 00 12 0B 00 00 00 00 00 00 00 00 00 00
  ```

  加上bmp头

  ![25](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/25.png)

  去掉bmp头然后AES解密，key是PHRACK-BROKENPIC

  ```python
  from Crypto.Cipher import AES
  key = 'PHRACK-BROKENPIC'
  aes = AES.new(key)
   
  with open('brokenpic.bmp', 'r') as f:
      data = f.read()
      pic = aes.decrypt(data)
   
  with open('2.bmp', 'w') as f:
      f.write(pic)
  ```

  ![26](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/26.png)

* #### 影之密码 

  密文`8842101220480224404014224202480122 `

  flag是8位大写字母

  有7个0正好分成八段，只有1248四个数字，应该和2^n有点关系

  查了一波和2^n有关系的加密方法，有一种叫做二进制幂数加密法

  任意的十进制数都可以用2^n或2^n+2^m+……的形式表示出来，可表示的单元数=2^(n+1)-1，只要2的0、1、2、3、4次幂就可以表示31个单元。通过用二进制幂数表示字母序号数来加密 

  `23 5 12 12 4 15 14 5`

  `WELLDONE `

* #### 简单ECC概念

   ```python
   M = 15424654874903
   a = 16546484
   G = (6478678675, 5636379357093)
   K = (0, 0)
   for i in range(546768):
       if K == (0, 0):
           K = G
           continue
       x1, y1 = K
       x2, y2 = G
       if K != G:
           p = (y2 - y1) * pow(x2 - x1, M - 2, M)
       else:
           p = (3 * x1 * x1 + a) * pow(2 * y1, M - 2, M)
       x3 = p * p - x1 - x2
       y3 = p * (x1 - x3) - y1
       K = (x3 % M, y3 % M)
   print(K[0]+K[1])
   #XUSTCTF{19477226185390}
   ```

* #### 神秘的压缩包

   CRC32爆破

   ```python
   #crc32_util.py
   # -*- coding: utf-8 -*-
   
   import itertools
   import binascii
   import string
   
   
   class crc32_reverse_class(object):
       def __init__(self, crc32, length, tbl=string.printable,
                    poly=0xEDB88320, accum=0):
           self.char_set = set(map(ord, tbl))
           self.crc32 = crc32
           self.length = length
           self.poly = poly
           self.accum = accum
           self.table = []
           self.table_reverse = []
   
       def init_tables(self, poly, reverse=True):
           # build CRC32 table
           for i in range(256):
               for j in range(8):
                   if i & 1:
                       i >>= 1
                       i ^= poly
                   else:
                       i >>= 1
               self.table.append(i)
           assert len(self.table) == 256, "table is wrong size"
           # build reverse table
           if reverse:
               found_none = set()
               found_multiple = set()
               for i in range(256):
                   found = []
                   for j in range(256):
                       if self.table[j] >> 24 == i:
                           found.append(j)
                   self.table_reverse.append(tuple(found))
                   if not found:
                       found_none.add(i)
                   elif len(found) > 1:
                       found_multiple.add(i)
               assert len(self.table_reverse) == 256, "reverse table is wrong size"
   
       def rangess(self, i):
           return ', '.join(map(lambda x: '[{0},{1}]'.format(*x), self.ranges(i)))
   
       def ranges(self, i):
           for kg in itertools.groupby(enumerate(i), lambda x: x[1] - x[0]):
               g = list(kg[1])
               yield g[0][1], g[-1][1]
   
       def calc(self, data, accum=0):
           accum = ~accum
           for b in data:
               accum = self.table[(accum ^ b) & 0xFF] ^ (
                   (accum >> 8) & 0x00FFFFFF)
           accum = ~accum
           return accum & 0xFFFFFFFF
   
       def findReverse(self, desired, accum):
           solutions = set()
           accum = ~accum
           stack = [(~desired,)]
           while stack:
               node = stack.pop()
               for j in self.table_reverse[(node[0] >> 24) & 0xFF]:
                   if len(node) == 4:
                       a = accum
                       data = []
                       node = node[1:] + (j,)
                       for i in range(3, -1, -1):
                           data.append((a ^ node[i]) & 0xFF)
                           a >>= 8
                           a ^= self.table[node[i]]
                       solutions.add(tuple(data))
                   else:
                       stack.append(((node[0] ^ self.table[j]) << 8,) + node[1:] + (j,))
           return solutions
   
       def dfs(self, length, outlist=['']):
           tmp_list = []
           if length == 0:
               return outlist
           for list_item in outlist:
               tmp_list.extend([list_item + chr(x) for x in self.char_set])
           return self.dfs(length - 1, tmp_list)
   
       def run_reverse(self):
           self.init_tables(self.poly)
           desired = self.crc32
           accum = self.accum
           if self.length >= 4:
               patches = self.findReverse(desired, accum)
               for patch in patches:
                   checksum = self.calc(patch, accum)
                   print 'verification checksum: 0x{0:08x} ({1})'.format(
                       checksum, 'OK' if checksum == desired else 'ERROR')
               for item in self.dfs(self.length - 4):
                   patch = map(ord, item)
                   patches = self.findReverse(desired, self.calc(patch, accum))
                   for last_4_bytes in patches:
                       if all(p in self.char_set for p in last_4_bytes):
                           patch.extend(last_4_bytes)
                           print '[find]: {1} ({0})'.format(
                               'OK' if self.calc(patch, accum) == desired else 'ERROR', ''.join(map(chr, patch)))
           else:
               for item in self.dfs(self.length):
                   if crc32(item) == desired:
                       print '[find]: {0} (OK)'.format(item)
   
   
   def crc32_reverse(crc32, length, char_set=string.printable,
                     poly=0xEDB88320, accum=0):
       obj = crc32_reverse_class(crc32, length, char_set, poly, accum)
       obj.run_reverse()
   
   
   def crc32(s):
       return binascii.crc32(s) & 0xffffffff
   ```

   ```python
   from crc32_util import *
   crc = [0x20AE9F17,
        0xD2D0067E,
        0x6C53518D,
        0x80DF4DC3,
        0x3F637A50,
        0xBCD9703B]
   for i in crc:
       crc32_reverse(i, 5)
   ```

   ```
   verification checksum: 0x20ae9f17 (OK)
   [find]: l./rc (OK)
   [find]: passw (OK)
   verification checksum: 0xd2d0067e (OK)
   [find]: "_YWn (OK)
   [find]: N,tS* (OK)
   [find]: Rc(R> (OK)
   [find]: ord:f (OK)
   [find]: s=8;r (OK)
   verification checksum: 0x6c53518d (OK)
   [find]: /8LWp (OK)
   [find]: CKaS4 (OK)
   [find]: ~Z-;l (OK)
   verification checksum: 0x80df4dc3 (OK)
   [find]: apEwF (OK)
   verification checksum: 0x3f637a50 (OK)
   ^Q6w (OK)
   [find]: \<0Zk (OK)
   [find]: a-|23 (OK)
   [find]: }b 3' (OK)
   verification checksum: 0xbcd9703b (OK)
   [find]: hyAo5 (OK)
   
   password:f~Z-;lapEwF\<0ZkhyAo5
   XUSTCTF{6ebd0342caa3cf39981b98ee24a1f0ac}
   ```

* #### 好多盐

   爆破就完事了

   ```python
   import hashlib
   password = '''f09ebdb2bb9f5eb4fbd12aad96e1e929 p5Zg6LtD
   6cea25448314ddb70d98708553fc0928 ZwbWnG0j
   2629906b029983a7c524114c2dd9cc36 1JE25XOn
   2e854eb55586dc58e6758cfed62dd865 ICKTxe5j
   7b073411ee21fcaf177972c1a644f403 0wdRCo1W
   6795d1be7c63f30935273d9eb32c73e3 EuMN5GaH
   d10f5340214309e3cfc00bbc7a2fa718 aOrND9AB
   8e0dc02301debcc965ee04c7f5b5188b uQg6JMcx
   4fec71840818d02f0603440466a892c9 XY5QnHmU
   ee8f46142f3b5d973a01079f7b47e81c zMVNlHOr
   e4d9e1e85f3880aedb7264054acd1896 TqRhn1Yp
   0fd046d8ecddefc66203f6539cac486b AR5lI2He
   f6326f02adaa31a66ed06ceab2948d01 Aax2fIPl
   720ba10d446a337d79f1da8926835a49 ZAOYDPR2
   06af8bcc454229fe5ca09567a9071e62 hvcECKYs
   79f58ca7a81ae2775c2c2b73beff8644 TgFacoR3
   46aaa5a7fef5e250a2448a8d1257e9cf GLYu0NO4
   2149ac87790dd0fe1b43f40d527e425a 5Xk2O1sG
   d15a36d8be574ac8fe64689c728c268e aZikhUEy
   ff7bced91bd9067834e3ad14cc1464cd E7UROqXn
   8cc0437187caf10e5eda345cb6296252 XPin3mVB
   5cfcdca4a9cb2985a0b688406617689e nsGqoafv
   5a7dfa8bc7b5dfbb914c0a78ab2760c6 YC1qZUFR
   8061d8f222167fcc66569f6261ddd3cc wNgQi615
   3d8a02528c949df7405f0b48afe4a626 CO2NMusb
   70651acbc8bd027529bbcccdbf3b0f14 CAXVjFMd
   a9dbe70e83596f2d9210970236bdd535 TL6sjEuK
   9ed6ef5780f705ade6845b9ef349eb8f tJ90ibsz
   4b46fac0c41b0c6244523612a6c7ac4a VTjOSNmw
   8141e6ecb4f803426d1db8fbeb5686ef lh75cdNC
   df803949fd13f5f7d7dd8457a673104b V39sEvYX
   19052cc5ef69f90094753c2b3bbcd41d YwoGExpg
   cf8591bdccfaa0cdca652f1d31dbd70f pJCLui49
   66e10e3d4a788c335282f42b92c760a1 NQCZoIhj
   94c3ae5bcc04c38053106916f9b99bda vOktelLQ
   e67e88646758e465697c15b1ef164a8d x0hwJGHj
   84d3d828e1a0c14b5b095bedc23269fb 2HVWe9fM
   264a9e831c3401c38021ba3844479c3f Cx4og6IW
   ed0343dec184d9d2c30a9b9c1c308356 g2rqmPkT
   ad5ba8dc801c37037350578630783d80 pFK2JDT5
   3f588bedb704da9448e68fe81e42bca6 4ANDOiau
   970c9cf3cad3dfa7926f53ccaae89421 R6ML7Qy8
   e0a097b7cceaa7a8949fe039884e4a2d dul2ynqL
   7df505218102c64b1fe4fa5981ddb6fa jPeoyS57
   fd4f6043da1f7d5dca993c946ef6cd7c 6p9CwGaY
   5fe6d99b9a2824949279187c246c9c30 OGQ2J57y
   135b150ad513a961089bb1c05085a3d9 h0dw1Fro
   ad6af4fb623b3c51181a371911667fed HbQT4dRz
   c9fa4b0db317d88e2b10060225e92494 ebVnpMzS
   d0deab17d115bd6fdce8592bb3667643 bL5zwgvX
   006f0cb3a422716692f143f28eb0d187 NHXg1Fof
   ddc125de34da1a6ec0cbe401f147bc8f GDai9Y0n
   be5052053c5a806e8f56ed64e0d67821 40alyH3w
   aaf18ac446b8c385c4112c10ae87e7dc ZJQzuIL0
   a2db20a4b7386dc2d8c30bf9a05ceef7 QnpOlPWH
   8a4fbc32a3251bb51072d51969ba5d33 rtcbipeq
   5e35d2c9675ed811880cea01f268e00f i1Hbne6h
   9da23007699e832f4e9344057c5e0bd3 EtbGpMSW
   f09233683d05171420f963fc92764e84 fxHoinEe
   4feabf309c5872f3cca7295b3577f2a8 KymkJXqA
   9b94da2fa9402a3fdb4ff15b9f3ba4d2 G3Tdr1Pg
   b3cd8d6b53702d733ba515dec1d770c5 Y71LJWZz
   6a5b3b2526bb7e94209c487585034534 rIwb4oxt
   e9728ef776144c25ba0155a0faab2526 e1sOXSb8
   d41a5e7a98e28d76dbd183df7e3bcb49 36bedvia
   81d5ebfea6aff129cf515d4e0e5f8360 dDG4qTjW'''
   
   password = password.split('\n')
   for i in password:
       print(i)
       pass_md5 = i.split(' ')[0]
       salt = i.split(' ')[1]
       for j in range(10000000000):
           #j = 1234567890
           md5 = hashlib.md5()
           md5.update('{FLAG:' + str(j).zfill(10) + '}' + salt)
           if md5.hexdigest() == pass_md5:
               print(j)
               break
   ```

* #### superexpress 

   ```python
   import sys
   key = '****CENSORED***************'
   flag = 'TWCTF{*******CENSORED********}'
   
   if len(key) % 2 == 1:
       print("Key Length Error")
       sys.exit(1)
   
   n = len(key) / 2
   encrypted = ''
   for c in flag:
       c = ord(c)
       for a, b in zip(key[0:n], key[n:2*n]):
           c = (ord(a) * c + ord(b)) % 251
       encrypted += '%02x' % c
   
   print encrypted
   ```

   若key=xyab

   zip后是[(x,a),(y,b)]
   加密操作为enc = (xyc + ay + b)%251

   所以无论key的长度为多少，加密的结果一定是(Ac + B)%251  A B小于251

   根据TWCTF爆破出A B，然后解密 

   ```python
   import string
   def find_key():
       for a in range(251):
           for b in range(251):
               if (ord("T") * a + b) % 251 == int_enc[0] and (ord("W") * a + b) % 251 == int_enc[1] and (ord("C") * a + b) % 251 == int_enc[2]:
               	return a, b
   enc = "805eed80cbbccb94c36413275780ec94a857dfec8da8ca94a8c313a8ccf9"
   int_enc = []
   chars = string.printable
   flag = ''
   for i in range(0, len(enc), 2):
       int_enc += [int(enc[i:i + 2], 16)]
   
   a, b = find_key()
   for enc in int_enc:
       for i in chars:
           if (ord(i) * a + b) % 251 == enc:
               flag += chr(ord(i))
   print(flag)
   #TWCTF{Faster_Than_Shinkansen!}
   ```

* #### Cry

   flag先AES再base64

   给出rsa加密后的key和iv

   给出p的高位

   Factoring with High Bits Known

   ```python
   import binascii
   n = 0x229c6c7ad1db1d76783f0bd9aabdab0af749ec4a57af53eb151d49bd3df89d083763c62cfc4d72e627c8c7d69672324fee3c6038ebf1fc576c15ee3620c95ddbaa04f39d4de32629709ac2ad55972680ab7b147e14bfae37773c462b3c6dd1a8d567ca4ae338d90337e583aaea5ec853a89083362e5cc10ca5e344870b0362897eefd508ea19378f4f6e8ef7aec87a3c226bd94a78a0f730c2ecb6aa9aef85eb3a4e3dc498f9471e538fdcbc0b4eee3075b2241072c1fe7bb004a1ae297af4d0e6a4d3c3fbadf6105afd0c4a97d9f96b80b27dc002f3bae6c3cbd0c3ede67135c30ff5a5cebbb7caae4e1b25ea3cba7eb12526d4b922a3d90edcd3ca73a54a83
   e2 = 0x10001
   pbits = 1024
   p4 = 0xd0e66845c611ec90ee05dc7667cac74cb06042083944ca017fdccf5abe9c25565d36144b1c1c7dad8fde1e0ee778d6d5ed12b57b4c9adf2c0a466aa7278a03bcc26750bcf1a333fa65490a5bd160ff51c85849dc3c45ad696e42
   kbits = pbits - p4.nbits() 
   p4 = p4 << kbits 
   PR.<x> = PolynomialRing(Zmod(n))
   f = x + p4
   roots = f.small_roots(X=2^kbits, beta=0.4) 
   if roots: 
     p = p4+int(roots[0])
     assert n % p == 0
     q = n/int(p)
     phin = (p-1)*(q-1)
     d = inverse_mod(e2,phin)
     print(d)
     cipher = 0x1889c9dd2e25036121397a86c886c794b083987f03af7ba71084203e22cb4848ddae3e74ce775b4375852c38ff4c3e62b2f22358f7aed0280b47758640627f587ef8d68472df0c354d582868aade4c0254c4e3284842340bcb8d7adea1355e0b722caa868b97832759987576f6007219d1238e0b922f3f658cda50724cc16514521bbbeffad842e2d6b77c37b92d0d5fcc7129637c8923ad7c0ebacbf8d1a873e01daffc6e8a6531a93c33b956ff0b0fdfc36dac756405910dae55e54e36760495783016d5e11ad473c703071396ec800bd1785cb1f3a1f0ee6d97f4513d2730296dbd65e9c84a7bd4d3b3e054906d93b5f58bb18fdde84ce987f845779dbe6e
     ck = pow(cipher,d,n)
     ck = hex(int(ck))[2:-1]
     print ((ck))
     cipher = 0x88cc37ab2c7193d44c8a211ea5544f2eb50912e46d24f0f8865382e6b8c411213065e00e8c0f9ca6dd98f5fa99048113e5f649c9cb5670e388ec3f9cf1b34ce378cdc3e6667b68c353c552decc3c1a1c6801af6ccc1977210cc4a123d0a5260c68ff180aa8cd83a48eb87bf253fbcede1e01e1c8873ee029876a673618a4665b59adee7c09c72e7f406f82ee2caa96de43e54b7c84800a83f720dbd54ba965dc25f284852f762328ca100f47c1149e811ce556ae7872778650dd87f6da0f7dfec8a97bc2fea14f4360e894e94dcb2b190cfa526b7a432d49617971c77c24f58416bbe6a8c5d11daf1a55c3e05a8fcb6ab04384b4efcd2b526a982ab4c6d43c8
     civ = pow(cipher,d,n)
     civ = hex(int(civ))[2:-1]
     print ((civ))
     
   '''
   5ae424d330bfd1f07954984a769a4e1d
   70eb30ba8af2eff16fe31eba5defb488
   '''
   ```

   ```python
   import binascii
   from Crypto.Cipher import AES
   flag = 'SDO8O44wLIpJegdPY0/VMkPGTtUYFpWdVImiJI42IYE='
   key = '5ae424d330bfd1f07954984a769a4e1d'
   iv = '70eb30ba8af2eff16fe31eba5defb488'
   cipher = AES.new(key.decode('hex'), AES.MODE_CBC, iv.decode('hex'))
   flag = cipher.decrypt(flag.decode('base64'))
   print(flag)
   #aaaaaflag{ok_this_is_a_flag_rsa}
   ```

* #### Complicated Crypto

   CRC32爆破

   ```python
   from crc32_util import *
   crc = [0x7C2DF918,
          0xA58A1926,
          0x4DAD5967]
   for i in crc:
       crc32_reverse(i, 6)
   ```

   然后出了一堆

   我寻思密码如果不是什么洋文句子没法做了搜一下crc

   ```
   [find]: _CRC32 (OK)
   ```

   改char_set再看一下

   ```python
   from crc32_util import *
   
   crc = [0x7C2DF918,
          0xA58A1926,
          0x4DAD5967]
   
   for i in crc:
       crc32_reverse(i, 6, char_set=string.letters + string.digits + '_')
   ```

   ```
   [find]: _i5_n0 (OK)
   [find]: t_s4f3 (OK)
   ```

   根据洋文判断，这俩是密码

   ```
   _CRC32_i5_n0t_s4f3
   ```

   解压之后是Vigenere

   给了一堆key，长度为40

   https://guballa.de/vigenere-solver试一下

   ```
   Clear text using key "yewrutewcybnhhipxoyubjjpqiraaymyoneomtsv"
   
   thegetenerecilgerisamethodofthensatingalphamagictextbutsingaseriesofschbycentcaesarnechersbasaconthelettersouumashorditisasticleformoboolyalphabetichodonttutionsoplofwordisvefenerecipherfucha
   ```

   解出来还是乱的，但是发现里面有一个word，那句话应该是password is，这个word对应的密钥位应该是准确的

   先把解出来的东西按密钥长度分组，顺手把word替换成特殊符号，分组之后找他

   ```python
   c = 'thegetenerecilgerisamethodofthensatingalphamagictextbutsingaseriesofschbycentcaesarnechersbasaconthelettersouumashorditisasticleformoboolyalphabetichodonttutionsoplof!@#$isvefenerecipherfucha'
   def devide(i):
   	cc = ''
   	while i < len(c):
   		cc += c[i]
   		i = i + 40
   	return cc
   for i in range(40):
   	print(devide(i))
   ```

   ```
   tpsss
   hhaao
   earsp
   gmntl
   eaeio
   tgccf
   eihl!
   ncee@
   etrf#
   reso$
   exbri
   ctams
   ibsov
   luabe
   gtcof
   esooe
   rinln
   intye
   sghar
   aaele
   mslpc
   eeehi
   trtap
   hitbh
   oeeee
   dsrtr
   oosif
   ffocu
   tsuhc
   hcuoh
   ehmda
   nbao
   sysn
   acht
   teot
   inru
   ntdt
   gcii
   aato
   lein
   ```

   密钥的7-10位是正确的，ewcy

   在给出的key中找

   YEWCQGEWCYBNHDHPXOYUBJJPQIRAPSOUIYEOMTSV

   ```python
   from pycipher import Vigenere
   key = 'YEWCQGEWCYBNHDHPXOYUBJJPQIRAPSOUIYEOMTSV'
   c = 'rlaxymijgpfppsotowqunncwelfftfqlgnxwzzsgnlwduzmyvcygibbhfbeutnaxuaffsatzmpibfvszqeneyvlatqcnzhkdkhfymnciuzjousyygusfpbldqeokcvpahmszviwdimyfqqjqubzchmpmbgxifbgiqslciyaktbjfclntkspydrywuzwucfm'
   print(Vigenere(key).decipher(c))
   #THEVIGENERECIPHERISAMETHODOFENCRYPTINGALPHABETICTEXTBYUSINGASERIESOFDIFFERENTCAESARCIPHERSBASEDONTHELETTERSOFAKEYWORDITISASIMPLEFORMOFPOLYALPHABETICSUBSTITUTIONSOPASSWORDISVIGENERECIPHERFUNNY
   #vigenere cipher funny
   ```

   sha1爆破

   ```python
   import hashlib
   import string
   s = "619c20c*a4de755*9be9a8b*b7cbfa5*e8b4365*"
   chars = string.printable
   for a in chars:
       for b in chars:
           for c in chars:
               for d in chars:
                   password = '%s7%s5-%s4%s3?' % (a, b, c, d)
                   enc = hashlib.sha1(password).hexdigest()
                   if enc[0:7] == s[0:7] and enc[8:15] == s[8:15] and enc[16:23] == s[16:23]:
                       print(password)
   #I7~5-s4F3?
   ```

   ```
   Hello World ;-)
   MD5校验真的安全吗？
   有没有两个不同的程序MD5却相同呢？
   如果有的话另一个程序输出是什么呢？
   解压密码为单行输出结果。
   
   Hello World ;-)
   MD5 check is really safe?
   There are two different procedures MD5 is the same?
   If so what is the output of another program?
   The decompression password is a single-line output.
   ```

   <http://www.win.tue.nl/hashclash/SoftIntCodeSign/HelloWorld-colliding.exe>  　　<http://www.win.tue.nl/hashclash/SoftIntCodeSign/GoodbyeWorld-colliding.exe> 

   wdnmd查了半天md5相同的程序，我寻思这题是用搜索引擎解密？

   password：Goodbye World :-( 

   下一步Wiener’s attack

   ```
   flag{W0rld_Of_Crypt0gr@phy}
   ```

* #### Jarvis's encryption system

   先把图片上的东西搞出来

   ```
   m1: 89372489723987498237894327984372
   c1: 792279062886162218096642776664224514933347584486280723004734021586336212749049858600481963227286459323970478541843083793725468708921717787221937249530784012084036132167698694870670989692185525559265359595824727956010042190235432643115112280623082788133230708728369892499755238276075667536752879449115011933006031581738186877618805996280847737363426887886868682686959858371130406926178828888575004380515988821399247906070333132810952695798429265793849588130806947806841034544612000197604854503195512120025729616966658790540157838337703936086683817085220432748606686965902101050255048796382841321391071407100767404596588780879740560771450534303617347553555472893929700798373187625224545676303975128589469709553887522697982505366205159178754377849727155295773459020853899833570753142832536760229326028534739725856990225488803963836214548294423502322319111713836053680359093114158912017408230992904911531693795674356749450578360594750306010644345865018135713049088702085668117922755659876667178408188245170381487842104129405699987082399408416605832498886309106565903612880735897179022046135207448286905927468981921408174446350113407999312543013150441972687118445672308468055301677455644948365453703227341347327118261153884632046860369729
   m2: flag here
   c2: 738822002752800877524466308025949155169562722946933006009883884249589602039677687891359871510923927357766748131398443497541198900771818831638644263405425815579383553019562159083788644122365536627592737115316351290153544908592280731090451811311680698586032725090719266003369555867584457372823678746133588560994163232730766388456903527206840527304843529539480355012405496730615078972755415860013097394363116913629756292725693596880188792245847698225435105827398989245800248197290718407831242734331874121327502564673597694670795036098967372950089253263743880807024448724715652660602771818683520844873803372738417012436219777372987997036211306992938395670636075660990930360358970016244484405618827909229400111542660072678812089441010001235353317911131109787281238112284352067511452432985149442969693926797740772628154057474332702139775407456229918917403138849681496015981718513476254353617586634306067889050783266988506871489696817574207289110594169371818597141857443042841485880477066344316648550850088971005108756497748568090122624591451915965314486079436499049418137147522360690326710468200339550170216543240318289067712843687012174036874897324652429812609807952220427326987655148639613323665786093803557065570465270944069296977739085
   ```

   这个程序的功能是输入e，输出d，然后输入明文，输出密文

   hint说需要爆破，那就爆破e

   根据m1和c1可以判断输入的e是否为加密flag的e，如果是就根据输出的d解flag

   ```python
   N = 808637320166213096433765975908829772554859069394497436792703828416763985949910999652518305818627321094257781267795371106923808192073932662313603219525599014635435542122940843344921727149256852355110338886574805360544004118210641173633231100848831019159519744863314748281129830905559513810272933968408858616937223539622595750248885831720830102914499513408356858587797522763592193335162884129664298938995394243273615798207065590802899685489088903478734288977143851327400816886878238915788561611104380001569848016035186213716602462262685777960742683591155978590371074585063550419528377002596163321548052257322263024813745933243795081592986850478573362522245788630785664119935566422559659277401321793012274415007906726880710258434953224297253000176721652344571059040066987969691706315602374506498087282531643212970147526356421919309049062439117990930204486012562031589114880474346559407445496718773030816258262150397230280669274725009415653773469037623986165899557423095323109994543129373149980880777219450714265152054529287453826506032747047856303879606356141420416161004589629524370677871918513405209191951229311529443558187652701599377904802383252318582028816524498306240682160249309341335405511246150908708558397938689907425750101507
   def enc(e):
   	m = 89372489723987498237894327984372
   	c = pow(m, e, N)
   	if c == 792279062886162218096642776664224514933347584486280723004734021586336212749049858600481963227286459323970478541843083793725468708921717787221937249530784012084036132167698694870670989692185525559265359595824727956010042190235432643115112280623082788133230708728369892499755238276075667536752879449115011933006031581738186877618805996280847737363426887886868682686959858371130406926178828888575004380515988821399247906070333132810952695798429265793849588130806947806841034544612000197604854503195512120025729616966658790540157838337703936086683817085220432748606686965902101050255048796382841321391071407100767404596588780879740560771450534303617347553555472893929700798373187625224545676303975128589469709553887522697982505366205159178754377849727155295773459020853899833570753142832536760229326028534739725856990225488803963836214548294423502322319111713836053680359093114158912017408230992904911531693795674356749450578360594750306010644345865018135713049088702085668117922755659876667178408188245170381487842104129405699987082399408416605832498886309106565903612880735897179022046135207448286905927468981921408174446350113407999312543013150441972687118445672308468055301677455644948365453703227341347327118261153884632046860369729:
   		print(e)
   for i in range(1000000000):
   	#7845741
   	enc(i)
   ```

   nc输入e拿到d

   ```python
   from rsa import transform
   N = 808637320166213096433765975908829772554859069394497436792703828416763985949910999652518305818627321094257781267795371106923808192073932662313603219525599014635435542122940843344921727149256852355110338886574805360544004118210641173633231100848831019159519744863314748281129830905559513810272933968408858616937223539622595750248885831720830102914499513408356858587797522763592193335162884129664298938995394243273615798207065590802899685489088903478734288977143851327400816886878238915788561611104380001569848016035186213716602462262685777960742683591155978590371074585063550419528377002596163321548052257322263024813745933243795081592986850478573362522245788630785664119935566422559659277401321793012274415007906726880710258434953224297253000176721652344571059040066987969691706315602374506498087282531643212970147526356421919309049062439117990930204486012562031589114880474346559407445496718773030816258262150397230280669274725009415653773469037623986165899557423095323109994543129373149980880777219450714265152054529287453826506032747047856303879606356141420416161004589629524370677871918513405209191951229311529443558187652701599377904802383252318582028816524498306240682160249309341335405511246150908708558397938689907425750101507
   d = 624460328909915360701402168639641282028094468418961878947574807290638891758678991143435088653980701535371225162050430031333639072505365342535152293096454464491608234713910040741806054032872119204341656042243036836513731486079394171780995659012791615808025872985542653518800484827924355387767430294123689221121329057758535673622344148467540232245253256875196052350040448617224502051601181938246394036842235391465615724331125440683500889291172253639404968619326290436776700446299000007994603663440301337649478949521483497552296109838827309290662620012954396232530653067994746179325738761837228276717554780296018709380622306541045859622715377902354561164951333797443088787938481179135106105790091663516291819506679417163921126064227724274720567814644923715786512409941352289194976639518315198223889636540907899742620702027181209723760906029440110133038012652837512150214031930229627563605747245442722777826347211476418833664939434587222857079580932195626606642019627685477852118740507133047312988551945249158637120933025247874896831420739652406553559282198158772718455703088272187706565621125165787151495212884395636011063049499722644316616892917352706474956487897928799493388647214180485284836266930315584778253034297866155428036121429
   c2 = 738822002752800877524466308025949155169562722946933006009883884249589602039677687891359871510923927357766748131398443497541198900771818831638644263405425815579383553019562159083788644122365536627592737115316351290153544908592280731090451811311680698586032725090719266003369555867584457372823678746133588560994163232730766388456903527206840527304843529539480355012405496730615078972755415860013097394363116913629756292725693596880188792245847698225435105827398989245800248197290718407831242734331874121327502564673597694670795036098967372950089253263743880807024448724715652660602771818683520844873803372738417012436219777372987997036211306992938395670636075660990930360358970016244484405618827909229400111542660072678812089441010001235353317911131109787281238112284352067511452432985149442969693926797740772628154057474332702139775407456229918917403138849681496015981718513476254353617586634306067889050783266988506871489696817574207289110594169371818597141857443042841485880477066344316648550850088971005108756497748568090122624591451915965314486079436499049418137147522360690326710468200339550170216543240318289067712843687012174036874897324652429812609807952220427326987655148639613323665786093803557065570465270944069296977739085
   m2 = pow(c2, d, N)
   print(transform.int2bytes(m2))
   #WUSTCTF{U_r_real33y_m4st3r_0f_math}
   ```

* #### DSA

   ```
   OpenSSL> sha1 -verify dsa_public.pem -signature packet1/sign1.bin  packet1/message1
   Verified OK
   OpenSSL> sha1 -verify dsa_public.pem -signature packet2/sign2.bin  packet2/message2
   Verified OK
   OpenSSL> sha1 -verify dsa_public.pem -signature packet3/sign3.bin  packet3/message3
   Verified OK
   OpenSSL> sha1 -verify dsa_public.pem -signature packet4/sign4.bin  packet4/message4
   Verified OK
   OpenSSL> asn1parse -inform der -in packet1/sign1.bin
       0:d=0  hl=2 l=  45 cons: SEQUENCE
       2:d=1  hl=2 l=  21 prim: INTEGER           :8158B477C5AA033D650596E93653C730D26BA409
      25:d=1  hl=2 l=  20 prim: INTEGER           :165B9DD1C93230C31111E5A4E6EB5181F990F702
   OpenSSL> asn1parse -inform der -in packet2/sign2.bin
       0:d=0  hl=2 l=  44 cons: SEQUENCE
       2:d=1  hl=2 l=  20 prim: INTEGER           :60B9F2A5BA689B802942D667ED5D1EED066C5A7F
      24:d=1  hl=2 l=  20 prim: INTEGER           :3DC8921BA26B514F4D991A85482750E0225A15B5
   OpenSSL> asn1parse -inform der -in packet3/sign3.bin
       0:d=0  hl=2 l=  44 cons: SEQUENCE
       2:d=1  hl=2 l=  20 prim: INTEGER           :5090DA81FEDE048D706D80E0AC47701E5A9EF1CC
      24:d=1  hl=2 l=  20 prim: INTEGER           :30EB88E6A4BFB1B16728A974210AE4E41B42677D
   OpenSSL> asn1parse -inform der -in packet4/sign4.bin
       0:d=0  hl=2 l=  44 cons: SEQUENCE
       2:d=1  hl=2 l=  20 prim: INTEGER           :5090DA81FEDE048D706D80E0AC47701E5A9EF1CC
      24:d=1  hl=2 l=  20 prim: INTEGER           :5E10DED084203CCBCEC3356A2CA02FF318FD4123
   ```

   3、4 r相同，共享了k

   $s1≡(H(m1)+xr)k−1modqs1≡(H(m1)+xr)k−1modq$

   $s2≡(H(m2)+xr)k−1modqs2≡(H(m2)+xr)k−1modq$

   $s1k≡H(m1)+xrs1k≡H(m1)+xr$

   $s2k≡H(m2)+xrs2k≡H(m2)+xr$

   $k(s1−s2)≡H(m1)−H(m2)modqk(s1−s2)≡H(m1)−H(m2)modq$

   可解出 k

   ```python
   from Crypto.PublicKey import DSA
   from hashlib import sha1
   import gmpy2
   with open('./dsa_public.pem') as f:
       key = DSA.importKey(f)
       y = key.y
       g = key.g
       p = key.p
       q = key.q
   f3 = open(r"packet3/message3", 'r')
   f4 = open(r"packet4/message4", 'r')
   data3 = f3.read()
   data4 = f4.read()
   sha = sha1()
   sha.update(data3)
   m3 = int(sha.hexdigest(), 16)
   sha = sha1()
   sha.update(data4)
   m4 = int(sha.hexdigest(), 16)
   s3 = 0x30EB88E6A4BFB1B16728A974210AE4E41B42677D
   s4 = 0x5E10DED084203CCBCEC3356A2CA02FF318FD4123
   r = 0x5090DA81FEDE048D706D80E0AC47701E5A9EF1CC
   ds = s4 - s3
   dm = m4 - m3
   k = gmpy2.mul(dm, gmpy2.invert(ds, q))
   k = gmpy2.f_mod(k, q)
   tmp = gmpy2.mul(k, s3) - m3
   x = tmp * gmpy2.invert(r, q)
   x = gmpy2.f_mod(x, q)
   print(int(x))
   #CTF{520793588153805320783422521615148687785086070744}
   ```

   

