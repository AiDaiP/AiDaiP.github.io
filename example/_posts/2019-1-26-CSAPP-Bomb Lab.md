# CSAPP-Bomb Lab

刚看到这东西有个大胆的想法，直接IDA pro F5

我就是饿死，死外面，从这跳下去，也不会用IDA pro

* phase_1 

  ```assembly
  08048b20 <phase_1>:
   8048b20:	55                   	push   %ebp
   8048b21:	89 e5                	mov    %esp,%ebp
   8048b23:	83 ec 08             	sub    $0x8,%esp
   8048b26:	8b 45 08             	mov    0x8(%ebp),%eax
   8048b29:	83 c4 f8             	add    $0xfffffff8,%esp
   8048b2c:	68 c0 97 04 08       	push   $0x80497c0
   8048b31:	50                   	push   %eax
   8048b32:	e8 f9 04 00 00       	call   8049030 <strings_not_equal>
   #比较0x80497c0对应的字符串和输入的字符串
   8048b37:	83 c4 10             	add    $0x10,%esp
   8048b3a:	85 c0                	test   %eax,%eax
   #相等跳转8048b43
   8048b3c:	74 05                	je     8048b43 <phase_1+0x23>
   #不相等爆炸
   8048b3e:	e8 b9 09 00 00       	call   80494fc <explode_bomb>
   8048b43:	89 ec                	mov    %ebp,%esp
   8048b45:	5d                   	pop    %ebp
   8048b46:	c3                   	ret    
   8048b47:	90                   	nop
  ```

  strings_not_equal应该是个判断字符串是否相等的函数，前面两个push是这个函数的传入实参，查看`0x80497c0` 是`Public speaking is very easy.`，eax应该是输入的字符串。

  <explode_bomb>就是炸了

  看一眼strings_not_equal

  ```assembly
  08049030 <strings_not_equal>:
   8049030:	55                   	push   %ebp
   8049031:	89 e5                	mov    %esp,%ebp
   8049033:	83 ec 0c             	sub    $0xc,%esp
   8049036:	57                   	push   %edi
   8049037:	56                   	push   %esi
   8049038:	53                   	push   %ebx
   8049039:	8b 75 08             	mov    0x8(%ebp),%esi
   804903c:	8b 7d 0c             	mov    0xc(%ebp),%edi
   804903f:	83 c4 f4             	add    $0xfffffff4,%esp
   8049042:	56                   	push   %esi
   8049043:	e8 d0 ff ff ff       	call   8049018 <string_length>
   8049048:	89 c3                	mov    %eax,%ebx
   804904a:	83 c4 f4             	add    $0xfffffff4,%esp
   804904d:	57                   	push   %edi
   804904e:	e8 c5 ff ff ff       	call   8049018 <string_length>
   8049053:	39 c3                	cmp    %eax,%ebx
   8049055:	74 09                	je     8049060 <strings_not_equal+0x30>
   8049057:	b8 01 00 00 00       	mov    $0x1,%eax
   804905c:	eb 21                	jmp    804907f <strings_not_equal+0x4f>
   804905e:	89 f6                	mov    %esi,%esi
   8049060:	89 f2                	mov    %esi,%edx
   8049062:	89 f9                	mov    %edi,%ecx
   8049064:	80 3a 00             	cmpb   $0x0,(%edx)
   8049067:	74 14                	je     804907d <strings_not_equal+0x4d>
   8049069:	8d b4 26 00 00 00 00 	lea    0x0(%esi,%eiz,1),%esi
   8049070:	8a 02                	mov    (%edx),%al
   8049072:	3a 01                	cmp    (%ecx),%al
   8049074:	75 e1                	jne    8049057 <strings_not_equal+0x27>
   8049076:	42                   	inc    %edx
   8049077:	41                   	inc    %ecx
   8049078:	80 3a 00             	cmpb   $0x0,(%edx)
   804907b:	75 f3                	jne    8049070 <strings_not_equal+0x40>
   804907d:	31 c0                	xor    %eax,%eax
   804907f:	8d 65 e8             	lea    -0x18(%ebp),%esp
   8049082:	5b                   	pop    %ebx
   8049083:	5e                   	pop    %esi
   8049084:	5f                   	pop    %edi
   8049085:	89 ec                	mov    %ebp,%esp
   8049087:	5d                   	pop    %ebp
   8049088:	c3                   	ret    
   8049089:	8d 76 00             	lea    0x0(%esi),%esi
   
  ```

  strings_not_equal判断两个字符串是否相等

  如果相等跳转到8048b43，不相等调用<explode_bomb>爆炸。

  答案是``Public speaking is very easy.``



* phase_2

  ```assembly
  08048b48 <phase_2>:
   8048b48:	55                   	push   %ebp
   8048b49:	89 e5                	mov    %esp,%ebp
   8048b4b:	83 ec 20             	sub    $0x20,%esp
   8048b4e:	56                   	push   %esi
   8048b4f:	53                   	push   %ebx
   8048b50:	8b 55 08             	mov    0x8(%ebp),%edx
   8048b53:	83 c4 f8             	add    $0xfffffff8,%esp
   8048b56:	8d 45 e8             	lea    -0x18(%ebp),%eax
   8048b59:	50                   	push   %eax
   8048b5a:	52                   	push   %edx
   8048b5b:	e8 78 04 00 00       	call   8048fd8 <read_six_numbers>
   #读取六个数字
   #-0x18(%ebp)a[0]，-0x14(%ebp)a[1]，-0x10(%ebp)a[2]，-0xC(%ebp)a[3]，-0x8(%ebp)a[4]，-0x4(%ebp)a[5]
   8048b60:	83 c4 10             	add    $0x10,%esp
   8048b63:	83 7d e8 01          	cmpl   $0x1,-0x18(%ebp)
   #a[1]和1比较
   8048b67:	74 05                	je     8048b6e <phase_2+0x26>
   #等于1跳转到8048b6e
   8048b69:	e8 8e 09 00 00       	call   80494fc <explode_bomb>
   #不是1爆炸
   8048b6e:	bb 01 00 00 00       	mov    $0x1,%ebx
   #ebx=1
   8048b73:	8d 75 e8             	lea    -0x18(%ebp),%esi
   8048b76:	8d 43 01             	lea    0x1(%ebx),%eax
   #esi=a[1]，eax=ebx+1
   8048b79:	0f af 44 9e fc       	imul   -0x4(%esi,%ebx,4),%eax
   #eax=eax*[(esi+ebx*4)-0x4]
   8048b7e:	39 04 9e             	cmp    %eax,(%esi,%ebx,4)
   #比较eax和(esi+ebx*4)
   8048b81:	74 05                	je     8048b88 <phase_2+0x40>
   #相等继续循环
   8048b83:	e8 74 09 00 00       	call   80494fc <explode_bomb>
   #不相等爆炸
   8048b88:	43                   	inc    %ebx
   8048b89:	83 fb 05             	cmp    $0x5,%ebx
   8048b8c:	7e e8                	jle    8048b76 <phase_2+0x2e>
   #ebx=5时跳出循环，每次循环ebx+1
   8048b8e:	8d 65 d8             	lea    -0x28(%ebp),%esp
   8048b91:	5b                   	pop    %ebx
   8048b92:	5e                   	pop    %esi
   8048b93:	89 ec                	mov    %ebp,%esp
   8048b95:	5d                   	pop    %ebp
   8048b96:	c3                   	ret    
   8048b97:	90                   	nop
  ```

  输入六个数字，先判断第一个数字是否为1，然后开始循环

  ```C
  for(i=1;i<=5;i++)
  {
      if(a[i]!=a[i-1]*(i+1))
          explode_bomb();
  }
  ```

  答案为`1 2 6 24 120 720 `

  

* phase_3

  ```assembly
  08048b98 <phase_3>:
   8048b98:	55                   	push   %ebp
   8048b99:	89 e5                	mov    %esp,%ebp
   8048b9b:	83 ec 14             	sub    $0x14,%esp
   8048b9e:	53                   	push   %ebx
   8048b9f:	8b 55 08             	mov    0x8(%ebp),%edx
   8048ba2:	83 c4 f4             	add    $0xfffffff4,%esp
   8048ba5:	8d 45 fc             	lea    -0x4(%ebp),%eax
   8048ba8:	50                   	push   %eax
   8048ba9:	8d 45 fb             	lea    -0x5(%ebp),%eax
   8048bac:	50                   	push   %eax
   8048bad:	8d 45 f4             	lea    -0xc(%ebp),%eax
   8048bb0:	50                   	push   %eax
   8048bb1:	68 de 97 04 08       	push   $0x80497de
   8048bb6:	52                   	push   %edx
   8048bb7:	e8 a4 fc ff ff       	call   8048860 <sscanf@plt>
   8048bbc:	83 c4 20             	add    $0x20,%esp
   8048bbf:	83 f8 02             	cmp    $0x2,%eax
   #eax是sscanf的返回值，和2比较，小于等于2爆炸，正确输入%d %c %d返回3，跳转到8048bc9
   #-0xc(%ebp)是第一个数字，-0x5(%ebp)是字母，-0x4(%ebp)是第二个数字
   8048bc2:	7f 05                	jg     8048bc9 <phase_3+0x31>
   8048bc4:	e8 33 09 00 00       	call   80494fc <explode_bomb>
   8048bc9:	83 7d f4 07          	cmpl   $0x7,-0xc(%ebp)
   #输入的第一个数字和7比较，大于7跳到8048c88爆炸
   8048bcd:	0f 87 b5 00 00 00    	ja     8048c88 <phase_3+0xf0>
   8048bd3:	8b 45 f4             	mov    -0xc(%ebp),%eax
   8048bd6:	ff 24 85 e8 97 04 08 	jmp    *0x80497e8(,%eax,4)
   8048bdd:	8d 76 00             	lea    0x0(%esi),%esi
   8048be0:	b3 71                	mov    $0x71,%bl
   #bl=0x71
   8048be2:	81 7d fc 09 03 00 00 	cmpl   $0x309,-0x4(%ebp)
   #第二个数字和309比较
   8048be9:	0f 84 a0 00 00 00    	je     8048c8f <phase_3+0xf7>
   #相等跳转到8048c8f比较第二个字母，不相等爆炸
   8048bef:	e8 08 09 00 00       	call   80494fc <explode_bomb>
   8048bf4:	e9 96 00 00 00       	jmp    8048c8f <phase_3+0xf7>
   8048bf9:	8d b4 26 00 00 00 00 	lea    0x0(%esi,%eiz,1),%esi
   8048c00:	b3 62                	mov    $0x62,%bl
   8048c02:	81 7d fc d6 00 00 00 	cmpl   $0xd6,-0x4(%ebp)
   8048c09:	0f 84 80 00 00 00    	je     8048c8f <phase_3+0xf7>
   8048c0f:	e8 e8 08 00 00       	call   80494fc <explode_bomb>
   8048c14:	eb 79                	jmp    8048c8f <phase_3+0xf7>
   8048c16:	b3 62                	mov    $0x62,%bl
   8048c18:	81 7d fc f3 02 00 00 	cmpl   $0x2f3,-0x4(%ebp)
   8048c1f:	74 6e                	je     8048c8f <phase_3+0xf7>
   8048c21:	e8 d6 08 00 00       	call   80494fc <explode_bomb>
   8048c26:	eb 67                	jmp    8048c8f <phase_3+0xf7>
   8048c28:	b3 6b                	mov    $0x6b,%bl
   8048c2a:	81 7d fc fb 00 00 00 	cmpl   $0xfb,-0x4(%ebp)
   8048c31:	74 5c                	je     8048c8f <phase_3+0xf7>
   8048c33:	e8 c4 08 00 00       	call   80494fc <explode_bomb>
   8048c38:	eb 55                	jmp    8048c8f <phase_3+0xf7>
   8048c3a:	8d b6 00 00 00 00    	lea    0x0(%esi),%esi
   8048c40:	b3 6f                	mov    $0x6f,%bl
   8048c42:	81 7d fc a0 00 00 00 	cmpl   $0xa0,-0x4(%ebp)
   8048c49:	74 44                	je     8048c8f <phase_3+0xf7>
   8048c4b:	e8 ac 08 00 00       	call   80494fc <explode_bomb>
   8048c50:	eb 3d                	jmp    8048c8f <phase_3+0xf7>
   8048c52:	b3 74                	mov    $0x74,%bl
   8048c54:	81 7d fc ca 01 00 00 	cmpl   $0x1ca,-0x4(%ebp)
   8048c5b:	74 32                	je     8048c8f <phase_3+0xf7>
   8048c5d:	e8 9a 08 00 00       	call   80494fc <explode_bomb>
   8048c62:	eb 2b                	jmp    8048c8f <phase_3+0xf7>
   8048c64:	b3 76                	mov    $0x76,%bl
   8048c66:	81 7d fc 0c 03 00 00 	cmpl   $0x30c,-0x4(%ebp)
   8048c6d:	74 20                	je     8048c8f <phase_3+0xf7>
   8048c6f:	e8 88 08 00 00       	call   80494fc <explode_bomb>
   8048c74:	eb 19                	jmp    8048c8f <phase_3+0xf7>
   8048c76:	b3 62                	mov    $0x62,%bl
   8048c78:	81 7d fc 0c 02 00 00 	cmpl   $0x20c,-0x4(%ebp)
   8048c7f:	74 0e                	je     8048c8f <phase_3+0xf7>
   8048c81:	e8 76 08 00 00       	call   80494fc <explode_bomb>
   8048c86:	eb 07                	jmp    8048c8f <phase_3+0xf7>
   8048c88:	b3 78                	mov    $0x78,%bl
   8048c8a:	e8 6d 08 00 00       	call   80494fc <explode_bomb>
   8048c8f:	3a 5d fb             	cmp    -0x5(%ebp),%bl
   #输入的字母的ascii码和bl比较
   8048c92:	74 05                	je     8048c99 <phase_3+0x101>
   #不相等爆炸
   8048c94:	e8 63 08 00 00       	call   80494fc <explode_bomb>
   8048c99:	8b 5d e8             	mov    -0x18(%ebp),%ebx
   8048c9c:	89 ec                	mov    %ebp,%esp
   8048c9e:	5d                   	pop    %ebp
   8048c9f:	c3                   	ret    
  ```

  调用了sscanf，先看一眼0x80497de

  `%d %c %d`应输入”数字 字母 数字“

  8048bc9后一堆cmpl je看着像switch语句

  答案

  |            |      |      |      |      |      |      |      |      |
  | ---------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
  | 第一个数字 | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    |
  | 字母       | q    | b    | b    | k    | o    | t    | v    | b    |
  | 第二个数字 | 777  | 214  | 755  | 251  | 160  | 458  | 780  | 524  |

* phase_4

  ```assembly
  08048ce0 <phase_4>:
   8048ce0:	55                   	push   %ebp
   8048ce1:	89 e5                	mov    %esp,%ebp
   8048ce3:	83 ec 18             	sub    $0x18,%esp
   8048ce6:	8b 55 08             	mov    0x8(%ebp),%edx
   8048ce9:	83 c4 fc             	add    $0xfffffffc,%esp
   8048cec:	8d 45 fc             	lea    -0x4(%ebp),%eax
   8048cef:	50                   	push   %eax
   8048cf0:	68 08 98 04 08       	push   $0x8049808
   8048cf5:	52                   	push   %edx
   8048cf6:	e8 65 fb ff ff       	call   8048860 <sscanf@plt>
   8048cfb:	83 c4 10             	add    $0x10,%esp
   8048cfe:	83 f8 01             	cmp    $0x1,%eax
   #eax是sscanf的返回值，正确输入返回1，如果eax不是1跳转到8048d09爆炸
   8048d01:	75 06                	jne    8048d09 <phase_4+0x29>
   8048d03:	83 7d fc 00          	cmpl   $0x0,-0x4(%ebp)
   #输入的数字和0比较，小于等于0爆炸
   8048d07:	7f 05                	jg     8048d0e <phase_4+0x2e>
   8048d09:	e8 ee 07 00 00       	call   80494fc <explode_bomb>
   8048d0e:	83 c4 f4             	add    $0xfffffff4,%esp
   8048d11:	8b 45 fc             	mov    -0x4(%ebp),%eax
   #eax=-0x4(%ebp)，eax是func4的传入实参
   8048d14:	50                   	push   %eax
   8048d15:	e8 86 ff ff ff       	call   8048ca0 <func4>
   8048d1a:	83 c4 10             	add    $0x10,%esp
   8048d1d:	83 f8 37             	cmp    $0x37,%eax
   #eax是func4的返回值，eax!=0x37爆炸
   8048d20:	74 05                	je     8048d27 <phase_4+0x47>
   8048d22:	e8 d5 07 00 00       	call   80494fc <explode_bomb>
   8048d27:	89 ec                	mov    %ebp,%esp
   8048d29:	5d                   	pop    %ebp
   8048d2a:	c3                   	ret    
   8048d2b:	90                   	nop
  ```

  还是调用sscanf，看一眼0x8049808

  `%d`应输入一个数字

  调用了func4，看一眼

  ```assembly
  8048ca0 <func4>:
   8048ca0:	55                   	push   %ebp
   8048ca1:	89 e5                	mov    %esp,%ebp
   8048ca3:	83 ec 10             	sub    $0x10,%esp
   8048ca6:	56                   	push   %esi
   8048ca7:	53                   	push   %ebx
   8048ca8:	8b 5d 08             	mov    0x8(%ebp),%ebx
   8048cab:	83 fb 01             	cmp    $0x1,%ebx
   #ebx和1比较，小于等于1跳转到8048cd0
   8048cae:	7e 20                	jle    8048cd0 <func4+0x30>
   8048cb0:	83 c4 f4             	add    $0xfffffff4,%esp
   8048cb3:	8d 43 ff             	lea    -0x1(%ebx),%eax
   #eax=ebx-1
   8048cb6:	50                   	push   %eax
   8048cb7:	e8 e4 ff ff ff       	call   8048ca0 <func4>
   8048cbc:	89 c6                	mov    %eax,%esi
   #esi=eax
   8048cbe:	83 c4 f4             	add    $0xfffffff4,%esp
   8048cc1:	8d 43 fe             	lea    -0x2(%ebx),%eax
   #eax=abx-2
   8048cc4:	50                   	push   %eax
   8048cc5:	e8 d6 ff ff ff       	call   8048ca0 <func4>
   8048cca:	01 f0                	add    %esi,%eax
   #eax=esi+eax
   8048ccc:	eb 07                	jmp    8048cd5 <func4+0x35>
   8048cce:	89 f6                	mov    %esi,%esi
   8048cd0:	b8 01 00 00 00       	mov    $0x1,%eax
   #eax=1，如果ebx==1，func4返回1
   8048cd5:	8d 65 e8             	lea    -0x18(%ebp),%esp
   8048cd8:	5b                   	pop    %ebx
   8048cd9:	5e                   	pop    %esi
   8048cda:	89 ec                	mov    %ebp,%esp
   8048cdc:	5d                   	pop    %ebp
   8048cdd:	c3                   	ret    
   8048cde:	89 f6                	mov    %esi,%esi
  ```

  func4里call func4，这玩意是个递归

  ```c
  int func4(int x)
  {
      if(x <= 1)
          return 1;
      else
          return func4(x - 1) + func4(x - 2);
  }
  ```

  暴力破解一波

  ```c
  #include<stdio.h>
  int func4(int x)
  {
      if(x <= 1)
          return 1;
      else
          return func4(x - 1) + func4(x - 2);
  }
  
  int main()
  {
  	int a = 0,i = 0;
  	while(a != 55)
  	{
  		i++;
  		a = func4(i);
  	}
  	printf("%d",i);
  } 
  ```

  答案是`9`

  

* phase_5

  ```assembly
  08048d2c <phase_5>:
   8048d2c:	55                   	push   %ebp
   8048d2d:	89 e5                	mov    %esp,%ebp
   8048d2f:	83 ec 10             	sub    $0x10,%esp
   8048d32:	56                   	push   %esi
   8048d33:	53                   	push   %ebx
   8048d34:	8b 5d 08             	mov    0x8(%ebp),%ebx
   8048d37:	83 c4 f4             	add    $0xfffffff4,%esp
   8048d3a:	53                   	push   %ebx
   8048d3b:	e8 d8 02 00 00       	call   8049018 <string_length>
   8048d40:	83 c4 10             	add    $0x10,%esp
   8048d43:	83 f8 06             	cmp    $0x6,%eax
   #输入一个字符串，长度和6比较，不是6爆炸,ebx是输入的字符串
   8048d46:	74 05                	je     8048d4d <phase_5+0x21>
   8048d48:	e8 af 07 00 00       	call   80494fc <explode_bomb>
   
   8048d4d:	31 d2                	xor    %edx,%edx
   #edx=0
   8048d4f:	8d 4d f8             	lea    -0x8(%ebp),%ecx
   8048d52:	be 20 b2 04 08       	mov    $0x804b220,%esi
   #esi是isrveawhobpnutfg
   8048d57:	8a 04 1a             	mov    (%edx,%ebx,1),%al
   #al=edx+ebx*1
   8048d5a:	24 0f                	and    $0xf,%al
   #al=al & 0xf
   8048d5c:	0f be c0             	movsbl %al,%eax
   #eax=al
   8048d5f:	8a 04 30             	mov    (%eax,%esi,1),%al
   #al=eax+esi*1
   8048d62:	88 04 0a             	mov    %al,(%edx,%ecx,1)
   #edx+ecx*1=al
   8048d65:	42                   	inc    %edx
   #edx=edx+1
   8048d66:	83 fa 05             	cmp    $0x5,%edx
   #edx<=5继续循环
   8048d69:	7e ec                	jle    8048d57 <phase_5+0x2b>
   
   8048d6b:	c6 45 fe 00          	movb   $0x0,-0x2(%ebp)
   8048d6f:	83 c4 f8             	add    $0xfffffff8,%esp
   
   8048d72:	68 0b 98 04 08       	push   $0x804980b
   8048d77:	8d 45 f8             	lea    -0x8(%ebp),%eax
   8048d7a:	50                   	push   %eax
   8048d7b:	e8 b0 02 00 00       	call   8049030 <strings_not_equal>
   8048d80:	83 c4 10             	add    $0x10,%esp
   8048d83:	85 c0                	test   %eax,%eax
   8048d85:	74 05                	je     8048d8c <phase_5+0x60>
   8048d87:	e8 70 07 00 00       	call   80494fc <explode_bomb>
   8048d8c:	8d 65 e8             	lea    -0x18(%ebp),%esp
   8048d8f:	5b                   	pop    %ebx
   8048d90:	5e                   	pop    %esi
   8048d91:	89 ec                	mov    %ebp,%esp
   8048d93:	5d                   	pop    %ebp
   8048d94:	c3                   	ret    
   8048d95:	8d 76 00             	lea    0x0(%esi),%esi
  ```

  循环开始之前取出0x804b220到esi，0x804b220是isrveawhobpnutfg

  最后调用了strings_not_equal，看一眼804980b

  `giants`

  循环后得到的字符串和giants比较

  ```c
  char a[7],b[17]='isrveawhobpnutfg',c[5]
  for(i=0;i<=5;i++)
  {
      int x=(int)(c[i]&0xf);
      a[i]=b[x];
  }
  ```

  &0xf后应为`0x0f 0x00 0x05 0x0b 0x0d 0x01`

  高四位随便改，低四位不动，可以得到多组答案

  其中一组为`opukma`

  

* phase_6

  ```assembly
  08048d98 <phase_6>:
   8048d98:	55                   	push   %ebp
   8048d99:	89 e5                	mov    %esp,%ebp
   8048d9b:	83 ec 4c             	sub    $0x4c,%esp
   8048d9e:	57                   	push   %edi
   8048d9f:	56                   	push   %esi
   8048da0:	53                   	push   %ebx
   8048da1:	8b 55 08             	mov    0x8(%ebp),%edx
   8048da4:	c7 45 cc 6c b2 04 08 	movl   $0x804b26c,-0x34(%ebp)
   8048dab:	83 c4 f8             	add    $0xfffffff8,%esp
   8048dae:	8d 45 e8             	lea    -0x18(%ebp),%eax
   8048db1:	50                   	push   %eax
   8048db2:	52                   	push   %edx
   8048db3:	e8 20 02 00 00       	call   8048fd8 <read_six_numbers>
   #输入六个数字，-0x18(%ebp)是首地址
   8048db8:	31 ff                	xor    %edi,%edi
   #edi=0
   8048dba:	83 c4 10             	add    $0x10,%esp
   8048dbd:	8d 76 00             	lea    0x0(%esi),%esi
   8048dc0:	8d 45 e8             	lea    -0x18(%ebp),%eax
   #eax是数组首地址
   8048dc3:	8b 04 b8             	mov    (%eax,%edi,4),%eax
   #eax=eax+adi*4
   8048dc6:	48                   	dec    %eax
   #eax=eax-1
   8048dc7:	83 f8 05             	cmp    $0x5,%eax
   #eax和5比较，大于5爆炸
   8048dca:	76 05                	jbe    8048dd1 <phase_6+0x39>
   8048dcc:	e8 2b 07 00 00       	call   80494fc <explode_bomb>
   
   8048dd1:	8d 5f 01             	lea    0x1(%edi),%ebx
   #ebx=edi+1
   
   8048dd4:	83 fb 05             	cmp    $0x5,%ebx
   8048dd7:	7f 23                	jg     8048dfc <phase_6+0x64>
   #ebx和5比较小于等于5跳转到8048dfc
   8048dd9:	8d 04 bd 00 00 00 00 	lea    0x0(,%edi,4),%eax
   8048de0:	89 45 c8             	mov    %eax,-0x38(%ebp)
   #-0x38(%ebp)=eax
   8048de3:	8d 75 e8             	lea    -0x18(%ebp),%esi
   #esi是输入数组的首地址
   8048de6:	8b 55 c8             	mov    -0x38(%ebp),%edx
   #edx=-0x38(%ebp)
   8048de9:	8b 04 32             	mov    (%edx,%esi,1),%eax
   8048dec:	3b 04 9e             	cmp    (%esi,%ebx,4),%eax
   8048def:	75 05                	jne    8048df6 <phase_6+0x5e>
   8048df1:	e8 06 07 00 00       	call   80494fc <explode_bomb>
   #edx+esi*1!=esi+ebx*4跳转到8048df6，相等爆炸
   8048df6:	43                   	inc    %ebx
   #ebx=ebx+1
   8048df7:	83 fb 05             	cmp    $0x5,%ebx
   8048dfa:	7e ea                	jle    8048de6 <phase_6+0x4e>
   8048dfc:	47                   	inc    %edi
   8048dfd:	83 ff 05             	cmp    $0x5,%edi
   8048e00:	7e be                	jle    8048dc0 <phase_6+0x28>
   #第一个循环结束
   8048e02:	31 ff                	xor    %edi,%edi
   #edi=0
   8048e04:	8d 4d e8             	lea    -0x18(%ebp),%ecx
   #ecx是数组首地址
   8048e07:	8d 45 d0             	lea    -0x30(%ebp),%eax
   #eax是地址-0x30(%ebp)
   8048e0a:	89 45 c4             	mov    %eax,-0x3c(%ebp)
   8048e0d:	8d 76 00             	lea    0x0(%esi),%esi
   8048e10:	8b 75 cc             	mov    -0x34(%ebp),%esi
   #esi=-0x34(%ebp)
   8048e13:	bb 01 00 00 00       	mov    $0x1,%ebx
   #ebx=1
   8048e18:	8d 04 bd 00 00 00 00 	lea    0x0(,%edi,4),%eax
   #eax=edi*4
   8048e1f:	89 c2                	mov    %eax,%edx
   #edx=eax
   8048e21:	3b 1c 08             	cmp    (%eax,%ecx,1),%ebx
   8048e24:	7d 12                	jge    8048e38 <phase_6+0xa0>
   #ebx小于eax+ecx*1进入循环
   8048e26:	8b 04 0a             	mov    (%edx,%ecx,1),%eax
   8048e29:	8d b4 26 00 00 00 00 	lea    0x0(%esi,%eiz,1),%esi
   8048e30:	8b 76 08             	mov    0x8(%esi),%esi
   #esi=esi+0x8
   8048e33:	43                   	inc    %ebx
   #ebx=ebx+1
   8048e34:	39 c3                	cmp    %eax,%ebx
   8048e36:	7c f8                	jl     8048e30 <phase_6+0x98>
   #ebx小于eax继续循环
   8048e38:	8b 55 c4             	mov    -0x3c(%ebp),%edx
   #edx=-0x3c(%ebp)
   8048e3b:	89 34 ba             	mov    %esi,(%edx,%edi,4)
   #edx+edi*4=esi
   8048e3e:	47                   	inc    %edi
   8048e3f:	83 ff 05             	cmp    $0x5,%edi
   8048e42:	7e cc                	jle    8048e10 <phase_6+0x78>
   #edi小于等于5继续循环
   8048e44:	8b 75 d0             	mov    -0x30(%ebp),%esi
   8048e47:	89 75 cc             	mov    %esi,-0x34(%ebp)
   8048e4a:	bf 01 00 00 00       	mov    $0x1,%edi
   #edi=1
   8048e4f:	8d 55 d0             	lea    -0x30(%ebp),%edx
   8048e52:	8b 04 ba             	mov    (%edx,%edi,4),%eax
   8048e55:	89 46 08             	mov    %eax,0x8(%esi)
   #eax=esi+0x8
   8048e58:	89 c6                	mov    %eax,%esi
   #esi=eax
   8048e5a:	47                   	inc    %edi
   8048e5b:	83 ff 05             	cmp    $0x5,%edi
   8048e5e:	7e f2                	jle    8048e52 <phase_6+0xba>
   #edi小于等于5继续循环
   8048e60:	c7 46 08 00 00 00 00 	movl   $0x0,0x8(%esi)
   8048e67:	8b 75 cc             	mov    -0x34(%ebp),%esi
   8048e6a:	31 ff                	xor    %edi,%edi
   #edi=0
   8048e6c:	8d 74 26 00          	lea    0x0(%esi,%eiz,1),%esi
   8048e70:	8b 56 08             	mov    0x8(%esi),%edx
   #eax=esi+0x8
   8048e73:	8b 06                	mov    (%esi),%eax
   #eax=esi
   8048e75:	3b 02                	cmp    (%edx),%eax
   8048e77:	7d 05                	jge    8048e7e <phase_6+0xe6>
   #eax>=edx跳转到8048e7e，否则爆炸
   8048e79:	e8 7e 06 00 00       	call   80494fc <explode_bomb>
   8048e7e:	8b 76 08             	mov    0x8(%esi),%esi
   8048e81:	47                   	inc    %edi
   #edi++
   8048e82:	83 ff 04             	cmp    $0x4,%edi
   8048e85:	7e e9                	jle    8048e70 <phase_6+0xd8>
   #edi小于等于4继续循环
   8048e87:	8d 65 a8             	lea    -0x58(%ebp),%esp
   8048e8a:	5b                   	pop    %ebx
   8048e8b:	5e                   	pop    %esi
   8048e8c:	5f                   	pop    %edi
   8048e8d:	89 ec                	mov    %ebp,%esp
   8048e8f:	5d                   	pop    %ebp
   8048e90:	c3                   	ret    
   8048e91:	8d 76 00             	lea    0x0(%esi),%esi
  
  ```

  开头有个804b26c，是`<node1> `

  ```
  node1:0xfd
  node2:0x2d5
  node3:0x12d
  node4:0x3e5
  node5:0xd4
  node6:0x1b0
  ```

  第一个循环是一个嵌套循环，每次循环先判断当前元素是否大于五再判断是否存在与当前元素相同的元素，所以数组内元素应大于等于0且小于5，且每个元素不同

  第二个循环应该是根据输入的数据对node进行排序，储存到一个新数组中

  0x8(%esi),%esi应该是指向下一个node\

  ```
  for(i=0; i<=5; i++)
  {
      if (a[i]>5)
          explode_bomb();
      for (j=i+1; j<=5; j++)
      {
          if (a[i]== a[j])
              explode_bomb();
      }
  }
  for(i=0;i<=5;i++)
  {
      for(j=1;j<a[i];j++)
      	node = node.next;
      s[i]=node;
  }
  ```

  第三个循环根据排序后的新数组重新排列node

  第四个循环判断排序后的node是否满足条件，显然是从大到小排列

  答案是`4 2 6 3 1 5`

  ![1](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/2019-1-25-CSAPP-Bomb%20Lab/1.png)

  搞完了，没有真香
