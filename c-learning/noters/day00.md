# Day00 - 环境搭建

## 开发工具链

### 编译器
- MinGW-w64 (Windows)
  - 下载: https://sourceforge.net/projects/mingw-w64/
  - 安装后配置 PATH 环境变量
  - 验证: gcc --version
- GCC (Linux/WSL)
  - sudo apt install gcc g++ gdb make
  - 验证: gcc --version, gdb --version

### IDE / 编辑器
- Visual Studio (Windows)
  - 社区版免费
  - 创建C++控制台应用
  - VS 调试功能强大
- VS Code
  - 插件: C/C++ (Microsoft), Code Runner
- Vim (Linux/WSL)
  - sudo apt install vim
  - 模式: 普通模式 / 插入模式 / 可视模式

### 调试器
- GDB (命令行)
  - gcc -g main.c -o main    # 编译时加 -g 生成调试信息
  - gdb ./main               # 启动调试
  - break main               # 打断点
  - run                       # 运行
  - next / step              # 逐过程 / 逐语句
  - print var                # 打印变量
  - quit                      # 退出

- VS 调试
  - F9: 打断点
  - F10: 逐过程
  - F11: 逐语句
  - 监视窗口: 查看变量变化
  - 调用堆栈: 查看函数调用关系

### 版本控制
- Git
  - git --version
  - git clone <url>
  - git pull                    # 拉取最新
  - git add . && git commit     # 提交
  - git push                    # 推送

## Linux / WSL 环境

### Windows Subsystem for Linux (WSL)
  - wsl --install               # 安装
  - wsl                         # 进入
  - 文件互通: \\wsl$\Ubuntu\

### 路径注意事项
  - Windows: 盘符开头, 如 C:\Users\
  - Linux: / 开头, 如 /home/user/
  - WSL: /mnt/c/Users/ 访问 Windows 文件

### 编译步骤 (GCC 四阶段)
  1. 预处理: gcc -E hello.c -o hello.i     (展开头文件/宏)
  2. 编译:   gcc -S hello.i -o hello.s     (生成汇编)
  3. 汇编:   gcc -c hello.s -o hello.o     (生成机器码)
  4. 链接:   gcc hello.o -o hello          (生成可执行文件)

  一步到位: gcc hello.c -o hello

## 第一个C程序

  #include <stdio.h>

  int main(void) {
      printf("hello, world\n");
      return 0;
  }

  // 编译: gcc hello.c -o hello
  // 运行: ./hello (Linux) 或 hello.exe (Windows)

## 学习建议 (来自课程)

  - 基础知识 + 实际代码能力 + debug能力 + 沟通能力 + 写文档能力
  - 晚自习多敲代码, 上课代码要自己敲一遍
  - 学会用自己的话描述知识点 (面试时很重要)
  - 使用 AI 辅助但不依赖 AI
  - 碰到 bug 的排查顺序:
    1. 自己尝试 (打印/debug)
    2. 搜索 (CSDN/StackOverflow/Google)
    3. 问同事/同学
    4. 问 leader

## 复习记录
  - [ ] R1 (次日)
  - [ ] R3 (3天后)
  - [ ] R7 (7天后)
  - [ ] R14 (14天后)
  - [ ] R30 (30天后)
