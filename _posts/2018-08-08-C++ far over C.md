---
layout: post
title: C++ far over C
date: 2018-08-08
tags: [编程语言]
---
### C++ 建议使用 `const` 常量或` enum` 枚举，而不是 `#define` 宏. ###

```cpp
const double PI=3.1415926;
```

```cpp
#define PI 3.1415926
```

前者 `PI` 不等同于后者 `PI`，C++ 建议使用 `const` 常量，而不是宏。

区别如下：

1. 编译器处理方式不一样

`#define` 宏在预处理阶段展开；
`const` 常量在编译和运行阶段使用。

2. 类型和安全检查不一样

`#define` 宏没有类型，仅展开，而不做任何类型安全检查；`const` 常量有具体的类型，编译阶段会进行具体的安全检查。

3. 存储方式不同

宏定义只做展开，不分配内存；`const`常量分配或占用一定内存，存储区域为堆区或栈区。

### C++ 建议使用 `inline` 函数(内联函数)，而不是宏。 ###

```cpp
int max(int a, int b);

inline int max(int a, int b){
    return (a > b ? a : b);
}
```

```cpp
#define max(a, b)   (((a) > (b)) ? (a) : (b))
```

区别如下：
1. 宏定义容易出错，预处理器在拷贝、替换宏代码的时候常常出现意想不到的边际效应。

```cpp
#define max(a, b)  (a) > (b) ? (a) : (b)
```

则 `result = max(x, y) + 5;` 被展开为 `result = (x) > (y) ? (x) : (y) + 5;`, 由于 `+` 优先级大于 `?`，该语句等效于 `result = x > y ? x : (y + 5);`。


修改为 `#define max(a, b)  (((a) > (b)) ? (a) : (b))`，则上述问题得到解决；但如有语句 `result = max(x++, y);`，展开后为 `result = ((x++) > y) ? (x++) : y;`, 该语句中 `x` 被两次求值。

```cpp
#include <iostream>

#define MAX(a, b)   (((a) > (b)) ? (a) : (b))

using namespace std;

int main(){
    int x = 5;
    int y = 4;
    int max = MAX(x++, y);

    cout <<"x = "<<x <<",y = "<<y <<", max = "<<max <<endl;
    
    return 0;
}
```

运行结果：

```shell
x = 7,y = 4, max = 6
```

`x++` 与 `y` 的最大值应该为5, 宏运算结果与期望结果不符合。

2. 宏是不可调试的，内联函数是可以调试的。

`debug` 版本会像普通函数那样为它生成含有调试信息的可执行程序，`release` 版本才会实现真正的内联。

编译器不会为宏生成调试信息, 符号表中查找不到`MAX`符号。

```shell
kang:cpp kang$ nm test | grep MAX
kang:cpp kang$ 
```

改为内联函数, 运算结果为 `5`, 符号表中能查找到符号 `max`。

```shell
kang:cpp kang$ g++ test.cpp -o test -g -Wall
kang:cpp kang$ ./test
x = 6,y = 4, max = 5
kang:cpp kang$ nm test | grep max
0000000100000f50 T __Z3maxii
```

3. 宏无法访问类的私有数据成员, 而内联函数可以。

4. 内联函数会做类型安全检查，和自动类型转换；而宏没有数据类型，不做安全检查和类型转换。

### C++ Template (模板) ###

example-1:

```cpp
template <class T> 
inline void swap(T &a, T &b){
    T tmp = a;
    a = b;
    b = tmp;
}
```

example-2:

```cpp
template <class T>
T sum(T data[], int size, T s = 0){
    for (int i = 0; i < size; i++){
        s += data[i];
    }
    
    return s;
}
```

### Why C++ is better than C? ###
- function overloading（函数重载）；
- References（引用）；
- Templates（模板）；
- namespace（命名空间）；
- Type safety（类型安全）;
- Exceptions（异常处理机制）;
- ...

## 参考文献 ##
1. [C++ For C Programmers, Part A](https://www.coursera.org/learn/c-plus-plus-a/home/welcome)