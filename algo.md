## lc155

```python3
from typing import List
from operator import add, sub, mul

class Solution:
    def evalRPN(self, tokens: List[str]) -> int:
        stack = []
        if len(tokens) == 1:
            return int(tokens[0])
        
        for expr in tokens:
            if expr == '+':
                right = stack[-1]
                stack.pop()
                left = stack[-1]
                stack.pop()
                stack.append(int(left) + int(right))
            elif expr == '-':
                right = stack[-1]
                stack.pop()
                left = stack[-1]
                stack.pop()
                stack.append(int(left) - int(right))
            elif expr == '*':
                right = stack[-1]
                stack.pop()
                left = stack[-1]
                stack.pop()
                stack.append(int(left) * int(right))
            elif expr == '/':
                right = stack[-1]
                stack.pop()
                left = stack[-1]
                stack.pop()
                stack.append(int(left) // abs(int(right)))
            else:
                stack.append(expr)
        
        return stack[-1]
    
    def evalRPN1(self, tokens: List[str]) -> int:
        op_to_binary_fn = {
            "+": add,
            "-": sub,
            "*": mul,
            "/": lambda x, y: int(x / y),
        }
        
        stack = list()
        for token in tokens:
            try:
                num = int(token)
            except ValueError:
                num2 = stack.pop()
                num1 = stack.pop()
                num = op_to_binary_fn[token](num1, num2)
            finally:
                stack.append(num)
        return stack[0]

    def evalRPN2(self, tokens: List[str]) -> int:
        s = []
        for token in tokens:
            if token not in '+-*/':
                s.append(int(token))
            else:
                n2 = s.pop()
                n1 = s.pop()
                if token == '+': n = n1 + n2
                elif token == '-': n = n1 - n2
                elif token == '*': n = n1 * n2
                else: n = int(n1 / n2)
                s.append(n)
        return s[-1]

def test1():
    solution = Solution()
    tokens = ["2","1","+","3","*"]
    assert(solution.evalRPN2(tokens) == 9)

def test2():
    solution = Solution()
    tokens = ["4","13","5","/","+"]
    assert(solution.evalRPN2(tokens) == 6)
    
def test3():
    solution = Solution()
    tokens = ["10","6","9","3","+","-11","*","/","*","17","+","5","+"]
    assert(solution.evalRPN2(tokens) == 22)
    
if __name__ == '__main__':
    test1()
    test2()
    test3()
```

```cpp
#include <stack>
#include <vector>
#include <string>
#include <cassert>
using namespace std;

class Solution {
public:
    int evalRPN(vector<string>& tokens) {
        stack<int> s;
        for (const string& token : tokens) {
            if (isdigit(token.back())) {
                s.push(stoi(token));
            } else {
                int n2 = s.top(); s.pop();
                int n1 = s.top(); s.pop();
                int n = 0;
                switch (token[0]) {
                    case '+': n = n1 + n2; break;
                    case '-': n = n1 - n2; break;
                    case '*': n = n1 * n2; break;
                    case '/': n = n1 / n2; break;
                }
                s.push(n);
            }
        }
        return s.top();
    }
};

void test1() {
    Solution solution;
    vector<string> vec{"2","1","+","3","*"};
    assert(solution.evalRPN(vec) == 9);
}

void test2() {
    Solution solution;
    vector<string> vec{"4","13","5","/","+"};
    assert(solution.evalRPN(vec) == 6);
}

void test3() {
    Solution solution;
    vector<string> vec{"10","6","9","3","+","-11","*","/","*","17","+","5","+"};
    assert(solution.evalRPN(vec) == 22);
}

int main() {
    test1();
    test2();
    test3();

    return 0;
}
```

## lc200

```python3
import collections
from typing import List

class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        x, y = len(grid), len(grid[0])
        cnt = 0
        if x == 0:
            return 0
        for r in range(x):
            for c in range(y):
                if grid[r][c] == "1":
                    cnt += 1
                    grid[r][c] = "0"
                    neighbors = collections.deque([(r, c)])
                    #print(list(neighbors))
                    while neighbors:
                        row, col = neighbors.popleft()
                        print("{} {}".format(row, col))
                        for xx, yy in [(row - 1, col), (row + 1, col), (row, col - 1), (row, col + 1)]:
                            if 0 <= xx < x and 0 <= yy < y and grid[xx][yy] == "1":
                                neighbors.append((xx, yy))
                                grid[xx][yy] = "0"
                    #print(grid)
        return cnt
    
    def dfs(self, grid, r, c):
        grid[r][c] = 0
        nr, nc = len(grid), len(grid[0])
        for x, y in [(r - 1, c), (r + 1, c), (r, c - 1), (r, c + 1)]:
            if 0 <= x < nr and 0 <= y < nc and grid[x][y] == "1":
                self.dfs(grid, x, y)
    
    def numIslands1(self, grid: List[List[str]]) -> int:
        nr = len(grid)
        if nr == 0:
            return 0
        nc = len(grid[0])
        
        num_islands = 0
        for r in range(nr):
            for c in range(nc):
                if grid[r][c] == "1":
                    num_islands += 1
                    self.dfs(grid, r, c)
        return num_islands
    
def test1():
    solution = Solution()
    l = [
        ["1","1","1","1","0"],
        ["1","1","0","1","0"],
        ["1","1","0","0","0"],
        ["0","0","0","0","0"]
    ]
    print(solution.numIslands(l))
    
if __name__ == "__main__":
    test1()
```

## lc279

```python3
import sys
import math

class Solution:
    def numSquares(self, n: int) -> int:
        f = [0] * (n + 1)
        for i in range(1, n + 1):
            minn = sys.maxsize
            j = 1
            while j * j <= i:
                minn = min(minn, f[i - j * j])
                j += 1
            f[i] = minn + 1
        
        return f[n]
    
    def isPerfectSquare(self, x: int) -> bool:
        y = math.floor(math.sqrt(x))
        return x == y * y

    def checkAnswer4(self, x: int) -> bool:
        while x % 4 == 0:
            x //= 4
        return x % 8 == 7

    def numSquares1(self, n: int) -> int:
        if self.isPerfectSquare(n):
            return 1
        if self.checkAnswer4(n):
            return 4
        i = 1
        while i * i <= n:
            j = n - i * i
            if self.isPerfectSquare(j):
                return 2
            i += 1
        return 3
    
def test1():
    solution = Solution()
    assert(solution.numSquares(12) == 3)
    assert(solution.numSquares(13) == 2)
    
def test2():
    solution = Solution()
    assert(solution.numSquares1(12) == 3)
    assert(solution.numSquares1(13) == 2)
    
if __name__ == '__main__':
    test1()
    test2()
```

## lc752

```python3
from collections import deque
from typing import Generator, List

class Solution:
    def openLock(self, deadends: List[str], target: str) -> int:
        if target == "0000":
            return 0
        
        dead = set(deadends)
        if "0000" in dead:
            return -1
        
        def num_prev(x: str) -> str:
            return "9" if x == "0" else str(int(x) - 1)
        def num_succ(x: str) -> str:
            return "0" if x == "9" else str(int(x) + 1)
        
        def get(status: str) -> Generator[str, None, None]:
            s = list(status)
            for i in range(4):
                num = s[i]
                s[i] = num_prev(num)
                yield "".join(s)
                s[i] = num_succ(num)
                yield "".join(s)
                s[i] = num
        
        q = deque([("0000", 0)])
        seen = {"0000"}
        while q:
            status, step = q.popleft()
            for next_status in get(status):
                if next_status not in seen and next_status not in dead:
                    if next_status == target:
                        return step + 1
                    q.append((next_status, step + 1))
                    seen.add(next_status)
        
        return -1
        
def test1():
    solution = Solution()
    deadends = ["0201","0101","0102","1212","2002"]
    target = "0202"
    assert(solution.openLock(deadends, target) == 6)
    
def test2():
    solution = Solution()
    deadends = ["8888"]
    target = "0009"
    assert(solution.openLock(deadends, target) == 1)
        
def test3():
    solution = Solution()
    deadends = ["8887","8889","8878","8898","8788","8988","7888","9888"]
    target = "8888"
    assert(solution.openLock(deadends, target) == -1)

        
if __name__ == "__main__":
    test1()
    test2()
    test3()
```

```cpp
#include <string>
#include <unordered_set>
#include <queue>
#include <vector>
#include <cassert>
using namespace std;

class Solution {
public:
    int openLock(vector<string>& deadends, string target) {
        const string start = "0000";
        unordered_set<string> dead(deadends.begin(), deadends.end());
        if (dead.count(start)) return -1;

        queue<string> q;
        unordered_set<string> visited{start};

        int steps = 0;
        q.push(start);
        while (!q.empty()) {
            ++steps;
            const int size = q.size();
            for (int i = 0; i < size; ++i) {
                const string curr = q.front();
                q.pop();
                for (int i = 0; i < 4; ++i) {
                    for (int j = -1; j <= 1; j += 2) {
                        string next = curr;
                        next[i] = (next[i] - '0' + j + 10) % 10 + '0';
                        if (next == target) return steps;
                        if (dead.count(next) || visited.count(next)) continue;
                        q.push(next);
                        visited.insert(next);
                    }
                }
            }
        }

        return -1;
    }
};

int main() {
    Solution solution;
    vector<string> vec{"0201","0101","0102","1212","2002"};
    string target = "0202";
    assert(solution.openLock(vec, target) == 6);

    return 0;
}
```
