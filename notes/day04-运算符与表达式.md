# Day04 - 运算符与表达式

## 运算符优先级 (从高到低)

  1   ()  []  ->  .         函数调用/下标/成员访问       左->右
  2   ! ~ ++ -- + - * &    单目运算符 (正负号/指针/取址) 右->左
      (type) sizeof         强制类型转换/sizeof
  3   * / %                 乘除取余                      左->右
  4   + -                   加减                          左->右
  5   << >>                 移位                          左->右
  6   < <= > >=             关系比较                      左->右
  7   == !=                 相等性比较                    左->右
  8   &                     按位与                        左->右
  9   ^                     按位异或                      左->右
  10  |                     按位或                        左->右
  11  &&                    逻辑与                        左->右
  12  ||                    逻辑或                        左->右
  13  ? :                   三目条件                      右->左
  14  = += -= *= /= %=      赋值                          右->左
      <<= >>= &= ^= |=
  15  ,                     逗号                          左->右

  ️ 建议: 不要死记硬背, 不确定就加括号

## 短路求值

  a && b    若 a 为假, 不计算 b
  a || b    若 a 为真, 不计算 b

  利用短路可以写出简洁代码:
    while (p && p->next)  // 先判空, 再访问成员
    if (ptr && *ptr == 5) // 先判指针有效性

## 交换两个元素

### 基础方法 (临时变量)

  void swap(int *a, int *b) {
      int temp = *a;
      *a = *b;
      *b = temp;
  }

### XOR 位运算法

  void swap(int *a, int *b) {
      *a = *a ^ *b;
      *b = *a ^ *b;
      *a = *a ^ *b;
  }

  优点: 不占额外内存
  缺点: 仅整数, 交换同一元素会变0

## 类型转换

### 隐式类型转换

  整数提升:
    char + char -> int
    short + short -> int

  常用算术转换: 结果取范围最大的类型
    int + double -> double
    float + int -> float

### 无符号陷阱 ⚠️

  int a = -10;
  unsigned b = 100;
  if (b > a)     // false! -10 转无符号变成大正数
  建议: 避免混用有符号和无符号

### 显式类型转换

  (type)expr
  printf("%d\n", (int)3.14);       // 3
  printf("%.2f\n", (float)5 / 2);  // 2.50

## 固定宽度整数类型

  #include <stdint.h>
  int8_t   int16_t   int32_t   int64_t
  uint8_t  uint16_t  uint32_t  uint64_t

## 布尔值

  C语言中: 0 为假, 非零为真
  #include <stdbool.h>  // 可使用 bool / true / false

## break / return / continue

  break     跳出当前循环 (for/while/do-while) 或 switch
  continue  跳过本轮循环, 进入下一轮
  return    结束当前函数, 返回调用处

  for (int i = 0; i < 10; i++) {
      if (i == 3) continue;  // 跳过3
      if (i == 7) break;     // 到7跳出
      printf("%d ", i);      // 输出: 0 1 2 4 5 6
  }

## 类型别名 typedef

  typedef 原类型 新名称;
  typedef unsigned long long ull;  // ull a = 100;
  typedef int* p_int;              // p_int p; 等价 int *p;

  xxx_t 通常是类型别名 (size_t, time_t)

## 代码示例

  #include <stdio.h>

  int main(void) {
      // 短路求值
      int a = 0, b = 10;
      if (a && b++) { /* 不执行 */ }
      printf("b = %d\n", b);  // 10, 因为 a=0 短路, b++ 没执行

      // 逗号表达式
      int x = (1, 2, 3);
      printf("x = %d\n", x);  // 3, 逗号表达式取最后一个值

      // 三目运算符
      int max = a > b ? a : b;

      // sizeof 不会计算表达式
      int y = 10;
      printf("%zu\n", sizeof(y = y + 1));  // 4
      printf("y = %d\n", y);               // 10, y 没变

      return 0;
  }

## 常见错误 / 盲点

  1. = 和 == 混淆:
     if (x = 5)    // 永远为真, 且 x 被改成5
     if (x == 5)   // 正确
  2. 优先级错误:
     *p.a    // 等价于 *(p.a), 应该是 (*p).a 或 p->a
     x << 2 + 1  // 等价于 x << (2+1), 移位优先级低于加减
  3. 浮点数比较陷阱:
     不要用 == 比较浮点数

## 面试常问

  Q: 说说短路求值?
  A: 逻辑表达式从左到右计算, 一旦能确定结果就停止。
     常用于: 先判指针非空再访问成员。

  Q: i++ 和 ++i 的区别?
  A: i++ 先返回 i 再自增, ++i 先自增再返回。
     for 循环中两者效果一样。

## 练习

  1. 写出以下代码的输出:
     int a = 1, b = 2, c = 3;
     int r = a++ && b++ && c++;
     printf("%d %d %d %d", a, b, c, r);
  2. 不用临时变量交换两个数 (三种方法)
  3. 输入三个整数, 用三目运算符找出最大值
  4. 写一个宏判断一个整数是否为2的幂

## 复习记录
  - [ ] R1 (次日)
  - [ ] R3 (3天后)
  - [ ] R7 (7天后)
  - [ ] R14 (14天后)
  - [ ] R30 (30天后)
