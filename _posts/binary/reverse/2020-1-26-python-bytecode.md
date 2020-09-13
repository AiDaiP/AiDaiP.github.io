---
clayout: post
title:  "python-bytecode"
date:   2020-1-26
desc: ""
keywords: ""
categories: [Binary]
tags: []
icon: icon-html
---

# python-bytecode

```
官方文档:https://docs.python.org/2/library/dis.html
```

### 栈基本操作

| TOS    |
| ------ |
| TOS1   |
| TOS2   |
| ...... |

* LOAD_NAME

  压栈

  ```
  LOAD_GLOBAL:全局变量
  LOAD_FAST:局部变量
  LOAD_CONST:常量
  LOAD_ATTR:对象中的属性
  ```

* POP_TOP

  ```
  pop
  ```

* ROT_TWO

  ```
  TOS1，TOS1互换
  ```

* ROT_THREE

  ```
  栈顶三元素循环位移
  1->3
  ```

* ROT_FOUR

  ```
  栈顶四元素循环位移
  1->4
  ```

* DUP_TOP

  ```
  复制栈顶元素压栈
  ```

* DUP_TOPX(count)

  ```
  复制栈顶count个元素再按原顺序压栈 count范围[1,5]
  ```



### UNARY

栈顶元素弹出去，运算后压栈

* UNARY_NEGATIVE

  取负

  ```
  TOS = -TOS
  ```

* UNARY_NOT

  逻辑取反

  ```
  TOS = not TOS
  ```
  
* UNARY_INVERT

  按位取反

  ```
  TOS = ~ TOS
  ```



### BINARY

栈顶两元素弹出去，运算后压栈

* BINARY_ADD

  加法

  ```
  TOS = TOS1 + TOS
  ```

* BINARY_SUBTRACT

  减法

  ```
  TOS = TOS1 - TOS
  ```

* BINARY_MULTIPLY

  乘法

  ```
  TOS = TOS1 * TOS
  ```

* BINARY_DIVIDE

  除法

  ```
  TOS = TOS1 / TOS
  ```

* BINARY_MODULO

  取模

  ```
  TOS = TOS1 % TOS
  ```

* BINARY_POWER

  幂运算

  ```
   TOS = TOS1 ** TOS
  ```

* BINARY_LSHIFT

  左移

  ```
  TOS = TOS1 << TOS
  ```

* BINARY_RSHIFT

  右移

  ```
  TOS = TOS1 >> TOS
  ```

* BINARY_AND

  与

  ```
  TOS = TOS1 & TOS
  ```

* BINARY_OR

  或

  ```
  TOS = TOS1 | TOS
  ```

* BINARY_XOR

  异或

  ```
  TOS = TOS1 ^ TOS
  ```

* BINARY_SUBSCR

  索引

  ```
  TOS = TOS1[TOS]
  ```



### INPLACE

栈顶两元素弹出去，运算后压栈

```python
def test(a):
	a +=1
	a = a + 1

dis.dis(test)
```

```
  6           0 LOAD_FAST                0 (a)
              3 LOAD_CONST               1 (1)
              6 INPLACE_ADD
              7 STORE_FAST               0 (a)

  7          10 LOAD_FAST                0 (a)
             13 LOAD_CONST               1 (1)
             16 BINARY_ADD
             17 STORE_FAST               0 (a)
             20 LOAD_CONST               0 (None)
             23 RETURN_VALUE
```



### BUILD

* BUILD_CLASS

  ```
  利用当前栈上信息创建类
  ```

* BUILD_TUPLE(count)

  ```
  从栈顶取count个元素，创建一个tuple对象压栈
  ```

* BUILD_LIST(count)

  ```
  从栈顶取count个元素，创建一个list对象压栈
  ```

* BUILD_SET(count)

  ```
  从栈顶取count个元素，创建一个set对象压栈
  ```

* BUILD_MAP(count)

  ```
  从栈顶取count个元素，创建一个map对象压栈
  ```

* BUILD_SLICE

  ```
  创建一个SLICE对象，保存start,stop,step三个参数
  ```

### SLICE

* SLICE+0

  ```python
  def test(a):
  	a[:]
  
  ```

  ```
    6           0 LOAD_FAST                0 (a)
                3 SLICE+0
                4 POP_TOP
                5 LOAD_CONST               0 (None)
                8 RETURN_VALUE
  ```

  

* SLICE+1

  ```python
  def test(a):
  	a[1:]
  
  dis.dis(test)
  ```

  ```
    6           0 LOAD_FAST                0 (a)
                3 LOAD_CONST               1 (1)
                6 SLICE+1
                7 POP_TOP
                8 LOAD_CONST               0 (None)
               11 RETURN_VALUE
  ```

  

* SLICE+2

  ```python
  def test(a):
  	a[:1]
  
  dis.dis(test)
  ```

  ```
    6           0 LOAD_FAST                0 (a)
                3 LOAD_CONST               1 (1)
                6 SLICE+2
                7 POP_TOP
                8 LOAD_CONST               0 (None)
               11 RETURN_VALUE
  ```

  

* SLICE+3

  ```python
  def test(a):
  	a[1:2]
  
  dis.dis(test)
  ```

  ```
    6           0 LOAD_FAST                0 (a)
                3 LOAD_CONST               1 (1)
                6 LOAD_CONST               2 (2)
                9 SLICE+3
               10 POP_TOP
               11 LOAD_CONST               0 (None)
               14 RETURN_VALUE
  ```

* BUILD_SLICE

  ```python
  def test(a):
  	a[1:2:3]
  
  dis.dis(test)
  ```

  ```
    6           0 LOAD_FAST                0 (a)
                3 LOAD_CONST               0 (None)
                6 LOAD_CONST               0 (None)
                9 LOAD_CONST               1 (3)
               12 BUILD_SLICE              3
               15 BINARY_SUBSCR
               16 POP_TOP
               17 LOAD_CONST               0 (None)
               20 RETURN_VALUE
  ```

  

### STORE

栈顶弹出去赋值

```python
def test(a):
	a = 1

dis.dis(test)
```

```
  6           0 LOAD_CONST               1 (1)
              3 STORE_FAST               0 (a)
              6 LOAD_CONST               0 (None)
              9 RETURN_VALUE
```

```
STORE_GLOBAL:全局变量
STORE_FAST:局部变量
STORE_ATTR:对象中的属性
```



### DELETE

删除

* DELETE_NAME

  ```python
  def test(a):
  	del a
  
  dis.dis(test)
  ```

  ```
    6           0 DELETE_FAST              0 (a)
                3 LOAD_CONST               0 (None)
                6 RETURN_VALUE
  ```

  ```
  DELETE_GLOBAL:全局变量
  DELETE_FAST:局部变量
  DELETE_ATTR:对象中的属性
  DELETE_SLICE+():SLICE
  ```

  

### 跳转

* COMPARE_OP(op)

  对栈顶的两个元素做op指定的比较操作，结果压栈

  ```
  def test(a,b):
  	a>b
  dis.dis(test)
  ```

  ```
    6           0 LOAD_FAST                0 (a)
                3 LOAD_FAST                1 (b)
                6 COMPARE_OP               4 (>)
                9 POP_TOP
               10 LOAD_CONST               0 (None)
               13 RETURN_VALUE
  ```

* POP_JUMP_IF_TRUE(target)

  栈顶弹出，如果为true跳转到target

  ```python
  def test(a,b):
  	if not a>b:
  		pass
  
  dis.dis(test)
  ```

  ```
    6           0 LOAD_FAST                0 (a)
                3 LOAD_FAST                1 (b)
                6 COMPARE_OP               4 (>)
                9 POP_JUMP_IF_TRUE        15
  
    7          12 JUMP_FORWARD             0 (to 15)
          >>   15 LOAD_CONST               0 (None)
               18 RETURN_VALUE
  ```

  

* POP_JUMP_IF_FALSE(target)

  栈顶弹出，如果为false跳转到target

  ```python
  def test(a,b):
  	if a>b:
  		pass
  
  dis.dis(test)
  ```

  ```
    6           0 LOAD_FAST                0 (a)
                3 LOAD_FAST                1 (b)
                6 COMPARE_OP               4 (>)
                9 POP_JUMP_IF_FALSE       15
  
    7          12 JUMP_FORWARD             0 (to 15)
          >>   15 LOAD_CONST               0 (None)
               18 RETURN_VALUE
  ```

* JUMP_IF_TRUE_OR_POP(target)

  ```
  如果栈顶是true跳转到target，保留栈顶。否则栈顶弹出
  ```

  

* JUMP_IF_FALSE_OR_POP

  ```
  如果栈顶是false跳转到target，保留栈顶。否则栈顶弹出
  ```

  

* JUMP_FORWARD(x)

  ```
  向前跳转x字节
  ```



### 循环

* for 循环

  ```python
  def test(a):
  	for i in range(10):
  		pass
  
  dis.dis(test)
  ```

  ```
    6           0 SETUP_LOOP              20 (to 23)
                3 LOAD_GLOBAL              0 (range)
                6 LOAD_CONST               1 (10)
                9 CALL_FUNCTION            1
               12 GET_ITER
          >>   13 FOR_ITER                 6 (to 22)
               16 STORE_FAST               1 (i)
  
    7          19 JUMP_ABSOLUTE           13
          >>   22 POP_BLOCK
          >>   23 LOAD_CONST               0 (None)
               26 RETURN_VALUE
  ```

* while 循环

  ```python
  def test(a):
  	while True:
  		pass
  
  dis.dis(test)
  ```

  ```
    6           0 SETUP_LOOP              10 (to 13)
          >>    3 LOAD_GLOBAL              0 (True)
                6 POP_JUMP_IF_FALSE       12
  
    7           9 JUMP_ABSOLUTE            3
          >>   12 POP_BLOCK
          >>   13 LOAD_CONST               0 (None)
               16 RETURN_VALUE
  ```

  

* GET_ITER

  ```
  取迭代器
  ```

* BREAK_LOOP

  ```
  break
  取pytry_block中的handler获取循环结束后的下一条指令地址,跳出循环
  ```

  

### 函数调用

* CALL_FUNCTION

  调用函数，调用前将pyfuntionobject和实参压栈

  ```python
  def test(a,b,c):
  	pass
  def test2():
  	test(1,2,3)
  dis.dis(test2)
  ```

  ```
    8           0 LOAD_GLOBAL              0 (test)
                3 LOAD_CONST               1 (1)
                6 LOAD_CONST               2 (2)
                9 LOAD_CONST               3 (3)
               12 CALL_FUNCTION            3
               15 POP_TOP
               16 LOAD_CONST               0 (None)
               19 RETURN_VALUE
  ```

  

  ```python
  def test(a,b):
  	dis.dis(a)
  
  dis.dis(test)
  ```

  ```
    6           0 LOAD_GLOBAL              0 (dis)
                3 LOAD_ATTR                0 (dis)
                6 LOAD_FAST                0 (a)
                9 CALL_FUNCTION            1
               12 POP_TOP
               13 LOAD_CONST               0 (None)
               16 RETURN_VALUE
  ```



### 返回值

* RETURN_VALUE

  ```
  栈顶元素作为返回值
  ```



### 打印

* PRINT_ITEM

  ```
  打印栈顶元素到标准输出
  ```

* PRINT_NEWLINE

  ```
  打印回车到标准输出
  ```

  ```python
  def test(a):
  	print(a)
  
  dis.dis(test)
  ```

  ```
    6           0 LOAD_FAST                0 (a)
                3 PRINT_ITEM
                4 PRINT_NEWLINE
                5 LOAD_CONST               0 (None)
                8 RETURN_VALUE
  ```

  

### NOP

* NOP

  ```
  nop
  ```

  







