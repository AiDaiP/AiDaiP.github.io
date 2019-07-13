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
  2. 攻击者能够触发密文的解密过程，且能够知道解密的结果是否符合Padding

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

  * ##### 白给样例

    ```python
    from Crypto.Cipher import DES
    from Crypto import Random
    def encrypt(key, raw):
        iv = Random.new().read(DES.block_size)
        cipher = DES.new(key, DES.MODE_CBC, iv)
        return iv + cipher.encrypt(raw.encode("utf-8"))
    
    def decrypt(key, enc):
        iv = enc[:DES.block_size]
        cipher = DES.new(key, DES.MODE_CBC, iv)
        return cipher.decrypt(enc[DES.block_size:]).decode("utf-8")
    
    ```

    

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

    以此类推，得到中间值，中间值与针正的$C_{i-1}$异或得到明文$P_i$

    去掉最后一组密文，只提交第一组到倒数第二组密文，通过构造倒数第三组密文得到倒数第二组密文的明文，以此类推，最终获得全部明文

  * ##### 构造任意明文的密文

    ![5](D:\Ai\GitHub\images\Padding Oracle Attack\5.png)

    我寻思这和解密任意明文里构造合法填充差不多一个意思

    搞出来中间值然后异或就完事了

    从最后一个数据块开始，向前依次生成所需的密文

* #### 防御方法

  俺不让你知道解密的结果是否符合Padding 