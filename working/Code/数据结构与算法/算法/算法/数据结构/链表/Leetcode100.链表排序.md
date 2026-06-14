## 概述
### 定义
#### CPP
```CPP
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
```
#### C
```C
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */

typedef struct ListNode *next ListNode_t;
```
## 冒泡
### 交换值
- 超时
```c++
class Solution {
public:
    ListNode* sortList(ListNode* head) {
        if(!head)return nullptr;

        auto dummyNode=new ListNode(-1);
        dummyNode->next=head;

        int cnt=0;
        auto node=head;
        while(node){
            cnt++;
            node=node->next;
        }

        while(cnt--){
            auto pre=dummyNode;
            auto cur=dummyNode->next;
            while(cur&&cur->next){
                if(cur->val>cur->next->val){
                    swap(cur->val,cur->next->val);
                }
                pre=cur;
                cur=cur->next;
            }
        }

        auto res=dummyNode->next;
        delete dummyNode;
        return res;

    }
};
```
### 交换节点
- 超时
```c++
class Solution {
public:
    ListNode* sortList(ListNode* head) {
        if(!head)return nullptr;

        auto dummyNode=new ListNode(-1);
        dummyNode->next=head;

        int cnt=0;
        auto node=head;
        while(node){
            cnt++;
            node=node->next;
        }

        while(cnt--){
            auto pre=dummyNode;
            auto cur=dummyNode->next;
            bool finished=true;

            while(cur&&cur->next){
                auto left=cur;
                auto right=cur->next;
                bool swap_flag=false;

                if(left->val>right->val){
                    swap_flag=true;
                    finished=false;
                    auto tmp=right->next;
                    pre->next=right;
                    right->next=left;
                    left->next=tmp;
                }
                pre=(swap_flag)? right : left;
                cur=(swap_flag)? left : right;
            }
            if(finished){
                break;
            }
        }
    
        auto res=dummyNode->next;
        delete dummyNode;
        return res;
    }
};
```
## 选择
- 超时
```c++
class Solution {
public:
    ListNode* sortList(ListNode* head) {
        if(!head)return nullptr;

        auto dummyNode=new ListNode(-1);
        dummyNode->next=head;

        int cnt=0;
        auto node=head;
        while(node){
            cnt++;
            node=node->next;
        }

        auto pre=head;
        while(cnt--){
            auto minNode=pre;
            auto cur=pre;
            while(cur){
                if(cur->val<minNode->val){
                    minNode=cur;
                }
                cur=cur->next;
            }
            swap(minNode->val,pre->val);
            pre=pre->next;
        }
       
        auto res=dummyNode->next;
        delete dummyNode;
        return res;
    }
};
```
## 插入
```C++
class Solution {
public:
    ListNode* sortList(ListNode* head) {
        if(!head||!head->next){
            return head;// 如果链表为空或只有一个节点，直接返回
        }

        auto dummyNode=new ListNode(-1);
        dummyNode->next=head;

        auto tail=head;// 已排序部分的最后一个节点
        auto cur=head->next;// 当前需要插入的节点

        while(cur){
            if(cur->val>=tail->val){
                // 当前节点值大于等于尾节点值，不需要移动
                tail=cur;
                cur=cur->next;
            }else{
                // 找到插入位置
                auto pre=dummyNode;
                while(pre->next->val<cur->val){
                    pre=pre->next;
                }

                // 插入节点
                tail->next=cur->next;
                cur->next=pre->next;
                pre->next=cur;
                
                // 移动到下一个节点
                cur=tail->next;
            }
        }

        auto res=dummyNode->next;
        delete dummyNode;
        return res;
    }
};
```
## 归并 ★
### 自顶向下
自顶向下的归并排序采用**递归**的方法，将链表分成两个子链表，分别排序后再合并。
```c++
class Solution {
public:
    ListNode* sortList(ListNode* head) {
        if(!head||!head->next){
            return head;// 如果链表为空或只有一个节点，直接返回
        }

        //找到中间节点
        ListNode *mid=middleNode(head);

        //分割链表
        ListNode *right=mid->next;
        mid->next=nullptr;

        //递归排序左右两部分
        ListNode *left=sortList(head);
        right=sortList(right);

        return merge(left,right);
    }
private:
    //寻找中间节点
    ListNode* middleNode(ListNode *head){
        auto slow=head;
        auto fast=head;
        while(fast->next&&fast->next->next){
            slow=slow->next;
            fast=fast->next->next;
        }
        return slow;
    }

    //合并两个有序链表
    ListNode *merge(ListNode *l1,ListNode *l2){
        auto dummy=new ListNode(-1);
        auto cur=dummy;

        while(l1&&l2){
            if(l1->val<l2->val){
                cur->next=l1;
                l1=l1->next;
            }else{
                cur->next=l2;
                l2=l2->next;
            }
            cur=cur->next;
        }
        if(l1){
            cur->next=l1;
        }
        if(l2){
            cur->next=l2;
        }

        ListNode *res=dummy->next;
        delete dummy;
        return res;
    }
};
```
### 自底向上
自底向上的归并排序采用**迭代**的方法，先将链表分成一个个小段，每段内部排序，然后再逐步合并成更大的段，直到整个链表排序完成。
```c++
class Solution {
public:
    ListNode* sortList(ListNode* head) {
        if(!head||!head->next){
            return head;// 如果链表为空或只有一个节点，直接返回
        }

        //计算链表长度
        int length=0;
        ListNode *node=head;
        while(node){
            length++;
            node=node->next;
        }

        ListNode *dummy=new ListNode(-1);
        dummy->next=head;

        //自底向上归并排序
        for(int subLength=1;subLength<length;subLength<<=1){
            auto pre=dummy;
            auto cur=dummy->next;

            while(cur){
                auto head1=cur;
                auto head2=split(cur,subLength);
                cur=split(head2,subLength);

                // 合并两个有序链表，并连接到 pre 的后面
                pre->next=merge(head1,head2);
                
                // 移动 pre 到合并后链表的末尾
                while(pre->next){
                    pre=pre->next;
                }
            }
        }

        auto res=dummy->next;
        delete dummy;
        return res;
    }
private:
    // 分割链表，返回第 subLength + 1 个节点
    ListNode* split(ListNode* head, int subLength){
        for(int i=1;head&&i<subLength;++i){
            head=head->next;
        }

        if(!head){
            return nullptr;
        }

        auto second=head->next;
        head->next=nullptr;
        return second;
    }
     

    //合并两个有序链表
    ListNode *merge(ListNode *l1,ListNode *l2){
        auto dummy=new ListNode(-1);
        auto cur=dummy;

        while(l1&&l2){ 
            if(l1->val<l2->val){
                cur->next=l1;
                l1=l1->next;
            }else{
                cur->next=l2;
                l2=l2->next;
            }
            cur=cur->next;
        }
        if(l1){
            cur->next=l1;
        }
        if(l2){
            cur->next=l2;
        }

        ListNode *res=dummy->next;
        delete dummy;
        return res;
    }
};
```
## 快排
## 堆排
