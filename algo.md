### lc203

```python3
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def removeElements(self, head: ListNode, val: int) -> ListNode:
        if head is None:
            return head
        
        while head is not None:
            if head.val == val:
                head = head.next
            else:
                break
        
        item = head
        while head is not None and head.next is not None:
            if head.next.val == val:
                head.next = head.next.next
            else:
                head = head.next
                
        return item
```
