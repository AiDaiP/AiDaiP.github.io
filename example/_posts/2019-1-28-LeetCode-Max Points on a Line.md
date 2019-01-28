# LeetCode-Max Points on a Line

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

```C++
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

