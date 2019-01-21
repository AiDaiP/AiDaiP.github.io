# LeetCode-Two Sum

什么都不会先搞一个最简单的试试

题目：给定一个整型数组，返回两个数的下标，满足两个数相加为一个特定整数。

例：

```
Given nums = [2, 7, 11, 15], target = 9,
Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1]
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

看一波时间复杂度是什么



时间复杂度是定性描述算法运行时间的函数，用O(f(n))表示，其中f(n)是算法基本操作重复执行次数函数T(n)的同数量级函数。

例：

```
int function(int n)
{
    printf("hello,world\n");//1
    return 0;
}//时间复杂度O(1)

int function(int n)
{
    for (int i = 0; i < n; ++i)//n
    {
        printf("hello,world\n");//1
    }
    return 0;
}//时间复杂度O(n)

int function(int n)
{
    for (int i = 0; i < n; ++i)//n
    {
        for (int j = 0; j < n; ++j)//n
        {
            printf("hello,world\n");//1
        }
    }
    return 0;
}
//时间复杂度O(n^2)

int function(int n)
{
    for (int i = 0; i < n; ++i)//n
    {
        for (int j = 0; j < n; ++j)//n
        {
            printf("hello,world\n");//1
            for(int k = 0; k < n; ++k)//n
            {
                printf("hello,world\n");//1
            }
        }
    }
    return 0;
}时间复杂度O(n^3)
```



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



