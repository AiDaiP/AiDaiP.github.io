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
        s = 'fo`guci';
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

