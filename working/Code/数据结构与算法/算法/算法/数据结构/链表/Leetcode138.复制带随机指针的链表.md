### 概述
#哈希表  #链表 
给定链表的头节点 `head` ，复制普通链表很简单，只需遍历链表，每轮建立新节点 `+` 构建前驱节点` pre `和当前节点` node `的引用指向即可。

本题链表的节点新增了 `random` 指针，指向链表中的 任意节点 或者` null `。这个` random `指针意味着在复制过程中，除了构建前驱节点和当前节点的引用指向 `pre.next `，还要构建前驱节点和其随机节点的引用指向 `pre.random` 。

**本题难点：** 在复制链表的过程中构建新链表各节点的 `random` 引用指向。
#### 法1：哈希表
利用哈希表的查询特点，构建 **原链表节点** 和 **新链表对应节点** 的键值对映射关系，再遍历构建新链表各节点的 `next` 和 `random` 引用指向即可。
**流程**：
- 若头节点 `head` 为空节点，直接返回 `null `。
- 初始化： 哈希表 `dic` ， 节点 `cur `指向头节点。
- 复制链表：
	建立新节点，并向 `dic `添加键值对 (原 `cur `节点, 新 `cur `节点） 。
	`cur` 遍历至原链表下一节点。
- 构建新链表的引用指向：
	构建新节点的 `next` 和 `random` 引用指向。
	`cur` 遍历至原链表下一节点。
- 返回值： 新链表的头节点 `dic[cur]` 。
**复杂度**：
- **时间复杂度 $O(N)$ ：** 两轮遍历链表，使用 $O(N)$ 时间。
- **空间复杂度 $O(N)$ ：** 哈希表 `dic` 使用线性大小的额外空间。
#### 法2：拼接+拆分
构建 `原节点 1 -> 新节点 1 -> 原节点 2 -> 新节点 2 -> …… `的拼接链表，如此便可在访问原节点的 `random` 指向节点的同时找到新对应新节点的 `random `指向节点。
**流程**：
- 复制各节点，构建拼接链表：设原链表为 `node1→node2→⋯ `，构建的拼接链表如下所示：
$$node1→node1//new→node2→node2//new →⋯$$
- 构建新链表各节点的 `random` 指向：当访问原节点 `cur `的随机指向节点 `cur.random` 时，对应新节点 `cur.next` 的随机指向节点为 `cur.random.next` 。
- 拆分原 / 新链表：设置 `pre / cur `分别指向原 / 新链表头节点，遍历执行 `pre.next = pre.next.next` 和 `cur.next = cur.next.next` 将两链表拆分开。
- 返回新链表的头节点 `res` 即可。
**复杂度**：
- **时间复杂度 $O(N)$ ：** 三轮遍历链表，使用 $O(N)$ 时间。
- **空间复杂度 $O(1)$ ：** 节点引用变量使用常数大小的额外空间。
### code
#### 哈希表
```c++
/*
// Definition for a Node.
class Node {
public:
    int val;
    Node* next;
    Node* random;

    Node(int _val) {
        val = _val;
        next = NULL;
        random = NULL;
    }
};
*/
/**
 * 复制一个带有随机指针的链表。
 * 
 * @param head 原始链表的头节点指针。
 * @return 复制链表的头节点指针。
 */
Node *copyRandomList(Node *head)
{
    // 如果原始链表为空，则返回空。
    if (head == nullptr)
    {
        return nullptr;
    }
    
    // 用于遍历原始链表的指针。
    Node *cur = head;
    // 用于存储原始节点和对应复制节点的映射关系。
    unordered_map<Node *, Node *> map;
    
    // 遍历原始链表，创建复制节点，并建立映射关系。
    while (cur != nullptr)
    {
        // 对于当前节点cur，创建一个值相同的复制节点，并将其添加到映射中。
        map[cur] = new Node(cur->val);
        // 移动到下一个节点。
        cur = cur->next;
    }
    
    // 重新遍历原始链表，设置复制节点的next和random指针。
    cur = head;
    while (cur != nullptr)
    {
        // 设置复制节点的next指针，指向对应原始节点的下一个节点的复制节点。
        map[cur]->next = map[cur->next];
        // 设置复制节点的random指针，指向对应原始节点的随机节点的复制节点。
        map[cur]->random = map[cur->random];
        // 移动到下一个节点。
        cur = cur->next;
    }
    
    // 返回复制链表的头节点，即原始链表头节点的复制节点。
    return map[head];
}
```
#### 拼接+拆分
```c++
/*
// Definition for a Node.
class Node {
public:
    int val;
    Node* next;
    Node* random;

    Node(int _val) {
        val = _val;
        next = NULL;
        random = NULL;
    }
};
*/

class Solution
{
public:
/**
 * 复制一个带有随机指针的链表。
 * 
 * @param head 原始链表的头节点。
 * @return 返回复制链表的头节点。
 */
Node *copyRandomList(Node *head)
{
    // 如果原始链表为空，则返回空。
    if (head == nullptr)
    {
        return nullptr;
    }
    
    // 遍历原始链表，在每个节点后插入一个复制的节点。
    Node *cur = head;
    Node *tmp = nullptr;
    while (cur != nullptr)
    {
        tmp = new Node(cur->val);
        tmp->next = cur->next;
        cur->next = tmp;
        cur = tmp->next;
    }
    
    // 遍历新链表，设置复制节点的随机指针。
    cur = head;
    while (cur != nullptr)
    {
        if (cur->random != nullptr)
        {
            cur->next->random = cur->random->next;
        }
        cur = cur->next->next;
    }
    
    // 分离复制的链表，并重新设置原链表的连接关系。
    cur = head->next;
    Node *pre = head, *res = head->next;
    while (cur->next != nullptr)
    {
        pre->next = pre->next->next;
        cur->next = cur->next->next;
        pre = pre->next;
        cur = cur->next;
    }
    
    // 结束复制链表的连接，防止形成环。
    pre->next = nullptr;
    
    // 返回复制链表的头节点。
    return res;
}
};
```