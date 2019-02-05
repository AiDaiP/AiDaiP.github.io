# LeetCode-Assign Cookies

Assume you are an awesome parent and want to give your children some cookies. But, you should give each child at most one cookie. Each child i has a greed factor gi, which is the minimum size of a cookie that the child will be content with; and each cookie j has a size sj. If sj >= gi, we can assign the cookie j to the child i, and the child i will be content. Your goal is to maximize the number of your content children and output the maximum number.

**Note:**
You may assume the greed factor is always positive. 
You cannot assign more than one cookie to one child.

**Example 1:**

```
Input: [1,2,3], [1,1]

Output: 1

Explanation: You have 3 children and 2 cookies. The greed factors of 3 children are 1, 2, 3. 
And even though you have 2 cookies, since their size is both 1, you could only make the child whose greed factor is 1 content.
You need to output 1.
```

**Example 2:**

```
Input: [1,2], [1,2,3]

Output: 2

Explanation: You have 2 children and 3 cookies. The greed factors of 2 children are 1, 2. 
You have 3 cookies and their sizes are big enough to gratify all of the children, 
You need to output 2.
```

给一堆小孩分小饼干，饼干有大小，小孩有食量，求已有饼干能让几个小孩吃饱

先排序，饼干从小到大排序，小孩从吃的少到吃的多排序。排序后用最小的饼干和吃的最少的小孩比较，能吃饱结果加一，用下一块饼干和下一位小孩比较，如果吃不饱就换下一块饼干和当前小孩比较。

```c++
class Solution 
{
public:
    int findContentChildren(vector<int>& g, vector<int>& s) 
    {
         int num = 0, ans = 0;
         sort(g.begin(), g.end());
         sort(s.begin(), s.end());
         for (int i = 0; i < s.size() && num < g.size(); i++) 
         {
             if (s[i] >= g[num]) 
                {
                    num++;
                    ans++;
                } 
         }
         return ans;
    }
};
```

