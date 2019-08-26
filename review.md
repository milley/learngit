# 为什么要使用pathlib

当我不就之前发现一个新的python模块pathlib，一开始我把它写的有点尴尬以为只是面向对象版本的不必要的os.path模块。我搞错了。Python的pathlib模块实际上非常完美！

在这个文章我打算尝试着把pathlib推销给你。通过这篇文章我希望鼓励你在相当多的时间在Python中需要用文件就使用pathlib模块。

## os.path是不灵活的

os.path模块一直以来可以达成工作路径的目标。它有相当多你需要的，但是有时候显得非常笨重。

你有没有这样导入过它？

```python
import os.path

BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
TEMPLATE_DIR = os.path.join(BASE_DIR, 'templates')
```

或者像这样？

```python
from os.path import abspath, dirname, join

BASE_DIR = dirname(dirname(abspath(__file__)))
TEMPLATES_DIR = join(BASE_DIR, 'templates')
```

或者join函数太通用了...所以我们这样使用：

```python
from os.path import abspath, dirname, join as joinpath

BASE_DIR = dirname(dirname(abspath(__file__)))
TEMPLATES_DIR = joinpath(BASE_DIR, 'templates')
```

我发现这些都有一些笨重。我们把字符串传入函数然后返回再次传入另外一个函数返回的字符串。所有的字符串都表示路径，但是他们依旧是字符串。

在os.path中字符串进出函数的确有些不灵活是因为代码需要从里面函数的返回来读取。有没有优雅的方法当我们需要把函数的调用串起来来代替呢？

使用pathlib就可以！

```python
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent
TEMPLATES_DIR = BASE_DIR.joinpath('templates')
```

os.path需要函数嵌套，但是pathlib模块的Path类允许我们把方法和属性串起来，在Path对象上等同于获取的路径。

我知道你在考虑什么：Path对象不是相同的事物：它们是对象，不是路径字符串！随后我将会讲解这块。

## os模块是拥挤的

Python著名的os.path模块仅仅用在路径。当你需要真正的操作路径(例如创建目录)你需要转到另外一个Python的模块，大部分是os模块。

os模块有很多使用的文件操作：mkdir,getcwd,chmod,stat,remove,rename和rmdir。还有chdir,link,walk,listdir,makedirs,renames,removedirs,unlink(和remove相同)和symlink。还有一串非文件系统的操作：fork,getenv,putenv,evniron,getlogin和system。还有很多这里没有介绍的。

Python的os模块有一些亮点，也有一些简直是个系统相关操作的垃圾抽屉。在os模块有很多好用的东西，但有时候你想用的时候又很难找：当你找路径相关或者文件系统相关的东西，你需要一些时间来挖掘。

pathlib模块替换了os实用的大多数文件系统相关操作，在Path对象上操作方法。

这里有代码例子创建srd/__pypackages__路径然后将.editorconfig文件改名到src/.editorconfig:

```python
import os
import os.path

os.makedirs(os.path.join('src', '__pypackages__'), exist_ok=True)
os.rename('.editorconfig', os.path.join('src', '.editorconfig'))
```

实用Path对象做同样的事情：

```python
from pathlib import Path

Path('src/__pypackages__').mkdir(parents=True, exist_ok=True)
Path('.editorconfig').rename('src/.editorconfig')
```

注意pathlib先把路径放入代码中是因为方法串联了！

作为Python之禅，"namespaces are one honking great idea, let's do more of those"。os模块是一个包含了很多有用东西的命名空间。pathlib.Path类是一个小巧的、特定命名空间。在Path命名空间增加了方法返回Path对象，相比嵌套函数它允许方法串起来。

## 不要忘记glob模块

os和os.path模块不仅仅是文件路径/文件系统相关功能的Python标准库。glob模块是另外一个易用的路径关系模块。

我们可以使用glob.glob函数来查找文件是否匹配：

```python
from glob import glob

top_level_csv_files = glob('*.csv')
all_csv_files = glob('**/*.csv', recursive=True)
```

用pathlibmokuai包含了glob的这类操作：

```python
from pathlib import Path

top_level_csv_files = Path.cwd().glob('*.csv')
all_csv_files = Path.cwd().rglob('*.csv')
```

你开始大量使用pathlib，你很快就会完全的忘记glob模块了：你所需要glob所有的功能Path对象都包含。

## pathlib使用更容易

pathlib模块使得一部分复杂的案例简单一些，但是它仍旧能把简单案例变得更简单。

需要读取一个或多个文件的文本？

你可以打开文件，读取它的内容然后使用with块关闭文件：

```python
from glob import glob

file_contents = []
for filename in glob('**/*.py', recursive=True):
    with open(filename) as python_file:
        file_contents.append(python_file.read())
```

你可以用Path对象的read_text方法然后读取文件内容到一个新的列表中：

```python
from pathlib import Path

file_contents = [
    path.read_text()
    for path in Path.cwd().rglob('*.py')
]
```

你需要写入到文件中？你可以使用open上下文管理器：

```python
with open('.editorconfig') as config:
    config.write('# config goes here')
```

你可以使用write_text方法：

```python
Path('.editorconfig').write_text('# config goes here')
```

如果你更喜欢使用open,是否是上下文管理器或者其他，你可以使用Path对象的open方法来代替。

```path
from pathlib import Path

path = Path('.editorconfig')
with path.open(mode='wt') as config:
    config.write('# config goes here')
```

在Python3.6你可以省略Path对象来使用内置的open函数：

```python
from pathlib import Path

path = Path('.editorconfig')
with open(path, mode='wt') as config:
    config.write('# config goes here')
```

## Path对象使你的代码更加清晰

以下3个变量指向什么？它们的值代表什么？

```python
person = '{"name": "Trey Hunner", "locaton": "San Diego"}'
pycon_2019 = "2019-05-01"
home_directory = "/home/trey"
```

它们每个变量都指向一个字符串。

这些字符串代表了不同的事情：一个是JSON blog，一个是date，一个是文件路径。

这些对这些对象更有用的表述：

```python
from datetime import data
from pathlib import Path

person = {"name": "Trey Hunner", "locaton": "San Diego"}
pycon_2019 = date(2019,5,1)
home_directory = Path('/home/trey')
```

JSON对象反序列化到字典，日期用datetime.date对象来描述，文件路径使用pathlib.Path对象描述。

使用Path对象使你的代码更加清晰。如果你需要表示一个日期你可以使用data对象。如果你需要表示一个文件路径使用Path对象。

我不是一个面向对象编程的强烈拥护者。类增加了另外一种抽象维度，抽象在一些时候增加了更多的复杂性。但是pathlib.Path类是一个有用的抽象。它快速的成为了普遍认可的抽象。

感谢PEP519,文件路径对象现在成为了标准。从Python3.6开始，内置open函数和各种os,shutil和os.path模块中的各种函数都可以在pathlib.Path很好的工作。从今天开始你可以使用pathlib不需要改变大多数的代码更好的工作在路径中。

## pathlib缺失了什么

虽然pathlib非常好，但是也没有包含所有。我碰巧发现缺失了一些我希望包括在pathlib模块内的。

第一个缺口是我注意到缺少了shutil等价的方法。当你可以传Path对象到高级别shutil函数来拷贝/删除/移动文件和文件夹，这没有等价的方法。

所以拷贝一个文件你依然需要这样：

```python
from pathlib import Path
from shutil import copyfile

source = Path('old_file.txt')
destination = Path('new_file.txt')
copyfile(source, destination)
```

这里也没有pathlib等价于os.chdir的方法。如果你需要改变当前工作路径你需要导入chdir：
# Works on earlier versions also
```python
from pathlib import Path
from os import chdir

parent = Path('..')
chdir(parent)
```

os.walk函数没有和pathlib相同的。尽管你可以使用pathlib很容易的创建自己的walk函数。我希望pathlib.Path对象应该包含更多的缺失操作。但即使有这些缺失的特征，我仍然觉得使用"pathlib"相比"os.path"更友好。

## 你经常使用pathlib吗

自从Python3.6，pathlib.Path对象几乎可以工作在任何路径相关的地方。所以如果你使用的Python3.6及以上我看没有任何理由不使用pathlib。

如果你使用的更早期版本的Python3，你可以将Path对象放到str对象中，以便在需要一个变更路径可以从字符串中拿到。虽然有点难看但是可以工作：

```python
from os import chdir
from pathlib import Path

chdir(Path('/home/trey'))   # Works on Python 3.6+
chdir(str(Path('/home/trey')))  # Works on earlier versions also
```

不管你使用的是Python3的哪个版本都建议你使用pathlib。如果你使用的是Python2那么第三方pathlib2模块可以通过PYPI来使用。

我发现使用pathlib可以经常让代码可读性更高。我大多数的工作在文件的代码都使用pathlib，我想你也可以做同样的事情。
