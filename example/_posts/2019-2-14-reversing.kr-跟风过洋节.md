# reversing.kr-跟风过洋节

跟风过洋节

![1](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/1.jpg)

* Easy_CrackMe

  ![2](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/2.png)

  四段拼起来就完事了

  `Ea5yR3versing`

  

* Easy_KeygenMe

  ```
  Find the Name when the Serial is 5B134977135E7D13
  ```

  ![3](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/3.png)

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

  

* ReversingKr UnpackMe

  ```
  Find the OEP
  ex) 00401000
  ```

  ![4](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/4.png)

  单步发现在这窗口弹出，俺寻思这就是入口点

  `000401150`

  

* Music_Player

  

  ```
  This MP3 Player is limited to 1 minutes.
  You have to play more than one minute.
  
  There are exist several 1-minute-check-routine.
  After bypassing every check routine, you will see the perfect flag.
  ```

  播放器只能播放一分钟，绕过这个限制就可以得到flag

  ![5](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/5.png)

  先查一波字符串，看看有没有和一分钟有关系的

  定位下断点

  ![6](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/6.png)

  59秒时断

  ![8](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/8.png)

  向上翻找跳转，这有个和0xEA60比较的，俺寻思这是600000毫秒

  把0040456B的改成jmp再跑一遍

  ![9](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/9.png)

  出现新的弹窗

  重新跑一遍，在004045B2断，然后单步，找到弹窗位置

  ![10](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/10.png)

  向上找跳转，改成jmp，再跑一遍

  ![7](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/7.png)

  `LIstenCare`

  

* ImagePrc

  ![11](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/11.png)

  这玩意应该是画完图之后比较

  IDA pro里也看不懂是咋比较的

  gg

  

* Replace

  

  ![14](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/14.png)

  ![13](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/13.png)

  定位到`Correct!`看一下

  ![15](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/15.png)

  ![16](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/16.png)

  IDA Pro里看一下，好像没法跳转到这里

  俺寻思直接把前面俩jjmp全改成nop不就完事了

  ![17](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/17.png)

  这题是要找输入的值，出个correct没啥用

  继续用OD看

  ![18](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/18.png)

  在GetDlgItemInt断，输入后步过

  ![19](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/19.png)

  在这里程序退出

  ![20](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/20.png)

  再跑一遍，进入40466F看一眼

  把eax的值对应地址的内容改成0x90，0x90是nop指令，所以eax=0x401071就行了

  输入0的时候eax=0x601605CB，输入1，eax=0x601605CC

  0x401071- ? = 0x601605CC

  显然需要eax溢出

  0x100401071 - 0x601605CC = ‭0xA02A0AA6‬

  ![21](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/21.png)

  `‭2687109798‬`

  

* Easy ELF

  ![22](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/22.png)

  ![23](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/23.png)

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

  

* CSHOP

  ![24](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/24.png)

  c#写的，用dnSpy看看

  ![25](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/25.png)

  这有个大小被设置为0的按钮，那按它一下会发生啥

  试试回车能不能按这个按钮

  ![26](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/reversing.kr/26.png)

  输进去试了一下还真是flag

  `P4W6RP6SES `