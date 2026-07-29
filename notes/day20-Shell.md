# Day20 - Shell 编程

## Shell 基础

### Shell 命令组成
```
命令 [选项] [参数]
```
- **命令**: 核心可执行程序（ls/cp/mv）或 shell 内建（cd/echo/pwd）
- **选项**: 以 `-` 开头修改行为（`-l`, `-a`, `--help`）
- **参数**: 命令操作的目标

### 内建命令 vs 外部程序
- 内建: shell 进程自带（cd/echo/pwd/exit/alias/history）
- 外部: 独立可执行文件（ls/cp/mv/gcc/vim）
- `type 命令` - 查看命令类型
- `which 命令` - 查找外部程序路径

### 引号规则
- 双引号 `"..."`: 保留空格，解析变量 `$VAR`
- 单引号 `'...'`: 纯字面量，不解析任何特殊字符

## 常用 Shell 命令

### 文件操作
| 命令 | 功能 | 常用选项 |
|------|------|----------|
| `ls` | 列出目录 | `-l` 详细, `-a` 隐藏文件, `-h` 人性化大小 |
| `cp` | 复制 | `-r` 递归目录, `-i` 交互提示 |
| `mv` | 移动/重命名 | `-i` 交互提示 |
| `rm` | 删除 | `-r` 递归, `-f` 强制, `-i` 交互 |
| `mkdir` | 创建目录 | `-p` 递归创建父目录 |
| `touch` | 创建空文件/更新修改时间 |
| `cat` | 查看文件内容 | `-n` 显示行号 |
| `less` | 分页查看（支持上下翻页） |
| `head` | 查看前 N 行 | `-n N` |
| `tail` | 查看后 N 行 | `-f` 实时跟踪 |

### 查找与过滤
| 命令 | 功能 | 示例 |
|------|------|------|
| `grep` | 文本搜索 | `grep -rn "keyword" .` |
| `find` | 查找文件 | `find . -name "*.c"` |
| `wc` | 计数 | `-l` 行数, `-w` 单词数, `-c` 字符数 |
| `sort` | 排序 | `-n` 数字排序, `-r` 逆序 |
| `uniq` | 去重 | `-c` 统计出现次数 |

### 文件信息
- `stat 文件` - 查看文件详细信息（大小/权限/时间等）
- `file 文件` - 判断文件类型
- `du -sh 目录` - 查看目录总大小
- `df -h` - 查看磁盘使用情况

### 网络与系统
- `ping` - 网络连通测试
- `ifconfig` / `ip a` - 查看网络配置
- `ps aux` - 查看所有进程
- `top` - 实时进程监控
- `kill PID` - 终止进程

## 重定向与管道

### 重定向
| 符号 | 功能 | 示例 |
|------|------|------|
| `>` | 覆盖输出到文件 | `ls > out.txt` |
| `>>` | 追加输出到文件 | `echo "hi" >> log.txt` |
| `<` | 从文件读取输入 | `sort < input.txt` |
| `2>` | 重定向错误输出 | `gcc a.c 2> err.log` |
| `&>` | 重定向全部输出 | `cmd &> all.log` |
| `<<` | Here Document | `cat << EOF` |

### 管道
```bash
命令1 | 命令2 | 命令3
```
- 前一个命令的 stdout 连接到后一个命令的 stdin
- 示例:
  - `ls -la | grep "\.c$"` - 查找 .c 文件
  - `ps aux | grep vim` - 查找 vim 进程
  - `cat file.txt | wc -l` - 统计行数
  - `history | grep git` - 查找 git 相关历史命令

## Shell 脚本基础

### 脚本结构
```bash
#!/bin/bash
# 注释用 #
echo "Hello, World!"
```

### 变量
```bash
name="Alice"                # 定义变量（= 两边不能有空格）
echo $name                   # 使用变量加 $
readonly name                # 只读变量
unset name                   # 删除变量
```

### 特殊变量
| 变量 | 含义 |
|------|------|
| `$0` | 脚本名 |
| `$1`-`$9` | 第 1-9 个参数 |
| `$#` | 参数个数 |
| `$@` | 所有参数列表 |
| `$?` | 上一条命令退出码（0=成功） |
| `$$` | 当前进程 PID |

### 条件判断
```bash
if [ 条件 ]; then
    # 执行
elif [ 条件 ]; then
    # 执行
else
    # 执行
fi
```

常用条件:
- `-f 文件`: 是否为普通文件
- `-d 目录`: 是否为目录
- `-z 字符串`: 是否为空
- `-n 字符串`: 是否非空
- `str1 = str2`: 字符串相等
- `num1 -eq num2`: 数字相等, `-ne`不等, `-gt`大于, `-lt`小于

### 循环
```bash
# for 循环
for i in 1 2 3 4 5; do
    echo $i
done

for file in *.c; do
    echo $file
done

# while 循环
count=1
while [ $count -le 5 ]; do
    echo $count
    count=$((count + 1))
done
```

### 函数
```bash
function hello {
    echo "Hello, $1!"
}
hello "World"    # 输出: Hello, World!
```

## 进程管理

### 进程查看
- `ps` - 当前终端进程
- `ps aux` - 所有进程（BSD风格）
- `ps -ef` - 所有进程（System V风格）
- `top` - 实时进程列表（q 退出）
- `htop` - top 增强版（如有安装）
- `pstree` - 进程树

### 进程控制
- `kill PID` - 终止进程（默认 SIGTERM）
- `kill -9 PID` - 强制终止（SIGKILL）
- `killall 进程名` - 按名终止
- `pkill 进程名` - 按名终止（支持正则）

### 后台运行
- `命令 &` - 后台运行
- `jobs` - 查看后台任务
- `fg %1` - 后台任务调到前台
- `bg %1` - 暂停任务后台继续
- `nohup 命令 &` - 终端退出后继续运行

## 权限管理

### 文件权限
```bash
-rwxr-xr-x   # 类型(1) + 所有者(3) + 组(3) + 其他(3)
```

| 权限 | 数字 | 含义 |
|------|------|------|
| `r` | 4 | 读 |
| `w` | 2 | 写 |
| `x` | 1 | 执行 |

### chmod
```bash
chmod 755 file        # 数字法: rwxr-xr-x
chmod u+x file        # 符号法: 所有者加执行
chmod g-w file        # 组去掉写
chmod o=r file        # 其他人设为只读
```

### chown / chgrp
```bash
chown user:group file   # 修改所有者和组
chown user file         # 仅修改所有者
chgrp group file        # 仅修改组
chown -R user dir/      # 递归修改目录
```

### 特殊权限
- `chmod +s file` - SUID（运行时以所有者权限执行）
- `chmod +t dir/` - 粘滞位（仅所有者可删自己的文件，如 /tmp）

## 实用技巧

### 历史命令
- `history` - 查看历史
- `!!` - 重复上一条命令
- `!$` - 上一条命令的最后一个参数
- `Ctrl+R` - 搜索历史命令

### 快捷键
- `Tab` - 自动补全
- `Ctrl+C` - 中断当前命令
- `Ctrl+Z` - 暂停当前命令
- `Ctrl+D` - 退出终端
- `Ctrl+A` - 行首 / `Ctrl+E` - 行尾
- `Ctrl+U` - 删除到行首 / `Ctrl+K` - 删除到行尾

## 自测

> 复习时先看自测题，想得出就不看上面内容，想不出再翻。

1. 查找当前目录下所有 .c 文件的 Shell 命令怎么写？
2. `>` 和 `>>` 有什么区别？
3. Shell 脚本里 `$?` 和 `$#` 分别表示什么？
4. chmod 755 是什么意思？
5. kill -9 和 kill 的区别？
6. 管道 `|` 的作用是什么？举例说明。
7. 写一个 for 循环遍历当前目录所有 `.md` 文件并打印文件名。
8. `ps aux` 和 `ps -ef` 有什么区别？
9. 如何把命令的错误输出重定向到文件？
10. 单引号和双引号在 Shell 中有什么区别？
