# 如何创建一个基于CMake的项目

[How to Build a CMake-Based Project](https://preshing.com/20170511/how-to-build-a-cmake-based-project/)

CMake是一个可以帮助你在任何你想要的平台编译C/C++程序的万能的工具。它被很多非常有名的开源项目使用，比如 LLVM, Qt, KDE 和Blender。

所有的基于CMake的项目都有一个名叫CMakeLists.txt的脚本，这个标识作为配置和创建这些项目的指南。本文不会给你教如何写CMake脚本，依我看来那是首要任务。

我准备了一个使用SDL2和OpenGL用来渲染3D logo的项目[CMake-based project](https://github.com/preshing/CMakeDemo)以供参考。

这里列出的信息适用于任何基于CMake的项目，可以随意跳过任何章节。但是，我建议先了解下前两个章节。

- The Source and Binary Folders
- The Configure and Generate Steps
- Running CMake from the Command Line
- Running cmake-gui
- Running ccmake
- Building with Unix Makefiles
- Building with Visual Studio
- Building with Xcode
- Building with Qt Creator
- Other CMake Features

如果你从来没使用过CMake，[这里](https://cmake.org/download/)可以找到安装包相关内容。在类Unix系统，包括Linux，通常可以使用包管理器安装。你可以使用MacPorts, Homebrew, Cygwin 或者 MSYS2来安装。

<img src="./img/cmake_1.png" width="60" >

## 源程序和二进制文件夹

CMake生成构建管道。构建管道可以是Visual Studio的.sln文件，Xcode的.xcodeproj文件或者Unix类型的Makefile。它也能生成其他类型。

生成一个构建管道，CMake需要知道源文件和二进制文件夹。源文件夹包含一个CMakeList.txt。二进制文件夹会生成构建管道。你可以在任何你想要的地方创建二进制文件夹。一般会在每个子目录下创建CMakeList.txt。

<img src="./img/cmake_2.png" width="60" >

保持二进制文件夹和源文件夹分开，你可以在任何时候通过clean命令清除二进制文件夹。你甚至可以创建不同的二进制文件夹，按不同的构建系统或者配置选项排列在一起。

catch是一个非常重要的概念。在二进制文件夹中有叫CMakeCache.txt的文本文件。这就是cache变量存储的地方。缓存变量包含了用户配置的选项就像[CMakeDemo](https://github.com/preshing/CMakeDemo)的DEMO_ENABLE_MULTISAMPLE选项，和加速CMake运行速度的预编译信息。

这意味着你不用把生成的构建管道提交到版本控制系统，通常包含的路径都是绝对路径。每次重新运行CMake克隆项目到新的文件夹。我通常在我的.gitignore文件增加一个规则*build*/。

## 配置和生成步骤

就像下面你看到的一样，将会有好多种方法运行CMake。不管如何运行，需要执行两个步骤：配置和生成。

<img src="./img/cmake_3.png" width="60" >

CMakeLists.txt脚本将在配置阶段执行。这个脚本负责定义目标。每个目标代表了可执行文件，库，或者其他构件管道的输出。

如果配置步骤成功--意味着CMakeList.txt完成没有任何错误--CMake将使用脚本定义的目标生成了构件管道。构件管道的类型依赖于使用的生成类型，将在下面章节解释。

在配置步骤还会发生额外的事情，依赖于CMakeLists.txt的内容。例如在我们的示例[CMakeDemo](https://github.com/preshing/CMakeDemo)中，配置就分为两步：

- 查找SDL2和OpenGL的头文件和库文件
- 在二进制文件夹中生成头文件demo-config.h，这将在C++代码中被包含

<img src="./img/cmake_4.png" width="60" >

在更复杂的项目中，配置步骤还会测试系统函数(例如传统Unix的configure脚本)，或者定义特殊的"install"目标(帮助创建可分配的包)。如果你在同一个二进制文件夹重新运行CMake，在子目录运行期间很多慢的步骤就会跳过，感谢缓存。

## 从命令行运行CMake

在运行CMake之前，确保你的项目和平台有必须的依赖项。[CMakeDemo](https://github.com/preshing/CMakeDemo)在Windows上你可以运行setup-win32.py。其他平台可以参考[Readme](https://github.com/preshing/CMakeDemo/blob/master/README.md)。

你总是想要告诉CMake使用哪个生成器。运行cmake --help列出了可用的生成器。

<img src="./img/cmake_5.png" width="60" >

创建二进制文件夹，cd到那个文件夹，然后运行cmake，在命令行指定源文件夹。使用-G指定想要的生成器。如果你忽略了-G选项，cmake将会给你选择一个(如果你不想要这次的选择，你也可以删除二进制文件夹然后再来一遍)。

```bash
mkdir build
cd build
cmake -G "Visual Studio 15 2017" ..
```

如果有项目指定的配置选项，你可以在命令行指定它们。例如CMakeDemo项目就有一个配置选项DEMO_ENABLE_MULTISAMPLE默认是0。你也可以通过-DDEMO_ENABLE_MULTISAMPLE=1选项来打开配置选项。修改DEMO_ENABLE_MULTISAMPLE的值将会更改demo-config.h的内容(在配置步骤由CMakeLists.txt生成的)。这个值也被存储在缓存中，因此随后运行也会持续。有的项目有不同的配置选项。

```bash
cmake -G "Visual Studio 15 2017" -DDEMO_ENABLE_MULTISAMPLE=1 ..
```

如果你改变主意重新指定了DEMO_ENABLE_MULTISAMPLE，你可以任何时候重新运行cmake。随后的运行中，你可以重新指定已有的二进制文件夹来代替传给cmake命令行的源文件夹。CMake将会从缓存中找到之前所有的设置，比如选择的生成器，然后重新运行它们。

```bash
cmake -DDEMO_ENABLE_MULTISAMPLE=0 .
```

你可以通过运行cmake -L -N .来查看缓存的项目定义变量。这里你将能看到[CMakeDemo](https://github.com/preshing/CMakeDemo)中DEMO_ENABLE_MULTISAMPLE选项保留默认值0。

## 运行cmake-gui

我倾向于命令行，但是CMake也提供了图形界面。图形界面提供了交互式的缓存变量设置方法。同样，确保你安装了项目的依赖。

运行cmake-gui来使用它，填入源文件路径和二进制路径，然后点击配置。

<img src="./img/cmake_6.png" width="60" >

如果二进制文件夹不存在，CMake将允许你创建它。它将再次询问你选择生成器。

<img src="./img/cmake_7.png" width="60" >

完成内部配置步骤后，图形界面将会展示缓存变量列表，和前面使用cmake -L -N .出来的效果差不多。新的缓存变量是红色的。你可以再次配置，红色高亮就会消失，自从变量不再是新的。

<img src="./img/cmake_8.png" width="60" >

如果你修改了一个缓存变量然后点击配置，新的缓存变量将会出现在你修改的结果中。红色高亮意味着帮助你看到任何新的变量，自定义它们然后重新配置。实际上，更改一个值不会经常引入新的缓存变量。它依赖于项目中CMakeLists.txt是如何写的。

一旦你将缓存变量自定义为你喜欢的内容，点击生成。这将会在二进制文件夹中生成构建管道。你可以使用它们来构建你的项目。

## 运行ccmake

ccmake是一个类似cmake-gui的命令行。类似于图形界面，它允许你交互式设置缓存变量。它可以很方便的运行在远程机器上，如果你只能运行命令行。

<img src="./img/cmake_8.png" width="60" >

## 运行Unix Makefiles

当在类Unix环境下使用命令行来运行CMake默认将会生成Unix makefile。当然，你可以显示的通过-G来指定。当生成makefile，你需要定义CMAKE_BUILD_TYPE变量。假定源文件夹是父目录：

```bash
cmake -G "Unix Makefiles" -DCMAKE_BUILD_TYPE=Debug ..
```

你需要定义变量CMAKE_BUILD_TYPE是因为CMake生成makefile是单配置。不像Visual Studio解决方案，你不能使用同一个makefile来构建多个配置例如Debug和Release。单个makefile恰好能够构建一种生成类型。默认，激活的类型是Debug, MinSizeRel, RelWithDebInfo 和 Release。小心，如果你忘记定义CMAKE_BUILD_TYPE，你可能会得到一个没有调试信息的未优化版本，这是无用的。更改不同的构建类型，你必须重新运行CMake来生成新的makefile。

个人觉得，CMake的默认Release版本配置没有用处，因为不生成任何调试信息。如果你打开了一个崩溃的dump或者在Release解决了一个bug，及时你在优化的版本中，你也会欣赏调试信息的重要性。这就是为什么在我的CMake项目中，我通常从CMakeLists.txt中删除Release配置，使用RelWithDebInfo来代替。

一旦makefile存在，你可以使用make来构建你的工程。默认make会构建CMakeLists.txt定义的每个目标。在[CMakeDemo](https://github.com/preshing/CMakeDemo)例子中，只有一个目标。你也可以指定目标来进行构建：

```bash
make CMakeDemo
```

CMake生成的makefile会自动探测头文件依赖，因此编辑了一个头文件不会重新编译整个工程。你可以使用并行命令-j 4(或者更大的数字)来make。

CMake也揭露了[Ninja](https://ninja-build.org/)生成器。Ninja类似于make，但是更快。它生成一个build.ninja的文件，类似于Makefile。Ninja生成器也是单配置。Ninja的-j选项也可以自动检测活动的CPU个数。

## 使用Visual Studio构建

我们将从命令行生成一个Visual Studio .sln文件。如果你安装了多个Visual Studio版本，你希望告诉cmake需要用哪个版本。同样假设源代码文件夹是父目录：

```bash
cmake -G "Visual Studio 15 2017" ..
```

上面命令会生成一个32位Visual Studio的.sln文件。使用CMake没有多平台的.sln文件，你必须指定64-bit生成器：

```bash
cmake -G "Visual Studio 15 2017 Win64" ..
```

在Visual Studio中打开.sln文件，转到解决方案浏览页面，右键点击你需要运行的目标，然后选择设置成启动项目。像之前一样构建和运行。

<img src="./img/cmake_9.png" width="60" >

注意CMake会增加2个目标到解决方案：ALL_BUILD 和 ZERO_CHECK。ZERO_CHECK会自动检测CMakeList.txt当变更的时候自动运行。ALL_BUILD通常构建所有的目标，使其在Visual Studio中有点冗余。如果你习惯于以某种方式设置解决方案，那么在.sln文件中设置这些额外的目标可能让人感到厌烦，但是你会习惯的。CMake允许你将目标和源文件组织到文件夹中，但我不会再例子中阐述这些。

就像Visual Studio解决方案，你可以从解决方案配置下拉框随时更改构建类型。下面是例子默认的构建类型。同样，我觉得默认的Release配置没什么用是因为它不会产生任何调试信息。在我其他的CMake项目中，我通常从CMakeLists.txt删除Release配置使用RelWithDebInfo来代替它。

## 在Visual Studio 2017内置CMake

在Visual Studio 2017，微软介绍了另一种使用CMake的方式。你可以通过Visual Studio的文件->打开->文件夹菜单打开CMakeLists.txt。这个新的方法避免了创建.sln和.vcxproj文件。它同样支持在同样的工作区构建32位和64位。那看起来很不错，在我看来，由于一些原因不适合：

- 如果在包含CMakeLists.txt文件的源文件目录外有一些源文件，在解决方案资源管理器就找不到
- C/C++属性页面不再显示
- 缓存变量只能通过编辑JSON文件来修改，相比Visual IDE不那么直观

我不是一个真粉。现在，我还是倾向于使用CMake手动生成.sln文件。

## 使用xcode构建

CMake网站发布了一个支持MacOS的.dmg文件。.dmg文件包含一个app，你可以拖拽到应用程序文件夹。确保你这样安装了cmake，cmake在你没有创建/Applications/CMake.app/Contents/bin/cmake链接，可能没法正常激活。我习惯于从[MacPorts](https://www.macports.org/)安装因为它会替你设置命令行，就像SDL2同样可以被安装依赖。

从CMake命令行指定xcode。同样，假设源文件路径是父目录：

```bash
cmake -G "Xcode" ..
```

这将会创建一个.xcodeproj文件夹。用xcode打开，在xcode工具条，点击"激活计划"下拉框选择你要运行的目标。

<img src="./img/cmake_10.png" width="60" >

然后点击"编辑计划"从同样的下拉框，在Run->Info下选择一个构建配置。同样不推荐使用Release配置，缺少调试信息会减少作用。

<img src="./img/cmake_10.png" width="60" >

最后，使用产品->构建按钮。可以让CMake生成xcode项目构建MacOS自用或框架。

## 使用Qt Creator构建

Qt Creator提供了内置的CMake支持，使用Makefile或者Ninja生成器。我使用Qt Creator 3.5.1测试了下面的步骤：

在Qt Creator，使用文件->打开文件或项目...然后选择CMakeLists.txt你可以构建。

<img src="./img/cmake_11.png" width="60" >

Qt Creator快速打开二进制文件夹位置，称作"构建目录"。默认，它建议是源文件夹相邻的位置。你可以按你的想法更改。

<img src="./img/cmake_12.png" width="60" >

当提示重新运行CMake，当Makefile生成器是单配置确保CMAKE_BUILD_TYPE已经定义。你也可以指定项目指定变量，就像例子中的DEMO_ENABLE_MULTISAMPLE选项。

<img src="./img/cmake_13.png" width="60" >

最后，你可以通过Qt Creator的菜单或者使用Ctrl+Shift+B快捷键构建和运行项目。

如果你想重新运行CMake，例如你想修改构建类型从Debug到RelWithDebInfo，转到Projects->Build & Run->Build，然后点击 “Run CMake”。

<img src="./img/cmake_14.png" width="60" >

CMakeDemo项目包含了单个可执行目标，但是如果你的项目包含了多个可执行目标，你可以告诉Qt Creator运行哪个，通过Projects -> Build & Run -> Run然后更改"Run configuration"。下拉框会从构建管道的可执行目标自动构成。

<img src="./img/cmake_15.png" width="60" >

## 其他CMake特性

- 你可以从命令行执行一个构建，不管生成器是否使用：cmake --build . --target CMakeDemo --config Debug
- 你可以通过CMAKE_TOOLCHAIN_FILE变量来创建一个给其他环境交叉编译的构建管道
- 你可以生成一个compile_commands.json文件供给Clang的[LibTooling](https://clang.llvm.org/docs/LibTooling.html)库。

我非常欣赏CMake帮助整合各种各样C/C++组件和编译它们到所有的环境中。它也不是没有缺点，但是一旦你熟练掌握了它，开源世界就是你的牡蛎，即使在集成非CMake项目中也是如此。我的下一篇文章将是一个速成课程，讲的是CMake脚本语言。

如果你想成为一个超级用户，而又不介意花几块钱，作者的大作[Mastering CMake](https://www.amazon.com/gp/product/1930934319?ie=UTF8&tag=preshonprogr-20&camp=1789&linkCode=xm2&creativeASIN=1930934319)是一个不错的选择。那些文章[The Architecture of Open Source Applications](http://aosabook.org/en/cmake.html)读起来也很有趣。
