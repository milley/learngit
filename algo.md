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
