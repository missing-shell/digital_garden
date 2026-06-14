### 题目概述
#链表 
- 需保存反转开始节点的前一个节点`p0`
- 当`left==1`时，`p0`指针无法正确指向对应的节点（`p0`指向反转开始节点的前一个节点）
- 主要分为三部分
	找到反转开始节点的前一个节点
	使用`pre,cur,nxt`来实现链表反转，反转结束后，`pre`指向反转后节点的头节点，`cur`指向`pre->next`
	因为`p0`代表反转开始节点的前一个节点，所以反转链表后，`p0`应指向反转后节点的头节点`pre`，即`p0->next=pre`；反转前的开始节点（`p0->next`）应指向反转节点后面的节点（`cur`)，即`p0->next->next=cur`
#### 哨兵节点
- 使用[[Leetcode21.合并两个有序链表#哑节点(dummy node)| 哑节点(dummy node)]]中创建哨兵节点的方式有一定不足，`ListNode dummy;`中`dummy`是一个栈上分配的对象，如果链表很长，可能会导致*栈溢出*。
- 为了避免这个问题，可以使*用动态内存分配*来创建`dummy`节点
```c++
ListNode *dummy = new ListNode();  
dummy->next = head;
ListNode *p0 = dummy;

return dummy->next;  
```
### code
```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution
{
public:
    ListNode *reverseBetween(ListNode *head, int left, int right)
    {
        /*创建一个哨兵节点*/
        ListNode *dummy = new ListNode(0, head);
        ListNode *p0 = dummy;
        /*找到p0节点--反转链表开始前一个的节点*/
        for (int i = 0; i < left - 1; i++)
        {
            p0 = p0->next;
        }
        ListNode *pre = nullptr;
        ListNode *cur = p0->next;
        ListNode *nxt = nullptr;
        /*反转链表*/
        for (int i = 0; i < right - left + 1; i++)
        {
            nxt = cur->next;
            cur->next = pre;
            pre = cur;
            cur = nxt;
        }
        /*拼接反转后的节点*/
        p0->next->next = cur;//先拼接cur
        p0->next = pre;
        return dummy->next;
    }
};
```