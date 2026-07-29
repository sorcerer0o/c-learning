###### <sub><font color = orange>王道C++班级参考资料</font></sub><br />——<br />Linux部分<sup><font color=white>卷3</font></sup>文件系统编程<br/><sup><sub><font color=cyan>节1</font></sub><font color=cyan>HelloWorld</font></sup><br/><br/>*最新版本`V3.0`*<br>**王道C++团队**<br/>*COPYRIGHT ⓒ 2021-2024. 王道版权所有*

[TOC]

# 概述

> _~Gn!~_
>
> 到此为止，我们已经：
>
> 1. 学完了Linux基础，已经基本会使用常见的shell指令，对Vim编辑器的使用也已基本入门；
> 2. 紧接着我们也学习了GNU工具集，熟悉了编译工具链，学会了如何使用GDB来调试程序，了解了库文件，懂得了C程序的工程管理工具Makefile；
> 3. 现在我们将正式进入 Linux 系统编程的阶段，开始 Linux 环境下的 C 语言编程之旅。
>
> 就像我们第一次在图形界面上编写 “Hello World” C 程序一样，我也将本节命名为 “HelloWorld”，象征着我们在 Linux 上进行 C 程序开发的新征程，新的开始。
>
> 在本节中，将要给大家讲解如何进行环境的配置，一些常见的代码规范，以及如何高效地利用工具在 Linux 环境下进行 C 程序的开发。

# 创建公共头文件

> _~Gn!~_
>
> 在后续的课程中：
>
> 1. 我们会学习很多库函数，系统调用库函数，这样就需要包含很多不同的标准库头文件；
> 2. 其次，我们可能需要定义很多宏，类型，别名等，这些内容如果每次编码都重复进行定义，那就太折磨了。
>
> 于是我们自然而然就想到了一个解决思路：
>
> <span style=color:red;background:yellow>**利用一个公共头文件，将以上内容装进这个公共头文件，于是每次写代码就只需要包含这个公共头文件就可以了。**</span>
>
> 那么具体怎么做呢？
>
> Linux系统存放系统头文件的目录是`/usr/include`，我们可以使用sudo权限在该目录下新建一个头文件，比如叫：
>
> > my_header.h
>
> 然后不要忘记给此文件的其它用户加上<font color=red>**写权限**</font>，这样后续就不用sudo权限就可以直接进行修改了。后续我们就可以把所有的头文件包含，宏定义等内容放到这个公共头文件中了。
>
> 在一开始，我们建议这个头文件的内容如下：
>
> ```` c
> #ifndef MY_HEADER_H
> #define MY_HEADER_H
> 
> #include <stdio.h>
> #include <stdlib.h>
> #include <string.h>
> #include <time.h>
> #include <ctype.h>
> #include <stdbool.h>
> 
> #endif
> ````
>
> 最后一个问题：现在如何包含这个公共头文件呢？
>
> 很简单：
>
> ```` c
> #include <my_header.h>
> ````
>
> 思考一下：为什么可以用`<>`，而不是`""`呢？
>
> 因为这个自定义头文件已经被我们放入系统头文件目录当中去了，所以可以使用`<>`进行头文件包含了。
>
> 注：
>
> <font color=red>**在系统库文件目录下新建头文件，不是一个标准推荐的做法，这里这么做是为了让大家在学习的过程中更方便，不用去管理琐碎的头文件以及一些常用代码片段。**</font>
>
> **等实际到了公司，还是要参考公司标准规范的做法，不要乱搞。**

# 设置.c文件模板

> _~Gn!~_
>
> 如果安装了Vim-plus插件，那么它默认使用文件：`~/.vim/plugged/prepare-code/snippet/snippet.c`作为`vim *.c`新建.c文件的模板。
>
> 于是我们就可以通过修改这个`snippet.c`文件的内容，改变默认生成的.c文件的内容，这个操作就非常类似于我们在VS中配置代码模板的操作是一样的。
>
> 当下的话，我们建议大家先把模板配置成：
>
> ```` c
> #include <my_header.h.h>
> 
> /* Usage:  */
> int main(int argc, char *argv[]){                                  
>        printf("Hello world\n");
>        return 0;
> }
> ````
>
> <span style=color:red;background:yellow>**注意，要将模板中的头文件包含设置为包含自定义的公共头文件。**</span>
>
> 注意，如果你直接复制路径找不到这个目录，可以使用指令：
>
> ```` shell
> find ~ -name snippet.c
> ````
>
> 在家目录下去寻找`snippet.c`文件，然后修改模板文件，若还找不到可直接询问老师。
>
> 这个文件模板，你可以随着我们的课程进度的推进，自己自由的进行修改，不用拘泥于老师给定的模板。

# Makefile

> _~Gn!~_
>
> 利用公共头文件，我们基本上不需要在当前目录下创建头文件了，在学习阶段时，我们完全可以用一个或几个.c文件完成所有任务。
>
> 所以我们可以创建一个Makefile如下：
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
> 这个Makefile会根据当前工作目录下的每一个`.c`文件，都生成一个对应的可执行程序。
>
> 把它放在一个固定的目录下，每次创建新目录写代码需要Makefile构建工具时，就把它拷贝过去，非常好用！
>
> 比如：
>
> 我会在`~/tools`目录下创建一个`Makefile`文件，然后把上述代码拷贝过去。每次需要Makefile时，就使用以下指令复制到当前目录下：
>
> ```` shell
> cp ~/tools/Makefile .   #.代表当前目录
> ````
>
> 你也去试试吧，非常好用。
>
> <font color=red>**当然，这个Makefile脚本会将当前目录下的每一个.c文件都生成对应的可执行程序，如果你需要多个.c文件生成一个可执行程序，该Makefile是实现不了的。**</font>

# 命令行参数

> _~Gn!~_
>
> 命令行参数的语法在C阶段时，我们就已经在《字符串》章节中学习过了，这里我们再复习一下。
>
> 所谓命令行参数，可以这样理解：
>
> <span style=color:red;background:yellow>**当你在命令行中启动一个C程序时，你在指令输入窗口输入的文本（即命令行参数）就由操作系统接收。操作系统再将这些参数解析为一个字符串数组，并在启动程序时传递给这个C程序。**</span>
>
> 所以命令行参数的传递过程，实际上是下面的流程：
>
> 1. 用户在命令行中启动C程序，传递给操作系统一个命令行参数字符串
> 2. 然后由操作系统解析这个字符串，解析成一个字符串数组
> 3. 最后操作系统启动C程序时，将字符串数组传递给这个C程序
>
> 标准C语言规定：
>
> 1. C程序可以不接收操作系统传递的命令行参数字符串数组，此时可以把mian函数声明为：
>
>    ```` c
>    int main(void);
>    ````
>
> 2. C程序若希望接收命令行参数字符串数组，则需要把main函数声明为：
>
>    ```` c
>    int main(int argc, char *argv[]);
>    ````
>
> 这里有几个问题：
>
> 1. 用户在命令行中启动程序，以什么格式给操作系统传递参数字符串呢？
> 2. 操作系统如何解析参数字符串，生成一个字符串数组呢？
>
> <font color=red>**第一个问题的答案是：**</font>
>
> 以Linux的命令行为例，假如要启动一个可执行程序`main`，并希望额外传递两个参数`test`和`test2`
>
> 那你就可以这样启动这个C程序：`./main test test2`
>
> <font color=red>**第二个问题的答案是：**</font>
>
> 操作系统接收到用户输入的命令行参数字符串后，会以空格为间隔，来分割每一个命令行参数，将用户输入的命令行字符串解析成一个字符串数组。
>
> 比如接收的命令行参数字符串是：`./main test test2`
>
> 最终就会解析成字符数组：["./main", "test", "test2"]
>
> 这个字符串数组就会被传递给main函数的形参argv字符串数组，当然操作系统还会传递该数组的长度，长度被main函数形参argc接收。
>
> 举例说明，对于下列代码：
>
> ```` c
> #include <stdio.h>
> 
> int main(int argc, char *argv[]){
>     printf("argc = %d\n", argc);
>     for(int i = 0; i < argc; i++){
>         printf("第%d个命令行参数是:%s\n", (i + 1), argv[i]);    
>     }
>     return 0;
> }
> ````
> 
>在当前目录启动这个C程序，指令是：`./main 100 200 test`，那么程序的输出是：
> 
>> argc = 4
> >
> > 第1个命令行参数是:./main
> >
> > 第2个命令行参数是:100
> >
> > 第3个命令行参数是:200
> >
> > 第4个命令行参数是:test
> 
><span style=color:red;background:yellow>**注意：命令行参数的第一个永远是可执行程序的路径名，从第二个参数开始才是可以自由给定的。**</span>

# HelloWorld

> _~Gn!~_
>
> 以上，完事具备，我们就可以正式的开始编写我们的第一个Linux环境下的C程序了。
>
> 我们就以C阶段最后学习的文件流为载体，利用C语言的标准库函数，完成一个带有命令行参数的文件复制功能。
>
> 其函数声明如下：
>
> ```` c
> void copy_file(const char *src, const char *dest);
> ````
>
> 库函数是具有跨平台性的，所以这段代码的写法和以往没有什么区别，如下所示：
>
> ```` c
> #include <my_header.h>		// 只需要包含公共头文件就可以了
> 
> // 将src文件复制到目标dest,若dest存在则直接覆盖
> void copy_file(const char *src, const char *dest) {
>        FILE *src_fp = fopen(src, "rb");		// b在Linux平台下加不加都无所谓
>        if (src_fp == NULL) {
>            // 将参数拼到当前errno的错误信息上
>            perror("fopen src");
>            exit(1);
>        }
> 
>        FILE *dest_fp = fopen(dest, "wb");
>        if (dest_fp == NULL) {
>            perror("fopen dest");
>            fclose(src_fp);  // 关闭已打开的源文件
>            exit(1);
>        }
> 
>        // 用于临时存储数据的中转站
>        char buf[1024] = { 0 }; 
>        size_t count;
> 
>        // 从源文件读取数据并写入目标文件
>        while ((count = fread(buf, 1, sizeof(buf), src_fp)) > 0) {
>            fwrite(buf, 1, count, dest_fp);
>        }
> 
>        // 关闭文件流
>        fclose(src_fp);
>        fclose(dest_fp);
> }
> 
> int main(int argc, char *argv[]) {
>        // ./00_copy_file src dest
>        if (argc != 3) {
>            fprintf(stderr, "args error!\n");
>            exit(1);
>        }
>        copy_file(argv[1], argv[2]);
>        return 0;
> }
> ````
>
> 这段代码仍然可以做一些优化，比如：
>
> 1. 在带命令行参数的main函数中，判断参数个数是否正确是必须要做的
> 2. `fopen`这类函数需要通过返回值来判定函数是否正确执行，需要对返回值做校验处理
>
> 我们可以把这部分代码提取出一段宏函数，然后加到公共头文件中，这样以后代码再有此类操作就可以很方便了。
>
> 于是我们的公共头文件变成了：
>
> ```` c
> #ifndef MY_HEADER_H
> #define MY_HEADER_H
> 
> #include <stdio.h>
> #include <stdlib.h>
> #include <string.h>
> 
> // 检查命令行参数数量是否符合预期
> #define ARGS_CHECK(argc, expected) \
>     do { \
>         if ((argc) != (expected)) { \
>             fprintf(stderr, "args num error!\n"); \ 
>             exit(1); \       
>         } \                          
>     } while (0)                      
> 
> // 检查返回值是否是错误标记,若是则打印msg和错误信息
> #define ERROR_CHECK(ret, error_flag, msg) \
>     do { \
>         if ((ret) == (error_flag)) { \
>             perror(msg); \
>             exit(1); \
>         } \
>     } while (0)
> 
> #endif
> ````
>
> 然后，我们就可以将这个C程序的实现改为：
>
> ```` c
> #include <my_header.h>
> 
> // 将src文件复制到目标dest,若dest存在则直接覆盖
> void copy_file(const char *src, const char *dest) {
>     // ./01_copy_file src dest
>     FILE *src_fp = fopen(src, "rb");
>     // 对返回值进行错误处理
>     ERROR_CHECK(src_fp, NULL, "fopen src");
> 
>     FILE *dest_fp = fopen(dest, "wb");
>     if (dest_fp == NULL) {
>         perror("fopen dest");
>         fclose(src_fp);  // 关闭已打开的源文件
>         exit(1);
>     }
> 
>     // 用于临时存储数据的中转站
>     char buf[1024] = { 0 }; 
>     size_t count;
> 
>     // 从源文件读取数据并写入目标文件
>     while ((count = fread(buf, 1, sizeof(buf), src_fp)) > 0) {
>         fwrite(buf, 1, count, dest_fp);
>     }
> 
>     // 关闭文件流
>     fclose(src_fp);
>     fclose(dest_fp);
> }
> 
> int main(int argc, char *argv[]) {
>     ARGS_CHECK(argc, 3);
>     copy_file(argv[1], argv[2]);
>     return 0;
> }
> ````
>
> 以上。
>
> 也就是说：在后续写代码过程中，如果有对命令行参数进行校验、对返回值进行校验的需求，就都可以直接复用这个宏。
>
> 最后思考一个问题：为什么宏定义中要使用一个`do...while`结构呢？

> _~Rd!~_
>
> 之所加这样的一个`do...while`结构是为了避免极端情况导致的问题。比如下列写法，如果不用`do...while`结构的宏函数就会导致悬空else行为，导致程序的编译失败：
>
> ```` c
> if(1)
>        ARGS_CHECK(argc, 3);   
> else
>        printf("hello world!");
> ````
>
> 如果使用`do...while`结构的宏函数，则不会出现这种问题。
>
> 但实际上，这种错误也可以避免，只需要不省略大括号就可以了：
>
> ```` c
> if(1){
>        ARGS_CHECK(argc, 3);
> }
> else{
>        printf("hello world!");
> }
> ````
>
> 总之`do...while(0)` 结构提供了一个在宏定义中增强安全性和健壮性的方式，尤其是在无法控制使用者如何使用宏的情况下。

# The End