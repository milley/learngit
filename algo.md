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

## lc542

```python3
from typing import List
import collections

class Solution:
    # BFS
    def updateMatrix(self, mat: List[List[int]]) -> List[List[int]]:
        m, n = len(mat), len(mat[0])
        dist = [[0] * n for _ in range(m)]
        zeroes_pos = [(i, j) for i in range(m) for j in range(n) if mat[i][j] == 0]
        q = collections.deque(zeroes_pos)
        seen = set(zeroes_pos)
        
        while q:
            i, j = q.popleft()
            for ni, nj in [(i - 1, j), (i + 1, j), (i, j - 1), (i, j + 1)]:
                if 0 <= ni < m and 0 <= nj < n and (ni, nj) not in seen:
                    dist[ni][nj] = dist[i][j] + 1
                    q.append((ni, nj))
                    seen.add((ni, nj))
                    
        return dist
    
    # DP
    def updateMatrix1(self, mat: List[List[int]]) -> List[List[int]]:
        m, n = len(mat), len(mat[0])
        dist = [[10**9] * n for _ in range(m)]
        
        for i in range(m):
            for j in range(n):
                if mat[i][j] == 0:
                    dist[i][j] = 0
        
        for i in range(m):
            for j in range(n):
                if i - 1 >= 0:
                    dist[i][j] = min(dist[i][j], dist[i - 1][j] + 1)
                if j - 1 >= 0:
                    dist[i][j] = min(dist[i][j], dist[i][j - 1] + 1)
                    
        for i in range(m - 1, -1, -1):
            for j in range(n - 1, -1, -1):
                if i + 1 < m:
                    dist[i][j] = min(dist[i][j], dist[i + 1][j] + 1)
                if j + 1 < n:
                    dist[i][j] = min(dist[i][j], dist[i][j + 1] + 1)
        return dist
        
        
def test1():
    solution = Solution()
    mat = [[0,0,0],[0,1,0],[0,0,0]]
    out = [[0,0,0],[0,1,0],[0,0,0]]
    assert(solution.updateMatrix1(mat) == out)
    
def test2():
    solution = Solution()
    mat = [[0,0,0],[0,1,0],[1,1,1]]
    out = [[0,0,0],[0,1,0],[1,2,1]]
    assert(solution.updateMatrix1(mat) == out)
    
def test3():
    solution = Solution()
    mat = [[0], [1]]
    out = [[0], [1]]
    assert(solution.updateMatrix1(mat) == out)
    
if __name__ == '__main__':
    test1()
    test2()
    test3()
```

```cpp
#include <vector>
#include <queue>
#include <cassert>
#include <cstdint>
using namespace std;

class Solution {
private:
    static constexpr int dirs[4][2] = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};

public:
    // BFS
    vector<vector<int>> updateMatrix(vector<vector<int>>& matrix) {
        int m = matrix.size(), n = matrix[0].size();
        vector<vector<int>> dist(m, vector<int>(n));
        vector<vector<int>> seen(m, vector<int>(n));
        queue<pair<int, int>> q;

        // insert all of zero into q
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (matrix[i][j] == 0) {
                    q.emplace(i, j);
                    seen[i][j] = 1;
                }
            }
        }

        while (!q.empty()) {
            auto [i, j] = q.front();
            q.pop();
            for (int d = 0; d < 4; ++d) {
                int ni = i + dirs[d][0];
                int nj = j + dirs[d][1];
                if (ni >= 0 && ni < m && nj >= 0 && nj < n && !seen[ni][nj]) {
                    dist[ni][nj] = dist[i][j] + 1;
                    q.emplace(ni, nj);
                    seen[ni][nj] = 1;
                }
            }
        }

        return dist;
    }

    // DP
    vector<vector<int>> updateMatrix1(vector<vector<int>>& matrix) {
        int m = matrix.size(), n = matrix[0].size();
        vector<vector<int>> dist(m, vector<int>(n, INT_MAX / 2));
        // if (i, j) is 0, distance is 0
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (matrix[i][j] == 0) {
                    dist[i][j] = 0;
                }
            }
        }

        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (i - 1 >= 0) {
                    dist[i][j] = min(dist[i][j], dist[i - 1][j] + 1);
                }
                if (j - 1 >= 0) {
                    dist[i][j] = min(dist[i][j], dist[i][j - 1] + 1);
                }
            }
        }

        for (int i = m - 1; i >= 0; --i) {
            for (int j = n - 1; j >= 0; --j) {
                if (i + 1 < m) {
                    dist[i][j] = min(dist[i][j], dist[i + 1][j] + 1);
                }
                if (j + 1 < n) {
                    dist[i][j] = min(dist[i][j], dist[i][j + 1] + 1);
                }
            }
        }

        return dist;
    }
};

void test1() {
    Solution solution;
    vector<vector<int>> mat = {{0,0,0},{0,1,0},{0,0,0}};
    vector<vector<int>> out = {{0,0,0},{0,1,0},{0,0,0}};
    assert(out == solution.updateMatrix1(mat));
}

void test2() {
    Solution solution;
    vector<vector<int>> mat = {{0,0,0},{0,1,0},{1,1,1}};
    vector<vector<int>> out = {{0,0,0},{0,1,0},{1,2,1}};
    assert(out == solution.updateMatrix1(mat));
}

void test3() {
    Solution solution;
    vector<vector<int>> mat = {{0},{1}};
    vector<vector<int>> out = {{0},{1}};
    assert(out == solution.updateMatrix1(mat));
}

int main() {
    test1();
    test2();
    test3();
}

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
