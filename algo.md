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
