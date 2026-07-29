###### <sub><font color = orange>王道C++班级参考资料</font></sub><br />——<br />Linux部分<sup><font color=white>卷2</font></sup>GNU工具集<br/><sup><sub><font color=cyan>节2</font></sub><font color=cyan>GDB调试程序</font></sup><br/><br/>*最新版本`V3.0`*<br>**王道C++团队**<br/>*COPYRIGHT ⓒ 2021-2024. 王道版权所有*

[TOC]

# 什么是调试程序？

> _~Gn!~_
>
> 代码写完了，可执行程序也成功生成了，但这并不意味着就万事大吉，直接完工并开始摸鱼了。在大多情况下，我们还需要调试程序。
>
> 那么什么是调试程序呢？
>
> <span style=color:red;background:yellow>**调试程序的本质，其实是信息确认的过程。**</span>
>
> 如何理解"信息确认"呢？
>
> 说得更清楚一点，调试程序实际上就是：
>
> 程序员先预判程序执行的行为和状态，然后再去和程序真实的行为和状态进行对比，确认信息。如果这个过程中，出现了不符合预期的意外情况，那么就有两种可能：
>
> 1. 程序员的思路出了问题，求解方式一开始就错了，代码写对了也没用。
> 2. 思路没问题，代码的逻辑写错了，或者代码的语法、<font color=red>**细节(比如边界值)**</font>等出现了问题。
>
> <span style=color:red;background:yellow>**所以调试程序的最终目的，就是通过信息确认，判断出是"人的思路错了"，还是"人的代码写错了"，从而进行改正。**</span>
>
> 最后我们请大家思考一个问题，当你看了上面的描述后，你觉得调试程序的前提是什么？
>
> 实际上，程序员能够调试程序的前提是：
>
> 1. 程序员要有清晰的求解思路，能够搞清楚自己想做什么
> 2. 程序员还要能够掌控自己的代码，能够预判程序的状态和行为
>
> 所以大家会发现很多初级程序员是不会调试程序的，因为它们缺乏清晰的逻辑以及对代码的掌控力，而这些能力是需要我们在不断的学习过程中去加强的。
>
> 在以往Windows的环境下，我们可以借助VS的图形界面Debug模式来调试程序。在Linux环境下，我们也有类似的调试工具，那就是<span style=color:red;background:yellow>**GDB**</span>，这一小节我们就主要学习使用GDB来调试程序。

# GDB是什么？

> _~Gn!~_
>
> GDB（GNU Debugger）是GNU项目的一部分，虽然GDB是一个跨平台的支持多种语言的调试工具，但它主要还是用于在Linux操作系统下调试C/C++编程语言。
>
> 在Linux系统环境下，GDB可以基本上实现Windows下VS的Debug功能，比如：
>
> 1. 断点设置
> 2. 逐语句
> 3. 逐过程
> 4. 监视窗口(查看/修改变量取值)
> 5. 查看内存
> 6. ....
>
> GDB调试程序的过程基本上和VS调试过程差不多，仍然可以总结为——<span style=color:red;background:yellow>**"停一停、瞧一瞧，再走一走、再看一看"**</span>。
>
> 总得来说，GDB肯定没有VS的调试功能好用(图形界面毕竟非常方便)，但在Linux环境下GDB已经是比较好的选择了。而且GDB作为Linux环境下，C/C++程序员调试代码的常用工具，也是实际面试中比较常问常考的话题。

# GDB快速入门

> _~Gn!~_
>
> 在下面的小节中，我们将快速演示如何使用GDB调试一个程序，即GDB快速入门。<span style=color:red;background:yellow>**但后续的提升和熟练使用，还需要大家多多练习。**</span>
>
> 若你还未安装GDB，可以使用下列指令进行安装：
>
> ```` shell
> sudo apt install gdb
> ````
>
> 这里给出一个用于GDB学习的简单代码，如下：
>
> ```` c
> #include <stdio.h>
> 
> // 函数用于计算数组元素的平均值
> double compute_average(int *arr, int len) {
>        int sum = 0;
>        for (int i = 0; i <= len; i++) { // 注意这里有一个错误
>            sum += arr[i];
>        }
>        return sum / (double)len;
> }
> 
> int main(void) {
>        int nums[] = {1, 2, 3, 4, 5};
>        int len = sizeof(nums) / sizeof(nums[0]);
>        double average = compute_average(nums, len);
>        printf("The average is: %f\n", average);
> 
>        // 打印5行3列的$图案                                        
>        for(int i = 0; i < 5; i++){
>            for(int j = 0; j < 3; j++){
>                printf("$");
>            }
>            printf("\n");
>        }
>        return 0;
> }
> ````
>
> 这个代码是存在Bug的，我们可以通过GDB调试找出这个Bug。

## 第一步：带调试信息编译代码

> _~Gn!~_
>
> 编译过程会去掉代码中诸如变量名这样的调试信息，从汇编代码开始这些调试信息就被替换成了内存地址。
>
> 所以要想使用GDB调试程序。首先第一步就是：<span style=color:red;background:yellow>**使用带"-g"的指令编译生成可执行程序。**</span>
>
> 比如指令：
>
> ```shell
> gcc hello.c -o hello -g -O0 -Wall		#-g为必加选项
> ```
>
> 注意：
>
> 1. 调试程序时最好不要设置编译优化，一般保持默认就可以了，或者主动加上`-O0`选项。
> 2. 这一步完成后，你会得到一个带调试信息的可执行程序`hello`，当然这个可执行程序也是可以正常启动执行的。

## 第二步：进入GDB调试界面

> _~Gn!~_
>
> 进入GDB调试界面（控制台），可以认为是<span style=color:red;background:yellow>**用GDB启动(调试)这个可执行程序**</span>，有以下两种方式：
>
> ```` SHELL
> # 第一种方式直接将可执行程序文件名作为参数
> gdb 可执行程序名字
> # 第二种需要先进入gdb控制台
> gdb 
> file 可执行程序名字		# 进入gdb控制台后再使用file指令
> ````
>
> 注意：第二种方式是先进入gdb控制台，再选择调试的可执行程序，如下图所示：
>
> ![进入GDB调试界面-图](https://hixiaodong123.oss-cn-hangzhou.aliyuncs.com/typora/202403022220544.png#padding#600w)
>
> 成功进入GDB调试界面后，就可以使用各种指令进行程序的调试了，首先我们介绍<span style=color:red;background:yellow>**一个最基础的指令——查看源代码**</span>。
>
> 它的格式如下：
>
> ```` shell
> list/l	[文件名:][行号|函数名]
> ````
>
> <span style=color:red;background:yellow>**注意，大多数GDB指令都有长短两种写法，推荐直接使用短指令形式！**</span>
>
> 一些使用举例：
>
> ```` shell
> (gdb) l 							# 下翻源代码
> (gdb) l -							# 上翻源代码
> (gdb) l 20							# 查看启动程序20行附近的源代码
> (gdb) l main						# 查看启动程序main函数附近的源代码
> (gdb) l main.c:20					# main.c文件第20行附近的源代码
> (gdb) l main.c:main					# main.c文件main函数附近的源代码
> ````
>
> 建议直接使用`l`简写的指令，没必要使用list这样的长指令。除此之外，还有一些比较零碎的指令，就都放这里给出了：
>
> ```` shell
> run/r 		#相当于在VS中点击以Debug模式启动的按钮，一般用于结束一次调试后重启调试或者中断并重启调试
> kill/k  	#停止当前正常调试的程序，但不会退出GDB 
> quit/q  	#直接退出GDB，当然调试也会立刻结束
> ````
>
> 直接执行run指令，发现程序会直接执行完，这是正常的。因为我们还没有打断点！

## 第三步：打断点

> _~Gn!~_
>
> 在VS当中调试程序起始于打断点，GDB中也是一样的，打断点可以使用以下指令：
>
> ```` shell
> break/b   [文件名:][行号|函数名]			#在某个位置设置一个普通的、持续生效的断点
> tbreak/tb [文件名:][行号|函数名]			#在某个位置设置一个只生效一次的一次性断点
> ````
>
> 常见用法举例：
>
> ```` shell
> (gdb) b 20							# 在第20行设置断点
> (gdb) b main						# 在main函数的开头设置断点
> (gdb) b main.c:20					# main.c文件的第20行设置断点
> (gdb) b main.c:main					# 在main.c文件的main函数开头设置断点
> ````
>
> 设置好断点后，可以使用以下指令来查看断点信息：
>
> ```` shell
> info break/i b						#可以省略到只剩下i b，但空格不能省略，不能使用ib
> ````
>
> 断点信息的输出结果可能如下：
>
> ![断点信息-图](https://hixiaodong123.oss-cn-hangzhou.aliyuncs.com/typora/image-20240807224051056.png#padding#600w)
>
> 这段信息从左往右内容是：
>
> 1. Num：断点的唯一性标识编号，一次Debug过程断点编号会从1开始，且不会重置一直累加。
> 2. Type：断点的类型，这里显示"breakpoint"，表示断点都只是普通断点。
> 3. Disp(Disposition，性格)：它表示断点触发后，是否会继续触发。
>    1. keep就表示它是一个持久的断点，只要不删除就一直存在，每次启动都会触发。
>    2. 这个值如果是**del**就表示，该断点是一个一次性断点。
> 4. Enb(enable)：指示断点是否生效，可以通过`dis`指令设置该属性。
> 5. What：指示断点在源代码的哪个位置
>
> 既然能打断点，那肯定也能删除断点，我们可以用 `delete/d` 指令删除断点：
>
> ```shell
> delete/d [n] 			#如果不加断点编号就是删除所有断点，若加上编号则表示删除n号断点
> ```
>
> 比如：
>
> ```shell
> (gdb) d 2				# 删除2号断点
> (gdb) d					# 如果不加断点编号，则表示删除所有断点
> ```
>
> 如果不想删除断点的话，也可以暂时让断点失效，成为一个"哑断点"，使用指令如下：
>
> ```` shell
> disable/dis	[n]			#使所有断点失效/单独使n号断点失效
> enable/en [n]			#使所有断点生效/单独使n号断点生效
> ````
>
> 打了断点后，就可以使用`run/r`指令来启动调试程序了，此时程序就会在断点处暂停运行，下面则是一些常用的调试命令。

## GDB常用调试指令

> _~Gn!~_
>
> <span style=color:red;background:yellow>**逐语句/单步调试**</span>
>
> step/s 命令可以用来进行单步调试，即遇到函数调用会进入函数。
>
> ```shell
> (gdb) step/s
> ```
>
> <span style=color:red;background:yellow>**跳出并执行完函数**</span>
>
> 我们可以使用 finish 命令执行完整个函数，并返回到函数调用处：
>
> ```shell
> (gdb) finish/fin
> ```
>
> 稍微需要注意的是：在main函数当中使用该指令是无效的，因为main一般处在调用栈的最底层。
>
> <span style=color:red;background:yellow>**逐过程**</span>
>
> next/n 命令表示逐过程，也就是说遇到函数调用，它不会进入函数，而是把函数调用看作一条语句直接执行完毕。
>
> ```shell
> (gdb) next/n
> ```
>
> <font color=red>**一般来说，我们更建议使用逐过程来控制程序继续执行，step可能会进入一些库函数的执行，这一般是不需要的。**</font>
>
> <span style=color:red;background:yellow>**监视窗口/查看变量取值**</span>
>
> print/p 命令可以打印表达式的值：
>
> ```shell
> print/p express				#后面直接跟一个表达式即可，你完全把这个功能当成Linux的计算器来使用
> ```
>
> 如：
>
> ```shell
> (gdb) p arr[0]			#输出查看当前a变量的值
> (gdb) p 3.14*2*2 		#作为计算器使用
> ```
>
> print/p 命令还可以改变变量的值，使用格式是：
>
> ```shell
> print/p express=val
> ```
>
> 比如：
>
> ```shell
> (gdb) p arr[0]=10
> ```
>
> <span style=color:red;background:yellow>**如果想要持续的，展示某个表达式的值，使用格式如下：**</span>
>
> ```shell
> display/disp express					# 每调试一步输出一次express的值
> undisplay/undisp [n]					# 删除所有或[n]号自动展示的表达式
> info display/i disp						# 显示所有自动展示的表达式信息
> ```
>
> <span style=color:red;background:yellow>**如果需要查看所有局部变量的值，局部变量窗口，使用格式如下：**</span>
>
> ```shell
> (gdb) info/i args					# 查看函数的参数
> (gdb) info/i locals 				# 查看函数所有局部变量的值
> ```
>
> <span style=color:red;background:yellow>**继续，跳过一次断点：**</span>
>
> continue/c 命令可以运行到逻辑上的下一个断点处：
>
> ```shell
> (gdb) c								#相当于VS当中的继续功能按钮
> ```
>
> <span style=color:red;background:yellow>**忽略断点n次**</span>
>
> 我们可以用 ignore 命令来指定忽略某个断点多少次，这在调试循环的时候非常有用。使用格式如下：
>
> ```shell
> ignore N COUNT
> ```
>
> 常见用法：
>
> ```shell
> (gdb) ignore 1 10			# 忽略1号断点10次
> ```
>
> 注意：
>
> 1. 设定忽略断点次数后，还需要按`c`继续跳过断点才会生效。
> 2. 设定一次，仅生效一次。
> 3. <font color=red>**跳过n次断点，那么程序会在(n + 1)次到达这个断点时停下来。**</font>
>
> <span style=color:red;background:yellow>**查看堆栈信息**</span>
>
> ```` shell
> bt/backtrace				#查看当前调用堆栈的信息，会一直追溯到程序启动
> ````
>
> 一般我们可以在报错的位置使用该指令，用于查看程序的执行流程以及排查相关的问题。
>
> <span style=color:red;background:yellow>**查看内存，内存窗口**</span>
>
> 我们可以用 x 命令查看内存的值，类似VS当中的内存窗口，虽然不如VS那么直观，但在某些场景中会有奇效。
>
> 基本使用格式如下：
>
> ```` shell
> x/(内存单元的个数)(内存数据的输出格式)(一个内存单元的大小) 数组名/指针/地址值...
> ````
>
> 该指令会从`数组名/指针/地址值..`开始，向后以`(内存数据的输出格式)`，展示`(内存单元的个数) * (一个内存单元的大小)`的内存数据。
>
> 其中：
>
> 内存单元的个数，直接输入一个整数即可。
>
> 内存数据的输出格式有：
>
> 1. o(octal)，八进制整数
> 2. x(hex)，十六进制整数
> 3. d(decimal)，十进制整数
> 4. u(unsigned decimal)，无符号整数
> 5. t(binary)，二进制整数
> 6. f(float)，浮点数
> 7. c(char)，字符
> 8. a(address)，地址值
> 9. c(character)：字符
> 10. s(string)，字符串
>
> 一个内存单元的大小的表示，有以下格式：
>
> 1. b(byte)，一个字节
> 2. h(halfword,  2 bytes)，二个字节
> 3. w(word, 4 bytes)，四个字节
> 4. g(giant, 8 bytes)，八个字节
>
> 演示代码案例：
>
> ````c
> #include <stdio.h>
> #define ARR_SIZE(arr) (sizeof(arr) / sizeof(arr[0]))
> 
> int main(void) {
>     int nums[] = {10, 20, 30, 40, 50};
> 	char* strs[] = {"abc", "123", "hello", "777", "666"};
>     int nums_len = ARR_SIZE(nums); 
>     int strs_len = ARR_SIZE(strs);
> 
>     // 断点可以打在这里下面然后用于查看内存
>     return 0;
> }
> ````
>
> 常见用法：
>
> ```shell
> # 自数组nums基地址起，每4个字节为一个内存单元，连续查看4个内存单元，以十进制整数的方式展示内存数据
> (gdb) x/4dw nums
> # 可能的输出结果如下所示：
> # 0x7fffffffe1f0:	10	20	30	40
> # 也就是说此int类型的nums数组，元素取值分别是10、20、30、40
> 
> # 自指针数组strs的基地址开始，每8个字节为一个内存单元，连续查看5个内存单元，以十六进制整数的方式展示内存数据
> # 由于使用64位系统生成64位程序，8个字节即为指针的大小，所以这里实际上是查看指针数组的每一个元素地址值，即每一个字符串元素的指针
> (gdb) x/5xg strs
> # 可能的输出结果如下所示：
> 	# 0x7fffffffe210:	0x0000555555556004	0x0000555555556008
> 	# 0x7fffffffe220:	0x000055555555600c	0x0000555555556012
> 	# 0x7fffffffe230:	0x0000555555556016
> # 冒号左边都是内存地址值，右边表示此内存地址上存储的地址（指针），指向一个字符串
> 
> # 你可以继续查看这些指针元素，指向的字符串内容
> (gdb) x/s 0x0000555555556004
> (gdb) x/s 0x0000555555556008
> # 可能的输出结果如下所示：
> # 0x555555556004:	"abc"
> # 0x555555556008:	"123"
> 
> # 对于字符串数组这样，需要解引用两次才能看到元素的，也可以使用下列方式来查看元素：
> (gdb) x/s *strs				# 查看第一个元素字符串
> (gdb) x/s strs[1]			# 查看第二个元素字符串
> (gdb) x/s *(strs + 1)		# 查看第二个元素字符串
> ```
>
> 注：如果忘记了查看内存指令如何使用，只需要使用`help x`查看帮助手册即可。
>
> <span style=color:red;background:yellow>**输入命令行参数**</span>
>
> 当main函数的形参列表是`int argc, char *argv[]`时，允许可执行程序传参命令行参数。如果想要使用GDB调试带命令行参数的可执行程序，有以下两种方式可以选择：
>
> 1. 在启动GDB时，使用指令`gdb --args ./a.out arg1 arg2 arg3...`即可表示传递命令行参数，其中a.out表示可执行程序的名字。
> 2. 如果已经启动了GDB，可以使用以下两种方式都可以传递命令行参数：
>    1. 使用指令`set args arg1 arg2....`，其中指令部分是`set args`，后面的部分则是参数。
>    2. 使用指令`run/r arg1 arg2`启动，也表示传递命令行参数。
>
> 以上，多多练习，熟悉后GDB用来调试程序，功能是完全足够的。

# 调试Coredump文件

> _~Gn!~_
>
> 如果代码压根跑不起来，或者跑起来就会直接崩溃，我们还可以利用<font color=red>**Core文件**</font>进行辅助调试。
>
> 通常情况下，程序异常终止时，会产生 Coredump 文件。Coredump 文件类似飞机上的"黑匣子"，它会保留程序"失事"瞬间的一些信息，通常包含寄存器的状态、栈调用情况等。
>
> Coredump 文件常用于辅助分析和 Debug，下面介绍一下这种调试手段。
>
> 首先，系统默认是不会生成Core文件的，这一点可以通过以下指令查看：
>
> ```` shell
> ulimit -a		#查看当前 shell 进程的各种资源限制，比如core文件最大大小、最大打开文件数、最大用户进程数等等。
> ````
>
> 默认情况下，该指令输出的第一行就是：
>
> > core file size          (blocks, -c) 0
>
> 表示此时系统允许生成的core文件最大是0个字节，即不允许生成。
>
> 所以我们需要用下列指令将core文件的大小设置为不受限制：
>
> ```` shell
> ulimit -c unlimited			# 将core文件的大小临时设置为不受限制 
> ````
>
> 默认情况下，上述操作后可能还是无法生成Core文件，你可以<font color=red>**切换到root用户或者使用sudo权限**</font>，然后补充一下core文件的配置信息到目标文件里。
>
> 具体的操作如下，先打开配置文件：
>
> ```` shell
> sudo vim /etc/sysctl.conf					# 打开配置文件，将下列信息补进去
> ````
>
> 将下列信息补充到配置文件末尾<span style=color:red;background:yellow>**（注意前面不要加#号）**</span>：
>
> ```` shell
> kernel.core_pattern = ./core_%e_%t			# %e:崩溃程序的名称, %t:崩溃时的时间戳
> ````
>
> 紧跟着你还需要执行以下指令让配置信息生效：
>
> ```` shell
> sudo sysctl -p								# 让配置文件生效，但每次重启会话连接可能都需要再执行一次
> ````
>
> 这段配置信息的目的是给core文件设定一个固定的格式，这样设置后，再次执行报错可执行程序就会生成core文件了。
>
> 这里给出几段用于调试错误的参考代码：
>
> ```` c
> int main(void){
>     int *p = NULL;
>     ++*p;		// 解引用空指针         
>     return 0;
> }
> 
> // -------------------------
> 
> int main(void){
>     int arr[] = {1, 2, 3};
>     printf("%d\n", arr[3]);	// 数组下标越界
>     return 0;
> }
> 
> // ----------------------------
> void test(int n){
>     return test(n - 1);
> }
> int main(void){
>     test(1);	// 栈溢出错误
>     return 0;
> }
> ````
>
> 使用这些代码正常生成带有调试信息的可执行程序，然后直接执行，就会在同目录下自动生成core文件了。
>
> 但是要注意：一般只有段错误才会生成对应core文件，像上面数组越界引发未定义行为是没有段错误的，也就不会生成core文件。
>
> 然后你就可以用指令：
>
> ```` shell
> gdb hello core_hello_1679196427		# gdb + 可执行文件的名字 + core文件名
> ````
>
> 查看报错的一些信息，此时再利用`bt`等指令就可以进行正常的程序调试了。
>
> 注：
>
> 实际上即便不依赖于core文件，直接在`gdb`中启动会报错的可执行程序，效果和使用core文件是一样的。比如：
>
> ![GDB调试错误信息-图](https://hixiaodong123.oss-cn-hangzhou.aliyuncs.com/typora/202403031348572.png#padding#600w)
>
> 接下来再通过查看堆栈信息、监视等功能，即可实现调试程序。
>
> 但在实际的生产环境中，你可能无法直接复现报错情况(或者很麻烦)。那么程序报错了，只能通过Core文件来检测程序的错误信息，进而修正代码。

# GDB练习

> _~Gn!~_
>
> 可以使用以下代码，利用GDB进行Debug调试的练习，找出并修改代码中的bug。
>
> ```` c
> #include <stdio.h>
> #include <stdlib.h>
> 
> typedef struct {
>        int value;
>        Node *next;
> } Node;
> 
> // 尾插法插入结点
> void add_before_tail(Node **head, int value) {
>        Node *new_node = (Node *)malloc(sizeof(Node));
>        new_node->value = value;
>        new_node->next = NULL;
>        if (*head == NULL) {
>            head = new_node;
>        } else {
>            Node *temp = *head;
>            while (temp != NULL) {
>                temp = temp->next;
>            }
>            temp->next = new_node;
>        }
> }
> 
> // 打印链表中的所有值
> void print_list(Node *head) {
>        Node *current = head;
>        while (current != NULL) {
>            current = current->next;
>            printf("%d -> ", current->value);
>        }
>        printf("NULL\n");
> }
> 
> int main() {
>        Node *head = NULL;
>        add_before_tail(&head, 1);
>        add_before_tail(&head, 2);
>        add_before_tail(&head, 3);
> 
>        print_list(head);
>        return 0;
> }
> ````
>
> 注意：
>
> 1. 编译代码时要增加`-Wall`选项，这样可以排查出代码的一些隐藏的问题。使用`gcc`编译器时，要重视代码中的警告信息，而不是无视。
> 2. gdb可以在随后的学习中，逐步练习，不要害怕更不要不想用这个东西。多练习才是王道！

# GDB练习2

> _~Gn!~_
>
> 可以使用以下代码，利用GDB进行Debug调试的练习，找出并修改代码中的bug。
>
> ````c
> #include <stdio.h>
> 
> /*
> 	双索引分区方法: 单向分区法
> 	用索引来模拟指针的作用
> 	思路:
> 	1.初始化第一个索引idx,用来遍历整个数组,idx从0开始
> 	2.再初始化一个索引odd_loc,从0开始,用来指示下一个奇数应该放置的位置
> 	3.在idx索引遍历数组的过程中,只要idx索引找到一个奇数,那就和odd_loc位置的元素交换
> 	4.当idx遍历完整个数组时,所有的奇数就都被交换到前面去了
> 
> 	优点: 遍历数组的次数更少效率更高, 不占用额外的临时空间
> 	缺点: 相对没那么好理解, 分区后改变了元素的相对位置，是一个不稳定的分区算法
> 	但和上面的双指针分区稍有区别的是: 奇数部分相对位置是可以保证的
> */
> void separate_odd_even(int arr[], int len) {
>     int odd_idx = 0;
>     for (int idx = 0; idx < len; idx++) {	// idx索引用于遍历数组
>         if (arr[idx] % 2 != 0) {	
>             // idx索引找到了一个奇数,就和odd_loc位置的元素交换
>             if (idx != odd_idx) {
>                 int tmp = arr[idx];
>                 arr[idx] = arr[odd_idx];
>                 arr[odd_idx] = tmp;
>                 odd_idx++;
>             }
>         }
>     }
> }
> 
> int main(void) {
>        int arr[] = { 1, 10, 22, 3, 132, 42, 23, 324, 5, 645, 10, 222 };
>        int len = sizeof(arr) / sizeof(arr[0]);
>        separate_odd_even(arr, len);
>        for (int i = 0; i < len; i++) {
>            printf("%d ", arr[i]);
>        }
>        printf("\n");
>        return 0;
> }
> ````
>
> 为了更好的调试这段代码，我们再讲两个常用的GDB功能：
>
> <span style=color:red;background:yellow>**利用display，持续显示数组特定范围的取值：**</span>
>
> ````shell
> display/disp arr[0]@len			# 持续显示数组arr从索引0开始的len个元素。
> ````
>
> 注意：len是一个表示数组长度的变量，必须在代码中明确给出的变量。
>
> <span style=color:red;background:yellow>**观察断点：**</span>
>
> ````shell
> watch/wa expression			# 设置一个断点，当expression的取值发生变化时，程序自动停止
> ````
>
> 如果你对数组中某个特定元素的变化很感兴趣，可以使用 `watch` 命令来设置断点监视该元素。
>
> 利用这两个功能，你可以很好的找到程序的bug，然后修正程序。

# The End