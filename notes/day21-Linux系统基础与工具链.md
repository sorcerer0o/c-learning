# Day21 - Linux 系统基础

## Linux 文件系统

### 虚拟文件系统 (VFS)
- VFS 是 Linux 内核的抽象层, 提供统一的文件操作接口
- 用户通过 VFS 访问不同的物理文件系统 (ext4, NTFS, FAT32 等)
- VFS → 映射 → 具体文件系统 → 磁盘设备

### inode
- inode (索引节点) 存储文件的元数据: 权限, 大小, 时间, 数据块指针
- 文件名存在目录项中: 文件名 → 目录项 → inode → 数据块
- `ls -i` 查看 inode 编号; `stat 文件` 查看 inode 详细信息

### 硬链接 vs 软链接

| 特性 | 硬链接 | 软链接 (符号链接) |
|------|--------|-------------------|
| inode | 共享同一 inode | 独立 inode |
| 跨文件系统 | 否 | 是 |
| 链接目录 | 否 | 是 |
| 原文件删除后 | 链接仍可用 (计数减1) | 失效 (死链接) |
| 原理 | 多个目录项指向同一 inode | 类似快捷方式/指针 |

```bash
ln file1 file2       # 创建硬链接
ln -s file1 slink    # 创建软链接
```

### 目录的链接数
- 新目录默认有 `.` (自己) 和 `..` (父目录), 所以链接数 = 2
- 每创建一个子目录, 父目录链接数 +1
- `ls -ld 目录` 查看链接数

### 查看磁盘信息
- `df -h` 查看磁盘分区使用情况
- `du -sh 目录` 查看目录总大小

## 文件权限

### 权限分类
```
-rwxr-xr-x
| ||||||||
| |||||||+-- 其他用户权限 (others)
| |||||+---- 同组用户权限 (group)
| |||+------ 文件所有者权限 (owner)
| |+-------- 文件类型: -普通文件, d目录, l符号链接, c字符设备, b块设备, s本地套字, p管道
```

### 权限数字表示
```
rwx = 111 = 7    rw- = 110 = 6    r-x = 101 = 5
r-- = 100 = 4    --- = 000 = 0
```
- `chmod 755 file` 等价于 `chmod u=rwx,g=rx,o=rx`
- `chmod u+x file` 给所有者加执行权限
- `chown user:group file` 修改文件所有者和组
- `chgrp group file` 修改文件所属组

### umask 权限掩码
```bash
umask          # 查看当前掩码
umask 022      # 设置掩码
```
- 实际权限 = 默认权限 & ~umask
- 文件默认 666 (rw-rw-rw-), 目录默认 777 (rwxrwxrwx)
- umask 022 → 文件: 644, 目录: 755

### 目录权限详解
- `r` — 可列出目录内容 (ls)
- `w` — 可在目录中创建/删除文件
- `x` — 可进入目录 (cd), **x 是目录的基础权限, 缺失将导致无法访问**

### sudo 与重定向的坑
```bash
sudo echo "test" > /root/file    # 失败! > 由当前 shell 执行, sudo 无效
echo "test" | sudo tee /root/file # 正确
sudo sh -c 'echo "test" > /root/file' # 正确
```

## Vim 编辑器

### 三大模式
- **普通模式** (默认): 执行命令, 移动光标, 删除/复制文本 —— Vim 的核心
- **插入模式**: 编辑文本, `i` 光标前插入, `a` 光标后插入, `o` 下行插入, `O` 上行插入
- **可视模式**: 选择文本, `v` 行选, `Ctrl-v` 竖选/块选

模式切换: ESC 回普通模式; i/a/o 进插入模式; v/Ctrl-v 进可视模式

### 光标移动
| 命令 | 作用 |
|------|------|
| `h` `j` `k` `l` | 左 下 上 右 |
| `gg` | 跳到文件第一行 |
| `G` | 跳到文件最后一行 |
| `:n` 或 `nG` | 跳到第 n 行 |
| `^` | 跳到行首非空字符 |
| `$` | 跳到行尾 |
| `w` | 跳到下一单词词首 |
| `b` | 跳到上一单词词首 |
| `fX` | 向右跳到字符 X |
| `FX` | 向左跳到字符 X |
| `%` | 在匹配括号间跳转 |

### 删除 (实际是剪切)
| 命令 | 作用 |
|------|------|
| `x` | 删除一个字符 |
| `dd` | 删除当前行 |
| `[n]dd` | 向下删除 n 行 |
| `d^` | 删到行首 |
| `d$` | 删到行尾 |
| `diw` | 删除光标所在单词 |
| `di"` | 删除 "" 内内容 |
| `di(` 或 `di)` | 删除 () 内内容 |
| `dfX` | 向后删到字符 X (含 X) |
| `dtX` | 向后删到字符 X (不含 X) |

### 复制与粘贴
| 命令 | 作用 |
|------|------|
| `yy` | 复制当前行 |
| `yyp` | 复制当前行到下一行 (常用) |
| `:m,ny` | 复制 [m,n] 行 |
| `p` | 粘贴到下一行/光标后 |
| `P` | 粘贴到上一行/光标前 |

### 修改 (删除 + 进入插入模式)
| 命令 | 作用 |
|------|------|
| `cc` 或 `S` | 删除当前行并进入插入模式 |
| `ciw` | 删除当前单词并进入插入模式 |
| `ci"` | 删除 "" 内容并进入插入模式 |
| `ci(` 或 `ci)` | 删除 () 内容并进入插入模式 |

### 查找与替换
```vim
/pattern   " 向后搜索 (回车确认, n/N 切换)
?pattern   " 向前搜索
:s/old/new/g      " 当前行全部替换
:5,10s/old/new/g  " 第5-10行替换
:%s/old/new/g     " 全文替换
:%s/old/new/gc    " 全文替换, 逐个确认
```

### 撤销与重做
- `u` 撤销 (Undo)
- `Ctrl-r` 重做 (Redo)

### 可视模式实用技巧
**批量注释多行:**
1. 光标移到起始行行首
2. `Ctrl-v` 进入竖选模式, 选中多行
3. 按 `I` (大写 i), 输入 `//`
4. 按 ESC 退出, 所有选中行自动添加注释

**批量取消注释:**
1. `Ctrl-v` 竖选注释符 `//`
2. 按 `d` 删除

### 多窗口操作
| 命令 | 作用 |
|------|------|
| `:sp [文件]` | 水平分割窗口 |
| `:vsp [文件]` | 垂直分割窗口 |
| `Ctrl-w w` | 循环切换窗口 |
| `:qall` | 退出所有窗口 |
| `:wqall` | 保存并退出所有窗口 |

### Vim 配置文件
- `~/.vimrc` — Vim 的启动配置文件
- 常用配置: `set number` 显示行号, `set hlsearch` 高亮搜索

## GCC 编译工具链

### GCC vs Clang
- **GCC** (GNU Compiler Collection): Linux 默认编译器, 稳定成熟
- **Clang**: 基于 LLVM, 错误信息更友好, 兼容 GCC

### 编译四步骤
| 选项 | 输出 | 阶段 | 说明 |
|------|------|------|------|
| `-E` | `.i` | 预处理 | 展开宏/头文件, 删注释 |
| `-S` | `.s` | 编译 | 生成汇编代码 |
| `-c` | `.o` | 汇编 | 生成目标文件 (机器码) |
| (无选项) | a.out | 链接 | 链接生成可执行文件 |

```bash
gcc -E main.c -o main.i      # 预处理
gcc -S main.i -o main.s      # 编译
gcc -c main.s -o main.o      # 汇编
gcc main.o -o main           # 链接
gcc main.c -o main           # 一步到位 (最常用)
```

### 常用选项
| 选项 | 作用 |
|------|------|
| `-o 文件名` | 指定输出文件名 |
| `-Wall` | 显示所有警告 (推荐总是加上) |
| `-g` | 生成调试信息 (供 GDB 使用) |
| `-O0/-O1/-O2/-O3` | 优化级别, 默认 -O1 |
| `-D宏名` | 定义宏, 等价于 `#define 宏名` |
| `-D宏名=值` | 定义宏并赋值 |
| `-I目录` | 添加头文件搜索路径 |

```bash
gcc main.c -o main -Wall -g          # 常用开发编译
gcc main.c -o main -O2               # 生产环境编译
gcc main.c -o main -DDEBUG           # 定义 DEBUG 宏
gcc main.c -o main -I../header       # 指定头文件路径
```

### 条件编译
```c
#ifdef DEBUG
    printf("调试信息\n");
#endif

#ifndef HEADER_H
#define HEADER_H
// 头文件保护
#endif
```
- 通过 `gcc -DDEBUG` 控制调试代码是否包含
- 也用于跨平台代码: `#ifdef WIN32` / `#ifdef __linux__`

## 通配符

| 通配符 | 含义 | 示例 |
|--------|------|------|
| `*` | 匹配 0 个或多个字符 | `*.c` 所有 .c 文件 |
| `?` | 匹配任意 1 个字符 | `file.?` file.c file.h |
| `[...]` | 匹配括号内任意字符 | `[abc]*` a/b/c 开头 |
| `[!...]` | 匹配不在括号内的字符 | `[!a]*` 非 a 开头 |

## 查找命令

### which / whereis / locate
- `which 命令` — 在 PATH 中查找可执行文件路径
- `whereis 命令` — 查找二进制/源码/手册页
- `locate 文件名` — 基于数据库快速查找, 支持通配符, 需定期 `updatedb`

### find 详细用法
```bash
find [起始目录] [选项]

# 按名称
find . -name "*.c"                       # .c 文件
find /usr -name "stdio.h"                # 查找 stdio.h

# 按类型
find . -type f                           # 普通文件 (file)
find /dev -type b                        # 块设备 (block)
find /dev -type c                        # 字符设备 (character)

# 按大小
find . -size +100k                       # 大于 100KB
find . -size -1M                         # 小于 1MB
find . -size 5M                          # 精确等于 5MB

# 按权限/用户
find . -perm 644                         # 权限为 644
find . -user root                        # root 用户的文件

# 按时间
find . -mtime +7                         # 7 天前修改过的文件
find . -amin -30                         # 30 分钟内访问过的文件

# 组合条件
find . -name "*.c" -a -size +100k        # 与 (默认, -a 可省略)
find . -type f -o -type d                # 或
find . -type f ! -name "*.c"             # 非

# 限制深度
find . -maxdepth 1 -name "*.md"          # 只在当前目录
```

### grep 与正则表达式基础
```bash
grep -rn "pattern" .                     # 递归搜索文件内容
grep -nE "\<main\(" *.c                  # 匹配 main( 单词边界
grep -niE "error" log.txt                # 忽略大小写
grep -cE "hello" file.txt                # 只计数
```
- `^` 行首, `$` 行尾, `.` 任意字符, `*` 重复 0 次或多次
- `+` 重复 1 次或多次, `?` 重复 0 或 1 次
- `\<` 单词开头, `\>` 单词结尾
- `[abc]` 集合匹配, `[^abc]` 取反

### find + xargs + grep 组合
```bash
find . -name "*.c" | xargs grep -nE "\<main\("
# 在所有 .c 文件中查找含有 main 函数的行
```

## 归档压缩

```bash
tar cfv package.tar test*               # 归档 (不压缩)
tar cfvz package.tar.gz test*           # 归档 + gzip 压缩
tar xfv package.tar                     # 解归档
tar xfvz package.tar.gz                 # 解压缩
tar tfv package.tar                     # 查看归档内容
```

## xxd 与 nm

- `xxd 文件` — 以十六进制查看文件内容
- `nm 目标文件` — 查看目标文件的符号表
  - `T` 代码段 (函数), `D` 已初始化全局变量
  - `U` 未定义符号 (需链接), `B` 未初始化全局变量

## 练习

1. 创建一个硬链接和一个软链接, 验证原文件删除后的行为差异
2. 用 umask 设置掩码为 027, 验证文件和目录的默认权限
3. 用 Vim 打开一个 C 文件, 练习光标移动, 复制粘贴, 查找替换
4. 用 Vim 的竖选模式给 5 行代码批量添加注释
5. 用 find + xargs + grep 查找当前目录下所有 .md 文件中包含 "Linux" 的行
6. 用 tar 打包压缩一个目录

## 自测

1. VFS 的作用是什么？硬链接和软链接有什么区别？
2. 目录的链接数为什么从 2 开始？
3. umask 022 时新文件和新目录的默认权限是什么？
4. 目录的 x 权限缺失会导致什么问题？
5. Vim 有哪三大模式？如何切换？
6. Vim 中 `di(`、`ci"`、`df)`、`yyp` 分别做什么？
7. 如何用 Vim 批量注释多行代码？
8. GCC 编译的四个步骤是什么？`-Wall -g -DDEBUG` 各有什么用？
9. find 命令中 `-type f -size +100k -name "*.c"` 表示什么？
10. tar cfvz 中的 c、f、v、z 分别代表什么？

## 复习记录

- [x] R1 (次日)
- [ ] R3 (3天后)
- [ ] R7 (7天后)
- [ ] R14 (14天后)
- [ ] R30 (30天后)
