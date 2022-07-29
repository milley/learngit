lc61

```python3
from typing import Optional

# Definition for singly-linked list.
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next
class Solution:
    def rotateRight(self, head: Optional[ListNode], k: int) -> Optional[ListNode]:
        if k == 0 or not head or not head.next:
            return head
        
        cur = head
        n = 0
        while cur:
            n += 1
            cur = cur.next
        
        if n - k % n == n:
            return head
        
        n = n - k % n
        
        
        
        cur = head
        t = 0
        
        left_head = None
        while cur:
            t += 1
            if t == n:
                left_head = cur.next
                cur.next = None
                break
            
            cur = cur.next
        
        new_head = left_head
        while left_head and left_head.next:
            left_head = left_head.next
        left_head.next = head
        
        return new_head

def test1():
    solution = Solution()
    
    node1 = ListNode(1)
    node2 = ListNode(2)
    node3 = ListNode(3)
    node4 = ListNode(4)
    node5 = ListNode(5)
    
    node1.next = node2
    node2.next = node3
    node3.next = node4
    node4.next = node5
    
    out = solution.rotateRight(node1, 2)
    #print(out.val)
    assert(out.val == 4)
def test2():
    solution = Solution()
    
    node1 = ListNode(1)
    node2 = ListNode(2)
    node1.next = node2
    
    out = solution.rotateRight(node1, 2)
    print(out.val)
    
if __name__ == '__main__':
    #test1()
    test2()
```

lc138

```python3
from typing import Optional

"""
# Definition for a Node.
"""
class Node:
    def __init__(self, x: int, next: 'Node' = None, random: 'Node' = None):
        self.val = int(x)
        self.next = next
        self.random = random

class Solution:
    cached_map = dict()
    
    def copyRandomList(self, head: 'Optional[Node]') -> 'Optional[Node]':
        if head is None:
            return None
        if self.cached_map.get(head) is None:
            head_new = Node(head.val, None, None)
            self.cached_map[head] = head_new
            head_new.next = self.copyRandomList(head.next)
            head_new.random = self.copyRandomList(head.random)
            
        return self.cached_map[head]
    
    def copyRandomList1(self, head: 'Optional[Node]') -> 'Optional[Node]':
        if head is None:
            return None
        
        cached_map = dict()
        
        p = head
        while p:
            new_node = Node(p.val, None, None)
            cached_map[p] = new_node
            p = p.next
        
        p = head
        while p:
            if p.next:
                cached_map[p].next = cached_map[p.next]
            if p.random:
                cached_map[p].random = cached_map[p.random]
            p = p.next
            
        return cached_map[head]
def test1():
    solution = Solution()
    
    node7 = Node(7, None, None)
    node13 = Node(13, None, None)
    node11 = Node(11, None, None)
    node10 = Node(10, None, None)
    node1 = Node(1, None, None)
    
    node7.next = node13
    node7.random = None
    node13.next = node11
    node13.random = node7
    node11.next = node10
    node11.random = node1
    node10.next = node1
    node10.random = node11
    node1.next = None
    node1.random = node7
    
    new_node = solution.copyRandomList(node7)
    
if __name__ == '__main__':
    test1()
    
```

## lc268

```python3
from typing import List

class Solution:
    def missingNumber1(self, nums: List[int]) -> int:
        nums.sort()
        t = 0
        for i in nums:
            if i != t:
                return t
            t += 1
        return t
    
    def missingNumber(self, nums: List[int]) -> int:
        n = len(nums)
        total = n * (n + 1) // 2
        arrSum = sum(nums)
        return total - arrSum
    
if __name__ == '__main__':
    solution = Solution()
    nums = [3, 0, 1]
    print(solution.missingNumber(nums))
```

