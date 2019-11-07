# lc37 solveSudoku

```c++
#include <vector>
#include <iostream>

using namespace std;

class Solution {
public:
	void solveSudoku(vector<vector<char>>& board) {
		isValidSudoku(board);
	}
	bool isValidSudoku(vector<vector<char>>& board) {
		int i = 0;
		for (vector<vector<char>>::iterator iterRow = board.begin(); iterRow != board.end(); ++iterRow)
		{
			int j = 0;
			vector<char> rowVector = *iterRow;
			for (vector<char>::iterator iterCol = rowVector.begin(); iterCol != rowVector.end(); ++iterCol)
			{
				char ch = *iterCol;
				if (ch >= '1' && ch <= '9')
				{
					j++;
					continue;
				}
				else if (ch == '.')
				{
					for (char inCh = '1'; inCh <= '9'; ++inCh)
					{
						if (isValid(board, inCh, i, j))
						{
							board[i][j] = inCh;
							if (isValidSudoku(board))
							{
								return true;
							}
							else
							{
								board[i][j] = '.';
							}
						}
					}
					return false;
				}
				j++;
			}
			i++;
		}
		return true;
	}

	bool isValid(vector<vector<char>>& board, char inCh, int idx, int jdx)
	{
		int i = 0;
		for (vector<vector<char>>::iterator rowIter = board.begin(); rowIter < board.end(); ++rowIter)
		{
			int j = 0;
			vector<char> rowVector = *rowIter;
			if (rowVector.at(jdx) == inCh)
			{
				return false;
			}
			for (vector<char>::iterator colIter = rowVector.begin(); colIter < rowVector.end(); ++colIter)
			{
				if (*colIter == inCh && i == idx)
				{
					return false;
				}
				int startRow = (idx / 3) * 3;
				int endRow = (idx / 3) * 3 + 2;
				int startCol = (jdx / 3) * 3;
				int endCol = (jdx / 3) * 3 + 2;
				if (i >= startRow && i <= endRow && j >= startCol && j <= endCol)
				{
					if (*colIter == inCh)
					{
						return false;
					}
				}
				j++;
			}
			i++;
		}
		return true;
	}
};

int main()
{
	char arr[9][9]  = {
	{'5','3','.','.','7','.','.','.','.'},
	{'6','.','.','1','9','5','.','.','.'},
	{'.','9','8','.','.','.','.','6','.'},
	{'8','.','.','.','6','.','.','.','3'},
	{'4','.','.','8','.','3','.','.','1'},
	{'7','.','.','.','2','.','.','.','6'},
	{'.','6','.','.','.','.','2','8','.'},
	{'.','.','.','4','1','9','.','.','5'},
	{'.','.','.','.','8','.','.','7','9'}
	};

	vector<vector<char>> board(9, vector<char>(9));
	for (int i = 0; i < 9; i++)
	{
		for (int j = 0; j < 9; j++)
		{
			board[i][j] = arr[i][j];
		}
	}
	
	Solution solution;
	solution.solveSudoku(board);

	for (int i = 0; i < 9; i++)
	{
		for (int j = 0; j < 9; j++)
		{
			cout << board[i][j] << " ";
		}
		cout << endl;
	}
}
```
