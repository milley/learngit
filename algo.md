## lc414

```python3
from typing import List
from sortedcontainers import SortedList

class Solution:
    def thirdMax(self, nums: List[int]) -> int:
        nums = list(set(nums))
        nums.sort(reverse = True)
        #print(nums)
        return nums[2] if len(nums) > 2 else nums[0]
    
    def thirdMax1(self, nums: List[int]) -> int:
        s = SortedList()
        for num in nums:
            if num not in s:
                s.add(num)
                if len(s) > 3:
                    s.pop(0)
        return s[0] if len(s) == 3 else s[-1]
    
    def thirdMax2(self, nums: List[int]) -> int:
        a, b, c = float('-inf'), float('-inf'), float('-inf')
        for num in nums:
            if num > a:
                a, b, c = num, a, b
            elif a > num > b:
                b, c = num, b
            elif b > num > c:
                c = num
        
        return a if c == float('-inf') else c
    
def test1():
    solution = Solution()
    nums = [3, 2, 1]
    assert(solution.thirdMax2(nums) == 1)
    
def test2():
    solution = Solution()
    nums = [1, 2]
    assert(solution.thirdMax2(nums) == 2)
    
def test3():
    solution = Solution()
    nums = 2, 2, 3, 1
    assert(solution.thirdMax2(nums) == 1)
    
if __name__ == '__main__':
    test1()
    test2()
    test3()
```
