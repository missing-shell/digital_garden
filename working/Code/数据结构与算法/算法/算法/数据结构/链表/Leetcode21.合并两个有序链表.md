### 题目描述
- #链表
- 链接：[21. 合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/)
####  哑节点(dummy node)
- 重点在于创建一个**虚拟头节点** `dummy`，其 `next` 指针指向实际的头节点。
- 使用一个指针 `cur` 来跟踪新链表的当前尾部。
- 正确地更新 `cur` 的 `next` 指针，并在每次循环结束时移动 `cur` 到新链表的尾部。
#### 从空状态开始构建
- 创建一个头节点和尾节点
```c++
 ListNode *head = nullptr, *tail = nullptr;
	if (!head) 
	{
		head = tail = new ListNode(sum % 10);
	} else {
		tail->next = new ListNode(sum % 10);
		tail = tail->next;
	}
```
#### 对比
- **使用哑节点**：适合需要频繁处理链表头部或链表可能为空的情况，可以简化代码逻辑。
- **不使用哑节点**：适合链表构建过程简单，仅需在尾部添加元素的情况，可以节省空间。
### code
#### 引用或直接连接
```c++
class Solution
{
public:
    /**
     * Definition for singly-linked list.
     * struct ListNode {
     *     int val;
     *     ListNode *next;
     *     ListNode(int x) : val(x), next(NULL) {}
     * };
     */

    /**
     * 合并两个有序链表。
     *
     * @param list1 第一个有序链表的头节点指针。
     * @param list2 第二个有序链表的头节点指针。
     * @return 返回合并后的新链表的头节点指针。
     *
     * 本函数通过比较两个链表的当前节点值，将较小的值加入到新链表中，
     * 以此方式合并两个有序链表，保持合并后的链表依然有序。
     */
	ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
	    ListNode dummy;
	    ListNode* cur = &dummy;
	    
	    while (list1 != nullptr && list2 != nullptr) {
	        if (list1->val < list2->val) {
	            cur->next = list1;
	            list1 = list1->next;
	        } else {
	            cur->next = list2;
	            list2 = list2->next;
	        }
	        cur = cur->next;
	    }
	    
	    // 直接连接剩余的部分
	    if (list1 != nullptr) {
	        cur->next = list1;
	    } else if (list2 != nullptr) {
	        cur->next = list2;
	    }
	    
	    return dummy.next;
	}
};
```
#### 创建一个新的链表存储合并结果
```c++
class Solution
{
public:
    /**
     * Definition for singly-linked list.
     * struct ListNode {
     *     int val;
     *     ListNode *next;
     *     ListNode(int x) : val(x), next(NULL) {}
     * };
     */

    /**
     * 合并两个有序链表。
     *
     * @param list1 第一个有序链表的头节点指针。
     * @param list2 第二个有序链表的头节点指针。
     * @return 返回合并后的新链表的头节点指针。
     *
     * 本函数通过比较两个链表的当前节点值，将较小的值加入到新链表中，
     * 以此方式合并两个有序链表，保持合并后的链表依然有序。
     */
    ListNode *mergeTwoLists(ListNode *list1, ListNode *list2)
    {
        // 使用一个哑节点(dummy)来简化链表操作，避免处理空链表的特殊情况
        ListNode dummy;
        // cur指针用于追踪新链表的当前尾节点
        ListNode *cur = &dummy;

        // 遍历两个链表，只要其中一个链表还有节点，就继续合并
        while (list1 != nullptr && list2 != nullptr)
        {
            // 比较两个链表当前节点的值，将较小的值加入到新链表中
            if (list1->val < list2->val)
            {
                cur->next = new ListNode(list1->val);
                list1 = list1->next;
            }
            else
            {
                cur->next = new ListNode(list2->val);
                list2 = list2->next;
            }
            // 移动cur指针到新链表的当前尾节点
            cur = cur->next;
        }

        // 处理剩余节点：如果有一个链表还有剩余节点，将剩余节点直接连接到新链表的尾部
        if (list1 != nullptr)
        {
            cur->next = list1;
        }
        else
        {
            cur->next = list2;
        }

        // 返回合并后的新链表的头节点（即哑节点的下一个节点）
        return dummy.next;
    }
};
```