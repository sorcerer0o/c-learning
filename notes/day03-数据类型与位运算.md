# Day03 - 数据类型与位运算

## sizeof 运算符

  sizeof(int)       // 类型
  sizeof(num)       // 变量
  sizeof(1+2)       // 表达式 (不会被计算)
  返回值类型: size_t (无符号整数别名, %zu 打印)

## 基本数据类型

### 整数类型 (64位Linux常见取值)

  short              2字节  -32768 ~ 32767
  unsigned short     2字节  0 ~ 65535
  int                4字节  -21亿 ~ 21亿
  unsigned int       4字节  0 ~ 43亿
  long               8字节  -9.22e18 ~ 9.22e18
  long long          8字节  -9.22e18 ~ 9.22e18

  C标准: short <= int <= long <= long long
  字面值: long 加 L, long long 加 LL, unsigned 加 U

### 整数字面值进制

  int a = 255;      // 十进制
  int b = 0377;     // 八进制 (0开头)
  int c = 0xFF;     // 十六进制 (0x开头)

### 浮点数类型 (IEEE754)

  float    4字节  约6位有效数字  ±3.4E38
  double   8字节  约15位有效数字 ±1.8E308

  浮点数字面量默认 double, float 需加 f: 3.14f
  0.1 转二进制是无限循环, 所以浮点数不精确
  ️ 不要直接用 == 比较浮点数

### 字符类型 char

  占用 1 字节, 存储的是字符的编码值 (整数)
  ASCII: 'A'=65, 'a'=97, '0'=48, 空格=32, '\0'=0
  char 可作为整数参与运算: 'a' + 1 = 'b'

### ctype.h 常用函数

  isalpha(c)  是否为字母
  isdigit(c)  是否为数字
  islower(c)  是否为小写
  isupper(c)  是否为大写
  isalnum(c)  是否为字母或数字
  tolower(c)  转小写
  toupper(c)  转大写

### getchar / putchar

  int putchar(int c);   // 输出一个字符
  int getchar(void);    // 读取一个字符, 返回int (可能返回EOF)

  循环读取一行:
    int ch;  // 用 int 存返回值
    while ((ch = getchar()) != '\n') { ... }

  清空 stdin 缓冲区:
    while (getchar() != '\n') ;  // 空语句

## 原码 / 反码 / 补码

  正数: 原码 = 反码 = 补码
  负数:
    原码: 符号位1 + 绝对值二进制
    反码: 符号位不变, 其余取反
    补码: 反码 + 1

  计算机存储的是补码

## 位运算符

  &  按位与    同为1结果为1
  |  按位或    有1结果为1
  ^  按位异或  同为0, 不同为1
  ~  按位取反  0变1, 1变0
  << 左移      高位丢弃, 低位补0 (相当于乘以2)
  >> 右移      低位丢弃, 高位补符号位 (相当于除以2)

## 位运算经典应用

### 1. 判断奇偶

  if (num & 1)     // 奇数 (最低位为1)
  if (!(num & 1))  // 偶数

### 2. 交换两个数 (不用临时变量)

  a = a ^ b;
  b = a ^ b;
  a = a ^ b;
  // 注意: 同一元素交换会变0

### 3. 找出数组中出现一次的元素

  int res = 0;
  for (int i = 0; i < len; i++) res ^= arr[i];
  return res;

### 4. 找出数组中两个出现一次的元素

  int res = 0;
  for (int i = 0; i < len; i++) res ^= arr[i];
  int lsb = res & (-res);   // 找最低位的1
  int a = 0, b = 0;
  for (int i = 0; i < len; i++) {
      if (arr[i] & lsb) a ^= arr[i];
      else b ^= arr[i];
  }

### 5. LSB (最低有效位)

  int lsb = num & (-num);     // 取最低位的1
  num & (num - 1)             // 消去最低位的1

### 6. 乘以2的幂

  int x = 5;
  x << 2   // 相当于 5 * 4 = 20

### 7. 设置/清除/翻转指定位

  x |= (1 << n);       // 设置第 n 位为1
  x &= ~(1 << n);      // 清除第 n 位
  x ^= (1 << n);       // 翻转第 n 位
  (x >> n) & 1         // 获取第 n 位

## 数据类型转换

### 隐式类型转换

  整数提升: char/short 参与运算时先提升为 int
  常用算术转换: 结果类型取精度/范围最大的

    int + double -> double
    float + int -> float

  ⚠️ 无符号陷阱:
    int a = -10;
    unsigned b = 100;
    b > a  ->  false   // -10 转无符号变成极大正数

### 显式类型转换

  (目标类型)表达式
  (float)a / b   // 整数除法保留小数

## 代码示例

  #include <stdio.h>
  #include <ctype.h>

  int main(void) {
      // sizeof
      printf("int: %zu\n", sizeof(int));

      // 位运算: 判断奇偶
      int n = 7;
      if (n & 1) printf("%d 是奇数\n", n);

      // 位运算: 乘以4
      printf("5 << 2 = %d\n", 5 << 2);

      // 字符处理
      char c = 'A';
      printf("tolower('%c') = '%c'\n", c, tolower(c));

      return 0;
  }

## 常见错误 / 盲点

  1. 浮点数用 == 比较:
     double a = 0.1 + 0.2;  // 可能不是精确的 0.3
     if (a == 0.3)           // 危险! 应该用差值比较
     if (fabs(a - 0.3) < 1e-6)
  2. 无符号和有符号混用:
     循环中使用 unsigned 做条件可能死循环
  3. 位运算优先级:
     (x & 1) == 0    // 注意加括号, 因为 == 优先级高于 &

## 面试常问

  Q: 讲一下补码?
  A: 计算机用补码表示有符号整数。
     正数补码等于原码, 负数补码 = 反码+1。
     好处: 统一加减法, 0 唯一表示。

  Q: 如何在不使用第三个变量的情况下交换两个数?
  A: 用异或: a ^= b; b ^= a; a ^= b;
     原理: x ^ x = 0, x ^ 0 = x

## 练习

  1. 输入一个整数, 用位运算判断它是2的幂
     (提示: n > 0 && (n & (n-1)) == 0)
  2. 输入一个无符号整数, 输出它的二进制表示
  3. 用位运算实现: 对一个整数的第3位进行设置/清除/翻转
  4. 输入一个字符, 如果是小写转大写, 大写转小写 (不用 ctype.h)

## 复习记录
  - [ ] R1 (次日)
  - [ ] R3 (3天后)
  - [ ] R7 (7天后)
  - [ ] R14 (14天后)
  - [ ] R30 (30天后)
