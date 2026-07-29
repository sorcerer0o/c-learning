# Day12 - 头文件、函数指针、链表

## 头文件

  #ifndef XXX_H
  #define XXX_H
  // 内容: 结构体, 类型别名, 宏定义, 函数声明
  #endif

  作用: 防止重复包含
  每个 .h 文件都要加头文件保护

## 函数指针

  声明: 返回值类型 (*指针名)(参数列表);
  赋值: func_ptr = func_name;
  调用: func_ptr(args);

  int (*cmp)(const void *, const void *);

  用途:
    1. 回调函数 (qsort, 线程等)
    2. 策略模式 (不同算法可切换)
    3. 实现多态效果

## qsort

  #include <stdlib.h>
  void qsort(void *base, size_t n, size_t size,
             int (*cmp)(const void *, const void *));

  // 示例: 排序 int 数组
  int cmp_int(const void *a, const void *b) {
      return *(int *)a - *(int *)b;
  }
  qsort(arr, len, sizeof(int), cmp_int);

## 链表

### 节点定义

  typedef struct node {
      int data;
      struct node *next;
  } Node;

### 基本操作

  头插法:
    Node *head = NULL;
    Node *new_node = malloc(sizeof(Node));
    new_node->data = val;
    new_node->next = head;
    head = new_node;

  尾插法:
    遍历到末尾, 将新节点接上去

  删除:
    找到前驱节点, 修改 next 指针

  遍历:
    for (Node *p = head; p; p = p->next)

### 套壳写法

  typedef struct {
      Node *head;
      int size;
  } LinkedList;

  好处: 操作时只需传递 LinkedList*, 不用二级指针

## 栈和队列

  栈: 先进后出 (LIFO), 可用数组/链表实现
  队列: 先进先出 (FIFO), 可用数组/链表实现

## 代码示例

  #include <stdio.h>
  #include <stdlib.h>

  typedef struct node {
      int data;
      struct node *next;
  } Node;

  // 头插
  void push_front(Node **head, int val) {
      Node *new_node = malloc(sizeof(Node));
      new_node->data = val;
      new_node->next = *head;
      *head = new_node;
  }

  // 遍历
  void print_list(Node *head) {
      for (Node *p = head; p; p = p->next)
          printf("%d -> ", p->data);
      printf("NULL\n");
  }

  int main(void) {
      Node *head = NULL;
      push_front(&head, 3);
      push_front(&head, 2);
      push_front(&head, 1);
      print_list(head);  // 1 -> 2 -> 3 -> NULL
      return 0;
  }

## 常见错误

  1. 头文件重复包含: 忘记加头文件保护
  2. 链表操作忘记更新头指针: 头插后 head 没变
  3. 内存泄漏: malloc 的节点 free 了吗
  4. 访问 NULL: 遍历链表时 p 可能是 NULL

## 面试常问

  Q: 链表反转?
  A: 三个指针: prev, curr, next, 逐个翻转指向

  Q: 如何判断链表有环?
  A: 快慢指针, 快指针每次走两步, 慢指针走一步

## 练习

  1. 实现链表: 头插, 尾插, 删除, 查找, 打印
  2. 实现链表反转 (迭代 + 递归两种方式)
  3. 用链表实现栈 (push/pop)
  4. 用 qsort 排序学生成绩数组

## 复习记录
  - [ ] R1 (次日)
  - [ ] R3 (3天后)
  - [ ] R7 (7天后)
  - [ ] R14 (14天后)
  - [ ] R30 (30天后)
