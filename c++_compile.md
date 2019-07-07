<meta http-equiv="content-type" content="text/html; charset=UTF-8">
<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#orgheadline5">1. c++编译过程</a>
<ul>
<li><a href="#orgheadline1">1.1. 预编译</a></li>
<li><a href="#orgheadline2">1.2. 编译</a></li>
<li><a href="#orgheadline3">1.3. 汇编</a></li>
<li><a href="#orgheadline4">1.4. 链接</a></li>
</ul>
</li>
<li><a href="#orgheadline20">2. 编译优化</a>
<ul>
<li><a href="#orgheadline6">2.1. 目的</a></li>
<li><a href="#orgheadline19">2.2. 编译时间优化</a>
<ul>
<li><a href="#orgheadline7">2.2.1. 代码优化</a></li>
<li><a href="#orgheadline14">2.2.2. 工程优化</a></li>
<li><a href="#orgheadline15">2.2.3. 编译优化实验</a></li>
<li><a href="#orgheadline16">2.2.4. 实验分析</a></li>
<li><a href="#orgheadline17">2.2.5. 组合使用</a></li>
<li><a href="#orgheadline18">2.2.6. 总结</a></li>
</ul>
</li>
</ul>
</li>
</ul>
</div>
</div>

c++作为c语言的继承者，属于静态语言，随着范型编程和面向对象等范式的引入，编译时间长一直被人诟病。特别是在大型项目中，遵循良好的设计原则，项目文件自然是小文件。而往往项目遗留代码都是几千行的大文件。对于重构项目，要保持接口的统一，势必要接受外围的巨大头文件，重构项目中使用大量模版元编程，这样一来编译时间的增加问题变得更为凸显。 本文主要对近期尝试的一些优化编译时间的相关技术。

# c++编译过程<a id="orgheadline5"></a>

想要优化编译，必须了解编译过程。c++编译过程主要分为：预编译，编译，汇编，链接几个过程，

## 预编译<a id="orgheadline1"></a>

主要预处理（宏定义的替换，＃include头文件展开等），该环节的主要目标是生存可编译的独立单元，一般是\*.i。

## 编译<a id="orgheadline2"></a>

就是根据预处理生成的独立编译单元，先编译为汇编语言\*.s, 模版的推导也在该阶段完成

## 汇编<a id="orgheadline3"></a>

通过汇编语言编译生成的机器码\*.o

## 链接<a id="orgheadline4"></a>

过程主要是链接符号，生成可执行文件或者库文件。

# 编译优化<a id="orgheadline20"></a>

## 目的<a id="orgheadline6"></a>

时下敏捷开发盛行，ci都有普遍的应用，理论上产品的编译和集成都应该在ci上完成，ci往往使用较高性能的机器。那是否就不需要关注编译的性能？ 非也，要知道ci的执行时机是代码提交的时候，而在代码提交之前程序员已经完成了数以百次的本地编译，看似提升的一点点时间，在日积月累之下也是非常客观的。 此外，就算ci性能再好，当项目规模巨大时，代码的commit次数非常之多，敏捷力求快速反馈，哪怕1s的提升对整个cycle的提升也是巨大的。时间是性能资源的一个外在 表现形式，我们能有效的缩短编译时间，对服务器资源的占用就会更小，从而让服务器的资源得到更好的应用。
 \*\* 编译时间消耗
编译过程中链接阶段的时间比较稳定和符号数相关，可优化的空间也不是很大。时间方面的优化工作，主要考虑减少预处理和编译的时间。而编译和预编译的时间浪费在哪里？
头文件展开和头文件解析！下面我们来设计几个实验：
基础实验：项目由1000个cpp，1000个.h组成, 每个cpp仅包含自己对应序号的.h。

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />

<col  class="org-left" />

<col  class="org-left" />

<col  class="org-right" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left">&#xa0;</th>
<th scope="col" class="org-left">每个头文件包含结构体个数</th>
<th scope="col" class="org-left">每个cpp包含头文件个数</th>
<th scope="col" class="org-left">big.h</th>
<th scope="col" class="org-right">编译时间(s)</th>
</tr>
</thead>

<tbody>
<tr>
<td class="org-left">base</td>
<td class="org-left">一个结构体</td>
<td class="org-left">1</td>
<td class="org-left">&#xa0;</td>
<td class="org-right">28.4</td>
</tr>


<tr>
<td class="org-left">大头文件</td>
<td class="org-left">一个结构体</td>
<td class="org-left">1+big.h</td>
<td class="org-left">500struct</td>
<td class="org-right">33.4</td>
</tr>


<tr>
<td class="org-left">更大的头文件</td>
<td class="org-left">一个结构体</td>
<td class="org-left">1+big.h</td>
<td class="org-left">5000struct</td>
<td class="org-right">73</td>
</tr>


<tr>
<td class="org-left">减少＃include</td>
<td class="org-left">501个结构体</td>
<td class="org-left">1</td>
<td class="org-left">&#xa0;</td>
<td class="org-right">33.4</td>
</tr>


<tr>
<td class="org-left">增加＃include</td>
<td class="org-left">一个结构体</td>
<td class="org-left">1+big.h</td>
<td class="org-left">500个头文件</td>
<td class="org-right">33.4</td>
</tr>


<tr>
<td class="org-left">＃include深度增加</td>
<td class="org-left">2个深度200的头文件＋1个深度100的头文件</td>
<td class="org-left">1</td>
<td class="org-left">&#xa0;</td>
<td class="org-right">33.4</td>
</tr>
</tbody>
</table>

结论：

-   \#include 打开文件和拷贝文件的时间非常短（在linux下），包含多个小头文件不会比相同内容的大头文件慢。
-   头文件包含深度并不影响编译时间
-   只有头文件包含更多的信息才会影响编译时间。

## 编译时间优化<a id="orgheadline19"></a>

### 代码优化<a id="orgheadline7"></a>

从之前的实验我们可以看出，有效的编译优化就是编译单元仅包含自己所需的编译需要的头文件即可，多余的头文件和头文件中与本编译单元无关的内容均须尽量减少。

-   删除不必要的头文件包含，减少预编译展开头文件时间,
-   减少没有必要的inline ，它会降低编译速度。inline是为了提高运行效率的，在日常开发过程中，可采用编译宏的方式取消inline带来的编译开销
-   减少头文件的编译依赖，头文件的依赖具有传染性，尽可能小的编译依赖均有助于提高整体的编译时间。
-   具体降低头文件编译依赖的方法如：尽量使用前置声明代替头文件包含，降低模版头文件的依赖，把跟模版无关的文件提出来基类，尽可能使用接口，

### 工程优化<a id="orgheadline14"></a>

所谓工程角度就是不修改代码的情况下，提高整个工程的编译速度。 目前已知的几种技术进行介绍:

1.  PCH 预编译头文件技术

    预编译头文件，将对于项目不常变化的头文件，包含到特定的头文件中，将该文件预先编译为二进制文件，并为需要的编译单元添加该文件作为必须添加的头文件。减少头文件解析的时间，有效降低编译时间，gcc 在3.4和4.0之后的版本开始支持这项技术。在编译时增加选项-x c++-header。
    
    -   优点: 可以根据模块灵活选择，不依赖平台
    -   缺点：可能造成不自满足的现象

2.  Unity Source 编译单元聚合技术

    将所有编译单元通过＃include 的方式，引入一个cpp中，通过预编译展开为1个编译单元，该项技术减少了 文件打开次数，头文件展开次数，.o文件的生成次数，链接打包的次数。 尤其是饱受文件过多，外部头文件依赖较重的工程。
    
    -   优点: 单机速度提升非常巨大
    -   缺点：破坏了匿名空间，同一编译单元下存在先后顺序的隐患。未自满足不易发现。宏定义前后同名会产生冲突。

3.  ccache

    安装ccache，设置CC CXX 为ccache gcc/clang等，设置ccache -m 设置缓存容量
    
    -   优点: 对代码无影响，完全兼容gcc，共享ccache
    -   缺点：linux／unix 平台工具，有无故编译失败的情况,

4.  分布式编译 distcc

    之前几个都是在单个机器上提高编译速度的方法，distcc可以有效组织多个机器进行分布式编译。理论上这种 技术对编译时间的优化是没有上限的，越多的机器共同作用效果越明显。distcc曾一度进入xcode的标配，如今却被删除.
    
    -   优点: 分布式思想，有效利用闲置资源。
    -   缺点：linux／unix 平台工具，对网络环境有要求，协作机的gcc版本必须一致。分布式算法简单，无法做到负荷均衡，需借助dmucs。提升幅度有限最多提升3倍。

5.  tmpfs

    tmpfs是Linux/Unix系统上的一种基于内存的文件系统, 可以将编译从硬盘放到内存上做，实际测试提升效果不明显，可见编译速度瓶颈不在io上,
    据说网上有人在Windows下用了RAMDisk有10倍的提升,尚未实验过。

6.  库文件使用

    将不常改变的模块编译成库文件，减少每次项目代码编译过程中对该模块的重复编译。没有实验的必要，减少了需要编译的单元，必然提速

### 编译优化实验<a id="orgheadline15"></a>

基础实验：项目由1000个cpp，1000个.h组成, 每个cpp仅包含自己对应序号的.h和big.h, 其中big.h包含5000个struct 。每个头文件有一个struct(纯结构体包含3个int类型成员)， 设编译1个结构体需要时间x，创建.o时间y

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />

<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left">使用技术</th>
<th scope="col" class="org-left">备注</th>
<th scope="col" class="org-left">平均编译时间</th>
<th scope="col" class="org-left">计算</th>
</tr>
</thead>

<tbody>
<tr>
<td class="org-left">base</td>
<td class="org-left">&#xa0;</td>
<td class="org-left">73s</td>
<td class="org-left">(5000＋1)\*x\*1000＋1000\*y＝5001000x＋1000y</td>
</tr>


<tr>
<td class="org-left">PCH</td>
<td class="org-left">将big.h进行预编译头文件</td>
<td class="org-left">29s</td>
<td class="org-left">5000\*x＋ 1000\*x＋1000\*y ＝ 6000x＋1000y</td>
</tr>


<tr>
<td class="org-left">UnitySource</td>
<td class="org-left">将所有cpp，＃include到main.cpp中</td>
<td class="org-left">1s</td>
<td class="org-left">5000\*x＋ 1000\*x＋y ＝ 6000x＋y</td>
</tr>


<tr>
<td class="org-left">Ccache</td>
<td class="org-left">编译使用Ccache工具</td>
<td class="org-left">第一次 93s 第二次19s</td>
<td class="org-left">（1＋1000）＊拷贝时间 ＊hit率＋1000y（第二次）</td>
</tr>


<tr>
<td class="org-left">PCH+Ccache</td>
<td class="org-left">&#xa0;</td>
<td class="org-left">第一次103s 第二次26s</td>
<td class="org-left">&#xa0;</td>
</tr>


<tr>
<td class="org-left">DistCc</td>
<td class="org-left">一台协作机</td>
<td class="org-left">43s</td>
<td class="org-left">base／2 ＋ 网络时延＊2 ＋ 计算时延 （两台机器性能相同）</td>
</tr>
</tbody>
</table>

### 实验分析<a id="orgheadline16"></a>

8.8us解析一个结构体时间
31ms编译一个编译单元

结论：

-   拷贝时间约为解析时间的1/6
-   PCH 的优化时间取决于big.h的组织，和使用频率，现实项目中并非像实验这样所有编译单元都可以使用。
-   distcc 的时间优化与网络时延和调度算法有关。
-   Ccache 对时间的优化限于hit命中率，重复的全编译提速很多，增量编译优化有限。
-   UnitySource 是唯一缩短y时间的技术，这也可以印证写文件时间远大于读文件时间！真实项目中模块分层，还存在多次打包.o 的情况，

### 组合使用<a id="orgheadline17"></a>

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />

<col  class="org-left" />

<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left">&#xa0;</th>
<th scope="col" class="org-left">PCH</th>
<th scope="col" class="org-left">UnitySource</th>
<th scope="col" class="org-left">DistCc</th>
<th scope="col" class="org-left">Ccache</th>
</tr>
</thead>

<tbody>
<tr>
<td class="org-left">PCH</td>
<td class="org-left">x</td>
<td class="org-left">可用</td>
<td class="org-left">可用</td>
<td class="org-left">延长</td>
</tr>


<tr>
<td class="org-left">UnitySource</td>
<td class="org-left">可用</td>
<td class="org-left">x</td>
<td class="org-left">x</td>
<td class="org-left">意义不大</td>
</tr>


<tr>
<td class="org-left">DistCc</td>
<td class="org-left">可用</td>
<td class="org-left">x</td>
<td class="org-left">x</td>
<td class="org-left">可用</td>
</tr>


<tr>
<td class="org-left">Ccache</td>
<td class="org-left">延长</td>
<td class="org-left">意义不大</td>
<td class="org-left">可用</td>
<td class="org-left">x</td>
</tr>
</tbody>
</table>

### 总结<a id="orgheadline18"></a>

cmake用户可以使用cotire这个工具完成pch和unity source 的工程自动部署。目前还不是很成熟，不能选择性的对某些文件加pch，再就是会导致cmake时间变长 对于我们上千文件的工程尚且如此，大型工程里的应用还有待提高。以上工具和方法都使用并经过测试，从总体上来说pch和unity source都会破坏程序的独立性。 不到万不得已，谨慎使用。
