###### <sub><font color = orange>王道C++班级参考资料</font></sub><br />——<br />Linux部分<sup><font color=white>卷2</font></sup>GNU工具集<br/><sup><sub><font color=cyan>节4</font></sub><font color=cyan>Makefile</font></sup><br/><br/>*最新版本`V3.0`*<br>**王道C++团队**<br/>*COPYRIGHT ⓒ 2021-2024. 王道版权所有*

[TOC]

# 问题引入

> _~Gn!~_
>
> 好，到目前为止，在Linux下进行开发，基础的shell指令、Vim编辑器以及编译调试工具你都已经学习过了，距离真正的在Linux上进行开发已经非常接近了，但还欠缺一个非常重要的部分。下面我们来通过一个案例引入这个问题。
>
> 正式的开发中，往往会以<font color=red>**工程**</font>会基础的开发单元，一个工程内部有大量的头文件、源文件等。这时就有问题了：
>
> 1. 如此庞大数量的源文件，怎么编译呢？先后顺序是什么？
> 2. 如此庞大数量的源文件，重新启动程序时，每一个文件都要重新编译吗？
>
> 前几周我们在Windows下的VS中开发时，没有思考过这样的问题，那是因为VS作为集成开发环境，已经替你做了这个事情。
>
> 但在Linux环境下，这一系列的问题都需要程序员自己思考，自己解决了。
>
> 那怎么办呢？
>
> 对于C/C++的程序而言，源文件的编译顺序一般不影响最终可执行程序的生成，因为各个源文件都是独立编译最终再链接到一起的。
>
> 所以我们来思考问题2，问题2其实非常重要：
>
> 对于一个大型工程来说，如果修改任一源文件就需要编译整个工程，那么也太耗费时间了。比如一个Linux系统内核在普通机器上编译一次，可能需要5~6小时，所以我们总是希望：
>
> 在一个工程的开发中，那些更新过的和新增的源文件需要重新编译，而那些没修改过的源文件，是不会重新编译的。这种编译工程的策略就是——<span style=color:red;background:yellow>**"增量编译"**</span>！
>
> 于是在C/C++的工程开发中，就有两个亟待实现的功能：
>
> 1. 那么多源文件需要编译，能不能制定一套规则，从而自动化的实现它们的编译链接呢？
> 2. 如何实现增量编译呢？
>
> 这两个功能的实现，就需要依靠Makefile来完成了。

# Makefile是什么？

> _~Gn!~_
>
> 那么，究竟什么是Makefile呢？
>
> <span style=color:red;background:yellow>**Makefile本质上是一脚本文件，这个文件中存放了编译和构建程序的脚本指令，定义了整个工程构建可执行程序的规则。**</span>
>
> 利用Makefile脚本文件制定好构建程序的规则后，我们就可以使用`make`指令来解释Makefile文件，从而实现自动化构建程序，以及增量编译。
>
> 也就是说，只要定义好了一个Makefile脚本文件，那么`make`指令的作用就相当于VS当中的`启动`按钮(不会自动执行)，非常Nice~
>
> 下面我们还是以一个quick start来给大家演示一下Makefile的基础功能。

# 快速开始Makefile

> _~Gn!~_
>
> 首先我们随便创建两个源文件`main.c`以及`add.c`，再创建一个头文件`compute.h`。让它们之间具有依赖关系，比如：
>
> ```` c
> // compute.h
> #ifndef __WD_COMPUTE_H
> #define __WD_COMPUTE_H
> 
> int add(int a, int b); 
> #endif
> 
> // add.c
> #include <stdio.h>
> #include "compute.h"             
> 
> int add(int a, int b){
>        return a + b;
> }
> 
> // main.c
> #include <stdio.h>
> #include "compute.h"
> 
> int main(void){
>        printf("4 + 2 = %d\n", add(4, 2));		// 该函数在compute.h中声明，add.c中定义
>        return 0;
> }
> ````
>
> 然后在当前目录下，新建一个Makefile文件<span style=color:red;background:yellow>**(文件名必须是Makefile或者makefile)**</span>，输入以下内容：
>
> ```` makefile
> main: add.o main.o
> 	#第二行命令的开头必须是一个制表符，如果是空格会运行报错  
> 	gcc add.o main.o -o main
> add.o: add.c compute.h
> 	gcc -c add.c -o add.o -Wall -g
> main.o: main.c compute.h
> 	gcc -c main.c -o main.o -Wall -g
> ````
>
> 那么这一段脚本指令是啥意思呢？

## Makefile脚本基本单元

> _~Gn!~_
>
> Makefile脚本指令的基本组成单元是<span style=color:red;background:yellow>**规则（Rule）**</span>，一个规则又分成以下几个小部分：
>
> ```` makefile
> target: prerequisites
> 	#command行的开头必须是一个制表符，若换成空格符，Makefile脚本会运行报错
> 	commands
> ````
>
> 也就是说，一个<span style=color:red;background:yellow>**规则**</span>由：<span style=color:red;background:yellow>**目标（Target）、依赖（Prerequisites）、命令（Commands）**</span>三部分组成。下面逐一解释：
>
> 1. <font color=red>**目标（Target）**</font>：目标通常是文件名，代表了要生成或更新的文件。比如：
>    1. 可以是可执行程序的名字
>    2. 可以是目标文件的名字
>    3. 或者是其它任何想要借助 Makefile 创建和维护的文件。
>    4. <span style=color:red;background:yellow>**注意：一条规则中，只有一个目标，不能有多个目标。**</span>
> 2. <font color=red>**依赖（Prerequisites）**</font>：代表生成目标时所依赖的文件，可以是一个，也可以是很多个。
> 3. <font color=red>**命令（Commands）**</font>：代表生成目标时所执行的指令，可以是一条，也可以是多条。
>    1. 注意：命令可以是任何shell命令，不局限于编译链接的gcc指令。
>    2. <span style=color:red;background:yellow>**注意：每个命令必须以一个制表符（Tab）开始，这是 Makefile 语法的强制要求！！！**</span>

## Makefile脚本基本工作原理

> _~Gn!~_
>
> 当我们写好了一个Makefile脚本文件后，执行`make`命令，Makefile工作流程如下：
>
> <font color=red>**读取 Makefile文件：**</font>当你运行 `make` 命令时，它会自动读取当前目录下的 Makefile/makefile 文件。
>
> <span style=color:red;background:yellow>**重复以下过程：**</span>
>
> 1. <font color=red>**确定目标：**</font>Makefile脚本代码，在默认情况下第一个目标就是构建的最终目标。
> 2. <font color=red>**检查目标是否存在或过时：**</font>
>    1. 如果目标已经存在，则检查所有依赖。如果任一依赖不存在，或者任何依赖的最后修改时间比目标的最后修改时间更晚，说明目标已经过时！
>    2. 如果目标不存在或者目标过时，则认定需要生成目标。
> 3. <font color=red>**生成目标：**</font>
>    1. 如果仅是依赖发生了更新(最后修改时间比目标晚)，那么就立刻执行该规则下的命令，生成目标。
>    2. <span style=color:red;background:yellow>**如果依赖不存在，那就递归的检查依赖，生成依赖，直到生成目标所需要的依赖都已存在且最新，那么就执行该规则下的命令，生成目标。**</span>
>
> 重复以上过程，直到：
>
> 1. <font color=red>**正常结束：**</font>第一个目标正确生成完毕，Makefile执行完毕。<font color=red>**（Makefile默认不会构建和第一个目标无关的其它目标，即便有）**</font>
> 2. <font color=red>**报错退出：**</font>如果执行任何命令过程中出现错误或者依赖的某个文件递归查找也无法生成或找到，Makefile会退出并打印错误信息。
>
> <span style=color:yellow;background:red>**当然以上只是Makefile脚本的基本工作原理，Makefile还可以实现一些更复杂的功能，但最常用和基本的还是上面讲的。**</span>

## 案例分析

> _~Gn!~_
>
> 那么以下Makefile脚本会如何执行呢？
>
> ```` makefile
> main: add.o main.o
> 	#第二行命令的开头必须是一个制表符，如果是空格会运行报错  
> 	gcc add.o main.o -o main
> add.o: add.c compute.h
> 	gcc -c add.c -o add.o -Wall -g
> main.o: main.c compute.h
> 	gcc -c main.c -o main.o -Wall -g
> ````
> 
>首先我们"工程"的初始状态是：只有main.c、compute.c以及compute.h三个文件存在。
> 
>那么Makefile脚本的工作如下：
> 
>1. 确定生成的目标是`main`
> 2. 目标main不存在，于是确定要生成目标main
> 3. 检查main的两个依赖：
>    1. main.o依赖不存在，于是找到以main.o为目标的规则，执行规则中的指令，生成main.o文件
>    2. add.o依赖不存在，于是找到以add.o为目标的规则，执行规则中的指令，生成add.o文件
> 
>4. 两个依赖都生成完毕后，执行main目标下的命令，生成main可执行文件。
> 
>执行完毕这个Makefile脚本，此时当前目录下会有文件：
> 
>> add.c  add.o  compute.h  main  main.c  main.o  Makefile
> 
>思考(这些问题都基于上述状态)：
> 
>1. 如果删除main可执行执行，再次make，会执行哪些命令？
> 2. 如果删除main.o文件，再次make，会执行哪些命令？
> 3. 如果touch add.c文件，再次main，会执行哪些命令？
> 4. 如果touch main，再touch main.o文件，会执行哪些命令？
> 5. 如果touch main.o，再touch main，会执行哪些命令？
> 
>你可以自己执行一下上面的操作，相信你可以理解什么叫做<span style=color:red;background:yellow>**"增量编译"**</span>。

## 需要注意的细节问题

> _~Gn!~_
>
> Makefile脚本语法的要求非常严格，指令的开头必须是一个制表符，而不能是一堆空格组合。如下图所示：
>
> ![需要注意制表符-图](https://hixiaodong123.oss-cn-hangzhou.aliyuncs.com/typora/202403061651719.png#padding#600w)
>
> 我们可以通过查看"指令"是否变色，来确定是否正确使用制表符，一定要注意这个细节。

# Makefile高级

> _~Gn!~_
>
> 以上，Makefile脚本最基本也最常用，最重要的部分我们就学完了，下面我们来看一下Makefile脚本的高级功能。
>
> 这些高级功能，有些需要理解掌握，有些了解即可。

## 伪目标（重要）

> _~Gn!~_
>
> 通过上面对Makefile语法的理解，我们知道：如果一个目标不存在，那么就会尝试利用给出的依赖，并执行命令来生成目标。
>
> 那么如果一个目标是：
>
> 1. 没有依赖，不给它添加任何依赖。
> 2. 但存在至少一条命令。
>
> 比如：
>
> ```` makefile
> FakeTarget:
> 	echo "hello world!"		#向屏幕输出hello world!
> ````
>
> 执行make，效果是什么呢？
>
> 效果是：
>
> 每次make，都一定会执行指令，比如上面的`echo "hello world!"`就一定会执行。
>
> 在Makefile脚本当中，像`FakeTarget`这样的，对应文件不存在且一定无法生成的目标，被称之为<span style=color:red;background:yellow>**"伪目标"**</span>。
>
> <span style=color:red;background:yellow>**伪目标的意义就在于：确保规则中的所有命令都必然要执行。**</span>
>
> 试想，如果写出这样的Makefile脚本：
>
> ```` makefile
> FakeTarget:
> 	echo "hello world!"
> 	make	#该规则中的第二个命令
> ````
>
> 效果是什么呢？
>
> 第二条指令当中的`make`会再次尝试构建目标`FakeTarget`，这个过程就类似无限递归，显然是一个不合理的Makefile脚本。
>
> 试想，如果写出这样的Makefile脚本：
>
> ```` makefile
> FakeTarget:
> 	echo "first fake"
> FakeTarget2:
> 	echo "second fake"
> ````
>
> 效果是什么呢？
>
> 效果是只输出"first fake"，<span style=color:red;background:yellow>**因为Makefile默认不会构建和第一个目标无关的其它目标，即便有这个目标。**</span>
>
> 但我们在使用`make`指令时，可以通过参数`make 目标名`来更改构建的目标。
>
> 比如执行`make FakeTarget2`，效果就是输出"second fake"，记住这个指令用法，它很重要。

## 伪目标的作用（重要）

> _~Gn!~_
>
> 从实际使用的角度出发，伪目标有什么实践价值呢？<span style=color:red;background:yellow>**或者换句话说：什么样的场景下，我们需要命令一定要执行呢？**</span>
>
> 伪目标在实际工作中非常有用，常用来实现<font color=red>**clean**</font>和<font color=red>**rebuild**</font>两个功能，脚本代码如下：
>
> ```` makefile
> main: main.o add.o
>	gcc main.o add.o -o main
> main.o: main.c compute.h
>	gcc -c main.c -Wall -g
> add.o: add.c compute.h
> 	gcc -c add.c -Wall -g
> clean:
> 	rm -f main main.o add.o		#删除所有生成的目标文件以及可执行程序，并且不给提示。-f是必须的
>rebuild: clean main				#clean和main要写在依赖的未知，而不是命令的位置！
> ````
>
> 这里定义了两个伪目标，clean和rebuild，我们可以利用指令`make clean`以及`make rebuild`来生成这两个目标。最终的效果是：
>
> 1. <font color=red>**make clean**</font>表示删除所有生成的目标文件以及可执行程序，这类似于VS当中的"清理解决方案/项目"的功能。
>2. <font color=red>**make rebuild**</font>表示要构建目标rebuild，于是去寻找依赖clean，发现clean不存在，于是先生成目标clean。然后再寻找依赖main，此时main可执行程序已经被清理了，于是就重新生成一次main可执行程序。这个过程实际上是：
>    1. 先执行"清理解决方案/项目"的功能。
>   2. 然后再将main可执行程序作为目标，重新编译链接生成一个新的main可执行程序。
>    3. <font color=red>**这类似于VS当中的"重新生成解决方案/项目"的功能。**</font>
>
> 建议在定义一个Makefile时，添加上这两个伪目标，以实现清理和重新生成的功能。
>
> 除此之外，实际工作中，我们还<span style=color:red;background:yellow>**建议用".PHONY"标记所有的伪目标**</span>，脚本代码如下：
> 
> ```` makefile
> #...其它脚本代码
> .PHONY: clean rebuild			#PHONY有假的意思, 用它标记伪目标是一个好的编程实践，注意.PHONY后面有一个冒号
>clean:
> 	rm -f main main.o add.o		#删除所有生成的目标文件以及可执行程序，并且不给提示。-f是必须的
>rebuild: clean main				#clean和main要写在依赖的未知，而不是命令的位置！
> ````
>
> 这么做有两个好处：
>
> 1. 增强代码可读取，立刻就知道某个目标是一个伪目标。
> 2. **<span style=color:red;background:yellow>即便当前目录下存在名为clean或rebuild的文件，"make clean"指令也能正常执行，而不会因文件clean/rebuild作为目标存在而不执行对应指令！</span>**

## 变量（了解）

> _~Gn!~_
>
> 通过上面的学习，你已经完全可以编写Makefile实现自己想要的功能了。但如果你希望你的Makefile脚本<span style=color:red;background:yellow>**更加通用**</span>，可以在Makefile中定义变量。
>
> Makefile脚本中的变量，非常类似于C/C++中的宏定义：
>
> 1. <font color=red>**Makefile变量，在本质上就是一个文本字符串，在执行脚本代码的时候会原模原样地展开在所使用的地方(类似于宏的文本替换)。**</font>
> 2. 但Makefile变量也有与C/C++的宏不同的地方，即我们可以<font color=red>**随时修改**</font> Makefile 中定义的变量的内容。
>
> 具体而言**Makefile变量**的语法如下：
>
> 1. **Makefile变量**的名称可以包含字母、数字和下划线，而且是大小写敏感的。也就是说："foo", "Foo"和"FOO"是三个不同的变量名。
>
> 2. 建议Makefile的变量命名风格是：<span style=color:red;background:yellow>**变量名全部大写，下划线分割单词。**</span>
>
> 3. **Makefile变量**在声明的时候，就需要赋初始值，后续使用也可以随时赋新值。语法如下：
>
>    ```` makefile
>    OUT := main		#定义了一个变量，变量名是OUT，初始值是main
>    ....
>    OUT := add		#OUT重新赋值为add
>    ````
>
> 4. **Makefile变量**在使用的时候：
>
>    1. <font color=red>**需要给在变量名前加上\$符号**</font>
>    2. <span style=color:red;background:yellow>**如果变量名不是单字符，就必须用()或大括号 {} 把变量名括起来。如果变量名是单字符，则允许不用括号。**</span>
>
>    演示如下所示：
>
>    ```` makefile
>    OUT := main
>    $(OUT): main.o add.o		#脚本执行时，等价于main: main.o add.o	变量名必须用()括起来
>    ````
>
> **Makefile当中的变量，又可以分为三类：**
>
> 1. 自定义变量
> 2. 自动变量
> 3. 预定义变量
>
> 下面逐一讲解。

### 自定义变量

> _~Gn!~_
>
> 自定义变量就是程序员自己编写代码定义的变量，上面语法演示中的变量都属于自定义变量。
>
> 对于上面我们已经实现过的脚本：
>
> ```` makefile
> main: main.o add.o
>    	gcc main.o add.o -o main
> main.o: main.c compute.h
>    	gcc -c main.c -o main.o -Wall -g
> add.o: add.c compute.h
>    	gcc -c add.c -o add.o -Wall -g
> 
> clean:
>    	rm -f main main.o add.o
> rebuild: clean main
> ````
>
> 可以利用自定义变量，修改成：
>
> ```` makefile
> OUT := main     #目标文件
> OBJS := main.o add.o       #生成目标文件所需要的依赖   
> COM_OP := -Wall -g        #编译选项
> 
> $(OUT):$(OBJS)
>    	gcc main.o add.o -o main
> main.o: main.c compute.h
>    	gcc -c main.c -o main.o $(COM_OP)
> add.o: add.c compute.h
>    	gcc -c add.c -o add.o $(COM_OP)
> 
> clean:
>    	rm -f $(OUT) $(OBJS) 
> rebuild: clean $(OUT)
> ````
>
> 下面，我们接着看自动变量。

### 自动变量

> _~Gn!~_
>
> Makefile中的自动变量是指那些在<span style=color:red;background:yellow>**规则**</span>执行时<font color=red>**自动设置的变量**</font>，它们通常用于引用规则的目标和依赖项。这些变量在Makefile的执行过程中非常有用，因为它们能够让规则更加简洁且易于管理。下面是一些常用的自动变量，<font color=red>**变量名**</font>及其描述如下<span style=color:red;background:yellow>**(注意是变量名)**</span>：
>
> 1. `@`：表示此规则的目标文件名。
> 2. `<`：表示此规则的第一个依赖文件名。
> 3. `^`：表示此所有的依赖文件列表，这些依赖项之间以空格分隔。
>
> <span style=color:red;background:yellow>**注意：这些都只是变量名，要使用这些变量的话，需要在变量名前加字符$（单字符名不用加括号）。**</span>
>
> 于是我们就可以进一步的修改上面的Makefile脚本代码，如下：
>
> ```` makefile
> OUT := main     #目标文件
> OBJS := main.o add.o       #生成目标文件所需要的依赖
> COM_OP := -Wall -g        #编译选项
> 
> $(OUT):$(OBJS)
>     gcc $^ -o $@
> main.o: main.c compute.h
>     gcc -c $< -o $@ $(COM_OP)
> add.o: add.c compute.h
>     gcc -c $< -o $@ $(COM_OP)                            
> 
> .PHONY: clean rebuild
> clean:
> 	rm -f $(OUT) $(OBJS)         
> rebuild: clean $(OUT)
> ````
>
> 这样一番修改后，可读性有一定的下降，但脚本代码确实更简洁了。

### 预定义变量

> _~Gn!~_
>
> 预定义变量，即由Makefile自身预先定义好的变量，我们可以直接拿来，也可以先重新赋值再用。
>
> 常用的预定义变量如下：
>
> |  变量名  |      功能      | 默认含义 |
> | :------: | :------------: | :------: |
> |    AR    | 打包静态库文件 |    ar    |
> |    AS    |    汇编程序    |    as    |
> |    CC    |    C编译器     |    cc    |
> |   CPP    |   C预编译器    | $(CC) -E |
> |   CXX    |   C++编译器    |   g++    |
> |    RM    |      删除      |  rm –f   |
> | ARFLAGS  |     库选项     |    无    |
> | ASFLAGS  |    汇编选项    |    无    |
> |  CFLAGS  |  C编译器选项   |    无    |
> | CPPFLAGS | C预编译器选项  |    无    |
> | CXXFLAGS | C++编译器选项  |    无    |
>
> <font color=red>**仍然需要注意的是，这些都是变量名，要使用这些变量的话，需要在变量名前加字符$以及括号。**</font>
>
> 举个例子，我们可以把Makefile脚本中的所有`gcc`都用`$(CC)`取代，可以这么做：
>
> 先把变量`CC`的取值改变为gcc：
>
> ```` makefile
> CC := gcc
> ````
>
> 利用Vim指令将全部的`gcc`替换为`$(CC)`，可以使用指令：
>
> ```` shell
> :%s/gcc/$(CC)/g			
> ````
>
> 其中：
>
> 1. `%`表示在整个文件范围内进行替换操作，如果你希望在[a, b]行内进行替换，可以将`%`替换成`a,b`
> 2. `s`是替换的指令。
> 3. 最后一个`g`的含义是一行内的所有匹配项都将被替换。
> 4. 如果你希望每次替换都询问是否执行，可以添加一个`c`标志。即从`g`改为`gc`，就表示全局替换并每次询问是否执行替换。
>
> 于是引入预定义变量后，我们可以将上面的 Makefile脚本改写成：
>
> ```makefile
> OUT := main     #目标文件
> OBJS := main.o add.o       #生成目标文件所需要的依赖
> COM_OP := -Wall -g        #编译选项
> CC := gcc        #修改CC的默认值                            
> 
> $(OUT):$(OBJS)
>     $(CC) $^ -o $@
> main.o: main.c compute.h
>     $(CC) -c $< -o $@ $(COM_OP)
> add.o: add.c compute.h
>     $(CC) -c $< -o $@ $(COM_OP)   
> 
> .PHONY: clean rebuild
> clean:
> 	$(RM) $(OUT) $(OBJS)         
> rebuild: clean $(OUT)
> ```
>
> 以上，我们就搞清楚了Makefile变量的基础使用。
>
> 此时，我们新增一个文件`sub.c`，包含头文件，并在main.c中调用其中的函数，于是我们就可以快速修改这个Makefile为：
>
> ```` makefile
> OUT := main     #目标文件
> OBJS := main.o add.o sub.o      #生成目标文件所需要的依赖
> COM_OP := -Wall -g        #编译选项
> CC := gcc        #修改CC的默认值
> 
> $(OUT):$(OBJS)
>     $(CC) $^ -o $@
> main.o: main.c compute.h
>     $(CC) -c $< -o $@ $(COM_OP)
> add.o: add.c compute.h
>     $(CC) -c $< -o $@ $(COM_OP) 
> sub.o: sub.c compute.h
>     $(CC) -c $< -o $@ $(COM_OP)
>     
> .PHONY: clean rebuild
> clean:
>     $(RM) $(OUT) $(OBJS)
> rebuild: clean $(OUT)
> ````
>
> 借助Makefile的变量，可以实现快速的修改脚本文件，以提高Makefile脚本的通用性。

## 模式规则（了解）

> _~Gn!~_
>
> 通过Makefile的变量，你已经将脚本代码写得很简洁了，但简洁不是我们的主要目的，通用才是。目前这个Makefile，我们觉得它仍然不够通用，于是我们继续学习Makefile的模式规则，以将Makefile写得更加通用。
>
> 学到现在，我们都知道Makefile脚本语法的基本单位是一个<font color=red>**规则**</font>，而<span style=color:red;background:yellow>**模式规则**</span>语法就是一种特殊的规则。
>
> 所谓模式规则：
>
> 用于定义如何从一种类型的文件生成另一种类型的文件，它直接指定了文件之间的转换规则。模式规则下一般使用百分号（%）来表示文件名中的通配符部分。
>
> 它的语法结构如下：
>
> ```` makefile
> %.target : %.source
> 	commands
> ````
>
> 解释如下：
>
> 1. `%.target`表示可以匹配任意目标文件。比如`%.o`表示任意.o文件作为目标
> 2. `%.source`表示可以匹配任意依赖文件。比如`%.c`表示任意.c文件作为依赖
>
> 下面示例就是 Makefile 的一个模式规则，即由.c文件生成.o文件的模式规则：
>
> ```makefile
> %.o : %.c
> 	$(CC) -c $< -o $@
> ```
>
> <span style=color:red;background:yellow>**使用这个模式规则后，Makefile可以自动推导如何从任何 .c 文件生成 .o 文件，这样就可以大幅度简化脚本代码。**</span>
>
> 于是有了模式规则后，我们可以这样写 Makefile：
>
> ```makefile
> OUT := main     #目标文件
> OBJS := main.o add.o sub.o      #生成目标文件所需要的依赖
> COM_OP := -Wall -g        #编译选项
> CC := gcc        #修改CC的默认值
> 
> $(OUT):$(OBJS)
>     $(CC) $^ -o $@
> 
> %.o : %.c compute.h
> 	$(CC) -c $< -o $@ $(COM_OP)
> #以下内容可以被上面一个模式规则替代
> #main.o: main.c compute.h
> #    $(CC) -c $< -o $@ $(COM_OP)
> #add.o: add.c compute.h
> #    $(CC) -c $< -o $@ $(COM_OP) 
> #sub.o: sub.c compute.h
> #    $(CC) -c $< -o $@ $(COM_OP)                       
> 
> .PHONY: clean rebuild
> clean:
>     $(RM) $(OUT) $(OBJS)         
> rebuild: clean $(OUT)
> ```
>
> 显然使用模式规则，可以极大的简化代码，而且还可以提高扩展性。
>
> 比如这时，我们再增加一个运算`mul`（乘法），创建一个`mul.c`文件包含头文件compute，并实现函数。此时我们该如何改这个Makefile呢？
>
> 很简单只需要修改一个`OBJS`自定义变量的值就可以了：
>
> ```` makefile
> OBJS := main.o add.o sub.o mul.o      #生成目标文件所需要的依赖
> ````
>
> 只需要改变上面一行就可以了。
>
> 注意：
>
> 1. <span style=color:red;background:yellow>**模式规则不能作为Makefile的第一个规则**</span>，因为Makefile会选择第一个明确的目标作为默认目标，而模式规则中的目标，不指定具体的目标文件名，而仅仅定义了从一种文件类型转换到另一种文件类型的通用规则。<font color=red>**模式规则中的目标不是一个明确的目标，Makefile脚本执行时若把模式规则写在最上面，脚本执行会跳过这个模式规则目标。**</font>
> 2. 模式规则定义了如何将一种类型的文件（比如.c源文件）转换成另一种类型的文件（比如.o目标文件），这种转换是根据文件名模式来完成匹配的。
> 3. 在模式规则中，通配符 `%` 的匹配是一致的，即目标和依赖列表中 `%` 所表示的内容是一致的。

## 内置函数（了解）

> _~Gn!~_
>
> 经过上面的一番折腾，我们的Makefile已经非常简单，非常具有扩展性了。但"不满足"是进步的阶梯，我们能不能把Makefile扩展到新增.c文件完全不需要改动这个Makefile呢？
>
> 于是我们就需要学习一下Makefile的内置函数语法。
>
> Makefile脚本语言，也像传统编程语言一样，支持函数调用，这种函数，我们称之为<font color=red>**Makefile的"内置函数"**</font>。
>
> 内置函数的调用语法，和使用变量的语法非常类似，如下：
>
> ```makefile
> $(<function> <arguments>)		#调用时要用小括号把函数名和参数括起来
> ${<function> <arguments>}		#也可以用大括号括起来，两种方式都可以，使用时二选一
> ```
>
> 以上两种方式都可以，解释如下：
>
> 1. \<function\>为函数名。<span style=color:red;background:yellow>**注意尖括号只是为了语法格式美观，它不是实际调用语法的一部分。**</span>
> 2. \<arguments\>为参数列表，允许出现一个或多个参数，参数之间以逗号分隔。<span style=color:red;background:yellow>**注意尖括号只是为了语法格式美观，它不是实际调用语法的一部分。**</span>
> 3. <font color=red>**函数名和参数列表之间以"空格"分隔。**</font>
> 4. <font color=red>**参数允许有多个，多个参数之间用","分隔。**</font>
>
> Makefile 内置的函数并不算多，我们这里只介绍两个最常用的：
>
> <span style=color:red;background:yellow>**通配符函数**</span>
>
> ```makefile
> $(wildcard <pattern>) 	
> ```
>
> 函数的名字是：wildcard
>
> **它的作用是：查找符合\<pattern>的所有文件列表**
>
> **函数的返回值是：返回所有符合\<pattern>的文件名，文件名之间以空格分隔。**
>
> 示例：
>
> ```makefile
> SRCS := $(wildcard *.c)
> ```
>
> 其效果是：查找当前目录下，所有以.c结尾的文件名，并将文件名以空格分隔，再赋值给变量SRCS。
>
> 比如当前目录下有.c文件：main.c、add.c、sub.c
>
> 那么这个函数调用的作用等价于：
>
> ```` makefile
> SRCS := main.c add.c sub.c
> ````
>
> <span style=color:red;background:yellow>**模式替换函数**</span>
>
> ```makefile
> $(patsubst <pattern>,<replacement>,<text>)
> ```
>
> **函数的名字是：patsubst，它是词组"pattern substitution"的简写，意为模式替换。**
>
> <font color=red>**它的作用是：查找\<text>中符合模式\<pattern>的单词(单词以空白字符分隔)，将其替换为\<replacement>。**</font>
>
> 注意事项：
>
> 1. \<pattern>可以包括通配符%，表示任意长度的字符串。
> 2. 如果\<replacement>中也含有%，那么\<replacement>中的%所代表的字符串和\<pattern>中%所代表的字符串相同。
> 3. \<text>作为替换的数据源，它是最后一个参数。
>
> **函数的返回值是：返回替换后的字符串**
>
> 示例：
>
> ```makefile
> OBJS := $(patsubst %.c, %.o, foo.c bar.c)
> ```
>
> 其效果是：将字符串 foo.c、bar.c 中符合模式 %.c 的单词替换成 %.o，返回结果为 foo.o 和 bar.o
>
> <span style=color:red;background:yellow>**注意：该函数的参数部分是以空格为间隔的，只需要一个空格作为间隔符，不要添加过多的空格！！比如下图中的用法就是错误的：**</span>
>
> ![Makefile函数空格问题-图](https://hixiaodong123.oss-cn-hangzhou.aliyuncs.com/typora/image-20240606212255549.png#padding#600w)
>
> 也就是说，相当于变量赋值：
>
> ```` makefile
> OBJS := foo.o bar.o
> ````
>
> 于是我们结合这两个内置函数，就可以将Makefile脚本改成下面的样子：
>
> ```makefile
> OUT := main
> SRCS := $(wildcard *.c) #将当前目录下的所有.c文件的文件名以空格分割，然后赋值给SRCS变量
> OBJS := $(patsubst %.c,%.o,$(SRCS)) #获取当前目录下所有.c文件对应的.o文件，以空格分割
> COM_OP := -Wall -g 
> CC := gcc        #修改CC的默认值
> 
> $(OUT):$(OBJS)
>     $(CC) $^ -o $@
> %.o : %.c compute.h
>     $(CC) -c $< -o $@ $(COM_OP)
> 
> .PHONY: clean rebuild
> clean:
>     $(RM) $(OUT) $(OBJS)
> rebuild: clean $(OUT)
> ```
>
> 以上，我们就的得到了一个极具通用性的Makefile脚本。
>
> 试想一下：
>
> 如果我再添加一个`div`运算，创建.c文件，修改main函数，需要对该Makefile脚本做任何修改吗？
>
> <span style=color:red;background:yellow>**答：不需要！**</span>
>
> 后续如果希望修改这个Makefile，那就需要搞清楚这个Makefile所对应的编译和链接过程的代码在哪里。
>
> 实际上：
>
> ```` makefile
> $(OUT):$(OBJS)
>     $(CC) $^ -o $@		#链接过程
> %.o : %.c compute.h
>     $(CC) -c $< -o $@ $(COM_OP)	#编译过程
> ````
>
> 所以如果想要修改这个脚本，可以在上述位置修改。
>
> 总的来说，只要是在同一个目录中，存在一个主函数，多个.c文件或.h文件最终生成一个可执行程序，这个Makefile就非常好用且通用。
>
> <span style=color:red;background:yellow>**注意：不要求大家掌握这个Makefile的写法，但要求大家会用，会改就行了。**</span>

# 扩展：多个可执行程序的通用Makefile

> _~Gn!~_
>
> 我们上面实现的Makefile是在一个目录下生成一个可执行程序，但在某些场景下(比如写单元测试)，我们可能需要在一个目录下生成多个可执行程序。
>
> 如何实现这个Makefile呢？要求尽量提高通用性。
>
> 每一位同学都应该可以写出一个固定形式的基础Makefile，脚本代码如下所示：
>
> ```` makefile
> #此Makefile可以自动生成test.c、test2.c、test3.c各自对应的可执行程序
> .PHONY: all clean rebuild
> 
> # 使用伪目标all作为第一个明确目标，表示此Makefile所要生成的所有目标
> # 使用all伪目标列出Makefile所需要生成的所有目标，是一个惯用法
> all: test test2 test3
> 
> test: test.c
> 	gcc test.c -o test  
> test2: test2.c
> 	gcc test2.c -o test2  
> test3: test3.c
> 	gcc test3.c -o test3 
> 
> clean:
> 	rm -f test test2 test3
> 
> rebuild: clean all
> ````
>
> 但上面的Makefile显然没有什么通用性，为了最好的通用性，可以参考下列脚本代码：
>
> ```` makefile
> SRCS := $(wildcard *.c)
> OUTS := $(patsubst %.c,%,$(SRCS))
> CC := gcc
> COM_OP := -Wall -g 
> .PHONY: clean rebuild all
> 
> all: $(OUTS)                      
> % : %.c
> 	$(CC) $^ -o $@ $(COM_OP)
> 	
> clean:
> 	$(RM) $(OUTS)
> 		
> rebuild: clean all
> ````
>
> 需要解释地方：
>
> 定义了一个伪目标`all`，它依赖于变量`OUTS`中定义的所有目标（即当前目录下所有`.c`文件，最终生成的可执行文件）。这是一个常见的约定目标，用来构建所有的可执行文件。
>
> <span style=color:red;background:yellow>**在后续的课程学习中，我们将使用这个Makefile来进行自动化的构建可执行程序，简化学习过程中重复繁琐的构建指令。**</span>

# The end