# Day14 - 哈希表

## 概念

  哈希表 = 数组 + 链表 (链地址法解决冲突)
  通过 key 快速定位到存储位置, 平均 O(1)

## 运行过程

  1. 对 key 计算哈希值 (转为整数)
  2. 取余: idx = hash % table_size
  3. 到 table[idx] 位置查找/插入

## 数据结构

  typedef struct KeyValueNode {
      char *key;
      char *value;
      struct KeyValueNode *next;
  } KeyValueNode;

  typedef struct {
      KeyValueNode **table;  // 指针数组 (每个元素是链表头)
      int size;
  } HashMap;

  为什么用指针数组?
    - 用结构体数组会初始化所有节点 (浪费)
    - 指针数组初始为 NULL, 有数据才申请节点

## 哈希函数

  uint32_t hash(const void *key, int len, uint32_t seed);
  - key: 数据地址 (字符串直接传, 整型传 &key)
  - len: 长度 (字符串 strlen, 其他 sizeof)
  - seed: 种子, 影响哈希结果

## 基本操作

  插入 (put):
    1. 计算 idx
    2. 遍历链表, key 存在则更新 value
    3. 不存在则头插新节点

  查找 (get):
    1. 计算 idx
    2. 遍历链表, 找到 key 返回 value

  删除 (remove):
    1. 计算 idx
    2. 链表删除操作

## 哈希表头文件结构

  #ifndef HASH_MAP_H
  #define HASH_MAP_H
  // 宏定义
  // 结构体定义
  // 函数声明
  #endif

## 代码示例

  #include <stdio.h>
  #include <stdlib.h>
  #include <string.h>

  #define TABLE_SIZE 100

  typedef struct node {
      char *key;
      char *value;
      struct node *next;
  } Node;

  typedef struct {
      Node **table;
      int size;
  } HashMap;

  // 简单的哈希函数
  unsigned hash(const char *key) {
      unsigned h = 0;
      while (*key) h = h * 31 + *key++;
      return h % TABLE_SIZE;
  }

  void put(HashMap *map, const char *key, const char *value) {
      int idx = hash(key);
      Node *cur = map->table[idx];
      while (cur) {
          if (strcmp(cur->key, key) == 0) {
              free(cur->value);
              cur->value = strdup(value);
              return;
          }
          cur = cur->next;
      }
      Node *new_node = malloc(sizeof(Node));
      new_node->key = strdup(key);
      new_node->value = strdup(value);
      new_node->next = map->table[idx];
      map->table[idx] = new_node;
  }

  char* get(HashMap *map, const char *key) {
      int idx = hash(key);
      Node *cur = map->table[idx];
      while (cur) {
          if (strcmp(cur->key, key) == 0)
              return cur->value;
          cur = cur->next;
      }
      return NULL;
  }

  int main(void) {
      HashMap map = {0};
      // 测试略
      return 0;
  }

## 面试常问

  Q: 哈希冲突如何解决?
  A: 链地址法 (数组+链表), 开放地址法, 再哈希法等。

  Q: 哈希表扩容?
  A: 当负载因子超过阈值, 扩大数组, 重新哈希所有元素。

## 练习

  1. 实现哈希表的 put / get / remove
  2. 实现哈希表的扩容 (rehash)
  3. 用哈希表统计一段文字中单词出现的次数

## 复习记录
  - [ ] R1 (次日)
  - [ ] R3 (3天后)
  - [ ] R7 (7天后)
  - [ ] R14 (14天后)
  - [ ] R30 (30天后)
