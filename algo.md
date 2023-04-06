## lc1638

```python3
class Solution:
    def countSubstrings(self, s: str, t: str) -> int:
        return self.countSubstrings2(s, t)

    def countSubstrings1(self, s: str, t: str) -> int:
        ans = 0
        for i in range(len(s)):
            for j in range(len(t)):
                diff = 0
                k = 0
                while i + k < len(s) and j + k < len(t):
                    if s[i + k] != t[j + k]:
                        diff += 1
                    if diff == 1:
                        print("i:{} j:{} k:{}".format(i, j, k))
                        ans += 1
                    elif diff > 1:
                        break
                    k += 1
        return ans

    def countSubstrings2(self, s: str, t: str) -> int:
        m, n = len(s), len(t)
        dpl = [[0] * (n + 1) for _ in range(m + 1)]
        dpr = [[0] * (n + 1) for _ in range(m + 1)]

        #print(dpl)
        #print(dpr)
        
        for i in range(m):
            for j in range(n):
                dpl[i + 1][j + 1] = (dpl[i][j] + 1) if s[i] == t[j] else 0

        for i in reversed(range(m)):
            for j in reversed(range(n)):
                dpr[i][j] = (dpr[i + 1][j + 1] + 1) if s[i] == t[j] else 0

        ans = 0
        for i in range(m):
            for j in range(n):
                if s[i] != t[j]:
                    ans += (dpl[i][j] + 1) * (dpr[i + 1][j + 1] + 1)
        return ans

def test1():
    solution = Solution()

    s1 = "aba"
    s2 = "baba"
    assert(solution.countSubstrings(s1, s2) == 6)


def test2():
    solution = Solution()

    s1 = "ab"
    s2 = "bb"
    assert(solution.countSubstrings(s1, s2) == 3)

def test3():
    solution = Solution()

    s1 = "a"
    s2 = "a"
    assert(solution.countSubstrings(s1, s2) == 0)

def test4():
    solution = Solution()

    s1 = "abe"
    s2 = "bbc"
    assert(solution.countSubstrings(s1, s2) == 10)

if __name__ == '__main__':
    test1()
    test2()
    test3()
    test4()

```

```cpp
#include <vector>
#include <string>
#include <iostream>

#include <cassert>

using namespace std;

class Solution {
public:
    int countSubstrings(string s, string t) {
        return countSubstrings2(s, t);
    }

    int countSubstrings1(string s, string t) {
        int m = s.size(), n = t.size();
        int ans = 0;

        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                int diff = 0;
                for (int k = 0; i + k < m && j + k < n; k++) {
                    diff += s[i + k] == t[j + k] ? 0 : 1;
                    if (diff > 1) {
                        break;
                    } else if (diff == 1) {
                        ans++;
                    }
                }
            }
        }

        return ans;
    }

    int countSubstrings2(string s, string t) {
        int m = s.size(), n = t.size();
        vector<vector<int>> dpl(m + 1, vector<int>(n + 1));
        vector<vector<int>> dpr(m + 1, vector<int>(n + 1));

        //print(dpl); print(dpr);
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                dpl[i + 1][j + 1] = s[i] == t[j] ? (dpl[i][j] + 1) : 0;
            }
        }
        for (int i = m - 1; i >= 0; --i) {
            for (int j = n - 1; j >= 0; --j) {
                dpr[i][j] = s[i] == t[j] ? (dpr[i + 1][j + 1] + 1) : 0;
            }
        }

        int ans = 0;
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (s[i] != t[j]) {
                    ans += (dpl[i][j] + 1) * (dpr[i + 1][j + 1] + 1);
                }
            }
        }

        return ans;
    }

private:
    void print(vector<vector<int>>& v) {
        cout << "[" << endl;
        for (vector<int>& vv : v) {
            cout << "[";
            for (int& x : vv) {
                cout << x << " ";
            }
            cout << "]" << endl;
        }

        cout << "]" << endl;
    }
};

int test(Solution& solution, string s, string t) {
    return solution.countSubstrings(s, t);
}

int main() {
    Solution solution;

    string s = "aba";
    string t = "baba";
    assert(test(solution, s, t) == 6);

    s = "ab";
    t = "bb";
    assert(test(solution, s, t) == 3);

    s = "a";
    t = "a";
    assert(test(solution, s, t) == 0);

    s = "abe";
    t = "bbc";
    assert(test(solution, s, t) == 10);

    return 0;
}

```

## lc2367

```python3
from typing import List

class Solution:
    def arithmeticTriplets(self, nums: List[int], diff: int) -> int:
        s = set(nums)
        res = 0
        for num in nums:
            if (num + diff) in s and (num + 2 * diff) in s:
                res += 1
        
        return res
    
def test1():
    solution = Solution()
    nums = [0,1,4,6,7,10]
    diff = 3
    assert(solution.arithmeticTriplets(nums, diff) == 2)
    
def test2():
    solution = Solution()
    nums = [4,5,6,7,8,9]
    diff = 2
    assert(solution.arithmeticTriplets(nums, diff) == 2)
    
if __name__ == '__main__':
    test1()
    test2()
```

## lc2600

```python3
class Solution:
    def kItemsWithMaximumSum(self, numOnes: int, numZeros: int, numNegOnes: int, k: int) -> int:
        if k < numOnes:
            return k
        elif k < numOnes + numZeros:
            return numOnes
        
        return numOnes - (k - numOnes - numZeros)
    
def test1():
    solution = Solution()
    numOnes = 3
    numZeros = 2
    numNegOnes = 0
    k = 2
    assert(solution.kItemsWithMaximumSum(numOnes, numZeros, numNegOnes, k) == 2)
    
def test2():
    solution = Solution()
    numOnes = 3
    numZeros = 2
    numNegOnes = 0
    k = 4
    assert(solution.kItemsWithMaximumSum(numOnes, numZeros, numNegOnes, k) == 3)
    
if __name__ == '__main__':
    test1()
    test2()
```
