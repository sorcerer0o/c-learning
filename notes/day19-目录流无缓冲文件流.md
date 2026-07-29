# Day19 - 目录流与无缓冲文件流

## 目录流

### 概念
- 目录流用于遍历目录中的文件条目
- 类比文件流，操作模式类似：打开 → 读取 → 关闭

### 核心函数

**打开目录流**
```c
#include <dirent.h>
DIR *opendir(const char *name);
```
- 成功返回 DIR* 指针，失败返回 NULL

**关闭目录流**
```c
int closedir(DIR *dirp);
```

**读取目录项**
```c
struct dirent *readdir(DIR *dirp);
```
- 返回下一个目录项，读完返回 NULL
- 返回值指向静态分配的内存，由系统管理，不需要 free

### dirent 结构体
```c
struct dirent {
    ino_t          d_ino;       // inode 编号
    off_t          d_off;       // 到下一个目录项的偏移
    unsigned short d_reclen;    // 目录项实际大小
    unsigned char  d_type;      // 文件类型
    char           d_name[256]; // 文件名
};
```

文件类型常量: DT_REG(普通), DT_DIR(目录), DT_LNK(符号链接) 等

### 标准用法
```c
DIR *dirp = opendir(argv[1]);
struct dirent *pdirent;
while ((pdirent = readdir(dirp)) != NULL) {
    printf("%s\n", pdirent->d_name);
}
closedir(dirp);
```

### 定位目录流
- `rewinddir(DIR *dirp)` - 倒带回到开头
- `telldir(DIR *dirp)` - 获取当前位置标记
- `seekdir(DIR *dirp, long loc)` - 移动到记录的位置

### stat 系统调用
```c
#include <sys/stat.h>
int stat(const char *path, struct stat *buf);
```
- 获取文件详细信息（类型、权限、大小、时间等）
- 配合目录流可实现简化版 ls -al

### struct stat 关键字段
```c
struct stat {
    mode_t    st_mode;   // 文件类型 + 权限
    nlink_t   st_nlink;  // 硬链接数
    uid_t     st_uid;    // 所有者 UID
    gid_t     st_gid;    // 组 GID
    off_t     st_size;   // 文件大小
    struct timespec st_mtim; // 最后修改时间
};
```

- 文件类型判断: `mode & S_IFMT` 然后匹配 S_IFREG/S_IFDIR 等
- 权限字符串: 从高到低按位判断 rwx 权限位

## 无缓冲文件流

### 概念
- 没有用户态缓冲区，用户进程直接和内核交互
- 操作的是文件描述符 int fd，而非 FILE*
- 属于 Linux 系统调用（POSIX 标准）

### 模型对比
- 有缓冲: 磁盘 → 内核态缓冲 → 用户态缓冲 → 用户进程
- 无缓冲: 磁盘 → 内核态缓冲 → 用户进程（通过文件描述符）

### 文件描述符
- 每个进程默认打开 0(stdin), 1(stdout), 2(stderr)
- 新打开的文件从 3 开始分配
- 通过 fd 间接与内核交互，保证安全性

### 核心函数

**open**
```c
#include <fcntl.h>
int open(const char *pathname, int flags);
int open(const char *pathname, int flags, mode_t mode);
```
- flags: O_RDONLY(0), O_WRONLY(1), O_RDWR(2) 三选一
- 可组合: O_CREAT, O_EXCL, O_TRUNC, O_APPEND
- 使用 O_CREAT 时需要 mode 参数指定权限

**close**
```c
int close(int fd);
```

**read**
```c
ssize_t read(int fd, void *buf, size_t count);
```
- 返回实际读取字节数，末尾返回 0，出错返回 -1
- 读磁盘文件非阻塞，读设备文件可能阻塞

**write**
```c
ssize_t write(int fd, const void *buf, size_t count);
```
- 按什么类型写入，就按什么类型读出

### 文件复制示例
```c
int fdr = open(src, O_RDONLY);
int fdw = open(dst, O_WRONLY|O_CREAT|O_TRUNC, 0666);
char buf[1024];
ssize_t n;
while ((n = read(fdr, buf, sizeof(buf))) > 0) {
    write(fdw, buf, n);
}
close(fdr);
close(fdw);
```

### 性能对比
- 无缓冲 IO 在 buf 很小时效率低（频繁系统调用）
- 有缓冲 IO（fread/fwrite）在多数场景下性能更好
- 无缓冲 IO 适用于: 实时性要求高、设备文件、网络通信、管道

### ftruncate 截断文件
```c
#include <unistd.h>
int ftruncate(int fd, off_t length);
```
- 增大文件用 0 填充，减小文件从末尾截断
- 可用于预分配空间（文件空洞）

### dup2 重定向
```c
#include <unistd.h>
int dup2(int oldfd, int newfd);
```
- 复制文件描述符，将 oldfd 重定向到 newfd
- 示例: 将 stdout 重定向到文件
```c
int fd = open("out.txt", O_WRONLY | O_CREAT, 0644);
dup2(fd, STDOUT_FILENO);
printf("这行会写入文件");
```

## mmap 内存映射

### 概念
- 将文件直接映射到进程的虚拟内存空间
- 操作内存 = 操作文件，无需 read/write

### 函数
```c
#include <sys/mman.h>
void *mmap(void *addr, size_t length, int prot,
           int flags, int fd, off_t offset);
int munmap(void *addr, size_t length);
```

### 参数
| 参数 | 常用值 | 说明 |
|------|--------|------|
| addr | NULL | 系统自动选择映射地址 |
| length | 文件大小 | 映射字节数 |
| prot | PROT_READ \| PROT_WRITE | 读写权限 |
| flags | MAP_SHARED | 修改回写文件 |
| fd | open 返回值 | 文件描述符 |
| offset | 0 | 从文件头开始 |

### 完整示例
```c
#include <sys/mman.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>

int main(void) {
    int fd = open("test.txt", O_RDWR);
    struct stat sb;
    fstat(fd, &sb);

    char *p = mmap(NULL, sb.st_size,
                   PROT_READ | PROT_WRITE, MAP_SHARED,
                   fd, 0);
    // 直接通过指针读写
    printf("%s", p);   // 读文件
    p[0] = 'H';         // 写文件（自动同步）

    munmap(p, sb.st_size);
    close(fd);
    return 0;
}
```

### mmap vs read/write
- mmap 减少数据拷贝（不需要中间缓冲区）
- 适合大文件随机访问
- 操作简单直观（指针操作）
- 但映射长度不能超过文件大小（否则总线错误）

## 常见错误
1. readdir 后忘记 checkedir 关闭
2. stat 传参用文件名而非路径名（需先 chdir 或拼接绝对路径）
3. open 时忘记 O_CREAT，文件不存在则失败
4. write 时直接写 sizeof(buf) 而非 read 的返回值

## 面试常问
Q: 有缓冲和无缓冲的区别？
A: 有缓冲有用户态缓冲区减少系统调用，性能好但实时性差；无缓冲直接系统调用，实时性好但性能在 buf 小时差。

Q: 文件描述符和 FILE* 的关系？
A: FILE* 是 C 标准库的封装，内部包含文件描述符。fopen 底层调用 open。

Q: mmap 的优势？
A: 减少数据拷贝（不需要 read/write 的中间缓冲），适合大文件随机访问，操作像指针一样方便。

## 练习
1. 用目录流实现简化版 ls 指令
2. 用 stat 获取文件信息并打印 rwx 权限字符串
3. 用 read/write 实现文件复制（对比 fread/fwrite 性能）
4. 用 ftruncate 创建指定大小文件
5. 用 mmap 读取文件并修改其中内容
6. 用 dup2 实现标准输出重定向到文件

## 复习记录
- [ ] R1 (次日)
- [ ] R3 (3天后)
- [ ] R7 (7天后)
- [ ] R14 (14天后)
- [ ] R30 (30天后)
