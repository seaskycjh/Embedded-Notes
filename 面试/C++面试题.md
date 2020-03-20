C++笔试题

[TOC]

------

4 声明对象指针的多种情况

<font color=red>易错的例子：</font>

- 声明一个对象指针，不分配内存，将不会调用构造函数。
- 声明一个基类指针，分配一个派生类对象空间，构造与直接声明派生类一样，析构的话只调用基类的析构函数，因为基类的析构函数未声明为虚函数。**所以基类的析构函数必须定义为虚函数，否则子类对象析构时，将无法调用子类的析构函数，造成内存泄露。**
- 声明一个派生类指针，分配一个基类对象空间，会报错，因为基类指针可以指向派生类对象而派生类指针不可以指向基类对象。

```c++
class CBase{
public:
    CBase(){
        cout << "CBase()" << endl;
    }
    ~CBase(){
        cout << "~CBase()" << endl;
    }
};
class CDerived : public CBase{
public:
    CDerived(){
        cout << "CDerived()" << endl;
    }
    ~CDerived(){
        cout << "~CDerived()" << endl;
    }
};
CBase *CBase1 = NULL;  //无输出
CBase *pCBase = new CDerived();  //输出结果：CBase() CDerived() ~CBase()
CDerived *pCDerived = new CBase();  //错误
```

**写出程序运行结果**：

```c++
#include<iostream>
using namespace std;
 
class Base
{
    int x;
public:
    Base(int b): x(b) {}
    virtual void display()
    {
        cout << x;
    };
};
class Derived: public Base
{
    int y;
public:
    Derived(int d): Base(d), y(d) {}
    void display()
    {
        cout << y;
    }
};
 
int main()
{
    Base b(1);  //b是Base类的一个对象
    Derived d(2);  //d是Derivative类的对象
    Base *p = &d;  //基类指针指向Derived类的对象
    b.display();
    d.display();
    p->display();
    return 0;
}
//输出结果为: 222
```

