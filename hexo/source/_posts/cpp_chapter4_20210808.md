---
title: C++ Primer Plus 课后习题 第四章
excerpt: 数组; 字符串; String类简介; 结构简介; 共用体; 枚举; 指针; 总结
index_img: /img/post/answer/cpp/chapter4.jpeg
date: 2021-08-05 9:00:00
categories: [课后习题答案, C++ Primer Plus 6th]
comments: true
---
## 第四章 复合类型

### 4.12.1

> 如何声明下述数据：

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



### 4.12.2

> 使用模板类array而不是数组来完成问题1。

```C++
#include<array>
// a. actor是由30个char组成的数组
array<char, 30> actor;
    
// b. betsie是由100个short组成的数组
array<short, 100> betsie;

// c. chuck是由13个float组成的数组
array<float, 13> chuck;

// d. dipsea是由64个long double组成的数组
array<long double, 64> dipsea;
```



### 4.12.3

> 声明一个包含5个元素的int数组，并将它初始化为前5个正奇数。

```C++
//方案一
int a[5] = {1, 3, 5, 7, 9};

//方案二
array<int, 5> a = {1, 3, 5, 7, 9};

//方案三
vector<int> a(5) = {1, 3, 5, 7, 9}
```



### 4.12.4

> 编写一条语句，将问题3中数组第一个元素和最后一个元素的和赋值给变量even.

```C++
// 方案一
int even = a[0] + a[4];

//方案二, 方案三
int even = a.at(0) + a.at(4);
int even = a[0] + a[4];
```



### 4.12.5

> 编写一条语句，显示float数组ideas中的第二个元素的值

```C++
cout << ideas[2];
```



### 4.12.6

> 声明一个char的数组，并将其初始化为字符串"cheeseburger".

```C++
char a[] = "cheeseburger";
```



### 4.12.7

> 声明一个string对象，并将其初始化为字符串“Waldorf Salad”.

```C++
#include<string>
string strs = "Waldorf Salad";
```



### 4.12.8

> 设计一个描述鱼的结构声明。结构中应当包括品种、重量（整数，单位为盎司）和长度（英寸，包括小数）.

```C++
struct fish{
    string brand;
    int weight;
    double length;
};
```



### 4.12.9

> 声明一个问题8中定义的结构的变量，并对他进行初始化。

```C++
fish golden_fish = {
    "golden",
    13,
    12.99
};
```



### 4.12.10

> 用enum定义一个名为Response的类型，它包含Yes、No、Maybe等枚举量，其中Yes的值为1，No的值为0， Maybe的值为2。

```C++
enum Response {Yes=1, No=0, Maybe=2};
```



### 4.12.11

> 将设ted是一个double变量，请声明一个指向ted的指针，并使用该指针来显示ted的值。

```C++
double ted = 9.9;
double *pted = &ted;
```



### 4.12.12

> 假设treacle是一个包含10个元素的float数组，请声明一个指向treacle的第一个元素的指针，并使用该指针来显示数组的第一个元素和最后一个元素。

```C++
float treacle[10];
float *ptreacle = &treacle;
cout << ptreacle[0] << endl;
cout << ptreacle[9] << endl;
```



### 4.12.13

> 编写一段代码，要求用户输入一个正整数，然后创建一个动态的int数组，其中包含的元素数目等于用户输入的值。首先使用new来完成这项任务，再使用vector对象来完成这项任务。

```C++
#include<iostream>
#include<vector>
using namespace std;

int main(){
    int n=0;
    cin >> n;
    int *parrs = new int[n];
    
    vector<int> arrs(n);
    
    delete parrs;
    return 0;
}
```



### 4.12.14

> 下面的代码是否有效？如果有效，它将打印出什么结果？
>
> cout << (int *) "Home of the jolly bytes";

```C++
输出：0x4007c1
有效，表达式“Home of the jolly bytes”为一个string类型数据，cout将其地址解释为打印字符串。但类型 （int *）将其地址强转为int指针，cout将其int指针内容输出，所以最终输出是地址信息。
```



### 4.12.15

> 编写一段代码，给问题8中描述的结构动态分配内存，再读取该结构的成员的值。

```C++
struct fish{
    string brand;
    int weight;
    double length;
};

fish *pglodenfish = new fish;
cout << pgoldenfish->weight;
cout << pgoldenfish->brand;
```



### 4.12.16

> 程序清单4.6指出了混合输入数字和一行字符串时存储的问题，如果将下面的代码：
>
> cin.getline(address, 80);
>
> 替换为：
>
> cin >> address;
>
> 将对程序的运行带来什么影响？

```C++
程序将不存在address获取空格符的问题，可以获取完整的address内容。
cin >> address使得程序跳过空白符、换行符，直接捕捉到address内容。但存在的问题是若整行中存在空格符，则遇到空格符后的内容都会被丢弃。
```



### 4.12.17

> 声明一个vector对象和一个array对象，他们都包含10个string对象，指出所需的头文件，但不要使用using。使用const来指定要包含的string对象数。

```C++
#include<iostream>
#include<vector>
#include<array>
#include<string>

const int str_nums = 10;

int main(){
    std::vector<std::string> vstrs(str_nums);
    std::array<std::string, str_nums> astrs;
    
    return 0;
}
```



### 4.13.1

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



### 4.13.2

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





### 4.13.3

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



### 4.13.4

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



### 4.13.5

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



### 4.13.6

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



### 4.13.7

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



### 4.13.8

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



### 4.13.9

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



### 4.13.10

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

