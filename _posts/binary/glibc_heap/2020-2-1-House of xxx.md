---
clayout: post
title:  "House of xxx"
date:   2020-2-1
desc: ""
keywords: ""
categories: [Binary]
tags: [pwn]
icon: icon-html
---

# House of xxx

👴之前没记录过House of xxx，现在随便写写

### House of Force

* 原理

  把top chunk的size改大，malloc时从top chunk切出来，然后main_arena中的top chunk指针根据size改变，如果malloc的size👴能控制，👴就能让top chunk去👴想让它去的地方，下一次malloc就到了👴想控制的地方

* 条件

  1. 👴能控制top chunk的size

     直接改成-1，验size的时候-1转换成无符号数，所以👴申请的size必能过验证

  2. 👴能自由控制分配的size



### house of spirit

* 原理

  在👴想控制的地方伪造一个fake fastbin，把它free，它就进了fastbin链，下次malloc就到了这个fake fastbin

  

* 条件

  1. 👴能伪造fake fastbin，目标区域的前后都是可控的，前面伪造size，后面伪造下一个chunk的size
  2. 👴能控制free的指针