# Day05 - 数组

## 概念

  - 连续内存区域, 存储多个相同类型元素
  - 通过下标 (index) 访问: arr[0], arr[1]
  - 聚合数据类型 (标量 vs 聚合: 数组/结构体)

### 数组 vs 链表

  数组: 连续内存, 随机访问 O(1), 固定大小
  链表: 离散内存, 顺序访问 O(n), 动态大小

## 数组寻址

  address(arr[i]) = base + i * sizeof(element)
  下标从0开始简化偏移量计算

## 数组声明与初始化

  int arr[5] = {1,2,3,4,5};     // 完整初始化
  int arr[] = {1,2,3,4,5};      // 自动推断长度=5
  int arr[5] = {1,2};           // 部分初始化, 剩余=0
  int arr[5] = {0};             // 全0
  int arr[5] = {0};             // 推荐清零方式

  长度必须是编译时常量, 不能为0

## 动态计算数组长度

  #define ARR_LEN(arr) (sizeof(arr) / sizeof(arr[0]))

## 数组与栈

  局部数组在栈上, 栈空间小, 不要开太大
  大数组用动态内存 (malloc)

## 二维数组

  二维数组: 装一维数组的数组
  内存连续: matrix[i][j] = *(*(matrix + i) + j)

  int arr[3][4] = {
      {1,2,3,4},
      {5,6,7,8},
      {9,10,11,12}
  };

## 数组名与指针

  int arr[5];
  arr        // 数组名, 首元素地址
  &arr[0]    // 首元素地址 (等价于 arr)
  &arr       // 整个数组的地址 (类型: int (*)[5])
  arr + 1    // 跳过 1 个 int (4字节)
  &arr + 1   // 跳过整个数组 (5 * 4 = 20字节)

## const 关键字

  const int a = 10;   // 不能通过 a 修改值
  scanf("%d", &a);    // 可通过地址修改 (不推荐)

## 随机数

  #include <stdlib.h>
  #include <time.h>
  srand((unsigned)time(NULL));  // 设置种子 (一次即可)
  int r = rand();               // [0, RAND_MAX]
  rand() % 100                  // [0, 99]
  rand() % 100 + 1              // [1, 100]

## 代码示例

  #include <stdio.h>
  #define ARR_LEN(arr) (sizeof(arr) / sizeof(arr[0]))

  int main(void) {
      int arr[] = {3, 7, 1, 9, 4};
      int len = ARR_LEN(arr);

      // 遍历
      for (int i = 0; i < len; i++) {
          printf("arr[%d] = %d\n", i, arr[i]);
      }

      // 数组名 + 1
      printf("arr    = %p\n", arr);
      printf("arr+1  = %p\n", arr + 1);
      printf("&arr   = %p\n", &arr);
      printf("&arr+1 = %p\n", &arr + 1);

      return 0;
  }

## 常见错误 / 盲点

  1. 数组越界:
     int arr[5]; arr[5] = 10;  // 越界, 未定义行为
  2. sizeof(arr) 退化为指针:
     函数传参后, arr 退化为指针, sizeof 不对
  3. 数组名不能被赋值:
     int arr[5]; arr = other;  // 编译错误

## 面试常问

  Q: 数组和指针的区别?
  A: 数组名是首元素地址的常量, 不能修改。
     sizeof(arr) 返回整个数组大小;
     sizeof(ptr) 返回指针大小。

  Q: arr 和 &arr 的区别?
  A: arr 是首元素地址, arr+1 加 sizeof(元素)
     &arr 是整个数组的地址, &arr+1 加整个数组大小

## 练习

  1. 输入10个整数, 求最大值/最小值/平均值
  2. 数组逆序 (不使用额外数组)
  3. 生成20个随机数 [1,100], 统计各个分数段
  4. 二维数组: 输入3*4矩阵, 转置后输出

## 自测

  1. 数组在内存中是如何存储的？下标从0开始的原因是什么？
  2. 如何动态获取数组长度？函数传参后 sizeof 为什么失效？
  3. arr、&arr[0]、&arr 三者的区别是什么？arr+1 和 &arr+1 分别跳过多少字节？
  4. 数组名能作为左值被赋值吗？为什么？
  5. 数组越界会有什么后果？
  6. 二维数组在内存中是如何排列的？如何寻址 matrix[i][j]？
  7. 声明 int arr[5] = {0} 的作用是什么？
  8. 什么是随机数种子？srand 应该调用几次？
  9. 数组和链表的主要区别有哪些？
  10. 局部数组为什么不能开太大？如果要大数组应该怎么办？

## 复习记录
  - [ ] R1 (次日)
  - [ ] R3 (3天后)
  - [ ] R7 (7天后)
  - [ ] R14 (14天后)
  - [ ] R30 (30天后)
