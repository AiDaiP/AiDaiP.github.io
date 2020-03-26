---
layout: post
title:  "Kernel userfaultfd"
date:   2020-3-26
desc: ""
keywords: ""
categories: [Binary]
tags: []
icon: icon-html
---

# Kernel userfaultfd

## 缺页

内核为每个进程都维护了一张页表，记录虚拟地址和物理地址的映射关系，页表储存在MMU中。CPU发出的访问地址是虚拟地址，通过MMU转换为物理地址，通过地址总线访问连接的所有外设

当CPU访问的虚拟地址在MMU的页表中查不到时，就会产生缺页错误

缺页错误分为硬性页错误和软性页错误

* 硬性页错误

  相关的页在页缺失发生时未被加载进内存

  异常处理：把数据写入选定的页，然后在MMU建立映射关系

* 软性页错误

  相关的页在页缺失发生时已经被加载进内存，但是没有建立映射关系，比如mmap创建的内存映射页

  异常处理：MMU建立映射关系

##userfaultfd

用户态的缺页处理机制，可以在用户空间定义Page Fault Hander

[文档](http://man7.org/linux/man-pages/man2/userfaultfd.2.html)

```c
/* userfaultfd_demo.c

  Licensed under the GNU General Public License version 2 or later.
*/
#define _GNU_SOURCE
#include <sys/types.h>
#include <stdio.h>
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
#include <pthread.h>

#define errExit(msg)    do { perror(msg); exit(EXIT_FAILURE); \
                       } while (0)

static int page_size;

static void *
fault_handler_thread(void *arg)
{
   static struct uffd_msg msg;   /* Data read from userfaultfd */
   static int fault_cnt = 0;     /* Number of faults so far handled */
   long uffd;                    /* userfaultfd file descriptor */
   static char *page = NULL;
   struct uffdio_copy uffdio_copy;
   ssize_t nread;

   uffd = (long) arg;

   /* Create a page that will be copied into the faulting region */

   if (page == NULL) {
       page = mmap(NULL, page_size, PROT_READ | PROT_WRITE,
                   MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
       if (page == MAP_FAILED)
           errExit("mmap");
   }

   /* Loop, handling incoming events on the userfaultfd
      file descriptor */

   for (;;) {

       /* See what poll() tells us about the userfaultfd */

       struct pollfd pollfd;
       int nready;
       pollfd.fd = uffd;
       pollfd.events = POLLIN;
       nready = poll(&pollfd, 1, -1);
       if (nready == -1)
           errExit("poll");

       printf("\nfault_handler_thread():\n");
       printf("    poll() returns: nready = %d; "
               "POLLIN = %d; POLLERR = %d\n", nready,
               (pollfd.revents & POLLIN) != 0,
               (pollfd.revents & POLLERR) != 0);

       /* Read an event from the userfaultfd */

       nread = read(uffd, &msg, sizeof(msg));
       if (nread == 0) {
           printf("EOF on userfaultfd!\n");
           exit(EXIT_FAILURE);
       }

       if (nread == -1)
           errExit("read");

       /* We expect only one kind of event; verify that assumption */

       if (msg.event != UFFD_EVENT_PAGEFAULT) {
           fprintf(stderr, "Unexpected event on userfaultfd\n");
           exit(EXIT_FAILURE);
       }

       /* Display info about the page-fault event */

       printf("    UFFD_EVENT_PAGEFAULT event: ");
       printf("flags = %llx; ", msg.arg.pagefault.flags);
       printf("address = %llx\n", msg.arg.pagefault.address);

       /* Copy the page pointed to by 'page' into the faulting
          region. Vary the contents that are copied in, so that it
          is more obvious that each fault is handled separately. */

       memset(page, 'A' + fault_cnt % 20, page_size);
       fault_cnt++;

       uffdio_copy.src = (unsigned long) page;

       /* We need to handle page faults in units of pages(!).
          So, round faulting address down to page boundary */

       uffdio_copy.dst = (unsigned long) msg.arg.pagefault.address &
                                          ~(page_size - 1);
       uffdio_copy.len = page_size;
       uffdio_copy.mode = 0;
       uffdio_copy.copy = 0;
       if (ioctl(uffd, UFFDIO_COPY, &uffdio_copy) == -1)
           errExit("ioctl-UFFDIO_COPY");

       printf("        (uffdio_copy.copy returned %lld)\n",
               uffdio_copy.copy);
   }
}

int
main(int argc, char *argv[])
{
   long uffd;          /* userfaultfd file descriptor */
   char *addr;         /* Start of region handled by userfaultfd */
   unsigned long len;  /* Length of region handled by userfaultfd */
   pthread_t thr;      /* ID of thread that handles page faults */
   struct uffdio_api uffdio_api;
   struct uffdio_register uffdio_register;
   int s;

   if (argc != 2) {
       fprintf(stderr, "Usage: %s num-pages\n", argv[0]);
       exit(EXIT_FAILURE);
   }

   page_size = sysconf(_SC_PAGE_SIZE);
   len = strtoul(argv[1], NULL, 0) * page_size;

   /* Create and enable userfaultfd object */

   uffd = syscall(__NR_userfaultfd, O_CLOEXEC | O_NONBLOCK);
   if (uffd == -1)
       errExit("userfaultfd");

   uffdio_api.api = UFFD_API;
   uffdio_api.features = 0;
   if (ioctl(uffd, UFFDIO_API, &uffdio_api) == -1)
       errExit("ioctl-UFFDIO_API");

   /* Create a private anonymous mapping. The memory will be
      demand-zero paged--that is, not yet allocated. When we
      actually touch the memory, it will be allocated via
      the userfaultfd. */

   addr = mmap(NULL, len, PROT_READ | PROT_WRITE,
               MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
   if (addr == MAP_FAILED)
       errExit("mmap");

   printf("Address returned by mmap() = %p\n", addr);

   /* Register the memory range of the mapping we just created for
      handling by the userfaultfd object. In mode, we request to track
      missing pages (i.e., pages that have not yet been faulted in). */

   uffdio_register.range.start = (unsigned long) addr;
   uffdio_register.range.len = len;
   uffdio_register.mode = UFFDIO_REGISTER_MODE_MISSING;
   if (ioctl(uffd, UFFDIO_REGISTER, &uffdio_register) == -1)
       errExit("ioctl-UFFDIO_REGISTER");

   /* Create a thread that will process the userfaultfd events */

   s = pthread_create(&thr, NULL, fault_handler_thread, (void *) uffd);
   if (s != 0) {
       errno = s;
       errExit("pthread_create");
   }

   /* Main thread now touches memory in the mapping, touching
      locations 1024 bytes apart. This will trigger userfaultfd
      events for all pages in the region. */

   int l;
   l = 0xf;    /* Ensure that faulting address is not on a page
                  boundary, in order to test that we correctly
                  handle that case in fault_handling_thread() */
   while (l < len) {
       char c = addr[l];
       printf("Read address %p in main(): ", addr + l);
       printf("%c\n", c);
       l += 1024;
       usleep(100000);         /* Slow things down a little */
   }

   exit(EXIT_SUCCESS);
}
```

写exp可以分成两部分

```c
#define _GNU_SOURCE
#include <sys/types.h>
#include <stdio.h>
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

#define FAULT_PAGE ((void*)(0x1337000))

void errExit(char* msg)
{
    puts(msg);
    exit(-1);
}


static void * fault_handler_thread(void *arg)
{
  //轮询uffd得到的信息储存在msg中
   static struct uffd_msg msg中;
   static int fault_cnt = 0;
   long uffd; 
   static char *page = NULL;
   struct uffdio_copy uffdio_copy;//UFFDIO_COPY
   ssize_t nread;

   int page_size = 0x1000;
   
   uffd = (long) arg;
   if (page == NULL) {
       page = mmap(NULL, page_size, PROT_READ | PROT_WRITE,
                   MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
       if (page == MAP_FAILED)
           errExit("mmap");
   }
   for (;;) {//poll，所以死循环
       struct pollfd pollfd;//poll
       int nready;
       pollfd.fd = uffd;
       pollfd.events = POLLIN;
       nready = poll(&pollfd, 1, -1);//等待copy_from_user访问addr

       if (nready == -1)
           errExit("poll");
         //copy_from_user访问addr产生缺页错误，停在copy_from_user直到handler执行结束
         //从这里开始可以进行利用
       printf("\nfault_handler_thread():\n");
       printf("    poll() returns: nready = %d; "
               "POLLIN = %d; POLLERR = %d\n", nready,
               (pollfd.revents & POLLIN) != 0,
               (pollfd.revents & POLLERR) != 0);
       nread = read(uffd, &msg, sizeof(msg));
       if (nread == 0) {
           printf("EOF on userfaultfd!\n");
           exit(EXIT_FAILURE);
       }

       if (nread == -1)
           errExit("read");
       if (msg.event != UFFD_EVENT_PAGEFAULT) {
           fprintf(stderr, "Unexpected event on userfaultfd\n");
           exit(EXIT_FAILURE);
       }

       printf("    UFFD_EVENT_PAGEFAULT event: ");
       printf("flags = %llx; ", msg.arg.pagefault.flags);
       printf("address = %llx\n", msg.arg.pagefault.address);
       memset(page, 'A' + fault_cnt % 20, page_size);
       fault_cnt++;

       uffdio_copy.src = (unsigned long) page;

       uffdio_copy.dst = (unsigned long) msg.arg.pagefault.address &
                                          ~(page_size - 1);
       uffdio_copy.len = page_size;
       uffdio_copy.mode = 0;
       uffdio_copy.copy = 0;
       if (ioctl(uffd, UFFDIO_COPY, &uffdio_copy) == -1)
           errExit("ioctl-UFFDIO_COPY");

       printf("        (uffdio_copy.copy returned %lld)\n",
               uffdio_copy.copy);
   }
}

size_t register_userfault(){
   long uffd;        
   char *addr;       
   unsigned long len = 0x1000;
   pthread_t thr; 
   struct uffdio_api uffdio_api;
   struct uffdio_register uffdio_register;
   int s;
   uffd = syscall(__NR_userfaultfd, O_CLOEXEC | O_NONBLOCK);//创建并返回一个错误处理fd
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
   //访问addr产生页错误
   uffdio_register.range.start = (unsigned long) addr;
   uffdio_register.range.len = len;
   uffdio_register.mode = UFFDIO_REGISTER_MODE_MISSING;
   if (ioctl(uffd, UFFDIO_REGISTER, &uffdio_register) == -1)
       errExit("ioctl-UFFDIO_REGISTER"); //注册页地址与uffd，访问addr时产生页错误，访问挂起，uffd收到信号

   s = pthread_create(&thr, NULL, fault_handler_thread, (void *) uffd);//进handler处理
   if (s != 0) {
       errno = s;
       errExit("pthread_create");
   }
   return addr;
}

```

## 利用方法

```c
copy_from_user(kernel_buf,user_buf,size);  
```

user_buf是mmap返回的地址，没有初始化，执行copy_from_user时触发缺页错误，copy_from_user挂起，此时开另一个线程把kernel_buf释放，再把其他结构申请过来