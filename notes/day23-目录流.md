# Day23 - 目录流: 系统调用与目录遍历

## 三类函数: 系统调用 / ISO-C / POSIX

| | 系统调用 | ISO-C 标准库 | POSIX 库函数 |
|---|---|---|---|
| 定义 | 内核对外暴露的功能接口 | 符合 C 语言规范的库函数 | 符合 POSIX 标准的库函数 |
| 跨平台 | 不跨平台 (植根特定内核) | 跨平台最好 (Windows/Linux/Mac 一致) | 仅类 Unix 平台 |
| 实现 | 直接进内核 | 库函数, 底层可能封装系统调用 | 库函数, 底层可能封装系统调用 |

- 文件系统的**系统编程**操作, 因为需要调用内核, 所以只能在类 Unix 中实现
- 类 Unix 的系统调用以 C 风格暴露, 一般遵循 POSIX 标准
- 基于 Linux 内核学系统调用 → 写出的代码仅能运行于 Linux 平台
- ISO-C 与 POSIX 库函数**相同点**: 同为库函数, 都不是系统调用, 但都可能利用系统调用实现功能
- 注意: 有的函数名在 man 1/2/3 卷中都存在 (如 chmod 既是指令、也是系统调用、也是库函数)

## 系统调用函数的学习步骤

1. 记住函数名、大致功能, 推测传入/传出参数、返回值等
2. 记牢 man 手册卷号: **卷 2 = 系统调用**, **卷 3 = 库函数**
3. 打开函数 man 手册, 按顺序看:
   - 先看 **NAME**: 函数基本作用
   - 再看 **SYNOPSIS**: 头文件 → 函数声明 → 返回值

## 函数设计规律 (错误处理核心)

### 为什么返回值如此重要
- C 语言**缺乏错误检查机制**, 函数执行出错只能依赖返回值判断
- 返回值是什么 → 决定程序应如何做错误处理

### 返回值规律
- 返回值通常**兼具两个任务**:
  1. 正常返回值 (业务语义)
  2. 出错标记
- 所以先检查返回值是否为 **-1 / NULL**, 再谈业务

### 参数规律
- 指针参数若加 `const` → 该入参是**只读参数** (函数不会修改)
- 指针参数若**没有** `const` → 大概率会被函数修改 (传出/传入传出)
- 例: `stat(path, &buf)` 的 buf 不带 const (函数内部会修改结构体), 是典型的传入传出参数

### xx_t 类型别名
- `_t` 结尾的类型 (如 `mode_t`, `off_t`, `size_t`, `ssize_t`, `nlink_t`, `ino_t`) 是**类型别名** (typedef)
- 作用: 屏蔽平台差异, 增强可移植性
- 例: `mode_t` 在 64 位 Linux 下一般是 `unsigned int`

## 如何查看类型别名的具体定义

### 方式 1: 查预处理 .i 文件 (推荐)
```bash
gcc -E main.c -o main.i          # main.c 中 #include <sys/stat.h>
grep -nE "typedef.*mode_t" main.i
```
- 直接查头文件可能受条件编译干扰, 查预处理文件最准确

### 方式 2: 代码计算推导
```c
printf("size of mode_t = %zu bytes\n", sizeof(mode_t));  // %zu 专用于 size_t
if ((mode_t)-1 > 0) {
    printf("无符号!\n");    // 无符号的 -1 会溢出成正数
} else {
    printf("有符号!\n");
}
```

## 复习 sprintf / snprintf (字符串拼接)

- `sprintf(buf, fmt, ...)`: 格式化写入字符串, 常用于拼接路径
- `snprintf(buf, size, fmt, ...)`: 限定最多写入 size 字节, **防止缓冲区溢出**, 优先使用

## 目录相关系统调用

> 此小节都是系统调用, 每学一个新函数, 把其头文件加入公共头文件 my_header.h

### chmod: 改变文件权限
```c
#include <sys/stat.h>
int chmod(const char *pathname, mode_t mode);
```
- mode 用**权限数字表示法** (八进制, C 语言中以 0 开头)
- 成功返回 0, 失败返回 -1 并设置 errno
- **掩码只影响新建文件, 不影响 chmod** → 设定的权限就是最终权限

```c
/* Usage: ./01_chmod pathname mode */
ARGS_CHECK(argc, 3);
mode_t mode;
sscanf(argv[2], "%o", &mode);          // %o 解析八进制无符号数
int ret = chmod(argv[1], mode);
ERROR_CHECK(ret, -1, "chmod");
```

### getcwd: 获取当前工作目录
```c
#include <unistd.h>
char *getcwd(char *buf, size_t size);
```
- POSIX 库函数, 类似 shell 的 `pwd`, 获取工作目录的**绝对路径**
- 成功返回 buf 指针; 失败返回 NULL (常见原因: buf 过小)
- buf 生命周期由程序员管理 (栈数组或 malloc/free)

```c
char path[1024] = {0};
char *p = getcwd(path, sizeof(path));
ERROR_CHECK(p, NULL, "getcwd");
```
- 变体: `getcwd(NULL, 0)` → 函数自己 malloc 空间返回, 用完后必须 `free` (否则内存泄漏)

### chdir: 改变当前工作目录
```c
#include <unistd.h>
int chdir(const char *path);
```
- Linux 系统调用, 类似 shell 的 `cd`; 成功返回 0, 失败返回 -1
- 当前工作目录是**进程的属性**, 每个进程都有自己的工作目录
- 父进程创建子进程时, 子进程**继承**父进程的工作目录
- **chdir 并不等同于 cd**:
  - 执行程序时 shell 创建子进程 → 子进程里 chdir 改的是子进程的目录, 不影响 shell (类似值传递)
  - cd 是 bash 的**内部指令**, 直接修改 bash 进程自己的工作目录 → 所以 `which cd` 找不到可执行文件

### mkdir: 创建目录
```c
#include <sys/stat.h>
#include <sys/types.h>
int mkdir(const char *pathname, mode_t mode);
```
- mode 为八进制权限 (如 0775); 成功返回 0, 失败返回 -1 (如目录已存在)
- **最终权限受 umask 影响**: 指定 777 可能实际生成 775 ("权限 - 掩码")
- umask 是三位八进制数, 用于创建新文件/目录时**移除**一些权限 (系统安全考虑)
- `umask` 指令查看当前掩码, `umask n` 设置新掩码

```c
/* Usage: ./04_mkdir pathname mode */
mode_t mode;
sscanf(argv[2], "%o", &mode);
int ret = mkdir(argv[1], mode);
ERROR_CHECK(ret, -1, "mkdir");
```

### rmdir: 删除目录
```c
#include <unistd.h>
int rmdir(const char *pathname);
```
- **只能删除空目录**; 成功返回 0, 失败返回 -1 (目录非空/不存在/无权限)

## 目录流 (POSIX 库函数)

### 目录中存储的是什么
- 目录中存储的是子目录/文件的**目录项** (directory entry), 近似以**链表**形式链接
- 目录流指针一开始指向第一个目录项, 可向后移动 (类比文件流 FILE 指针)
- **目录流 vs 文件流 API 对比**:

| 文件流 | 目录流 |
|:---:|:---:|
| fopen | opendir |
| fclose | closedir |
| fread | readdir |
| fwrite | × (无) |
| ftell | telldir |
| fseek | seekdir |
| rewind | rewinddir |

- **目录流没有写操作, 只能读**: 开放写权限极易破坏文件系统目录结构, 只读是为了保障文件系统安全
- 目录流是 POSIX API → Windows 平台不可直接使用

### 打开/关闭目录流
```c
#include <dirent.h>      // dirent = directory entry
#include <sys/types.h>
DIR *opendir(const char *name);
int closedir(DIR *dirp);
```
- opendir: 成功返回目录流指针, 失败返回 NULL 并设置 errno
- closedir: 成功返回 0, 失败返回 -1; 释放目录流资源, 避免内存泄漏

```c
/* Usage: ./06_dirent pathname */
ARGS_CHECK(argc, 2);
DIR *dirp = opendir(argv[1]);
ERROR_CHECK(dirp, NULL, "opendir");
// 读目录流操作...
closedir(dirp);
```
- 惯用法: 打开 → 检查错误 → 操作 → 关闭 (与文件流完全类似)

### readdir: 读目录流
```c
#include <dirent.h>
struct dirent *readdir(DIR *dirp);
```
- 依次读取目录中的每个条目, 直到没有更多条目返回 NULL
- 出错也会返回 NULL → 但只要传入的是打开的目录流指针一般不会出错, **可把 NULL 当作读完标记**
- **内存管理**: 返回值指向的结构体是**静态分配**的, 不需要也不能 free (man 手册: statically allocated)
- 用 closedir 关闭目录流时, 相应结构体对象自动释放销毁

```c
struct dirent *pdirent;
while ((pdirent = readdir(dirp)) != NULL) {
    printf("inode = %lu, reclen = %hu, type = %u, name = %s\n",
           pdirent->d_ino, pdirent->d_reclen,
           pdirent->d_type, pdirent->d_name);
}
```

### dirent 结构体 (重点)
```c
struct dirent {
    ino_t          d_ino;       // inode 编号 (64 位无符号整数)
    off_t          d_off;       // 到下一个目录项的偏移量 (可视为"链表指针")
    unsigned short d_reclen;    // 目录项实际大小 (字节), 不是文件大小!
    unsigned char  d_type;      // 文件类型 (整数宏)
    char           d_name[256]; // 文件名, 一般决定目录项实际大小
};
```

d_type 可选值 (宏常量整数):

| 宏 | 含义 | 值 |
|---|---|---|
| DT_BLK | 块设备文件 | 6 |
| DT_CHR | 字符设备文件 | 2 |
| DT_DIR | 目录文件 | 4 |
| DT_FIFO | 有名管道文件 | 1 |
| DT_LNK | 符号链接文件 | 10 |
| DT_REG | 普通文件 | 8 |
| DT_SOCK | 套接字文件 | 12 |
| DT_UNKNOWN | 未知类型 | 0 |

### 如何查询结构体类型的具体定义
- 预处理成 .i 文件后用 vim 搜索, 或:
```bash
grep -nEC10 "struct dirent" main.i    # -C10 显示匹配行及其周围 10 行
```
- 类型别名同样可用此法查询

### seekdir / telldir: 移动目录流指针
```c
#include <dirent.h>
long telldir(DIR *dirp);
void seekdir(DIR *dirp, long loc);
```
- telldir: 返回当前读取位置的标记 (给 seekdir 用, 不作其他用途); 出错返回 -1, 一般不做错误处理
- seekdir: 设置目录流指针位置, **没有参照点概念, 只能使用 telldir 的返回值**; 无返回值 → 设计者认为不需要错误处理 (纯内存操作, 很安全)
- **readdir 的移动行为 (易错点)**: readdir 读取当前目录项并返回 dirent 指针后, 内部指针**自动前移到下一个条目**
  - 想记录 `.` 的位置 → 在 readdir 返回 `file1` 时 telldir
  - 想记录 `file1` 的位置 → 在 readdir 返回 `..` 时 telldir
  - 规律: **在 readdir 返回上一个目录项时, 记录下一个目录项的位置**

### rewinddir: 倒带目录流
```c
#include <dirent.h>
void rewinddir(DIR *dirp);
```
- 重置目录流位置到开始处 (类比文件流 rewind), 无返回值 → 不会出错
- 之后第一次 readdir 返回目录第一个条目

## stat 系统调用: 获取文件详细信息

> 实现 ls -al 前必须掌握; 目录项信息 (dirent) 只有文件名等少量信息, 权限/大小/时间等需要 stat

```c
#include <sys/stat.h>
int stat(const char *path, struct stat *buf);
```
- path: 文件的路径名; buf: 指向 stat 结构体的指针 (传入传出, 函数内部给它赋值, 类似 scanf)
- 成功返回 0, 失败返回 -1 并设置 errno (文件不存在/无权限)
- buf 内存由程序员管理, **推荐传入局部变量结构体指针** (栈自动管理)

### path 参数的问题 (文件名 ≠ 路径名)
- dirent 里的 d_name 是**文件名**, 不一定是 stat 需要的路径名:
  1. 若进程当前工作目录就是该目录 → 文件名即路径名 (相对路径)
  2. 否则需要 "目录路径/文件名" 拼成路径名
- 两种解决方式:
  1. `chdir` 切换进程工作目录到该目录 → 文件名即路径名
  2. 用 sprintf/strcpy+strcat **拼接绝对路径**: `sprintf(path, "%s/%s", dir, d_name)`
- 惯用方式: opendir + chdir + readdir + stat

### stat 结构体 (重点)
```c
struct stat {
    mode_t    st_mode;   // 文件类型 + 权限信息
    nlink_t   st_nlink;  // 硬链接数量
    uid_t     st_uid;    // 所有者用户 ID
    gid_t     st_gid;    // 所有者组 ID
    off_t     st_size;   // 文件实际大小 (字节)
    struct timespec st_mtim;  // 最后修改时间
};
```
- 64 位 Linux 下: mode_t = 32 位无符号; nlink_t = 64 位无符号; uid_t/gid_t = 32 位无符号; off_t = 64 位无符号
- timespec 结构体:
```c
struct timespec {
    __time_t tv_sec;    // 时间戳秒数 (一般就是 long)
    __syscall_slong_t tv_nsec;  // 纳秒, 补足秒以下部分
};
```
- **宏定义**: `#define st_mtime st_mtim.tv_sec`
  - `stat_buf.st_mtim.tv_sec` 与 `stat_buf.st_mtime` 完全等价
  - st_mtim 是结构体成员; st_mtime 是宏, 表示时间戳秒数

## 实现青春版 ls -al

### 步骤 1: 文件类型和权限 (st_mode 位运算)
- st_mode 是 32 位无符号整数, 实际只用:
  - 前 4~5 位: 文件类型
  - 最后 9 位: 权限
- 文件类型: `mode & S_IFMT` 取类型值, 再用 switch 匹配
```c
switch (sb.st_mode & S_IFMT) {
    case S_IFBLK:  tm_str[0] = 'b'; break;
    case S_IFCHR:  tm_str[0] = 'c'; break;
    case S_IFDIR:  tm_str[0] = 'd'; break;
    case S_IFIFO:  tm_str[0] = 'p'; break;
    case S_IFLNK:  tm_str[0] = 'l'; break;
    case S_IFREG:  tm_str[0] = '-'; break;
    case S_IFSOCK: tm_str[0] = 's'; break;
    default:       tm_str[0] = '?'; break;
}
```
- 权限: 低 9 位逐位判断
```c
void mode_to_string(mode_t mode, char str[10]) {
    str[0] = (mode & 0400) ? 'r' : '-';   // 所有者读
    str[1] = (mode & 0200) ? 'w' : '-';   // 所有者写
    str[2] = (mode & 0100) ? 'x' : '-';   // 所有者执行
    str[3] = (mode & 0040) ? 'r' : '-';   // 组读
    str[4] = (mode & 0020) ? 'w' : '-';
    str[5] = (mode & 0010) ? 'x' : '-';
    str[6] = (mode & 0004) ? 'r' : '-';   // 其他读
    str[7] = (mode & 0002) ? 'w' : '-';
    str[8] = (mode & 0001) ? 'x' : '-';
    str[9] = '\0';
}
```

### 步骤 2: 拥有者名和组名 (POSIX 库函数)
```c
#include <sys/types.h>
#include <pwd.h>
#include <grp.h>
struct passwd *getpwuid(uid_t uid);   // uid → 用户名
struct group *getgrgid(gid_t gid);    // gid → 组名
```
- 直接查系统用户/组数据库 (/etc/passwd, /etc/group), 不是系统调用
- 用法: `getpwuid(sb.st_uid)->pw_name`, `getgrgid(sb.st_gid)->gr_name`

### 步骤 3: 最后修改时间戳
```c
#include <time.h>
struct tm *localtime(const time_t *timer);
```
- 将时间戳转换为本地时间的 struct tm 结构体
- 传参: `localtime(&sb.st_mtim.tv_sec)` 或 `localtime(&sb.st_mtime)` (time_t 一般就是 long)
- tm 结构体取值注意: `tm_mon` 范围 [0,11], `tm_year` 从 1900 开始
- 返回静态分配的结构体, **不需要手动 free**

### 步骤 4: 最终实现
```c
/* Usage: ./07_myls pathname 或 ./07_myls (打印当前目录) */
int main(int argc, char* argv[]) {
    char* dir_name;
    if (argc == 1)      dir_name = ".";       // 无参数打印当前目录
    else if (argc == 2) dir_name = argv[1];
    else { fprintf(stderr, "args num error!\n"); exit(1); }

    DIR* dirp = opendir(dir_name);
    ERROR_CHECK(dirp, NULL, "opendir");
    int ret = chdir(dir_name);                 // 切换工作目录, 让文件名即路径名
    ERROR_CHECK(ret, -1, "chdir");

    struct dirent* pdirent;
    while ((pdirent = readdir(dirp)) != NULL) {
        struct stat stat_buf;
        int ret = stat(pdirent->d_name, &stat_buf);   // 不切换目录则需拼接路径
        ERROR_CHECK(ret, -1, "stat");

        char mode_str[1024] = { 0 };
        set_type_mode(stat_buf.st_mode, mode_str);    // 类型+权限字符串

        char time_str[1024] = { 0 };
        set_time(stat_buf.st_mtime, time_str);        // 时间字符串

        printf("%s %2lu %s %s %6lu %s %s\n",
               mode_str, stat_buf.st_nlink,
               getpwuid(stat_buf.st_uid)->pw_name,
               getgrgid(stat_buf.st_gid)->gr_name,
               stat_buf.st_size, time_str,
               pdirent->d_name);
    }
    closedir(dirp);
    return 0;
}
```
- 不 chdir 时拼接路径方案:
```c
char path[1024] = {0};
sprintf(path, "%s%s%s", argv[1], "/", pdirent->d_name);   // 或 strcpy + strcat
```

### 扩展: 排序功能 (qsort)
- 先遍历一遍数出目录项个数 count
- malloc 申请 `struct dirent**` 指针数组, rewinddir 后第二次遍历存入指针
- `qsort(dir_arr, count, sizeof(struct dirent*), compare)` 按字典序排序
- **注意**: 排序期间不能提前 closedir — readdir 返回的 dirent 内存随目录流关闭释放, 提前关闭会导致数据丢失
- 排序完打印, 最后 free 指针数组 + closedir
- compare 函数: 接收二级指针, `*(struct dirent**)a` 解引用; 转小写后 strcmp 比较

## 实现青春版 tree (DFS 递归)

- 思路: 打印根目录名 + 带缩进递归打印每个目录项 + 统计文件/目录数
- 树形结构 → **先序深度优先遍历 (DFS)**, 优先打印父目录再深入

```c
static int dirs = 0;
static int files = 0;

void DFS_print(char* dirpath, int width) {
    DIR* dirp = opendir(dirpath);
    ERROR_CHECK(dirp, NULL, "opendir");

    struct dirent* pdirent;
    while ((pdirent = readdir(dirp)) != NULL) {
        // 跳过 "." 和 ".."
        if (strcmp(pdirent->d_name, ".") == 0 || strcmp(pdirent->d_name, "..") == 0)
            continue;

        printf("└");
        for (int i = 1; i < width; ++i) printf("─");
        printf("%s\n", pdirent->d_name);          // 先序打印

        if (pdirent->d_type == DT_DIR) {          // 目录则递归
            dirs++;
            char path[1024] = { 0 };
            sprintf(path, "%s%s%s", dirpath, "/", pdirent->d_name);  // 拼接绝对路径
            DFS_print(path, width + 4);           // 增加缩进
        } else {
            files++;
        }
    }
    closedir(dirp);
}
```
- 两种实现方式:
  1. **拼接绝对路径** (推荐): 每次递归前 sprintf 拼出子目录路径, 代码直观
  2. **chdir 切换目录**: 递归进子目录前 chdir, 返回后 chdir(".."); 频繁切换工作目录思考难度大, 容易出错

## 练习

1. 写 chmod 程序: 命令行传入路径和八进制权限, 改变文件权限, 用 `ls -l` 验证
2. 写 getcwd + chdir 程序: 打印 chdir 前后的工作目录, 对比观察
3. 用 mkdir 创建 0777 权限目录, 观察实际权限为什么是 775 (umask)
4. 用 opendir + readdir 打印某目录下所有文件名 (破产版 ls), 注意用 closedir 释放
5. 给练习 4 加功能: 用 d_type 判断并标注每个是文件还是目录
6. 用 telldir/seekdir 实现: 记录 file1 位置后, 回到该位置重新读
7. 实现青春版 ls -al (含类型/权限/用户名/组名/时间), 支持无参数打印当前目录
8. 实现青春版 tree, 打印目录树和统计信息

## 自测

1. 系统调用、ISO-C 标准库、POSIX 库函数三者的跨平台性如何? 举例说明
2. man 手册卷 2 和卷 3 分别是什么? 查 open 用哪卷?
3. C 语言函数出错如何判断? 返回值有什么规律?
4. 指针参数加 const 和不加 const 分别说明什么? 以 stat 的 buf 参数为例
5. xx_t 是什么? 如何查看类型别名的具体定义? (两种方法)
6. chmod 函数设定权限后, umask 会不会再影响它?
7. getcwd(NULL, 0) 调用方式有什么注意点?
8. chdir 和 cd 有什么区别? 为什么?
9. rmdir 能删除非空目录吗?
10. 目录流中存储的是什么? 为什么目录流没有写操作?
11. readdir 返回的 dirent 结构体需要手动 free 吗? 为什么?
12. seekdir 能否任意指定位置? 必须配合什么函数使用?
13. readdir 读取后指针如何移动? 想记录 file1 的位置, 应在 readdir 返回什么时 telldir?
14. stat 的 path 参数, 文件名一定是路径名吗? 两种解决方式是什么?
15. st_mtime 和 st_mtim.tv_sec 是什么关系?
16. stat 结构体中哪个成员包含文件类型和权限? 如何用位运算取出类型?
17. getpwuid/getgrgid 是系统调用吗? 它们从哪里查数据?
18. 实现排序 ls -al 时, 为什么排序期间不能提前 closedir?

## 复习记录

- [ ] R1 (次日)
- [ ] R3 (3天后)
- [ ] R7 (7天后)
- [ ] R14 (14天后)
