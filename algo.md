### lc707

```python3
class ListNode:
    
    def __init__(self, data: int, next=None):
        self.val = data
        self.next = next
        
        

class MyLinkedList:
    def __init__(self):
        self.size = 0
        self.head = ListNode(0)


    def get(self, index: int) -> int:
        if index < 0 or index >= self.size:
            return -1

        p = self.head
        position = 0
        while p and position != index + 1:
            p = p.next
            position += 1
        return p.val


    def addAtHead(self, val: int) -> None:
        self.addAtIndex(0, val)


    def addAtTail(self, val: int) -> None:
        self.addAtIndex(self.size, val)


    def addAtIndex(self, index: int, val: int) -> None:
        if index > self.size:
            return
        
        if index < 0:
            index = 0
            
        self.size += 1
        
        new_node = ListNode(val)
        
        p = self.head
        position = 0
        while p and position != index:
            p = p.next
            position += 1
        new_node.next = p.next
        p.next = new_node
    

    def deleteAtIndex(self, index: int) -> None:
        if index < 0 or index >= self.size:
            return
        
        self.size -= 1
        
        p = self.head
        position = 0
        while p and position != index:
            p = p.next
            position += 1
        tmp = p.next.next
        p.next = tmp




# Your MyLinkedList object will be instantiated and called as such:
# obj = MyLinkedList()
# param_1 = obj.get(index)
# obj.addAtHead(val)
# obj.addAtTail(val)
# obj.addAtIndex(index,val)
# obj.deleteAtIndex(index)

def test_get():
    l = MyLinkedList()
    for i in range(10):
        l.addAtHead(i)
    
    # test get
    i9 = l.get(0)
    assert i9 == 9
    i5 = l.get(4)
    assert i5 == 5
    i0 = l.get(9)
    assert i0 == 0

def test_addAtTail():
    l = MyLinkedList()
    for i in range(10):
        l.addAtTail(i)

    i0 = l.get(0)
    assert i0 == 0
    
    i5 = l.get(5)
    assert i5 == 5
    
    i9 = l.get(9)
    assert i9 == 9

def test_deleteAtIndex():
    l = MyLinkedList()
    for i in range(3):
        l.addAtTail(i)
    
    i0 = l.get(0)
    assert(i0 == 0)
    
    i1 = l.get(1)
    assert(i1 == 1)
    
    i2 = l.get(2)
    assert(i2 == 2)
    
    l.deleteAtIndex(1)
    assert(l.get(1) == 2)

if __name__ == "__main__":
    test_get()
    test_addAtTail()
    test_deleteAtIndex()

```

## lc707 double

```python3
class ListNode:
    def __init__(self, data: int):
        self.val = data
        self.prev = None
        self.next = None
        

class MyLinkedList:
    def __init__(self):
        self.size = 0
        self.head, self.tail = ListNode(0), ListNode(0)
        self.head.next = self.tail
        self.tail.prev = self.head
        
    def get(self, index: int) -> int:
        if index < 0 or index >= self.size:
            return -1
        
        # choose the fastest way: to move from the head or to move from the tail
        if index + 1 < self.size - index:
            curr = self.head
            for _ in range(index + 1):
                curr = curr.next
        else:
            curr = self.tail
            for _ in range(self.size - index):
                curr = curr.prev
            
        return curr.val
        
    def addAtHead(self, val: int) -> None:
        pred, succ = self.head, self.head.next
        
        self.size += 1
        to_add = ListNode(val)
        to_add.prev = pred
        to_add.next = succ
        pred.next = to_add
        succ.prev = to_add
        
    def addAtTail(self, val: int) -> None:
        succ, pred = self.tail, self.tail.prev
        
        self.size += 1
        to_add = ListNode(val)
        to_add.prev = pred
        to_add.next = succ
        pred.next = to_add
        succ.prev = to_add
        
        
    def addAtIndex(self, index: int, val: int) -> None:
        if index > self.size:
            return
        if index < 0:
            index = 0
        
        # find processor and successor of the node to be added
        if index < self.size - index:
            pred = self.head
            for _ in range(index):
                pred = pred.next
            succ = pred.next
        else:
            succ = self.tail
            for _ in range(self.size - index):
                succ = succ.prev
            succ = succ.prev
            
        # insertion itself
        self.size += 1
        to_add = ListNode(val)
        to_add.prev = pred
        to_add.next = succ
        pred.next = to_add
        succ.prev = to_add
        
    def deleteAtIndex(self, index: int) -> None:
        if index < 0 or index >= self.size:
            return
        
        if index < self.size - index:
            pred = self.head
            for _ in range(index):
                pred = pred.next
            succ = pred.next.next
        else:
            succ = self.tail
            for _ in range(self.size - index):
                succ = succ.prev
            pred = succ.prev.prev
            
        # delete pred.next
        self.size -= 1
        pred.next = succ
        succ.prev = pred
```

## lc430 v0

```python3
class Node:
    def __init__(self, val, prev, next, child):
        self.val = val
        self.prev = prev
        self.next = next
        self.child = child


class Solution:
    def flatten(self, head: 'Node') -> 'Node':
        save_root = []
        root = head
        while head and (head.next or head.child):
            if head.child is None:
                head = head.next
            elif head.next:
                head.next.prev = None
                save_root.append(head.next)
                head.child.prev = head
                head.next = head.child
                head.child = None
                head = head.next
            else:
                while head.child:
                    if head.next:
                        save_root.append(head.next)
                        head.next.prev = None
                    head.child.prev = head
                    head.next = head.child
                    head.child = None
                    head = head.next
                
        while len(save_root) > 0:
            tmp = save_root.pop()
            tmp.prev = head
            head.next = tmp
            head.child = None
            while head and head.next:
                head = head.next
        
                    
        return root
def test1():
    solution = Solution()
    node1 = Node(1, None, None, None)
    node2 = Node(2, None, None, None)
    node3 = Node(3, None, None, None)
    
    node1.next = node2
    node2.prev = node1
    node1.child = node3
    
    root = solution.flatten(node1)
    assert(root.val == 1)
    root = root.next
    assert(root.val == 3)
    root = root.next
    assert(root.val == 2)
    
def test2():
    solution = Solution()
    node1 = Node(1, None, None, None)
    node2 = Node(2, None, None, None)
    node3 = Node(3, None, None, None)
    node4 = Node(4, None, None, None)
    node5 = Node(5, None, None, None)
    node6 = Node(6, None, None, None)
    node1.next = node2
    node2.prev = node1
    node2.next = node3
    node3.prev = node2
    node3.next = node4
    node4.prev = node3
    node4.next = node5
    node5.prev = node4
    node5.next = node6
    node6.prev = node5
    
    node7 = Node(7, None, None, None)
    node8 = Node(8, None, None, None)
    node9 = Node(9, None, None, None)
    node10 = Node(10, None, None, None)
    node7.next = node8
    node8.prev = node7
    node8.next = node9
    node9.prev = node8
    node9.next = node10
    node10.prev = node9
    
    node3.child = node7
    
    node11 = Node(11, None, None, None)
    node12 = Node(12, None, None, None)
    node11.next = node12
    node12.prev = node11
    
    node8.child = node11
    
    root = solution.flatten(node1)
    for _ in range(12):
        print(root.val)
        root = root.next
    
def test3():
    solution = Solution()
    node1 = Node(1, None, None, None)
    node2 = Node(2, None, None, None)
    node3 = Node(3, None, None, None)
    node1.child = node2
    node2.child = node3
    
    root = solution.flatten(node1)
    assert(root.val == 1)
    root = root.next
    assert(root.val == 2)
    root = root.next
    assert(root.val == 3)

def test4():
    solution = Solution()
    node1 = Node(1, None, None, None)
    node2 = Node(2, None, None, None)
    node3 = Node(3, None, None, None)
    node4 = Node(4, None, None, None)
    node5 = Node(5, None, None, None)
    node6 = Node(6, None, None, None)
    node1.next = node2
    node2.prev = node1
    node2.next = node3
    node3.prev = node2
    node3.next = node4
    node4.prev = node3
    node4.next = node5
    node5.prev = node4
    node5.next = node6
    node6.prev = node5
    
    node7 = Node(7, None, None, None)
    node8 = Node(8, None, None, None)
    node7.next = node8
    node8.prev = node7
    
    node3.child = node7
    
    node11 = Node(11, None, None, None)
    node12 = Node(12, None, None, None)
    node11.next = node12
    node12.prev = node11
    
    node8.child = node11
    
    root = solution.flatten(node1)
    for _ in range(9):
        print(root.val)
        root = root.next

def test5():
    solution = Solution()
    node1 = Node(1, None, None, None)
    node2 = Node(2, None, None, None)
    node4 = Node(4, None, None, None)
    node1.next = node2
    node2.prev = node1
    node2.next = node4
    node4.prev = node2
    
    node3 = Node(3, None, None, None)
    node2.child = node3
    
    node5 = Node(5, None, None, None)
    node3.child = node5
    
    root = solution.flatten(node1)
    assert(root.val == 1)
    root = root.next
    assert(root.val == 2)
    root = root.next
    assert(root.val == 3)
    root = root.next
    assert(root.val == 5)
    root = root.next
    assert(root.val == 4)
    
def test6():
    solution = Solution()
    node1 = Node(1, None, None, None)
    node2 = Node(2, None, None, None)
    node4 = Node(4, None, None, None)
    
    node2.next = node4
    node4.prev = node2
    
    node1.child = node2
    
    node3 = Node(3, None, None, None)
    node2.child = node3
    
    node5 = Node(5, None, None, None)
    node3.child = node5
    
    root = solution.flatten(node1)
    assert(root.val == 1)
    root = root.next
    assert(root.val == 2)
    root = root.next
    assert(root.val == 3)
    root = root.next
    assert(root.val == 5)
    root = root.next
    assert(root.val == 4)

if __name__ == '__main__':
    # test1()
    # test2()
    # test3()
    # test4()
    # test5()
    test6()
```
