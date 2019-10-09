```python
import os

if len(os.sys.argv) < 2:
    print('参数缺少，请检查参数')
    exit(-1)

weekIndex = os.sys.argv[1]
artsDir = r"E:\ARTS\week"

print(os.path.exists(artsDir+weekIndex))
artsPath = artsDir + weekIndex
dirExists = os.path.exists(artsPath)

if not dirExists:
    os.makedirs(artsPath + r"\algo")
    os.makedirs(artsPath + r"\review")
    os.makedirs(artsPath + r"\tip")
    os.makedirs(artsPath + r"\share")
    print(artsPath + " 创建成功")
```
