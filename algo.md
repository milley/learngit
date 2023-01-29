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

## lc1442

```python3
from collections import Counter
from typing import List
class Solution:
    def countTriplets(self, arr: List[int]) -> int:
        return self.countTriplets4(arr)
    
    def countTriplets1(self, arr: List[int]) -> int:
        n = len(arr)
        s = [0]
        for val in arr:
            s.append(s[-1] ^ val)
        
        ans = 0
        for i in range(n):
            for j in range(i + 1, n):
                for k in range(j, n):
                    if s[i] == s[k + 1]:
                        #print(i, j, k)
                        ans += 1
        return ans
    
    def countTriplets2(self, arr: List[int]) -> int:
        n = len(arr)
        s = [0]
        for val in arr:
            s.append(s[-1] ^ val)
        ans = 0
        for i in range(n):
            for k in range(i + 1, n):
                if s[i] == s[k + 1]:
                    ans += k - i
                    
        return ans
    
    def countTriplets3(self, arr: List[int]) -> int:
        n = len(arr)
        s = [0]
        for val in arr:
            s.append(s[-1] ^ val)
        
        cnt, total = Counter(), Counter()
        ans = 0
        for k in range(n):
            if s[k + 1] in cnt:
                ans += cnt[s[k + 1]] * k - total[s[k + 1]]
            cnt[s[k]] += 1
            total[s[k]] += k
            
        return ans
    
    # Assignment expressions start at python3.8
    # def countTriplets4(self, arr: List[int]) -> int:
    #     print("------------------------")
    #     cnt, total = Counter(), Counter()
    #     ans = s = 0
        
    #     for k, val in enumerate(arr):
    #         if (t := s ^ val) in cnt:
    #             ans += cnt[t] * k - total[t]
    #         cnt[s] += 1
    #         total[s] += k
    #         s = t
            
    #     return ans
    
def test1():
    solution = Solution()
    
    arr = [2, 3, 1, 6, 7]
    assert(solution.countTriplets(arr) == 4)
    
def test2():
    solution = Solution()
    
    arr = [1, 1, 1, 1, 1]
    assert(solution.countTriplets(arr) == 10)
    
def test3():
    solution = Solution()
    
    arr = [2, 3]
    assert(solution.countTriplets(arr) == 0)
    
def test4():
    solution = Solution()
    
    arr = [1, 3, 5, 7, 9]
    assert(solution.countTriplets(arr) == 3)
    
def test5():
    solution = Solution()
    
    arr = [7, 11, 12, 9, 5, 2, 7, 17, 22]
    assert(solution.countTriplets(arr) == 8)
    
if __name__ == '__main__':
    test1()
    test2()
    test3()
    test4()
    test5()
```
