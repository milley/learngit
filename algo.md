## 494

```python3
from typing import List

class Solution:
    def findTargetSumWays(self, nums: List[int], target: int) -> int:
        n = len(nums)
        total = sum(nums)
        if total < target: return 0
        
        dp = {(0, 0): 1}
        for i in range(1, n + 1):
            for j in range(-total, total + 1):
                dp[(i, j)] = dp.get((i-1, j - nums[i - 1]), 0) + dp.get((i - 1, j + nums[i - 1]), 0)
        return dp.get((n, target), 0)
    
    def findTargetSumWays1(self, nums: List[int], target: int) -> int:
        d = {}
        def dfs(cur, i, d):
            if i < len(nums) and (cur, i) not in d:
                d[(cur, i)] = dfs(cur + nums[i], i + 1, d) + dfs(cur - nums[i], i + 1, d)
            return d.get((cur, i), int(cur == target))
        return dfs(0, 0, d)
    
def test1():
    solution = Solution()
    nums = [1,1,1,1,1]
    target = 3
    assert(solution.findTargetSumWays1(nums, target) == 5)
    
if __name__ == '__main__':
    test1()
```

## lc733

```python3
from typing import List
import collections

class Solution:
    # BFS
    def floodFill(self, image: List[List[int]], sr: int, sc: int, color: int) -> List[List[int]]:
        currColor = image[sr][sc]
        if currColor == color:
            return image
        n, m = len(image), len(image[0])
        que = collections.deque([(sr, sc)])
        image[sr][sc] = color
        
        while que:
            x, y = que.popleft()
            for mx, my in [(x - 1, y), (x + 1, y), (x, y - 1), (x, y + 1)]:
                if 0 <= mx < n and 0 <= my < m and image[mx][my] == currColor:
                    que.append((mx, my))
                    image[mx][my] = color
        
        return image
    
    # DFS
    def floodFill1(self, image: List[List[int]], sr: int, sc: int, color: int) -> List[List[int]]:
        n, m = len(image), len(image[0])
        currColor = image[sr][sc]
        
        def dfs(x: int, y: int):
            if image[x][y] == currColor:
                image[x][y] = color
                for mx, my in [(x -1, y), (x + 1, y), (x, y - 1), (x, y + 1)]:
                    if 0 <= mx < n and 0 <= my < m and image[mx][my] == currColor:
                        dfs(mx, my)
            
        if currColor != color:
            dfs(sr, sc)
 
        return image
    
def test1():
    solution = Solution()
    image = [[1,1,1],[1,1,0],[1,0,1]]
    sr, sc = 1, 1
    newColor = 2
    solution.floodFill1(image, sr, sc, newColor)
    print(image)
    assert(image == [[2, 2, 2], [2, 2, 0], [2, 0, 1]])
    
def test2():
    solution = Solution()
    image = [[0,0,0],[0,0,0]]
    sr, sc = 0, 0
    newColor = 2
    solution.floodFill1(image, sr, sc, newColor)
    print(image)
    assert(image == [[2, 2, 2], [2, 2, 2]])
    
if __name__ == '__main__':
    test1()
    test2()
    
```
