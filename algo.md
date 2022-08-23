## lc144

```python3
from typing import Optional, List

# Definition for a binary tree node.
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
class Solution:
    def __init__(self):
        self.array = []
        
    def preorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        if not root:
            return []
        self.array.append(root.val)
        self.preorderTraversal(root.left)
        self.preorderTraversal(root.right)
        
        return self.array
    
def test1():
    solution = Solution()
    node1 = TreeNode(1, None, None)
    node2 = TreeNode(2, None, None)
    node3 = TreeNode(3, None, None)
    node1.right = node2
    node2.left = node3
    
    #print(solution.preorderTraversal(node1))
    assert(solution.preorderTraversal(node1) == [1, 2, 3])
    
if __name__ == '__main__':
    test1()
```

## lc145

```python3
from typing import Optional, List

# Definition for a binary tree node.
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
class Solution:
    def __init__(self):
        self.array = []
        
    def postorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        if not root:
            return []
        
        self.postorderTraversal(root.left)
        self.postorderTraversal(root.right)
        self.array.append(root.val)
        
        return self.array
    
def test1():
    solution = Solution()
    node1 = TreeNode(1, None, None)
    node2 = TreeNode(2, None, None)
    node3 = TreeNode(3, None, None)
    node1.right = node2
    node2.left = node3
    
    assert(solution.postorderTraversal(node1) == [3, 2, 1])
    
if __name__ == '__main__':
    test1()
```

## lc324

```python3
class Solution:
    def isPowerOfFour(self, n: int) -> bool:
        while n and n % 4 == 0:
            n //= 4
        return n == 1
    
def test1():
    solution = Solution()
    assert(solution.isPowerOfFour(16))
    
            
if __name__ == '__main__':
    test1()
```

## lc326

```python3
class Solution:
    def isPowerOfThree(self, n: int) -> bool:
        while n and n % 3 == 0:
            n //= 3
        
        return n == 1
           
def test1():
    solution = Solution()
    assert(solution.isPowerOfThree(27))
    
            
if __name__ == '__main__':
    test1()
```

## lc344

```python3
from typing import List

class Solution:
    def reverseString(self, s: List[str]) -> None:
        n = len(s)
        i = 0
        while i < n // 2:
            s[i], s[n - i - 1] = s[n - i - 1], s[i]
            i += 1
        
        #print(s)
def test1():
    solution = Solution()
    s = ["h","e","l","l","o"]
    #s = ["H","a","n","n","a","h"]
    solution.reverseString(s)
            
if __name__ == '__main__':
    test1()
```

## lc509

```python3
class Solution:
    def fib(self, n: int) -> int:
        if n == 0: return 0
        if n == 1: return 1
        return self.fib(n - 1) + self.fib(n - 2)
    
    # x * x = x + 1
    def fib1(self, n: int) -> int:
        sqrt5 = 5 ** 0.5
        fibN = ((1 + sqrt5) / 2) ** n - ((1 - sqrt5) / 2) ** n
        return round(fibN / sqrt5)
    
if __name__ == '__main__':
    solution = Solution()
    assert(solution.fib1(2) == 1)
    assert(solution.fib1(3) == 2)
    assert(solution.fib1(4) == 3)
```
