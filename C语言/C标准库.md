## C标准库

[TOC]

------

### 1 <assert.h>

 **assert** ：一个宏，它可用于验证程序做出的假设，并在假设为假时输出诊断消息。

**ASSERT**：是一个调试程序时经常使用的宏，在程序运行时它计算括号内的表达式，如果表达式为 FALSE (0), 程序将报告错误，并终止执行。 **只有在 Debug 版本中才有效，如果编译为 Release 版本则被忽略**。

### 2 <ctype.h>

#### 1.1 库函数

```c
/* 该函数把大写字母转换为小写字母 */
int tolower(int c);
/* 把小写字母转换为大写字母 */
int toupper(int c);
```

### 3 <errno.h>

#### 1.1 简介

定义了整数变量**errno**，它是通过系统调用设置的，可以被一个程序读取和修改。

### 4 <limits.h>

决定了各种变量类型的各种属性。定义在该头文件中的宏限制了各种变量类型（比如 char、int 和 long）的值。

### 5 <math.h>

```c
/* 返回弧度角 x 的余弦 */
double cos(double x);
/* 返回 x 的 y 次幂 */
double pow(double x, double y);
/* 返回 x 的绝对值 */
double fabs(double x);
```



### 1 <string.h>

**string .h** 头文件定义了一个变量类型、一个宏和各种操作字符数组的函数。

#### 1.1 库变量

**size_t**：无符号整数类型，它是**sizeof**关键字的结果。

#### 1.2 库宏

**NULL**：此宏为一个空指针常量的值。

#### 1.3 库函数

```c
#include <string.h>

/* 在参数str所指向的字符串的前n个字节中搜索第一次出现字符c的位置 */
void *memchr(const void *str, int c, size_t n);
/* 把str1和str2的前n个字节进行比较 */
int memcmp(const void *str1, const void *str2, size_t n);
/* 从 src 复制 n 个字符到 dest */
void *memcpy(void *dest, const void *src, size_t n);
/* 复制字符c到参数str所指向的字符串的前n个字符 */
void *memset(void *str, int c, size_t n);

/* 把 src 所指向的字符串追加到 dest 所指向的字符串的结尾 */
char *strcat(char *dest, const char *src);
char *strncat(char *dest, const char *src, size_t n);

/* 把 str1 所指向的字符串和 str2 所指向的字符串进行比较 */
int strcmp(const char *str1, const char *str2);
int strncmp(const char *str1, const char *str2, size_t n);

/* 把 src 所指向的字符串复制到 dest */
char *strcpy(char *dest, const char *src);
char *strncpy(char *dest, const char *src, size_t n);

/* 从内部数组中搜索错误号 errnum，并返回一个指向错误消息字符串的指针 */
char *strerror(int errnum);
/* 计算字符串 str 的长度，直到空结束字符，但不包括空结束字符 */
size_t strlen(const char *str);
/* 在字符串 haystack 中查找第一次出现字符串 needle（不包含空结束字符）的位置 */
char *strstr(const char *haystack, const char *needle);
```

