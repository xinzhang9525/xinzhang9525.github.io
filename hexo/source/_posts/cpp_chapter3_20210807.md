---
title: C++ Primer Plus 课后习题 第三章
excerpt: 处理数据; 简单变量; const限定符; 浮点数; C++算数运算符; 总结
index_img: /img/post/answer/cpp/chapter3.jpeg
date: 2021-08-04 9:00:00
categories: [课后习题答案, C++ Primer Plus 6th]
comments: true
---
## 第三章 处理数据

### 3.6.1

> 为什么C++有多种整型？

```Tex
C++中多种整型的存在是为了针对特殊场景提供合适的解决方案。
C++中不同整型使用不同内存存储，使用的内存越大，则可以表示的整数值范围越大。同时有些整型可以表示无符号数，有些可以表示有符号数，而他们所使用的内存不一样。
一般而言会针对特定的场景选用合适的整型，以求在需求和性能上达到平衡。
```



### 3.6.2

> 声明与下述描述相符的变量。

``` c++
// short整型， 值为80
short a = 80;

// unsigned int整数，值为42110
unsigned int b = 42110;

// 值为3000000000的整数
long c = 3000000000;
```



### 3.6.3

> C++提供了什么措施来防止超出整型的范围？

``` c++
// C++没有提供自动防止超出整型限制的功能，可以使用头文件climits来确定每个整型的范围大小
#include<iostream>
#include<climits>
using namespace std;

int main(){
    cout << INT_MAX << endl;
    cout << INT_MIN << endl;
    return 0;
} 
```



### 3.6.4

> 33L和33之间有什么区别

``` C++
整型类型不一样。33L是long类型，33是int类型。
```



### 3.6.5

> 下面两条C++语句是否等价？
>
> char grade = 65;
> char grade = 'A';

```tex
不一定等价。
65是int整型变量，而‘A’是字符型变量。在ASCII编码的系统中，65对应的ASCII字符是’A‘.
但若系统不是ASCII编码的系统，则65可能对应其他字符。
```



### 3.6.6

> 如何使用C++来找出编码88表示的字符？指出至少两种方法。

```C++
//方法一
char ch = 88;
cout << ch << endl;

//方法二
cout << char(88) << endl;

//方法三
cout.put(char(88))
```



### 3.6.7

> 将long值赋给float变量会导致舍入误差，将long值赋给double变量呢？将long long值赋给double变量呢？

```tex
取决于类型的长度，若long为4字节，则能表达的数字为10位，而double至少表达的位数为13位，因此不存在舍入误差。但long long能表达的数字为19位，超过了double所能表达的范围，因此存在舍入误差。
```



### 3.6.8

> 下列C++表达式的结果分别是多少？
>
> a. 8*9+2
>
> b. 6*3/4
>
> c. 3/4*6
>
> d. 6.0*3/4
>
> e. 15%4

```C++
8*9+2 = 74
6*3/4 = 4
3/4*6 = 0
6.0*3/4 = 4.5
15%4 = 3
```



### 3.6.9

> 假设x1和x2是两个double变量，您要将他们作为整数相加，再将结果赋给一个整型变量。请编写一条完成这项任务的C++语句。如果要将它们作为double值相加并转换为int呢？

```C++
int value = int(x1) + int(x2);
int value = int(x1+x2);
```



### 3.6.10

> 下面每条语句声明的变量都是什么类型？
>
> a. auto cars = 15;
>
> b. auto iou = 150.37f;
>
> c. auto level = 'B';
>
> d. auto crat = U'/U00002155';
>
> e. auto fract = 8.25f/2.5;

```C++
/*
a: int
b: float
c: char
d: char32_t
e: float
*/

//  验证变量类型代码
#include<iostream>
#include<typeinfo>
using namespace std;

int main(){
    auto fract = 8.25f/2.5;
    cout << "Type is: " << typeid(fract).name() << endl;
    return 0;
}
```



### 3.7.1

> 编写一个小程序，要求用户使用一个整数指出自己的身高(单位为英寸)，然后将身高转换为英尺和英寸。该程序使用下划线字符来指示输入位置，另外，使用一个const符号常量来表示转换因子。

```C++
#include<iostream>
using namespace std;

int main(){
    const int factor = 12;
    int heights = 0;
    cont << "_";
    cin >> heights;
    int height_foot = heights / factor;
    int height_inch = heights % factor;
    cout << height_foot << height_inch;
    return 0;
}
```



### 3.7.2

> 编写一个小程序，要求以几英尺几英寸的方式输入其身高，并以磅为单位输入其体重（使用三个变量来存储这些信息）。该程序报告其BMI（Body Mass Index，体重指数）。为了计算BMI，该程序以英寸的方式指出用户的身高（1英寸为12英尺），并将以英寸为单位的身高转换为以米为单位的身高（1英寸=0.0254米）。然后，将以磅为单位的体重转换为以千克为单位的体重（1千克=2.2磅）。最后，计算相应的BMI-体重（千克）除以身高（米）的平方。用符号常量表示各种转换因子。

```C++
#include<iostream>
using namespace std;

int main(){
    int height_foot = 0;
    int height_inch = 0;
    double weight = 0.0;
    cout << "Enter your height and weight: " << endl;
    cin >> height_foot >> height_inch >> weight;
    
    double height_meter = (height_foot*12 + height_inch)*0.0254;
    double weight_kg = weight/2.2;
    double bmi = weight_kg / (height_meter * height_meter);
    cout << "Your BMI: " << bmi << endl;
    return 0;
}
```



### 3.7.3

> 编写一个程序，要求用户以度、分、秒的方式输入一个纬度，然后以度为单位显示该纬度。1度为60分，1分等于60秒，请以符号常量的方式表示这些值。对于每个输入值，应使用一个独立的变量存储它。下面是该程序运行时的情况：
>
> Enter a latitude in degrees, minutes, and seconds:
>
> First, enter the degrees: 37
>
> Next, enter the minutes of arc: 51
>
> Finally, enter the seconds of arc: 19
>
> 37 degrees, 51 minutes, 19 seconds = 37.8553 degrees

```C++
#include<iostream>
using namespace std;

int main(){
    int degrees = 0;
    int minutes = 0;
    int seconds = 0;
    const int time_factor = 60;

    cout << "Enter a latitude in degrees, minutes, and seconds:" << endl;
    cout << "First, enter the degrees:";
    cin >> degrees;
    cout << "Next, enter the minutes of arc: ";
    cin >> minutes;
    cout << "Finally, enter the seconds of arc:";
    cin >> seconds;
    
    double degrees_beta = degrees + (double)minutes / time_factor + (double)seconds / (time_factor*time_factor);
    
    cout << degrees <<" degrees, " << minutes << " minutes, " << seconds << " seconds = " << degrees_beta << " degrees" << endl;
    
    return 0;
}
```



### 3.7.4

> 编写一个程序，要求用户以整数方式输入秒数（使用long或long long存储），然后以天、小时、分钟和秒的方式显示这段时间。使用符号常量来表示每天有多少小时、每小时有多少分钟以及每分钟有多少秒。该程序的输出应与下面类似：
>
> Enter the number of seconds: 31600000
>
> 31600000 seconds = 365 days, 17 hours, 46 minutes, 40 seconds

```C++
#include<iostream>
using namespace std;

int main(){
    int days = 0;
    int hours = 0;
    int minutes = 0;
    int seconds = 0;
    long long big_seconds = 0;
    const int one_day_equal_hours = 24;
    const int one_hour_equal_minutes = 60;
    const int one_minute_equal_seconds = 60;
    
    cout << "Enter the number of seconds: ";
    cin >> big_seconds;
    
    long long big_seconds_temp = big_seconds;
    
    days = big_seconds_temp/(one_day_equal_hours*one_hour_equal_minutes*one_minute_equal_seconds);
    big_seconds_temp = big_seconds_temp%(one_day_equal_hours*one_hour_equal_minutes*one_minute_equal_seconds);
    
    hours = big_seconds_temp / (one_hour_equal_minutes*one_minute_equal_seconds);
    big_seconds_temp = big_seconds_temp % (one_hour_equal_minutes*one_minute_equal_seconds);
    
    minutes = big_seconds_temp / one_minute_equal_seconds;
    seconds = big_seconds_temp % one_minute_equal_seconds;
    
    cout << big_seconds << " seconds = " << days << " days, " << hours << " hours, " << minutes << " minutes, " << seconds << " seconds." << endl;
    
    return 0;
}
```



### 3.7.5

> 编写一个程序，要求用户输入全球当前的人口和美国当前的人口（或其他国家的人口）。将这些信息存储在long long变量中，并让程序显示美国（或其他国家）的人口占全球人口的百分比。该程序的输出应与下面类似：
>
> Enter the world's population: 6898758899
>
> Enter the population of the US: 310783781
>
> The population of the US is 4.50492% of the world population

```C++
#include<iostream>
using namespace std;

int main(){
    long long world_population = 0;
    long long us_population = 0;
    
    cout << "Enter the world's population: ";
    cin >> world_population;
    cout << "Enter the population of the US: ";
    cin >> us_population;
    
    cout << "The population of the US is " << (double)us_population / world_population * 100 << "% of the world population" << endl;
    return 0;
}
```



### 3.7.6

> 编写一个程序，要求用户输入驱车里程（英里）和使用汽油量（加仑），然后指出汽车耗油量为一加仑的里程。如果愿意，也可以让程序要求用户以公里为单位输入距离，并以升为单位输入汽油量，然后指出欧洲风格的结果-即每100公里的耗油量（L）。

```C++
#include<iostream>
using namespace std;

int main(){
    double distance = 0.0;
    double gas = 0.0;
    cout << distance / gas << endl;
    
    return 0;
}
```



### 3.7.7

> 编写一个程序，要求用户按照欧洲风格输入汽车的耗油量（每100公里消耗的汽油量（升）），然后将其转换为美国风格的耗油量--每加仑多少英里。注意，除了使用不同的单位计量外，美国方法（距离/燃料）与欧洲方法（燃料/距离）相反。100公里等于62.14英里，1加仑等于3.785升。因此，19mpg大约合12.41/100km，27mpg大约合8.71/100km。

```C++
#include<iostream>
using namespace std;

int main(){
    double total_gas_eng = 0.0;
    double total_gas_usa = 0.0;
    const double hundred_km_equal_miles = 62.14;
    const double one_gallon_equal_litres = 3.785;

    cout << "Enter the car's gas waste: ";
    cin >> total_gas_eng;
    
    double gallons = total_gas_eng  / one_gallon_equal_litres;
    cout << "The mpg is: " << hundred_km_equal_miles/gallons << endl;
    
    return 0;
}
```

