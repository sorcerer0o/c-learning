> ~(Gn!)~
>
> 很多同学对重定向stdout到文件，这个过程中的一些程序输出结果，颇有问题。
>
> 我们直接来看一段代码：
>
> ```` c++
> int main(int argc, char *argv[])
> {
>       ARGS_CHECK(argc, 2);
>       int fd=open(argv[1],O_WRONLY|O_CREAT|O_TRUNC,0664);
>       ERROR_CHECK(fd,-1,"open");
> 
>       //先备份标准输出的文件描述符
>       int temp=dup(STDOUT_FILENO);
>       //将标准输出重定向指向文件
>       dup2(fd, STDOUT_FILENO);
>       printf("How are you?\n");
>       //将标准输出重新终端
>       dup2(temp, STDOUT_FILENO);
>       printf("I'm fine, and you?\n");
>       //再将标准输出重定向到文件
>       dup2(fd, STDOUT_FILENO);
>       printf("Me too.\n");
> 
>       close(fd);
>       return 0;
> }
> ````
>
> 按照我们的理解，这段代码将使得标准输出在终端和文件之间"横跳"：
>
> 1. "how are u\\n"将在文件中输出
> 2. "I'm fine, and you?\\n"将在终端打印
> 3. "Me too.\\n"又将在文件中输出
>
> <font color=red>**但实际上，这三段文本都将输出到文件，这是为什么呢？**</font>
>
> 这就涉及到操作系统、C标准库对标准输出缓冲区模式的管理了，<span style=color:red;background:yellow>**第一次使用stdout时的上下文（它指向的是终端还是文件）将决定stdout的缓冲区刷新机制，并且在整个进程生命周期，这个刷新机制将默认保持不变：**</span>
>
> 1. 如果第一次使用stdout时它指向一个终端（比如用printf），此时缓冲区就是行缓冲的。
> 2. 如果第一次使用时stdout被重定向到文件或其他非终端设备，缓冲区就变成了满缓冲区，这意味着缓冲区会积累输出直到满了或者手动刷新。
>
> 比如上面的代码：
>
> 虽然重定向都成功了，但由于stdout缓冲区的性质是满缓冲区，所以只能在最终进程结束时，刷新缓冲区，将所有内容全部输出到文件中。
>
> 尝试将代码改成下面的样子：
>
> ```` c++
> int main(int argc, char *argv[])
> {
>     ARGS_CHECK(argc, 2);
>     int fd=open(argv[1],O_WRONLY|O_CREAT|O_TRUNC,0664);
>     ERROR_CHECK(fd,-1,"open");
> 
>     //先备份标准输出的文件描述符
>     int temp=dup(STDOUT_FILENO);
>     //将标准输出重定向为文件描述符对应的文件
>     dup2(fd, STDOUT_FILENO);
>     printf("How are you?\n");
>     // 手动刷新缓冲区
>     fflush(stdout);
>     //将原来的标准输出文件描述符恢复给标准输出
>     dup2(temp, STDOUT_FILENO);
>     printf("I'm fine, and you?\n");
>     // 手动刷新缓冲区
>     fflush(stdout);
> 
>     //再将文件的文件描述符赋给标准输出
>     dup2(fd, STDOUT_FILENO);
>     printf("Me too.\n");
>     // 手动刷新缓冲区
>     fflush(stdout);
> 
>     close(fd);
>     return 0;
> }
> ````
>
> 即每一个printf输出后面都加上一个手动刷新缓冲区的`fflush`操作，此时程序的行为将符合大家的预期：
>
> 1. "how are u\\n"将在文件中输出
> 2. "I'm fine, and you?\\n"将在终端打印
> 3. "Me too.\\n"又将在文件中输出
>
> 再来看一段代码：
>
> ```` c
> int main(int argc, char *argv[])
> {
>     ARGS_CHECK(argc, 2);
>     int fd=open(argv[1],O_WRONLY|O_CREAT|O_TRUNC,0664);
>     ERROR_CHECK(fd,-1,"open");
> 
>     // 先printf输出一句话
>     printf("start.\n");
>     //先备份标准输出的文件描述符
>     int temp=dup(STDOUT_FILENO);
>     //将标准输出重定向指向文件
>     dup2(fd, STDOUT_FILENO);
>     printf("How are you?\n");
>     //将标准输出重新终端
>     dup2(temp, STDOUT_FILENO);
>     printf("I'm fine, and you?\n");
>     //再将标准输出重定向到文件
>     dup2(fd, STDOUT_FILENO);
>     printf("Me too.\n");
> 
>     close(fd);
>     return 0;
> }
> ````
>
> 这段代码和第一段代码，只有一个区别，在程序一开头就使用`printf`输出一句话，于是<span style=color:red;background:yellow>**标准输出的缓冲区就是一个行缓冲区**</span>。
>
> 此时程序的行为将符合大家的预期：
>
> 1. 终端显示：start、I'm fine, and you
> 2. 文件中：How are you、Me too
>
> 那么再改一下代码：
>
> ```` c++
> int main(int argc, char *argv[])
> {
>     ARGS_CHECK(argc, 2);
>     int fd=open(argv[1],O_WRONLY|O_CREAT|O_TRUNC,0664);
>     ERROR_CHECK(fd,-1,"open");
> 
>     // 先printf输出一句话
>     printf("start.");
>     //先备份标准输出的文件描述符
>     int temp=dup(STDOUT_FILENO);
>     //将标准输出重定向指向文件
>     dup2(fd, STDOUT_FILENO);
>     printf("How are you?");
>     //将标准输出重新终端
>     dup2(temp, STDOUT_FILENO);
>     printf("I'm fine, and you?\n");
>     //再将标准输出重定向到文件
>     dup2(fd, STDOUT_FILENO);
>     printf("Me too.\n");
> 
>     close(fd);
>     return 0;
> }
> ````
>
> 注意第一个换行符的位置：在`printf("I'm fine, and you?\n");`这一行
>
> 此时程序的输出结果是什么？
>
> 答：
>
> 1. 终端输出：前三句
> 2. 文件内容：最后一句
>
> 你懂了吗？