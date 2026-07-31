# Day22 - GNU 工具集: 调试、库文件与 Makefile

## 条件编译

### 概念
- 条件编译: 在**预处理阶段**决定代码片段是否进入编译
- 常用于: 调试信息开关, 跨平台代码, 头文件保护

### #if / #elif / #else / #endif
```c
#define DEBUG 1

#if DEBUG
    printf("i = %d\n", i);   // DEBUG 为真时编译
#else
    printf("no debug\n");
#endif
```
- 条件必须是**常量表达式** (宏, 字面值)
- 组合 `#if 0 ... #endif` 可以屏蔽代码, 且能屏蔽含注释的代码 (多行注释不能嵌套)

### #ifdef / #ifndef
```c
#ifdef DEBUG      // 等价于 #if defined(DEBUG)
    // 调试代码 (编译时用 -DDEBUG 控制)
#endif

#ifndef HEADER_H  // 头文件保护
#define HEADER_H
// 头文件内容
#endif
```
- 只要宏被定义即成立, 与宏的值无关
- 头文件保护语法: `#ifndef` + `#define` + `#endif`

### 实际应用
```c
#ifdef WIN32
    new_line = "\r\n";
#else
    new_line = "\n";
#endif
```
- 跨平台: 根据系统宏选择代码
- 调试开关: `gcc main.c -DDEBUG` 控制是否包含调试代码

## 相对路径和绝对路径

- **绝对路径**: 从根目录 `/` 开始的完整路径, 如 `/home/user/main.c`
- **相对路径**: 相对于当前工作目录的路径
  - `.` 当前目录
  - `..` 上级目录
  - `./main.c` 当前目录下的文件
  - `../header/test.h` 上级目录的 header 目录
- 相对路径的关键是**起点** (当前工作目录), 可用 `pwd` 查看

## Debug 的大致思路

1. 大致扫一遍代码, 检查是否有明显的错误
2. 关注高风险区域: 指针、链表、边界值、数组下标等
3. 采用注释法/取消注释法排查崩溃位置
4. 发现崩溃位置后, 顺着数据链检查: 数据从哪来, 流向哪里

## GDB 调试

### 调试的本质
- 信息确认: 预判程序行为, 与实际行为对比
- 判断是"思路错了"还是"代码写错了"

### 基本流程
```bash
gcc main.c -o main -g -O0 -Wall    # 1. 必须加 -g 编译
gdb main                            # 2a. 直接进入
gdb                                 # 2b. 先进入再加载
(gdb) file main
```

### 常用指令

| 指令 (短) | 作用 |
|-----------|------|
| `l [行号\|函数]` | 查看源代码 |
| `b [行号\|函数]` | 打断点 |
| `i b` | 查看断点信息 |
| `d [n]` | 删除断点 |
| `dis [n]` / `en [n]` | 断点失效/生效 |
| `r [参数]` | 运行 (带参运行) |
| `s` | 单步调试 (进入函数) |
| `n` | 逐过程 (不进入函数) |
| `finish` | 执行完当前函数 |
| `c` | 继续到下一个断点 |
| `k` | 停止程序调试 |
| `q` | 退出 GDB |

### 查看与修改变量
```bash
(gdb) p arr[0]          # 查看变量/表达式值
(gdb) p arr[0]=10       # 修改值
(gdb) i locals          # 查看所有局部变量
(gdb) i args            # 查看函数参数
(gdb) display expr      # 每步自动显示表达式
(gdb) display arr[0]@len   # 持续显示数组元素 (len 为变量)
(gdb) watch expr        # 表达式变化时自动停止 (观察断点)
(gdb) ignore 1 10       # 忽略 1 号断点 10 次
```

### 查看内存 (内存窗口)
```
x/(个数)(格式)(单元大小) 地址
```
- 格式: `x` 十六进制, `d` 十进制, `o` 八进制, `u` 无符号, `t` 二进制, `c` 字符, `s` 字符串, `f` 浮点, `a` 地址
- 单元大小: `b` 1字节, `h` 2字节, `w` 4字节, `g` 8字节

```bash
(gdb) x/4dw nums        # 从 nums 起, 4 个 4 字节单元, 十进制
(gdb) x/5xg strs        # 指针数组每个元素 (8字节) 的地址值
(gdb) x/s *strs         # 查看第一个字符串
(gdb) x/s strs[1]       # 查看第二个字符串
```

### 调用堆栈
```bash
(gdb) bt                # 查看调用堆栈 (在报错处使用, 追溯调用链)
```

### GDB 传递命令行参数
```bash
gdb --args ./main arg1 arg2    # 启动时传参
(gdb) set args arg1 arg2       # 已启动时设置
(gdb) r arg1 arg2              # run 时直接带参
```

## Linux 目录结构

| 目录 | 用途 |
|------|------|
| `/` | 根目录 |
| `/bin` | 基本命令 (ls/cp), 软链接到 /usr/bin |
| `/sbin` | 系统管理命令 (root 使用) |
| `/etc` | 系统配置文件 |
| `/home` | 普通用户家目录 |
| `/root` | root 用户家目录 |
| `/usr` | 应用程序和文件 (头文件在 /usr/include) |
| `/lib` | 系统库文件 (libc.so 等) |
| `/var` | 变化的数据 (日志, 缓存) |
| `/tmp` | 临时文件 |
| `/dev` | 设备文件 (块/字符设备) |
| `/proc` | 进程和内核信息 (虚拟文件系统) |
| `/mnt` / `/media` | 挂载点 |

## Coredump 调试

### 概念
- 程序异常终止 (段错误) 时生成的核心转储文件, 类似飞机"黑匣子"
- 记录崩溃瞬间: 寄存器状态, 调用栈等

### 开启 Coredump
```bash
ulimit -a                        # 查看限制 (core file size 默认 0)
ulimit -c unlimited              # 临时设置为不受限制
sudo vim /etc/sysctl.conf        # 配置 core 文件命名
# 末尾添加: kernel.core_pattern = ./core_%e_%t
sudo sysctl -p                   # 使配置生效
```

### 使用
```bash
# 崩溃程序生成 core 文件后
gdb 可执行程序 core_文件名       # gdb + 程序 + core 文件
(gdb) bt                        # 查看崩溃时的调用栈
```
- 注意: 只有**段错误**等异常终止才生成 core 文件, 数组越界这类未定义行为不一定生成
- 生产环境中无法复现错误时, core 文件是排查利器

## 库文件

### 概念
- 库文件 = 目标文件 (.o) 的集合, 即"写好的轮子"
- 避免重复造轮子, 提供常用功能: 数学、字符串、文件操作等

### 分类与后缀

| 类型 | Linux 后缀 | Windows 后缀 | 链接时机 |
|------|-----------|-------------|---------|
| 静态库 | `.a` | `.lib` | 链接阶段 (整体合并进程序) |
| 动态库 | `.so` | `.dll` | 运行时才载入内存 |

- 系统标准库: `libc.a` (静态), `libc.so` (动态), 位于 `/usr/lib`

### 优缺点对比

| | 静态库 | 动态库 |
|--|--------|--------|
| 优点 | 可执行文件独立运行, 移植方便 | 体积小, 多程序共享省内存, 更新库无需重新编译 |
| 缺点 | 文件大, 库更新需重新编译发布 | 缺库无法运行, 运行时链接有性能开销 |

### 生成静态库
```c
// my_compute.h
#ifndef __WD_MY_COMPUTE_H
#define __WD_MY_COMPUTE_H
int add(int a, int b);
int sub(int a, int b);
#endif
```
```bash
gcc -c add.c -o add.o              # 生成目标文件
gcc -c sub.c -o sub.o
ar crsv my_compute.a add.o sub.o   # 打包成静态库
```
- `ar` 选项: `c` 创建, `r` 添加/替换, `s` 生成索引, `v` 显示详情

### 生成动态库
```bash
gcc -c add.c -o add.o -fpic        # -fpic 生成位置无关代码 (必须)
gcc -c sub.c -o sub.o -fpic
gcc -shared add.o sub.o -o libmy_compute.so   # -shared 生成动态库
```

### 使用库文件
```bash
sudo mv my_compute.a /usr/lib/libmy_compute.a   # 放入系统库目录
sudo mv libmy_compute.so /usr/lib/              # 动态库必须放入系统目录

gcc main.c -o main -lmy_compute    # -l 链接库 (名字必须以 lib 开头)
```
- **静态库**: 链接时合并进程序, 删除库不影响已生成程序
- **动态库**: 运行时链接, 删除库程序无法运行; 更新库内容程序行为会变
- 同名 .a 和 .so 都存在时, **gcc 默认优先链接动态库** (可用 `-static` 强制静态)
- 版本管理: `libmy_compute.so.0.0.1` + 软链接 `ln -s libmy_compute.so.0.0.1 libmy_compute.so`

## Makefile

### 为什么需要 Makefile
- 工程有大量源文件, 需要自动化编译链接
- 实现**增量编译**: 只重新编译修改过的文件 (大工程全量编译可能数小时)

### 基本结构
- 文件名固定: `Makefile` 或 `makefile`
- 基本组成单元: **规则 (Rule)** = 目标 + 依赖 + 命令

```makefile
main: add.o main.o          # 目标: 依赖
	gcc add.o main.o -o main    # 命令 (必须以 Tab 制表符开头!)
add.o: add.c compute.h
	gcc -c add.c -o add.o -Wall -g
main.o: main.c compute.h
	gcc -c main.c -o main.o -Wall -g
```

- 一条规则只有一个目标
- 命令可以是任意 shell 命令
- **命令开头必须是制表符 (Tab), 不能是空格**

### 工作原理
1. `make` 读取当前目录的 Makefile
2. 默认目标 = **第一条规则的目标**
3. 检查目标是否过时: 依赖比目标新, 或依赖不存在, 则执行命令重建
4. 依赖不存在则递归生成依赖, 直到目标生成

### 伪目标
- 没有依赖、不生成实际文件的目标, 保证命令必然执行
- 用途: `clean` (清理), `rebuild` (重新生成)

```makefile
.PHONY: clean rebuild      # 标记伪目标 (推荐)

clean:
	rm -f main main.o add.o
rebuild: clean main        # 先 clean 再重新构建
```

### 变量
```makefile
OUT := main                # 自定义变量 (:= 赋值)
OBJS := main.o add.o
COM_OP := -Wall -g
CC := gcc                  # 覆盖预定义变量

$(OUT):$(OBJS)
	gcc $^ -o $@
```

| 变量类型 | 说明 | 示例 |
|---------|------|------|
| 自定义变量 | 自己定义, `:=` 赋值, `$(变量名)` 使用 | `OUT := main` |
| 自动变量 | 规则内自动设置 | `$@` 目标, `$<` 第一个依赖, `$^` 所有依赖 |
| 预定义变量 | Make 内置 | `CC` 编译器, `RM` 删除, `CFLAGS` 编译选项 |

### 模式规则
```makefile
%.o : %.c
	$(CC) -c $< -o $@ $(COM_OP)
```
- `%` 匹配任意文件名, 一条规则适用所有 .c → .o
- 模式规则**不能作为第一条规则**

### 内置函数 (了解)
```makefile
SRCS := $(wildcard *.c)                  # 列出所有 .c 文件
OBJS := $(patsubst %.c,%.o,$(SRCS))      # .c 替换为 .o
```

### 通用 Makefile 模板
```makefile
OUT := main
SRCS := $(wildcard *.c)
OBJS := $(patsubst %.c,%.o,$(SRCS))
COM_OP := -Wall -g
CC := gcc

$(OUT):$(OBJS)
	$(CC) $^ -o $@
%.o : %.c
	$(CC) -c $< -o $@ $(COM_OP)

.PHONY: clean rebuild
clean:
	$(RM) $(OUT) $(OBJS)
rebuild: clean $(OUT)
```

## 练习

1. 写一个含 bug 的程序 (数组越界), 用 GDB 调试: 打断点、单步、查看变量和内存
2. 用 GDB 的 `x/` 命令查看数组和指针数组的内存布局
3. 写一个空指针解引用的程序, 开启 coredump 后用 `gdb + core` 和 `bt` 排查
4. 将 add/sub 函数打包成静态库和动态库, 分别链接生成可执行程序
5. 编写一个带 clean/rebuild 伪目标和变量的 Makefile, 实现增量编译
6. 用 `-DDEBUG` 和条件编译实现调试开关

## 自测

1. #ifdef 和 #if 有什么区别？头文件保护为什么用 #ifndef？
2. 相对路径和绝对路径的区别？相对路径的关键点是什么？
3. 使用 GDB 前编译必须加什么选项？为什么？
4. GDB 中 s、n、c、finish 分别是什么作用？
5. GDB 中如何查看数组内存？x/4dw 每个字符代表什么？
6. 如何给 GDB 调试的程序传递命令行参数？
7. Coredump 文件是什么？如何开启？用哪个指令分析？
8. 静态库和动态库的优缺点？后缀分别是什么？
9. 生成静态库和动态库的命令分别是什么？-fpic 的作用？
10. Makefile 中规则的三要素？命令开头为什么必须是 Tab？
11. 自动变量 $@、$<、$^ 分别代表什么？
12. 伪目标是什么？clean 和 rebuild 的作用？

## 复习记录

- [ ] R1 (次日)
- [ ] R3 (3天后)
- [ ] R7 (7天后)
- [ ] R14 (14天后)
- [ ] R30 (30天后)
