### 题目描述
- #链表
- 链接：[82. 删除排序链表中的重复元素 II](https://leetcode.cn/problems/remove-duplicates-from-sorted-list-ii/)

- 仔细阅读题目要求和示例，题目要求的是删除重复元素而不是去重
- 创建三个节点：`*pre=dummy`,`*cur=head`,`*nxt=cur->next`
- 首先找到`cur`不为重复节点的情况

- 一步一步考虑，在本轮循环只需要确保当前`cur`指向的不是重复节点，不需要考虑`nxt`指向的是不是重复节点

- 如果`cur->next==nxt`，那就说明`cur`节点不是重复节点，此时更新`pre->next=cur,pre=cur`

- 反之则说明`cur`节点为重复节点，则跳过该节点`pre->next=nxt`---此处不需要更新`pre=nxt`，因为并不知道`nxt`是否为重复节点
	
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
    ListNode *deleteDuplicates(ListNode *head)
    {
        if (head == nullptr)
        {
            return head;
        }

        ListNode *dummy = new ListNode(0, head);
        ListNode *pre = dummy, *cur = head;

        while (cur != nullptr)
        {
            // 找到第一个不同的元素
            ListNode *nxt = cur->next;
            while (nxt != nullptr && nxt->val == cur->val)
            {
                nxt = nxt->next;
            }

            // 如果 cur 指向的不是重复元素，则更新 pre 和 cur
            if (cur->next == nxt)
            {
                pre->next = cur;
                pre = cur;
            }
            // 如果 cur 指向的是重复元素，则直接跳过这些元素
            else
            {
                pre->next = nxt;
            }

            cur = nxt;
        }

        return dummy->next;
    }
};
```