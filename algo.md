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

## lc452

```python3
from typing import List

class Solution:
    def findMinArrowShots(self, points: List[List[int]]) -> int:
        if not points:
            return 0
        
        points.sort(key = lambda balloon: balloon[1])
        pos = points[0][1]
        ans = 1
        for balloon in points:
            if balloon[0] > pos:
                pos = balloon[1]
                ans += 1
                
        return ans
    
def test1():
    solution = Solution()
    points = [[10,16],[2,8],[1,6],[7,12]]
    assert(solution.findMinArrowShots(points) == 2)
    
def test2():
    solution = Solution()
    points = [[1,2],[3,4],[5,6],[7,8]]
    assert(solution.findMinArrowShots(points) == 4)
    
def test3():
    solution = Solution()
    points = [[1,2],[2,3],[3,4],[4,5]]
    assert(solution.findMinArrowShots(points) == 2)
    
if __name__ == "__main__":
    test1()
    test2()
    test3()
```

```cpp
#include <vector>
#include <algorithm>
#include <cassert>
using namespace std;

class Solution {
public:
    int findMinArrowShots(vector<vector<int>>& points) {
        if (points.size() == 0) {
            return 0;
        }

        sort(points.begin(), points.end(), [](const vector<int>& a, const vector<int>& b) { return a[1] < b[1]; });
        int pos = points[0][1];
        int ans = 1;
        for (auto& balloon : points) {
            if (balloon[0] > pos) {
                pos = balloon[1];
                ans++;
            }
        }

        return ans;
    }
};

void test1() {
    Solution solution;
    vector<vector<int>> points = {{10,16},{2,8},{1,6},{7,12}};
    assert(solution.findMinArrowShots(points) == 2);
}

void test2() {
    Solution solution;
    vector<vector<int>> points = {{1,2},{3,4},{5,6},{7,8}};
    assert(solution.findMinArrowShots(points) == 4);
}

void test3() {
    Solution solution;
    vector<vector<int>> points = {{1,2},{2,3},{3,4},{4,5}};
    assert(solution.findMinArrowShots(points) == 2);
}

int main() {
    test1();
    test2();
    test3();
}
```
