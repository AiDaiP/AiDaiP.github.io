# LeetCode-Multiply Strings

Given two non-negative integers `num1` and `num2` represented as strings, return the product of `num1` and `num2`, also represented as a string.

**Example 1:**

```
Input: num1 = "2", num2 = "3"
Output: "6"
```

**Example 2:**

```
Input: num1 = "123", num2 = "456"
Output: "56088"
```

输入两个字符串，返回对应数字的乘积，返回的乘积也是字符串

每一位相乘再相加

字符串的每一位减去‘0’可以得到这一位的数字，每一位相乘，乘积储存到一个数组中，角标相同的位置相同，求和。先不考虑进位，计算每一位之间的乘积后计算进位。若有进位后一位加上前一位与10的模，前一位与10取余。

计算结束后每一位加上‘0'得到结果的字符串。

```
class Solution 
{
public:
	string multiply(string num1, string num2) 
	{
		int n1 = num1.size();
		int n2 = num2.size();
		int n = n1 + n2 - 2, k = 0;
		int len = n1 + n2 - 1;
		string ans;
		vector<int> num(n1 + n2, 0);
		for (int i = 0; i < n1; ++i)
			for (int j = 0; j < n2; ++j)
				num[n - i - j] += (num1[i] - '0') * (num2[j] - '0');
		for (int i = 0; i < n1 + n2; ++i) 
		{
			num[i] += k;
			k = num[i] / 10;
			num[i] %= 10;
		}
		while (num[len] == 0)
			len--;
		if (len < 0)
			ans = "0";
		while (len >= 0)
		{
			ans.push_back(num[len] + '0');
			len--;
		}
		return ans;
	}
};
```

