# lc200

```python3
import collections
from typing import List

class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        x, y = len(grid), len(grid[0])
        cnt = 0
        if x == 0:
            return 0
        for r in range(x):
            for c in range(y):
                if grid[r][c] == "1":
                    cnt += 1
                    grid[r][c] = "0"
                    neighbors = collections.deque([(r, c)])
                    #print(list(neighbors))
                    while neighbors:
                        row, col = neighbors.popleft()
                        print("{} {}".format(row, col))
                        for xx, yy in [(row - 1, col), (row + 1, col), (row, col - 1), (row, col + 1)]:
                            if 0 <= xx < x and 0 <= yy < y and grid[xx][yy] == "1":
                                neighbors.append((xx, yy))
                                grid[xx][yy] = "0"
                    #print(grid)
        return cnt
    
    def dfs(self, grid, r, c):
        grid[r][c] = 0
        nr, nc = len(grid), len(grid[0])
        for x, y in [(r - 1, c), (r + 1, c), (r, c - 1), (r, c + 1)]:
            if 0 <= x < nr and 0 <= y < nc and grid[x][y] == "1":
                self.dfs(grid, x, y)
    
    def numIslands1(self, grid: List[List[str]]) -> int:
        nr = len(grid)
        if nr == 0:
            return 0
        nc = len(grid[0])
        
        num_islands = 0
        for r in range(nr):
            for c in range(nc):
                if grid[r][c] == "1":
                    num_islands += 1
                    self.dfs(grid, r, c)
        return num_islands
    
def test1():
    solution = Solution()
    l = [
        ["1","1","1","1","0"],
        ["1","1","0","1","0"],
        ["1","1","0","0","0"],
        ["0","0","0","0","0"]
    ]
    print(solution.numIslands(l))
    
if __name__ == "__main__":
    test1()
```

## lc279

```python3
import sys
import math

class Solution:
    def numSquares(self, n: int) -> int:
        f = [0] * (n + 1)
        for i in range(1, n + 1):
            minn = sys.maxsize
            j = 1
            while j * j <= i:
                minn = min(minn, f[i - j * j])
                j += 1
            f[i] = minn + 1
        
        return f[n]
    
    def isPerfectSquare(self, x: int) -> bool:
        y = math.floor(math.sqrt(x))
        return x == y * y

    def checkAnswer4(self, x: int) -> bool:
        while x % 4 == 0:
            x //= 4
        return x % 8 == 7

    def numSquares1(self, n: int) -> int:
        if self.isPerfectSquare(n):
            return 1
        if self.checkAnswer4(n):
            return 4
        i = 1
        while i * i <= n:
            j = n - i * i
            if self.isPerfectSquare(j):
                return 2
            i += 1
        return 3
    
def test1():
    solution = Solution()
    assert(solution.numSquares(12) == 3)
    assert(solution.numSquares(13) == 2)
    
def test2():
    solution = Solution()
    assert(solution.numSquares1(12) == 3)
    assert(solution.numSquares1(13) == 2)
    
if __name__ == '__main__':
    test1()
    test2()
```

## lc752

```python3
from collections import deque
from typing import Generator, List

class Solution:
    def openLock(self, deadends: List[str], target: str) -> int:
        if target == "0000":
            return 0
        
        dead = set(deadends)
        if "0000" in dead:
            return -1
        
        def num_prev(x: str) -> str:
            return "9" if x == "0" else str(int(x) - 1)
        def num_succ(x: str) -> str:
            return "0" if x == "9" else str(int(x) + 1)
        
        def get(status: str) -> Generator[str, None, None]:
            s = list(status)
            for i in range(4):
                num = s[i]
                s[i] = num_prev(num)
                yield "".join(s)
                s[i] = num_succ(num)
                yield "".join(s)
                s[i] = num
        
        q = deque([("0000", 0)])
        seen = {"0000"}
        while q:
            status, step = q.popleft()
            for next_status in get(status):
                if next_status not in seen and next_status not in dead:
                    if next_status == target:
                        return step + 1
                    q.append((next_status, step + 1))
                    seen.add(next_status)
        
        return -1
        
def test1():
    solution = Solution()
    deadends = ["0201","0101","0102","1212","2002"]
    target = "0202"
    assert(solution.openLock(deadends, target) == 6)
    
def test2():
    solution = Solution()
    deadends = ["8888"]
    target = "0009"
    assert(solution.openLock(deadends, target) == 1)
        
def test3():
    solution = Solution()
    deadends = ["8887","8889","8878","8898","8788","8988","7888","9888"]
    target = "8888"
    assert(solution.openLock(deadends, target) == -1)

        
if __name__ == "__main__":
    test1()
    test2()
    test3()
```
