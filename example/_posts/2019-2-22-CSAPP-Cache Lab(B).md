# CSAPP-Cache Lab(B)

Part B 矩阵转制优化

编写一个函数，计算给定矩阵的转置，最小化模拟缓存中的未命中数

测试矩阵的尺寸为32\*32、64\*64、61\*67

测试缓存 s = 5，E = 1，b = 5

32组，一组1行，一个块32字节，存放8个int



* 矩阵转置

```c
for (int i = 0; i < M; i++)
{
    for (int j = 0; j < N; j++)
    {
        B[j][i] = A[i][j];
    }
}
```



* 32 * 32

  ![3](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/cache%20lab/3.png)

  矩阵一行32个int，数字代表数据储存的组，缓存32组，可以储存8\*32个int，8行填满一个cache，每隔8行出现冲突，造成冲突不命中

  把矩阵分成16个8\*8的小块，分别转置

  ```c
      if ( M == 32 && N == 32)
      {
          for (int i = 0; i < N; i += 8)
          {
              for (int j = 0; j < M; j += 8)
              {
                  for (int k = i; k < i + 8; k++)
                  {
                      int temp0 = A[k][j];
                      int temp1 = A[k][j+1];
                      int temp2 = A[k][j+2];
                      int temp3 = A[k][j+3];
                      int temp4 = A[k][j+4];
                      int temp5 = A[k][j+5];
                      int temp6 = A[k][j+6];
                      int temp7 = A[k][j+7];
                  
                      B[j][k] = temp0;
                      B[j+1][k] = temp1;
                      B[j+2][k] = temp2;
                      B[j+3][k] = temp3;
                      B[j+4][k] = temp4;
                      B[j+5][k] = temp5;
                      B[j+6][k] = temp6;
                      B[j+7][k] = temp7;
                  }
              }
          }       
      }
  ```

* 64*64

  ![5](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/cache%20lab/5.png)

  一行64个int，4行填满一个cache，每隔4行出现冲突，造成冲突不命中

  分成4\*4的试试

  ```c
  if ( M == 64 && N == 64)
      {
      	for (int i = 0; i < N; i += 4)
          {
              for (int j = 0; j < M; j += 4)
              {
                  for (int k = i; k < i + 4; k++)
                  {
                      int temp0 = A[k][j];
                      int temp1 = A[k][j+1];
                      int temp2 = A[k][j+2];
                      int temp3 = A[k][j+3];
                  
                      B[j][k] = temp0;
                      B[j+1][k] = temp1;
                      B[j+2][k] = temp2;
                      B[j+3][k] = temp3;
                  }
              }
          }     
  	}
  ```

  ![6](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/cache%20lab/6.png)

  miss应该在1300以下，1699，gg

  不可能用16\*16的，还是要用8\*8的搞，转置时还是要分成4\*4，俺寻思得把8\*8分成4个4\*4。

  然后就不会了

  记录一个对b操作，先转置再平移的做法

  ```c
  for (i = 0; i < N; i += 8) 
  {
          for (j = 0; j < M; j += 8) 
          {
              for (k = i; k < i + 4; k++) 
              {
                  a0 = A[k][j];
                  a1 = A[k][j + 1];
                  a2 = A[k][j + 2];
                  a3 = A[k][j + 3];
                  a4 = A[k][j + 4];
                  a5 = A[k][j + 5];
                  a6 = A[k][j + 6];
                  a7 = A[k][j + 7];
  
                  B[j][k] = a0;
                  B[j + 1][k] = a1;
                  B[j + 2][k] = a2;
                  B[j + 3][k] = a3;
  
                  B[j][k + 4] = a4;
                  B[j + 1][k + 4] = a5;
                  B[j + 2][k + 4] = a6;
                  B[j + 3][k + 4] = a7;
              }
              for (l = j + 4; l < j + 8; l++) 
              {
  
                  a4 = A[i + 4][l - 4]; // A left-down col
                  a5 = A[i + 5][l - 4];
                  a6 = A[i + 6][l - 4];
                  a7 = A[i + 7][l - 4];
  
                  a0 = B[l - 4][i + 4]; // B right-above line
                  a1 = B[l - 4][i + 5];
                  a2 = B[l - 4][i + 6];
                  a3 = B[l - 4][i + 7];
  
                  B[l - 4][i + 4] = a4; // set B right-above line 
                  B[l - 4][i + 5] = a5;
                  B[l - 4][i + 6] = a6;
                  B[l - 4][i + 7] = a7;
  
                  B[l][i] = a0;         // set B left-down col
                  B[l][i + 1] = a1;
                  B[l][i + 2] = a2;
                  B[l][i + 3] = a3;
  
                  B[l][i + 4] = A[i + 4][l];
                  B[l][i + 5] = A[i + 5][l];
                  B[l][i + 6] = A[i + 6][l];
                  B[l][i + 7] = A[i + 7][l];
              }
          }
      }
  ```

* 61\*67

  非对称矩阵，相差4行不一定冲突，直接分块

  试了一下8\*8，miss超出2000，16\*16可以

  ```c
  if ( M ==61 && N == 67)
      {
          for (int i = 0; i < N; i += 16)
          {
              for (int j = 0; j < M; j += 16)
              {
                  for (int k = i; k < i + 16 && k < 67; k++)
                  {
                  	for (int l = j; l < j + 16 && l < 61; l++)
                  	{
                  		  B[l][k] = A[k][l];
                  	}
                  }
              }
          }       	
      }
  ```

![7](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/cache%20lab/7.png)



* 