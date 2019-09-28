# cmake 高级技巧

网上很多关于cmake的基本介绍很多，本文关注几个比较有用的cmake命令让你在使用cmake的时候能游刃有余，如鱼得水。我们常常会看到很多cmake工程还提供了shell脚本，bat脚本，去运行可执行文件，甚至负责传递参数给cmake，岂不知cmake本身就是为了跨平台而生的产物，如此一来不同操作系统下脚本的维护反倒成了负担。让我们真正把cmake用起来，不仅仅用来生成makefile，甚至可以用来代替脚本语言，完成工程相关的所有管理事务。

## function与macro
区别在于function有单独的作用域，在function中定义的变量，离开后就没有用。而macro中就像c语言中的一样，完全的展开，因此可以用来定义顶层的变量。

## assert
cmake 没有提供assert方法，这对程序员来说，太坑爹了。补充自定义assert做一些生成makefile期的检查,生生又给c++的构建加了个步骤。

```cmake
function(assert is_ok dscrption)
  if(NOT ${is_ok})
    message(FATAL_ERROR "${dscrption}")
  endif()
endfunction()
```

## try_compile
尝试进行一些编译，编译结果返回于一个局部bool变量，那它又什么用呢？我们可以把一些头文件包含于check.c中，在编译正式工程的时候，先编译check.c，里可以加入一些静态检查，这样可以保证一些宏定义是否满足要求。

```cmake
try_compile(IS_OK ${CMAKE_CURRENT_SOURCE_DIR}/tmp ${CMAKE_CURRENT_SOURCE_DIR}/hello1.c)
assert(IS_OK "compile failed")
```
## 全局变量修改
cmake是以文件为作用域的，在subdir的CMakeLists里，如果要修改上一级里定义的变量，可以通过如下方式，否则即相当于新定义了一个_time变量。

```cmake
 set(${_time} "1:2:3" PARENT_SCOPE)
```
# cmake 工具
cotire是开源的编译加速工具，用cmake写的，帮助添加预编译头文件，和组织UnitySource。项目地址https://github.com/sakra/cotire

# cmake 设计
## 不同系统编译
### 编译命令不同
在cmake生成的目录下，使用如下命令，屏蔽不同工程编译方法不同，造成的分歧。

```shell
cmake --build .
```

### 可执行文件路径不同
使用如下命令可以指定可执行文件位置，以保障不同系统下可以使用相同的cmakelist

``` cmake
set(EXECUTABLE_OUTPUT_PATH "your output path")
```

### 订制target
为了使可执行文件路径和外部的脚本无关，可以订制一个命令

```cmake
add_custom_target(
    exe
    COMMAND ./output/your-exe
    DEPENDS your-exe
    )
```
使用命令，运行可执行文件

``` shell
cmake exe

```


## 文件操作 
常见的文件操作都有

```cmake 
    file(WRITE filename "message to write"... )
    file(APPEND filename "message to write"... )
    file(READ filename variable [LIMIT numBytes] [OFFSET offset] [HEX])
    file(STRINGS filename variable [LIMIT_COUNT num]
        [LIMIT_INPUT numBytes] [LIMIT_OUTPUT numBytes]
        [LENGTH_MINIMUM numBytes] [LENGTH_MAXIMUM numBytes]
        [NEWLINE_CONSUME] [REGEX regex]
        [NO_HEX_CONVERSION])
    file(GLOB variable [RELATIVE path] [globbing expressions]...)
    file(GLOB_RECURSE variable [RELATIVE path] 
        [FOLLOW_SYMLINKS] [globbing expressions]...)
    file(RENAME <oldname> <newname>)
    file(REMOVE [file1 ...])
    file(REMOVE_RECURSE [file1 ...])
    file(MAKE_DIRECTORY [directory1 directory2 ...])
    file(RELATIVE_PATH variable directory file)
    file(TO_CMAKE_PATH path result)
    file(TO_NATIVE_PATH path result)
    file(DOWNLOAD url file [TIMEOUT timeout] [STATUS status] [LOG log]
        [EXPECTED_MD5 sum] [SHOW_PROGRESS])
```
举个下载文件的例子
```cmake
file(DOWNLOAD "http://10.93.73.27:8000/README.md" "./readme")
```
可以用来管理库的依赖就靠它了

## 执行shell命令
执行shell命令除了设计一个target外，还可以在cmake里直接执行。一个简单的例子

``` cmake

execute_process(COMMAND "ls")

```

结果就不贴了，虽然看似简单的命令，实则将cmake和整个系统联系起来，为作为程序员的你打开一扇大门。

## 从构建文件集合中删除特定文件

通过以下命令将testing.cpp排除在构建之外，一般用在少量排他的特殊场景。
```cmake 
list(REMOVE_ITEM sources ${CMAKE_CURRENT_SOURCE_DIR}/testing.cpp)
```
