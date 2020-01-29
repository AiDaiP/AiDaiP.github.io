---
layout: post
title:  "buuoj-从Reverse到去世"
date:   2020-1-17
desc: ""
keywords: ""
categories: [Ctf]
tags: [CTF,buuoj,Reverse]
icon: icon-html
---

# buuoj-从Reverse到去世

### reverse3

```python
import base64
enc = 'e3nifIH9b_C@n@dH'
flag = ''
for i in range(len(enc)):
    flag += chr(ord(enc[i]) - i)

flag = base64.b64decode(flag)
print(flag)
```

flag{i_l0ve_you}

### 不一样的flag

走迷宫

```
*1111
01000
01010
00010
1111#
```

222441144222

### 刮开有奖

base64

```
V1Axak1w
WP1jMp
```

flag长度为8，还剩前面两位，👴⑧想看了直接爆破

UJWP1jMp

###  SimpleRev

```python
chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
key1 = 'ADSFKNDCLS'.lower()
key2 = 'killshadow'

flag = ''
for i in range(len(key2)):
    str2 = key2[i]
    for j in chars:
        if str2 == chr((ord(j) - 39 - ord(key1[i % len(key1)]) + 97) % 26 + 97):
            flag += j
print(flag)
```

### Java逆向解密

```python
fuck = [180, 136, 137, 147, 191, 137, 147, 191, 148, 136, 133, 191, 134, 140, 129, 135, 191, 65]
flag = ''
for i in range(len(fuck)):
	flag += chr(fuck[i] - ord('@') ^ 0x20)
print(flag)
```

### findit

凯撒

```
pvkq{m164675262033l4m49lnp7p9mnk28k75}
flag{c164675262033b4c49bdf7f9cda28a75}
```



### 相册

libcore.so，看字符串

```
MTgyMTg0NjUxMjVAMTYzLmNvbQ==
dXF0c3F5aXpsZXN0dGxqdg==

18218465125@163.com
uqtsqyizlesttljv
```



### [GXYCTF2019]luck_guy

```
f1 'GXY{do_not_
```

```
      case 4:
        v6 = 0;
        s = ' fo`guci';
        strcat(&f2, (const char *)&s);
        break;
      case 5:
        for ( j = 0; j <= 7; ++j )
        {
          if ( j % 2 == 1 )
            v1 = *(&f2 + j) - 2;
          else
            v1 = *(&f2 + j) - 1;
          *(&f2 + j) = v1;
        }
```

```
nmsl = 'icug`of\x7f'
flag = ''
for i in range(8):
	if i % 2 == 1:
		flag += chr(ord(nmsl[i])-2)
	else:
		flag += chr(ord(nmsl[i])-1)
print(flag)
```

### [GXYCTF2019]simple CPP

```c
  v19 = b & a;
  *v18 = b & a;
  v20 = c & ~a;
  v18[1] = v20;
  not_b = ~b;
  v22 = c & not_b;
  v18[2] = c & not_b;
  v23 = a & not_b;
  v18[3] = v23;
  if ( v20 != 1176889593874i64 )
  {
    v18[1] = 0i64;
    v20 = 0i64;
  }
  v24 = v20 | v19 | v22 | v23;
  b_1 = v15[1];
  c_1 = v15[2];
  v27 = v22 & *v15 | c_1 & (v19 | b_1 & ~*v15 | ~(b_1 | *v15));
  v28 = 0;
  if ( v27 == 577031497978884115i64 )
    v28 = v24 == 4483974544037412639i64;
  if ( (v24 ^ v15[3]) == 4483974543195470111i64 )
    v0 = v28;
  if ( (v20 | v19 | b_1 & c_1) != (~*v15 & c_1 | 864693332579200012i64) || v0 != 1 )
  {
    sub_1400019C0(std::cout, "Wrong answer!try again");
    j_j_free(v3);
  }
  else
  {
    v29 = sub_1400019C0(std::cout, "Congratulations!flag is GXY{");
    v30 = &Memory;
    if ( v38 >= 0x10 )
      v30 = Memory;
    v31 = sub_140001FD0(v29, v30, v37);
    sub_1400019C0(v31, "}");
    j_j_free(v3);
  }
```

```python
from z3 import *
a,b,c,d=BitVecs('a b c d',64)
fucker=Solver()
fucker.add(~a&c==1176889593874)
fucker.add((1176889593874|(a&b)|(c&~b)|(a&~b))==4483974544037412639)
fucker.add(4483974544037412639^d==4483974543195470111)
fucker.add(((c&~b)&a|c&((a&b)|(b&~a)|~(b|a)))==577031497978884115)
print(fucker.check())
print(fucker.model())
```

```
sat
[b = 0,
 a = 4483973367147818765,
 d = 842073600,
 c = 577031497978884115]
 
 b = 3906943046058528520
```



### [GWCTF 2019]pyre

pyc，反编译拿源码

```python
print 'Welcome to Re World!'
print 'Your input1 is your flag~'
l = len(input1)
for i in range(l):
    num = ((input1[i] + i) % 128 + 128) % 128
    code += num

for i in range(l - 1):
    code[i] = code[i] ^ code[i + 1]

print code
code = ['\x1f', '\x12', '\x1d', '(', '0', '4', '\x01', '\x06', '\x14', '4', ',', '\x1b', 'U', '?', 'o', '6', '*', ':', '\x01', 'D', ';', '%', '\x13']
```

```python
code = ['\x1f', '\x12', '\x1d', '(', '0', '4', '\x01', '\x06', '\x14', '4', ',', '\x1b', 'U', '?', 'o', '6', '*', ':', '\x01', 'D', ';', '%', '\x13']
code = code[::-1]
for i in range(len(code)-1):
    code[i+1] = chr(ord(code[i+1])^ord(code[i]))
code = code[::-1]
for i in range(len(code)):
    code[i] = chr((ord(code[i]) - i)%128)
print(''.join(code))
```

### [GWCTF 2019]re3

SMC，0x402219异或0x99

AES ECB

key在0x603170

```
Breakpoint *0x402219
pwndbg> x/10gx 0x603170
0x603170:       0x4c7ab42135498dcb      0xce669222627eaec1
0x603180:       0x0000000000000000      0x0000000000000000
```

密文在0x6030A0

```
pwndbg> x/10gx 0x6030A0
0x6030a0:       0xcc5e7c14c0ad0abc      0x2bd5519cbc40b1e0
0x6030b0:       0x4b32e54d43b9b246      0x5b4bdb9cb3b47fad
```

```python
from Crypto.Cipher import AES
key = 'cb8d493521b47a4cc1ae7e62229266ce'.decode('hex')
cipher = 'bc0aadc0147c5ecce0b140bc9c51d52b46b2b9434de5324bad7fb4b39cdb4b5b'.decode('hex')
fucker = AES.new(key, mode=AES.MODE_ECB)
flag = fucker.decrypt(cipher)
print(flag)

```

### [GWCTF 2019]xxor

```c
  for ( i = 0; i <= 5; ++i )
  {
    printf("%s", "input: ", i);
    __isoc99_scanf("%d", &input + 4 * i);
  }
  v11 = 0LL;
  v12 = 0LL;
  v13 = 0LL;
  v14 = 0LL;
  v15 = 0LL;
  for ( j = 0; j <= 4; j += 2 )
  {
    a = *(&input + j);
    b = *(&input + j + 1);
    xor(&a, &a2234);
    *(&v11 + j) = a;
    *(&v11 + j + 1) = b;
  }
  if ( judge(&v11) != 1 )
  {
    puts("NO NO NO~ ");
    exit(0);
  }
  puts("Congratulation!\n");
  puts("You seccess half\n");
  puts("Do not forget to change input to hex and combine~\n");
  puts("ByeBye");
```

judge

```c
signed __int64 __fastcall sub_400770(_DWORD *a1)
{
  signed __int64 result; // rax

  if ( a1[2] - a1[3] != 2225223423LL || a1[3] + a1[4] != 4201428739LL || a1[2] - a1[4] != 1121399208LL )
  {
    puts("Wrong!");
    result = 0LL;
  }
  else if ( *a1 != 3746099070 || a1[5] != 2230518816 || a1[1] != 550153460 )
  {
    puts("Wrong!");
    result = 0LL;
  }
  else
  {
    puts("good!");
    result = 1LL;
  }
  return result;
}
```

解出六个数据

```python
from z3 import *
a0,a1,a2,a3,a4,a5 = BitVecs('a0 a1 a2 a3 a4 a5',64)
fucker = Solver()
fucker.add(a2 - a3 == 2225223423)
fucker.add(a3 + a4 == 4201428739)
fucker.add(a2 - a4 == 1121399208)
fucker.add(a0 == 3746099070)
fucker.add(a5 == 2230518816)
fucker.add(a1 == 550153460)

print(fucker.check())
print(fucker.model())
```

```
sat
[a4 = 2652626477,
 a1 = 550153460,
 a5 = 2230518816,
 a0 = 3746099070,
 a2 = 3774025685,
 a3 = 1548802262]
```

解flag

```c
#include <stdio.h>
#include <stdint.h>

int main(int argc, char const *argv[])
{

	uint32_t res[7] = {3746099070,550153460,3774025685,1548802262,2652626477,2230518816};
	for (int i = 0; i <= 4; ++i)
	{
		uint32_t v3 = res[i];
		uint32_t v4 = res[i+1];
		uint32_t v5 = 0x62F35080;
		for (int j = 0; j <= 0x3f; ++j)
		{
			v4 -= (v3 + v5 + 20) ^ ((v3 << 6) + 3) ^ ((v3 >> 9) + 4) ^ 0x10;
			v3 -= (v4 + v5 + 11) ^ ((v4 << 6) + 2) ^ ((v4 >> 9) + 2) ^ 0x20;
			v5 -= 0x458BCD42;
		}
		printf("%x %x\n",v3,v4);
	}
	return 0;
}
```

```
666c61 677b72
61ceace1 aa78edcc
655f69 735f67
28a3dff5 acb93d26
726561 74217d
[0x666c61,0x677b72,0x655f69,0x735f67,0x726561,0x74217d]
```



### [GWCTF 2019]babyvm

```c
unsigned __int64 sub_F83()
{
  int i; // [rsp+Ch] [rbp-14h]
  unsigned __int64 v2; // [rsp+18h] [rbp-8h]

  v2 = __readfsqword(0x28u);
  for ( i = 0; dword_2022A4 - 1 > i; ++i )
  {
    if ( *(qword_2022A8 + i + 32) != aFzAmAmFmtSum[i] )
    {
      puts("WRONG!");
      exit(0);
    }
  }
  puts("Congratulation?");
  puts("tips: input is the start");
  return __readfsqword(0x28u) ^ v2;
}
```

Congratulation带问号必是假flag，给👴爬

顺着假opcode往下翻找到真的

真judge

```
unsigned __int64 judge()
{
  int i; // [rsp+Ch] [rbp-14h]
  unsigned __int64 v2; // [rsp+18h] [rbp-8h]

  v2 = __readfsqword(0x28u);
  for ( i = 0; dword_2022A4 - 1 > i; ++i )
  {
    if ( *(qword_2022A8 + i) != byte_202020[i] )
      exit(0);
  }
  return __readfsqword(0x28u) ^ v2;
}
```



看vm咋跑

```
unsigned __int64 __fastcall sub_CD1(__int64 a1)
{
  unsigned __int64 v1; // ST18_8

  v1 = __readfsqword(0x28u);
  *a1 = 0;
  *(a1 + 4) = 18;
  *(a1 + 8) = 0;
  *(a1 + 12) = 0;
  *(a1 + 16) = &unk_202060;
  *(a1 + 24) = 0xF1u;
  *(a1 + 32) = mov;
  *(a1 + 40) = 0xF2u;
  *(a1 + 48) = xor;
  *(a1 + 56) = 0xF5u;
  *(a1 + 64) = vm_read;
  *(a1 + 72) = 0xF4u;
  *(a1 + 80) = ret;
  *(a1 + 88) = 0xF7u;
  *(a1 + 96) = mul;
  *(a1 + 104) = 0xF8u;
  *(a1 + 112) = exch;
  *(a1 + 120) = 0xF6u;
  *(a1 + 128) = add;
  qword_2022A8 = malloc(0x512uLL);
  memset(qword_2022A8, 0, 0x512uLL);
  return __readfsqword(0x28u) ^ v1;
}
```

mov

```
0xe1 : r1 = stack[offset]
0xe2 : r2 = stack[offset]
0xe3 : r3 = stack[offset]
0xe4 : stack[offset] = r1
0xe5 : r5 = stack[offset]
0xe7 : stack[offset] = r2
```

xor

```
r1 = r1^r2
```

mul

```
f1 = r1*r4
```

add

```
r1 = 3*r1+2*r2+r3
```

exch

```
fuck = r1
r1 = r2
r2 = fuck
```

opcode

```c
0xF5, read

0xF1, 0xE1, 0x00, 0x00, 0x00, 0x00, r1 = stack[0]
0xF1, 0xE2, 0x01, 0x00, 0x00, 0x00, r2 = stack[1]
0xF2, r1 = r1^r2
0xF1, 0xE4, 0x00, 0x00, 0x00, 0x00, stack[0] = r1

0xF1, 0xE1, 0x01, 0x00, 0x00, 0x00, r1 = stack[1]
0xF1, 0xE2, 0x02, 0x00, 0x00, 0x00, r2 = stack[2]
0xF2, r1 = r1^r2
0xF1, 0xE4, 0x01, 0x00, 0x00, 0x00, stack[1] = r1

0xF1, 0xE1, 0x02, 0x00, 0x00, 0x00, r1 = stack[2]
0xF1, 0xE2, 0x03, 0x00, 0x00, 0x00, r2 = stack[3]
0xF2, r1 = r1^r2
0xF1, 0xE4, 0x02, 0x00, 0x00, 0x00, stack[2] = r1

0xF1, 0xE1, 0x03, 0x00, 0x00, 0x00, r1 = stack[3]
0xF1, 0xE2, 0x04, 0x00, 0x00, 0x00, r2 = stack[4]
0xF2, r1 = r1^r2
0xF1, 0xE4, 0x03, 0x00, 0x00, 0x00, stack[3] = r1

0xF1, 0xE1, 0x04, 0x00, 0x00, 0x00, r1 = stack[4]
0xF1, 0xE2, 0x05, 0x00, 0x00, 0x00, r2 = stack[5]
0xF2, r1 = r1^r2
0xF1, 0xE4, 0x04, 0x00, 0x00, 0x00, stack[4] = r1

0xF1, 0xE1, 0x05, 0x00, 0x00, 0x00, r1 = stack[5]
0xF1, 0xE2, 0x06, 0x00, 0x00, 0x00, r2 = stack[6]
0xF2, r1 = r1^r2
0xF1, 0xE4, 0x05, 0x00, 0x00, 0x00, stack[5] = r1

0xF1, 0xE1, 0x06, 0x00, 0x00, 0x00, r1 = stack[6]
0xF1, 0xE2, 0x07, 0x00, 0x00, 0x00, r2 = stack[7]
0xF1, 0xE3, 0x08, 0x00, 0x00, 0x00, r3 = stack[8]
0xF1, 0xE5, 0x0C, 0x00, 0x00, 0x00, r4 = stack[12]
0xF6, r1 = 3*r1+2*r2+r3
0xF7, r1 = r1*r4
0xF1, 0xE4, 0x06, 0x00, 0x00, 0x00, stack[6] = r1

0xF1, 0xE1, 0x07, 0x00, 0x00, 0x00, r1 = stack[7]
0xF1, 0xE2, 0x08, 0x00, 0x00, 0x00, r2 = stack[8]
0xF1, 0xE3, 0x09, 0x00, 0x00, 0x00, r3 = stack[9]
0xF1, 0xE5, 0x0C, 0x00, 0x00, 0x00, r4 = stack[12]
0xF6, r1 = 3*r1+2*r2+r3
0xF7, r1 = r1*r4
0xF1, 0xE4, 0x07, 0x00, 0x00, 0x00, stack[7] = r1

0xF1, 0xE1, 0x08, 0x00, 0x00, 0x00, r1 = stack[8]
0xF1, 0xE2, 0x09, 0x00, 0x00, 0x00, r2 = stack[9]
0xF1, 0xE3, 0x0A, 0x00, 0x00, 0x00, r3 = stack[10]
0xF1, 0xE5, 0x0C, 0x00, 0x00, 0x00, r3 = stack[12]
0xF6, r1 = 3*r1+2*r2+r3
0xF7, r1 = r1*r4
0xF1, 0xE4, 0x08, 0x00, 0x00, 0x00, stack[8] = r1

0xF1, 0xE1, 0x0D, 0x00, 0x00, 0x00, r1 = stack[13]
0xF1, 0xE2, 0x13, 0x00, 0x00, 0x00, r2 = stack[19]
0xF8, r1,r2
0xF1, 0xE4, 0x0D, 0x00, 0x00, 0x00, stack[13] = r1
0xF1, 0xE7, 0x13, 0x00, 0x00, 0x00, stack[19] = r2

0xF1, 0xE1, 0x0E, 0x00, 0x00, 0x00, r1 = stack[14]
0xF1, 0xE2, 0x12, 0x00, 0x00, 0x00, r2 = stack[18]
0xF8, r1,r2
0xF1, 0xE4, 0x0E, 0x00, 0x00, 0x00, stack[14] = r1
0xF1, 0xE7, 0x12, 0x00, 0x00, 0x00, stack[18] = r2

0xF1, 0xE1, 0x0F, 0x00, 0x00, 0x00, r1 = stack[15]
0xF1, 0xE2, 0x11, 0x00, 0x00, 0x00, r2 = stack[17]
0xF8, r1,r2
0xF1, 0xE4, 0x0F, 0x00, 0x00, 0x00, stack[15] = r1
0xF1, 0xE7, 0x11, 0x00, 0x00, 0x00, stack[17] = r2

0xF4, ret
```

```python
from z3 import *
enc = [0x69, 0x45, 0x2A, 0x37, 0x09, 0x17, 0xC5, 0x0B, 0x5C, 0x72, 0x33, 0x76, 0x33, 0x21, 0x74, 0x31, 0x5F, 0x33, 0x73, 0x72]
a,b,c = BitVecs('a b c',64)
fucker = Solver()
fucker.add((3*a+2*b+c)*0x33&0xff==0xc5)
fucker.add((3*b+2*c+0x72)*0x33&0xff==0x0b)
fucker.add((3*c+2*0x72+0x33)*0x33&0xff==0x5c)

print(fucker.check())
print(fucker.model())
#[c = 95, b = 51, a = 118]
enc[6] = 118
enc[7] = 51
enc[8] = 95

for i in range(6):
	enc[6-i-1] ^= enc[6-i]

for i in range(3):
	fuck = enc[13+i]
	enc[13+i] = enc[19-i]
	enc[19-i] = fuck

flag = ''
for i in enc:
	flag += chr(i)
print(flag)
```

### 简单注册器

```java
class Untitled {
	public static void main(String[] args) {
                int v11 = 31;
                int v9 = 2;
                int v2 = 1;

                if(v2 == 1) {
                    char[] v5 = "dd2940c04462b4dd7c450528835cca15".toCharArray();
                    v5[v9] = ((char)(v5[v9] + v5[3] - 50));
                    v5[4] = ((char)(v5[v9] + v5[5] - 48));
                    v5[30] = ((char)(v5[v11] + v5[9] - 48));
                    v5[14] = ((char)(v5[27] + v5[28] - 97));
                    int v4;
                    for(v4 = 0; v4 < 16; ++v4) {
                        char v0 = v5[31 - v4];
                        v5[31 - v4] = v5[v4];
                        v5[v4] = v0;
                    }
					System.out.println(v5);
                    
                }
            
	}
}
```

直接跑出flag



### [FlareOn2]very_success

堆栈不平衡，0x401000改nop

输入的文件在0x402159

从0x401084开始跑，设置参数，返回到0x401064，找success

```python
import angr

fuck_addr = 0x402159
p = angr.Project('very_success', auto_load_libs=False)
s = p.factory.blank_state(addr=0x401084)
s.mem[s.regs.esp+8:].dword = fuck_addr
s.mem[s.regs.esp+4:].dword = 0x4010e4
s.mem[s.regs.esp:].dword = 0x401064
s.memory.store(fuck_addr, s.solver.BVS('flag', 8*40))
sm = p.factory.simulation_manager(s)
sm.explore(find=0x40106b, avoid=0x401072)
print(sm.found[0].solver.eval(sm.found[0].memory.load(fuck_addr, 40), cast_to=bytes))

```



下断点拿v7

```
    v7 = (_BYTE *)(a2 + 36);
```

```
004010E0  1C FF FF FF AF AA AD EB  AE AA EC A4 BA AF AE AA
004010F0  8A C0 A7 B0 BC 9A BA A5  A5 BA AF B8 9D B8 F9 AE
00401100  9D AB B4 BC B6 B3 90 9A  A8 00 00 00 00 00 00 00
```

```python
sum = 0
flag = ''
fuck = [0xa8,0x9a,0x90,0xb3,0xb6,0xbc,0xb4,0xab,0x9d,0xae,0xf9,0xb8,0x9d,0xb8,0xaf,0xba,0xa5,0xa5,0xba,0x9a,0xbc,0xb0,0xa7,0xc0,0x8a,0xaa,0xae,0xaf,0xba,0xa4,0xec,0xaa,0xae,0xeb,0xad,0xaa,0xaf]
for i in range(37):
    nmsl = (1 << (sum & 3))
    flag += chr((455 ^ (fuck[i] - nmsl - 1)) % 256)
    sum += fuck[i]
print(flag)
```



### HellScream

RSA

```
n = 0xA324F100182D501F6F6F78F397A3AA59641023D6A3DED8A4BF344F1E0FC71C188F4D
p = 62822928286347608648869628628072621308819
q = 76979057716441943100156208562083628995999
e = pow(0x6D6F632E6D736131352E777777,17,n)
E <= e+99
m = 'Happy Birthday Buddy\x00ok'
m = 0x6B6F007964647542207961646874726942207970706148
```



### [GUET-CTF2019]re

查看字符串有upx，先upx -d 

```
flag = ''
flag += chr(166163712 / 1629056)
flag += chr(731332800 / 6771600)
flag += chr(357245568 / 3682944)
flag += chr(1074393000 / 10431000)
flag += chr(489211344 / 3977328)
flag += chr(518971936 / 5138336)
flag += '1'
flag += chr(406741500 / 7532250)
flag += chr(294236496 / 5551632)
flag += chr(177305856 / 3409728)
flag += chr(650683500 / 13013670)
flag += chr(298351053 / 6088797)
flag += chr(386348487 / 7884663)
flag += chr(438258597 / 8944053)
flag += chr(249527520 / 5198490)
flag += chr(445362764 / 4544518)
flag += chr(981182160 /10115280)
flag += chr(174988800 / 3645600)
flag += chr(493042704 / 9667504)
flag += chr(257493600 / 5364450)
flag += chr(767478780 / 13464540)
flag += chr(312840624 / 5488432)
flag += chr(1404511500 / 14479500)
flag += chr(316139670 / 6451830)
flag += chr(619005024 / 6252576)
flag += chr(372641472 / 7763364)
flag += chr(373693320 / 7327320)
flag += chr(498266640 / 8741520)
flag += chr(452465676 / 8871876)
flag += chr(208422720 / 4086720)
flag += chr(515592000 / 9374400)
flag += chr(719890500 / 5759124)
print(flag)

```



### [GUET-CTF2019]encrypt

三个步骤

最后一步直接根据密文爆破

```python
enc = 'Z`TzzTrD|fQP[_VVL|yneURyUmFklVJgLasJroZpHRxIUlH\\vZE='
print(len(enc))
print(len(enc)%3)#1
fuck = ''
for i in range(3,len(enc),4):
	print(i)
	flag = False
	for j in range(0xff+1):
		for k in range(0xff+1):
			for l in range(0xff+1):
				if ord(enc[i-3]) == ((j >> 2) & 0x3F) + 61 and \
				ord(enc[i-2]) == ((((k & 0xFF) >> 4) | 16 * j) & 0x3F) + 61 and \
				 ord(enc[i-1]) == ((((l & 0xFF) >> 6) | 4 * k) & 0x3F) + 61 and \
				 ord(enc[i]) == (l & 0x3F) + 61:
					print(hex(j),hex(k),hex(l))
					fuck = fuck + chr(j)+chr(k)+chr(l)
	print(fuck.encode('hex'))
#7635fdf57d47fe95137a26593fff31a1857c63026ebd936a3e4d8dd727732d5ecc62f2dfe5d2

```

0x4006b6断，看rsi拿地址，跑到0x4007db断

```c
0x7ffffffed960: 0x0000000000000000      0x00000031000000b0
0x7ffffffed970: 0x0000007000000075      0x000000df000000f8
0x7ffffffed980: 0x0000003c00000007      0x0000007100000078
0x7ffffffed990: 0x0000002900000050      0x000000160000002c
0x7ffffffed9a0: 0x0000001200000069      0x0000002b000000c8
0x7ffffffed9b0: 0x0000007f0000003b      0x000000e7000000b2
0x7ffffffed9c0: 0x000000680000004b      0x000000c50000008c
0x7ffffffed9d0: 0x00000015000000a6      0x0000005800000003
0x7ffffffed9e0: 0x0000000400000047      0x0000008d00000013
0x7ffffffed9f0: 0x0000002600000087      0x000000ed00000009
0x7ffffffeda00: 0x0000008a00000017      0x000000f2000000c2
0x7ffffffeda10: 0x000000c000000043      0x00000059000000ac
0x7ffffffeda20: 0x000000f500000097      0x000000670000003f
0x7ffffffeda30: 0x000000390000005e      0x000000d500000086
0x7ffffffeda40: 0x0000006100000072      0x000000f7000000da
0x7ffffffeda50: 0x0000000500000001      0x000000c30000008b
0x7ffffffeda60: 0x00000077000000b1      0x0000001d000000af
0x7ffffffeda70: 0x000000c600000030      0x0000000e00000045
0x7ffffffeda80: 0x000000ee0000005f      0x000000f0000000ae
0x7ffffffeda90: 0x000000ce00000028      0x000000a7000000cd
0x7ffffffedaa0: 0x0000002a0000009b      0x0000004800000019
0x7ffffffedab0: 0x0000004400000008      0x000000fe00000020
0x7ffffffedac0: 0x000000b50000006d      0x0000006a0000002e
0x7ffffffedad0: 0x00000034000000f1      0x0000001e000000bc
0x7ffffffedae0: 0x000000cc0000003e      0x0000009200000041
0x7ffffffedaf0: 0x000000bd000000d8      0x000000e8000000a5
0x7ffffffedb00: 0x0000000a0000004d      0x0000000d00000049
0x7ffffffedb10: 0x000000fa000000a2      0x0000007400000062
0x7ffffffedb20: 0x00000083000000d4      0x0000009400000096
0x7ffffffedb30: 0x000000cb0000003d      0x0000006300000018
0x7ffffffedb40: 0x0000004600000099      0x000000b7000000ca
0x7ffffffedb50: 0x000000cf0000008e      0x000000a3000000fb
0x7ffffffedb60: 0x0000007e0000006c      0x0000002700000051
0x7ffffffedb70: 0x0000009a00000060      0x000000f300000011
0x7ffffffedb80: 0x0000006e0000005c      0x00000042000000ba
0x7ffffffedb90: 0x0000002f00000076      0x000000bf000000ef
0x7ffffffedba0: 0x000000aa00000021      0x000000d6000000e4
0x7ffffffedbb0: 0x000000550000001b      0x000000be0000007d
0x7ffffffedbc0: 0x000000d3000000ea      0x000000f400000010
0x7ffffffedbd0: 0x0000004a000000c7      0x0000007900000023
0x7ffffffedbe0: 0x000000a400000084      0x000000ab0000001c
0x7ffffffedbf0: 0x000000db00000014      0x0000003a0000004c
0x7ffffffedc00: 0x00000052000000b8      0x00000037000000ec
0x7ffffffedc10: 0x000000b600000038      0x000000a0000000d2
0x7ffffffedc20: 0x0000005b0000005a      0x0000006600000098
0x7ffffffedc30: 0x0000009e00000054      0x0000004f0000004e
0x7ffffffedc40: 0x000000c4000000b4      0x000000d0000000c9
0x7ffffffedc50: 0x0000009c00000025      0x000000de00000080
0x7ffffffedc60: 0x000000060000002d      0x0000000b00000022
0x7ffffffedc70: 0x0000006b00000091      0x000000f60000009f
0x7ffffffedc80: 0x000000e2000000e6      0x0000000f000000c1
0x7ffffffedc90: 0x0000009000000093      0x0000009d0000007b
0x7ffffffedca0: 0x000000dd0000008f      0x00000065000000e5
0x7ffffffedcb0: 0x000000ad00000035      0x000000dc000000a9
0x7ffffffedcc0: 0x000000bb00000082      0x0000005300000000
0x7ffffffedcd0: 0x000000a8000000d1      0x000000e900000033
0x7ffffffedce0: 0x0000001a00000040      0x000000a1000000ff
0x7ffffffedcf0: 0x0000003600000095      0x000000eb000000d9
0x7ffffffedd00: 0x000000e300000089      0x000000730000007c
0x7ffffffedd10: 0x0000008800000085      0x000000e00000007a
0x7ffffffedd20: 0x00000064000000fd      0x000000570000000c
0x7ffffffedd30: 0x000000b300000032      0x0000001f000000b9
0x7ffffffedd40: 0x000000fc000000d7      0x000000e100000081
0x7ffffffedd50: 0x000000f900000002      0x000000560000005d
0x7ffffffedd60: 0x000000240000006f      0x00007fffff629170
```

```python
fuck =  [0x0,0x0,0xb0,0x31,0x75,0x70,0xf8,0xdf,0x07,0x3c,0x78,0x71,
0x50,0x29,0x2c,0x16,0x69,0x12,0xc8,0x2b,0x3b,0x7f,0xb2,0xe7,
0x4b,0x68,0x8c,0xc5,0xa6,0x15,0x03,0x58,0x47,0x04,0x13,0x8d,
0x87,0x26,0x09,0xed,0x17,0x8a,0xc2,0xf2,0x43,0xc0,0xac,0x59,
0x97,0xf5,0x3f,0x67,0x5e,0x39,0x86,0xd5,0x72,0x61,0xda,0xf7,
0x01,0x05,0x8b,0xc3,0xb1,0x77,0xaf,0x1d,0x30,0xc6,0x45,0x0e,
0x5f,0xee,0xae,0xf0,0x28,0xce,0xcd,0xa7,0x9b,0x2a,0x19,0x48,
0x08,0x44,0x20,0xfe,0x6d,0xb5,0x2e,0x6a,0xf1,0x34,0xbc,0x1e,
0x3e,0xcc,0x41,0x92,0xd8,0xbd,0xa5,0xe8,0x4d,0x0a,0x49,0x0d,
0xa2,0xfa,0x62,0x74,0xd4,0x83,0x96,0x94,0x3d,0xcb,0x18,0x63,
0x99,0x46,0xca,0xb7,0x8e,0xcf,0xfb,0xa3,0x6c,0x7e,0x51,0x27,
0x60,0x9a,0x11,0xf3,0x5c,0x6e,0xba,0x42,0x76,0x2f,0xef,0xbf,
0x21,0xaa,0xe4,0xd6,0x1b,0x55,0x7d,0xbe,0xea,0xd3,0x10,0xf4,
0xc7,0x4a,0x23,0x79,0x84,0xa4,0x1c,0xab,0x14,0xdb,0x4c,0x3a,
0xb8,0x52,0xec,0x37,0x38,0xb6,0xd2,0xa0,0x5a,0x5b,0x98,0x66,
0x54,0x9e,0x4e,0x4f,0xb4,0xc4,0xc9,0xd0,0x25,0x9c,0x80,0xde,
0x2d,0x06,0x22,0x0b,0x91,0x6b,0x9f,0xf6,0xe6,0xe2,0xc1,0x0f,
0x93,0x90,0x7b,0x9d,0x8f,0xdd,0xe5,0x65,0x35,0xad,0xa9,0xdc,
0x82,0xbb,0x00,0x53,0xd1,0xa8,0x33,0xe9,0x40,0x1a,0xff,0xa1,
0x95,0x36,0xd9,0xeb,0x89,0xe3,0x7c,0x73,0x85,0x88,0x7a,0xe0,
0xfd,0x64,0x0c,0x57,0x32,0xb3,0xb9,0x1f,0xd7,0xfc,0x81,0xe1,
0x02,0xf9,0x5d,0x56,0x6f,0x24]
```

```python
fuck =  [0x0,0x0,0xb0,0x31,0x75,0x70,0xf8,0xdf,0x07,0x3c,0x78,0x71,
0x50,0x29,0x2c,0x16,0x69,0x12,0xc8,0x2b,0x3b,0x7f,0xb2,0xe7,
0x4b,0x68,0x8c,0xc5,0xa6,0x15,0x03,0x58,0x47,0x04,0x13,0x8d,
0x87,0x26,0x09,0xed,0x17,0x8a,0xc2,0xf2,0x43,0xc0,0xac,0x59,
0x97,0xf5,0x3f,0x67,0x5e,0x39,0x86,0xd5,0x72,0x61,0xda,0xf7,
0x01,0x05,0x8b,0xc3,0xb1,0x77,0xaf,0x1d,0x30,0xc6,0x45,0x0e,
0x5f,0xee,0xae,0xf0,0x28,0xce,0xcd,0xa7,0x9b,0x2a,0x19,0x48,
0x08,0x44,0x20,0xfe,0x6d,0xb5,0x2e,0x6a,0xf1,0x34,0xbc,0x1e,
0x3e,0xcc,0x41,0x92,0xd8,0xbd,0xa5,0xe8,0x4d,0x0a,0x49,0x0d,
0xa2,0xfa,0x62,0x74,0xd4,0x83,0x96,0x94,0x3d,0xcb,0x18,0x63,
0x99,0x46,0xca,0xb7,0x8e,0xcf,0xfb,0xa3,0x6c,0x7e,0x51,0x27,
0x60,0x9a,0x11,0xf3,0x5c,0x6e,0xba,0x42,0x76,0x2f,0xef,0xbf,
0x21,0xaa,0xe4,0xd6,0x1b,0x55,0x7d,0xbe,0xea,0xd3,0x10,0xf4,
0xc7,0x4a,0x23,0x79,0x84,0xa4,0x1c,0xab,0x14,0xdb,0x4c,0x3a,
0xb8,0x52,0xec,0x37,0x38,0xb6,0xd2,0xa0,0x5a,0x5b,0x98,0x66,
0x54,0x9e,0x4e,0x4f,0xb4,0xc4,0xc9,0xd0,0x25,0x9c,0x80,0xde,
0x2d,0x06,0x22,0x0b,0x91,0x6b,0x9f,0xf6,0xe6,0xe2,0xc1,0x0f,
0x93,0x90,0x7b,0x9d,0x8f,0xdd,0xe5,0x65,0x35,0xad,0xa9,0xdc,
0x82,0xbb,0x00,0x53,0xd1,0xa8,0x33,0xe9,0x40,0x1a,0xff,0xa1,
0x95,0x36,0xd9,0xeb,0x89,0xe3,0x7c,0x73,0x85,0x88,0x7a,0xe0,
0xfd,0x64,0x0c,0x57,0x32,0xb3,0xb9,0x1f,0xd7,0xfc,0x81,0xe1,
0x02,0xf9,0x5d,0x56,0x6f,0x24]

v7 = fuck[0]
v8 = fuck[1]

enc = '7635fdf57d47fe95137a26593fff31a1857c63026ebd936a3e4d8dd727732d5ecc62f2dfe5d2'.decode('hex')
flag = ''
for i in range(len(enc)):
	v7 = (v7+1)&0xff
	v3 = fuck[v7+2]
	v8 = (v8+v3)&0xff
	v4 = fuck[2+v8]
	fuck[2+v7] = v4
	fuck[2+v8] = v3
	key = fuck[2+((v3+v4)&0xff)]
	for j in range(0xff+1):
		if j ^ key == ord(enc[i]):
			flag += chr(j)
print(flag)
```



### [GUET-CTF2019]number_game

10位数字，每位只能是0-4

👴寻思直接爆破

```python
from pwn import *
i = 0
while True:
	r = process('./number_game')
	r.sendline(str(i))
	fuck = r.recvline()
	if 'TQL' in fuck:
		print(i)
		break
	i += 1
	r.close()
```



###  [FlareOn4]login

rot13

### [FlareOn6]Overlong

```c
v4 = sub_401160(Text, &unk_402008, 0x1C);
```

把0x1c改大，运行出flag



### [FlareOn4]IgniteMe

```c
signed int check()
{
  int v0; // ST04_4
  int i; // [esp+4h] [ebp-8h]
  unsigned int j; // [esp+4h] [ebp-8h]
  char v4; // [esp+Bh] [ebp-1h]

  v0 = get_len((int)flag);
  v4 = fuck_v4();
  for ( i = v0 - 1; i >= 0; --i )
  {
    res[i] = v4 ^ flag[i];
    v4 = flag[i];
  }
  for ( j = 0; j < 0x27; ++j )
  {
    if ( res[j] != (unsigned __int8)key[j] )
      return 0;
  }
  return 1;
}
//v4 = __ROL4__(0x80070000, 4) >> 1;
//v4 = 0x4;
```



### [BJDCTF2020]BJD hamburger competition

> 一日三餐没烦恼，今天就吃老八秘制小汉堡。即实惠，还管饱。凑豆腐，腐掳，加柠檬，你看这汉堡做的行不行。奥利给，兄弟们，造它就完了

```c#
// ButtonSpawnFruit
// Token: 0x0600000C RID: 12 RVA: 0x000021C8 File Offset: 0x000003C8
public void Spawn()
{
	FruitSpawner component = GameObject.FindWithTag("GameController").GetComponent<FruitSpawner>();
	if (component)
	{
		if (this.audioSources.Length != 0)
		{
			this.audioSources[Random.Range(0, this.audioSources.Length)].Play();
		}
		component.Spawn(this.toSpawn);
		string name = this.toSpawn.name;
		if (name == "汉堡底" && Init.spawnCount == 0)
		{
			Init.secret += 997;
		}
		else if (name == "鸭屁股")
		{
			Init.secret -= 127;
		}
		else if (name == "胡罗贝")
		{
			Init.secret *= 3;
		}
		else if (name == "臭豆腐")
		{
			Init.secret ^= 18;
		}
		else if (name == "俘虏")
		{
			Init.secret += 29;
		}
		else if (name == "白拆")
		{
			Init.secret -= 47;
		}
		else if (name == "美汁汁")
		{
			Init.secret *= 5;
		}
		else if (name == "柠檬")
		{
			Init.secret ^= 87;
		}
		else if (name == "汉堡顶" && Init.spawnCount == 5)
		{
			Init.secret ^= 127;
			string str = Init.secret.ToString();
			if (ButtonSpawnFruit.Sha1(str) == "DD01903921EA24941C26A48F2CEC24E0BB0E8CC7")
			{
				this.result = "BJDCTF{" + ButtonSpawnFruit.Md5(str) + "}";
				Debug.Log(this.result);
			}
		}
		Init.spawnCount++;
		Debug.Log(Init.secret);
		Debug.Log(Init.spawnCount);
	}
}

// ButtonSpawnFruit
// Token: 0x0600000B RID: 11 RVA: 0x00002170 File Offset: 0x00000370
public static string Sha1(string str)
{
	byte[] bytes = Encoding.UTF8.GetBytes(str);
	byte[] array = SHA1.Create().ComputeHash(bytes);
	StringBuilder stringBuilder = new StringBuilder();
	foreach (byte b in array)
	{
		stringBuilder.Append(b.ToString("X2"));
	}
	return stringBuilder.ToString();
}

// ButtonSpawnFruit
// Token: 0x0600000A RID: 10 RVA: 0x00002110 File Offset: 0x00000310
public static string Md5(string str)
{
	byte[] bytes = Encoding.UTF8.GetBytes(str);
	byte[] array = MD5.Create().ComputeHash(bytes);
	StringBuilder stringBuilder = new StringBuilder();
	foreach (byte b in array)
	{
		stringBuilder.Append(b.ToString("X2"));
	}
	return stringBuilder.ToString().Substring(0, 20);
}

```

```python
from hashlib import sha1,md5
for i in range(100000):
	if sha1(str(i)).hexdigest() == 'DD01903921EA24941C26A48F2CEC24E0BB0E8CC7'.lower():
		print(i)
		print(md5(str(i)).hexdigest().upper()[:20])
		break
```

汉堡底，臭豆腐，白拆，白拆，胡罗贝，汉堡顶，你看这汉堡做的行不行。奥利给，兄弟们，造它就完了