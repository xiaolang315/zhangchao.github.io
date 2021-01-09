# C++ Modules 
Modules 被认为是C++20的BigFour特性（Modules，Concept，Ranges Library，Coroutines）之一，是C++逐步剥离历史包袱，向现代语言进化的重要历程。目前三大编译器厂商还没有完全支持Modules，本文希望通过对标准提案的一些分析，让对这一特性感兴趣的朋友能有个初步了解。本文不会详细的介绍Modules所有的语法细节，需要的朋友可以从文末的参考文献中查阅，笔者会结合自己的工程经历谈谈对modules这个新的特性的思考，并给出警示从而避免一些犯错的可能。

## Modules 总览
早在2017年Modules就作为标准的提案呼声就很高，但最终并未被采纳，也许是当时的技术还不够成熟，C++20的采纳可以说是众望所归。任何技术的引入都不是“银弹”，它在解决了众多困扰C++程序员良久的问题同时，仍然遗留了一些工作并产生一些新的误区。以下将从这两方面来阐述：
### Modules 带来的改善
#### 1. 缩短了编译时间: 
  C++的构建速度被诟病多年，其中很大一部分原因是`#include`头文件的预处理方式，导致头文件在各个编译单元中重复展开，重复编译。引入Modules后，放弃原来的文本替换的方式，每个Module都将以构建产物的形式得以复用，减少了大量的重复编译过程。
#### 2. 加强了可见性的控制: 
  之前的C++版本中，默认符号是全部可见的（虽然各厂商定义了自己的控制可见性方式），只要包含了头文件，那么头文件中声明的所有信息，都是可以看到并使用的。Module可以通过新增的export关键字，来控制自己接口文件中的符号可见性以及使用的方式。
#### 3. 改善了宏定义的脆弱性：
  一直以来宏定义冲突都是C++程序员非常头疼的问题，头文件中的宏定义会与被包含在同一编译单元的其他头文件中的定义重复或冲突，也可能影响到编译单元中实现中的定义，这种问题的出现往往都是非常难排查的。Modules重新定义了Import的关键字代替了include，也赋予了它更严格的约束，使宏定义更容易被约束和控制。
#### 4. 提高了工具可理解性: 
  使用`#include`的方式，对代码分析工具来说可以说是一场噩梦，包含进编译单元的头文件按照怎样的顺序发挥作用，头文件中哪些是被客户真正使用的完全依赖应用的上下文，这些都导致工具想通过源码完成依赖分析非常困难。引入Modules后，依赖关系将更清晰，更容易被工具化。

### Modules 尚未解决的问题

#### 1. 不可避免的代码重写：
  切换成modules需要大量的重写和改造工作，替换include为import也会引入大量的用户代码变更。一方面标准库要大量改造，一方面自定义的库也需要适配工作。
  
#### 2. 模块名与版本信息：
  module的命名方式支持语义化版本的描述（例如a.b.c），但模块名仅作为导入的关键字，标准不支持模块名的语义逻辑和层次化关系提取。后文会对这个问题详细阐述。
  
#### 3. 是否消除了命名空间：
  模块和命名空间仍然是两个独立的概念，模块并不具备命名空间的作用，因此符号名重复的问题仍然需要依赖命名空间进行隔离。从一处细节可以看出，就是可以export 一个namespace。

#### 4. 仍然存在的跨平台问题：
  Module的复用不是跨平台的，不同平台和CPU架构下的构建产物，包括编译器版本等差异，都可能引起Module的不可用，因此Modules的复用仍然推荐在相同构建环境下。
  
## Modules 新增概念
### 新的单元（Unit）
* Module Unit: 是包含Module声明或定义的编译单元；
* Global Module : 全局模块片段（Global-Module-Fragments）和其他非Module的编译单元的统称；
* Header Unit：不包含模块声明的头文件，这类文件被导入时，默认可以看到所有内容，这些内容属于Global Module。

熟悉的读者已经可以看出标准的意图，之前的头文件和实现文件被重新进行了划分，除了属于Module的，剩下的部分归属到Header和Global Module里。

个人感觉20标准推荐的最佳实践，应该是宏定义留在Header Unit中，其他的都整合到Modules中，暂时没重构的都在Global module中。

### 可达性
对比之前的头文件，现在可以约束客户的使用方式，让使用者看到struct，并不能直接使用struct，只能通过有限的访问接口进行访问。这就不得不提到新引入的可达性的概念。

``` cpp

//module A TU1
export module A;
struct X {};
export X getX() {
  return X();
}

//module B TU2
module B;
import A;
X x; // compile error， X not visable
auto x = getX(); // OK, definition of X is reachable
X x = getX(); // compile error， X construct is not visable
```
Instantiation context 的概念是跟可达性相关的，所谓实例化上下文是顺着import去找的，如果import链上找不到类型定义，就无法直接使用这个类型的定义，就不属于Instantiation context 。在上下文中的内容均属于可达内容。

仍然有一种绕过可见性，通过可达性，达成对可达但不可见的struct的hack，就是利用auto和decltype。为了避免教唆嫌疑，此处不列举详细代码。此坑请大家知晓，在设计时遵循最小可见性原则，设计自己的接口。

## Modules 语法

### module 声明
module声明格式如下，描述方式沿用了标准里的语法，module的权限范围是整个module的文件。
```
module-declaration:
  exportopt module module-name module-partitionopt attribute-specifier-seqopt ;
module-name:
  module-name-qualifieropt identifier
module-partition:
  : module-name-qualifieropt identifier
module-name-qualifier:
  identifier .
  module-name-qualifier identifier .
```
Module Units 分类：
* Interface
* Implementation
* Partition

既然Modules是为了消除头文件，为什么还要分Interface Unit和Implementation Unit ？我们从两方面来看待这个问题：
* 从编码习惯的角度，更符合C++程序员一直以来的开发习惯。
* 从依赖管理的角度，如果单文件的module会引入更多Moudle内部实现方面的内部依赖，对整体的构建速度会有影响，也更容易产生循环依赖的问题。

需要额外注意的是Module Partition里并不会隐式的import主Module的Interface的内容。

标准中这个例子还是挺全面的出现了绝大多数应用场景。
``` cpp
// TU 1
export module A;
export import :Foo;
export int baz();
// TU 2
export module A:Foo;
import :Internals;
export int foo() { return 2 * (bar() + 1); }
// TU 3
module A:Internals;
int bar();
// TU 4
module A;
import :Internals;
int bar() { return baz() - 10; }
int baz() { return 30; }
```
* TU1是Module Interface 
* TU4是Module Implementation，
* TU2的Foo和TU3的Internals都是 A的module partition，差别在于Foo使用了`export import :Foo`开放了这个内部模块，
* TU1中module A选择性的开放了baz，而bar是作为内部使用的，内部声明 在TU3中。

#### private && global module fragment
Private module fragment：采用`module : private ;`定义，只定义在module interface里，所有export的最后，
Global module fragment：采用 `module;`定义，作用范围也只在unit内部，

``` cpp
module;
#include "some.h"
//模块内没有用到的东西，当作不存在
module M;

module :private;
//对模块外不可见
```
### export 

export用来控制导出哪些符号，语法比较简单，标准中有详细的例子和介绍，export 后跟着的是需要导出的内容，下面列举一些使用中的注意事项：
* 宏定义不能export
* private，匿名命名空间 都不能export
* export必须有内容，有名字的命名空间也算。
* implementation 不能包含export，

此外还可以采用group的方式来写，以下是一些简单的例子:
``` cpp
export  {
    module math;
    int add(int x, int y);
}
struct X {

};
export using Y = X;
export typedef X Z;
```

### import
import用来控制导入哪些模块，导入的内容是依赖模块中export的内容。import只能是以module为单位，不能由引入者选择导入哪些符号（`import xx-func from x-module`），只能全面的import所属模块开放的全部内容；也不能提供别名机制（`import as`）。标准中除了详细的语法描述外，还有以下一些约束需要格外注意：
* import语句，标准建议在正文的最前面书写，方便编译器实现。
* 不能import自己。
* import也有循环依赖的风险。
* import后的初始化顺序问题，要求编译器做到顺序无关。

#### import 头文件
C++20以后推荐使用import来引入头文件(Header Unit)，使用新的import后，不再沿用原来直接文本展开的预处理形式，对宏定义给出了`active`的动作，而 `active`语义后，遵循ODR（one define rule）宏的不一致定义将被识别为一种错误。


引入Modules之前，编译单元TU都是相互独立的，头文件都是文本拷贝的方式进入到TU内的，所以之前的标准ODR定义要求在一个TU内唯一就可以。现在编译阶段import了其他的module，那么ODR的原则就被扩展到所有可达的TU内，要求可达路径内没有重复的定义。


### export 与 import 连用

export和import在其他语言中也有类似的用法，但C++20还支持一种相对怪异的用法，在上文中也出现过，关于它的用途希望强调说明以下。
``` cpp
// TU 1
export module FooWrapper;
export import Foo;
// TU2
export module Foo;
export import :doo;
```
* TU1的方式可以用来组合和修饰一个或多个现有的module，
* TU2的可以选择性的开放模块内部复用的子模块。
是不是想到了设计模式中的应用，没错export和import也是一种通用的抽象接口，在软件模块组织方面也可以通过组合和修饰的方式降低开发过程中的复杂度。

## 标准实现情况

### 关于标准内容
从gcc和clang的官方文档可以看出，支持Modules需要实现如下的标准：
* P1103R3 (Merging Modules)
* P1703R1 (Recognizing Header Unit Imports Requires Full Preprocessing)
* P1766R1 (Mitigating minor modules maladies); DR for the changes therein for
default arguments and classes having typedef names for linkage purposes.
* P1779R3 (ABI isolation for member functions)
* P1811R0 (Relaxing redefinition restrictions for re-exportation robustness)
* P1874R1 (Dynamic Initialization Order of Non-Local Variables in Modules)
* P1979R0 (Resolution to US086)
* P2115R0 (US069: Merging of multiple definitions for unnamed unscoped enumerations)

以下2个标准网页没法打开
* P1815R2 (Translation-unit-local entities)
* P1857R3 (Modules Dependency Discovery)

目前只有clang11和MSVC宣布部分支持了，gcc还没有支持，目前还是在分支上，构建工具方面也只看到xmake宣布试验性支持，主流的cmake等都还未公布相关的消息。

### Clang
clang对modules的实现方式，采用的是预编头文件的方式，下面是一个clang编译并应用modules的构建过程示例：
``` shell
clang++ -std=c++2a -fmodules-ts --precompile math.cppm -o math.pcm
clang++ -std=c++2a -fmodules-ts -c math.pcm -o math.o
clang++ -std=c++2a -fmodules-ts -fprebuilt-module-path=. math.o main.cpp -o math
```
在构建Module时，先预编译产生可被别人依赖的二进制（以pcm后缀命名），再构建出被链接的二进制（.o），对使用者来说，注意此处的文件名结合prebuilt-module-path要能准确的找到所依赖的Module的pcm和.o才可以正常通过编译链接。

### MSVC
MSVC在2017上已经据说可以支持一些Modules，在2019上更是增强了对Modules的支持，从2019的官方文档中看到Modules的介绍和简单使用，但整体上相关的文档还比较少。微软已经把一部分标准库做了modules，可以import，并没有像clang和gcc那样对具体标准的实现进展进行特别详细的跟踪说明。

### Gcc
从2017年就拉出了分支在做尝试，至今还没有合入主分支，所以想要实现需要还不支持的特性：
* Order of initialization, Belfast '19, US82, p1874
* Translation-Unit Local entities, Prague '20, p1815 (includes p2003)
* ABI Isolation, Prague '20, p1779
* ADL implementation bugs discovered by FR39
* Partition definition visibility rules
* Private module fragment
* All STL headers as header units

Gcc 官网有 Modules专门对wiki，里面有比较多详细对知识和概念解读，相信会对想了解更深入Modules有很大的帮助。

## 关于 C++ Modules 的一些思考
### 编译加速效果分析
原本打算做个实验对比使用和不使用Modules的编译速度差别，但通过clang的例子我们可以看到，本质上还是采用了预编译头文件的方式，所以并没有新的编译技术方面的诞生，最重要的还是根据可复用性进行的划分。由于不能在原有头文件中直接控制可见性，没法通过少量的改造进行直接对比，如果一个模块化设计一个非模块化设计，那么不是机制的问题，是设计上的问题，因此实验的事情只好作罢。

本质的改变在于引入Modules机制后，较强的约束导致可复用的编译单元被强制的分割开了，这样为使用者的灵活选择提供了可能性，在这样的基础上再辅以成熟的预编译头文件和未来编译器内置的缓存机制，才真正把重复的编译过程彻底消除，编译速度最大化。

如果头文件原本就组织的比较独立，再辅以预编译头文件的技术，加上ccache这样的外部工具，那么整体的编译速度提升应该不会特别大。对大多数项目来说，最难的都是“合理划分”这一步。

### C++ Modules 设计上的不足

当看到如下代码时，你一定会有些误会
``` cpp 
import math.add
```
add是math的子模块吗？抱歉，它只是一个模块，甚至可能跟math一点关系都没有！math.add只是Module的名字，这个`.`命名方式的支持，可以说是语义化的妥协，但又是语义上的缺失。
``` shell
clang++ -std=c++2a -fmodules-ts --precompile math.cppm -o math.add.pcm
clang++ -std=c++2a -fmodules-ts -c math.add.pcm -o math.o
```
如果想利用这样的命名方式来表达层次化关系，例如模块a包含a.b和a.c，引入a则表示a.b和a.c同时被引入，标准和编译器是无法提高帮助，这就需要自己组织Module之间的关系，在非底层的Module最好使用纯包装的Module, 在里面export 所有的下层模块，这样`import a`的效果才是`import` a下的所有包，同时其他模块也可以单独引用a.b或a.c。

由于模块和文件名的严格对应关系，未来这种层次化管理和变化跟踪最好由工具和最佳实践来支持，靠程序员管理太过复杂。

### 切换到Modules
在文章的开头我们提到过Modules并不能解决大规模的软件改造工作，但编译器厂商已经在做一些努力，比如Module maps 是clang做的从头文件转到Modules的映射文件，里面没有具体业务实现，使用Module Map Language语法，这种实践比较适合存在巨大切换成本的遗留系统。

如果是不借助工具的手动切换，可以预想到一个项目的切换过程应该是这样：
1. 找到可复用的单元，逐步提取出到新的Module文件中，export相应的接口。
2. 原有的代码需要用到Module的，import 新的Module。
3. 重复1，2直至所有可复用的单元都提炼完成

### 从整个软件生命周期看，Modules的引入带来了怎样的变化
* 开发：更好的做到可见性的管理，不再依赖各个厂商的差异处理，闭包性增强。更容易独立开发
* 构建：以Modules为单位构建，依赖管理更重要，相对之前有更容易工具化的依赖关系分析，二进制构建产物复用，构建更快。
* 测试：相较原来设计良好的模块，测试方面没有明显改善。
* 发布：以Modules为单位发布，但标准和官方没有独立的中央仓和标准，也没有相应的版本号等策略。
* 部署和升级：Modules更多作为构建复用，对部署和升级没有明显优化的支持。

### 展望和担忧
C++20的标准会在今年完成正式稿的发布，Modules特性是一次变革性的尝试，带来巨大优势的同时，也引入了相应的复杂度和转换成本，对于新项目来说有了一个比较良好的开端，但对于既有系统是否全面切换到Modules，还要看对项目的收益和能投入的成本是否匹配。在组件化设计思维日益成熟的当下，C++作出了顺势而为的改变，反观编译器大厂的跟进程度，这项新特性恐怕还需要1-2年的时间，待到相关工具和生态都相对成熟后，才能发挥它的价值。

## 参考文献
* [cpp modules](https://en.cppreference.com/w/cpp/language/modules)
* [modules clang 实验](https://modernescpp.com/index.php/c-20-modules)
* [modules 分析使用](https://vector-of-bool.github.io/2019/03/10/modules-1.html)
* [gcc modules的WIKI](https://gcc.gnu.org/wiki/cxx-modules)
* [msvc modules](https://docs.microsoft.com/en-us/cpp/cpp/modules-cpp?view=vs-2019)
* [clang modules](https://clang.llvm.org/docs/Modules.html)
* [msvc c++20标准实现情况
](https://docs.microsoft.com/en-us/cpp/overview/visual-cpp-language-conformance?view=vs-2019)
* [gcc cxx2a 标准实现情况](https://gcc.gnu.org/projects/cxx-status.html#cxx2a)
* [clang cxx2a 标准实现情况](http://clang.llvm.org/cxx_status.html)
