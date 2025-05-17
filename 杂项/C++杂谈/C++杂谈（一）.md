## 判断容器是否为空

每个容器都有一个 end 迭代器，这个迭代器不会存储任何有效的元素，只是用来表明到达此处意味着容器的元素已经访问完毕。

特殊情况下 begin 迭代器和 end 迭代器会相等，这就意味着容器为空，没有有效的迭代器，不存在要这个容器的元素进行相关操作（根本就没有存储任何元素）。

因此，我们在操作容器的时候，有必要进行begin 迭代器和 end 迭代器是否相等的判断，合理之后再进行相应的操作。

```c++
void isEmpty(const string& name) {
	if (name.begin() != name.end()) {
		cout << "name is no empty \n";
	}
	else {
		cout << "name is empty \n";
	}
}
```

测试代码：

```c++
int main() {

	string n1 = "xy";
	isEmpty(n1);

	string n2;
	isEmpty(n2);

	string n3 = "";
	isEmpty(n3);

	return 0;
}

/*
name is no empty
name is empty
name is empty
*/
```

## extern 关键字

我不知道你了解头文件重复包含问题不，尽管我们会写头文件保护，但是不代表就不会出现头文件重复包含到来的重定义问题。

global.h

```c++
#ifndef DAY05_EXTERN_GLOBAL_H
#define DAY05_EXTERN_GLOBAL_H
#include <string>
int global_age = 10;
std::string global_name = "xy";
#endif
```

global.cpp

```c++
#include "global.h"
```

main.cpp

```c++
#include <iostream>
#include "global.h"
int main() {
    std::cout << "Hello, World!" << std::endl;
    std::cout << "globbal name is" << global_name << std::endl;
    std::cout << "global age is " << global_age << std::endl;
    return 0;
}
```

这个时候我们将其替换一下：

main.cpp

```c++
#include <iostream>
#include <string>
int global_age = 10;
std::string global_name = "xy";
int main() {
    std::cout << "Hello, World!" << std::endl;
    std::cout << "globbal name is" << global_name << std::endl;
    std::cout << "global age is " << global_age << std::endl;
    return 0;
}
```

global.cpp

```c++
#include <string>
int global_age = 10;
std::string global_name = "xy";
```

链接的时候就会发现 main.cpp 和 global.cpp 重定义，导致程序无法生成可执行程序。

我们可以利用 extern 关键字声明变量，而不是定义变量。

当你在一个源文件中使用一个变量，但这个变量并不在当前文件中定义，而是在其他文件中定义时，你可以使用 `extern` 来声明这个变量。这样，编译器就知道这个变量在其他地方有定义，并且会链接到正确的位置。

global.h

```c++
#ifndef DAY05_EXTERN_GLOBAL_H
#define DAY05_EXTERN_GLOBAL_H
#include <string>
extern int global_age;
extern global_name;
#endif
```

global.cpp

```c++
#include "global.h"
int global_age = 10;
std::string global_name = "xy";
```

main.cpp

```C++
#include <iostream>
#include "global.h"
int main() {
    std::cout << "Hello, World!" << std::endl;
    std::cout << "globbal name is" << global_name << std::endl;
    std::cout << "global age is " << global_age << std::endl;
    return 0;
}
```

这个时候我们将其替换一下：

main.cpp

```C++
#include <iostream>
#include <string>
extern int global_age;
extern global_name;
int main() {
    std::cout << "Hello, World!" << std::endl;
    std::cout << "globbal name is" << global_name << std::endl;
    std::cout << "global age is " << global_age << std::endl;
    return 0;
}
```

global.cpp

```c++
extern int global_age;
extern global_name;
int global_age = 10;
std::string global_name = "xy";
```

extern 声明不是定义，也不分配存储空间。事实上，它只是说明变量定义在程序的其他地方。因此同一个程序中出现 相同的声明是不会报错的。

## 常量和非常量迭代器

只读：常量迭代器，`std::vector<int>::const_iterator it`

读写：非常量迭代器，`std::vector<int>::iterator it`

## 合并两个容器到另一个容器中

```c++
vector<int>	data1 = { 1, 2, 3, 4 };
vector<int>	data2 = { 9, 10, 8 };

{
	vector<int> result;
	result.insert( result.begin(), data1.begin(), data1.end() );
	result.insert( result.begin() + data1.size(), data2.begin(), data2.end() );

	for ( const auto & it : result )
	{
		cout << it << " ";
	}
}

cout << endl;

{
	vector<int> result;
	result.insert( result.end(), data1.begin(), data1.end() );
	result.insert( result.end(), data2.begin(), data2.end() );

	for ( const auto & it : result )
	{
		cout << it << " ";
	}
}
```

对比来看的话，第二种方式更简单和统一。

## 不要在构造函数中调用 return

我们有些人在构造函数会进行相关操作，比方说打开某个文件描述符，如果打开失败就调用 return 返回。

这样你以为就会导致程序结束？

非也！只会让这个对象构造不完整，交给其他人使用，这是相当不恰当的。你可以选择直接 exit 退出程序，或者 抛出异常来终止程序。

## delete 和 nullptr 的区别

`delete` 一个对象用来释放动态分配的内存，并调用对象的析构函数。

- **作用：** `delete` 会释放对象占用的内存空间（如果是通过 `new` 动态分配的内存），并调用对象的析构函数，销毁对象。

&nbsp;

将一个对象指针置为 `nullptr` 是一个简单的指针操作，它不会涉及内存的释放或析构函数的调用。

- **作用：** 将指针设为 `nullptr` 仅仅是改变指针的值，让它不再指向原来的对象。这并不会释放内存，也不会调用析构函数。

## malloc 和 new 浅谈

malloc分配内存，但不会初始化数据，都是脏数据。因此  new 的出现有个好处，它是由两步操作组成：分配内存，然后初始化这块空间。分配内存由malloc完成，初始化空间的数据是构造函数完成。

## 为什么需要初始化列表

> [!NOTE]
>
> 实际上这个问题是在问哪些成员变量必须在初始化列表完成初始化，而在作用域内赋值是不被允许的

- 常量成员变量
- 引用成员变量
- 基类成员（因为派生类继承基类，需要初始化基类）
- 成员变量是类对象，如果此类对象没有默认构造就必须得在初始化列表初始化，否则会报错

最后这句话不容易被理解，如果这个成员变量的类对象是引用或指针，可以不用在初始化列表来做，完全可以在作用域内赋值。但如果你这个成员变量的类对象的生命周期伴随这个类对象的死亡而死亡，比如：

```c++
class Test {
public:
	Test() {}
	~Test() {}
private:
	MyClass obj;	// 生命周期和类对象共存，会自动调用默认构造，如果你不在初始化列表初始化这个对象的话
};
```

那么 MyClass 如果没有默认构造，你就需要在 初始化列表对其进行初始化，否则就要报错，下面是正确的写法：

```c++
class MyClass
{
public:
	MyClass(int num) :num_(num)
	{}
	~MyClass() {}
private:
	int num_;
};

class Test {
public:
	Test() : obj(10){	// 初始化列表初始化
	}
	~Test() {}
private:
	MyClass obj;
};
```

## 静态成员初始化

静态成员不属于具体的类，因此不会在构造函数中初始化或赋值，而是在对应的源文件中进行初始化（通常选择在最前方）。

因为构造函数是在创建对象的时候会自动调用，那么构造函数中初始化或赋值的成员变量就属于这个对象，因此不会在构造函数中初始化或赋值静态成员。
