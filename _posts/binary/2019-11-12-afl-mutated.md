---
clayout: post
title:  "afl-mutated"
date:   2019-11-12
desc: ""
keywords: ""
categories: [Binary]
tags: [afl]
icon: icon-html
---

# afl-mutated

```
bit flips
byte flips
arithmetics
known ints
dictionary
havoc
splice
```

前五项无随机性，后两项有随机性

## bit flips

按位翻转，1变为0，0变为1

根据翻转量/步长进行多种不同的变异，依次为

1. bitflip 1/1，每次翻转1bit，按照1bit的步长从头开始
2. bitflip 2/1，每次翻转2bit，按照1bit的步长从头开始
3. bitflip 4/1，每次翻转4bit，按照1bit的步长从头开始

### 自动检测token

bitflip 1/1时，自动检测token

如果连续多个byte的最低位被翻转后，执行路径都没有变化且与原始路径不同，就把这一段连续byte判断为token

##byte flips

按字节进行按位反转

根据翻转量/步长进行多种不同的变异，依次为

1. bitflip 8/8，每次翻转1byte，按照1byte步长从头开始
2. bitflip 16/8，每次翻转2byte，按照1byte步长从头开始
3. bitflip 32/8，每次翻转4byte，按照1byte步长从头开始

### effector map

bitflip 8/8时，生成effector map

```
/* Effector map setup. These macros calculate:                        设置效应地图：
   EFF_APOS      - position of a particular file offset in the map.   文件偏移
   EFF_ALEN      - length of a map with a particular number of bytes. 特殊字符的长度
   EFF_SPAN_ALEN - map span for a sequence of bytes.                  一个字节序列的映射
   */
#define EFF_APOS(_p)          ((_p) >> EFF_MAP_SCALE2)
#define EFF_REM(_x)           ((_x) & ((1 << EFF_MAP_SCALE2) - 1))
#define EFF_ALEN(_l)          (EFF_APOS(_l) + !!EFF_REM(_l))
#define EFF_SPAN_ALEN(_p, _l) (EFF_APOS((_p) + (_l) - 1) - EFF_APOS(_p) + 1)
```

在对每个byte进行翻转时，如果翻转后造成执行路径与原始路径不同，就将该byte在effector map中标记为1，有效，否则标记为0，无效

如果一个byte完全翻转不影响执行路径，那么这个byte对整个fuzzing意义不大，后续的变异中会跳过这些byte

## arithmetics

按字节进行整数加减运算

根据运算量/步长进行多种不同的变异，依次为

1. arithmetics 8/8，每次对1byte进行加减运算，按照1byte步长从头开始
2. arithmetics 16/8，每次对2byte进行加减运算，按照1byte步长从头开始
3. arithmetics 32/8，每次对4byte进行加减运算，按照1byte步长从头开始

config.h中

```c
#define ARITH_MAX           35
```

宏定义上限，也就是+1,+2...+35,-1,-2...-35



## known ints

替换字节

根据替换量/步长进行多种不同的变异，依次为

1. interest 8/8，每次对1byte进行替换，按照1byte步长从头开始

   ```c
   out_buf[i] = interesting_8[j]
   ```

2. interest 16/8，每次对2byte进行替换，按照1byte步长从头开始

   ```c
   *(u16*)(out_buf + i) = interesting_16[j];
   ```

3. interest 32/8，每次对4byte进行替换，按照1byte步长从头开始

   ```c
   *(u32*)(out_buf + i) = interesting_32[j];
   ```



```c
static s8  interesting_8[]  = { INTERESTING_8 };
static s16 interesting_16[] = { INTERESTING_8, INTERESTING_16 };
static s32 interesting_32[] = { INTERESTING_8, INTERESTING_16, INTERESTING_32 };
```

config.h中

```c
/* List of interesting values to use in fuzzing. */

#define INTERESTING_8 \
  -128,          /* Overflow signed 8-bit when decremented  */ \
  -1,            /*                                         */ \
   0,            /*                                         */ \
   1,            /*                                         */ \
   16,           /* One-off with common buffer size         */ \
   32,           /* One-off with common buffer size         */ \
   64,           /* One-off with common buffer size         */ \
   100,          /* One-off with common buffer size         */ \
   127           /* Overflow signed 8-bit when incremented  */

#define INTERESTING_16 \
  -32768,        /* Overflow signed 16-bit when decremented */ \
  -129,          /* Overflow signed 8-bit                   */ \
   128,          /* Overflow signed 8-bit                   */ \
   255,          /* Overflow unsig 8-bit when incremented   */ \
   256,          /* Overflow unsig 8-bit                    */ \
   512,          /* One-off with common buffer size         */ \
   1000,         /* One-off with common buffer size         */ \
   1024,         /* One-off with common buffer size         */ \
   4096,         /* One-off with common buffer size         */ \
   32767         /* Overflow signed 16-bit when incremented */

#define INTERESTING_32 \
  -2147483648LL, /* Overflow signed 32-bit when decremented */ \
  -100663046,    /* Large negative number (endian-agnostic) */ \
  -32769,        /* Overflow signed 16-bit                  */ \
   32768,        /* Overflow signed 16-bit                  */ \
   65535,        /* Overflow unsig 16-bit when incremented  */ \
   65536,        /* Overflow unsig 16 bit                   */ \
   100663045,    /* Large positive number (endian-agnostic) */ \
   2147483647    /* Overflow signed 32-bit when incremented */

```

宏定义interesting values



## dictionary

字典替换或插入，来源为用户提供的tokens和自动检测生成的tokens，依次为

1. user extras (over)，从头开始，将用户提供的tokens依次替换到原文件中
2. user extras (insert)，从头开始，将用户提供的tokens依次插入到原文件中
3. auto extras (over)，从头开始，将自动检测的tokens依次替换到原文件中

如果用户使用字典，使用-x指定字典文件或字典目录



## havoc

各种随机生成的变异

包含了对原文件的多轮变异，每一轮都是将多种方式组合而成

```
随机选取某个bit进行翻转
随机选取某个byte，将其设置为随机的interesting value
随机选取某个word，并随机选取大、小端序，将其设置为随机的interesting value
随机选取某个dword，并随机选取大、小端序，将其设置为随机的interesting value
随机选取某个byte，减去一个随机数
随机选取某个byte，加上一个随机数
随机选取某个word，并随机选取大、小端序，减去一个随机数
随机选取某个word，并随机选取大、小端序，加上一个随机数
随机选取某个dword，并随机选取大、小端序，减去一个随机数
随机选取某个dword，并随机选取大、小端序，加上一个随机数
随机选取某个byte，将其设置为随机数
随机删除一段bytes
随机选取一个位置，插入一段随机长度的内容，其中75%的概率是插入原文中随机位置的内容，25%的概率是插入一段随机内容
随机选取一个位置，替换为一段随机长度的内容，其中75%的概率是替换为原文中随机位置的内容，25%的概率是替换为一段随机内容
随机选取一个位置，用随机选取的token替换
随机选取一个位置，用随机选取的token插入
```



## splice

将两个seed文件拼接得到新的文件，并对新文件继续执行havoc变异