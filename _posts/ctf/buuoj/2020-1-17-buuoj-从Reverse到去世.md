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

