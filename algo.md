## lc414

```python3
from typing import List

class Solution:
    def thirdMax(self, nums: List[int]) -> int:
        nums = list(set(nums))
        nums.sort(reverse = True)
        #print(nums)
        return nums[2] if len(nums) > 2 else nums[0]
    
def test1():
    solution = Solution()
    nums = [3, 2, 1]
    assert(solution.thirdMax(nums) == 1)
    
def test2():
    solution = Solution()
    nums = [1, 2]
    assert(solution.thirdMax(nums) == 2)
    
def test3():
    solution = Solution()
    nums = 2, 2, 3, 1
    assert(solution.thirdMax(nums) == 1)
    
if __name__ == '__main__':
    test1()
    test2()
    test3()
    
```
