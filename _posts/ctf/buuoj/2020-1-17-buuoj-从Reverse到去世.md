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

```
code = ['\x1f', '\x12', '\x1d', '(', '0', '4', '\x01', '\x06', '\x14', '4', ',', '\x1b', 'U', '?', 'o', '6', '*', ':', '\x01', 'D', ';', '%', '\x13']
code = code[::-1]
for i in range(len(code)-1):
    code[i+1] = chr(ord(code[i+1])^ord(code[i]))
code = code[::-1]
for i in range(len(code)):
    code[i] = chr((ord(code[i]) - i)%128)
print(''.join(code))
```

