# Day10 - 结构体、枚举与动态内存

## 结构体 struct

### 定义

  struct Student {
      char name[20];
      int age;
      float score;
  };

### 使用

  struct Student stu1;
  struct Student stu2 = {"Alice", 20, 95.5};
  stu1.age = 20;           // . 访问
  p->age = 20;             // -> 访问 (等价于 (*p).age)

### typedef 简化

  typedef struct Student {
      char name[20];
      int age;
      float score;
  } Student;

  Student stu;             // 不用写 struct
  Student *p = &stu;

### 传参

  结构体传参默认值传递 (复制整个结构体)
  大结构体建议传指针:
    void print(Student *s);

## 数据对齐 (padding)

  提高 CPU 访问效率, 编译器在成员间插入填充字节
  成员地址对齐到自身大小的整数倍
  结构体总大小是最大成员对齐大小的整数倍

  struct Example {
      char c;     // 1 字节, 偏移 0
      // 3 字节 padding
      int i;      // 4 字节, 偏移 4
      short s;    // 2 字节, 偏移 8
      // 2 字节 padding
  };  // sizeof = 12

## 枚举 enum

  enum Color { RED, GREEN, BLUE };
  // RED=0, GREEN=1, BLUE=2

  enum Color { RED=1, GREEN=3, BLUE=5 };
  // C 中枚举本质是整型

## 联合体 union

  union Data {
      int i;
      float f;
      char str[4];
  };  // sizeof = max(成员大小)

  所有成员共享同一块内存
  常用于: 节省空间, 类型双关

## 存储期限

  自动存储期限  栈     局部变量, 函数执行期间
  静态存储期限  数据段  全局/static 变量, 程序运行期间
  动态存储期限  堆     malloc/free 手动管理

## malloc / free

  #include <stdlib.h>

  申请: void *malloc(size_t size);
  释放: void free(void *ptr);

  int *p = (int *)malloc(10 * sizeof(int));
  if (p == NULL) { /* 处理失败 */ }
  free(p);
  p = NULL;  // 避免悬空

  ️ malloc 不初始化内存, 内容随机

### calloc / realloc

  calloc(n, size);     // 分配 n*size 并置零
  realloc(p, newSize); // 调整大小, 可能移动数据

## 通用指针 void*

  void *p = malloc(100);
  int *arr = (int *)p;  // 使用时需转换

## 函数指针

  int (*func_ptr)(int, int);  // 声明
  func_ptr = add;              // 赋值
  int r = func_ptr(3, 4);      // 调用

  用途: 回调函数, 实现策略模式

## 代码示例

  #include <stdio.h>
  #include <stdlib.h>
  #include <string.h>

  typedef struct {
      char name[20];
      int age;
  } Person;

  Person* create_person(const char *name, int age) {
      Person *p = (Person *)malloc(sizeof(Person));
      if (p == NULL) return NULL;
      strcpy(p->name, name);
      p->age = age;
      return p;
  }

  void print_person(Person *p) {
      if (p) printf("%s: %d\n", p->name, p->age);
  }

  int main(void) {
      Person *p = create_person("Alice", 20);
      print_person(p);
      free(p);
      return 0;
  }

## 常见错误 / 盲点

  1. 忘记 free: 内存泄漏
  2. free 后继续使用: 悬空指针
  3. malloc 返回值未判空
  4. 结构体太大传值: 栈溢出

## 面试常问

  Q: malloc 和 calloc 的区别?
  A: calloc 会初始化内存为 0, malloc 不会。
     calloc 接受两个参数: 个数和大小。

  Q: struct 对齐是什么?
  A: 成员地址对齐到自身大小的整数倍,
     结构体总大小是最大成员对齐的整数倍。
     目的是提高 CPU 访问效率。

## 练习

  1. 定义学生结构体, 实现输入/输出函数
  2. 用动态内存创建学生数组, 按成绩排序
  3. 实现一个简单的链表节点 (struct Node)
  4. 写一个函数指针: 实现一个计算器 (加减乘除)

## 自测

  1. 结构体定义中 typedef 的作用是什么？不用 typedef 怎么声明变量？
  2. 结构体传参默认是值传递还是地址传递？大结构体应该怎么传？
  3. 什么是数据对齐？为什么需要 padding？举例说明。
  4. 枚举在 C 中本质是什么类型？枚举值默认从几开始？
  5. union 和 struct 有什么区别？union 的大小怎么计算？
  6. malloc 和 calloc 有什么区别？malloc 分配的内存初始值是什么？
  7. malloc 后忘记 free 会怎样？free 后还需要做什么？
  8. 什么是悬空指针？如何避免？
  9. 函数指针如何声明？有什么用途？
  10. void* 有什么限制？为什么 malloc 返回 void*？

## 复习记录
  - [ ] R1 (次日)
  - [ ] R3 (3天后)
  - [ ] R7 (7天后)
  - [ ] R14 (14天后)
  - [ ] R30 (30天后)
