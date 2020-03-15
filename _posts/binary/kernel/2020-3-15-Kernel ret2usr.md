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

上一篇rop执行的commit_creds(prepare_kernel_cred(0))是在内核空间，如果内核空间没有commit_creds和prepare_kernel_cred，或者没有好使的gadget，或者搞不到地址，这种方法就不好使

在用户空间构造一个commit_creds(prepare_kernel_cred(0))很方便，ret2usr就是返回到用户空间的commit_creds(prepare_kernel_cred(0))

ROP链

1. ROP执行用户空间的commit_creds(prepare_kernel_cred(0))
2. 返回到用户态执行system("/bin/sh")

但是有个叫SMEP的东西，这玩意开了禁止内核执行用户空间的代码

那就直接给ret2usr🐏了🐎？

👴选择给SMEP🐏了

判断是否开启smep是根据CR4寄存器的第20位，保护开启值为1，关闭值为0

来个mov cr4, ?就给他🐏了



## 强网杯2018-core

1

## CISCN2017 babydriver

1