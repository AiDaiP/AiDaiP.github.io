# LeetCode-Reverse Integer

Given a 32-bit signed integer, reverse digits of an integer.

**Example 1:**

```
Input: 123
Output: 321
```

**Example 2:**

```
Input: -123
Output: -321
```

**Example 3:**

```
Input: 120
Output: 21
```

**Note:**
Assume we are dealing with an environment which could only store integers within the 32-bit signed integer range: [−231,  231 − 1]. For the purpose of this problem, assume that your function returns 0 when the reversed integer overflows.



x%10得到最后一位数字，x/10丢弃最后一位。

把得到的最后一位数字放到res的后面即每次循环res*10再加上最后一位数字。

反转可能出现溢出，res的取值范围为 [−2^31,  2^31 − 1] 再计算新的res前判断是否会溢出

```c++
class Solution 
{
public:
    int reverse(int x) 
    {
        int res = 0;
        int num = 0;
        while (x != 0)
        {
        	num = x % 10;
        	x /= 10;
        	if (res > INT_MAX / 10 || res < INT_MIN / 10)
        	{
        		return 0;
        	}
        	res = res * 10 + num;
        }

        return res;
    }
};
```

这个能过，但是看了答案发现漏了点东西，res == INT_MAX / 10 或 res == INT_MIN / 10时也可能溢出

再加上一个判断

```c++
class Solution 
{
public:
    int reverse(int x) 
    {
        int res = 0;
        int num = 0;
        while (x != 0)
        {
        	num = x % 10;
        	x /= 10;
        	if (res > INT_MAX / 10 || res < INT_MIN / 10)
        	{
        		return 0;
        	}
        	else if ((res == INT_MAX / 10 && num > 7) || (res == INT_MIN / 10 && num < -8))
        	{
        		return 0;
        	}
        	res = res * 10 + num;
        }

        return res;
    }
};
```

