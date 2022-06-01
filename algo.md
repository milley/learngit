## lc167 two sum II

```python3
from typing import List

class Solution:
    def twoSum(self, numbers: List[int], target: int) -> List[int]:
        n = len(numbers)
        for i in range(n):
            low, high = i + 1, n - 1
            while low <= high:
                mid = (low + high) // 2
                if numbers[mid] == target - numbers[i]:
                    return [i + 1, mid + 1]
                elif numbers[mid] > target - numbers[i]:
                    high = mid - 1
                else:
                    low = mid + 1
        return [-1, -1]
    
    def twoSum1(self, numbers: List[int], target: int) -> List[int]:
        n = len(numbers)
        i = 0
        j = n - 1
        while i < j:
            if numbers[i] + numbers[j] == target:
                return [i + 1, j + 1]
            else:
                if numbers[i] + numbers[j] < target:
                    i += 1
                else:
                    j -= 1
        return []
    
def main():
    solution = Solution()
    list = [2,7,11,15]
    target = 9
    l = solution.twoSum1(list, target)
    print(l)
    
if __name__ == '__main__':
    main()

```

## lc125 Valid Palindrome

```python3
class Solution:
    def isPalindrome(self, s: str) -> bool:
        n = len(s)
        i = 0
        j = n - 1
        while i < j:
            if not s[i].isalnum():
                i += 1
                continue
            if not s[j].isalnum():
                j -= 1
                continue
            if s[i].lower() == s[j].lower():
                i += 1
                j -= 1
            else:
                #print("i={}, j={}".format(i, j))
                return False
        return True
    
def main():
    solution = Solution()
    #s = "A man, a plan, a canal: Panama"
    s = "race a car"
    b = solution.isPalindrome(s)
    assert(b == False)

if __name__ == '__main__':
    main()
```

## lc345 Reverse Vowels of a String

```python3
class Solution:
    def reverseVowels(self, s: str) -> str:
        i, j = 0, len(s) - 1
        l = list(s)
        vowels = "aeiou"
        while i < j:
            if vowels.find(s[i].lower()) == -1:
                i += 1
                continue
            if vowels.find(s[j].lower()) == -1:
                j -= 1
                continue
            if vowels.find(s[i].lower()) != -1 and vowels.find(s[j].lower()) != -1:
                l[i], l[j] = l[j], l[i]
                i, j = i + 1, j - 1
        return "".join(l)
                
def main():
    solution = Solution()
    #s = "hello"
    s = "leetcode";
    ret_str = solution.reverseVowels(s)
    print(ret_str)
    
if __name__ == '__main__':
    main()
```

## lc11 Container With Most Water

```python3
from typing import List


class Solution:
    def maxArea(self, height: List[int]) -> int:
        n = len(height)
        i, j = 0, n - 1
        max_area = 0
        while i < j:
            tmp = 0
            if height[i] < height[j]:
                tmp = (j - i) * height[i]
                i += 1
            else:
                tmp = (j - i) * height[j]
                j -= 1
            if tmp > max_area:
                max_area = tmp
        return max_area
    
def main():
    solution = Solution()
    #height = [1,8,6,2,5,4,8,3,7]
    height = [1,1]
    m = solution.maxArea(height)
    #assert(m == 49)
    assert(m == 1)
    
if __name__ == '__main__':
    main()
```
