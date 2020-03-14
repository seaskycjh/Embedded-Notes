STL库

1 简介

STL（Standard Template Library），即标准模板库，是一个高效的C++程序库。包含了诸多常用的基本数据结构和基本算法。

1.1 STL组件

STL有六大组件，但主要包含容器、迭代器和算法三个部分。

- 容器（Containers）：用来管理某类对象的集合。每一种容器都有其优点和缺点，所以，为了应付程序中的不同需求，STL 准备了七种基本容器类型。
- 迭代器（Iterators）：用来在一个对象集合的元素上进行遍历动作。这个对象集合或许是个容器，或许是容器的一部分。每一种容器都提供了自己的迭代器，而这些迭代器了解该种容器的内部结构。
- 算法（Algorithms）：用来处理对象集合中的元素。

1.2 容器

容器用来管理某类对象。

向量（vector）、双端队列（deque）、列表（list）、集合（set）、多重集合（multiset）、映射（map）和多重映射（multimap）。

**序列式容器**：向量、双端队列、列表。

|      | vector                                   | list                                                         | deque                                                    |
| ---- | ---------------------------------------- | ------------------------------------------------------------ | -------------------------------------------------------- |
| 特点 | 快速的随机存取，快速的在最后插入删除元素 | 可以快速的在任意位置添加删除元素，只能快速的访问最开始和最后面的元素 | 在开始和最后添加删除元素一样快，并且提供了随机访问的方法 |
| 适用 | 需要高效的随机存取，不在乎插入删除的效率 | 需要大量的插入和删除操作，不关心随机存取                     | 需要随机存取，也需要高效的在两端进行插入删除操作         |

**关联式容器**：集合、多重集合、映射和多重映射。

1.3 算法

1.4 小结

- 如果需要高效的随机存取，不在乎插入和删除的效率，使用vector。
- 如果需要大量的插入和删除元素，不关心随机存取的效率，使用list。
- 如果需要随机存取，并且关心两端数据的插入和删除效率，使用deque。
- 求方便地根据key找到value，一对一的情况使用map，一对多的情况使用multimap。
- 查找一个元素是否存在于某集合中，唯一存在的情况使用set，不唯一存在的情况使用multiset。 

2 迭代器

迭代器（Iterator）是一种检查容器内元素并遍历元素的数据类型。迭代器是指针的泛化，它允许程序员用相同的方式处理不同的数据结构（容器）。类似于下标运算符，用于访问对象的元素。所有标准库容器都可以使用迭代器。

```c++
/* 初始化 */
vector<int> v(10, 1);
auto b = v.begin();  //返回指向第一个元素的迭代器
auto e = v.end();  //返回指向容器尾元素的下一位置的迭代器
/* 运算符 */
*b;  //返回迭代器b所指元素的引用
b->men;  //解引用b并获取该元素的名为men的成员，等价于(*b).mem;
++iter;  //令b指示容器中的下一个元素
a - b;  //两迭代器相减的结果为它们之间的距离
```

3 vector

3.1 简介

vector是一种序列式容器，事实上和数组差不多。一般来说数组不能动态拓展，因此在程序运行的时候不是浪费内存，就是造成越界。而vector相当于可分配拓展的数组（动态数组），它的随机访问快，在中间插入和删除慢，但在末端插入和删除快。

3.2 用法

```c++
#include <vector>
/* 初始化 */
vector<int> vec; //定义一个int类型的向量a
vector<int> vec(10); //定义一个int类型的向量a，并设置初始大小为10
vector<int> vec(10, 1); //定义一个int类型的向量a，并设置初始大小为10且初始值都为1
vector<int> b(a); //定义并用向量a初始化向量b
vector<int> b(a.begin(), a.begin()+3);  ////将a向量中从前3个元素作为向量b的初始值
int n[] = {1, 2, 3, 4, 5} ;
vector<int> a(n, n+5) ;  //将数组n的前5个元素作为向量a的初值，不包括n[5]
vec.assign(int nSize, const T& x);  //多个元素赋值
/* 基本操作-容量 */
vec.size();  //容器大小
vec.capacity();  //容器容量，即预分配的内存空间
vec.resize();  //更改容器大小
vec.empty();  //容器判空
vec.shrink_to_fit();  //减少容器大小到满足元素所占存储空间的大小
/* 基本操作-增加/删除 */
vec.push_back(const T& x);  //末尾添加元素
vec.insert(iterator it, const T& x);  //在指定位置it前面插入元素x,返回指向这个元素的迭代器
vec.pop_front();  //头部删除元素
vec.pop_back();  //末尾删除元素
vec.erase(iterator it);  //任意位置删除一个元素
vec.erase(iterator first, iterator last);  //删除[first,last]之间的元素
vec.clear();  //清空所有元素
/* 基本操作-访问元素 */
vec[1];  //下标访问，不会检查是否越界
vec.at(1);  //at方法访问，会检查是否越界，是则抛出out of range异常
vec.front();  //访问第一个元素
vec.back();  //访问最后一个元素
int* p = vec.data();  //返回一个指针
/* 迭代器 */
vec.begin();  //开始指针
vec.end();  //末尾指针
vec.cbegin();  //指向常量的开始指针，即不能通过这个指针来修改所指的内容，但指针可以移动
vec.cend();  //指向常量的末尾指针
/* 算法-遍历 */
vector<int>::iterator it;
for(it = vec.begin(); it != vec.end(); it++)
    cout << *it << endl;
//下标访问
for(int i = 0; i < vec.size(); i++) {
    cout << vec.at(i) << endl;
}
/* 算法-排序 */
sort(vec.begin(), vec.end()); //采用的是从小到大的排序
```

4 deque

4.1 简介

deque（双端队列）是一个动态数组，可以向两端发展，因此不论在尾部或头部安插元素都十分迅速。 在中间部分安插元素则比较费时，因为必须移动其它元素。

4.2 用法

```c++
#include <deque>
deque<int> deq;
//deque 与 vector 的用法基本一致，除了以下几点
/* deque有而vector没有 */
deq.push_front();  //头部添加元素
deq.pop_front();  //头部删除元素
/* vector有而deque没有 */
vec.data();
vec.capacity();
```

5 list

5.1 简介

List由双向链表（doubly linked list）实现而成，内存空间可以是不连续的。它的随机存取变得非常没有效率，因此它没有提供[]操作符的重载。但它可以很有效率的支持任意地方的插入和删除操作。

5.2 用法

```c++
#include <list>
//list 与 vector 的用法基本一致，除了以下几点
list不支持下标访问和at方法访问;
lst.unique();  //删除容器中相邻的重复元素
```

6 set

6.1 简介

set容器内的元素会被自动排序，set与map不同，set中的元素即是键值又是实值，set不允许两个元素有相同的键值。当对容器中的元素进行插入或者删除时，操作之前的所有迭代器在操作之后依然有效。

6.2 用法

```c++
#include <set>
/* 初始化 */
set<int> a;  //定义一个int类型的集合a
set<int> b(a);  //定义并用集合a初始化集合b
set<int> b(a.begin(), a.end());  //将集合a中的所有元素作为集合b的初始值
/* 基本操作-容量 */
st.size();  //容器大小
st.empty();  //容器判空
st.count(key);  //查找键key的元素个数
/* 基本操作-增加/删除 */
st.insert(const T& x);  //在容器中插入元素
st.insert(iterator it, const T& x);  //任意位置插入一个元素
st.pop_back(const T& elem);  //删除容器中值为elem的元素
st.erase(iterator it);  //删除it迭代器所指的元素
st.clear();  //清空所有元素
/* 基本操作-访问元素 */
st.find(key);  //查找键key是否存在,若存在，返回该键的元素的迭代器；若不存在，返回st.end()
```

6.3 总结

set 与序列式容器的用法有以下不同：

- set 只支持默认构造函数和拷贝构造函数，没有其它重载的构造函数。

- set 只能使用insert的两种重载函数插入，不支持push_back()和push_front()函数。
- set 能不通过迭代器，只通过元素值来删除该元素。
- set 容器不提供下标操作符。为了通过键从 set 中获取元素，可使用 find 运算。
- set 不支持STL里的reverse和sort算法。

7 map

7.1 简介

map由红黑树实现，其元素都是“键值/实值”所形成的一个对组（key/value pairs)。每个元素有一个键，是排序准则的基础。每一个键只能出现一次，不允许重复。

Map主要用于资料一对一映射(one-to-one)的情况，map内部自建一颗红黑树(平衡二叉树中的一种)，这颗树具有对数据自动排序的功能，所以在map内部所有的数据都是有序的。

7.2 用法

```c++
#include <map>
map<int, string> a; //定义一个int类型的映射a
//map与set用法基本一致
//map 可以像数组那样插入元素，而 set 不行
a[1] = "java";
a[2] = "python";
```

8 算法

#### 8.1 查找算法

**count**：利用等于操作符，把标志范围内的元素与输入值比较，返回相等元素个数。

**binary_search**：在有序序列中查找value，找到返回true。

**find**：利用底层元素的等于操作符，对指定范围内的元素与输入值进行比较。

**search**：给出两个范围，返回一个ForwardIterator，查找成功指向第一个范围内第一次出现子序列(第二个范围)的位置，查找失败指向last1。

```c++
#include <algorithm>
int count(_InIt _First, _InIt _Last, const _Ty& _Val);

int main(void)
{
    int a[] = {0,1,2,3,4,5,6,6,7,8};
    vector<int> vec(a, 10);
    count(vec.begin(), vec.end(), 6);  //统计6的个数
    binary_search(iv.begin(), iv.end(), 4);  //查找序列中是否存在4
    find(vec.begin(), vec.end(), 4);  //返回元素为4的元素的下标位置
    search(iv.begin(), iv.end(), iv3.begin(), iv3.end())；
}
```

#### 8.2 排序和通用算法

**merge**：合并两个有序序列，存放到另一个序列。

**reverse**：将指定范围内元素重新反序排序。

**sort**：以升序重新排列指定范围内的元素。

```c++
#include <algorithm>
sort(vec.begin(), vec.end()); //采用的是从小到大的排序
//自定义从大到小的比较器，用来改变排序方式
bool Comp(const int& a, const int& b){
    return a > b;
}
sort(vec.begin(), vec.end(), Comp);
```

