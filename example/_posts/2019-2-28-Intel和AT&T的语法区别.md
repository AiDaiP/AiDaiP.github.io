# Intel和AT&T的语法区别

记录一下Intel和AT&T的语法区别，防止脑残



* 操作数前缀

  在 Intel 语法中，寄存器和和立即数都没有前缀。而在 AT&T 中，寄存器前加“％”，立即数前加“$”。

  在 Intel 语法中，十六进制立即数使用“h”后缀，而在 AT&T 中，十六进制立即数前加“0x”

  | Intel 语法     | AT&T 语法         |
  | -------------- | ----------------- |
  | Mov eax,8      | movl $8,%eax      |
  | Mov ebx,0ffffh | movl $0xffff,%ebx |
  | int 80h        | int $0x80         |

* 源/目的操作数顺序

  在 Intel 语法中，第一个操作数是目的操作数，第二个操作数源操作数。而在 AT&T 中，第一个数是源操作数，第二个数是目的操作数 

  | Intel 语法 | AT&T 语法    |
  | ---------- | ------------ |
  | Mov EAX,8  | movl $8,%eax |

* 寻址方式

   Intel语法的指令格式是segreg： [base+index*scale+disp]

   AT&T 的格式是%segreg：disp(base,index,scale) 

| Intel 语法                                 | AT&T 语法                                  |
| ------------------------------------------ | ------------------------------------------ |
| Instr foo,segreg： [base+index*scale+disp] | instr %segreg： disp(base,index,scale),foo |
| [eax]                                      | (%eax)                                     |
| [eax + _variable]                          | _variable(%eax)                            |
| [eax*4 + _array]                           | _array(,%eax,4)                            |
| [ebx + eax*8 + _array]                     | _array(%ebx,%eax,8)                        |

* 操作码的前缀和后缀

  在 AT&T 汇编中远程跳转指令和子过程调用指令的操作码使用前缀“l”，分别为 ljmp，lcall，与之相应的返回指令为 lret。

  | Intel 语法         | AT&T 语法                  |
  | ------------------ | -------------------------- |
  | LL SECTION:OFFSET  | lcall secion:secion:offset |
  | FAR SECTION:OFFSET | ljmp secion:secion:offset  |
  | FAR STACK_ADJUST   | lret $stack_adjust         |

  AT&T 的操作码后面有时还会有一个后缀，其含义就是指出操作码的大小。“l”表示长整数（32 位），“w”表示字（16 位），“b”表示字节（8 位）。而在 Intel 的语法中，则要在内存单元操作数的前面加上 byte ptr、 word ptr,和 dword ptr

  | Intel 语法               | AT&T 语法        |
  | ------------------------ | ---------------- |
  | Mov al,bl                | movb %bl,%al     |
  | Mov ax,bx                | movw %bx,%ax     |
  | Mov eax,ebx              | movl %ebx,%eax   |
  | Mov eax, dword ptr [ebx] | movl (%ebx),%eax |

  

  

  | AT&T             | Intel               |
  | ---------------------- | ----------------------- |
  | 寄存器前加%            | 寄存器无需另加符号      |
  | 立即数前加$            | 立即数无需另加符号      |
  | 16进制立即数使用0x前缀 | 16进制的立即数使用h后缀 |
  | 源操作数在前，目的操作数在后（从前往后读） | 目的操作数在前，源操作数在后（从后往前读） |
  | 间接寻址使用小括号()                       | 间接寻址使用中括号[]                       |
  | 间接寻址完整格式：%sreg:disp(%base,index,scale) | 间接寻址完整格式：sreg:[basereg + index*scale + disp] |
  | 操作位数：指令+l、w、b                          | 指令+ dword ptr、word ptr、byte ptr                   |