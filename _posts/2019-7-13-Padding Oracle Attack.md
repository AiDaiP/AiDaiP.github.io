---
layout: post
title:  "Padding Oracle Attack"
date:   2019-7-13
desc: ""
keywords: ""
categories: [Crypto]
tags: [Crypto]
icon: icon-html
---

# Padding Oracle Attack

* #### Padding Oracle Attack

  针对加密算法中的CBC Mode，与具体的加密算法无关。满足攻击条件时，可以在不知道密钥的情况下，解密任意密文或构造出任意明文的密文

* #### 条件

  1. 攻击者能够获得密文，以及密文对应的IV

  2. 攻击者能够触发密文的解密过程，且能够知道密文的解密结果

     * 密文不能正常解密
     * 密文可以正常解密但解密结果错误
     * 密文可以正常解密并且解密结果正确

     可以判断出密文不能正常解密也就是填充不合法即可

* #### 攻击原理

  * ##### CBC

    - 加密：

      $ C_1 = Enc(XOR(IV, P_1) $

      $C_i = Enc(XOR(C_{i-1}, P_i) $

    - 解密

      $P_1 = XOR(IV, Dec(C_1)) $

      $ P_i = XOR(C_{i-1}, Dec(C_i)) $

  * ##### PKCS7Padding

    假设数据长度需要填充n(n>0)个字节才对齐，那么填充n个字节，每个字节都是n;如果数据本身就已经对齐了，则填充一块长度为块大小的数据，每个字节都是块大小。 

  * ##### 解密任意明文

    若已知$C_{i-1}$和中间值$Dec(C_i)$就可以绕过加密算法获得明文

    若$XOR(C_{i-1}, Dec(C_i))$不符合PKCS7Padding填充规则，则返回解密失败，此时可以构造IV也就是前一组密文来获取中间值

    ![3](D:\Ai\GitHub\images\Padding Oracle Attack\3.png)

    ![1](D:\Ai\GitHub\images\Padding Oracle Attack\1.png)

    枚举$C_{i-1}$最后一字节，解密，直到得到合法填充，返回解密成功，即$XOR(C_{i-1}, Dec(C_i))$最后一字节为0x1

    $C_{i-1}$最后一字节与0x1异或可以得到中间值的最后一字节

    中间值的最后一字节最后一字节与0x2异或，得到下一轮构造的$C_{i-1}$最后一字节。

    ![4](D:\Ai\GitHub\images\Padding Oracle Attack\4.png)

    ![2](D:\Ai\GitHub\images\Padding Oracle Attack\2.png)

    枚举$C_{i-1}$倒数第二字节，解密，直到得到合法填充，返回解密成功，即$XOR(C_{i-1}, Dec(C_i))$最后两字节为0x2，0x2

    $C_{i-1}$倒数第二字节与0x2异或可以得到中间值倒数第二字节

    以此类推，得到中间值，中间值与针正的$C_{i-1}$异或得到明文

    

    

    

    