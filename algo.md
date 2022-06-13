### lc142

```python3
# Definition for singly-linked list.
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None

class Solution:
    def detectCycle(self, head: ListNode) -> ListNode:
        if head is None:
            return None
        
        slow = head
        fast = head
        while fast is not None:
            slow = slow.next
            if fast.next is not None:
                fast = fast.next.next
            else:
                return None
            
            if fast == slow:
                ptr = head
                while ptr != slow:
                    ptr = ptr.next
                    slow = slow.next
                return ptr
        
        return None
        
        
def main():
    solution = Solution()
    
    node1 = ListNode(3)
    headA = node1
    node1.next = ListNode(2)
    node1 = node1.next
    node1.next = ListNode(0)
    node1 = node1.next
    node1.next = ListNode(4)
    node1 = node1.next
    node1.next = headA.next
    
    node_out = solution.detectCycle(headA)
    print(node_out.val)
    
    index = 0
    while headA != node_out:
        headA = headA.next
        index += 1
    
    assert(index == 1)
    
if __name__ == '__main__':
    main()

```

### lc160

```python3
# Definition for singly-linked list.
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None
    

class Solution:
    def getIntersectionNode(self, headA: ListNode, headB: ListNode) -> ListNode:
        if not headA or not headB:
            return None
        nodeA = headA
        nodeB = headB
        while nodeA != nodeB:
            nodeA = nodeA.next if nodeA else headB
            nodeB = nodeB.next if nodeB else headA
        return nodeA
        
def main():
    solution = Solution()
    
    node1 = ListNode(4)
    headA = node1
    node1.next = ListNode(1)
    node1 = node1.next
    node1.next = ListNode(8)
    node1 = node1.next
    node1.next = ListNode(4)
    node1 = node1.next
    node1.next = ListNode(5)
    
    node2 = ListNode(5)
    headB = node2
    node2.next = ListNode(6)
    node2 = node2.next
    node2.next = ListNode(1)
    node2 = node2.next
    intersectNode = headA.next.next
    node2.next = intersectNode
    
    node_out = solution.getIntersectionNode(headA, headB)
    assert(node_out.val == 8)
    print(node_out.val)
        
if __name__ == '__main__':
    main()
        
```
