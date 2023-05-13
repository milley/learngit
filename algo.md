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

## lc1048

```python3
from collections import defaultdict
from typing import List


class Solution:
    def longestStrChain(self, words: List[str]) -> int:
        ans = 0
        cnt = defaultdict(int)
        words.sort(key=len)
        #print(words)
        
        for word in words:
            cnt[word] = 1
            for i in range(len(word)):
                prev = word[:i] + word[i+1:]
                #print('--' + prev)
                #print(cnt)
                if prev in cnt:
                    cnt[word] = max(cnt[word], cnt[prev] + 1)
                    
            ans = max(ans, cnt[word])
        
        return ans
    
def test1():
    solution = Solution()
    words = ["a","b","ba","bca","bda","bdca"]
    assert(solution.longestStrChain(words) == 4)
    
def test2():
    solution = Solution()
    words = ["xbc","pcxbcf","xb","cxbc","pcxbc"]
    assert(solution.longestStrChain(words) == 5)
    
def test3():
    solution = Solution()
    words = ["abcd","dbqca"]
    assert(solution.longestStrChain(words) == 1)
    
if __name__ == '__main__':
    test1()
    test2()
    test3()
```

## lc 556

```python3
class Solution:
    def nextGreaterElement(self, n: int) -> int:
        nums = list(str(n))
        i = len(nums) - 2
        while i >= 0 and nums[i] >= nums[i + 1]:
            i -= 1
        if i < 0:
            return -1
        
        j = len(nums) - 1
        while j >= 0 and nums[i] >= nums[j]:
            j -= 1
        
        nums[i], nums[j] = nums[j], nums[i]
        nums[i + 1:] = nums[i + 1:][::-1]
        ans = int(''.join(nums))
        
        
        return ans if ans < 2 ** 31 else -1
    
def test1():
    solution = Solution()
    n = 12
    assert(solution.nextGreaterElement(n) == 21)
    
def test2():
    solution = Solution()
    n = 21
    assert(solution.nextGreaterElement(n) == -1)
    
def test3():
    solution = Solution()
    n = 1234
    assert(solution.nextGreaterElement(n) == 1243)
    
if __name__ == '__main__':
    test1()
    test2()
    test3()
```
