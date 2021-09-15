---
title: C++ Primer Plus 课后习题 第五章
excerpt: for循环; 表达式和语句; 复合语句; 逗号运算符; while循环; 总结
index_img: /img/post/answer/cpp/chapter5.jpeg
date: 2021-08-06 9:00:00
categories: [课后习题答案, C++ Primer Plus 6th]
comments: true
---
## 第五章 复合类型

### 5.8.1

> 入口条件循环和出口条件循环之间的区别是什么？各种C+=循环分别属于其中的哪一种？

```C++
// a. actor是由30个char组成的数组
char actor[30];
    
// b. betsie是由100个short组成的数组
char betsie[100];

// c. chuck是由13个float组成的数组
float chuck[13];

// d. dipsea是由64个long double组成的数组
long double dipsea[64]
```



### 5.8.2

> 如果下面的代码片段是有效程序的组成部分，它将打印什么内容？

```C++
int i;
for(int i = 0; i < 5; i++)
    cout << i;
    cout << endl;
```



### 5.8.3

> 如果下面的代码片段是有效程序的组成部分，它将打印什么内容？

```C++
int j;
for(int j = 0; j < 11; j+=3)
    cout << j;
cout << endl << j << endl;
```



### 5.8.4

> 如果下面的代码片段是有效程序的组成部分，它将打印什么内容？

```C++
int j = 5;
while ( ++j < 9)
    cout << j++ << endl;
```



### 5.8.5

> 如果下面的代码片段是有效程序的组成部分，它将打印什么内容？

```C++
int k = 8;
do
    cout << " k = " << k << endl;
while(k++ < 5);
```



### 5.8.6

> 编写一个打印1、2、4、8、16、32、64的for循环，每轮循环都将计数变量的值乘以2.

```C++
char a[] = "cheeseburger";
```



### 5.8.7

> 如何在循环体中包括多条语句？

```C++
#include<string>
string strs = "Waldorf Salad";
```



### 5.8.8

> 下面的语句是否有效？如果无效，原因是什么？如果有效，它将完成什么工作？

```C++
int x = (1,024);

//下面的语句又如何呢？
int y;
y = 1,024;
```



### 5.8.9

> 在查看输入方面，cin >> ch同cin.get(ch)和ch=cin.get()有什么不同？

```C++
fish golden_fish = {
    "golden",
    13,
    12.99
};
```



### 5.9.1

> 编写一个C++程序，如下述输出示例所示的那样请求并显示信息：
>
> What is your first name? Betty Sue
>
> What is your last name? Yewe
>
> What letter grade do you deserve? B
>
> What is your age? 22
>
> Name: Yewe, Betty Sue
>
> Grade: C
>
> Age: 22
>
> 注意：该程序应该接受的名字包含多个单词。此外，程序将向下调整成绩，即向上调一个字母。假设用户请求A、B、或C，所以不必担心D和F之间的空档。

```C++
#include<iostream>
#include<string>

using namespace std;

struct peopleinfo{
    char *name[2];
    char grade;
    unsigned int age;
};
    
int main(){
    peopleinfo people;
    cout << "What is your first name? ";
    char name[20];
    cin.getline(name);
    cin.get();
    people.name[0] = name;
    
    cout << "What is your last name? ";
    cin.getline(name);
    cin.get();
    people.name[1] = name;
    
    cout << "What letter grade do you deserve? ";
    cin >> people.grade;
    
    cout << "What is your age? ";
    cin >> people.age;
    
    cout << "Name: " << people.name[0] << ", " << people.name[1] << endl;
    cout << "Grade: " << ++people.grade << endl;
    cout << "Age: " << people.age << endl;
    
    return 0;
}
```



### 5.9.2

> 修改程序清单4.4，使用C++ string类而不是char数组。

```C++
//清单4.4程序代码
#include<iostream>
using namespace std;

int main(){
    const int ArSize = 20;
    char name[ArSize];
    char dessert[ArSize];
    
    cout << "Enter your name: \n";
    cin.getline(name, ArSize);
    cout << "Enter your favorite dessert: \n";
    cin.getline(dessert, ArSize);
    cout << "I have some delicious " << dessert;
    cout << " for you, " << name << endl;
    
    return 0;
}
```

```C++
//string类改写版程序代码
#include<iostream>
#include<string>
using namespace std;

int main(){
    string name, dessert;
    
    cout << "Enter your name: \n";
    getline(cin, name);
    cout << "Enter your favorite dessert: \n";
    getline(cin, dessert);
    cout << "I have some delicious " << dessert;
    cout << " for you, " << name << endl;
    
    return 0;
}
```





### 5.9.3

> 编写一个程序，它要求用户首先输入其名，然后输入其姓；然后程序使用一个逗号和空格将姓和名组合起来，并存储和显示组合结果。请使用char数组和头文件cstring中的函数。下面是该程序运行时的情形：
>
> Enter your first name: Flip
>
> Enter your last name: Fleming
>
> Here's the information in a single string: Fleming, Flip

```C++
#include<iostream>
#include<cstring>
using namespace std;

int main(){
    char name[2][20];
    cout << "Enter your first name: ";
    cin.getline(name[0], 20);
    cout << "Enter your last name: ";
    cin.getline(name[1], 20);
    cout << "Here's the information in a single string: ";
    cout << strcat(strcat(name[0], ", "), name[1]);
    
    return 0;
}
```



### 5.9.4

> 编写一个程序，它要求用户首先输入其名，再输入其姓；然后程序使用一个逗号和空格将姓和名组合起来，并存储和显示组合结果。请使用string对象和头文件string中的函数。下面是该程序运行时的情形：
>
> Enter your first name: Flip
>
> Enter your last name: Fleming
>
> Here's the information in a single string: Fleming, Flip

```C++
#include<iostream>
#include<string>
using namespace std;

int main(){
    string name[2];
    cout << "Enter your first name: ";
    getline(cin, name[0]);
    cout << "Enter your last name: ";
    getline(cin, name[1]);
    cout << "Here's the information in a single string: ";
    cout << name[0] + ", " + name[1] << endl;
    
    return 0;
}
```



### 5.9.5

> 结构CandyBar包含三个成员。第一个成员存储了糖块的品牌；第二个成员存储糖块的重量（可以有小数）；第三个成员存储了糖块的卡路里含量（整数）。请编写一个程序，声明这个结构，创建一个名为snack的CandyBar变量，并将其成员分别初始化为“Mocha Munch”、2.3和350.初始化应在声明snack时进行。最后，程序显示snack变量的内容。

```C++
#include<iostream>
#include<string>
using namespace std;

struct CandyBar{
    string brand;
    double weight;
    unsigned int calorie;
};

int main(){
    CandyBar snack = {
        "Mocha Munch",
        2.3,
        350
    };
    
    cout << snack.brand << endl;
    cout << snack.weight << endl;
    cout << snack.calorie << endl;
    
    return 0;
}
```



### 5.9.6

> 结构CandyBar包含3个成员，如编程练习5所示。请编写一个程序，创建一个包含3个元素的CandyBar数组，并将它们初始化为所选择的值，然后显示每个结构的内容。

```C++
#include<iostream>
#include<string>
using namespace std;

struct CandyBar{
    string brand;
    double weight;
    unsigned int calorie;
};

int main(){
    CandyBar snack[3] = {
        {"Mocha Munch", 2.3, 350},
        {"Mocha Munch", 2.4, 450},
        {"Mocha Munch", 2.5, 550}
    };
    
    cout << snack[0].brand << endl;
    cout << snack[1].weight << endl;
    cout << snack[3].calorie << endl;
    
    return 0;
}
```



### 5.9.7

> William Wingate从事披萨饼分析服务。对于每个披萨饼，他都需要记录下列信息：
>
> 披萨饼公司的名称，可以有多个单词组成。
>
> 披萨饼的直径。
>
> 披萨饼的重量。
>
> 请设计一个能够存储这些信息的结构，并编写一个使用这些结构变量的程序。程序将请求用户输入上述信息，然后显示这些信息。请你使用cin（或其他方法）和cout。

```C++
#include<iostream>
#include<string>
using namespace std;

struct Pizza{
    string comp;
    double diam;
    double weight;
};

int main(){
    Pizza bug_pizza;
    cout << "Enter the company name of Pizza: ";
    getline(cin, bug_pizza.comp);
    
    cout << "Enter the diameter of Pizza: ";
    cin >> bug_pizza.diam;
    
    cout << "Enter the weight of Pizza: ";
    cin >> bug_pizza.weight;
    
    cout << "company: " << bug_pizza.comp << endl;

    return 0;
}
```



### 5.9.8

> 完成编程练习7，但使用new来动态分配数组，而不是声明一个结构变量。另外，让程序在请求输入披萨饼公司名称之前输入披萨饼的直径。

```C++
#include<iostream>
#include<string>
using namespace std;

struct Pizza{
    string comp;
    double diam;
    double weight;
};

int main(){
    Pizza *bug_pizza = new Pizza;
    
    cout << "Enter the diameter of Pizza: ";
    cin >> bug_pizza->diam;
    cin.get();
    
    cout << "Enter the company name of Pizza: ";
    getline(cin, bug_pizza->comp);
    
    cout << "Enter the weight of Pizza: ";
    cin >> bug_pizza->weight;
    
    cout << "company: " << bug_pizza->comp << endl;
    
    delete bug_pizza;

    return 0;
}
```



### 5.9.9

> 完成编程练习6，但使用new来动态分配数组，而不是声明一个包含3个元素的CandyBar数组。

```C++
#include<iostream>
#include<string>
using namespace std;

struct CandyBar{
    string brand;
    double weight;
    unsigned int calorie;
};

int main(){
    CandyBar *snack = new CandyBar[3];
    *snack = {"Mocha Munch", 2.3, 350};
    *(snack+1) = {"Mocha Munch", 2.4, 450};
    *(snack+2) = {"Mocha Munch", 2.5, 550};
    
    cout << snack[0].brand << endl;
    cout << snack[1].weight << endl;
    cout << snack[3].calorie << endl;
    
    return 0;
}
```



### 5.9.10

> 编写一个程序，让用户输入三次40码跑的成绩（如果您愿意，也可以让用户输入40米跑的成绩），并显示次数和平均成绩。请使用一个array数组对象来存储数据（如果编译器不支持array类，请使用数组）。

```C++
#include<iostream>
#include<array>
using namespace std;

int main(){
    unsigned int count = 0;
    array<double, 3> match_grade;
    cout << "input: ";
    cin >> match_grade[0];
    count++;
    cin >> match_grade[1];
    count++;
    cin >> match_grade[2];
    count++;
   
    cout << "Total times: " << count << endl;
    cout << (match_grade[0] + match_grade[1] + match_grade[2]) / count << endl;
    return 0;
}
```

