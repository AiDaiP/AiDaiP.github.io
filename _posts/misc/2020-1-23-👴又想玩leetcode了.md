---
layout: post
title:  "👴又想玩leetcode了"
date:   2020-1-23
desc: ""
keywords: ""
categories: [Misc]
tags: []
icon: icon-html
---

# 👴又想玩leetcode了

### 20. Valid Parentheses

```
Given a string containing just the characters '(', ')', '{', '}', '[' and ']', determine if the input string is valid.

An input string is valid if:

Open brackets must be closed by the same type of brackets.
Open brackets must be closed in the correct order.
Note that an empty string is also considered valid.

Example 1:

Input: "()"
Output: true
Example 2:

Input: "()[]{}"
Output: true
Example 3:

Input: "(]"
Output: false
Example 4:

Input: "([)]"
Output: false
Example 5:

Input: "{[]}"
Output: true
```

左括号压栈，右括号来了能匹配就弹出去，不能匹配就压栈

```c++
class Solution {
public:
    bool isValid(string s) {
        int len = s.size();
        for (int i = 0; i < len; ++i)
            if (!st.empty())
                if (check(st.top(), s[i]))
                    st.pop();
                else
                    st.push(s[i]);
            else
                st.push(s[i]);
        if (!st.empty())
            return false;
        else
            return true;
    }
private:
    stack<char> st;
    bool check(char a, char b)
    {
        if (a == '[' && b == ']' || a == '(' && b == ')' || a == '{' && b == '}')
            return true;
        else
            return false;
    }
};
```



### 26. Remove Duplicates from Sorted Array

```
Given a sorted array nums, remove the duplicates in-place such that each element appear only once and return the new length.

Do not allocate extra space for another array, you must do this by modifying the input array in-place with O(1) extra memory.

Example 1:

Given nums = [1,1,2],

Your function should return length = 2, with the first two elements of nums being 1 and 2 respectively.

It doesn't matter what you leave beyond the returned length. Example 2:

Given nums = [0,0,1,1,1,2,2,3,3,4],

Your function should return length = 5, with the first five elements of nums being modified to 0, 1, 2, 3, and 4 respectively.

It doesn't matter what values are set beyond the returned length. Clarification:

Confused why the returned value is an integer but your answer is an array?

Note that the input array is passed in by reference, which means modification to the input array will be known to the caller as well.

Internally you can think of this:

// nums is passed in by reference. (i.e., without making a copy)
int len = removeDuplicates(nums);

// any modification to nums in your function would be known by the caller.
// using the length returned by your function, it prints the first len elements.
for (int i = 0; i < len; i++) {
    print(nums[i]);
}
```

#### unique

重排容器unique，将重复的元素排到最后，返回不重复区域之后的一个位置迭代器，调用erase删除

```c++
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        auto it = unique(nums.begin(), nums.end());
        nums.erase(it, nums.end());
        return nums.size();
    }
};
```

#### 快慢指针

两个指针指向数字相同快指针+1，不同慢指针+1然后把快指针指向的数字赋给慢指针

```c++
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        if(nums.size()==0) 
            return 0;
        int slow = 0;
        for (int fast = 1; fast < nums.size(); ++fast)
            if (nums[fast] != nums [slow])
            {
                slow++;
                nums[slow] = nums[fast];
            }
        return slow + 1;
    }
};
```



### 53. Maximum Subarray

```
Given an integer array nums, find the contiguous subarray (containing at least one number) which has the largest sum and return its sum.

Example:

Input: [-2,1,-3,4,-1,2,1,-5,4],
Output: 6
Explanation: [4,-1,2,1] has the largest sum = 6.
Follow up:

If you have figured out the O(n) solution, try coding another solution using the divide and conquer approach, which is more subtle.
```

#### 暴力

```c++
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int maxnum = nums[0];
        for (int i = 0; i < nums.size(); ++i)
        {
            int temp = 0;
            for (int j = i; j < nums.size(); ++j)
            {
                temp += nums[j];
                if (temp > maxnum)
                    maxnum = temp;
	        }
        }
        return maxnum;
    }
};
```

#### 动态规划

dp[i]表示以nums[i]结尾的最大子序和

```
dp[i] = max(dp[i-1]+nums[i],nums[i])
```

```c++
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        vector<int> dp(nums.size());
        dp[0] = nums[0];
        int maxnum = nums[0];
        for (int i = 1; i < nums.size(); ++i)
        {
            dp[i] = max(dp[i-1]+nums[i], nums[i]);
            maxnum = max(maxnum, dp[i]);
        }
        return maxnum;
    }
};
```

```c++
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
    	int dp;
        dp = nums[0];
        int maxnum = nums[0];
        for (int i = 1; i < nums.size(); ++i)
        {
            dp = max(dp+nums[i], nums[i]);
            maxnum = max(maxnum, dp);
        }
        return maxnum;
    }
};
```



#### 贪心

sum<0时从下一个位置开始找子序

```c++
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int maxnum = nums[0];
        int sum = 0;
        for (int i = 0; i < nums.size(); ++i)
        {
        	sum += nums[i];
        	maxnum = max(maxnum, sum);
        	if (sum < 0)
        		sum = 0;
        }
        return maxnum;
    }
};
```



### 88. Merge Sorted Array

```
Given two sorted integer arrays nums1 and nums2, merge nums2 into nums1 as one sorted array.

Note:

The number of elements initialized in nums1 and nums2 are m and n respectively.
You may assume that nums1 has enough space (size that is greater or equal to m + n) to hold additional elements from nums2.
Example:

Input:
nums1 = [1,2,3,0,0,0], m = 3
nums2 = [2,5,6],       n = 3

Output: [1,2,2,3,5,6]
```

#### 合并排序

nums2插到nums后面然后排序

```c++
class Solution {
public:
    void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {
        for(int i=m,j=0; i<m+n,j<n; ++i,++j)
            nums1[i]=nums2[j];
        sort(nums1.begin(),nums1.end());
    }
};

```

