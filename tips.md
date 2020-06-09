```python
from bs4 import BeautifulSoup
import pandas as pd
import os

with open("C:\\test.html") as fp:
    soup = BeautifulSoup(fp, features="lxml")

table = soup.body.table
result = []
for tr in table.find_all("tr"):
    listTemp = []
    td0 = tr.find_all("td")[0]
    td1 = tr.find_all("td")[1]
    td2 = tr.find_all("td")[2]
    td3 = tr.find_all("td")[3]
    td4 = tr.find_all("td")[4]
    listTemp.append(td0.string)
    listTemp.append(td1.string)
    listTemp.append(td2.string)
    listTemp.append(td3.string)
    listTemp.append(td4.string)
    result.append(listTemp)

df1 = pd.DataFrame(result)
df1.to_excel("C:\\log.xlsx", header=False, index=False)
```
