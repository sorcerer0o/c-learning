# Day16 - 高级排序与文件流基础

## 分治与递归

  分治: 解决问题的策略 (分而治之)
  递归: 具体技术手段 (函数调用自身)

## 快速排序

  核心: 选枢纽值, 左边都小于它, 右边都大于它
  枢纽值位置正确后, 递归处理左右两边
  O(n log n), 不稳定

  void qsort(int arr[], int left, int right) {
      if (left >= right) return;
      int pivot = partition(arr, left, right);
      qsort(arr, left, pivot - 1);
      qsort(arr, pivot + 1, right);
  }

  int partition(int arr[], int left, int right) {
      int pivot = arr[left];
      int i = left, j = right;
      while (i < j) {
          while (i < j && arr[j] >= pivot) j--;
          arr[i] = arr[j];
          while (i < j && arr[i] <= pivot) i++;
          arr[j] = arr[i];
      }
      arr[i] = pivot;
      return i;
  }

## 堆排序

  大顶堆: 每个节点 >= 左右孩子
  小顶堆: 每个节点 <= 左右孩子

  建堆: 从最后一个非叶子节点开始, 向下调整
  排序: 堆顶与末尾交换, 调整堆

  左孩子: 2*idx + 1
  右孩子: 2*idx + 2

## 文件流基础

  流: 编程模型, 通过固定步骤读写数据

  步骤:
    1. 打开文件流
    2. 判断是否成功
    3. 使用 (读/写)
    4. 关闭

  路径:
    绝对路径: win盘符开头, linux / 开头
    相对路径: 相对工作目录

## 常用函数

  fopen/fclose    打开/关闭
  fgetc/fputc     字符读写
  fgets/fputs     字符串读写
  fprintf/fscanf  格式化读写
  fread/fwrite    二进制读写
  fseek/ftell     文件指针定位

## 代码示例

  #include <stdio.h>

  // 复制文件
  void copy_file(const char *src, const char *dst) {
      FILE *in = fopen(src, "rb");
      FILE *out = fopen(dst, "wb");
      if (!in || !out) return;

      int ch;
      while ((ch = fgetc(in)) != EOF)
          fputc(ch, out);

      fclose(in);
      fclose(out);
  }

  int main(void) {
      copy_file("input.txt", "output.txt");
      return 0;
  }

## 练习

  1. 实现快排
  2. 实现堆排序
  3. 实现文件复制 (字符/二进制两种模式)
  4. 读取文件中的整数, 排序后写入新文件

## 复习记录
  - [ ] R1 (次日)
  - [ ] R3 (3天后)
  - [ ] R7 (7天后)
  - [ ] R14 (14天后)
  - [ ] R30 (30天后)
