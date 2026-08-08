# Day25 - 无缓冲文件流与文件映射 (mmap)

## 三种文件操作方式总览

| 方式 | 缓冲 | 代表函数 | 平台 |
|---|---|---|---|
| 有缓冲文件流 | 用户态缓冲区 | fopen/fread/fwrite | ISO-C 跨平台 |
| 无缓冲文件流 | 无用户态缓冲区 | open/read/write | 系统调用, 仅类 Unix |
| 文件映射 | 直接映射页缓存 | mmap/munmap | 系统调用, 仅类 Unix |

## 复习: 流模型概念

- 流 = 数据从源头到目的的**流动过程**, 类似水流/流水线
- 关注点: 数据流向、读写位置 (指针)
- 文件流/目录流/无缓冲流都是流模型, 只需搞清数据流向与对应函数

## 复习: stat 的两种路径写法

### 绝对路径写法: sprintf 拼接路径
```c
char path[1024] = {0};
sprintf(path, "%s%s%s", dir_path, "/", pdirent->d_name);
stat(path, &sb);
```
- 适用: 工作目录与目标目录不同, 需要完整的路径名

### 相对路径写法: chdir 改变路径
```c
chdir(dir_path);
stat(pdirent->d_name, &sb);   // 切换后文件名即路径名
```
- 适用: 通过 chdir 把进程工作目录切到目标目录

### 实现 "ls" 不传参查看当前目录
```c
char *path = ".";
if (argc == 2) {
    path = argv[1];   // 传了参数就用参数
}
chdir(path);          // 不传参时 path = "." 即当前目录
```

## 无缓冲文件流

### 概念与文件描述符 (fd)
- 无缓冲文件流: **无用户态缓冲区**, 用户进程直接和内核态缓冲区进行系统调用交互
- 内核不向外保留真实地址 → 返回给用户进程的是一个 int (文件描述符)
- int 不能完全描述文件 → 内核维护**索引指针数组** (约 1024 个), 存文件对象地址
- 文件描述符 = 索引指针数组的下标 (非负整数), 用户进程通过 fd 间接与内核交互
- 保证安全: 用户进程拿不到内核地址, 不能直接改内核数据

### open 函数
```c
#include <fcntl.h>
#include <sys/types.h>
#include <sys/stat.h>
int open(const char *pathname, int flags);
int open(const char *pathname, int flags, mode_t mode);
```
- 有一系列**宏**表示不同含义, 本质: flags 用**二进制某一位为 1** 表示不同含义
- 返回 fd 是文件描述符, 类型 int
- **fd 从 3 开始分配**: 0/1/2 分别被标准输入、标准输出、标准错误占用
- 有 O_CREAT 必须加第三个权限参数 (mode), 否则新建文件权限未知 (随机)
- 权限掩码 (umask): 新文件实际权限 = 指定权限 - 掩码 (复习: `umask` 命令查看)

| 打开模式 | 对应 flags |
|---|---|
| r | O_RDONLY |
| w | O_WRONLY \| O_CREAT \| O_TRUNC |
| a | O_WRONLY \| O_CREAT \| O_APPEND |
| r+ | O_RDWR |
| w+ | O_RDWR \| O_CREAT \| O_TRUNC |

- **O_EXCL 必须与 O_CREAT 连用**: 文件已存在则 open 失败 (防覆盖)
- 有创建需求一定要加权限参数 (mode)

```c
int fd = open(argv[1], O_RDONLY | O_CREAT, 0666);
ERROR_CHECK(fd, -1, "open");
```

### read 函数
```c
#include <unistd.h>
ssize_t read(int fd, void *buf, size_t count);
```
- **不会做任何额外的填充**: 只把数据读到数组中, 通过返回值确认实际读到多少
- 参数 count: 最多读取多少字节
- 返回值: 实际读取到的字节数 (可能小于 count); 读到末尾返回 0; 出错返回 -1
- 注意: read 不会自动补 '\0', 若当字符串用需要 memset 清零或按返回值处理

### write 函数
```c
#include <unistd.h>
ssize_t write(int fd, const void *buf, size_t count);
```
- 参数 count: 要写入的字节数
- 返回值: 实际写入的字节数 (通常等于 count)
- **与 read 的区别**:
  - read 的 count 是**最多读多少** → 返回可能小于 count
  - write 的 count 是**要写多少** → 返回通常等于 count

### lseek: 移动流指针
```c
#include <sys/types.h>
#include <unistd.h>
off_t lseek(int fd, off_t offset, int whence);
```
- 移动内核态缓冲区中文件的读写指针 (类似文件流 fseek)
- whence: `SEEK_SET` 文件头 / `SEEK_CUR` 当前位置 / `SEEK_END` 文件尾
- offset: 正数向文件尾偏移, 负数向文件头偏移, 0 不偏移
- 返回值: 从文件头到当前指针位置的字节数 → **可用它获取文件大小**:
```c
off_t size = lseek(fd, 0, SEEK_END);   // 跳到文件尾, 返回值即文件大小
```

### close 函数
```c
#include <unistd.h>
int close(int fd);
```
- 关闭文件描述符 (成功 0, 失败 -1)
- close 只是断开 fd 与文件对象的映射关系; 引用计数为 0 时才释放文件对象

### 使用无缓冲文件流复制文件
```c
#define BUFFER_SIZE 1024
int fdr = open(argv[1], O_RDONLY);
ERROR_CHECK(fdr, -1, "open read");
int fdw = open(argv[2], O_WRONLY | O_CREAT | O_TRUNC, 0666);
ERROR_CHECK(fdw, -1, "open write");

char buf[BUFFER_SIZE] = {0};
ssize_t read_count;
// read 读到 0 (文件尾) 或 -1 (出错) 时结束循环
while ((read_count = read(fdr, buf, sizeof(buf))) > 0) {
    write(fdw, buf, read_count);   // 写 read 实际读到的字节数
}
close(fdr);
close(fdw);
```
- 要点: **write 的字节数用 read 的返回值** (实际读多少写多少), 避免末尾多写

### 复习: shell 重定向符号
- `>`: 覆盖写; `>>`: 追加写

## 缓冲区类型

### 行缓冲 (line buffer)
- printf 是行缓冲, **以换行符 `\n` 为标识**: 遇到 \n 才刷新输出
- 终端 (屏幕) 输出时: 行缓冲
- 所以 printf 不写 \n 可能看不到输出, 或进程结束/fflush 才输出

### 满缓冲 (full buffer)
- 文件流是满缓冲: **缓冲区不认 `\n`**, 缓冲区写满或手动刷新才输出
- 重定向到文件/非终端设备时: stdout 变满缓冲
- 手动刷新: `fflush(stdout)`

### 缓冲模式的决定时机 (重要)
- **第一次使用 stdout 时的上下文决定缓冲区刷新机制**, 进程生命周期内默认不变:
  - 第一次用 printf 时指向终端 → 行缓冲
  - 第一次时已被重定向到文件 → 满缓冲
- 重定向场景中, 建议**先 printf 输出一句话** (让 stdout 先以终端行缓冲模式启动)

## 文件映射 mmap (重点)

### 概念
- 内存映射文件技术: 把文件内容映射到进程内存, 程序像访问内存一样访问文件
- 对映射内存的读 = 读文件, 写 = 写文件 (自动同步)
- 优势: 内存支持随机访问 → **可对文件内容随机访问**, read/write 只能顺序读写

### mmap 函数
```c
#include <sys/mman.h>
void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);
```
| 参数 | 含义 | 建议 |
|---|---|---|
| addr | 映射起始地址 | **NULL** (操作系统决定) |
| length | 映射大小 (字节), 必须 > 0 | 文件大小, 建议 4K(4096) 整数倍 |
| prot | 映射区访问权限 | PROT_READ 或 PROT_READ \| PROT_WRITE |
| flags | 回写特性 | **MAP_SHARED** |
| fd | open 得到的文件描述符 | - |
| offset | 从文件何处开始映射 | 0 (从头) |

- **prot 注意事项**: 只有 `PROT_READ` 或 `PROT_READ | PROT_WRITE`, **没有单独的 PROT_WRITE** (mmap 需要先读入文件内容)
- 返回值: 成功返回映射区指针; **失败返回 MAP_FAILED** (即 (void*)-1, 不是 NULL), 并设置 errno
- 限制: **offset 只能是 4K (4096) 及其整数倍**, 否则报错 (页对齐要求)

### 总线错误 (Bus Error)
- 映射区域大于文件实际大小时访问超出部分 → **未定义行为, 可能出现总线错误**
- 常见场景: 空文件映射非零字节 / 映射长度超过文件大小
- 解决: 必要时 mmap 前用 `ftruncate` 调整文件大小
- open 的模式要与 prot 一致: prot 可写 (PROT_READ|PROT_WRITE) → open 必须用 O_RDWR (mmap 需要先读文件内容, O_WRONLY 会导致映射失败)

### munmap 解除映射
```c
#include <sys/mman.h>
int munmap(void *addr, size_t length);
```
- addr 为 mmap 返回值, length 与 mmap 一致; 成功 0, 失败 -1

### 简单示例
```c
int fd = open(argv[1], O_RDWR);
ERROR_CHECK(fd, -1, "open");
int ret = ftruncate(fd, 5);                    // 调整文件大小为 5 字节
ERROR_CHECK(ret, -1, "ftruncate");

char *p = (char *)mmap(NULL, 5, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
ERROR_CHECK(p, MAP_FAILED, "mmap");

for (int i = 0; i < 5; ++i) printf("%c", p[i]);  // 随机读
p[4] = 'O';                                       // 随机改, 自动同步到文件

munmap(p, 5);
close(fd);                                        // 先解映射, 再关 fd
```

### 原理: 页缓存 (Page Cache)
- read/write: 数据在**内核态文件对象/页缓存 与 用户态内存之间来回拷贝** (两次拷贝)
- mmap: 把**页缓存中的文件数据直接映射到用户态** → 避免一次用户态↔内核态拷贝
- 性能结论:
  - **read/write 顺序读写性能更好** (可预读优化)
  - **mmap 随机访问/大文件处理性能更好**
  - 日常使用更多还是 read/write (顺序场景多)

### memcpy 内存复制
```c
#include <string.h>
void *memcpy(void *dest, const void *src, size_t n);
```
- 从 src 拷贝 n 个字节到 dest, 返回 dest 指针
- 用于 mmap 场景: 把 src 映射区数据拷到 dest 映射区 (实现文件复制)

### 大文件复制 (分段映射)
- 思路: 不能一次映射整个大文件 (内存装不下) → **分段映射 + memcpy**
  1. 打开源/目标文件, 获取源文件大小 (`lseek(fd,0,SEEK_END)` 或 `fstat`)
  2. `ftruncate(dest_fd, src_size)` 预占目标文件空间, 防映射失败/中途空间不足
  3. 循环: 按块 (如 8MB) 分别 mmap 源和目标, memcpy, munmap, 累加偏移
- fstat: `int fstat(int fd, struct stat *buf);` stat 的变种, 直接传文件描述符

## 练习

1. 用 open/read/write 循环复制一个文件, 观察 write 用 read 返回值的重要性
2. 用 lseek 获取文件大小并打印 (不读文件内容)
3. 用 ftruncate 创建 1MB 的占位文件, 用 stat 指令观察 Size 与 Blocks (文件空洞)
4. mmap 一个小文件, 随机修改其中几个字节, cat 验证文件被修改
5. 写程序: 用 ftruncate 调整文件到指定大小后 mmap, 再访问越界区域观察总线错误
6. 用 mmap 分段复制一个大文件, 与 read/write 循环复制对比耗时 (clock_gettime)

## 自测

1. 三种文件操作方式: 有无缓冲? 代表函数? 平台?
2. stat 获取文件信息有哪两种路径写法? 分别适用于什么场景?
3. 为什么 open 返回的是 int 而不是指针? 文件描述符的本质是什么?
4. open 的 fd 为什么从 3 开始?
5. 有 O_CREAT 为什么必须加 mode 参数?
6. O_EXCL 为什么必须和 O_CREAT 连用?
7. read 和 write 的 count 参数语义区别? 返回值语义区别?
8. read 为什么不会自动补 '\0'? 当字符串用时要注意什么?
9. lseek 如何获取文件大小?
10. printf 是行缓冲还是满缓冲? 文件重定向后呢?
11. 缓冲模式是何时决定的? 重定向 stdout 前为什么要先 printf 一句?
12. mmap 的 prot 为什么没有单独的 PROT_WRITE?
13. mmap 失败返回什么? 和 NULL 有什么区别?
14. mmap 的 offset 有什么限制? 为什么?
15. 什么是总线错误? 什么场景触发? 如何避免?
16. mmap 和 read/write 各自在什么场景性能更好? 为什么?
17. ftruncate 在 mmap 中的两个作用是什么?
18. 大文件复制为什么用分段映射?

## 复习记录

- [ ] R1 (次日)
- [ ] R3 (3天后)
- [ ] R7 (7天后)
- [ ] R14 (14天后)
