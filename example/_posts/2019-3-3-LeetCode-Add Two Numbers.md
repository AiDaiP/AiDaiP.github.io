# Add Two Numbers

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

