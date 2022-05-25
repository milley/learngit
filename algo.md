## lc27 removeElements

```python3
class Solution:
    def removeElement(self, nums: List[int], val: int) -> int:
        n = len(nums)
        i = -1
        j = 0
        cnt = 0
        while j <= n - 1:
            if nums[j] != val:
                i += 1
                cnt += 1
                nums[i] = nums[j]
            j += 1
        
        #print(nums)
        return cnt
```

## lc283 removeZeros

```python3
from typing import List

class Solution:
    def moveZeroes(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        length = len(nums)
        for i in range(length):
            j = i + 1
            while j < length and nums[j] == 0:
                j = j + 1
            
            #print("i = {0}, j = {1}".format(i, j))
            if nums[i] == 0 and j < length:
                nums[i], nums[j] = nums[j], nums[i]
            
        #print(nums)
        
    def moveZeroes1(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        n = len(nums)
        i = -1
        j = 0
        # nums[0....i]表示非0元素的数列,初始值i=-1
        while j <= n-1:
            if nums[j] != 0:
                i += 1
                nums[i] = nums[j]
            j += 1
        for k in range(i+1, n):
            nums[k] = 0
        print(nums)
```
