C和C++的关键字

[TOC]

------

### 4 sizeof关键字

#### 4.1 32位和64位编译器

```c
/* 32位 */
sizeof(long)=4;
/* 64位 */
sizeof(long)=8;
```

#### 4.2 指针和数组

```c
void print(char str[]) {}
int main()
{
    //str是数组名，所以sizeof(str)=5，代表整个数组所占内存空间
    char str[5] = {0};
    //str1是数组名，"hello"是字符串，末尾还有一个'\0'，故sizeof(str1)=6
    char str1[] = "hello";
    //str2是指针，故sizeof(str2)=4，代表指针变量所占内存空间
    char *str2 = "hello";
    //str3是指针数组名，数组元素都是指针，故sizeof(str3)=4*4=16
    char *str3[4];
    //str4是数组指针，指向一个字符数组，故sizeof(str4)=4
    char *(str4)[4];
    //当数组名作为参数传入时退化为指针，故sizeof(str)=4
    print(str);
    return 0;
}

```

#### 4.3 sizeof与strlen

- sizeof是关键字，strlen是函数。sizeof后若为类型必须加括号，若为变量名可不加括号。
- sizeof返回数据类型的长度。strlen函数返回字符串的长度，不包含字符'\0'。
- sizeof的参数可以是类型，而strlen的参数只能是字符类型指针。

注意：'\0' 和 NULL的值都是0，但空格 ' ' 的值不为0，而是32。

```c
int main()
{
    //str数组中的元素都为0，故strlen(str)=0
    char str[5] = {0};
    //字符'0'的值不为0，故strlen(str1)=5
    char str1[5] = {'0'};
    //"hello"字符串的长度为5，不包括'\0'，故strlen(str2)=5
    char str2 = "hello";
}
```

#### 4.4 结构体

结构体有时候需要**字节对齐**。一般而言，struct的sizeof是所有成员字节对齐后长度相加，而union的sizeof是取最大的成员长度。

可以通过下面的方法来改变默认的对界条件：

- 使用伪指令#pragma pack(n)，C编译器将按照n个字节对齐。

- 使用伪指令#pragma pack()，取消自定义字节对齐方式。

字节对齐遵循两条原则：

结构体变量中成员的偏移量必须是成员大小的整数倍（0被认为是任何数的整数倍）。

结构体大小必须是所有成员大小的整数倍。

```c
#pragma pack(2)
typedef struct{
    char c;
    int i;
}str;
//sizeof(str)的值为6;
//若取消自定义对齐，则sizeof(str)值为8;
struct{
    char a[10];
    int b;
    short c[3];
}
//数组只是多个元素聚在一起，本质没有改变，故需要对齐的长度还是4个字节
//故结构体长度为 4+4+2+4+4+2=24（考虑对齐）
```

#### 4.5 位域

```c
typedef struct
{
    char c1:1;
    char c2:2;
    char c3:3;
}str;
//1+2+3=6位，故只占1个字节，sizeof(str)=1
typedef struct
{
    char c1:5;
    char c2:5;
    char c3:4;
}str1;
//5+5+4=14位，但考虑字节补齐，sizeof(str1)=3
```

#### 4.6 C++的类

```c++
class Test
{
public:
    virtual void fun(){}
private:
    char c;
    int i;
    static char sc;
};
//考虑字节对齐，4(虚表指针)+4+4=12，static变量不计入
```

