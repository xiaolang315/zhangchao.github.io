# Linux-eclipse开发提效

***
<center>
<font size=6>
时间是当今社会唯一公平的资源
低效的人已经在起跑线上落后
</font>
</center>
*** 
### 一般程序员

==鼠标复制粘贴流==

磨损按键

> ctrl+c, ctrl+v ,鼠标

***
### 高级程序员

==基本抛弃鼠标，玩转控制键盘区==。核心理念是用键盘代替鼠标控制光标的位置，避免打字过程中频繁将手移出键盘区。

磨损按键

> home， end， ctrl, shift, 方向键 ， page up/down 

***
### 骨灰程序员

==选择自己的阵营（emacs，vim等）并熟练使用快捷键，放弃控制键盘区==。追求极致编程效率，觉得把手移到控制键盘区，也是浪费时间。主流的几个阵营，都是长久以来比较受程序员欢迎的几个编辑器所产生的。进而演化到IDE也会支持类似编辑器的操作方式。

磨损按键
> all
***

###对比下相关操作

|操作|vim|emacs|高级|初级|
|-----|---|---|---|---|
|向左移动一个字符|h|ctrl+b|left|鼠标点一下|
|向右移动一个字符|l|ctrl+f|right|鼠标点一下|
|向左移动一个单词|b|alt+b|ctrl+left|鼠标点一下|
|向右移动一个单词|w/e|alt+f|ctrl+right|鼠标点一下|
|向上移动光标|j|ctrl+p|up|鼠标点一下|
|向下移动光标|k|ctrl+n|down|鼠标点一下|
|选择|v+移动光标|ctrl+space后移动光标|shift+移动光标|鼠标选择|
|复制|y|ctrl+y|ctrl+c|ctrl+c||
|粘贴|p|alt+y|ctrl+v|ctrl+v||
|跳转指定行|:+行号|ctrl+l|依赖IDE|鼠标点一下|
|跳转行头|0|ctrl+a|home|鼠标点一下|
|跳转行尾|$|ctrl+e|end|鼠标点一下|
|删除行|dd| ctrl+a ctrl+k| home,shift+end,delete|鼠标选择后删除|

***
<center>
<font size=6>
 Eclipse快捷键
</font>
</center>
***
### 浏览相关
| 按键   | 描述|
|--------|--------|
| ctrl+h |全局内容搜索/替换|
| ctrl+shift+r |文件全局搜索|
| ctrl+shift+t |符号全局搜索|
| ctrl+f |文件内搜索替换|
| f3 |跳转到定义/实现|
| ctrl +g |搜索声明|
| ctrl+tab |头文件和实现文件之间切换|
| f4 |继承关系|
| ctrl+e |在打开文件中搜索，范围比ctrl+shift+r小，快些|
| ctrl+f6 |之前打开的文件|

***
### 浏览相关-进阶
| 按键   | 描述|
|--------|--------|
| ctrl+j| 增量查找,|
| ctrl+k |快速匹配下一个|
| ctrl+t |快速查看继承关系, 这种可以不依赖鼠标跳到子类|
| ctrl+shift+g |查找所有调用|
| ctrl+shift+h |打开类型搜索|
| ctrl+alt+h |查找调用栈|
| ctrl+alt+i |查找头文件包含链|
| ctrl+o |在文件中快速索引函数，变量等|
| ctrl + =|自动宏展开提示|
| f2 |自动宏展开提示|

一般查找类的快捷键，加shift是反向的操作，比如ctrl+k 是向下匹配第一个，ctrl+shift+k就是想上匹配

***
### 编辑相关-基础

| 按键   | 描述|
|--------|--------|
| ctrl+shift+n | 添加光标所在位置的头文件，|
| alt+shift+r|  重命名|
| alt+up/down|  交换临近行内容|
| ctrl+d | 删除当前行|
| ctrl+/ | 注释所选内容/取消注释，没有选择注释当前行|
***

### 编辑相关-进阶

| 按键   | 描述|
|--------|--------|
|  ctrl+shift+o       |      整理整个文件的头文件（慎用）  |
|  alt+/       |      补全和提示，忽略大小写，根据包含的头文件给出， 不必非等到.或->才提示，提前用让ide帮你敲更多代码  |
|ctrl+i |根据format调整所选的格式， ctrl+shift+f 调整整个文件的|
|ctrl+q |回到上次编辑的地方。|
|ctrl+shift+x |所选内容变大写|
|ctrl+shift+y |所选内容变小写|
|ctrl+shift+up/down |上下复制一行|
|alt+shift+j |合并行|
|alt+shift+a |进入和退出列编辑模式|

***
### 快捷键风格
eclipse可以配置快捷键风格，支持emacs，vs，
`Window -> Preferences -> General -> Keys`
想使用vim的需要安装个插件(vapper)。

### 模板
善用编辑模板，常用的cn（生成类命），ns（命名空间包含），role（以本文件名生成define_role)等，别在没有智力的代码上浪费时间。
`Window -> Preferences -> C/C++ -> Editor -> Templates ` 

***
### 调试相关
| 按键   | 描述|
|--------|--------|
| f11|开始调试 |
| f5|单步运行，遇到函数调用，进入被调用函数|
| f6|单步运行，只在本函数内逐句运行;|
| f7|跳出当前函数，回到调用栈上级|
| f8|运行到下一处断点|
| ctrl+r|运行到光标所在位置|
| ctrl+f11|直接运行不调试|
| ctrl+f2|终止调试|
| ctrl+shift+b|设置或取消断点|

学会用表达式视窗，来订阅你想观测的内容，这里可以完成一些计算。
学会在查看内存和添加内存断点来定位内存被踩的问题。

***
### 工程相关
#### Project
工程相关的文件有：`.cproject` and `.project`
* cmake生成工程，免配置，但不方便编辑
* 自建工程，使用cmake生成的makefile脚本，便于共享
* `Project -> Properties -> Resource`  可以修改编解码方式
* 文件移动最好使用project视图中的move，ide帮助你调整头文件


***
#### 索引
影响跳转，语法检查，头文件自动包含等功能
`Project -> Properties -> C/C++ General -> Paths and Symbols`
主要配置如下内容，关注c和c++有区分
* 宏定义
* 头文件搜索路径
***
#### 编译 
配置好后可以通过ctrl+b进行IDE内编译
`Project -> Properties -> C/C++ Build ` 
`Build Variables` 里设置的变量可以给上文中的`Paths and Symbols`使用。
可指定编译目录，和编译命令
***

#### 运行
`Project -> Properties -> Run/Debug Setting ` 
* 可指定可执行程序的位置，及运行参数
* 可配置多个可执行程序机器配到参数，对比运行调试
***

#### 格式
`Project -> Properties -> C/C++ General -> Formater `
可以编辑，导入导出团队统一的格式化模板，为`ctrl+i`和`ctrl+shift+f`使用
***
### IDE相关

|按键| 描述|
|-----|--------|
|alt+enter |当前页面的首选项|
|ctrl+3 |搜索快速打开配置中的内容或近期文件，|
|ctrl+. |在当前文件中跳到下个错误（红线识别的位置） ctrl+, 上一个|
|ctrl+1 |快速重命名，支持文件中重命名和工程中重命名，比起alt+shift+r好记些|
|ctrl++和ctrl+- |来快速调整字体大小，neao版本后加入的|
|ctrl+shift+l |快捷键帮助，可以根据记忆碎片和功能的英文名来搜索快捷键|
|ctrl+b |编译,配置好工程情况下|
|ctrl+f7 |切换idle工作区（editor和project等）|
|ctrl+f8 |切换调试和编辑模式|
|ctrl+m |切换全屏|

***
## Linux小工具

grep 简单应用，递归“查找路径”下文件中包含“搜索内容”的文件名和行号
常用场景，在eclipse工程中找不到符号，需要找到符号所在的头文件，或cpp时。

> grep -Frn [搜索内容] [查找路径]

find 简单应用 查找“搜索路径”下文件名包含“文件匹配”字符串的文件路径。
常用场景，在创建eclipse工程时，引入的现有文件保含的头文件不在eclipse的查找路径中。
> find [搜索路径] -name [文件匹配]

***
## shell快捷键
在shell编辑过程中操作光标的方法类似emacs，有少许不同请根据不同终端探索。

| 按键   | 描述|
|--------|--------|
| ctrl+shift+t 	| 创建标签|
|alt+数字  		|在标签之间切换|
|cd + - |回到上次的目录，类似pushd/popd， 区别是cd+ -只能回退一次，pushd/popd栈可以比较深|
| ctrl + u	|删除此处至开始的所有内容|
| ctrl + k	|删除此处至末尾的所有内容|
| ctrl + w|	删除此处到左边的单词|
|     ctrl + y|	粘贴由ctrl+u， ctrl+d， ctrl+w删除的单词|
|     ctrl + l|	相当于clear，即清屏|
|     ctrl + &|	恢复 ctrl+h 或者 ctrl+d 或者 ctrl+w 删除的内容|

***

Q&A
