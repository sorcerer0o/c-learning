# Day17 - 文件流深入

## 文件流操作流程

  1. fopen 打开文件
  2. 检查是否成功 (fp == NULL)
  3. 使用文件 (读/写)
  4. fclose 关闭

## fopen 模式

  "r"   只读 (文件必须存在)
  "w"   只写 (清空重建)
  "a"   追加 (文件末尾写)
  "rb"  二进制读
  "wb"  二进制写
  "r+"  读写 (文件必须存在)
  "w+"  读写 (清空重建)

## 字符串读写

  fgets(buf, count, fp);
    - 最多读 count-1 个字符, 末尾加 \0
    - 读到换行或 EOF 停止 (保留换行符)
    - 成功返回 buf, 结束返回 NULL

  fputs(str, fp);
    - 写入字符串, 不自动加换行

## 格式化读写

  fprintf(fp, "format", args);
  fscanf(fp, "format", &args);

  示例:
    fprintf(fp, "%s %d\n", name, age);
    fscanf(fp, "%s %d", name, &age);

## 二进制读写

  fread(buf, size, count, fp);
    - 读 count 个 size 字节的数据到 buf
    - 返回实际读取的项数

  fwrite(buf, size, count, fp);
    - 从 buf 写 count 个 size 字节到文件

## 文件定位

  fseek(fp, offset, whence);
    SEEK_SET  文件开头
    SEEK_CUR  当前位置
    SEEK_END  文件末尾

  ftell(fp);  返回当前偏移量

  示例:
    fseek(fp, 0L, SEEK_SET);  // 回到开头
    fseek(fp, 0L, SEEK_END);  // 跳到末尾
    long size = ftell(fp);    // 获取文件大小

## 错误处理

  #include <errno.h>
  #include <string.h>

  errno      错误编号
  strerror(errno)   错误描述字符串
  perror(msg)       输出: msg + 错误描述

## 代码示例

  #include <stdio.h>
  #include <errno.h>
  #include <string.h>

  int main(void) {
      FILE *fp = fopen("test.txt", "w");
      if (fp == NULL) {
          perror("打开文件失败");
          return 1;
      }

      // 写入
      fprintf(fp, "hello %d\n", 123);
      fputs("world\n", fp);

      fclose(fp);

      // 读取
      fp = fopen("test.txt", "r");
      char buf[100];
      while (fgets(buf, sizeof(buf), fp)) {
          printf("%s", buf);
      }
      fclose(fp);

      return 0;
  }

## 常见错误

  1. 打开文件失败不检查:
     fp = fopen(...);  // 忘记判空
  2. 忘记 fclose: 文件句柄泄漏
  3. fgets 忘记去掉换行符:
     buf[strcspn(buf, "\n")] = '\0';
  4. 以 w 模式打开已存在文件: 内容被清空

## 练习

  1. 实现文件拷贝 (fread/fwrite)
  2. 读取文本文件, 统计行数/单词数/字符数
  3. 将结构体数组写入二进制文件, 再读回
  4. 实现断点续传的文件定位

## 复习记录
  - [ ] R1 (次日)
  - [ ] R3 (3天后)
  - [ ] R7 (7天后)
  - [ ] R14 (14天后)
  - [ ] R30 (30天后)
