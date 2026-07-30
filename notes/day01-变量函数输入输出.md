# Day01 - C语言基础(上)：变量、函数、输入输出

## 注释

  // 单行注释 (C99起支持)
  /* 多行注释 */  (不能嵌套)

## 关键字与标识符

### C90 的 32 个关键字

  auto     double   int       struct
  break    else     long      switch
  case     enum     register  typedef
  char     extern   return    union
  const    float    short     unsigned
  continue for      signed    void
  default  goto     sizeof    volatile
  do       if       static    while

  - 关键字全小写, 不可用作标识符
  - main 不是关键字, 是特殊标识符

### 标识符命名规则

  1. 首字符: 字母或下划线 _
  2. 组成: 字母, 数字, 下划线
  3. 区分大小写
  4. 见名知意, 全小写 + 下划线分隔, 如 student_name

### 变量三要素

  - 变量名
  - 变量类型
  - 变量值

### 声明 / 定义 / 初始化 / 赋值

  int x;           // 声明 + 定义 (分配内存)
  int x = 10;      // 初始化 (第一次赋值)
  x = 20;          // 赋值 (后续修改)

  - 局部变量不初始化 = 随机值 (未定义行为)
  - 全局变量不初始化 = 0 (C标准规定)

## 转义序列

  \n   换行
  \t   水平制表符
  \\   反斜杠
  \'   单引号
  \"   双引号
  \0   空字符 (编码值 0)
  \a   响铃
  \b   退格
  \r   回车 (回到行首)

## 函数

### 函数定义

  返回值类型 函数名(形参列表) {
      函数体
  }

  int add(int a, int b) {
      return a + b;
  }

### 函数声明 vs 定义

  int add(int a, int b);   // 声明 (告知编译器存在)
  int add(int a, int b) {  // 定义 (完整实现)
      return a + b;
  }

  - 被调函数定义在调用位置之后, 需先声明
  - 函数声明可以不写形参名: int add(int, int);
  - main 由操作系统调用, 一个项目只有一个 main
  - int main(void) 明确表示无参

### 函数的好处

  1. 复用逻辑, 避免重复代码
  2. 方便修改 (改一处即可)
  3. 代码更易读, 结构化

### #include 预处理

  #include <stdio.h>    // 标准库头文件
  #include "my.h"       // 自定义头文件

  - 作用: 将头文件内容复制到当前位置
  - <> 在系统路径搜索, "" 先搜索当前目录

## printf 基础

  格式: printf("格式字符串", 参数列表);

  占位符:
    %d     int
    %f     float
    %lf    double
    %c     char
    %s     字符串
    %p     指针地址
    %x     十六进制
    %u     无符号整数

  精度控制:
    %.2f    保留两位小数
    %5d     宽度5, 右对齐
    %-5d    宽度5, 左对齐

  示例:
    int age = 20;
    printf("age = %d\n", age);
    double pi = 3.14159;
    printf("pi = %.2f\n", pi);  // 输出: pi = 3.14

  - 换行用 \n, 它触发行缓冲区刷新

## scanf 基础

  格式: scanf("格式字符串", 变量地址);

  示例:
    int num;
    scanf("%d", &num);    // 注意 & 取地址

  要点:
    1. 从左到右匹配格式字符串
    2. 匹配成功继续, 失败直接结束
    3. 空白字符匹配任意空白 (空格/Tab/换行)
    4. 变量传参需取地址 & (数组名本身是地址, 不需要 &)
    5. %c 不会跳过空白字符, 需加空格: " %c"

  char ch;
  scanf(" %c", &ch);     // 加空格跳过前面的空白

## 变量地址

  - 变量占用多个字节, 地址取其首字节地址
  - 用 & 取地址, 用 %p 打印

  int x = 10;
  printf("%p\n", &x);    // 打印 x 的地址

## 代码示例

  // 输入两个整数, 输出它们的和
  #include <stdio.h>

  int main(void) {
      int a, b;
      printf("请输入两个整数: ");
      scanf("%d%d", &a, &b);
      int sum = a + b;
      printf("%d + %d = %d\n", a, b, sum);
      return 0;
  }

## 常见错误 / 盲点

  1. scanf 忘记加 &:
     scanf("%d", num);    // 错误, 应该是 &num
  2. 局部变量未初始化:
     int x; printf("%d", x);  // 随机值, 未定义行为
  3. %c 不跳过空白:
     scanf("%c", &ch);    // 会读取到之前的换行符
  4. main 写错:
     void main()          // 非标准, 应使用 int main(void)

## 面试常问

  Q: 变量的声明和定义有什么区别?
  A: 声明告诉编译器变量的类型和名字, 不分配内存;
     定义分配内存空间。int x; 既是声明也是定义。
     extern int x; 只是声明, 不分配内存。

  Q: int main(void) 和 int main() 的区别?
  A: void 明确表示无参; 空括号在C中表示参数未知(旧风格)。
     推荐使用 int main(void)。

## 练习

  1. 编写程序输入你的姓名和年龄, 输出自我介绍
  2. 输入两个浮点数, 计算它们的乘积并保留两位小数输出
  3. 输入一个字符, 输出它的 ASCII 码值 (%d 打印 char)

## 自测

  1. C90 有多少个关键字？main 是关键字吗？
  2. 声明和定义有什么区别？举例说明。
  3. printf 和 scanf 中 %d、%f、%lf、%c 分别对应什么类型？
  4. scanf 为什么需要 &？什么情况下不需要？
  5. %c 有什么特殊之处？如何跳过空白字符？
  6. int main(void) 和 int main() 的区别？
  7. 局部变量不初始化会怎样？全局变量呢？
  8. 转义序列 \n、\t、\0、\r 分别代表什么？
  9. #include <> 和 #include "" 有什么区别？
  10. 写一个完整的 C 程序：输入两个整数，输出它们的和。

## 复习记录
  - [x] R1 (次日)
  - [ ] R3 (3天后)
  - [ ] R7 (7天后)
  - [ ] R14 (14天后)
  - [ ] R30 (30天后)
