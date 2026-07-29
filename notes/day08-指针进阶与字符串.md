# Day08 - 指针进阶与字符串

## 指针运算

  arr[2]       == *(arr + 2)
  arr[i][j]    == *(*(arr + i) + j)

## 数组指针 vs 指针数组

  区分方法: 看最后一个词 (后缀是什么, 它就是什么)
           看 * 和 [] 的优先级

  int *p[5];       // 指针数组: 元素是指针的数组 (本质是数组)
  int (*p)[5];     // 数组指针: 指向整个数组的指针 (本质是指针)

  数组指针偏移:
    int arr[5];
    int (*p)[5] = &arr;
    p + 1  // 跳过整个数组 (5*sizeof(int) 字节)

## const 与指针

  原则: const 右边修饰谁, 谁就不能变

  const int *p;        // *p 不可改, p 可改
  int const *p;        // 同上
  int * const p;       // p 不可改, *p 可改
  const int * const p; // 两者都不可改

## 字符串

### 概念
  - 字符串 = 字符数组 + \0 结尾
  - C 字符串缺点: 无法存 \0, 求长度 O(n)

### 字符串字面值
  - 存储在只读数据段
  - 特点: 只读, 生命周期长, \0 结尾

### 字符串输入输出

  // scanf: 遇空白停止, 自动加 \0
  char str[100];
  scanf("%99s", str);   // 指定最大长度, 安全

  // gets: 不安全, 无长度限制 (不要用)
  // fgets: 安全替代
  fgets(str, sizeof(str), stdin);  // 保留换行符

  // puts: 输出并换行
  puts(str);

## string.h 常用函数

  strlen(s)      返回长度 (不含 \0), O(n)
  strcpy(d,s)    复制到 dst, 含 \0
  strcat(d,s)    追加到 dst 末尾
  strcmp(s1,s2)  比较: 0相等, 负s1<s2, 正s1>s2
  strncpy(d,s,n) 安全版, 最多复制 n 个字符
  strncat(d,s,n) 安全版, 最多追加 n 个字符
  strncmp(s1,s2,n) 最多比较前 n 个字符

  memset(s, 0, n)   内存置零
  memcpy(d, s, n)   内存拷贝
  strchr(s, c)      查找字符第一次出现位置
  strstr(s1, s2)    查找子串

  ️ strncpy 在 src 长度 >= n 时不会加 \0, 需手动添加

## 字符串数组

  二维数组:
    char strs[3][10] = {"hello", "world", "c"};
    // 连续内存, 内容可改, 可能浪费空间

  指针数组:
    char *strs[3] = {"hello", "world", "c"};
    // 字符串在只读区, 不可修改

## 代码示例

  #include <stdio.h>
  #include <string.h>

  int main(void) {
      // 数组指针 vs 指针数组
      int arr[5] = {1,2,3,4,5};
      int (*pa)[5] = &arr;      // 数组指针
      int *parr[5];             // 指针数组 (未初始化)

      // 字符串
      char s1[] = "hello";
      char *s2 = "world";       // 只读, 不能修改

      printf("strlen: %zu\n", strlen(s1));  // 5
      strcat(s1, " world");     // 拼接
      printf("strcat: %s\n", s1);

      // 安全复制
      char buf[10];
      strncpy(buf, "hello world", sizeof(buf) - 1);
      buf[sizeof(buf) - 1] = '\0';

      // 查找
      char *p = strchr(s1, 'o');
      if (p) printf("found: %c\n", *p);

      return 0;
  }

## 常见错误 / 盲点

  1. 修改字符串字面量:
     char *p = "hello"; p[0] = 'H';  // 崩溃! 只读段
  2. strcpy 缓冲区溢出:
     char buf[5]; strcpy(buf, "hello world");  // 越界!
  3. strncpy 不加 \0:
     strncpy(buf, src, n); buf[n] = '\0';  // 手动加!
  4. gets 使用: 永远不要用 gets

## 面试常问

  Q: 数组指针和指针数组的区别?
  A: int *p[5] 是指针数组, 存指针的数组
     int (*p)[5] 是数组指针, 指向数组的指针

  Q: const int *p 和 int * const p 的区别?
  A: const int *p: p 可变, *p 不可变
     int * const p: p 不可变, *p 可变

## 练习

  1. 实现自己的 strlen, strcpy, strcmp (不用库函数)
  2. 输入一行字符串, 统计其中的单词个数
  3. 输入两个字符串, 判断是否互为回文
  4. 用指针数组存储星期名称, 输入数字输出对应星期

## 自测

  1. 数组指针和指针数组如何区分？写一个声明示例。
  2. const int *p 和 int * const p 的区别是什么？
  3. 字符串字面值存在哪个内存区域？可以修改吗？
  4. char s[] = "hello" 和 char *s = "hello" 有什么区别？
  5. strlen 的返回值是什么？它是否包含 \0？
  6. strncpy 有什么安全陷阱？如何正确处理？
  7. 为什么永远不要用 gets？应该用什么替代？
  8. 字符串数组用二维 char 数组和指针数组各有什么优缺点？
  9. strcmp 的返回值有什么含义？
  10. strchr 和 strstr 分别用来做什么？

## 复习记录
  - [ ] R1 (次日)
  - [ ] R3 (3天后)
  - [ ] R7 (7天后)
  - [ ] R14 (14天后)
  - [ ] R30 (30天后)
