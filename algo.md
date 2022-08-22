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
