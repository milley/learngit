## lc144

```python3
from typing import Optional, List

# Definition for a binary tree node.
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
class Solution:
    def __init__(self):
        self.array = []
        
    def preorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        if not root:
            return []
        self.array.append(root.val)
        self.preorderTraversal(root.left)
        self.preorderTraversal(root.right)
        
        return self.array
    
def test1():
    solution = Solution()
    node1 = TreeNode(1, None, None)
    node2 = TreeNode(2, None, None)
    node3 = TreeNode(3, None, None)
    node1.right = node2
    node2.left = node3
    
    #print(solution.preorderTraversal(node1))
    assert(solution.preorderTraversal(node1) == [1, 2, 3])
    
if __name__ == '__main__':
    test1()
```

## lc145

```python3
from typing import Optional, List

# Definition for a binary tree node.
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
class Solution:
    def __init__(self):
        self.array = []
        
    def postorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        if not root:
            return []
        
        self.postorderTraversal(root.left)
        self.postorderTraversal(root.right)
        self.array.append(root.val)
        
        return self.array
    
def test1():
    solution = Solution()
    node1 = TreeNode(1, None, None)
    node2 = TreeNode(2, None, None)
    node3 = TreeNode(3, None, None)
    node1.right = node2
    node2.left = node3
    
    assert(solution.postorderTraversal(node1) == [3, 2, 1])
    
if __name__ == '__main__':
    test1()
```

## lc324

```python3
class Solution:
    def isPowerOfFour(self, n: int) -> bool:
        while n and n % 4 == 0:
            n //= 4
        return n == 1
    
def test1():
    solution = Solution()
    assert(solution.isPowerOfFour(16))
    
            
if __name__ == '__main__':
    test1()
```

## lc326

```python3
class Solution:
    def isPowerOfThree(self, n: int) -> bool:
        while n and n % 3 == 0:
            n //= 3
        
        return n == 1
           
def test1():
    solution = Solution()
    assert(solution.isPowerOfThree(27))
    
            
if __name__ == '__main__':
    test1()
```

## lc344

```python3
from typing import List

class Solution:
    def reverseString(self, s: List[str]) -> None:
        n = len(s)
        i = 0
        while i < n // 2:
            s[i], s[n - i - 1] = s[n - i - 1], s[i]
            i += 1
        
        #print(s)
def test1():
    solution = Solution()
    s = ["h","e","l","l","o"]
    #s = ["H","a","n","n","a","h"]
    solution.reverseString(s)
            
if __name__ == '__main__':
    test1()
```

## lc451

```python3
class Solution:
    def frequencySort(self, s: str) -> str:
        d = {}
        for ch in s:
            if ch in d.keys():
                d[ch] += 1
            else:
                d[ch] = 1
        
        res = {key: val for key, val in sorted(d.items(), key = lambda ele: ele[1], reverse = True)}
        out = ""
        for k, v in res.items():
            out += k * v
        return out
    
def test1():
    solution = Solution()
    #s = "tree"
    #s = "cccaaa"
    s = "Aabb"
    print(solution.frequencySort(s))
    
if __name__ == "__main__":
    test1()
```

## lc509

```python3
class Solution:
    def fib(self, n: int) -> int:
        if n == 0: return 0
        if n == 1: return 1
        return self.fib(n - 1) + self.fib(n - 2)
    
    # x * x = x + 1
    def fib1(self, n: int) -> int:
        sqrt5 = 5 ** 0.5
        fibN = ((1 + sqrt5) / 2) ** n - ((1 - sqrt5) / 2) ** n
        return round(fibN / sqrt5)
    
if __name__ == '__main__':
    solution = Solution()
    assert(solution.fib1(2) == 1)
    assert(solution.fib1(3) == 2)
    assert(solution.fib1(4) == 3)
```

## lc1921

```python3
from typing import List

class Solution:
    def eliminateMaximum(self, dist: List[int], speed: List[int]) -> int:
        digits = []
        n = len(dist)
        for i in range(n):
            digits.append((dist[i] - 1) // speed[i])
        #digits.sort()
        digits = sorted(digits)
        
        for i in range(n):
            if digits[i] < i:
                return i
        
        return n
    
def test1():
    solution = Solution()
    dist = [1,3,4]
    speed = [1,1,1]
    assert(solution.eliminateMaximum(dist, speed) == 3)
    
def test2():
    solution = Solution()
    dist = [1,1,2,3]
    speed = [1,1,1,1]
    assert(solution.eliminateMaximum(dist, speed) == 1)
    
def test3():
    solution = Solution()
    dist = [3,2,4]
    speed = [5,3,2]
    assert(solution.eliminateMaximum(dist, speed) == 1)
    
def test4():
    solution = Solution()
    dist = [3,5,7,4,5]
    speed = [2,3,6,3,2]

    assert(solution.eliminateMaximum(dist, speed) == 2)
    
def test5():
    solution = Solution()
    dist = [4,3,4]
    speed = [1,1,2]

    assert(solution.eliminateMaximum(dist, speed) == 3)
    
if __name__ == '__main__':
    test1()
    test2()
    test3()
    test4()
    test5()
```

## lc2165

```python3
class Solution:
    def smallestNumber(self, num: int) -> int:
        arr = []
        abs_num = abs(num)
        while abs_num != 0:
            last_num = abs_num % 10
            arr.append(last_num)
            abs_num //= 10
        
        
        if num >= 0:
            arr.sort()
            
            zero_start = False
            if len(arr) > 0 and arr[0] == 0:
                zero_start = True
            for i in range(len(arr)):
                if zero_start and arr[i] != 0:
                    arr[0], arr[i] = arr[i], arr[0]
                    break
                    
        else:
            arr.sort(reverse=True)
        
        n = len(arr)
        result = 0
        i = 0
        while i < n:
            left_num = arr[i]
            result += left_num * (10 ** (n - i - 1))
            i += 1
        
        return result if num > 0 else result * -1
    
    def smallestNumber1(self, num: int) -> int:
        if num == 0:
            return 0
        
        negative = (num) < 0
        num = abs(num)
        digits = sorted(int(digit) for digit in str(num))
        
        if negative:
            digits = digits[::-1]
        else:
            if digits[0] == 0:
                i = 1
                while digits[i] == 0:
                    i += 1
                digits[0], digits[i] = digits[i], digits[0]

        ans = int("".join(str(digit) for digit in digits))
        return -ans if negative else ans
    
def test1():
    solution = Solution()
    assert(solution.smallestNumber1(301) == 103)
    assert(solution.smallestNumber1(-7605) == -7650)
    
if __name__ == "__main__":
    test1()
```
