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

## offer113

```python3
import collections
from typing import List


class Solution:
    def findOrder(self, numCourses: int, prerequisites: List[List[int]]) -> List[int]:
        return self.findOrderBfs(numCourses, prerequisites)
        
    def findOrderDfs(self, numCourses: int, prerequisites: List[List[int]]) -> List[int]:
        # 存储有向图
        edges = collections.defaultdict(list)
        # 标记每个节点的状态：0=未搜索 1=搜索中 2=已完成
        visited = [0] * numCourses
        # 用数组来模拟栈，下标0为栈底，n-1为栈顶
        result = list()
        # 判断有向图中是否有环
        valid = True
        
        for info in prerequisites:
            edges[info[1]].append(info[0])
            
        def dfs(u: int):
            nonlocal valid
            # 将节点标记为「搜索中」
            visited[u] = 1
            # 搜索其相邻节点
            # 只要发现有环，立刻停止搜索
            for v in edges[u]:
                # 如果「未搜索」那么搜索相邻节点
                if visited[v] == 0:
                    dfs(v)
                    if not valid:
                        return
                # 如果「搜索中」说明找到了环
                elif visited[v] == 1:
                    valid = False
                    return
            # 将节点标记为「已完成」
            visited[u] = 2
            # 将节点入栈
            result.append(u)
            
        # 每次挑选一个「未搜索」的节点，开始进行深度优先搜索
        for i in range(numCourses):
            if valid and not visited[i]:
                dfs(i)
        
        if not valid:
            return list()
        
        # 如果没有环，那么就有拓扑排序
        # 注意下标 0 为栈底，因此需要将数组反序输出
        return result[::-1]
    
    
    def findOrderBfs(self, numCourses: int, prerequisites: List[List[int]]) -> List[int]:
        # 存储有向图
        edges = collections.defaultdict(list)
        # 存储每个节点的入度
        indeg = [0] * numCourses
        # 存储答案
        result = list()
        
        for info in prerequisites:
            edges[info[1]].append(info[0])
            indeg[info[0]] += 1
            
        # 将所有入度为 0 的节点放入队列中
        q = collections.deque([u for u in range(numCourses) if indeg[u] == 0])
        
        while q:
            # 从队首取出一个节点
            u = q.popleft()
            # 放入答案
            result.append(u)
            for v in edges[u]:
                indeg[v] -= 1
                # 如果相邻节点 v 的入度为 0，就可以选 v 对应的课程了
                if indeg[v] == 0:
                    q.append(v)
                    
        if len(result) != numCourses:
            result = list()
        return result
    
def test1():
    solution = Solution()
    num = 2
    l = [[1, 0]]
    
    ll = solution.findOrder(num, l)
    print(ll)
    assert(ll == [0, 1])
    
def test2():
    solution = Solution()
    num = 4
    l = [[1, 0], [2, 0], [3, 1], [3, 2]]
    
    ll = solution.findOrder(num, l)
    print(ll)
    assert(ll in [[0, 1, 2, 3], [0, 2, 1, 3]])
    
def test3():
    solution = Solution()
    num = 1
    l = []
    
    ll = solution.findOrder(num, l)
    print(ll)
    assert(ll == [0])
    
if __name__ == '__main__':
    test1()
    test2()
    test3()
```

## lc1129

```python3
from typing import List


class Solution:
    def shortestAlternatingPaths(self, n: int, redEdges: List[List[int]], blueEdges: List[List[int]]) -> List[int]:
        g = [[] for _ in range(n)]
        for x, y in redEdges:
            g[x].append((y, 0))
        for x, y in blueEdges:
            g[x].append((y, 1))
        
        dis = [-1] * n
        vis = {(0, 0), (0, 1)}
        q = [(0, 0), (0, 1)]
        level = 0
        while q:
            tmp = q
            q = []
            for x, color in tmp:
                if dis[x] == -1:
                    dis[x] = level
                for p in g[x]:
                    if p[1] != color and p not in vis:
                        vis.add(p)
                        q.append(p)
            
            level += 1
        return dis
    
def test1():
    solution = Solution()
    n = 3
    red_edges = [[0,1],[1,2]]
    blue_edges = []
    #print(solution.shortestAlternatingPaths(n, red_edges, blue_edges))
    assert(solution.shortestAlternatingPaths(n, red_edges, blue_edges) == [0, 1, -1])
    
if __name__ == '__main__':
    test1()
```

## lc103

```python3
from typing import List, Optional

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
        
class Solution:
    def zigzagLevelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        ans = []
        if not root:
            return ans
        
        queue = [(root, 1)]
        while queue:
            node, level = queue.pop(0)
            if level > len(ans):
                ans.append([node.val])
            else:
                ans[-1].append(node.val)
            
            if node.left:
                queue.append((node.left, level + 1))
            if node.right:
                queue.append((node.right, level + 1))
                
        for i in range(len(ans)):
            if i % 2 == 1:
                ans[i] = ans[i][::-1]
        
        
        return ans

def test1():
    solution = Solution()
    root = TreeNode(3, None, None)
    node9 = TreeNode(9, None, None)
    node20 = TreeNode(20, None, None)
    root.left = node9
    root.right = node20
    
    node15 = TreeNode(15, None, None)
    node7 = TreeNode(7, None, None)
    node20.left = node15
    node20.right = node7
    
    print(solution.zigzagLevelOrder(root))

if __name__ == "__main__":
    test1()
```

## lc786

```python3
from functools import cmp_to_key
import heapq
from typing import List, Tuple

class Frac:
    def __init__(self, idx: int, idy: int, x: int, y: int) -> None:
        self.idx = idx
        self.idy = idy
        self.x = x
        self.y = y
        
    def __lt__(self, other: "Frac") -> bool:
        return self.x * other.y < self.y * other.x
    
class Solution:
    def kthSmallestPrimeFraction(self, arr: List[int], k: int) -> List[int]:
        n = len(arr)
        q = [Frac(0, i, arr[0], arr[i]) for i in range(1, n)]
        heapq.heapify(q)
        
        for _ in range(k - 1):
            frac = heapq.heappop(q)
            i, j = frac.idx, frac.idy
            if i + 1 < j:
                heapq.heappush(q, Frac(i + 1, j, arr[i + 1], arr[j]))
                
            
        
        return [q[0].x, q[0].y]
    
    def kthSmallestPrimeFraction1(self, arr: List[int], k: int) -> List[int]:
        def cmp(x: Tuple[int, int], y: Tuple[int, int]) -> int:
            return -1 if x[0] * y[1] < x[1] * y[0] else 1
        
        n = len(arr)
        frac = list()
        for i in range(n):
            for j in range(i + 1, n):
                frac.append((arr[i], arr[j]))
                
        frac.sort(key=cmp_to_key(cmp))
        return list(frac[k - 1])
        
def test1():
    solution = Solution()
    arr = [1, 2, 3, 5]
    k = 3
    out = solution.kthSmallestPrimeFraction(arr, k)
    print(out)
    assert(out == [2, 5])
    
def test2():
    solution = Solution()
    arr = [1, 2, 3, 5]
    k = 3
    out = solution.kthSmallestPrimeFraction1(arr, k)
    print(out)
    assert(out == [2, 5])
    
if __name__ == '__main__':
    test1()
    test2()
```
