## 链表

#### Leetcode 146 [LRU缓存机制](https://leetcode-cn.com/problems/lru-cache/)

> 运用你所掌握的数据结构，设计和实现一个 [LRU (最近最少使用) 缓存机制](https://baike.baidu.com/item/LRU) 。
>
> 实现 LRUCache 类：
>
> - LRUCache(int capacity) 以正整数作为容量 capacity 初始化 LRU 缓存
> - int get(int key) 如果关键字 key 存在于缓存中，则返回关键字的值，否则返回 -1 。
> - void put(int key, int value) 如果关键字已经存在，则变更其数据值；如果关键字不存在，则插入该组「关键字-值」。当缓存容量达到上限时，它应该在写入新数据之前删除最久未使用的数据值，从而为新的数据值留出空间。

```c++
struct DLinkedNode {
    int key, value;
    DLinkedNode* prev;
    DLinkedNode* next;
    DLinkedNode(): key(0), value(0), prev(nullptr), next(nullptr) {}
    DLinkedNode(int _key, int _value): key(_key), value(_value), prev(nullptr), next(nullptr) {}
};

class LRUCache {
private:
    unordered_map<int, DLinkedNode*> cache;
    DLinkedNode* head;
    DLinkedNode* tail;
    int size;
    int capacity;

public:
    LRUCache(int _capacity): capacity(_capacity), size(0) {
        // 使用伪头部和伪尾部节点
        head = new DLinkedNode();
        tail = new DLinkedNode();
        head->next = tail;
        tail->prev = head;
    }
    
    int get(int key) {
        if (!cache.count(key)) {
            return -1;
        }
        // 如果 key 存在，先通过哈希表定位，再移到头部
        DLinkedNode* node = cache[key];
        moveToHead(node);
        return node->value;
    }
    
    void put(int key, int value) {
        if (!cache.count(key)) {
            // 如果 key 不存在，创建一个新的节点
            DLinkedNode* node = new DLinkedNode(key, value);
            // 添加进哈希表
            cache[key] = node;
            // 添加至双向链表的头部
            addToHead(node);
            ++size;
            if (size > capacity) {
                // 如果超出容量，删除双向链表的尾部节点
                DLinkedNode* removed = removeTail();
                // 删除哈希表中对应的项
                cache.erase(removed->key);
                // 防止内存泄漏
                delete removed;
                --size;
            }
        }
        else {
            // 如果 key 存在，先通过哈希表定位，再修改 value，并移到头部
            DLinkedNode* node = cache[key];
            node->value = value;
            moveToHead(node);
        }
    }

    void addToHead(DLinkedNode* node) {
        node->prev = head;
        node->next = head->next;
        head->next->prev = node;
        head->next = node;
    }
    
    void removeNode(DLinkedNode* node) {
        node->prev->next = node->next;
        node->next->prev = node->prev;
    }

    void moveToHead(DLinkedNode* node) {
        removeNode(node);
        addToHead(node);
    }

    DLinkedNode* removeTail() {
        DLinkedNode* node = tail->prev;
        removeNode(node);
        return node;
    }
};
```



#### Leetcode 148 [排序链表](https://leetcode-cn.com/problems/sort-list/)

> 在 *O*(*n* log *n*) 时间复杂度和常数级空间复杂度下，对链表进行排序。

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
class Solution {
public:
	ListNode* sortList(ListNode* head) {
		int length = getLength(head);
		ListNode* dummy = new ListNode(-1);
		dummy->next = head;

		for (int step = 1; step < length; step *= 2)
		{
			ListNode* pre = dummy;
			ListNode* cur = dummy->next;
			while (cur != nullptr)
			{
				ListNode* h1 = cur;
				ListNode* h2 = splitList(h1, step);
				cur = splitList(h2, step);
				ListNode* tmpMerge = mergeList(h1, h2);
				pre->next = tmpMerge;
				while (pre->next != nullptr)
				{
					pre = pre->next;
				}
			}
		}
		return dummy->next;
	}

	ListNode* splitList(ListNode* head, int step)
	{
		if (head == nullptr) return nullptr;
		ListNode* dummy = new ListNode(-1);
		dummy->next = head;
		ListNode* pre = dummy;
		while (step--)
		{
			if (head!= nullptr)
			{
				head = head->next;
				pre = pre->next;
			}
			else break;
		}
		pre->next = nullptr;
		return head;
	}

	ListNode* mergeList(ListNode* h1, ListNode* h2)
	{
		ListNode* dummy = new ListNode(-1);
		ListNode* pre = dummy;
		while (h1 != nullptr && h2 != nullptr)
		{
			if (h1->val < h2->val)
			{
				pre->next = h1;
				h1 = h1->next;
			}
			else
			{
				pre->next = h2;
				h2 = h2->next;
			}
			pre = pre->next;
		}
		if (h1 != nullptr) pre->next = h1;
		if (h2 != nullptr) pre->next = h2;
		return dummy->next;
	}

	int getLength(ListNode* head)
	{
		int len = 0;
		while (head != nullptr)
		{
			len++;
			head = head->next;
		}
		return len;
	}
};
```

用迭代的方法，进行归并

#### Leetcode 206 [反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)

> 反转一个单链表。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode prev = null;
        ListNode curr = head;
        while (curr != null) {
            ListNode nextTemp = curr.next;
            curr.next = prev;
            prev = curr;
            curr = nextTemp;
        }
        return prev;
    }
}
```

