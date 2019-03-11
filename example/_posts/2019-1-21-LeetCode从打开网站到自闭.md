# LeetCode从打开网站到自闭

* #### 1. Two Sum

  Given an array of integers, return **indices** of the two numbers such that they add up to a specific target.

  You may assume that each input would have **exactly** one solution, and you may not use the *same* element twice.

  **Example:**

  ```
  Given nums = [2, 7, 11, 15], target = 9,
  
  Because nums[0] + nums[1] = 2 + 7 = 9,
  return [0, 1].
  ```

  穷举

  ```c
  int* twoSum(int* nums, int numsSize, int target) 
  {
      int *res = (int*)malloc(2*sizeof(int));
      for(int i = 0;i<numsSize;i++)
      {
          for(int j = i+1;j<numsSize && j != i;j++)
          {
              if(nums[i] + nums[j] == target)
              {
                  res[0] = i;
                  res[1] = j;
              }
          }
      }
      return res;
  }
  ```

  过了

  有些大佬说“平生不识TwoSum，刷尽LeetCode也枉然”，俺寻思这题不应该这么简单。

  查了一下发现有人说时间复杂度O(n^2)的过不了

  先排序再用二分法找符合条件的下标，时间复杂度是O(nlogn)

  也有时间复杂度是O(n)的算法，用到了哈希表

  大概了解一下哈希表，自己试了一波

  遍历数组建立映射，再遍历一遍查找target和数组内数字的差，找到则记录索引

  ```cpp
  class Solution 
  {
  public:
      vector<int> twoSum(vector<int>& nums, int target) 
      {
          unordered_map<int, int> m;
          vector<int> res;
          for (int i = 0; i < nums.size(); ++i) 
              m[nums[i]] = i;
          for (int i = 0; i < nums.size(); ++i) 
          {
              int t = target - nums[i];
              if (m.count(t) && m[t] != i) 
              {
                  res.push_back(i);
                  res.push_back(m[t]);
                  break;
              }
          }
          return res;
      }
  };
  ```

  

* #### 2. Add Two Numbers

  You are given two **non-empty** linked lists representing two non-negative integers. The digits are stored in **reverse order** and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

  You may assume the two numbers do not contain any leading zero, except the number 0 itself.

  **Example:**

  ```
  Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
  Output: 7 -> 0 -> 8
  Explanation: 342 + 465 = 807.
  ```

  遍历链表，取值相加（同时加上进位的值），sum/10得到当前节点的值，储存到新链表中，sum%10得到进位的值。

  l1，l2都到达尾端时循环结束，处理最后一次进位。

  ```c++
  /**
   * Definition for singly-linked list.
   * struct ListNode {
   *     int val;
   *     ListNode *next;
   *     ListNode(int x) : val(x), next(NULL) {}
   * };
   */
  class Solution 
  {
  public:
      ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) 
      {
          ListNode *res = new ListNode(0);
          ListNode *p1 = l1;
          ListNode *p2 = l2;
          ListNode *p3 = res;
          int carry = 0;
          int sum = 0;
  
          while (p1 != nullptr || p2 != nullptr)
          {
          	int x = 0;
          	int y = 0;
          	if (p1 != nullptr)
          	{
          		x = p1->val;
          		p1 = p1->next;
          	}
          	if (p2 != nullptr)
          	{
          		y = p2->val;
          		p2 = p2->next;
          	}
          	
          	sum = x + y + carry;
          	carry = sum / 10;
          	res->next = new ListNode(sum % 10);
          	res = res->next;
  
          }
          if (carry == 0)
          {
          	res->next = nullptr;
          }
          else
          {
          	res->next = new ListNode(carry);
          }
          
          return p3->next;
          
      }
  };
  ```

  

* #### 7. Reverse Integer

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

  

* #### 149. Max Points on a Line

  Given n points on a 2D plane, find the maximum number of points that lie on the same straight line.

  ```
  Input: [[1,1],[2,2],[3,3]]
  Output: 3
  Explanation:
  ^
  |
  |        o
  |     o
  |  o  
  +------------->
  0  1  2  3  4
  ```

  ```
  Input: [[1,1],[3,2],[5,3],[4,1],[2,3],[1,4]]
  Output: 4
  Explanation:
  ^
  |
  |  o
  |     o        o
  |        o
  |  o        o
  +------------------->
  0  1  2  3  4  5  6
  ```

  找到最多有多少个点在同一条直线上

  暴力统计，每次取一个点，计算其他点和这个点的斜率，找出有多少个斜率相同，遇到重复点计数不计算斜率。

  计算斜率时可能出现不是整数的情况，而且如果与x轴垂直无法计算斜率，所以用（x2-x1,y2-y1)除以最大公约数代替斜率，可以避开这些情况。

  使用map<pair<int, int>, int>储存用(x2-x1,y2-y1)表示的斜率和相同斜率点的个数。

  处理完斜率之后遍历map，相同斜率点加上重复点就是当前直线上相同点的个数，每次记录最大值。

  ```c++
  /**
   * Definition for a point.
   * struct Point {
   *     int x;
   *     int y;
   *     Point() : x(0), y(0) {}
   *     Point(int a, int b) : x(a), y(b) {}
   * };
   */
  class Solution
  {
  public:
  	int maxPoints(vector<Point>& points)
  	{
  		map<pair<int, int>, int> slopes;
  		int n = points.size();
  		if (n < 3)
  			return n;
  		int maxnum = 0;
  		for (int i = 0; i < n; i++)
  		{
  			int same = 1;
  			for (int j = i + 1; j < n; j++)
  			{
  				if (points[i].x == points[j].x && points[i].y == points[j].y)
  				{
  					same++;
  					continue;
  				}
  				int dx = points[j].x - points[i].x;
  				int dy = points[j].y - points[i].y;
  				slopes[make_pair(dx / gcd(dx, dy), dy / gcd(dx, dy))]++;
  			}
  			maxnum = max(maxnum, same);
  			for (auto _slopes : slopes)
  				if (_slopes.second + same > maxnum)
  					maxnum = _slopes.second + same;
  			slopes.clear();
  		}
  		return maxnum;
  	}
  	int gcd(int a, int b)
  	{
  		while (b)
  		{
  			int t = b;
  			b = a % b;
  			a = t;
  		}
  		return a;
  	}
  };
  ```

  

* #### 43. Multiply Strings

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

  ```c++
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

  

* #### 583. Delete Operation for Two Strings

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
  		vector<vector<int>> dp(n1 + 1, vector<int>(n2 + 1, 0));
  		for (int i = 1; i <= n1; i++)
  		{
  			for (int j = 1; j <= n2; j++)
  			{
  				if (word1[i - 1] == word2[j - 1])
  					dp[i][j] = dp[i - 1][j - 1] + 1;
  				else
  					dp[i][j] = max(dp[i - 1][j],dp[i][j - 1]);
  			}
  
  		}
  		int ans = n1 + n2 - 2 * dp[n1][n2];
  		return ans;
  	}
  };
  
  ```

  

* #### 455. Assign Cookies

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

  

* #### 718. Maximum Length of Repeated Subarray

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

  

* #### 463. Island Perimeter

  You are given a map in form of a two-dimensional integer grid where 1 represents land and 0 represents water.

  Grid cells are connected horizontally/vertically (not diagonally). The grid is completely surrounded by water, and there is exactly one island (i.e., one or more connected land cells).

  The island doesn't have "lakes" (water inside that isn't connected to the water around the island). One cell is a square with side length 1. The grid is rectangular, width and height don't exceed 100. Determine the perimeter of the island.

   

  **Example:**

  ```
  Input:
  [[0,1,0,0],
   [1,1,1,0],
   [0,1,0,0],
   [1,1,0,0]]
  
  Output: 16
  
  Explanation: The perimeter is the 16 yellow stripes in the image below:
  ```

  ![island](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/LeetCode/island.png)

  对每一格的四边分别处理就完事了

  ```c++
  class Solution 
  {
  public:
      int islandPerimeter(vector<vector<int>>& grid) 
      {
          int m = grid.size();
          int n = grid[0].size();
          int res = 0;
          for (int i = 0; i < m; i++) 
          {
              for (int j = 0; j < n; j++)
              {
                  if (grid[i][j] == 1) 
                  {
                  	if (j == 0 || grid[i][j - 1] == 0) 
                  	    res++;
                  	if (i == 0 || grid[i - 1][j] == 0) 
                  	    res++;
                  	if (j == n - 1 || grid[i][j + 1] == 0)
                  	    res++;
                  	if (i == m - 1 || grid[i + 1][j] == 0) 
                   	   res++;
                  }
              }
          }
          return res;
      }
  };
  ```

  

* #### 237. Delete Node in a Linked List

  Write a function to delete a node (except the tail) in a singly linked list, given only access to that node.

  Given linked list -- head = [4,5,1,9], which looks like following:

  ![img](https://assets.leetcode.com/uploads/2018/12/28/237_example.png)

   

  **Example 1:**

  ```
  Input: head = [4,5,1,9], node = 5
  Output: [4,1,9]
  Explanation: You are given the second node with value 5, the linked list should become 4 -> 1 -> 9 after calling your function.
  ```

  **Example 2:**

  ```
  Input: head = [4,5,1,9], node = 1
  Output: [4,5,9]
  Explanation: You are given the third node with value 1, the linked list should become 4 -> 5 -> 9 after calling your function.
  ```

   

  **Note:**

  - The linked list will have at least two elements.
  - All of the nodes' values will be unique.
  - The given node will not be the tail and it will always be a valid node of the linked list.
  - Do not return anything from your function.

  把下一个节点的值和next指针赋给被删除的节点就完事了

  ```c++
  /**
   * Definition for singly-linked list.
   * struct ListNode {
   *     int val;
   *     ListNode *next;
   *     ListNode(int x) : val(x), next(NULL) {}
   * };
   */
  class Solution 
  {
  public:
      void deleteNode(ListNode* node) 
      {
          node->val = node->next->val;
          node->next = node->next->next;
      }
  };
  ```

  

* #### 206. Reverse Linked List

  Reverse a singly linked list.

  **Example:**

  ```
  Input: 1->2->3->4->5->NULL
  Output: 5->4->3->2->1->NULL
  ```

  **Follow up:**

  A linked list can be reversed either iteratively or recursively. Could you implement both？

  * 迭代

    三个指针分别指向前节点，当前节点，后节点，遍历链表，使当前节点的next指针指向前节点，前节点指针指向当前节点，然后当前节点和后节点分别向下一节点移动。

    ```c++
    /**
     * Definition for singly-linked list.
     * struct ListNode {
     *     int val;
     *     ListNode *next;
     *     ListNode(int x) : val(x), next(NULL) {}
     * };
     */
    class Solution 
    {
    public:
        ListNode* reverseList(ListNode* head) 
        {
            if (head == NULL)
            {
                return NULL;
            }
            
            ListNode *curNode = head;
            ListNode *preNode = nullptr;
            ListNode *nextNode = head->next;
            while (nextNode != nullptr)
            {
                curNode->next = preNode;
                preNode = curNode;
                curNode = nextNode;
                nextNode = curNode->next;
    
            }
            curNode->next = preNode;
            return curNode;
        }
    };
    ```

  * 递归

    从后向前翻转，每次使当前节点的下一节点的next指针指向当前节点，再把当前节点的next指针设为null

    ```c++
    /**
     * Definition for singly-linked list.
     * struct ListNode {
     *     int val;
     *     ListNode *next;
     *     ListNode(int x) : val(x), next(NULL) {}
     * };
     */
    class Solution
    {
    public:
        ListNode* reverseList(ListNode* head) 
        {
            if (head == NULL || head->next == NULL)
            {
                return head;
            }
            
            ListNode *curNode = head;
            ListNode *p = reverseList(curNode->next);
            curNode->next->next = curNode;
            curNode->next = NULL;
            return p;
        }
    };
    
    ```

    
