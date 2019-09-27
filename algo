#include <vector>
#include <map>
#include <iostream>

using namespace std;

class Solution
{
public:
	bool lemonadeChange(vector<int>& bills) {
		map<int, int> billMap;
		for(vector<int>::iterator iter = bills.begin(); iter != bills.end(); ++iter)
		{
			if (*iter == 5)
			{
				billMap[5]++;
			}
			else if (*iter == 10)
			{
				if (billMap[5] > 0)
				{
					billMap[5]--;
					billMap[10]++;
				}
				else
				{
					return false;
				}
			}
			else if (*iter == 20)
			{
				if (billMap[10] > 0)
				{
					if (billMap[5] > 0)
					{
						billMap[10]--;
						billMap[5]--;
						billMap[20]++;
					}
					else
					{
						if (billMap[5] >= 3)
						{
							billMap[5] = billMap[5] - 3;
						}
						else
						{
							return false;
						}
					}
				}
				else
				{
					if (billMap[5] >= 3)
					{
						billMap[5] = billMap[5] - 3;
					}
					else
					{
						return false;
					}
				}
			}
		}
		return true;
	}

	bool lemonadeChangeWithoutMap(vector<int>& bills) {
		int fiveCnt = 0, tenCnt = 0;
		for(vector<int>::iterator iter = bills.begin(); iter != bills.end(); ++iter)
		{
			switch (*iter)
			{
			case 5:
				fiveCnt++;
				break;
			case 10:
				if (fiveCnt > 0)
				{
					fiveCnt--;
					tenCnt++;
				}
				else
				{
					return false;
				}
				break;
			case 20:
				if (tenCnt > 0 && fiveCnt > 0)
				{
					tenCnt--;
					fiveCnt--;
				}
				else if (fiveCnt >= 3)
				{
					fiveCnt -= 3;
				}
				else return false;
				break;
			default:
				break;
			}
		}
		return true;
	}
};
