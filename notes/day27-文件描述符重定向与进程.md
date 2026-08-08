# Day27 - 文件描述符重定向与进程基础

## 文件流与文件描述符的关系

### fopen 底层就是 open
- fopen 是 ISO-C 库函数, open 是系统调用
- **fopen 在 Linux 中底层用到了 open**, fopen 得到的 FILE 内部存有文件描述符
- 验证: fopen 打开文件后, 用 `write(fileno(fp), ...)` 也能写文件

### fileno 获取文件描述符
```c
#include <stdio.h>
int fileno(FILE *stream);
```
- ISO-C 库函数, 返回文件流 (FILE*) 对应的文件描述符
- 传入已打开的文件流, 返回其文件描述符 int

### FILE 结构体 (更高层看 fd)
- FILE 结构体 (Linux 下实际是 _IO_FILE) 内部直接保存:**文件描述符成员 `_fileno`**
- 还可以通过 `fp->_fileno` 直接访问 (不推荐: 破坏封装, 缺乏跨平台性)
- 标准做法: 用 `fileno(fp)` 函数, 而不是直接访问内部成员
- 查看 FILE 结构体: 预处理后 .i 文件里 `/struct.*FILE` 搜索

### 程序自带三个标准流
- 每个程序启动时自动打开三个流: stdin / stdout / stderr, 类型都是 `FILE*`
- 对应文件描述符宏: `STDIN_FILENO`=0, `STDOUT_FILENO`=1, `STDERR_FILENO`=2
- `fileno(stdin)` / `fileno(stdout)` / `fileno(stderr)` 可打印验证

## 文件描述符的分配规则

- **系统分配 fd 从小到大**: 找到最小的空闲 (未使用) 的 fd 分配
- close 释放 fd 后, 这个 fd 会被**再次利用** (下次 open/dup 分到)
- 进程启动时 0/1/2 已被三个标准流占用, 所以第一个 open 通常得到 3

## close 与重定向的原理

### close 的原理
- close(1) 只关闭**文件描述符 → 文件对象**的映射关系, 不会真正销毁
- 文件对象采用**引用计数法**: 所有 fd 都被关闭后才释放文件对象
- 标准流对应文件对象 (0/1/2 指向的): 即使 fd 全关闭, 系统仍保留其文件对象 (等待重新分配 fd 指向它)

### 重定向的原理
- 重定向 = 断开/更改"标准流缓冲区 → 外部设备(屏幕/文件)"的连接
- 经典手法: close(1) → open 新文件 (分到 fd=1) → 以后 printf 写向文件
```c
printf("先向终端输出!\n");     // 重定向前先打印一句
close(STDOUT_FILENO);          // 关闭 fd 1
int fd = open(argv[1], O_RDWR); // 拿到最小的空闲 fd = 1, 指向文件
printf("fd = %d\n", fd);        // 这会写到文件里
```
- 为什么建议先 printf 一句? 让 stdout 先以**终端行缓冲**模式启动 (见缓冲区机制)

## dup / dup2: 复制文件描述符

### dup
```c
#include <unistd.h>
int dup(int oldfd);
```
- 作用: 向系统要一个新的文件描述符, 新 fd 和旧 fd **指向同一个文件对象**
- 系统分配规则: 从小到大分配**空闲**的 fd
- 返回值: 新 fd (必然 ≠ oldfd, 但指向同一个文件对象); 失败返回 -1

```c
int old_fd = open(argv[1], O_RDWR);   // old_fd = 3
int new_fd = dup(old_fd);              // new_fd = 4, 指向同一文件对象
write(old_fd, "Hello", 5);
write(new_fd, "World", 5);             // 最终文件内容: HelloWorld
close(new_fd);
close(old_fd);
```
- 两个 fd 指向同一文件对象 → **共享同一读写偏移量**, 写内容像续写

### 用 dup 重定向 stdout
1. open 一个文件 → fd (一般为 3)
2. close(STDOUT_FILENO) → fd 1 空出
3. dup(fd) → 分配最小空闲 = 1, 指向文件对象
4. 此后所有 stdout 输出都到文件

```c
printf("先输出一句!\n");
int fd = open(argv[1], O_RDWR);
close(STDOUT_FILENO);
int fd_cp = dup(fd);          // 分到 1, 重定向完成
printf("这行会进文件!\n");
fflush(stdout);               // 刷新, 避免丢数据
close(fd);
close(fd_cp);
```

### dup2
```c
#include <unistd.h>
int dup2(int oldfd, int newfd);
```
- 与 dup 的唯一区别: **可以自己指定复制出来的 fd 数字 (newfd)**
- 行为:
  - newfd 已被使用 → 先 close(newfd), 再让 newfd 指向 oldfd 的对象
  - newfd 空闲 → 直接让 newfd 指向 oldfd 的对象
- 返回值: 成功返回 newfd; oldfd 无效则返回 -1 且不关闭 newfd
- 相当于 "close + dup" 合并, 更简洁:

```c
int fd = open(argv[1], O_RDWR);
printf("先输出到终端!\n");
dup2(fd, STDOUT_FILENO);      // 一条搞定重定向 stdout 到文件
printf("这行会进文件!\n");
fflush(stdout);
close(fd);
```

### "反复横跳": 备份 + 重定向
- 想来回切换: 屏幕输出 ↔ 文件输出
- 先备份原始 stdout (dup2(1, 7)), 之后反复 dup2 切换
```c
int fd = open(argv[1], O_RDWR);
int bak = dup(STDOUT_FILENO);     // 备份 stdout 指向
dup2(fd, STDOUT_FILENO);          // 重定向到文件
printf("文件1\n");
dup2(bak, STDOUT_FILENO);         // 恢复 stdout
printf("屏幕1\n");
```

## stdout 缓冲机制 (重定向注意)

- 第 一次使用 stdout 时的**上下文**决定缓冲模式, 进程内不再改变:
  - 第一次指向终端/屏幕 → **行缓冲** (遇 \n 输出)
  - 第一次就被重定向到文件 → **满缓冲** (都在最后刷新)
- 重定向演示程序之所以"结果全进文件"是缓冲导致, 每个 printf 后加 `fflush(stdout)` 可立即刷新
- close 只中断映射, 缓冲内容不会立即输出; **不 fflush 等到进程结束时才输出**

## 进程基础

### 什么是进程
- 程序: 静态的磁盘文件
- **进程: 运行起来的程序** (有独立地址空间、栈等)

### 进程相关概念
- **PID / PPID**: 进程 ID / 父进程 ID
- 进程内部获取自己 pid/ppid: `getpid()` / `getppid()` (系统调用)
- **task_struct**: 内核中描述进程的结构体 (PCB)
- 进程状态: 运行/就绪/阻塞等
- 时间片: CPU 分时调度单位, 与 CPU 主频相关
- 调度策略: 分时调度等

### 进程相关 shell 命令
- `ps -elf` / `ps aux`: 查看当前进程
- `top`: 实时监控 CPU/内存; load average 依次为 1/10/15 分钟平均负载
- `kill -2` (SIGINT, Ctrl+C 同) / `kill -9` (强杀)

### 创建进程的三种方式
| 函数 | 标准 | 特点 |
|---|---|---|
| system | ISO-C 库函数 | 执行 shell 能执行的命令 (简单但开销大) |
| fork | 系统调用 | **创建一个新进程, 和新进程执行相同代码** |
| execl | POSIX 库函数 | 替换当前进程为另一个程序 |

### fork 系统调用 (重点)
```c
#include <unistd.h>
pid_t fork(void);
```
- **创建子进程**, 新进程和老进程执行**一样**的代码 (从 fork 返回处继续)
- **返回值是 pid_t 类型**, 用来区分父子进程:
  - **父进程中: 返回子进程的 PID**
  - **子进程中: 返回 0**
  - 失败: 返回 -1
```c
pid_t pid = fork();
if (pid == 0) {
    // 子进程分支
} else if (pid > 0) {
    // 父进程分支, pid 是子进程 PID
}
```

### fork 的写时复制 (COW)
- 刚开始 fork 时**并不会直接复制物理内存** (地址映射共享)
- 当**某个进程修改内存**时, 才把对应的**物理内存页**复制一份 (复制单位是**页**, 不是整个内存)
- 另一进程修改时, 直接使用原先位置的物理内存 (各自私有)
- 好处: fork 速度快、省内存; 只有确实需要写时才复制

### 进程的内存隔离
- fork 之后, 父子进程的内存是**隔离**的 (虚拟内存空间独立, 互不影响)
- 修改父子进程中的变量互不可见

### shell / fork / system 的关系
- shell 进程是父进程, 执行命令 (可执行程序) 时创建子进程
- C 程序里 system 调起 shell 执行命令: shell 等创建出的进程结束, **不会等这个进程的子进程** (子进程稍后还是会输出)

## 练习

1. 用 fileno 打印 stdin/stdout/stderr 的文件描述符
2. 用 close(1) + open 实现 stdout 重定向, 验证 printf 写进文件
3. 用 dup 复制 fd, 验证两个 fd 写入文件是连续内容 (共享偏移量)
4. 用 dup2 实现 stdout 重定向到文件
5. 用 dup + dup2 实现 stdout 在屏幕和文件间反复横跳
6. 写 fork 程序: 父进程打印 pid, 子进程打印 pid, 分别验证 getpid/getppid
7. fork 后用 child 修改变量, 父进程观察该变量是否变化 (验证内存隔离)
8. 用 system("ls") 在一个 C 程序里执行 ls 命令

## 自测

1. fopen 底层使用什么系统调用? 如何用代码验证?
2. fileno 的作用? fp->_fileno 为什么不推荐?
3. 系统分配文件描述符的规则? close 的 fd 会被再利用吗?
4. close(fd) 后文件对象什么时候才释放? 用什么计数?
5. 重定向的原理 (close 1 + open) 每一步发生了什么?
6. dup 与 dup2 的区别?
7. dup2 当 newfd 已被使用时会发生什么?
8. 反复横跳的实现思路?
9. 第一次使用 stdout 时指向终端 vs 文件, 缓冲模式有何不同? 这与重定向结果有什么关系?
10. 进程与程序的区别? task_struct 是什么?
11. 进程如何获取自己的 pid / ppid? 用什么函数?
12. fork 的返回值在父子进程中分别是什么? 如何区分?
13. 什么是 fork 的写时复制? 复制单位是什么?
14. 父进程的变量在子进程修改后, 会互相影响吗? 为什么?

## 复习记录

- [ ] R1 (次日)
- [ ] R3 (3天后)
- [ ] R7 (7天后)
- [ ] R14 (14天后)