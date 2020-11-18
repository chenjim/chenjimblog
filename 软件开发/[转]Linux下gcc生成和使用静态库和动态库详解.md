[TOC]

>本文在[原文](http://blog.chinaunix.net/uid-23592843-id-223539.html)的基础上做一些详细验证，部分内容排版稍有调整
>本文地址：http://blog.csdn.net/CSqingchen/article/details/51546784

## 一、基本概念
### 1.1 什么是库
在windows平台和linux平台下都大量存在着库。
本质上来说库是一种可执行代码的二进制形式，可以被操作系统载入内存执行。
由于windows和linux的平台不同（主要是编译器、汇编器和连接器的不同），因此二者库的二进制是不兼容的。
本文仅限于介绍linux下的库。
 
 
### 1.2 库的种类
linux下的库有两种：静态库和共享库（动态库）。
二者的不同点在于代码被载入的时刻不同。
静态库的代码在编译过程中已经被载入可执行程序，因此体积较大。
共享库的代码是在可执行程序运行时才载入内存的，在编译过程中仅简单的引用，因此代码体积较小。
 
 
### 1.3 库存在的意义
库是别人写好的现有的，成熟的，可以复用的代码，你可以使用但要记得遵守许可协议。
现实中每个程序都要依赖很多基础的底层库，不可能每个人的代码都从零开始，因此库的存在意义非同寻常。
共享库的好处是，不同的应用程序如果调用相同的库，那么在内存里只需要有一份该共享库的实例。
 
 
### 1.4 库文件是如何产生的在linux下
静态库的后缀是.a，它的产生分两步
1. 由源文件编译生成一堆.o，每个.o里都包含这个编译单元的符号表
2. ar命令将很多.o转换成.a，成为静态库
动态库的后缀是.so，它由gcc加特定参数编译产生。
具体方法参见后文实例。
 
### 1.5 库文件是如何命名的，有没有什么规范
在linux下，库文件一般放在/usr/lib和/lib下，
静态库的名字一般为libxxxx.a，其中xxxx是该lib的名称
动态库的名字一般为libxxxx.so.major.minor，xxxx是该lib的名称，major是主版本号， minor是副版本号
 
 
### 1.6 如何知道一个可执行程序依赖哪些库
ldd命令可以查看一个可执行程序依赖的共享库，
例如`$ ldd /bin/lnlibc.so.6`
=> /lib/libc.so.6 (0×40021000)/lib/ld-linux.so.2
=> /lib/ld- linux.so.2 (0×40000000)
可以看到ln命令依赖于libc库和ld-linux库
 
 
### 1.7 可执行程序在执行的时候如何定位共享库文件
当系统加载可执行代码时候，能够知道其所依赖的库的名字，但是还需要知道绝对路径。
此时就需要系统动态载入器(dynamic linker/loader)
对于elf格式的可执行程序，是由ld-linux.so*来完成的，它先后搜索elf文件的 DT_RPATH段—环境变量LD_LIBRARY_PATH—/etc/ld.so.cache文件列表—/lib/,/usr/lib目录找到库文件后将其载入内存
如：`export LD_LIBRARY_PATH='pwd'`
将当前文件目录添加为共享目录
 
### 1.8 在新安装一个库之后如何让系统能够找到他
如果安装在/lib或者/usr/lib下，那么ld默认能够找到，无需其他操作。
如果安装在其他目录，需要将其添加到/etc/ld.so.cache文件中，步骤如下
1.编辑`/etc/ld.so.conf`文件，加入库文件所在目录的路径
2.运行`ldconfig 目录名字`，该命令会重建/etc/ld.so.cache文件

## 二、用gcc生成静态和动态链接库的示例
我们通常把一些公用函数制作成函数库，供其它程序使用。
函数库分为静态库和动态库两种。
 
静态库在程序编译时会被连接到目标代码中，程序运行时将不再需要该静态库。
 
动态库在程序编译时并不会被连接到目标代码中，而是在程序运行是才被载入，因此在程序运行时还需要动态库存在。
 
本文主要通过举例来说明在Linux中如何创建静态库和动态库，以及使用它们。
 
为了便于阐述，我们先做一部分准备工作。
 
### 2.1 准备好测试代码
hello.h(见程序1)为该函数库的头文件。
 
hello.c(见程序2)是函数库的源程序，其中包含公用函数hello，该函数将在屏幕上输出"Hello XXX!"。

main.c(见程序3)为测试库文件的主程序，在主程序中调用了公用函数hello。

测试机器环境：gcc version 4.8.4 ；ubuntu14.04.3

三个程序放在文件夹`~/testso`中
 程序1: hello.h
```
#ifndef HELLO_H 
#define HELLO_H 
    void hello(const char *name); 
#endif
```
程序2：hello.c
```
#include <stdio.h> 
void hello(const char *name) { 
    printf("Hello %s!\n", name); 
}
```
程序3：main.c
```
#include  "hello.h"
int main() 
 { 
     hello("everyone"); 
     return 0; 
 }
```
 
### 2.2 问题的提出
注意：这个时候，我们编译好的hello.o是无法通过gcc –o 编译的，这个道理非常简单，
hello.c是一个没有main函数的.c程序，因此不够成一个完整的程序，如果使用gcc –o 编译并连接它，GCC将报错。
无论静态库，还是动态库，都是由.o文件创建的。因此，我们必须将源程序hello.c通过gcc先编译成.o文件。
这个时候我们有三种思路：
1）  通过编译多个源文件，直接将目标代码合成一个.o文件。
2）  通过创建静态链接库libmyhello.a，使得main函数调用hello函数时可调用静态链接库。
3）  通过创建动态链接库libmyhello.so，使得main函数调用hello函数时可调用静态链接库。
### 2.3 思路一：编译多个源文件
在系统提示符下键入以下命令得到hello.o文件。
`gcc -c hello.c`
为什么不使用`gcc –o hello hello.c`这个道理我们之前已经说了，使用-c是什么意思呢？这涉及到gcc 编译选项的常识。
 
`gcc –o`是将.c源文件编译成为一个可执行的二进制代码(-o选项其实是制定输出文件文件名，如果不加-c选项，gcc默认会编译连接生成可执行文件，文件的名称有-o选项指定)，这包括调用作为GCC内的一部分真正的C编译器（ccl），以及调用GNU C编译器的输出中实际可执行代码的外部GNU汇编器（as）和连接器工具（ld）。
`gcc –c`是使用GNU汇编器将源文件转化为目标代码之后就结束，在这种情况下,只调用了C编译器（ccl）和汇编器（as），而连接器(ld)并没有被执行，所以输出的目标文件不会包含作为Linux程序在被装载和执行时所必须的包含信息，但它可以在以后被连接到一个程序。
我们运行ls命令看看是否生存了hello.o文件。
`$ ls`
`hello.c hello.h hello.o main.c`
在ls命令结果中，我们看到了hello.o文件，本步操作完成。
 
同理编译main
`$gcc –c main.c`
 
将两个文件链接成一个.o文件。
`$gcc hello.o main.o -o hello`
 
运行
`$ ./hello`
 
`Hello everyone!`
 
完成^ ^!

### 2.4 思路二：静态链接库
 
下面我们先来看看如何创建静态库，以及使用它。
 
静态库文件名的命名规范是以lib为前缀，紧接着跟静态库名，扩展名为.a。例如：我们将创建的静态库名为myhello，则静态库文件名就是libmyhello.a。在创建和使用静态库时，需要注意这点。创建静态库用ar命令。
 
在系统提示符下键入以下命令将创建静态库文件libmyhello.a。
`$ ar rcs libmyhello.a hello.o`
 
我们同样运行ls命令查看结果：
`$ ls`
`hello.c  hello.h hello.o libmyhello.a main.c`
 
ls命令结果中有libmyhello.a。
 
静态库制作完了，如何使用它内部的函数呢？只需要在使用到这些公用函数的源程序中包含这些公用函数的原型声明，然后在用gcc命令生成目标文件时指明静态库名，gcc将会从静态库中将公用函数连接到目标文件中。注意，gcc会在静态库名前加上前缀lib，然后追加扩展名.a得到的静态库文件名来查找静态库文件,因此，我们在写需要连接的库时，只写名字就可以，如libmyhello.a的库，只写:-lmyhello
 
在程序3:main.c中，我们包含了静态库的头文件hello.h，然后在主程序main中直接调用公用函数hello。下面先生成目标程序hello，然后运行hello程序看看结果如何。

`$ gcc -o hello main.c -static -L. -lmyhello`
`$ ./hello`
`Hello everyone!`
 
我们删除静态库文件试试公用函数hello是否真的连接到目标文件 hello中了。
 `$ rm libmyhello.a`
`$ ./hello`
`Hello everyone!`
 
程序照常运行，静态库中的公用函数已经连接到目标文件中了。
静态链接库的一个缺点是，如果我们同时运行了许多程序，并且它们使用了同一个库函数，这样，在内存中会大量拷贝同一库函数。这样，就会浪费很多珍贵的内存和存储空间。使用了共享链接库的Linux就可以避免这个问题。

共享函数库和静态函数在同一个地方，只是后缀有所不同。比如，在一个典型的Linux系统，标准的共享数序函数库是/usr/lib/libm.so。

当一个程序使用共享函数库时，在连接阶段并不把函数代码连接进来，而只是链接函数的一个引用。当最终的函数导入内存开始真正执行时，函数引用被解析，共享函数库的代码才真正导入到内存中。这样，共享链接库的函数就可以被许多程序同时共享，并且只需存储一次就可以了。共享函数库的另一个优点是，它可以独立更新，与调用它的函数毫不影响。

### 2.5 思路三、动态链接库（共享函数库）
 
我们继续看看如何在Linux中创建动态库。我们还是从.o文件开始。
 
动态库文件名命名规范和静态库文件名命名规范类似，也是在动态库名增加前缀lib，但其文件扩展名为.so。例如：我们将创建的动态库名为myhello，则动态库文件名就是libmyhello.so。用gcc来创建动态库。
 
在系统提示符下键入以下命令,得到动态库文件libmyhello.so。
`$ gcc -shared -fPIC -c hello.c`  注：chenjim添加，原文没有，下一行会报错
 `$ gcc -shared -fPIC -o libmyhello.so hello.o`
 
 “PIC”命令行标记告诉GCC产生的代码不要包含对函数和变量具体内存位置的引用，这是因为现在还无法知道使用该消息代码的应用程序会将它连接到哪一段内存地址空间。这样编译出的hello.o可以被用于建立共享链接库。建立共享链接库只需要用GCC的”-shared”标记即可。
 
我们照样使用ls命令看看动态库文件是否生成。
`$ ls`
 `hello.cpp hello.h hello.o libmyhello.so main.cpp`
调用动态链接库编译目标文件。
 
在程序中使用动态库和使用静态库完全一样，也是在使用到这些公用函数的源程序中包含这些公用函数的原型声明，然后在用gcc命令生成目标文件时指明动态库名进行编译。我们先运行gcc命令生成目标文件，再运行它看看结果。
 
如果直接用如下方法进行编译，并连接：
`$ gcc -o hello main.c -L. -lmyhello`
 
（使用”-lmyhello”标记来告诉GCC驱动程序在连接阶段引用共享函数库libmyhello.so。”-L.”标记告诉GCC函数库可能位于当前目录）
 
`$ ./hello`
`./hello: error while loading shared libraries: libmyhello.so: cannot open shared object file: No such file or directory`
 
错误提示，找不到动态库文件libmyhello.so。程序在运行时，会查找需要的动态库文件，顺序参考后文介绍。若找到，则载入动态库，否则将提示类似上述错误而终止程序运行。有多种方法可以解决，
（1）我们将文件 libmyhello.so复制到目录/usr/lib中，再试试。
`$ sudo mv libmyhello.so /usr/lib`
（2）既然连接器会搜寻LD_LIBRARY_PATH所指定的目录，那么我们可以将这个环境变量设置成当前目录：
`export LD_LIBRARY_PATH=$(pwd)`
（3）`sudo ldconfig ~/testso`
注:   当用户在某个目录下面创建或拷贝了一个动态链接库,若想使其被系统共享,可以执行一下"ldconfig   目录名"这个命令。此命令的功能在于让ldconfig将指定目录下的动态链接库被系统共享起来,意即:在缓存文件`/etc/ld.so.cache`中追加进指定目录下的共享库.本例让系统共享了`~/tests`目录下的动态链接库。

---
下面的这个错误我没有遇到，不过也记录下，给遇到的人：
报错内容如下：/hello: error while loading shared libraries: /usr/lib/libmyhello.so: cannot restore segment prot after reloc: Permission denied
google了一下，发现是SELinux搞的鬼，解决办法有两个：
1. `chcon -t texrel_shlib_t   /usr/lib/libmyhello.so`
    (chcon -t texrel_shlib_t "你不能share的库的绝对路径")
2. `$vi /etc/sysconfig/selinux file`
    或者用
   `$gedit /etc/sysconfig/selinux file`
    修改`SELINUX=disabled`
    重启
    
---

这也进一步说明了动态库在程序运行时是需要的。
可以查看程序执行时调用动态库的过程：
`$ ldd hello`
可以看到它是如何调用动态库中的函数的。
	linux-vdso.so.1 =>  (0x00007fffe8f9b000)
	libmyhello.so => /home/chenjw/testso/libmyhello.so (0x00007fbe807d5000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fbe80410000)
	/lib64/ld-linux-x86-64.so.2 (0x000055dc016c2000)

原文中说，使用静态库和使用动态库编译成目标程序使用的gcc命令完全一样，
可能因为我们的环境不一样，上文我多加了一行编译命令`gcc -shared -fPIC -c hello.c`，
所以原文的验证，那当静态库和动态库同名时，gcc命令会使用哪个库文件呢？
在我的环境没有意义，add by chenjim。

### 2.6 编译参数解析

最主要的是GCC命令行的一个选项:
- -shared 该选项指定生成动态连接库（让连接器生成T类型的导出符号表，有时候也生成弱连接W类型的导出符号），不用该标志外部程序无法连接。相当于一个可执行文件
- l -fPIC：表示编译为位置独立的代码，不用此选项的话编译后的代码是位置相关的所以动态载入时是通过代码拷贝的方式来满足不同进程的需要，而不能达到真正代码段共享的目的。
- l -L.：表示要连接的库在当前目录中
- l -ltest：编译器查找动态连接库时有隐含的命名规则，即在给出的名字前面加上lib，后面加上.so来确定库的名称
- l LD_LIBRARY_PATH：这个环境变量指示动态连接器可以装载动态库的路径。
- l 当然如果有root权限的话，可以修改/etc/ld.so.conf文件，然后调用 /sbin/ldconfig来达到同样的目的，不过如果没有root权限，那么只能采用输出LD_LIBRARY_PATH的方法了。  
调用动态库的时候有几个问题会经常碰到，有时，明明已经将库的头文件所在目录 通过 “-I” include进来了，库所在文件通过 “-L”参数引导，并指定了“-l”的库名，但通过ldd命令察看时，就是死活找不到你指定链接的so文件，这时你要作的就是通过修改 LD_LIBRARY_PATH或者/etc/ld.so.conf文件来指定动态库的目录。通常这样做就可以解决库无法链接的问题了。

### 2.7 静态库链接时搜索路径顺序：

1. ld会去找GCC命令中的参数-L

2. 再找gcc的环境变量LIBRARY_PATH

3. 再找内定目录 /lib /usr/lib /usr/local/lib 这是当初compile gcc时写在程序内的

### 2.8 动态链接时、执行时搜索路径顺序:

1.  编译目标代码时指定的动态库搜索路径；

2.  环境变量LD_LIBRARY_PATH指定的动态库搜索路径；

3.  配置文件/etc/ld.so.conf中指定的动态库搜索路径；

4. 默认的动态库搜索路径/lib；

5. 默认的动态库搜索路径/usr/lib。

### 2.9 有关环境变量：

LIBRARY_PATH环境变量：指定程序静态链接库文件搜索路径

LD_LIBRARY_PATH环境变量：指定程序动态链接库文件搜索路径

## 三、linux开发之文件夹和应用程序

### 1. 应用程序(Applications)

应用程序通常都有固定的文件夹，系统通用程序放在/usr/bin，日后系统管理员在本地计算机安装的程序通常放在/usr/local/bin或者/opt文件夹下。除了系统程序外，大部分个人用到的程序都放在/usr/local下，所以保持/usr的整洁十分重要。当升级或者重装系统的时候，只要把/usr/local的程序备份一下就可以了。

一些其他的程序有自己特定的文件夹，比如X Window系统，通常安装在/usr/X11中，或者/usr/X11R6。GNU的编译器GCC，通常放置在/usr/bin或者/usr/local/bin中，不同的Linux版本可能位置稍有不同。

### 2. 头文件(Head Files)

在C语言和其他语言中，头文件声明了系统函数和库函数，并且定义了一些常量。对于C语言，头文件基本上散落于/usr/include和它的子文件夹下。其他的编程语言的库函数分布在编译器定义的地方，比如在一些Linux版本中，X Window系统库函数分布在/usr/include/X11，GNU C++的库函数分布在/usr/include/g++。这些系统库函数的位置对于编译器来说都是“标准位置”，即编译器能够自动搜寻这些位置。

如果想引用位于标准位置之外的头文件，我们需要在调用编译器的时候加上-I标志，来显式的说明头文件所在文件夹。比如，
`$ gcc -I/usr/openwin/include hello.c`

会告诉编译器除了标准位置外，还要去/usr/openwin/include看看有没有所需的头文件。详细情况见编译器的使用手册(man gcc)。

### 3. 库函数(Library Files)

库函数就是函数的仓库，它们都经过编译，重用性不错。通常，库函数相互合作，来完成特定的任务。比如操控屏幕的库函数(cursers和ncursers库函数)，数据库读取库函数(dbm库函数)等。

系统调用的标准库函数一般位于/lib以及/usr/lib。C编译器(精确点说，连接器)需要知道库函数的位置。默认情况下，它只搜索标准C库函数。

库函数文件通常开头字母是lib。后面的部分标示库函数的用途(比如C库函数用c标识， 数学库函数用m标示)，小数点后的后缀表明库函数的类型：

.a 指静态链接库
.so 指动态链接库
去/usr/lib看一下，你会发现，库函数都有动态和静态两个版本。

与头文件一样，库函数通常放在标准位置，但我们也可以通过-L标识符，来添加新的搜索文件夹，-l指定特定的库函数文件。比如
` $ gcc -o x11fred -L/usr/openwin/lib x11fred.c -lX11`
上述命令就会在编译期间，链接位于/usr/openwin/lib文件夹下的libX11函数库，编译生成x11fred。

静态链接库(Static Libraries)

最简单的函数库就是一些函数的简单集合。调用库函数中的函数时，需要在调用函数中include定义库函数的头文件。我们用-l选项添加标准函数库之外的函数库。