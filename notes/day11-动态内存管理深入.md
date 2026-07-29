# Day11 - 动态内存管理深入

## free

  void free(void *ptr);

  - 释放 malloc/calloc/realloc 申请的堆内存
  - free 释放的是指针指向的空间, 不是指针本身
  - 释放后应将指针置为 NULL, 避免悬空指针
  - 重复 free 同一块内存会导致未定义行为

## calloc

  void *calloc(size_t num, size_t size);

  - 申请 num 个大小为 size 的元素空间
  - 自动将内存初始化为 0
  - 返回 void *

  int *arr = (int *)calloc(10, sizeof(int));
  // 等价于:
  int *arr = (int *)malloc(10 * sizeof(int));
  memset(arr, 0, 10 * sizeof(int));

## realloc

  void *realloc(void *ptr, size_t new_size);

  - 调整已申请内存的大小
  - 可能移动到新地址, 原数据自动复制
  - 新扩展空间数据未初始化 (随机值)
  - ptr=NULL 等价 malloc, new_size=0 等价 free

  ️ 正确用法:
    void *new_arr = realloc(arr, new_size);
    if (new_arr != NULL) arr = new_arr;

  错误用法:
    arr = realloc(arr, new_size);  // 失败后 arr 丢失

## vector (动态数组)

  - 底层用堆内存 + realloc 实现自动扩容
  - 容量满时翻倍 (均摊 O(1))

  typedef struct {
      int *data;
      size_t size;
      size_t capacity;
  } Vector;

  操作: init / push_back / pop_back / get / set / destroy

## 内存泄漏

  原因: malloc 后忘记 free
  后果: 程序占用的内存持续增长, 最终耗尽

  ️ 每次 malloc 都要配对 free
  ️ 分配函数嵌套时注意释放

## 常见错误

  1. 内存泄漏: malloc 不 free
  2. 悬空指针: free 后继续使用
  3. 重复 free: 同一指针 free 两次
  4. 越界写: 分配 5 个 int, 写第 6 个

## 面试常问

  Q: malloc 和 calloc 的区别?
  A: calloc 会初始化为 0, malloc 不会。
     calloc 参数是 (个数, 大小)。

  Q: realloc 的原理?
  A: 如果当前块后面有足够空间, 直接扩展；
     否则申请新块, 复制原数据, 释放旧块。

## 练习

  1. 实现一个动态数组 (vector), 支持 push/pop/get/set
  2. 用 realloc 实现一个可增长的数据缓冲区

## 复习记录
  - [ ] R1 (次日)
  - [ ] R3 (3天后)
  - [ ] R7 (7天后)
  - [ ] R14 (14天后)
  - [ ] R30 (30天后)
