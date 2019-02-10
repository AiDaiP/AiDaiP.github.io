# LeetCode-Maximum Length of Repeated Subarray

Given two integer arrays `A` and `B`, return the maximum length of an subarray that appears in both arrays.

**Example 1:**

```
Input:
A: [1,2,3,2,1]
B: [3,2,1,4,7]
Output: 3
Explanation: 
The repeated subarray with maximum length is [3, 2, 1].
```

**Note:**

1. 1 <= len(A), len(B) <= 1000
2. 0 <= A[i], B[i] < 100

动态规划，如果A[i-1]和B[j-1]相等，dp\[i][j]等于它左上角值加一

```c++
class Solution 
{
public:
    int findLength(vector<int>& A, vector<int>& B) 
    {
		int n1 = A.size();
		int n2 = B.size();
		int ans = 0;
		vector<vector<int>> dp(n1 + 1, vector<int>(n2 + 1, 0));
		for (int i = 1; i <= n1; i++)
		{
			for (int j = 1; j <= n2; j++)
			{
				if (A[i - 1] == B[j - 1])
					dp[i][j] = dp[i - 1][j - 1] + 1;
				else
					dp[i][j] = 0;
				ans = max(ans,dp[i][j]);
			}
		}
		return ans;
    }
};
```

