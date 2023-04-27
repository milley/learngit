## lc746

```python3
from typing import List


class Solution:
    def minCostClimbingStairs(self, cost: List[int]) -> int:
        n = len(cost)
        dp = [0] * (n + 1)
        for i in range(2, n + 1):
            dp[i] = min(dp[i - 1] + cost[i - 1], dp[i - 2] + cost[i - 2])
        
        
        return dp[n]
    
def test1():
    solution = Solution()
    cost = [10, 15, 20]
    assert(solution.minCostClimbingStairs(cost) == 15)
    
def test2():
    solution = Solution()
    cost = [1,100,1,1,1,100,1,1,100,1]
    assert(solution.minCostClimbingStairs(cost) == 6)
    
if __name__ == '__main__':
    test1()
    test2()
```
