---
layout: post
title:  "Docker配置sqli-labs"
date:   2019-10-27
desc: ""
keywords: ""
categories: [Misc]
tags: []
icon: icon-html
---

# Docker配置sqli-labs

高版本的php淘汰了mysql_connect()

在执行php setup-db.php时会报错

```
php setup-db.php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>SETUP DB</title>
</head>

<body bgcolor="#000000">

<div style=" margin-top:20px;color:#FFF; font-size:24px; text-align:center">
Welcome&nbsp;&nbsp;&nbsp;
<font color="#FF0000"> Dhakkan </font>
<br>
</div>

<div style=" margin-top:10px;color:#FFF; font-size:23px; text-align:left">
<font size="3" color="#FFFF00">
SETTING UP THE DATABASE SCHEMA AND POPULATING DATA IN TABLES:
<br><br>



PHP Fatal error:  Uncaught Error: Call to undefined function mysql_connect() in /var/www/html/sqli-labs/sql-connections/setup-db.php:29
Stack trace:
#0 {main}
  thrown in /var/www/html/sqli-labs/sql-connections/setup-db.php on line 29
```

所以使用docker配置