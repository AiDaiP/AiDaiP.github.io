# LeetCode-Delete Operation for Two Strings

Given two words *word1* and *word2*, find the minimum number of steps required to make *word1* and *word2* the same, where in each step you can delete one character in either string.

**Example 1:**

```
Input: "sea", "eat"
Output: 2
Explanation: You need one step to make "sea" to "ea" and another step to make "eat" to "ea".
```

**Note:**

1. The length of given words won't exceed 500.
2. Characters in given words can only be lower-case letters.

有两个字符串，每次只能从其中一个字符串中删除一个字符，最少多少步可以使两个字符串相同。

其实就是找两个字符串最长相同子序列

用两个字符串的长度和减去最长相同子序列的长度的二倍就是答案。

用`len[i][j]`表示word1的前i位和word2的前j位最长相同子序列的长度，每次循环比较一位，若字母相同则word1的前i位和word2的前j位最长相同子序列的长度等于word1的前i-1位和word2的前j-1位最长相同子序列的长度加1，若不相同则word1的前i位和word2的前j位最长相同子序列的长度等于word1的前i-1位和word2的前j位最长相同子序列的长度和word1的前i位和word2的前j-1位最长相同子序列的长度中的较大值

```c++
class Solution
{
public:
	int minDistance(string word1, string word2)
	{
		int n1 = word1.size();
		int n2 = word2.size();
		vector<vector<int>> len(n1 + 1, vector<int>(n2 + 1, 0));
		for (int i = 1; i <= n1; i++)
		{
			for (int j = 1; j <= n2; j++)
			{
				if (word1[i - 1] == word2[j - 1])
					len[i][j] = len[i - 1][j - 1] + 1;
				else
					len[i][j] = max(len[i - 1][j],len[i][j - 1]);
			}

		}
		int ans = n1 + n2 - 2 * len[n1][n2];
		return ans;
	}
};

```

