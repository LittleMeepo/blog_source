## 链表

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

