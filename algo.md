## lc1137

```python3
class Solution:
    def tribonacci(self, n: int) -> int:
        if n == 0:
            return 0
        elif n == 1 or n == 2:
            return 1
        else:
            return self.tribonacci(n - 3) + self.tribonacci(n - 2) + self.tribonacci(n - 1)
        
    def tribonacci1(self, n: int) -> int:
        if n == 0:
            return 0
        elif n == 1 or n == 2:
            return 1
        
        p = 0
        q = r = 1
        for i in range(3, n + 1):
            s = p + q + r
            p, q, r = q, r, s
        return s
```

## lc404

```python3
import collections
from typing import Optional

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
class Solution:
    def sumOfLeftLeaves(self, root: Optional[TreeNode]) -> int:
        isLeafNode = lambda node: not node.left and not node.right
        
        def dfs(node: TreeNode) -> int:
            ans = 0
            if node.left:
                ans += node.left.val if isLeafNode(node.left) else dfs(node.left)
            if node.right and not isLeafNode(node.right):
                ans += dfs(node.right)
            return ans
        
        return dfs(root) if root else 0
    
    def sumOfLeftLeaves1(self, root: Optional[TreeNode]) -> int:
        if not root:
            return 0
        isLeafNode = lambda node: not node.left and not node.right
        
        q = collections.deque([root])
        ans = 0
        while q:
            node = q.popleft()
            if node.left:
                if isLeafNode(node.left):
                    ans += node.left.val
                else:
                    q.append(node.left)
            if node.right:
                if not isLeafNode(node.right):
                    q.append(node.right)
        
        return ans
```

## lc501

```python3
from typing import List, Optional

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
class Solution:
    def __init__(self):
        self.base = 0
        self.count = 0
        self.maxCount = 0
        self.answer = []
        
    def findMode(self, root: Optional[TreeNode]) -> List[int]:
        self.dfs(root)
        return self.answer
        
        
    def dfs(self, node: Optional[TreeNode]):
        if node is None:
            return
        self.dfs(node.left)
        self.update(node.val)
        self.dfs(node.right)
    
    def update(self, val: int):
        if val == self.base:
            self.count += 1
        else:
            self.count = 1
            self.base = val
            
        if self.count == self.maxCount:
            self.answer.append(self.base)
        if self.count > self.maxCount:
            self.maxCount = self.count
            self.answer.clear()
            self.answer.append(self.base)
            
def test1():
    solution = Solution()
    node1 = TreeNode(1, None, None)
    node2 = TreeNode(2, None, None)
    node3 = TreeNode(2, None, None)
    node1.right = node2
    node2.left = node3
    
    print(solution.findMode(node1))
if __name__ == '__main__':
    test1()
```
