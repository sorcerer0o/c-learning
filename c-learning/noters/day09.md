# Day09 - 字符串进阶与命令行参数

## scanf 安全问题

  scanf 不会检查缓冲区大小, 输入过长会越界
  推荐:
    char buf[100];
    scanf("%99s", buf);          // 指定最大宽度
    fgets(buf, sizeof(buf), stdin);  // 更安全的替代

## 字符串数组

### 二维数组

  char strs[3][10] = {"hello", "world", "c"};
  - 连续内存, 内容可修改
  - 可能浪费空间 (每行固定长度)

### 指针数组

  char *strs[3] = {"hello", "world", "c"};
  - 指针在栈, 字符串在只读数据段
  - 不可修改字符串内容

## 命令行参数

  int main(int argc, char *argv[])

  argc: 参数个数 (含程序名)
  argv: 参数字符串数组
        argv[0] = 程序名
        argv[argc] = NULL

  示例: ./a.out hello world
    argc = 3
    argv = ["./a.out", "hello", "world", NULL]

## 代码示例

  #include <stdio.h>

  // 命令行参数: 实现简单的计算器
  // gcc calc.c -o calc
  // ./calc 10 + 20
  int main(int argc, char *argv[]) {
      if (argc != 4) {
          printf("用法: %s num1 op num2\n", argv[0]);
          printf("示例: %s 10 + 20\n", argv[0]);
          return 1;
      }

      double a = atof(argv[1]);
      double b = atof(argv[3]);
      char op = argv[2][0];

      switch (op) {
          case '+': printf("%.2f\n", a + b); break;
          case '-': printf("%.2f\n", a - b); break;
          case '*': printf("%.2f\n", a * b); break;
          case '/': printf("%.2f\n", a / b); break;
          default: printf("不支持的运算符\n");
      }
      return 0;
  }

## string.h 补充

  memset(p, 0, n)       将 p 前 n 字节置 0
  memcpy(d, s, n)       从 s 复制 n 字节到 d
  memmove(d, s, n)      同上, 但处理重叠区域
  strchr(s, c)          查找字符 c 首次位置
  strrchr(s, c)         查找字符 c 末次位置
  strstr(s1, s2)        查找子串 s2
  strtok(s, delim)      分割字符串 (会修改原串)

## 常见错误 / 盲点

  1. fgets 保留换行符:
     fgets(buf, n, stdin);
     buf[strcspn(buf, "\n")] = '\0';  // 去掉换行
  2. strtok 修改原字符串:
     不要传入字符串字面量 (只读)
  3. atoi/atof 无错误检测:
     推荐 strtol/strtod 替代

## 练习

  1. 实现命令行计算器: ./calc 3.14 * 2.5
  2. 用 fgets 和 sscanf 实现安全的输入解析
  3. 编写程序: 输入一行, 统计单词个数 (用 strtok)
  4. 编写程序: 输入文件名, 读取文件内容并输出 (复习文件操作)

## 复习记录
  - [ ] R1 (次日)
  - [ ] R3 (3天后)
  - [ ] R7 (7天后)
  - [ ] R14 (14天后)
  - [ ] R30 (30天后)
