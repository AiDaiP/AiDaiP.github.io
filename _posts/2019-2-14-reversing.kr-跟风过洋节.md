---
layout: post
title:  "跟风过洋节"
date:   2019-2-14
desc: ""
keywords: ""
categories: [CTF]
tags: [CTF,reversing.kr]
icon: icon-html
---

# reversing.kr-跟风过洋节

跟风过洋节

![1](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/1.jpg)

* #### Easy_CrackMe

  ![2](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/2.png)

  四段拼起来就完事了

  `Ea5yR3versing`

  

* #### Easy_KeygenMe

  ```
  Find the Name when the Serial is 5B134977135E7D13
  ```

  ![3](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/3.png)

  输入的name转化为16进制后每位与16，32，48异或，得到的值储存在v13，输入的serial储存在v9，v9与v13比较。

  ```python
  a = [0x5B,0x13,0x49,0x77,0x13,0x5E,0x7D,0x13]
  b = [16,32,48]
  name = ''
  for i in range(8):
  	name += chr(a[i] ^ b[i%3])
  print name
  ```

  `K3yg3nm3`

  

* #### ReversingKr UnpackMe

  ```
  Find the OEP
  ex) 00401000
  ```

  ![4](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/4.png)

  单步发现在这窗口弹出，俺寻思这就是入口点

  `000401150`

  

* #### Music_Player

  

  ```
  This MP3 Player is limited to 1 minutes.
  You have to play more than one minute.
  
  There are exist several 1-minute-check-routine.
  After bypassing every check routine, you will see the perfect flag.
  ```

  播放器只能播放一分钟，绕过这个限制就可以得到flag

  ![5](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/5.png)

  先查一波字符串，看看有没有和一分钟有关系的

  定位下断点

  ![6](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/6.png)

  59秒时断

  ![8](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/8.png)

  向上翻找跳转，这有个和0xEA60比较的，俺寻思这是600000毫秒

  把0040456B的改成jmp再跑一遍

  ![9](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/9.png)

  出现新的弹窗

  重新跑一遍，在004045B2断，然后单步，找到弹窗位置

  ![10](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/10.png)

  向上找跳转，改成jmp，再跑一遍

  ![7](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/7.png)

  `LIstenCare`

  

* #### ImagePrc

  ![11](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/11.png)

  这玩意应该是画完图之后比较

  IDA pro里也看不懂是咋比较的

  gg

  

* #### Replace

  

  ![14](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/14.png)

  ![13](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/13.png)

  定位到`Correct!`看一下

  ![15](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/15.png)

  ![16](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/16.png)

  IDA Pro里看一下，好像没法跳转到这里

  俺寻思直接把前面俩jjmp全改成nop不就完事了

  ![17](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/17.png)

  这题是要找输入的值，出个correct没啥用

  继续用OD看

  ![18](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/18.png)

  在GetDlgItemInt断，输入后步过

  ![19](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/19.png)

  在这里程序退出

  ![20](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/20.png)

  再跑一遍，进入40466F看一眼

  把eax的值对应地址的内容改成0x90，0x90是nop指令，所以eax=0x401071就行了

  输入0的时候eax=0x601605CB，输入1，eax=0x601605CC

  0x401071- ? = 0x601605CC

  显然需要eax溢出

  0x100401071 - 0x601605CC = ‭0xA02A0AA6‬

  ![21](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/21.png)

  `‭2687109798‬`

  

* #### Easy ELF

  ![22](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/22.png)

  ![23](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/23.png)

  ```c
  #include <stdio.h>
  int main()
  {
  	int a[5];
  	a[0] = 120^52;
  	a[1] = 49;	
  	a[2] = 124^50;
  	a[3] = -35^136;
  	a[4] = 88;
  	for (int i = 0; i < 5; i++)
  		printf("%c", a[i]);
  	return 0;
  }
  ```

  `L1NUX`

  

* #### CSHOP

  ![24](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/24.png)

  c#写的，用dnSpy看看

  ![25](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/25.png)

  这有个大小被设置为0的按钮，那按它一下会发生啥

  试试回车能不能按这个按钮

  ![26](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr/26.png)

  输进去试了一下还真是flag

  `P4W6RP6SES`






![15](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr-2/15.jpg)

- #### Position

  ```
  Find the Name when the Serial is 76876-77776
  This problem has several answers.
  
  Password is ***p//找了半天Password，它说的其实是Name
  ```

  ![2](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr-2/2.png)

  找出Serial对应的Name，Name四位，最后一位是p

  ![1](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr-2/1.png)

  sub_401740

  ![3](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr-2/3.png)

  读取name和serial，name每一位都是小写字母

  ![4](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr-2/4.png)

  ![5](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr-2/5.png)

  name进行位运算，其中两次的运算结果相加储存在v52，该结果与serial对应位比较

  ```python
  serial = '76876-77776'
  for i in range(ord('a'), ord('z') + 1):
      for j in range(ord('a'), ord('z') + 1):
          v7 = (i & 1) + 5
          v48 = ((i >> 4) & 1) + 5
          v42 = ((i >> 1) & 1) + 5
          v44 = ((i >> 2) & 1) + 5
          v46 = ((i >> 3) & 1) + 5
  
          v34 = (j & 1) + 1
          v40 = ((j >> 4) & 1) + 1
          v36 = ((j >> 1) & 1) + 1
          v9 = ((j >> 2) & 1) + 1
          v38 = ((j >> 3) & 1) + 1
  
          if (v7 + v9 == int(serial[0]) and v46 + v38 == int(serial[1]) and v42 + v40 == int(serial[2]) and v44 + v34 == int(serial[3]) and v48 + v36 == int(serial[4])):
              print(chr(i), chr(j))
              break
  
  for i in range(ord('a'), ord('z') + 1):
      v21 = (i & 1) + 5
      v49 = ((i >> 4) & 1) + 5
      v43 = ((i >> 1) & 1) + 5
      v45 = ((i >> 2) & 1) + 5
      v47 = ((i >> 3) & 1) + 5
  
      v35 = (ord('p') & 1) + 1
      v41 = ((ord('p') >> 4) & 1) + 1
      v37 = ((ord('p') >> 1) & 1) + 1
      v23 = ((ord('p') >> 2) & 1) + 1
      v39 = ((ord('p') >> 3) & 1) + 1
  
      if (v21 + v23 == int(serial[6]) and v47 + v39 == int(serial[7]) and v43 + v41 == int(serial[8]) and v45 + v35 == int(serial[9]) and v49 + v37 == int(serial[10])):
          print(chr(i))
          break
          
  ```

  ```
  b u
  c q
  f t
  g p
  m
  可能的name为
  bump
  cqmp
  ftmp
  gpmp
  ```

  答案为`bump`

- #### Direct3D FPS

  ![6](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr-2/6.png)

  我说我天天在宿舍打游戏还有人不信，gg

  ![7](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr-2/7.png)

  应该是游戏胜利出flag，跟着game clear找判断是否胜利的函数

  ![8](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr-2/8.png)

  byte_407028可能是加密后的flag

  这个函数应该是判断是否存在存活的怪物，一个怪物对象占132字节，如果没有存活的怪物就输出flag和game clear

  向上找解密函数

  ![9](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr-2/9.png)

  这里出现了dword_409194，根据判断胜利的函数可以知道这代表怪物是否存活，v3小于0时，把0赋给dword_409194，显然v3即dword_409190代表怪物血量，这个函数是一个减血函数。

  这里又使用v2作为索引，而v2=132 * result，一个怪物对象的大小为132字节，result是sub_403440的返回值，byte_407028又使用result作为索引，可以猜测result的值为1，2，3.....

  sub_403440的功能应该是获取被击中的怪物

  现在应该去找byte_409184[v2 * 4]的值

  运行程序查看byte_409184

  ![10](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr-2/10.png)

  ![11](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr-2/11.png)

  ......

  找到规律

  ```python
  a = [0x43, 0x6B, 0x66, 0x6B, 0x62, 0x75, 0x6C, 0x69, 0x4C, 0x45, 0x5C, 0x45, 0x5F, 0x5A, 0x46, 0x1C, 0x07, 0x25, 0x25,
       0x29, 0x70, 0x17, 0x34, 0x39, 0x01, 0x16, 0x49, 0x4C, 0x20, 0x15, 0x0B, 0x0F, 0xF7, 0xEB, 0xFA, 0xE8, 0xB0, 0xFD,
       0xEB, 0xBC, 0xF4, 0xCC, 0xDA, 0x9F, 0xF5, 0xF0, 0xE8, 0xCE, 0xF0, 0xA9]
  for i in range(len(a)):
      print(chr(a[i] ^ i * 4), end='')
  
  ```

  `Congratulation~ Game Clear! Password is Thr3EDPr0m`

- #### HateIntel

  ![12](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr-2/12.png)

  vars0 + i - 92就是输入的字符串，加密后与byte_3004比较

  sub_232C是加密函数

  ![13](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr-2/13.png)

  加密循环四次，sub_2494是加密函数

  ![14](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr-2/14.png)

  直接爆破

  ```python
  a = [0x44, 0xF6, 0xF5, 0x57, 0xF5, 0xC6, 0x96, 0xB6, 0x56, 0xF5, 0x14, 0x25, 0xD4, 0xF5, 0x96, 0xE6, 0x37, 0x47, 0x27,
       0x57, 0x36, 0x47, 0x96, 0x03, 0xE6, 0xF3, 0xA3, 0x92, 0x00]
  flag = ''
  for i in range(len(a)):
      for j in range(256):
          k = j
          for t in range(4):
              v3 = k * 2
              if (v3 & 0x100):
                  v3 |= 1
              k = v3 % 256
          if (k == a[i]):
              flag += chr(j)
  print(flag)
  
  ```

  `Do_u_like_ARM_instructi0n?:) `



![15](https://raw.githubusercontent.com/AiDaiP/images/master/reversing.kr-2/15.jpg)

