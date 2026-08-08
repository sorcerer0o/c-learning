# C语言学习笔记

## 目录

```
├── README.md
├── .gitignore
├── notes/                    # 学习笔记 (day-by-day)
│   ├── day01-变量函数输入输出.md
│   ├── day02-预处理内存模型.md
│   ├── day03-数据类型与位运算.md
│   ├── day04-运算符与表达式.md
│   ├── day05-数组.md
│   ├── day06-函数详解.md
│   ├── day07-指针基础.md
│   ├── day08-指针进阶与字符串.md
│   ├── day09-字符串进阶与命令行参数.md
│   ├── day10-结构体枚举动态内存.md
│   ├── day11-动态内存管理深入.md
│   ├── day12-头文件函数指针链表.md
│   ├── day13-链表操作复习.md
│   ├── day14-哈希表.md
│   ├── day15-二叉树与基础排序.md
│   ├── day16-高级排序与文件流基础.md
│   ├── day17-文件流深入.md
│   ├── day19-目录流无缓冲文件流.md
│   ├── day20-Shell.md
│   ├── day21-Linux系统基础与工具链.md
│   ├── day22-GNU工具集.md
│   ├── day23-目录流.md
│   ├── day24-目录流进阶.md
│   ├── day25-无缓冲文件流与mmap.md
│   ├── day26-管道与select.md
│   ├── day27-文件描述符重定向与进程.md
│   ├── day28-进程回收与IPC.md
│   ├── AI/                   # AI 辅助开发专题
│   └── Linux/                # Linux 系统编程专题
│       ├── 01_Linux基础/
│       ├── 02_GNU工具集/
│       └── 03_文件系统编程/
├── mindmaps/                 # 思维导图 (.mm + .pos)
├── preview/                  # 预习资料 (PDF)
└── misc/                     # 盲点记录、备忘、参考书等
```

## 使用方式

```bash
# 克隆仓库
git clone https://github.com/sorcerer0o/c-learning.git

# 配合代码练习
git clone https://github.com/sorcerer0o/c.git
```

学习流程：看 `notes/dayXX-主题.md` → 对照 `mindmaps/dayXX-主题` → 在 `~/code/c/` 下写代码

## 笔记列表

| 文件 | 主题 |
|------|------|
| day01-变量函数输入输出 | 变量/函数/printf/scanf/注释/关键字 |
| day02-预处理内存模型 | 预处理/宏/虚拟内存/字节序/调试 |
| day03-数据类型与位运算 | sizeof/整数类型/浮点数/位运算/原码反码补码 |
| day04-运算符与表达式 | 运算符优先级/类型转换/短路求值 |
| day05-数组 | 一维数组/二维数组/随机数/数组寻址 |
| day06-函数详解 | 栈帧/局部变量/全局变量/static/递归 |
| day07-指针基础 | 指针/地址/解引用/空指针/值传递/指针参数 |
| day08-指针进阶与字符串 | 数组指针/指针数组/const指针/字符串处理 |
| day09-字符串进阶与命令行参数 | scanf安全/字符串函数/命令行参数/main函数 |
| day10-结构体枚举动态内存 | 结构体/枚举/联合体/malloc/free |
| day11-动态内存管理深入 | calloc/realloc/内存泄漏/vector实现 |
| day12-头文件函数指针链表 | 头文件保护/函数指针/qsort/链表基础 |
| day13-链表操作复习 | 链表反转/合并/线性表对比 |
| day14-哈希表 | 哈希表原理/链地址法/扩容 |
| day15-二叉树与基础排序 | BST/选择/冒泡/插入排序 |
| day16-高级排序与文件流基础 | 快排/归并/堆排/文件流基础 |
| day17-文件流深入 | fopen模式/fgets/fread/fseek/errno |
| day19-目录流无缓冲文件流 | 目录流/无缓冲IO/mmap/dup2重定向 |
| day20-Shell | Shell命令/管道重定向/脚本/进程/权限 |
| day21-Linux系统基础与工具链 | 文件系统/权限/Vim/GCC/find/tar/xxd |
| day22-GNU工具集 | 条件编译/GDB调试/Coredump/库文件/Makefile |
| day23-目录流 | 系统调用/ISO-C/POSIX/目录系统调用/目录流/stat/ls-al/tree |
| day24-目录流进阶 | 复习巩固/目录流/dirent/指针栈区/stat/缓冲区/类型别名查看 |
| day25-无缓冲文件流与mmap | 缓冲类型/open-read-write-lseek/ftruncate/mmap/页缓存 |
| day26-管道与select | 阻塞队列/管道/半双工/select/fd_set |
| day27-文件描述符重定向与进程 | fopen底层/fileno/dup-dup2/重定向/缓冲机制/fork |
| day28-进程回收与IPC | wait/waitpid/僵尸孤儿进程/IPC总览 |

### 专题目录

| 目录 | 内容 |
|------|------|
| AI/ | 大模型介绍、Prompt工程、AI辅助开发、Tool/MCP |
| Linux/01_Linux基础/ | Linux环境安装、Shell命令、Vim编辑器 |
| Linux/02_GNU工具集/ | GCC/GDB/Makefile/库文件 |
| Linux/03_文件系统编程/ | 目录流/无缓冲IO/文件映射/重定向 |

## 相关仓库

| 仓库 | 说明 |
|------|------|
| [c-learning](https://github.com/sorcerer0o/c-learning) | 本仓库: C 学习笔记 |
| [c](https://github.com/sorcerer0o/c) | C 练习代码 |
| [cpp-learning](https://github.com/sorcerer0o/cpp-learning) | C++ 学习笔记 |
| [cpp](https://github.com/sorcerer0o/cpp) | C++ 练习代码 |
