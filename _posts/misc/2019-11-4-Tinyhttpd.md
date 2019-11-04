---
layout: post
title:  "Tinyhttpd"
date:   2019-11-4
desc: ""
keywords: ""
categories: [Misc]
tags: []
icon: icon-html
---

# Tinyhttpd

Tinyhttpd 是J. David Blackstone在1999年写的超轻量型 Http Serve

项目地址:https://github.com/nengm/Tinyhttpd

## 踩坑带师

需要安装perl和perl-cgi

如果是wsl，不要把项目放到windows上，因为需要chmod改权限，所以我扔到~/了

make时会报

```
undefined reference to `pthread_create'
collect2: error: ld returned 1 exit status
Makefile:4: recipe for target 'httpd' failed
make: *** [httpd] Error 1
```

改Makefile，加上-lpthread

```
all: httpd client
LIBS = -lpthread #-lsocket
httpd: httpd.c
        gcc -g -W -Wall $(LIBS) -o $@ $< -lpthread

client: simpleclient.c
        gcc -W -Wall -o $@ $<
clean:
        rm httpd
```

htdoc中，index.html不能有可执行权限，否则不能正常显示，check.cgi和color.cgi必须有可执行权限，否则输入颜色后没反应



## 源码阅读

