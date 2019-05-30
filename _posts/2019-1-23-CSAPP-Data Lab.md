# CSAPP-Data Lab



* bitAnd

  * 按位与
  * 可用运算符：~ |
  * 最大操作数：8

  x&y得到x，y相同位置的1，~x|~y，得到~x，~y相同位置的0，~(~x|~y)=x&y

  ```C
  int bitAnd(int x, int y) 
  {
  	return ~(~x | ~y);
  }
  ```

* getByte

  * 从x中求出第n个字节
  * 可用运算符：! ~ & ^ | + << >>
  * 最大操作数：6

  x右移n个字节&0xFF，移动n个字节就是移动n\*8位，n\*8就是n左移3位

  ```C
  int getByte(int x, int n) 
  {
  	return (x >> (n << 3)) & 0xFF;
  }
  ```

* logicalShift

  * 使用逻辑位移将x移动n位（0<=n<=31）
  * 可用运算符： ! ~ & ^ | + << >>
  * 最大操作数：20

  先算术右移，再&0....01....1(n个0)

  构造0....01....1

  1左移31位再按位取反得到10....0(31个0)，按位取反再右移n位得到0...01...1(n+1)个0，左移一位得到0..01..10(前面n个0)，末尾加1，得到0....01....1(n个0)

  ```C
  int logicalShift(int x, int n) 
  {
  	return (x >> n) & ((~(1 << 31) >> n << 1) + 1);
  }
  ```

* bitCount

  * 计算1的个数
  * 可用运算符： ! ~ & ^ | + << >>
  * 最大操作数：40

* bang

  * 不用!求出!x
  * 可用运算符：~ & ^ | + << >>
  * 最大操作数：12

  正数取相反数，除符号位之外按位取反，然后加1，得到负数的补码。

  0和-0的补码相同，0|-0 符号位为0，其余正数|相反数符号位为1

  计算x|-x，右移31位，若x不为0则得到0xFFFFFFFF，若x为0则得到0x0，加一得到!x

  ```C
  int bang(int x) 
  {
  	return (((x | (~x + 1)) >> 31) + 1);
  }
  ```

* tmin

  * 返回补码整数的最小值
  * 可用运算符：! ~ & ^ | + << >>
  * 最大操作数：15

  -2147483648

  ```
  int tmin(void) 
  {
  	return (1<<31);
  }
  ```

* fitsBits

  * 若x能被表示为一个n位的补码则返回1
  * 可用运算符：! ~ & ^ | + << >>
  * 最大操作数：15

  对于正数若能表示则第32位到第n位全为0

  对于负数若能表示则第32位到第n位全是1

  所以若能表示则x先左移32-n位再右移32-n位一定与x相同

  ```C
  int fitsBits(int x, int n) 
  {
  	int a;	
   	a=32+(~n+1);
  	return !(((x<<a)>>a)^x);
  }
  ```

* divpwr2

  * 求x/2^n，向0取整
  * 可用运算符：! ~ & ^ | + << >>
  * 最大操作数：15

  正数直接右移n位，但负数这样搞就向下取整了

  如果是负数应该加上2^n-1再右移

  ```C
  int divpwr2(int x, int n) 
  {
  	return (x + ((x >> 31) & ((1 << n) + ~0))) >> n;
  }
  ```

  

* negate

  * 求x的相反数
  * 可用运算符：! ~ & ^ | + << >>
  * 最大操作数：8

  (~x)+1

  ```C
  int negate(int x) 
  {
  	return ((~x) + 1 );
  }
  ```

* isPositive

  * 若x为正数返回1
  * 可用运算符：! ~ & ^ | + << >>
  * 最大操作数：24

  判断是否为0或负数

  ```C
  int isPositive(int x) 
  {
  	return !(!x | (x >> 31));
  }
  ```

* isLessOrEqual

  * 如果x<=y返回1，否则返回0
  * 可用运算符：! ~ & ^ | + << >>
  * 最大操作数：24

  y-x>=0，如果x，y异号可能会溢出，所以两种情况。x，y异号，根据x正负判断，x，y同号，根据y-x判断

  ```C
  int isLessOrEqual(int x, int y) 
  {
  	int xx;
  	int yy;
  	int flag;
  	int diff;
  	xx = (x >> 31) & 1;
  	yy = (y >> 31) & 1;
  	flag = xx ^ yy;
  	diff = ((y + (~x + 1)) >> 31) & 1;
  	return (flag & xx) | (!diff & !flag);
  }
  ```

  

* ilog2
  * 求log2(x)
  * 可用运算符：! ~ & ^ | + << >>
  * 最大操作数：90

* float_neg

  * 输入uf返回-uf，如果uf是NaN返回uf
  * 最大操作数：10

  如果是NaN，一定大于0x7F800000

  ```C
  unsigned float_neg(unsigned uf) 
  {
  	unsigned res;
  	res = uf ^ 0x80000000;
  	if( (uf & 0x7FFFFFFF;) > 0x7F800000) 
          return uf;
  	return res;
  }
  ```

  

* float_i2f

  * 将int型x转为float
  * 最大操作数：30

  ```c
  unsigned float_i2f(int x) 
  {
      unsigned shiftLeft=0;
      unsigned afterShift, tmp, flag;
      unsigned absX=x;
      unsigned sign=0;
      if (x == 0) 
     		return 0;
      if (x < 0)
      {
          sign = 0x80000000;
          absX = -x;
      }
      afterShift = absX;
      while (1)
      {
          tmp=afterShift;
          afterShift <<= 1;
          shiftLeft++;
          if (tmp & 0x80000000) break;
      }
      if ((afterShift & 0x01ff)>0x0100)
          flag = 1;
      else if ((afterShift & 0x03ff)==0x0300)
          flag = 1;
      else
          flag = 0;
      return sign + (afterShift>>9) + ((159-shiftLeft)<<23) + flag;
  }
  ```

  

* float_twice
  * 输入浮点数f，返回2f
  * 最大操作数：30

  ```c
  unsigned float_twice(unsigned uf) 
  {
      unsigned f = uf;
      if ((f & 0x7F800000) == 0) 
      {
          f = ((f & 0x007FFFFF) << 1) | (0x80000000 & f);
      }
      else if ((f & 0x7F800000) != 0x7F800000)
      {
          f = f + 0x00800000;
      }
      return f;
  }
  ```

  
