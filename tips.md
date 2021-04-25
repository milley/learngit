mingw64链接失败:

```cmd
$ g++ -g -Wall -I../../leveldb-mingw-master/include ./simple_main.cc ./lib/libLeveldb.a -o simple_main
./lib/libLeveldb.a(env_win.o): In function `leveldb::Win32::Win32MapFile::_Init(wchar_t const*)':
E:\gh_source\leveldb-mingw-master\build/../util/env_win.cc:624: undefined reference to `__imp_PathFileExistsW'
./lib/libLeveldb.a(env_win.o): In function `leveldb::Win32::Win32Env::FileExists(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&)':
E:\gh_source\leveldb-mingw-master\build/../util/env_win.cc:764: undefined reference to `__imp_PathFileExistsW'
collect2.exe: error: ld returned 1 exit status
```

查API需要包含#include "Shlwapi.h"头文件，再通过[stackoverflow](https://stackoverflow.com/questions/48882332/c-undefined-reference-to-pathfileexistsw-imp-pathfileexistsw4)描述得知需要链接-lshlwapi，因此修改编译命令解决。

```cmd
g++ -g -Wall -I../../leveldb-mingw-master/include ./simple_main.cc ./lib/libLeveldb.a -o simple_main -lshlwapi
```
