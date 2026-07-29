# Day20 - 无缓冲文件流 / mmap / AI 工具

## 有缓冲 vs 无缓冲

  有缓冲 (fopen/fclose 家族):
    - 用户空间有缓冲区
    - 减少用户态到内核态的切换
    - 返回 FILE*

  无缓冲 (open/close 家族):
    - 用户空间无缓冲区
    - 每次调用都涉及内核态
    - 返回文件描述符 fd (int)
    - 是系统调用, POSIX标准

## open / close

  #include <sys/types.h>
  #include <sys/stat.h>
  #include <fcntl.h>
  #include <unistd.h>

  int fd = open("file", O_RDONLY);
  int fd = open("file", O_WRONLY | O_CREAT, 0644);
  close(fd);

  常用 flags:
    O_RDONLY   只读
    O_WRONLY   只写
    O_RDWR     读写
    O_CREAT    不存在则创建
    O_TRUNC    清空
    O_APPEND   追加

## read / write

  ssize_t read(int fd, void *buf, size_t count);
    - 返回实际读取字节数, 0=EOF, -1=错误

  ssize_t write(int fd, const void *buf, size_t count);
    - 返回实际写入字节数, -1=错误

## 文件描述符

  0: STDIN_FILENO  标准输入
  1: STDOUT_FILENO 标准输出
  2: STDERR_FILENO 标准错误
  3+: 打开的文件

  FILE* 底层的 fd 可通过 fileno(fp) 获取

## mmap (内存映射)

  #include <sys/mman.h>

  void *mmap(void *addr, size_t length, int prot,
             int flags, int fd, off_t offset);

  addr:   建议填 NULL (系统选)
  length: 映射长度
  prot:   PROT_READ | PROT_WRITE
  flags:  MAP_SHARED (修改回写文件)
  fd:     文件描述符
  offset: 偏移量

  返回值: 映射内存的起始地址

  munmap(addr, length);  // 解除映射

  mmap 特点:
    将文件直接映射到内存
    操作内存 = 操作文件
    适合大文件随机访问

## 重定向

  dup2(old_fd, new_fd);  // 复制文件描述符

  示例: 将标准输出重定向到文件
    int fd = open("out.txt", O_WRONLY | O_CREAT);
    dup2(fd, STDOUT_FILENO);
    printf("这行会写入文件");

## 代码示例

  #include <sys/types.h>
  #include <sys/stat.h>
  #include <fcntl.h>
  #include <unistd.h>
  #include <stdio.h>

  int main(void) {
      // 无缓冲文件读写
      int fd = open("test.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
      if (fd == -1) { perror("open"); return 1; }

      write(fd, "hello\n", 6);
      close(fd);

      // 读取
      fd = open("test.txt", O_RDONLY);
      char buf[100];
      int n = read(fd, buf, sizeof(buf) - 1);
      buf[n] = '\0';
      printf("read: %s", buf);
      close(fd);

      return 0;
  }

## 面试常问

  Q: fopen 和 open 的区别?
  A: fopen 是标准C库函数, 有缓冲, 返回 FILE*。
     open 是 POSIX 系统调用, 无缓冲, 返回 fd。
     fopen 底层实现就是 open。

  Q: mmap 的优势?
  A: 减少数据拷贝 (不需要 read/write 中间缓冲),
     适合大文件的随机访问, 操作方便。

## 练习

  1. 用 open/read/write 实现文件复制
  2. 用 mmap 读取文件并修改内容
  3. 实现 dup2 重定向: 将 printf 输出到文件

## 复习记录
  - [ ] R1 (次日)
  - [ ] R3 (3天后)
  - [ ] R7 (7天后)
  - [ ] R14 (14天后)
  - [ ] R30 (30天后)
