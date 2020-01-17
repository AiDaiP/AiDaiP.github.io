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

