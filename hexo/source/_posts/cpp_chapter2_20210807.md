---
title: C++ Primer Plus 课后习题 第二章
excerpt: 开始学习C++; 进入C++; C++语句; 其他C++语句; 函数; 总结
index_img: /img/post/answer/cpp/chapter2.png
date: 2021-08-04 7:00:00
categories: [课后习题答案, C++ Primer Plus 6th]
---
## 第二章 开始学习C++

### 2.6.1 

> C++程序的模块叫什么？

``` tex
函数。C++程序由一个或多个被称为函数的模块组成。
```



### 2.6.2 

> 下面的预处理器编译指令是做什么用的？ 
>
> #include \<iostream>

``` bash
该编译指令导致预处理器将iostream文件的内容添加到程序中。这是一种典型的预处理操作：在源文件代码被编译之前，替换或添加文本。
其中iostream中的io指的是输入和输出，C++的输入输出方案涉及iostream文件中的多个定义。
```



### 2.6.3 

> 下面的语句是做什么用的？
>
> using namespace std;

``` bash
C++使用名称空间来管理不同厂商提供的相同函数名，这里表示指定编译器使用std名称空间下的函数。
```



### 2.6.4 

> 什么语句可以用来打印短语 "Hello, world", 然后开始新的一行？

``` c++
#include<iostream>
using namespace std;

int main(){
	cout << "Hello, world" << endl;
    return 0;
}
```



### 2.6.5 

> 什么语句可以用来创建名为cheeses的整数变量？

``` c++
#include<iostream>
using namespace std;

int main(){
	int cheeses;
    return 0;
}
```



### 2.6.6 

> 什么语句可以用来将值32赋给变量cheeses？

``` C++
#include<iostream>
using namespace std;

int main(){
	int cheeses;
	cheeses = 32;
    return 0;
}
```



### 2.6.7 

> 什么语句可以用来将从键盘输入的值读入变量cheeses中？

``` c++
#include<iostream>
using namespace std;

int main(){
	cin >> cheeses;
    return 0;
}
```



### 2.6.8 

> 什么语句可以用来打印“We have X varieties of cheese”，其中X为变量cheeses的当前值？

``` c++
#include<iostream>
using namespace std;

int main(){
    int cheeses;
	cin >> cheeses;
	cout << "We have X varieties of " << cheeses << endl;
    
    return 0;
}
```



### 2.6.9 

> 下面的函数原型指出了关于函数的哪些信息？
>
> int froop(double t);
>
> void rattle(int n);
>
> int prune(void);

``` tex
定义了函数的返回类型。
定义了函数的传入参数类型。
```



### 2.6.10

> 定义函数时，在什么情况下不必使用关键字return？

```tex
在函数不需要具体返回值时。
```



### 2.6.11

> 假设您编写的main()函数包含如下代码：
>
> cout << "Please enter you PIN: ";
>
> 而编译器指出cout是一个未知标识符，导致这种问题的原因是什么？指出三种修复这种问题的方法。

```
未引入cout的相关声明
未引入相应名称空间
```

```C++
// 方案一
#include<iostream>
using namespace std;

int main(){
    cout << "Please enter you PIN: ";
    return 0;
}


//方案二
#include<iostream>

int main(){
    std::cout << "Please enter you PIN: ";
    return 0;
}


//方案三
#include<iostream>

int main(){
    using std::cout;
    cout << "Please enter you PIN: ";
    
    return 0;
}
```



### 2.7.1 

> 编写一个C++程序，它显示您的姓名和地址

```c++
#include<iostream>
using namespace std;

int main(){
    cout << "Yuanzhi" << endl;
    cout << "yuanzhi@ruc.edu.cn" << endl;
    
    return 0;
}
```



### 2.7.2 

> 编写一个C++程序，它要求用户输入一个以long为单位的距离，然后将它转换为码（一long等于200码）。

```C++
#include<iostream>
using namespace std;

int main(){
    double distance = 0.0;
    cin >> distance;
    cout << distance*220;
    return 0;
}
```



### 2.7.3

> 编写一个C++程序，它使用3个用户定义的函数（包括main()），并生成下面的输出：
>
> Three blind mice
>
> Three blind mice
>
> See how they run
>
> See how they run
>
> 其中一个函数要调用两次，该函数生成前两行；另一个函数也要被调用两次，生成余下的输出。

```C++
#include<iostream>
using namespace std;

void cluster_one(){
    cout << "Three blind mice" << endl;
}

void cluster_two(){
    cout << "See how they run" << endl;
}

int main(){
    cluster_one();
    cluster_one();
    cluster_two();
    cluster_two();
    return 0;
}
```



### 2.7.4

> 编写一个程序，让用户输入其年龄，然后显示该年龄包含多少个月，如下所示：
>
> Enter your age: 29

```C++
#include<iostream>
using namespace std;

int main(){
    int age_years = 0;
    cout << "Enter your age: ";
    cin >> age_years;
    
    cout << age_years * 12;
    
    return 0;
}
```



### 2.7.5

> 编写一个程序，其中的main()调用一个用户定义的函数（以摄氏温度值为参数，并返回相应的华氏温度值）。该程序按照下面的格式要求用户输入摄氏温度值，并显示结果：
>
> Please enter a Celsius value: 20
>
> 20 degrees Celsius is 68 degrees Fahrenheit.
>
> 下面是转换公式：华氏温度=1.8 × 摄氏温度 + 32.0

```C++
#include<iostream>
using namespace std;

double convert_celsius_to_fahrenheit(double celsius_value){
    return 1.8*celsius_value + 32.0;
}

int main(){
    double celsius_value = 0.0;
    cout << "Please enter a Celsius value: ";
    cin >> celsius_value;
    
    fahrenheit_value = convert_celsius_to_fahrenheit(celsius_value);
    cout << celsius_value << " degrees Celsius is " << fahreheit_value << " degrees Fahrenheit." << endl;
    return 0;
}
```



### 2.7.6

> 编写一个程序，其main()调用一个用户定义的函数（以光年值为参数，并返回对应天文单位的值）。该程序按下面的格式要求用户输入光年值，并显示结果：
>
> Enter the number of light years: 4.2
>
> 4.2 light years = 265608 astronomical units.
>
> 转换公式：1光年=63240天文单位。

```C++
#include<iostream>
using namespace std;

double conver_lightyears_to_astronomical(double light_years){
    return light_years * 63240;
}

int main(){
    double light_years = 0.0;
    cout << "Enter the number of light years: ";
    cin >> light_years;
    
    astronomical = conver_lightyears_to_astronomical(light_years);
    
    cout << light_years << " light years = " << astronomical << " astronomical units" << endl;
    return 0;
}
```



### 2.7.7

> 编写一个程序，要求用户输入小时数和分钟数。在main()函数中，将这两个值传递给一个void函数，后者以下面这样的格式显示这两个值：
>
> Enter the number of hours: 9
>
> Enter the number of minutes: 28
>
> Time: 9:28

```C++
#include<iostream>
using namespace std;

void concat_hour_and_minute(int hours, int minutes){
    cout << hours << ":" << minutes << endl;
}

int main(){
    int hours = 0;
    int minutes = 0;
    cout << "Enter the number of hours: ";
    cin >> hours;
    cout << "Enter the number of minutes: ";
    cin >> minutes;
    concat_hour_and_minute(hours, minutes);
    
    return 0;
}
```

