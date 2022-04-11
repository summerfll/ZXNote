一.变量和基本类型

## 1.字符和字符串区别

单引号括起来的一个字符称为char型，如‘abs’

双引号括起来的零个或多个字符构成字符串,如“hello world”

编译器会在每个字符串的结尾处添加一个空字符‘\0’，所以字符串的实际长度比它的内容多1

## 2.类型转换

当一个算数表达式中既有无符号数又有int值时，那个int值就会转换成无符号数。

例：

~~~c
unsigned u = 10;

int i = -42;

std::cout << i + i<<std:endl;//输出-84

std::cout << u + i<<std::endl;//输出4294967264，如果int占32位

-1=4294967296-1  
~~~

### **2.1 四个显式强制转换**

static_cast:静态类型转换，编译时c++编译器会做类型检查，如int转换成char

reinterpret_cast：为运算对象的位模式提供较低层次上的重新解释

dynamic_cast：动态类型转换，如子类和父类之间的多态类型转换

const_cast：只能改变运算对象的**底层const**，字面上理解就是去const属性

**格式：**

TYPE B=static_cast<TYPE>(A)

~~~c++
//static_cast 静态类型转换，编译时c++编译器会做类型检查，基本类型能转换，但不能转换指针类型,一般隐式转化能成功的，用static_cast转换也能成功
//reinterpret_cast 强制类型转换
#include<iostream>
using namespace std;

void main()
{
	double i = 2.1333;
	int k = i;//隐式类型转换
	int a = static_cast<int>(i);//静态类型转换，编译时c++编译器会做类型检查，基本类型能转换，但不能转换指针类型,一般隐式转化能成功的，用static_cast转换也能成功

	char *p = "hello ...";
	int *s = NULL;
	s = reinterpret_cast<int *>(p);//若不同类型之间，进行强制类型转换，用reinterpret_cast<>()进行重新解释
	cout << "p: " << p << endl;//%s
	cout << "s: " << s << endl;//%d
	//总结：通过reinterpret_cast<>()和static_cast<>()把c语言中的强制类型转换都覆盖了
	system("pause");
}
~~~

~~~c++
//dynamic_cast 父类对象 ==> 子类对象 向下转型
#include<iostream>
using namespace std;
class Tree
{
public:
}
class Amimal
{
public:
	virtual void cry() = 0;
};
class Cat:public Amimal
{
public:
	virtual void cry()
	{
		cout << "喵喵。。" << endl;
	}
	void catchmouse()
	{
		cout << "catchmouse" << endl;
	}
};
class Dog :public Amimal
{
public:
	virtual void cry()
	{
		cout << "旺旺。。" << endl;
	}
	void protecthome()
	{
		cout << "protecthome" << endl;
	}
};
void play(Amimal *base)
{
	base->cry();//继承，虚函数重写，父类指针指向子类对象==>多态
	Dog *d = dynamic_cast<Dog *>(base);//dynamic_cast 运行时类型识别
	if (d != NULL)
	{
		d->protecthome();
	}
	Cat *t = dynamic_cast<Cat *>(base);//父类对象 ==> 子类对象 向下转型
	if (t != NULL)
	{
		t->catchmouse();
	}
}
void main()
{
	Dog d1;
	Cat t1;
	play(&d1);
	play(&t1);
    Tree t;
    t1=reinterpret_cast<Cat *>(t);//reinterpret_cast 强制类型转换
	system("pause");
}
~~~

结果：

![39](F:\学习专用\note\pic\39.png)

~~~c++
//const_cast 去除变量的只读属性
#include<iostream>
using namespace std;

void fun(const char *p)
{
	char *p1 = NULL;
	p1 = const_cast<char *>(p);//通过使用const_cast 去除底层const属性
	p1[0] = 'H';
	cout << p << endl;
}
void main()
{
	char buf[] = "hello";
	fun(buf);
	//char *p = "sss";//err 要保证指针所执行的内存空间嫩个修改才行 若不能修改 会引起程序异常
	//fun(p);
	system("pause");
}
~~~

## 3.初始化和赋值的区别

初始化和赋值是两个完全不同的操作，初始化是创建变量时赋予其一个初始值。

赋值：把对象的当前值擦除，以一个新值来替代

## 4.指针数组和数组指针

“数组指针”和“指针数组”，只要在名词中间加上“的”字，就知道中心了——

数组的指针：是一个指针，什么样的指针呢？指向数组的指针。

指针的数组：是一个数组，什么样的数组呢？装着指针的数组。



然后，需要明确一个优先级顺序：()>[]>*，所以：

(*p)[n]：根据优先级，先看括号内，则p是一个指针，这个指针指向一个一维数组，数组长度为n，这是“数组的指针”，即数组指针；

*p[n]根据优先级，先看[],则p是一个数组，再结合，这个数组的元素是指针类型，共n个元素，这是指针的数组，即指针数组

根据上面两个分析，可以看出，p是什么，则词组的中心词就是什么，即数组“指针”和指针“数组”。

图解：

![38](F:\学习专用\note\pic\38.png)

指针数组：数组元素全为指针char *pl[5]={"你","好",'"世","界"}

数组指针：指向[数组](https://baike.baidu.com/item/%E6%95%B0%E7%BB%84)首元素的地址的[指针](https://baike.baidu.com/item/%E6%8C%87%E9%92%88)，其本质为指针（这个指针存放的是数组首地址的地址，相当于2级指针，这个指针不可移动） char (*num)[3]={"你","好"}

~~~c++
#include<iostream>
using namespace std;

void main()
{
	int a[10] = {1,2,4,5,6,7};
	int b;
	//定义一个数组类型
	typedef int(Array)[10];
	Array arr;
	arr[0] = 2;
	cout << arr[0] << endl;
	//定义一个数组指针类型
	typedef int (*Array1)[10];//Array1为一个数组指针类型
	Array1 array;//定义一个数组指针变量，本质为一个指针
	array = &a;
	(*array)[0] = 20;//通过指针，修改a[0]的值，注意加括号
	(*array)[1] = 30;
	(*array)[2] = 10;
	(*array)[3] = 50;
	cout << a[0] << endl;
	cout << a[1] << endl;
	cout << (*array)[4] << endl;//打印a[4]的值
	//定义一个指向数组类型的指针 数组指针
	int (*Array2)[10];
	Array2 = &a;
	(*Array2)[0] = 30;
	cout << a[0] << endl;
	//定义一个指针数组
	int c[3][4] = { {1,2,3},{2},{3} };
	int *array3[10];//本质为一个含有10个指针的数组
	for (int i=0;i<3;i++)
	{
		array3[i] = c[i];//array3[0],array3[1],array3[2],分别指向c[0],c[1],c[2]的地址,也就是&[0][0],&c[1][0],&c[2][0]
	*array3[1] = 5;//修改c[1][0]的值
	cout << c[1][0]<< endl;
	system("pause");
}
~~~

## 5.指针

~~~c
int num = 10;

int *p = &num;//p是指向num的指针，或p中存放num的地址

cout <<p<<endl;//打印num的地址

cout<<*p<<endl;//打印num的值
~~~

**指向指针的指针**

int **p2= &p;//p2指向一个int型的指针

## 6.引用

只存在c++中，引用标识符以符号&开头

作用：

- 引用作为其他变量的**别名**而存在，在一些场合可以代替指针
- 引用相对于指针来说具有更好的可读性和实用性

~~~c++
int i = 10;

int &r=i;//r是一个引用，与i绑定在一起

r=20;//则i=20

cout<<&r<<&i;//输出i和r的地址相同

cout<<r<<i<<endl;//输出r和i的值均为20
i=40;
cout<<r<<i<<endl;//输出r和i的值均为40
~~~

### 6.1 引用本质探究

- 单独定义的引用时，必须初始化：说明很像一个常量
- 普通引用有自己的内存空间，很像指针所占的内存大小

~~~c++
#include<iostream> 
using namespace std;
struct t
{
	double &a;//占用4
	int &b;//占用4
};

int main() {
   cout << sizeof(t) << endl;
	system("pause");
	return 0;	
}
~~~

输出：8

**结论：**

1）引用在C++中的内部实现是一个**常指针**

​          **Type& name == Type* const name**

2）C++编译器在编译过程中使用常指针作为引用的内部实现，因此引用所占用的空间大小与指针相同。

3）从使用的角度，引用会让人误会其只是一个别名，没有自己的存储空间。这是C++为了实用性而做出的细节隐藏

~~~c++
void fun(int &a)
{
    a=90;
}
void fun(int *const a)//实现效果相同
{
    *a=90;
}
~~~

### 6.2 对指针的引用

语法：类型 &引用名=指针名;//可以理解为：（类型） &引用名=指针名，即将指针的类型当成类型*

~~~c
#include<iostream> 
using namespace std; 
int main(){
    int a=10; 
    int *ptr=&a; 
    int *&new_ptr=ptr; 
    *new_ptr=20;
    cout<<&ptr<<" "<<&new_ptr<<endl;
    cout<<*ptr<<" "<<a<<" "<<*new_ptr<<endl;
    return 0;  
}
~~~

结果：

![17](F:\学习专用\note\pic\17.png)



#### 指针引用作函数参数

~~~c++
#include<iostream>
using namespace std;

struct teacher
{
	char name[32];
	int age;
};
int fun(teacher **p)
{
	teacher *tmp = NULL;//定义tmp为指向teacher类型的指针
	if (p == NULL)
	{
		return -1;
	}
	tmp = (teacher *)malloc(sizeof(teacher));
	if (tmp == NULL)
	{
		return -2;
	}
	strcpy(tmp->name,"myteacher");
	//p是实参的地址，*p用间接方式改变实参的值
	*p = tmp;
}

int fun2(teacher* &p1)
{
	//给p1赋值相当于给main函数中的t1赋值
	p1 = (teacher *)malloc(sizeof(teacher));
	if (p1 == NULL)
	{
		return -1;
	}
	strcpy(p1->name, "myteacher");
}

void freeT(teacher *p)
{
	if (p == NULL)
	{
		return;
	}
	free(p);
}
int main() {
	//c中的二级指针
	teacher *t1 = NULL;
	fun(&t1);
	cout << t1->name << endl;
	freeT(t1);

	//c++中的引用（指针引用）
	teacher *t2 = NULL;
	fun2(t2);
	cout << t2->name << endl;
	freeT(t2);

	system("pause");
	return 0;
	
}
~~~

### 6.3 引用作函数参数

~~~c++
void swap(int &a,int &b)
{
    int tmp =a;
    a=b;
    b=tmp;
}
void main()
{
    int x=10,y=90;
    swap(x,y);
    cout<<x<<y<<endl;
}
~~~

### 6.4 函数的返回值是引用

~~~c++
//例1
#include<iostream> 
using namespace std;

int fun()
{
	int a = 10;
	return a;
}
//返回a的内存空间 返回一个a的副本10
int& fun1()
{
	int a1 = 10;
	return a1;
}

int main() {
	int a1 = fun();
	int a2 = fun1();//10
	int &a3 = fun1();//左值为引用，则将a1地址赋给a3
	cout << a1 << " " << a2 << " " << a3;//打印a3时，由于a3为引用，将打印*a3，此时栈中无a1的值（已被清理），所以打印的是地址
	system("pause");
	return 0;
}
~~~

结果：

![19](F:\学习专用\note\pic\19.png)

~~~c++
//例2
#include<iostream> 
using namespace std;

int fun()
{
	int a = 10;
	return a;
}
//返回a的内存空间 返回一个a的副本10
int& fun1()
{
    static int a1 = 10;//静态变量
    a1++;
	return a1;
}

int main() {
	int a = fun();
	int a2 = fun1();//11
	int &a3 = fun1();//左值为引用，则将a1地址赋给a3
	cout << a << " " << a2 << " " << a3<<endl;//打印a3时，由于a3为引用，将打印*a3，此时栈中含有a1的值
	fun1() = 100;//函数返回值是一个引用，并且当左值，此时左值为a1的内存地址
	cout << fun1() << endl;//打印a1的值，效果与上a3相同
	system("pause");
	return 0;	
}
~~~

结果：

![20](F:\学习专用\note\pic\21.png)

**结论：**

当函数返回值为引用时：

- 若返回栈变量，不能成为其它引用的初始值，不能作为左值使用（例1）
- 若返回静态变量或全局变量可以成为其他引用的初始值即可作为右值使用，也可作为左值使用（例2）

### 6.5 常量引用

```c++
//常引用 初始化 分为2种情况
//1）用变量 初始化 常引用
int a = 90;
const int &b = a;//用a初始化常引用
//b=20;//错，常引用，是让变量引用只读属性，不能通过b去修改a

//2）用字面量 初始化 常引用
//int &c = 20;//普通引用 引用一个字面量，字面量没有内存地址，会出错，因为引用就是给内存取多个别名
const int &c = 20;//c++编译器 会分配内存空间
```
结论：

- const int &e 相当于const int * const e
- 普通引用 相当于int *const e
- 当使用常量（字面值）对const引用进行初始化时，c++编译器会为常量分配空间，并将引用名作为这段空间的别名
- 使用字面值对const引用初始化后，将生成一个只读变量

## 7.const限定符

const对象创建后值就不能改变，c++中会将定义的const变量存入**符号表**中

**好处：**

- 指针作函数参数，可以有效的提高代码可读性，减少bug
- 清楚的分清参数的输入和输出特性

### 7.1 顶层const

指针本身为常量

~~~c
int i=0;
int *const pi = &i;//顶层const，不能改变pi的值，即指针变量本身不能被修改
pi=10;//错，不能改变pi的值
*pi=20;//对，可以改变i的值
~~~

### 7.2 底层const

指针所指的对象为常量

~~~c
const int r=32;
const int *p = &r;//int const *p同， 底层const，允许改变p的值，即指针所指向的内存空间不能被修改
p=12;//对，改变p的值
*p = 20;//错，不能改变r的值
~~~

### 7.3 特定const

~~~c++
const int r1=32;
const int *const p1 = &r1;//指针所指向的内存空间不能被修改，且指针变量本身不能被修改
p1=12;//错，不能改变p1的值
*p1 = 20;//错，不能改变r1的值
~~~

### 7.4 一般const

~~~c++
const int i=90;
int const s=80;//与上效果相同，即i和s的值不能被修改
~~~

### 7.5 const分配内存时机

可能分配存储空间,也可能不分配存储空间  

- 当const常量为全局，并且需要在其它文件中使用，会分配存储空间
- 当使用&操作符，取const常量的地址时，会分配存储空间
- 当const int &a = 10; const修饰引用时，也会分配存储空间
- **const在编译器编译时就分配内存，而不是在运行时**

~~~c++
#include<iostream>
using namespace std;

void main()
{
	int a;
	const int b=0;
	int c;
	cout << &a << " "<<&c <<"  "<< &b;
	cout << "hello" << endl;
	system("pause");
}
~~~

结果：

![16](F:\学习专用\note\pic\16.png)

### 7.6 const与#define的相同与不同

相同：

~~~c++
const int i=8;
#define i 8  //效果与上相同
~~~

不同：

- const常量是有编译器处理的，宏定义是由预处理器处理，单纯的文本替换
- const常量提供类型检查和作用域检查，即有作用域限制，而宏定义没有，宏定义在一个作用域定义后可在另一个作用域中使用

## 8.constexpr和常量表达式

### 8.1 常量表达式

值不会改变并且在编译过程就能得到计算结果的表达式。

例：字面值，用常量表达式初始化的const对象

~~~c
const int i=20;//i是一个常量表达式，字面值为常量表达式
const int j=i+1;//j是一个常量表达式
const int s=get_size();//s不是常量表达式
~~~

### 8.2 constexpr

c++新标准规定，允许将变量声明为constexpr类型对象以便编译器来验证变量的值是否是一个常量表达式。声明为constexpr变量一定是一个常量，而且必须用常量表达式初始化

~~~c
constexpr int sa=20;//20是常量表达式，初始化正确
constexpr int la=sa+19;//sa+19是常量表达式，初始化正确
constexpr int s=size();//只有当size是一个constexpr函数时，才是一条正确的语句
~~~

## 9.decltype类型指示符

c++新标准中引入decltype，作用是选择并返回操作数的数据类型。

~~~c
decltype(f()) sum = x;//sum的类型为函数f的返回类型
const int ci=0,&cj = ci;
declytpe(ci) x=0;//x的类型是const int
decltype(cj) y=x;//y的类型是const int &,y绑定到变量x
~~~

## 10.size_t类型

size_t是一些C/C++标准在stddef.h中定义的。这个类型足以用来表示对象的大小。size_t的真实类型与操作系统有关。


在**32位**架构中被普遍定义为：

typedef   unsigned int size_t;

而在**64位**架构中被定义为：

typedef  unsigned long size_t;  

**与int的不同点：**

- size_t在32位架构上是4字节，在64位架构上是8字节，在不同架构上进行编译时需要注意这个问题。而int在不同架构下都是4字节，与size_t不同；
- 且int为带符号数，size_t为无符号数。

# 二.字符串、向量和数组

## 1.string标准库

**string对象和字面值相加**

string对象可以和一个字面值相加

string s1= "hello";

string s3 = s1+"world";

两个字面值不能直接相加：如 string s4= "hello"+"world";---错误

## 2.范围for语句!!!!!

用于遍历给定序列中的每个元素并对序列中的每个元素执行某种操作

表示格式：

~~~c
for(declaration : expression)

     statement
~~~

例：

~~~c
string str("some string");

for(auto c : str)  //遍历str中的每个字符

   cout<< c << endl;//输出当前字符
~~~

例：将字符串中的所有s字符改为a---------试用于对一个字符串中的字符进行修改

~~~c
string str("some string");
for(auto &c : str)//c为引用，依次绑定str中的每一个字符（即c为啥str中的那个字符便是啥），如果字符为s则将s改为a
{ 
    if(c=='s')
        c='a';
}
cout << str <<endl;
~~~

结果：

![img](file:///E:/%E6%9C%89%E9%81%93%E7%AC%94%E8%AE%B0/zhangxiafll@163.com/e5d25f8e4f9a409998b4aa24d435c691/clipboard.png)

## 3.处理string对象中的单个字符的方法

（1.使用下标的方式：s[0]为第一个字符，s[1]为第二个字符，类推，直到s[s.size()-1]

（2.使用迭代器

## 4.vector使用

头文件：#include<vector>

**（1）定义和初始化对象--注意区别括号和大括号**

~~~c
vector<int> num;//初始状态为空

vector<vector<int>> fi;//元素为vector的vector对象

vector<string> str(3,"hello");//初始化3个string元素，每个均为字符串hello

vector<string> str2{"hello","my","world"};
等同于vector<string> str2={"hello","my","world"}；//初始化vector对象str2包含三个元素，即3个字符串
vector<int> num1(10);//num1中有10个0

vector<int> num2{10};//num2中有1个10
~~~

**（2）向vector添加元素**

使用push_back向尾端添加元素

~~~c
vector<int> nb;

for(int i=0;i<10;i++)
    nb.push_back(i);
~~~

**（3）其他操作**

vector<string> v;

v.empty;//如果v为空，则为真，否则假 v.size();//v的大小 v[n];//返回第n个元素的引用

例：	

~~~c
  vector<int> num{ 1,2,5,4,7 };

	for (auto &i : num)//引用num中的每个元素

		i *= i;

	for (auto c : num)

		cout << c << endl;
~~~



## 5.迭代器

**（1）迭代器和指针的区别：**

迭代器：

​      （1）迭代器不是指针，是类模板，表现的像指针。他只是模拟了指针的一些功能，通过重载了指针的一些操作符，->,*,++ --等封装了指针，是一个“可遍历STL（ Standard Template Library）容器内全部或部分元素”的对象， 本质是封装了原生指针，是指针概念的一种提升（lift），提供了比指针更高级的行为，相当于一种智能指针，他可以根据不同类型的数据结构来实现不同的++，--等操作；

​      （2）迭代器返回的是对象引用而不是对象的值，所以cout只能输出迭代器使用*解引用后的值而不能直接输出其自身。

​      （3）在设计模式中有一种模式叫迭代器模式，简单来说就是提供一种方法，在不需要暴露某个容器的内部表现形式情况下，使之能依次访问该容器中的各个元素，这种设计思维在STL中得到了广泛的应用，是STL的关键所在，通过迭代器，容器和算法可以有机的粘合在一起，只要对算法给予不同的迭代器，就可以对不同容器进行相同的操作。

指针：

​        指针能指向函数而迭代器不行，迭代器只能指向容器；指针是迭代器的一种。指针只能用于某些特定的容器；迭代器是指针的抽象和泛化。所以，指针满足迭代器的一切要求。

​        总之，指针和迭代器是有很大差别的，虽然他们表现的行为相似，但是本质是不一样的！一个是类模板，一个是存放一个家伙的地址的指针变量。

**（2）迭代器使用**

定义迭代器：vector<int>::iterator it;//能读写vector<int>中的元素

vector<int>::const_iterator it;//只能读元素，不能写

或使用auto方式，自动声明格式

begin（）指向第一个元素的迭代器，end（）指向最后一个元素的下一个元素

若v.begin()==v.end(),则容器为空

迭代器使用（++）运算符：指向后一个位置

cbegin()，cend()指向容器位置与begin()和end()同，但返回值为const_iterator

auto it2 = v.cbegin();//it2类型为const_iterator 

### 5.1 迭代器运算

iter + n ;//指向位置向后移动n个位置

iter - n ;//指向位置向前移动n个位置

例:使用迭代器完成二分搜索

~~~c
	vector<int> number1 = { 12,45,89,120,230,450 };
	int find_number =45;
	vector<int>::iterator start, end,mid;
	start = number1.begin();
	
	end = number1.end();
	mid = number1.begin()+(end - start) / 2;
	while (mid != end && *mid != find_number)
	{
		if (find_number < *mid)
			end = mid;
		else
			start = mid + 1;
		mid = start + (end - start) / 2;
	}
	if (mid == end)
		cout << "not find" << endl;
	else
		cout << " find" <<  find_number<<endl;
~~~

## 6.数组

字符数组：

const char a1[5]="hello";//错--没有空间存放空字符

char a2[]="hello";

char a3[]={'h','e','l','l','o','\n'}；与上相等

数组声明：

~~~c
int *ptr1[10];//ptr1是一个含有10个整型指针的数组，即指针数组

int (*ptr)[10] = &arry;//ptr是一个指向含有10个整数的数组

int (&arr)[10]=arry;//arr是一个引用含有10个整数的数组

int *(&arr)[10]=arry;//arr是数组的引用，该数组含有10个指针
~~~

利用范围for，遍历数组中的所有元素：

~~~c
int array[10]={12,45,12,56,9,9845,656};
for(auto c : array)
    cout<<c<<endl;
~~~



## 7.多维数组

int ia[3][4];第一维为行，第二维为列

**多维数组初始化**

~~~c
int id[3][4] = {
{1,2,3,4},
{5,6,7,8},
{9,10,11,12}
};
int ia[3][4]={1,2,3,4,5,6,7,8,9,10,11,12};//与上表达式相同
~~~

**多维数组下标引用**

int (&row)[4]=ia[2];//把row定义成一个含有4个整数的数组的引用，绑定到ia二维数组的第3行

# 三.表达式

## 1.条件运算符

**条件运算符**

cond ? expr1 : expr2 判断cond条件是否为真，真执行expr1，假执行expr2

**嵌套条件运算符**

例：

fial_grade = (grade > 90) ? "high":(grade > 60)? "pass":"false"

## 2.break语句

break语句负责终止离他最近的while, do while , for或swich语句

## 3.continue语句

终止最近的循环中的当前迭代并立即开始下一次迭代。continue只能出现在for,  while 和 do while循环的内部

# 四.函数

## 1.形参和实参

实参是形参的初始值，与形参存在对应的关系，必须与对应形参类型匹配。

### 1. 1 形参

形参是一种自动对象（只存在于块执行期间的对象），函数开始时为形参申请存储空间，函数终止，形参也被销毁

**无形参函数**

~~~c
void func1();
void func2(void);
~~~

**有参函数**

需写出每个形参的声明符

~~~c
int fun(int a1,int a2,char a3)
~~~

## 2.局部静态对象

局部变量定义为static类型的对象，第一次定义对象后，直到程序终止才销毁。

~~~c
int func1()
{
	static int num = 0;
	return ++num;
}
int main()
{	
    for (int i=0;i < 10;i++)
		cout << func1() << endl;
    return 0;
}
~~~

结果：

![4](F:\学习专用\note\pic\4.png)

## 3.函数声明

函数只能定义一次，但可以多次声明，即重构函数

函数三要素：返回类型，函数名，形参类型

## 4.参数传递

### 4.1 传值

对形参的所有操作不影响实参，实参值不变

~~~c
void fun(int a,int b)
{
    int tmp;
    tmp=a;
    a=b;
    b=tmp;
}
int main()
{
    int a=10,b=20;
    fun(a,b);
    cout<<"a="<<a<<"b="<<b<<endl;
}
~~~

结果：

![5](F:\学习专用\note\pic\5.png)

### 4.2 传地址

指针形参指向实参的地址，可以改变实参的值

~~~c
void fun(int *a,int *b)
{
    int tmp;
    tmp=*a;
    *a=*b;
    *b=tmp;
}
int main()
{
    int x=10,y=20;
    fun(&x,&y);
    cout<<"x="<<x<<"y="<<y<<endl;
}
~~~

结果：

![6](F:\学习专用\note\pic\6.png)

### 4.3 传引用

通过形参通过引用方式绑定实参，改变实参的值

~~~c
void fun(int &a,int &b)
{
    int tmp;
    tmp=a;
    a=b;
    b=tmp;
}
int main()
{
    int x=10,y=20;
    fun(x,y);
    cout<<"x="<<x<<"y="<<y<<endl;
}
~~~

结果：

![7](F:\学习专用\note\pic\7.png)

## 5.指针或引用形参与const

可以用非常量初始化一个底层const对象，但是反过来不行；一个普通引用必须用同一类型的对象初始化

~~~c
const int &r = 43;//使用非常量初始化一个底层const
int &r1=24;//错误，不能用字面值初始化一个非常量引用
~~~

## 6.main:处理命令行选项

~~~c
int main(int argc,char *argv[]);
~~~

argr是一个整数，argv是指向char型指针的指针，即指向字符串的指针（指针数组）

脚本命令：prog -d  -o file

即argc=4，argv[0]="prog";argv[1]="d";...argv[4]=0;

## 7.含可变形参的函数

为了编写能处理不同数量实参的函数，c++11新标准提供了两种方法：

- **实参类型相同**

可以传递一个名为initializer_list的标准库类型，其提供的操作与vector类似

![8](F:\学习专用\note\pic\8.png)

例：

~~~c
void err(initializeer_list<string> li)
{
    for(auto beg=li.begin();beg!=li.end();beg++)
        cout<<*beg<<endl;
}
~~~

- **实参类型不同**

后面介绍

## 8.无返回值函数

返回void的函数不要求非的有return语句，函数的最后一句会隐式执行return，若在中间位置想提前退出，课使用return语句。

## 9.函数重载

同一个作用域内的几个函数名字相同但**形参列表不同**（类型/个数），称为重载函数（**返回值类型需相同**）

~~~
int f(int i);
int f(double j);
f(9);//调用第一个函数
f(9.22);//调用第二个函数
~~~

**注意：**

- 函数重载在本质上是相互独立的不同函数（静态链编）
- 重载函数的函数类型是不同的
- 函数返回值不能作为函数重载的依据，返回值不同，不为重载函数
- 函数重载由函数名和参数列表决定

### 9.1 重载和const形参

一个拥有顶层const的形参无法和另一个没有顶层const的形参区分开来

~~~c
Record look(Phone *);
Record look(Phone *const);//重复声明了Record look(Phone *)
Record look(Phone);
Record look(const Phone);//重复声明了Record look(Phone)
~~~

### 9.2 重载与作用域

在不同的作用域中无法重载函数名，内层作用域中的声明会隐藏外层作用域中的声明

~~~c
void f(int i);
void f(double j);
void func(int val)
{
    void f(const string c);//新作用域：隐藏了之前的f
    f("hello");//正确
    f(2);//错误：void f(int i)被隐藏
    f(2.99);//错误：void f(double j)被隐藏
    
}
~~~

正确使用方式：将一个函数的重载放同一个作用域中

~~~c
void f(int i);
void f(double j);
void f(const string c);
void func(int val)
{
    void f(const string c);//新作用域：隐藏了之前的f
    f("hello");//正确：调用void f(const string c)
    f(2);//正确：调用void f(int i)
    f(2.99);//正确：调用void f(double j)
    
}
~~~

## 10.默认实参

**注意**

1）默认参数规则，如果默认参数出现，那么默认参数右边的都必须有默认参数

如：

~~~c++
void fun(int a=9,int b=90,int c=23);//对
void fun2(int a=9,int c);//错
void fun3(int a,int b=90,int c=80);//对
~~~

2）若你填写参数，则使用你填写的参数，不写则使用默认的参数

~~~c++
string func(int i=20,int j = 10,char sa = "he");
string wind;
wind=func();//等价于func(20,10,"he")
wind=func(12);//等价于func(12,10,"he")
wind=func(2,12);//等价于func(2,12,"he")
~~~

## 11.函数匹配

确定某次调用应该选用哪个重载函数

~~~c
void f(int);
void f(int,int);
void f(double,double=3.14);
f(3.22);//将调用f(double,double=3.14)
~~~

分析：

- f(int)可行，可将double转换成int
- f(double,double=3.14)可行，第二个形参提供默认值，第一个形参为double更匹配，所以选用次重载函数

## 12.函数指针

```c++
//函数指针
//声明一个函数类型
typedef void(myFunc)(int a,int b);//函数别名创建，创建void fun(int a,int b)的别名为myFunc
//myFunc为一般类型，如int
myFunc *myfunc=NULL;//定义一个函数指针，这个指针指向函数的入口地址

//声明一个函数指针类型
typedef void(*myFunc2)(int a,intb);//声明了一个指针的数据类型
myFunc2 myfunc;//通过 函数指针类型定义了一个函数指针变量

//定义一个函数指针 变量
void （*myFunc3）(int a,int b);
```

```c++
#include<iostream>
using namespace std;

void fun(double a, double b)
{
	cout <<"a: "<< a << " b: " << b << endl;
}
void fun(int a, int b)//函数重载
{
	cout <<"a: "<< a << " b: " << b << endl;
}
void fun(int a)//函数重载
{
	cout <<"a: "<< a << endl;
}
typedef void(Func)(int a, int b);//声明一个函数类型
typedef void(*Func1)(double a, double b);//声明一个函数指针类型
void(*fun2)(int a);//定义一个函数指针变量
int main() {

	Func *myfun = NULL;//定义一个函数指针
	myfun = fun;
	myfun(1, 2);

	Func1 myfun1;//定义一个函数指针
	myfun1 = fun;
	myfun1(4.23, 5.22);//调用重载函数

	fun2 = fun;
	fun2(3);
	system("pause");
	return 0;
}
```

结果：

![22](F:\学习专用\note\pic\22.png)

### 12.1 函数指针作函数参数思想剖析

**分析：**

![01_函数指针做函数参数思想剖析](F:\学习专用\note\pic\01_函数指针做函数参数思想剖析.bmp)



~~~c++
//函数指针作函数参数思想剖析
#include<iostream>
using namespace std;
//回调函数，
int fun(int a)//子任务的实现者，函数返回值，函数参数列表必须与定义的函数指针类型相同
{
	cout << "a: " << a << endl;
	return a;
}
int fun1(int a)//子任务的实现者
{
	cout << "a*a: " << a*a << endl;
	return a*a;
}
int fun2(int a)//子任务的实现者
{
	cout << "a+a: " << a+a << endl;
	return a + a;
}
int myfun(int a)//函数扩展
{
	cout << "a的三次方 " << a*a*a << endl;
	return a*a*a;
}
typedef int(*Func1)(int a);//声明一个函数指针类型，作用：把函数返回值、函数参数列表提前约定
int  play(Func1 myfun)//框架，函数指针 做 函数形参
{
	int ret=myfun(3);//间接调用，有利于在原有的框架上进行扩展
	return ret;
}
int play2(int(*Func)(int a))//与上效果相同，使用函数指针
{
	int ret =Func(5);
	return ret;
}
//任务的调用和任务的编写可以分开,解耦合
int main() {

	play(fun);//传入函数的入口地址
	play(fun1);
	play(fun2);
	play(myfun);
	play2(myfun);
	system("pause");
	return 0;
}
~~~

### 12.2重载函数的指针

~~~c
void f(int *);
void f(double);
void (*d)(double)=f;//d指向f（double）
~~~

## 13.内联函数

函数返回值前含有inline的函数为内联函数，如：

~~~c++
inline int fun(int a)
~~~

注意：

1)内联函数声明和函数体的实现，写在一起,如下所示

~~~c++
inline int fun(int a)
{
    int b=a;
    cout<<b<<endl;
}
~~~

2）c++编译器直接将函数体插入在函数调用的地方

~~~c++
inline int fun(int a)
{
    int b=a;
    cout<<b<<endl;
}
void main()
{
    int a=10;
    fun(a);//实际上c++编译器把 int b=a;cout<<b<<endl;两句内容插入到了主函数中，没有普通函数调用时的额外开销（压栈，跳转，返回）
}
~~~

3）现代c++编译器能够进行编译优化，因此一些函数即使没有使用inline声明，也可能被编译器内联编译

4）c++中内联编译的限制：

- 不能存在任何形式的循环语句
- 不能存在过多的添加判断语句

结论：

- 内联函数在编译时直接将函数体插入函数调用的地方
- inline只是一种请求，c++编译器不一定准许这种请求
- 内联函数省去了普通函数调用时压栈，跳转和返回的开销

## 14.函数占位参数

~~~c++
void fun(int a,int b,int c)
{
    cout<<a<<b<<endl;
}
void main()
{
    fun(90,80,70);//对
    fun(90,80);//错
}
~~~

# 四. 类

基本思想：数据抽象和封装

数据抽象：一种依赖于接口和实现分离的编程技术。

## 1.构造函数

初始化类对象的数据成员，一个类可含有多个构造函数与重载函数类似

特点：

- 与类名相同
- 没有返回类型

### 1.1合成的默认构造函数

类通过一个特殊的构造函数来控制默认初始化过程，这个函数为默认构造函数，默认构造函数无需任何实参。如果类没有显式的定义构造函数，那么编译器就会隐式的定义一个默认构造函数。也称合成的默认构造函数。

初始化类的数据成员的规则：

- 如果存在类内的初始值，用它来初始化成员
- 否则，默认初始化该成员

### 1.2 定义构造函数

c++新标准中，使用默认构造函数，可以在参数列表后写上=default来要求编译器生成构造函数，如果=default在类的内部，则默认构造函数是内联的

~~~c++
class f{
    f()=default;
    f(const std::string &sa):book(sa){}
    std::string book;
}
~~~

不使用default则构造函数在类的外部

~~~c++
f::f(const std::string &sa)
{
    book(sa);
}
~~~

## 2.访问控制与封装

封装的两层含义：

- 把属性和方法进行封装
- 对属性和方法进行访问控制

使用访问说明符加强类的封装性：

- public：成员在整个程序内可被访问，类内和类外均可
- protect：成员可以被类的成员函数访问，只能在类内被访问，不能在类外访问，用在继承中
- private：成员可以被类的成员函数访问，只能在类内被访问，不能在类外访问

**说明：**

类内：指类的内部：

~~~c++
class f
{
public:
int a;
void func(int b)//1.直接在类class里面进行函数定义
{
    a=b;
}
void func2(int a);
}
void f::func2(int a)//2. 在类里面进行函数声明，在外部使用域名
{
    cout<<a<<endl;
}
~~~

### 2.1 class和struct区别

使用class和struct关键字开始类的定义

**区别：**struct和class的默认访问权限不同，包含默认成员访问说明符和默认派生访问说明符

- 使用struct关键字，定义在第一个访问说明符之前的成员是public，使用class关键字，则这些成员为private
- 默认派生访问说明符不同，class关键字默认private继承，struct默认public继承

~~~c++
//默认成员访问说明符
class func {
	   int j;//private成员
	public:
		double num;
	};
	struct func {
	   int i;//public成员
	public:
		double num;
	};
//默认派生访问说明符
	struct child1:func{};//默认public继承
	class child2:func{};//默认private继承
~~~

### 2.2 封装的优点

- 确保用户代码不会无意间破坏封装对象的状态
- 被封装的类的具体实现细节可以随时改变，无须调整用户级别的代码

## 3.友元

类可以允许其他类或者函数访问他的非公有成员，方法是令其他类或者函数成为他的友元，添加一条以friend关键字开始的函数声明的语句。

### 友元函数

~~~c++
#include<iostream>
using namespace std;

class Test 
{
	//1. 声明友元函数 前加friend 友元函数声明的位置与访问限制符无关
	friend void modify(Test *p,int c);//2.函数modify 是类Test的好朋友
public:
	Test (int);
	void getT()
	{
		cout << a << endl;
	}
private:
	int a;
};

Test ::Test (int a)
{
	this->a = a;
}

void modify(Test *p,int c)
{
	p->a = c;
}
void main()
{
	Test t(90);
	cout << "a的初始值为： ";
	t.getT();
	cout << "友元函数修改类中私有成员a后a的值为：";
	modify(&t,100);
	t.getT();
	system("pause");
}
~~~

结果：

![30](F:\学习专用\note\pic\30.png)

### 友元类

若B类是A类的友元类，则B类的所有成员函数都是A类的友元函数



~~~c++
	class B {
	   friend class A;//声明友元类,A为B的友元类
	public:
		int num;
	};
	class A{
	public:
		void f(B *b,int a)
        {
            b->num=a;
        }；//对：A是B的友元类
	};
~~~

**注意：**

友元关系不能继承

## 4.委托构造函数

c++新标准扩展了构造函数初始值的功能，一个委托构造函数使用它所属类的其他构造函数执行它自己的初始化过程。

~~~c++
class sale{
public:
    sale(std::string s,int cn,double p):book(s),sold(cn),rev(p){}//非委托构造函数使用对应的实参初始化成员
    sale():sale("",0,0){}//构造函数委托给另一个构造函数初始化成员
    sale(std::string s):sale(s,0,0){}//构造函数委托给另一个构造函数初始化成员
}
~~~

## 5.聚合类

用户可以直接访问成员，聚合类的特点：

- 所有成员都是public
- 没有定义任何构造函数
- 没有类内初始值
- 没有基类，没有virtual函数

例：

~~~c++
struct data{
    int val;
    string s;
}
data da={0,"ame"};//初始化聚合类数据成员
~~~

## 6.类的静态成员

### 6.1 声明静态成员

在成员的声明前加关键字static使得其与类关联在一起

- 静态**数据成员**的类型可以是常量、引用、指针、类类型
- 静态**成员函数**不与任何对象绑定一起，不包含this指针，不能被声明为const的

~~~c++
class accout{
public:
    static void rate(double);//静态成员函数
private:
    int a;//普通变量
    static double interestrate;//静态成员变量
}
~~~

### 6.2 定义静态成员

~~~c
void account::rate(double newrate)//方式1
{
    interestrate=newrate;//对
    a=10;//错，不能对非静态成员进行操作，因为c++编译器不知道所操作的对象是哪一个
    //accout a1.a2; a1.rate(2);a2.rate(3);
}
double accout::interestrate=10;//方式2
void main()
{
    accout::rate(2.12);//调用静态成员函数的方式1
    accout a1;
    a1.rate(2.22);//调用静态成员函数的方式2
}
~~~

一般来说，不能在类的内部初始化静态成员，一个静态数据成员只能被定义一次。类似于全局变量，一旦定义将一直存在于程序的整个生命周期。

静态函数中，不能使用 普通成员函数和变量

**分析：**

![07_静态成员变量和静态成员函数分析](F:\学习专用\note\pic\07_静态成员变量和静态成员函数分析.bmp)

### 6.3 静态成员的类内初始化

特殊情况：静态成员const整数类型的类内初始化

要求：

- 静态成员必须是字面值常量类型的constexpr
- 初始值必须是常量表达式

~~~c++
class accout{
public:
    static void rate(double);
private:
    static constexpr double interestrate = 30;
}
~~~

### 6.4 静态变量的特殊作用

静态变量独立于任何对象，所有某些非静态数据成员可能非法的场合，静态成员能正常使用

#### 6.4.1 静态变量可以是不完成类型

~~~c
class bar{
    static bar men;//对，静态指针可以是不完全类型
    bar *men2;//对，指针成员可以是不完全类型
    bar men3;//错误，数据成员必须是完全类型
}
~~~

#### 6.4.2 使用静态成员作为默认实参

~~~c
class accout{
public:
    static void rate(double i=interestrate);
private:
    static double interestrate;
}
~~~

# 五.面向对象的程序设计

##  c++面向对象模型初探

~~~c++
#include "iostream"

using namespace std;

class C1
{
public:
	int i;  //4
	int j; //4
	int k;  //4
protected:
private:
}; //12

class C2
{
public:
	int i;
	int j;
	int k;

	static int m; //4
public:
	int getK() const { return k; } //4
	void setK(int val) { k = val; }  //4

protected:
private:
}; //12

struct S1
{
	int i;
	int j;
	int k;
}; //12

struct S2
{
	int i;
	int j;
	int k;
	static int m;
}; //12

int main()
{
	printf("c1:%d \n", sizeof(C1));
	printf("c2:%d \n", sizeof(C2));
	printf("s1:%d \n", sizeof(S1));
	printf("s2:%d \n", sizeof(S2));

	system("pause");
}

~~~

**结果：**

![28](F:\学习专用\note\pic\28.png)

**结论：**

C++类对象中的成员变量和成员函数是分开存储的

- 普通成员变量：存储于对象中，与struct变量有相同的内存布局和字节对齐方式
- 静态成员变量：存储于全局数据区中

###  C++编译器对类的内部处理 

![29](F:\学习专用\note\pic\29.png)

结论：

1. C++类对象中的成员变量和成员函数是分开存储的。C语言中的内存四区模型仍然有效！
2. C++中类的普通成员函数都隐式包含一个指向当前对象的this指针。
3. 静态成员函数与普通成员函数的区别

- 静态成员函数不包含指向具体对象的指针
- 普通成员函数包含一个指向具体对象的指针

### this指针

~~~c++
#include "iostream"

using namespace std;

class C1
{
public:
	C1(int a)//==>C1(C1 * const this,int a)
	{
		i = a;//this->i=a;
	}
	void print()
	{
		cout << i << endl;
	}
    void fun(int x,int y) const //void fun(const C1 *const this,int x,int y)
    {
        i=x;//报错,this->i=x;const修饰的是this指针所指向的内存空间
    }
protected:
private:
	int i;
}; 
int main()
{
	C1 c1(10);
	c1.print();//==》c1.print(&c1)
	system("pause");
}
~~~

### 三大特性：

封装，继承，多态

### 虚函数：

基类希望他的派生类各自定义适合自身的版本，此时基类就将这些函数声明为虚函数

### **派生类中的虚函数：**

一个派生类的函数如果覆盖了某个继承而来的虚函数，则它的形参类型必须与被覆盖的基类函数完全一致，返回类型也必须与基类函数匹配。

```c++
class que{
public:
    virtual double func(string str) const;//const表示这个函数不能修改类里面成员变量的值
    virtual double func2(int) const;
}
```

### **override：**

c++中新标准允许派生类显式的注明他将使用哪个成员函数改写基类的虚函数

注意：使用override的好处：

- 使得程序员的意图更加清晰
- 让编译器发现一些错误（未覆盖虚函数则报错）

```c++
class que1::public que{
public:
     double func(string str) const override;//正确
     double func2(double) const override;//错：que中无形如func2（double）的虚函数
     double func3(string) const override；//错：que中无func3虚函数     
}
```

### **final:**

- c++新标准提供一种**防止继承发生**的方法，即在类名后面跟一个关键字**final**：

```c++
class deve final { };//deve不能作为基类
```

- 在某个函数后加final，则派生类无法覆盖改函数

~~~c++
class que{
public:
    virtual double func2(int) const final;
}
class qu:public que{
public:
     double func2(int) const;//错误，无法覆盖func2
}
~~~

### **动态绑定**：

使用动态绑定能用一段代码同时处理基类和派生类的对象。函数的运行版本由实参决定，即在运行时选择函数的版本，所以动态绑定又称为运行时绑定。**实现多态的技术**

**注意：**动态绑定只有当我们通过指针或引用调用虚函数时才会发生

~~~c++
#include<iostream>
using namespace std;

//动态绑定
namespace test1 {
	class func {
	public:
		virtual double print(double) const;//虚函数
	};
	double test1::func::print(double x) const
	{
		return x;
	
	}

	class child_func :public func {
	public:
		double print(double) const override;
	};
	double child_func::print(double x) const
	{
		return x*5;

	}
	void f(const func &item, double n)
	{
		cout << item.print(n) << endl;
	}

}
int main()
{
	//动态绑定
	double i = 2.134;
	test1::func func1;
	test1::child_func child_func1;
    test1::f(func1,i);//func1的类型是func，调用func的print
	test1::f(child_func1, i);//child_func1的类型是child_func，调用child_func的print

}
~~~

## 1.继承和派生

| 基类成员   | 基类private成员 |        | 基类public成员 |        |
| ---------- | --------------- | ------ | -------------- | ------ |
| 派生方式   | private         | public | private        | public |
| 派生类成员 | 不可见          | 不可见 | 可见           | 可见   |
| 外部函数   | 不可见          | 不可见 | 不可见         | 可见   |

| 派生方式      | 基类的public成员  | 基类的protected成员 | 基类的private成员 | 派生方式引起的访问属性变化概括                 |
| ------------- | ----------------- | ------------------- | ----------------- | ---------------------------------------------- |
| private派生   | 变为private成员   | 变为private成员     | 不可见            | 基类中的非私有成员都成为派生类中的私有成员     |
| protected派生 | 变为protected成员 | 变为protected成员   | 不可见            | 基类中的非私有成员都成为派生类的保护成员       |
| public派生    | 仍为public成员    | 仍为protected成员   | 不可见            | 基类中的非私有成员在派生类中的访问属性保持不变 |

**类之间的关系**

1.组合关系

例：

~~~c++
class A
{
public:
     A(int a)
    {
        m_a=a;
    }
private:
	int m_a;
};
class B
{
    B(int a):a(2)//初始化话组合对象
    {
        m_c=a;
    }
private:
	int m_c;
	A a;//组合关系
};
~~~

2.继承关系

**派生和继承的区别：**

继承是从子类的角度讲的
派生是从基类的角度讲的

其实是一个意思

**三看原则**：

c++中继承方式会影响子类的对外访问属性

判断某一句话，能否被访问

- 看语句在子类的内部还是外部
- 看子类如何从父类继承（public,protected,private）
- 看父类的访问级别（public,protected,private）

~~~c++
//派生类访问控制的结论
//1. protected修饰的成员变量和成员函数是为了在家族中使用，是为了继承
//2. 项目开发中 一般用public继承
#include<iostream>
using namespace std;
class parent
{
public:
	int m_a;
protected:
	int m_b;
private:
	int m_c;
};
//公有继承
class child:public parent
{
public:
	void fun()
	{
		m_a = 1;//ok public
		m_b = 2;//ok protected
		//m_c = 3;//err 不可见
	}
};
//保护继承
class child2 :protected parent
{
public:
	void fun()
	{
		m_a = 1;//ok protected
		m_b = 2;//ok private
	    //m_c = 3;//err 不可见
	}
};
//私有继承
class child3 :private parent
{
public:
	void fun()
	{
		m_a = 1;//ok private
		m_b = 2;//ok private 
		//m_c = 3;//err 不可见
	}
};
void main()
{
	child c1;
	c1.m_a = 10;//ok 
	//c1.m_b = 20;//err
	//c1.m_c = 30;//err

	child2 c2;
	//c2.m_a = 10;//err
	//c2.m_b = 20;//err
	//c2.m_c = 30;//err

	child3 c3;
	//c3.m_a = 10;//err
	//c3.m_b = 20;//err
	//c3.m_c = 30;//err
	system("pause");
}
~~~

### 1.1 继承中的构造与析构

#### 1.1.1 赋值兼容性原则

~~~c++
//赋值兼容性原则
//子类对象可以当作父类对象使用
//子类对象可以直接赋值给父类对象
//子类对象可以直接初始化给父类对象
//父类指针可以直接指向子类对象
//父类引用可以直接引用子类对象
#include<iostream>
using namespace std;
class parent
{
public:
	int m_a;
	void printP()
	{
		cout << "我是爹" << endl;
	}
private:
	int m_c;
};
//公有继承
class child:public parent
{
public:
	void printC()
	{
		cout << "我是儿子" << endl;
	}
};
void whotoprint(parent *p)
{
	p->printP();
}
void whotoprint2(parent &p)
{
	p.printP();
}
void main()
{
	//赋值兼容性原则
	//第一层
	//1. 基类指针（引用）指向子类对象
	child c1;
	parent p;
	parent *p1=NULL;
	p1 = &c1;
	p1->printP();
	//2. 指针做函数参数
	whotoprint(&p);
	whotoprint(&c1);
	//3.引用做函数参数
	whotoprint2(p);
	whotoprint2(c1);
	//第二层
	//可以让子类对象 初始化父类对象
	parent p2 = c1;
	system("pause");
}
~~~

#### 1.1.2 继承中构造析构的调用顺序

~~~c++
//继承中构造 析构的调用顺序
//先调用父类的构造，再子类，当父类的构造函数有参数时，需要在子类的初始化列表中显式调用
//先调用子类的析构 再父类

#include<iostream>
using namespace std;
class parent
{
public:
	parent(int a,int b)
	{
		m_a = a;
		m_b = b;
		cout << "父类构造函数" << endl;
	}
	~parent()
	{
		cout << "父类析构函数" << endl;
	}
private:
	int m_a;
	int m_b;
};

class child :public parent
{
public:
	child(int a,int b,int c):parent(a,b)//使用初始化列表初始化父类
	{
		m_c = c;
		cout << "子类构造函数" << endl;
	}
	~child()
	{
		cout << "子类析构函数" << endl;
	}
private:
	int m_c;
};
void play()
{
	child c(1, 2, 3);
}

void main()
{
	play();
	system("pause");
}
~~~

#### 1.1.3 构造与组合混搭下，构造和析构的调用原则

~~~c++
//构造与组合混搭下，构造和析构的调用原则
//先调用父类构造，再组合成员变量，最后构造自己
//先析构自己，再析构组合成员，最后析构父类

#include<iostream>
using namespace std;

class grandparent
{
public:
	grandparent(int a, int b)
	{
		m_a = a;
		m_b = b;
		cout << "grandparent构造函数" << endl;
	}
	~grandparent()
	{
		cout << "grandparent析构函数" << endl;
	}
private:
	int m_a;
	int m_b;
};

class parent:public grandparent
{
public:
	parent(int a):grandparent(1,2)
	{
		m_c = a;
		cout << "父类构造函数" << endl;
	}
	~parent()
	{
		cout << "父类析构函数" << endl;
	}
private:
	int m_c;
};

class child :public parent
{
public:
	child(char *p):parent(1),g1(2,3),g2(4,5)
	{
		cout << "子类构造函数" << endl;
	}
	~child()
	{
		cout << "子类析构函数" << endl;
	}
private:
	char *m_p;
	grandparent g1;
	grandparent g2;
};
void play()
{
	child c("aa");
}

void main()
{
	play();
	system("pause");
}
~~~

#### 1.1.4 继承中的同名变量和函数

~~~c++
//
#include<iostream>
using namespace std;

class parent 
{
public:
	parent(int a,int b) 
	{
		m_a = a;
		m_b = b;
		cout << "父类构造函数" << endl;
	}
	~parent()
	{
		cout << "父类析构函数" << endl;
	}
	void print()
	{
		cout << " 父类成员函数" << endl;
	}
	int m_a;
	int m_b;
};

class child :public parent
{
public:
	child(int a,int b) :parent(a,b)
	{
		m_b = a;
		m_c = b;
		cout << "子类构造函数" << endl;
	}
	void print()
	{
		cout << "子类成员函数" << endl;
	}
	~child()
	{
		cout << "子类析构函数" << endl;
	}
	int m_b;
	int m_c;

};
void play()
{
	child c(1, 2);
	cout << c.m_b << endl;//默认打印子类的m_b
	cout << c.m_a << endl;
	c.print();//默认使用子类成员函数
	cout << c.parent::m_b << endl;//打印父类的m_b，需要加父类的域名
	
}
void main()
{
	play();
	system("pause");
}
~~~

### 1.2继承与静态成员

~~~c++
class Base{
public:
   static void st();
};
class deve:public Base{
    void f();
};
void deve::f()
{
    Base::st();//对：通过Base定义了st
    deve::st();//对：通过deve继承了st
    st();//对：通过this对象访问
}
~~~

~~~c++
//1. static 关键字 遵守 派生类的访问控制规则
//2. int parent::l = 10;不是简单的赋值 而是告诉c++编译器 分配内存 不然在继承中使用了l会报错
#include<iostream>
using namespace std;

class parent
{
public:
	static int l;
	parent(int a, int b)
	{
		m_a = a;
		m_b = b;
		cout << "父类构造函数" << endl;
	}
	~parent()
	{
		cout << "父类析构函数" << endl;
	}
	void print()
	{
		cout << " 父类成员函数" << endl;
	}
	int m_a;
	int m_b;
};
int parent::l = 10;
class child :public parent
{
public:
	child(int a, int b) :parent(a, b)
	{
		m_b = a;
		m_c = b;
		cout << "子类构造函数" << endl;
	}
	void print()
	{
		cout << "子类成员函数" << endl;
	}
	~child()
	{
		cout << "子类析构函数" << endl;
	}
	int m_b;
	int m_c;

};
void play()
{
	child c(1, 2);
}

void main()
{
	play();
	system("pause");
}
~~~

### 1.3 多继承

**说明：**

工程开发中真正意义上的多继承是几乎不被使用的，原因：

- 多继承带来的代码复杂性远多于其带来的便利
- 多继承对代码维护上的影响是灾难性的

![35](F:\学习专用\note\pic\35.png)

~~~c++
#include<iostream>
using namespace std;

class A
{
public:
	int a;
}；
class B1:virtual public A//虚继承
{
public:
	int b1;
};
class B2 :virtual public A//虚继承，将a承递给C
{
public:
	int b2;
};
class C :public B1,public B2
{
public:
	int c;
};
void main()
{
	C c;
	c.b1 = 10;
	c.b2 = 20;
	c.c = 30;
	c.a = 40;// 多继承中继承的二义性 使用a不知道是从B2继承还是B1继承 通过虚继承解决（含共同基类）
	system("pause");
}
~~~

#### 虚继承

如果一个派生类从多个基类派生，而这些基类又有一个共同的基类，则在对该基类中声明的名字进行访问时，可能产生二义性。

![36](F:\学习专用\note\pic\36.png)

~~~c++
#include<iostream>
using namespace std;

class A
{
public:
	int a;
};
class B1 :virtual public A//虚继承
{
public:
	int b1;
};
class B2 : public A//虚继承，将a承递给C
{
public:
	int b2;
};

void main()
{
	cout << sizeof(A) << endl;//4 int型所占内存
	cout << sizeof(B1) << endl;//12 加上虚继承后，c++编译器会给变量增加属性
	cout << sizeof(B2) << endl;//8 int a,int b2
	system("pause");
}
~~~

## 2.抽象类

含有（或者未经覆盖直接继承）纯虚函数的类是抽象类，抽象类负责定义接口，而后续的其他类可以覆盖该接口。

**注意：**不能定义抽象类的对象

### 2.1 纯虚函数

纯虚函数没有实际意义，一个纯虚函数无须定义，在函数声明语句后书写=0就可以将一个虚函数说明为纯虚函数

~~~c++
#include<iostream>
using namespace std;

class figure//抽象类
{
public:
	virtual void getArea() = 0;//纯虚函数，约定一个统一的接口，让子类使用
};
class tria:public figure
{
public:
	tria(int a=10,int b=20)
	{
		this->a = a;
		this->b = b;
	}
	void getArea()
	{
		cout << "三角形面积为： " << a*b / 2 << endl;
	}
private:
	int a;
	int b;
};
class circle :public figure
{
public:
	circle(int a = 10)
	{
		this->a = a;
	}
	void getArea()
	{
		cout << "圆形面积为： " << 3.14*a*a << endl;
	}
private:
	int a;
};
class square :public figure
{
public:
	square(int a = 10, int b = 20)
	{
		this->a = a;
		this->b = b;
	}
	void getArea()
	{
		cout << "四边形面积为： " << a*b << endl;
	}
private:
	int a;
	int b;
};
void play(figure &f)
{
	f.getArea();//多态
}
void main()
{
	//figure f;//err 抽象类不能被实例化
	circle c(5);
	tria t(3, 4);
	square s(10, 30);
	//面向抽象类编程
	play(c);//圆形面积为： 
	play(t);//三角形面积为： 
	play(s);//四边形面积为： 
	system("pause");
}
~~~

### 2.2 抽象类在多继承中的应用

~~~c++
//c++中没有接口的概念，但可以使用纯虚函数实现接口，接口类（即抽象类）只有函数原形的定义，没有任何数据的定义
#include<iostream>
using namespace std;

class Interface1//抽象类
{
public:
	virtual int add(int a, int b) = 0;
	virtual void print() = 0;
};
class Interface2//抽象类
{
public:
	virtual int mult(int a, int b) = 0;
	virtual int add(int a, int b,int c) = 0;
	virtual void print() = 0;
};
class parent
{
private:
	int a;
};
class child:public parent,public Interface1,public Interface2//多继承
{
public:
	virtual int add(int a, int b)
	{
		cout << "child add(int,int)" << endl;
		return a + b;
	}
	virtual void print()
	{
		cout << "child print" << endl;
	}
	virtual int mult(int a, int b) 
	{
		cout << "child mult(int,int)" << endl;
		return a*b;
	}
	virtual int add(int a, int b, int c)
	{
		cout << "child add(int,int,int)" << endl;
		return a + b + c;
	}

private:
	int a;
	int b;
};

void main()
{
	child c;
	Interface1 *i1 = &c;
	i1->add(1, 2);//child add(int,int)
	Interface2 *i2 = &c;
	i2->add(1, 2,3);//child add(int,int,int)
	system("pause");
}
~~~

### 2.3 重构

在上述例子中，func继承体系中增加func1类是重构的典型例子。

重构是负责重新设计类的体系以便**将操作和数据从一个类移动到另一个类中**。

### 2.4 面向抽象类编程

~~~c++
//计算程序员( programmer )工资
#include<iostream>
using namespace std;

class programmer
{
public:
	virtual void money() = 0;//提供接口
};
class junior_programmer :public programmer
{
public:
	junior_programmer(int a)
	{
		this->a = a;
	}
	void money()
	{
		cout << 100 * a << endl;
	}
private:

	int a;
};
class mid_programmer :public programmer
{
public:
	mid_programmer(int a)
	{
		this->a = a;
	}
	void money()
	{
		cout << 200 * a << endl;
	}
private:
	int a;
};
class adv_programmer :public programmer
{
public:
	adv_programmer(int a)
	{
		this->a = a;
	}
	void money()
	{
		cout << 300 * a << endl;
	}
private:
	int a;
};
void play(programmer *p)
{
	p->money();
}
void main()
{
	junior_programmer jum(30);
	mid_programmer midm(20);
	adv_programmer adm(25);
	play(&jum);
	play(&midm);
	play(&adm);
	system("pause");
}
~~~



## 3.访问控制和继承

### 受保护的成员

类使用protected关键字来声明那些它希望与**派生类**分享但不想被其他公共访问使用的成员。

**特点：**

- 和私有成员类似，受保护的成员对于类的用户来说是不可访问的
- 和公有成员类似，受保护的成员对于派生类的成员和友元来说是可访问的
- 派生类的成员或友元只能通过派生类对象来访问基类的受保护成员，派生类对于一个基类对象中的受保护成员没有访问权

## 4.虚函数与作用域

~~~c++
class base{
public:
  virtual int fun();//虚函数
}
class c1:public base{
public:
    int fun(int);//与虚函数fun()不一致，隐藏基类的fun()
    virtual int fun2();//虚函数fun2（）
}
class c2:public c1{
public:
    int fun(int);//一个非虚函数，隐藏了c1的fun(int)
    int fun();//覆盖了base的虚函数fun()
    int fun2();//覆盖了c1的虚函数fun2()
}
~~~

### 通过基类调用隐藏的虚函数

~~~c++
base bo1;c1 bo2;c2 bo3;
base *ba1 = &bo1,*ba2=&bo2,*ba3=&bo3;
ba1->fun();//虚调用，调用base::fun()
ba2->fun();//虚调用，调用base::fun()
ba3->fun();//虚调用，调用c2::fun()
ba2->fun2();//静态类型和动态类型不一致，一个对象、引用或指针的静态类型决定该对像的哪些成员可见，静态类型base *，动态类型为c1，所以base类可见，c1不可见，而base中无fun2，语句错误
c1 *ca1 = &bo2;c2 *ca2=&bo3;
ca1->fun2();//虚调用，调用c1::fun2()
ca2->fun2();//虚调用，调用c2::fun2()
~~~

## 5.构造函数与拷贝控制

在基类中将析构函数定义成虚函数以确保执行正确的析构版本

~~~c++
class base{
public:
   virtual ~base()=default;//动态绑定析构函数
}
base *item=new base;//静态类型与动态类型一致
delete item;//调用base析构函数
item=new c1;//静态类型与动态类型不一致（参考前一段代码）
delete c1;//调用c1的析构函数
~~~

## 6.构造函数与析构函数

构造函数作用：为类中的成员进行初始化

析构函数作用：释放内存，对象生命周期结束后被自动调用

#### **构造函数分类**

-  无参构造函数
- 有参构造函数
- 拷贝构造函数（赋值构造函数）--若类中没有拷贝构造函数，c++编译器会自动提供一个默认的拷贝构造函数
- 默认构造函数（类中没有构造函数，c++编译器会自动提供一个默认构造函数）

~~~c++
#include<iostream>
using namespace std;

class point
{
public:
	int p_a;
	int p_b;
public:
	point()
	{
		p_a = 0;
		p_b = 0;
		cout << "无参构造函数" << endl;
	}
	point(int x, int y)
	{
		p_a = x;
		p_b = y;
		cout << "有参构造函数" << endl;
	}
	point(int x)
	{
		p_a = x;
		p_b = 0;
		cout << "有参构造函数" << endl;
	}
	point(const point& obj)//拷贝构造函数 作用：用一个对象初始化另一个对象
	{
		cout << "拷贝构造函数" << endl;
		p_a = obj.p_a;

	}
	void pointp()
	{
		cout << "p_a: " << p_a << endl;
	}
	~point()
	{
		cout << "析构函数，释放内存空间" << endl;
	}
};
void f(point p)
{
	cout << p.p_a << endl;
}
void play()
{
	point c5(1, 2);
	point c6 = c5;
	f(c6);//c实参初始化形参p,会调用构造函数
}
//f2函数返回一个元素
//返回的是一个新的匿名对象（所以会调回匿名对象类的拷贝构造函数）
point f2()
{
	point c7(2, 3);
	return c7;
}
void play2()
{
	point m = f2();//用匿名对象初始化m，此时c++编译器直接把匿名对象转成m（扶正）
				   //匿名函数去留：1. 匿名函数被扶正
	cout << "匿名函数去留：1. 匿名函数被扶正" << endl;
	cout << m.p_a << endl;
}
void play3()
{
	point m2(1, 2);
	m2 = f2();//用匿名对象 赋值给另外一个同类型的对象，匿名函数被直接析构，
			  //匿名函数去留：2. 匿名函数被直接析构
	cout << "匿名函数去留：2. 匿名函数被直接析构" << endl;
	cout << m2.p_a << endl;
}
int main() {
	//调用无参构造函数
	point p;

	//调用有参构造函数
	//1. 括号法  c++编译器自动调用构造函数
	point p2(11, 55);
	//2. 等号法 c++编译器自动调用构造函数
	//调用有一个参数的构造函数
	point p4 = (1, 2, 3);//c++对等号功能增强，括号中取最后一个数进行赋值，与point p4 = 3同义;
	point p1 = point(12, 45);//调用有两个参数的构造函数
	//3. 直接调用构造函数 手动的调用构造函数
	point p5 = point(2, 3);//产生一个匿名对象，后将匿名对象直接扶正为p5 

	//调用拷贝构造函数
	//1. 
	point c1(12, 23);
	point c2 = c1;//用c1来初始化c2
	c2.pointp();
	//2.
	point c3(10, 20);
	point c4(c3);//用c3来初始化c4
	c3.pointp();
	//3.函数形参需调用拷贝构造函数
	play();//调用三次析构函数，释放内存先后顺序为，匿名对象-c6-c5
	//4.函数返回值调用拷贝构造函数
	play2();//调用两次析构函数，一次释放c7内存，一次释放匿名对象内存
	//关于匿名函数的去留
	play3();//调用三次析构函数，一次释放c7内存，一次释放匿名对象内存，一次释放m2的内存
	system("pause");
	return 0;
}
~~~

结果：

![24](F:\学习专用\note\pic\24.png)

匿名对象的去留：

![25](F:\学习专用\note\pic\25.bmp)

#### **构造函数使用规则**

当类中定义了构造函数时，则c++编译器不会提供默认的构造函数。**即类中写了构造函数，就必须用**

#### 构造函数初始化列表

~~~c++
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
using namespace std;

class Test
{
private:
	int m_a;
public:
	Test(int a)
	{
		m_a = a;
		cout << "Test的构造函数: " << m_a <<endl;
	}
	~Test()
	{
		cout << "Test的析构函数: " << m_a << endl;
	}

};
class Test2
{
public:
	Test2(int a, int b, int c, int d) :t1(c), t2(d),c(d)//初始化列表
	{
		n_a = a;
		n_b = b;
		cout << "Test2的构造函数" << endl;
	}
	~Test2()
	{
		cout << "Test2的析构函数 "<< endl;
	}
private:
	int n_a;
	int n_b;
	Test t2;
	Test t1;
    const int c;

};
//1. 先执行 被组合对象的构造函数
//2. 被组合对象的构造顺序 与定义顺序有关 与初始化列表的顺序无关
//3.析构函数：与构造函数的调用顺序相反
//4.初始化列表用来给const成员赋值
void testplay()
{
	Test2 t(1, 2, 3, 4);
}
int main() {
	testplay();
	system("pause");
	return 0;
}
~~~

结果：

![27](F:\学习专用\note\pic\27.png)

#### 构造函数总结

- 构造函数是c++中用于初始化对象状态的特殊函数
- 构造函数在对象创建时自动被调用
- 构造函数和普通成员函数都遵循重载规则
- 拷贝构造函数是对象正确初始化的重要保证
- 必要的时候，必须手工编写拷贝构造函数

## 7.深拷贝和浅拷贝

### 浅拷贝

默认拷贝构造函数可以完成对象的数据成员值简单的复制，对象的数据资源是由指针指示的堆时，默认拷贝构造函数仅作指针值复制，即拷贝的对象中成员也指向了相同的内存空间，析构时会出现对同一内存空间的两次内存释放，导致出错

![26](F:\学习专用\note\pic\26.bmp)

### 深拷贝

深拷贝，重新为对象成员分配新的内存空间，上图蓝色线条所示，手动编写拷贝构造函数代码，解决浅拷贝出现的问题

~~~c++
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
using namespace std;

class Test
{
public:
	char *p_str;
	int p_len;
public:
	Test(const char *p)
	{
		p_len = strlen(p);
		p_str = (char *)malloc(p_len+1);//注意+1 strlen计算的长度不包含 '\0'
		strcpy(p_str, p);
	}
	Test(const Test& obj)//深拷贝，重新为对象分配新的内存空间，上图蓝色线条所示，手动编写拷贝构造函数代码，解决浅拷贝出现的问题
	{
		p_len = obj.p_len;
		p_str = (char *)malloc(p_len + 1);//注意+1 strlen计算的长度不包含 '\0'
		strcpy(p_str, obj.p_str);
	}
	~Test()
	{
		if (p_str != NULL)
		{
			free(p_str);
			p_str = NULL;
			p_len = 0;
		}
	}
	void print()
	{
		cout << p_str << endl;
	}
};

int main() {

	Test p("zxfll");
	p.print();
	Test t1 = p;
	t1.print();
    Test t2;
    t2=t1;//等号操作符，默认的为浅拷贝，分析图如下所示
	system("pause");
	return 0;
}
~~~

结果：

![25](F:\学习专用\note\pic\25.png)

**等号操作符图示：**

蓝色线为obj3本来分配的内存，红色线为等号操作符，浅拷贝后的修改的

![04_默认的等号赋值操作也是浅拷贝](F:\学习专用\note\pic\04_默认的等号赋值操作也是浅拷贝.bmp)

## 8.多态

同一操作作用于不同的对象，可以有不同的解释，产生不同的执行结果，这就是多态性。简单的说:就是用基类的指针指向子类的对象。

**多态意义：**

多态是设计模式的基础，多态是框架的基础

**多态的分类：**

编译时多态：指函数的重载

运行时多态：运用动态绑定

**多态理论基础：**

静态联编和动态联编

1、联编是指一个程序模块、代码之间互相关联的过程。

2、静态联编（static binding），是程序的匹配、连接在编译阶段实现，也称为早期匹配。 重载函数使用静态联编。

3、动态联编是指程序联编推迟到运行时进行，所以又称为晚期联编（迟绑定）。即动态绑定

switch 语句和 if 语句是动态联编的例子。

~~~c++
//多态成立的三个条件：
//1. 要有继承
//2. 要有虚函数重写
//3. 用父类指针（父类引用）指向子类对象
#include<iostream>
using namespace std;

class Fighter
{
public:
	virtual int energy()
	{
		return 20;
	}
};
class Fighter_updata :public Fighter
{
public:
	int energy() override
	{
		return 30;
	}
};

class Enemy
{
public:
	int energy()
	{
		return 25;
	}
};
//多态的使用
void pk(Fighter *f1, Enemy *e)//基类的指针指向子类的对象
{
	//不写virtual关键字 为静态联编，c++编译器根据Fighter类型，去执行这个类型的energy函数，在编译器编译阶段就已经决定了函数的调用
	//动态联编（迟绑定）：在运行时，根据具体的对象，执行不同对象的函数，表现成多态
	if (f1->energy() > e->energy())
	{
		cout << "Fighter win" << endl;
	}
	else
	{
		cout << "Enemy win" << endl;
	}
}
void main()
{
	Fighter f1;
	Fighter_updata f2;
	Enemy e;
	pk(&f1, &e);//Enemy win 传入父类引用
	pk(&f2, &e);//Fighter win 传入子类引用
	system("pause");
}
~~~

#### 8.1 虚析构函数

~~~c++
#include<iostream>
using namespace std;

class A
{
public:
	int a;
	A()
	{
		a = 10;
		cout << "A构造" << endl;
	}
	virtual ~A()//虚析构函数
	{
		cout << "A析构" << endl;
	}
};
class B:public A
{
public:
	int b;
	B()
	{
		b= 10;
		cout << "B构造" << endl;
	}
	virtual ~B()//虚析构函数
	{
		cout << "B析构" << endl;
	}
};

class C:public B
{
public:
	int c;
	C()
	{
		c = 10;
		cout << "C构造" << endl;
	}
	~C()
	{
		cout << "C析构" << endl;
	}
};
//父类构造函数 不加virtual,只执行父类的析构函数

//父类虚析构函数，通过父类指针 把所有的子类对象的析构函数 都执行一编
//通过父类指针 释放所有子类资源（虚析构函数的作用）
void play(A *a)
{
	delete a;//多态
}
void main()
{
	C *c = new C;
	play(c);
	system("pause");
}
~~~

#### 8.2 重写，重载，重定义理解（面试）

函数重载：

- 必须写在同一个类中
- 子类无法重载父类的函数，父类同名函数将被覆盖
- 重载是在编译期间根据参数类型和个数决定函数调用

函数重写：包括多态和重定义

- 必须发生于父类与子类之间，且父类和子类中的函数必须有完全相同的原型
- 使用virtual声明后能够产生多态（如果不使用virtual，叫重定义）
- 多态是在运行期间根据具体对象的类型决定函数调用

~~~c++
//重写 重载
//重写发生在父类和子类之间
//重载发生在一个类之间

//重写
//1.重定义
//2. 多态，虚函数重写
#include<iostream>
using namespace std;

class A
{
public:
	virtual void fun()
	{
		cout << "A" << endl;
	}
	virtual void fun(int a,int b)
	{
		cout << "A" << endl;
	}
	virtual void fun(int a, int b, int c)//同一个类中的3个fun为重载
	{
		cout << "A" << endl;
	}
	void func(int a)
	{
		cout << "A" << endl;
	}
};
class B :public A
{
public:
	void func()//函数重写--1. 函数重定义
	{
		cout << "B" << endl;
	}
	virtual void fun()//函数重写--2. 多态
	{
		cout << "B" << endl;
	}
	virtual void fun(int a, int b, int c)
	{
		cout << "B" << endl;
	}
};
void main()
{
	B b;
	b.A::func(3);
	//func函数的名字，在子类发生覆盖，子类的函数名字，占用了父类的函数的名字的位置
	//编译器开始在子类中找func函数，没有1个参数的func函数，将报错
	//使用父类中的func函数，需要添加父类的域名
	system("pause");
}
~~~

#### 8.3 多态原理探究（面试）

解析图：

![37](F:\学习专用\note\pic\37.png)

~~~c++
//当类中声明虚函数时，编译器会在类中生成一个虚函数表
//虚函数表是一个存储类成员函数指针的数据结构
//虚函数表是由编译器自动生成与维护的
//virtual成员函数会被编译器放入虚函数表中
//当存在虚函数时，每个对象中都有一个指向虚函数表的指针（C++编译器给父类对象、子类对象提前布局vptr指针；当进行play(parent *p)函数时，C++编译器不需要区分子类对象或者父类对象，只需要在p指针中，找vptr指针即可。）
//VPTR一般作为类对象的第一个成员
#include<iostream>
using namespace std;

class parent
{
public:
	virtual void print()//1. 写virtual c++会特殊处理 	virtual成员函数会被编译器放入虚函数表中
	{
		cout << "parent" << endl;
	}
};
class child:public parent
{
public:
	virtual void print()
	{
		cout << "child" << endl;
	}
};
void play(parent *p)
{
	p->print();//多态 //2. 父类对象和子类对象均有vptr指针，==>虚函数表==>函数的入口地址
	//迟绑定（运行的时候，c++编译器才去判断）
}
void main()
{
	parent	p;//3. 提前布局 
			  //用类定义对象的时候，c++编译器会在对象中添加一个vptr指针
	child	c;//子类里面也有一个vptr指针
	play(&p);
	play(&c);
	system("pause");
}
~~~

#### 8.4 探究vptr指针的存在

~~~c++
#include<iostream>
using namespace std;

class parent
{
public:
	int a;
	void print()
	{
		cout << "parent" << endl;
	}
};
class parent2
{
public:
	int a;
	virtual void print()
	{
		cout << "parent2" << endl;
	}
};

void main()
{
	cout << sizeof(parent) << endl;//4
	cout << sizeof(parent2) << endl;//8 加上virtual关键字 c++编译器会增加一个指向虚函数表的指针
	system("pause");
}
~~~

#### 8.5 构造函数中能调用虚函数，实现多态吗？（面试）

答：不行

解析：

![01_子类的vptr指针是分步初始化的](F:\学习专用\note\pic\01_子类的vptr指针是分步初始化的.bmp)

~~~c++
//对象在创建的时,由编译器对VPTR指针进行初始化 
//只有当对象的构造完全结束后VPTR的指向才最终确定
//父类对象的VPTR指向父类虚函数表
//子类对象的VPTR指向子类虚函数表
#include<iostream>
using namespace std;

class parent
{
public:
	int a;
	parent(int a)
	{
		this->a = a;
		cout << "parent构造" << endl;
        print();
	}
	void print()
	{
		cout << "parent" << endl;
	}
};
class child:public parent 
{
public:
	int b;
	child(int a=10,int b=20) :parent(b)
	{
		this->b = a;
		cout << "child构造" << endl;
	}
	virtual void print()
	{
		cout << "child" << endl;
	}
};

void main()
{
	child c;
	system("pause");
}
~~~

#### 8.6 面试相关

##### 8.6.1 面试题1：请谈谈你对多态的理解

**多态的实现效果**

多态：同样的调用语句有多种不同的表现形态，一个函数在子类进行穿梭的时候表现不同的形态

**多态实现的三个条件** 

​       有继承、有virtual重写、有父类指针（引用）指向子类对象。

**多态的C++实现**

   virtual关键字，告诉编译器这个函数要支持多态；不是根据指针类型判断如何调用；而是要根据指针所指向的实际对象类型来判断如何调用
 **多态的理论基础** 

   动态联编 PK 静态联编。根据实际的对象类型来判断重写函数的调用。

**多态的重要意义** 

   设计模式的基础 是框架的基石。

**实现多态的理论基础** 

  函数指针做函数参数

C函数指针是C++至高无上的荣耀。C函数指针一般有两种用法（正、反）。

**多态原理探究**

 与面试官展开讨论，c++编译器提前布局的vptr指针，指向虚函数表，通过vptr指针找到虚函数表，找到函数入口地址，实现动态联编

##### 8.6.2 面试题2：谈谈C++编译器是如何实现多态

c++编译器多态实现原理

##### 8.6.3 面试题3：谈谈你对重写，重载理解

##### 8.6.4 面试题4：是否可类的每个成员函数都声明为虚函数，为什么

##### 8.6.5 面试题5：构造函数中调用虚函数能实现多态吗？为什么？

##### 8.6.6 面试题6：虚函数表指针（VPTR）被编译器初始化的过程，你是如何理解的？

##### 8.6.7 面试题7：父类的构造函数中调用虚函数，能发生多态吗？

##### 8.6.8 面试题8：为什么要定义虚析构函数？

通过父类指针 释放所有子类资源

# 六.顺序容器

## 1.顺序容器概述

### 1.1 顺序容器类型

| vector       | 可变大小数组。支持快速随机访问。在**尾部**插入数据或删除元素很快，可以插入到其他位置，不过很耗时 |
| ------------ | ------------------------------------------------------------ |
| deque        | 双端队列。支持快速随机访问。在**头尾**位置插入/删除数据很快  |
| list         | 双向链表。只支持**双向**顺序访问。可在**任何位置**进行插入/删除操作 |
| forward_list | 单向链表。只支持**单向**顺序访问。可在**任何位置**进行插入/删除操作 |
| array        | 固定大小数组。支持快速随机访问。不能添加或删除元素           |
| string       | 与vector相似，专门用于保存字符。随机访问快，在**尾部**插入/删除速度快 |

### 1.2 选择容器基本原则

- 一般情况下选择vector容器
- 程序要求随机访问元素，应使用vector或deque
- 程序要求在中间位置插入或删除元素，应使用list或forward_list
- 程序需要在头尾插入或删除元素，且不在中间位置插入或删除，选用deque
- 程序在读取输入时需要在容器中间位置插入元素，随后需要随机访问元素，则输入阶段使用list，输入完成后，将list中的内容拷贝到vector中

## 2.顺序容器操作

### 2.1 向顺序容器添加元素

~~~c
push_back();//向后添加
push_front();//向前添加
insert();//插入元素
~~~

**注意：**

- forward_list 有自己专有的insert和emplace，不支持push_back和emplace_back
- vector和string不支持push_front和emplace_front
- vector，list，deque和string支持push_back
- list，deque，forward_list支持push_front
- 除了array，其他顺序容器均支持insert操作
- 元素插入到vector，deque和string中的任何位置都是合法的，只是这样做很耗时

### 2.2 insert操作

将元素插入到迭代器所指定的位置**之前**,返回指向**新添加的元素的迭代器**。

~~~c++
slit.insert(iter,"hello");//将"hello"插入到迭代器指向的位置的前一个位置
slit.insert(slit.end(),10,"al");//将10个"al"插入到slit的末尾
~~~

### 2.3 emplace操作

c++11新标准引入三个新成员-emplace_front,emplace和emplace_back这些是操作构造而不是拷贝元素，分别对应push_front、insert和push_back，允许我们将元素放置在容器头部、指定位置或容器尾部。

调用emplace_back时，会在容器管理的内存空间中直接创建对象，调用push_back则会创建一个临时对象，并将其压入容器中。

### 2.4 访问元素

顺序容器中的front和back函数，返回容器中的首元素和尾元素的**引用**。

与begin和end的区别：begin是指向容器首元素的位置，end是指向元素尾元素的下一个位置

**注意：**

- 每个顺序容器均有front函数，包含array
- 除forward_list外，每个顺序容器均有back函数

~~~C++
auto val1=*s.begin();auto val2 = s.front();//val1和val2均为s的第一个元素
auto end=s.end();auto val3=*(--end);
auto val4=s.back();//val3和val4均为s的最后一个元素
auto &sa=s.back();//获得最后一个元素的引用
sa = 90;//将s中的最后一个元素变为90
~~~

### 2.5 删除元素

~~~c
pop_back;//删除容器中的尾元素
pop_front();//删除容器中的首元素
erase();//删除迭代器所指的元素，返回指向被删元素后的下一个元素的迭代器
~~~

**注意：**

- forward_list有特殊版本的erase
- forward_list不支持pop_back
- vector和string不支持pop_front

### 2.6 erase操作

~~~c++
//删除单个元素
auto it=lis.begin();
it=lis.erase(it);//删除第一个元素，此时it指向第二个元素的位置
//删除多个元素
lis.clear();
lis.erase(lis.begin(),lis.end());//与上同，删除lis中的所有元素，删除连个迭代器所表示的范围
~~~

### 2.7 特殊的forward_list 操作

forward_list单向链表，未定义insert，emplace和erase，而是insert_after,emplace_after和erase_after

~~~c++
lis.before_begin();//指向链表首元素之前不存在的元素的迭代器，有点像end()
lis.insert_after(p,t);//在迭代器p之后插入元素t
lis.emplace_after();
lis.erase_after(p);//删除p指向位置之后的元素，返回一个指向被删除元素之后的元素的迭代器
~~~

例：

~~~c++
forward_list<int> lis={0,1,2,3,4,5,6,7,8,9};
auto bef_begin=lis.before_begin();
auto begin=lis.begin();
while(begin != lis.end())
{
    if(*begin%2)//如果lis中元素为奇数，则删除该元素
        begin=lis.erase_after(bef_begin);
    else{
        bef_begin=begin;
        ++begin;
    }
}
~~~

### 2.8 容器操作可能使迭代器失效

**向容器中添加元素：**

- 容器为vector或string，指向插入位置前的元素迭代器，指针和引用仍有效，指向插入位置之后的任何位置都会导致迭代器。指针和引用将失效
- 对于list，forward_list，指向容器的迭代器，指针和引用仍有效

**删除一个元素：**

- 容器为vector或string，指向删除位置之前的迭代器。指针和引用仍有效
- 对于list，forward_list，指向容器的迭代器，指针和引用仍有效

**注意：删除元素时，尾后迭代器总会失效，所以一般不保存尾迭代器的值**

## 3. string操作

#### append和replace函数

append操作是在string末尾进行插入操作

~~~c++
string s("hello "),s2=s;
s.insert(s.size(),"world");
s2.append("world");//s2==s 为"hello world"
~~~

replace函数，调用erase和insert的简写

~~~c
s.erase(6,5);//s=="hello "
s.insert(6,"c++");//s=="hello c++"
s2.replace(6,5,"c++");//s2=="hello c++"
s2.replace(6,5,"myfamily");//删除6个字符后的5个字符，插入字符s2=="hello myfamily"
~~~

## 4.vector和string内存分配

capacity操作告诉我们容器在不作扩张内存空间的情况下可以容纳多少个元素

reserve操作允许恶魔通知容器它应该准备保存多少个元素

~~~c
vector<int> icr;
cout<<icr.size()<<icr.capacity()<<endl;//此时打印的值均为0
icr.push_back(12);
cout<<icr.size()<<icr.capacity()<<endl;//此时size为1，capacity应大于等于1,具体值依赖于标准库实现
~~~

# 七. 模板与泛型编程

函数的业务逻辑一样，函数需要的参数不一样，则需要函数模板

## 1.函数模板

**作用：让类型参数化，方便程序员编码**

#### 函数模板机制结论

- 编译器并不是把函数模板处理成能够处理任意类的函数

- 编译器从函数模板通过具体类型产生不同的函数

- 编译器会对函数模板进行**两次编译**：

  在声明的地方对模板代码本身进行编译,生成简陋的函数模板原形；

  在调用的地方对参数替换后的代码进行编译，产生具体的函数原形。

~~~c++
template<typename T> //模板参数,关键字template
int compare(const T &v1,const T &v2)
{
    if(v1<v2) return -1;
    if(v2>v1) return 1;
    return 0;
}
~~~

模板参数表示在类或函数中定义中用到的类型或值。当使用模板时，指定模板实参，将其绑定到模板参数上。

### 1.1 实例化函数模板

- **显式类型调用**，用户自己定义模板参数类型

~~~c++
int x=10,y=20;
compare<int>(x,y);
char a='a',b='b';
compare<char>(a,b);
~~~

- **自动类型推导**，调用函数模板时，编译器用函数实参来为我们推断模板实参。

~~~c++
cout<<compare(2.12,3.11)<<endl;//T为double，实参类型为double，编译器会推断出模板实参为double，并将它绑定到模板参数T
cout<<compare(2,3)<<endl;//T为int
~~~

### 1.2 模板类型参数

~~~c
template<typename T> T foo(T *P)//返回类型和参数类型相同
{
    T tmp = *p;
    ...
    return tmp;
}
~~~

类型参数前必须使用关键字class或typename，两者没什么不同

~~~c
template<typename T,class U> cals (const T&,const U&);
~~~

### 1.3 非类型模板参数

~~~c
template<unsigned N, unsigned M>
int compare(const char (&p1)[N],const char (&p2)[M])
{
    return strcmp(p1,p2);
}
compare("hi","mom");//编译器实例化版本为int compare(const char (&p1)[3],const char (&p2)[4])
~~~

### 1.4 函数模板案例

~~~c++

#include <iostream>
using namespace std;
//让 类型参数化 ===, 方便程序员进行编码
// 泛型编程 
//template 告诉C++编译器 我要开始泛型编程了 .看到T, 不要随便报错
//排序
template <typename T1, typename T2>
void mySort(T1 *a, T2 b)
{
	int i, j;
	for (i = 0;i < b;i++)
	{
		for (j = i + 1;j < b;j++)
		{
			int tmp = 0;
			if (a[i] < a[j])
			{
				tmp = a[i];
				a[i] = a[j];
				a[j] = tmp; 
			}
		}
	}

}
//函数模板作参数
template <typename T1, typename T2>
void myprint(T1 *a, T2 b)
{
	for (int i = 0;i < b;i++)
		cout << a[i] << " ";
}
//函数模板的调用
// 显示类型 调用 
// 自动类型 推导
void main()
{
	////int型
	//int array[9] = { 12,45,98,78,546,125,2,3,4 };
	//int size = sizeof(array) / sizeof(array[9]);
	//mySort<int, int>(array, size); //显示类型调用 模板函数 <>
	//myprint<int, int>(array, size);

	//char型
	char str[] = "sadsdasd4554512";
	int size1 = strlen(str);
	mySort<char, int>(str, size1);
	myprint<char, int>(str, size1);
}
~~~

### 1.5 C++编译器模板机制剖析

思考：为什么函数模板可以和函数重载放在一块。C++编译器是如何提供函数模板机制的？

#### 编译器编译原理

#### **什么是gcc** 

| gcc（GNU Compiler Collection）编译器的作者是Richard   Stallman，也是GNU项目的奠基者。 |
| ------------------------------------------------------------ |
| *什么是gcc：gcc是GNU   Compiler Collection的缩写。最初是作为C语言的编译器（GNU C Compiler），现在已经支持多种语言了，如C、C++、Java、Pascal、Ada、COBOL语言等*。 |
| gcc支持多种硬件平台，甚至对Don Knuth 设计的 MMIX 这类不常见的计算机都提供了完善的支持 |

#### **gcc主要特征**

1）gcc是一个可移植的编译器，支持多种硬件平台

2）gcc不仅仅是个本地编译器，它还能跨平台交叉编译

3）gcc有多种语言前端，用于解析不同的语言

4）gcc是按模块化设计的，可以加入新语言和新CPU架构的支持

5）gcc是自由软件

#### **gcc编译过程**

![174](F:\学习专用\note\pic\174.png)

预处理（Pre-Processing）

编译（Compiling）

汇编（Assembling）

链接（Linking）

Gcc *.c –o 1exe (总的编译步骤)

Gcc –E 1.c –o 1.i  //宏定义 宏展开

Gcc –S 1.i –o 1.s //汇编

Gcc –c 1.s –o 1.o  //二进制文件

Gcc 1.o –o 1exe

结论：gcc编译工具是一个工具链。。。。

![9](F:\学习专用\note\pic\9.png)

hello程序是一个高级Ｃ语言程序，这种形式容易被人读懂。为了在系统上运行hello.c程序，每条Ｃ语句都必须转化为低级机器指令。然后将这些指令打包成可执行目标文件格式，并以二进制形式存储器于磁盘中。

**gcc常用编译选项**

| 选项  | 作用                                                         |
| ----- | ------------------------------------------------------------ |
| -o    | 产生目标（.i、.s、.o、可执行文件等）                         |
| -c    | 通知gcc取消链接步骤，即编译源码并在最后生成目标文件          |
| -E    | 只运行C预编译器                                              |
| -S    | 告诉编译器产生汇编语言文件后停止编译，产生的汇编语言文件扩展名为.s |
| -Wall | 使gcc对源文件的代码有问题的地方发出警告                      |
| -Idir | 将dir目录加入搜索头文件的目录路径                            |
| -Ldir | 将dir目录加入搜索库的目录路径                                |
| -llib | 链接lib库                                                    |
| -g    | 在目标文件中嵌入调试信息，以便gdb之类的调试程序调试          |

**练习** 

- gcc -E   hello.c -o hello.i（预处理）
- gcc -S   hello.i -o hello.s（编译）
- gcc -c   hello.s -o hello.o（汇编）
- gcc   hello.o -o hello（链接）
- 以上四个步骤，可合成一个步骤   gcc   hello.c -o hello（直接编译链接成可执行目标文件）   gcc -c   hello.c或gcc -c hello.c -o hello.o（编译生成可重定位目标文件）   

~~~c

#include <iostream>
using namespace std;

// c.cpp

// g++ -S c.cpp  -o c.s
template <typename T>
void myswap(T &a, T &b)
{
	T c = 0;
	c = a;
	a = b;
	b = c;
	cout << "hello ....我是模板函数 欢迎 calll 我" << endl;
}
int main()
{
    {
		int x = 10; 
		int y = 20;

		myswap<int>(x, y); //1 函数模板 显示类型 调用
	}

	{
		char a = 'a'; 
		char b = 'b';

		myswap<char>(a, b); //1 函数模板 显示类型 调用
	}
	return 0;
}

~~~

~~~c
//通过gcc -S c.cpp -o c.s生成汇编文件，可以看到模板函数在编译过程中会根据显示内容调用生成不同的函数
    .file	"cc.cpp"
	.local	_ZStL8__ioinit
	.comm	_ZStL8__ioinit,1,1
	.text
	.globl	main
	.type	main, @function
main:
.LFB1022:
	.cfi_startproc
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	subq	$16, %rsp
	movq	%fs:40, %rax
	movq	%rax, -8(%rbp)
	xorl	%eax, %eax
	movl	$10, -16(%rbp)
	movl	$20, -12(%rbp)
	leaq	-12(%rbp), %rdx
	leaq	-16(%rbp), %rax
	movq	%rdx, %rsi
	movq	%rax, %rdi
	call	_Z6myswapIiEvRT_S1_       //25行转到52行
	movb	$97, -16(%rbp)
	movb	$98, -12(%rbp)
	leaq	-12(%rbp), %rdx
	leaq	-16(%rbp), %rax
	movq	%rdx, %rsi
	movq	%rax, %rdi
	call	_Z6myswapIcEvRT_S1_      //32行转到90行
	movl	$0, %eax
	movq	-8(%rbp), %rcx
	xorq	%fs:40, %rcx
	je	.L3
	call	__stack_chk_fail
.L3:
	leave
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE1022:
	.size	main, .-main
	.section	.rodata
	.align 8
.LC0:
	.string	"hello ....\346\210\221\346\230\257\346\250\241\346\235\277\345\207\275\346\225\260 \346\254\242\350\277\216 calll \346\210\221"
	.section	.text._Z6myswapIiEvRT_S1_,"axG",@progbits,_Z6myswapIiEvRT_S1_,comdat
	.weak	_Z6myswapIiEvRT_S1_
	.type	_Z6myswapIiEvRT_S1_, @function
_Z6myswapIiEvRT_S1_:            //52行
.LFB1023:
	.cfi_startproc
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	subq	$32, %rsp
	movq	%rdi, -24(%rbp)
	movq	%rsi, -32(%rbp)
	movl	$0, -4(%rbp)
	movq	-24(%rbp), %rax
	movl	(%rax), %eax
	movl	%eax, -4(%rbp)
	movq	-32(%rbp), %rax
	movl	(%rax), %edx
	movq	-24(%rbp), %rax
	movl	%edx, (%rax)
	movq	-32(%rbp), %rax
	movl	-4(%rbp), %edx
	movl	%edx, (%rax)
	movl	$.LC0, %esi
	movl	$_ZSt4cout, %edi
	call	_ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc
	movl	$_ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_, %esi
	movq	%rax, %rdi
	call	_ZNSolsEPFRSoS_E
	nop
	leave
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE1023:
	.size	_Z6myswapIiEvRT_S1_, .-_Z6myswapIiEvRT_S1_
	.section	.text._Z6myswapIcEvRT_S1_,"axG",@progbits,_Z6myswapIcEvRT_S1_,comdat
	.weak	_Z6myswapIcEvRT_S1_
	.type	_Z6myswapIcEvRT_S1_, @function
_Z6myswapIcEvRT_S1_:      //90行
.LFB1024:
	.cfi_startproc
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	subq	$32, %rsp
	movq	%rdi, -24(%rbp)
	movq	%rsi, -32(%rbp)
	movb	$0, -1(%rbp)
	movq	-24(%rbp), %rax
	movzbl	(%rax), %eax
	movb	%al, -1(%rbp)
	movq	-32(%rbp), %rax
	movzbl	(%rax), %edx
	movq	-24(%rbp), %rax
	movb	%dl, (%rax)
	movq	-32(%rbp), %rax
	movzbl	-1(%rbp), %edx
	movb	%dl, (%rax)
	movl	$.LC0, %esi
	movl	$_ZSt4cout, %edi
	call	_ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc
	movl	$_ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_, %esi
	movq	%rax, %rdi
	call	_ZNSolsEPFRSoS_E
	nop
	leave
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE1024:
	.size	_Z6myswapIcEvRT_S1_, .-_Z6myswapIcEvRT_S1_
	.text
	.type	_Z41__static_initialization_and_destruction_0ii, @function
_Z41__static_initialization_and_destruction_0ii:
.LFB1033:
	.cfi_startproc
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	subq	$16, %rsp
	movl	%edi, -4(%rbp)
	movl	%esi, -8(%rbp)
	cmpl	$1, -4(%rbp)
	jne	.L8
	cmpl	$65535, -8(%rbp)
	jne	.L8
	movl	$_ZStL8__ioinit, %edi
	call	_ZNSt8ios_base4InitC1Ev
	movl	$__dso_handle, %edx
	movl	$_ZStL8__ioinit, %esi
	movl	$_ZNSt8ios_base4InitD1Ev, %edi
	call	__cxa_atexit
.L8:
	nop
	leave
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE1033:
	.size	_Z41__static_initialization_and_destruction_0ii, .-_Z41__static_initialization_and_destruction_0ii
	.type	_GLOBAL__sub_I_main, @function
_GLOBAL__sub_I_main:
.LFB1034:
	.cfi_startproc
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	movl	$65535, %esi
	movl	$1, %edi
	call	_Z41__static_initialization_and_destruction_0ii
	popq	%rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE1034:
	.size	_GLOBAL__sub_I_main, .-_GLOBAL__sub_I_main
	.section	.init_array,"aw"
	.align 8
	.quad	_GLOBAL__sub_I_main
	.hidden	__dso_handle
	.ident	"GCC: (Ubuntu 5.4.0-6ubuntu1~16.04.10) 5.4.0 20160609"
	.section	.note.GNU-stack,"",@progbits
~~~

#### 函数模板机制结论：

- 编译器并不是把函数模板处理成能够处理任意类的函数

- 编译器从函数模板通过具体类型产生不同的函数

- 编译器会对函数模板进行**两次编译**：

  在声明的地方对模板代码本身进行编译,生成简陋的函数模板原形；

  在调用的地方对参数替换后的代码进行编译，产生具体的函数原形。

## 2.类模板

**作用：**

-  类模板用于实现类所需数据的类型参数化 
-  类模板在表示如数组、表、图等数据结构显得特别重要 ， 这些数据结构的表示和算法不受所包含的元素类型的影响

与函数模板不同的是，编译器不能为类模板推断模板参数类型，只能使用显式类型调用

~~~c
//类的类型参数化 抽象的类
//单个类模板
template<typename T>
class A 
{
public:
	A(T t)
	{
		this->t = t;
	}
	T &getT()
	{
		return t;
	}
protected:
public:
	T t;
};
void main()
{
   //模板了中如果使用了构造函数,则遵守以前的类的构造函数的调用规则
	A<int> a(100); 
	a.getT();
	printAA(a);
	return ;
}
~~~

### 2.1 实例化类模板(单个类模板)

一个类模板的每个实例都形成一个独立的类

~~~c
#include<iostream>
using namespace std;

//类模板
template<typename T>
class A {
public:
	A(T b)
	{
		this->a = b;
	}
	void printA() {
		cout << a << endl;
	}
private:
	T a;
};
//类模版作函数参数，c++编译器要求具体的类所以要加A<double>
void printAA(A<double> &a1)
{
    a1.printA();
}
int main() {
    //实例化类模板
	A<double> a(11.32)，b(5.23);//模板类A<double>==>具体类a，b==>定义具体的变量11.32
	a.printA();
    printAA(a);
    printAA(b);
	return 0;
}
~~~

### 2.2 模板类派生

#### 模板类派生普通类

~~~c++
#include<iostream>
using namespace std;

//类模板
template<typename T>
class A {
public:
	A(T b)
	{
		this->a = b;
	}
	void printA() {
		cout << a << endl;
	}
protected:
	T a;
};
//模板类派生时，需要具体化模板类，c++编译器需要知道父类具体数据类型
//要知道父类所占的内存大小，只有数据类型固定下来，才知道如何分配内存
class B : public A<int>
{
public:
	B(int a, int b) : A<int>(a)//注意：必须调用父类的构造函数
	{

		this->b = b;
	}
	void printB()
	{
		cout << "a: " << a << "b: " << b << endl;
	}
private:
	int b;
};
int main() {
	B b1(15, 56);
	b1.printB();
	return 0;
}
~~~

结果：

![10](F:\学习专用\note\pic\10.png)

#### 模板类派生模板类

~~~c++
#include<iostream>
using namespace std;

//类模板
template<typename T>
class A {
public:
	A(T b)
	{
		this->a = b;
	}
	void printA() {
		cout << a << endl;
	}
protected:
	T a;
};
//模板类 派生 模板类
template<typename T>
class C : public A<T>
{
public:
	C(T a, T c) : A<T>(a)
	{
		this->c = c;
	}
	void printC()
	{
		cout << "a: " << a << "b: " << c << endl;
	}
private:
	T c;
};
int main() {
	C<double> c1(1.23,5.63);
	c1.printC();
	return 0;
}
~~~

### 2.3 类内中类模板使用

在类模板自己的作用域中，我们可以直接使用模板名而不提供实参

~~~c++
#include<iostream>
using namespace std;

//类模板函数，所有成员写在内部
template<typename T>
class A {
	//运算符重载的正规写法
	//重载<< >>只能用友元函数，其他运算符重载都携程成员函数
	friend ostream &operator<<(ostream &out, A &a)//重载<<
	{
		out << "a1+a2="<<a.a << endl;
		return out;
	}
public:
	A(T b)
	{
		this->a = b;
	}
	void printA() {
		cout << a << endl;
	}
	A operator+(A &x)//运算符“+”号重载,传入参数为“+”号右边的实参
	{
		A tmp(a + x.a);
		return tmp;
	}
protected:
	T a;
};
int main() {

	A<int> a1(10), a2(20);
	A<int> a3 = a1 + a2;//调用重载+号
	cout << a3;//调用重载<<

	return 0;
}
~~~

结果：

![11](F:\学习专用\note\pic\11.png)

### 2.4 类外使用类模板

不在类的作用域内，直到遇到类名才表示进入类的作用域

~~~c++
#include<iostream>
using namespace std;

//类模板函数，所有成员写在外部
//注意修改形参(引用类的)，返回值（返回为类类型）和域名，需要添加模板参数<T>，缺一不可
template<typename T>
class A {
	//运算符重载的正规写法
	//重载<< >>只能用友元函数，其他运算符重载都携程成员函数
	//注意：类外部实现重载 << ，需要在声明中operator<<后添加<T>，否则报错
	friend ostream &operator<< <T> (ostream &out, A &a);//重载<< 注意加<T> !!!!!!!
public:
	A(T b);
	void printA();
	A operator-(A &x);//运算符“+”号重载,传入参数为“+”号右边的实参
protected:
	T a;
};
//构造函数的实现 写在类外部
template<typename T>
A<T>::A(T b)
{
	this->a = b;
}
//函数printA的实现
template<typename T>
void A<T>::printA()
{
	cout << a << endl;
}
//运算符“-”号重载的类外实现，返回值和形参均需要添加模板参数<T>
template<typename T>
A<T> A<T>::operator-(A<T> &x)
{
	A tmp(a - x.a);
	return tmp;
}
//友元函数 实现 << 运算符重载
template<typename T>
ostream &operator<<(ostream &out, A<T> &a)//重载<<
{
	out << "a1-a2=" << a.a << endl;
	return out;
}
int main() {

	A<int> a1(10), a2(20);
	A<int> a3 = a1 - a2;//调用重载-号
	cout << a3;//调用重载<<

	return 0;
}
~~~

**结果：**

![12](F:\学习专用\note\pic\12.png)

**注意：**

- **类外部实现重载 << ，需要在声明中operator<<后添加<T>，否则报错**
- **由于类模版中友元函数使用较复杂，所以只用来进行左移，右移操作符的重载**

### 2.5 类外使用类模板，且类模板的声明和实现在不同的.h和.cpp文件中

也就是类模板函数说明和类模板实现分开

//类模板函数

构造函数

普通成员函数

友元函数：用友元函数重载<<>>; 

​                   使用类模板函数需添加类模板实现的头文件即**要包含.cpp**

### 2.6 类模板中的static关键字

- 从类模板实例化的每个模板类有自己的类模板数据成员，该模板类的所有对象共享一个static数据成员
- 和非模板类的static数据成员一样，模板类的static数据成员也应在文件范围定义和初始化
- 每个模板类有自己的类模板的static数据成员副本

分析：

![13](F:\学习专用\note\pic\13.png)

~~~c++
#include<iostream>
using namespace std;

template<typename T>
class A
{
public:
	static T sa;
};
template<typename T>
T A<T>::sa = 0;
int main() {

	A<int> a1,a2,a3;
	a1.sa = 10;
	a2.sa++;
	a3.sa++;
	cout << A<int>::sa << endl;//编译器会将类模版中的static，编译为int类型中的static

	A<char> b1, b2, b3;
	b1.sa = 'a';
	b2.sa++;
	b3.sa++;
	cout << A<char>::sa << endl;//编译器会将类模版中的static，编译为char类型中的static
	return 0;
}
~~~

结果：

![14](F:\学习专用\note\pic\14.png)

### 2.7 类模板案例

编写一个类似vector容器的数组模板类（MyVector），完成对int、char等类型元素的管理

设计：

- 类模版中的成员函数包含：
- 构造函数，初始化对象
- 拷贝构造函数，拷贝对象
- 重载<< , 可以直接打印对象中的元素，将调用内部元素过程写入重载函数中
- 重载=，可以直接进行赋值操作（a2=a1）
- 重载[] ，实现a.m_space[1]与a[1]同一个效果

~~~c++
//MyVector.h
#include<iostream>
using namespace std;

template<typename T>
class MyVector
{
	friend ostream &operator<< <T>(ostream &out, const MyVector &obj);//重载<<
protected:
	int size;
	T *m_space; 

public:
	MyVector(int size);//构造函数

	MyVector(const MyVector &obj);//拷贝构造函数
	~MyVector();//析构函数

	int get_len()
	{
		return size;
	}

	T &operator[](int index);//重载[]
	//a1=a2
	MyVector &operator=(const MyVector &obj);//重载=
};
~~~

~~~c++
//MyVector.cpp
#include<iostream>
#include"MyVector.h"
using namespace std;

template<typename T>
MyVector<T>::MyVector(int size)//构造函数
{
	this->m_space = new T[size];
	this->size = size;
}
//MyVector<int> a2 = a1;
template<typename T>	   
MyVector<T>::MyVector(const MyVector<T> &obj)//拷贝构造函数
{
	//根据a1的大小分配内存
	size = obj.size;
	m_space = new T[size];
	//拷贝数据
	for (int i = 0;i < size;i++)
	{
		m_space[i] = obj.m_space[i];
	}

}
template<typename T>
MyVector<T>::~MyVector()//析构函数
{
	if (m_space != NULL)
	{
		delete[] m_space;
		m_space = NULL;
		size = 0;
	}
}
template<typename T>
T& MyVector<T>::operator[](int index)//重载[]
{
	return m_space[index];
}
//cout<<a1[1]
template<typename T>
ostream &operator<<(ostream &out, const MyVector<T> &obj)//重载<<
{
	for (int i = 0;i < obj.size;i++)
	{
		out << obj.m_space[i] << " ";
	}
	out << endl;
	return out;
}
//a2=a1
template<typename T>	 
MyVector<T> &MyVector<T>::operator=(const MyVector<T> &obj)//重载=
{
	//先把a2的旧内存释放
	if (m_space != NULL)
	{
		delete[] m_space;
		m_space = NULL;
		size = 0;
	}
	//根据a1的大小分配内存
	size = obj.size;
	m_space = new T[size];
	//拷贝数据
	for (int i = 0;i < size;i++)
	{
		m_space[i] = obj.m_space[i];
	}
	return *this;//返回a2本身
}

~~~

~~~c++
//test_MyVector.cpp
#include<iostream>
#include"MyVector.cpp"//注意：添加的头文件为cpp文件
using namespace std;

int main()
{

	MyVector<int> a1(10);//定义int型数组
	for (int i = 0;i < a1.get_len();i++)
	{
		a1[i] = i;//重载[] 案例
	}
	cout << "a1的值为： " << a1 << " ";//重载<< 的案例
	cout << endl;

	MyVector<int> a2 = a1;//拷贝对象的案例
	cout << "拷贝a1后，a2的值为：" << a2 << " ";
	cout << endl;
	MyVector<int> a3(3);
	for (int i = 0;i < a3.get_len();i++)
	{
		a3[i] = i;
	}
	cout << "初始化后a3的值为： " << a3 << endl;
	a3 = a2;//重载= 案例
	cout <<"用a2对a3进行重新赋值后，a3为： "<< a3 << endl;

	MyVector<char> a4(3);//定义char型数组
	a4[0] = 'a';
	a4[1] = 'b';
	a4[2] = 'c';
	cout << "a4中的字符为： "<<a4 << endl;
	return 0;

}
~~~

结果：

![15](F:\学习专用\note\pic\15.png)

## 3.重载与模板

**函数模板和普通函数的区别：**

函数模板不允许自动类型转化，而普通函数可以

**函数模板和普通函数在一起，调用规则：**

- 函数模板可以像普通函数一样被重载
- c++编译器优先考虑普通函数
- 如果函数模板可以产生一个更好的匹配，那么选择模板
- 可以通过空模板实参列表的语法限定编译器只通过模板匹配

例：

~~~c++
#include "iostream"
using namespace std;

int Max(int a, int b)
{
	cout<<"int Max(int a, int b)"<<endl;
	return a > b ? a : b;
}

template<typename T>
T Max(T a, T b)
{
	cout<<"T Max(T a, T b)"<<endl;
	return a > b ? a : b;
}

template<typename T>
T Max(T a, T b, T c)
{
	cout<<"T Max(T a, T b, T c)"<<endl;
	return Max(Max(a, b), c);
}


void main()
{
	int a = 1;
	int b = 2;

	cout<<Max(a, b)<<endl; //当函数模板和普通函数都符合调用时,优先选择普通函数
	cout<<Max<>(a, b)<<endl; //若显示使用函数模板,则使用<> 类型列表

	cout<<Max(3.0, 4.0)<<endl; //如果 函数模板产生更好的匹配 使用函数模板

	cout<<Max(5.0, 6.0, 7.0)<<endl; //重载

	cout<<Max('a', 100)<<endl;  //调用普通函数 可以隐式类型转换 
	system("pause");
	return ;
}
~~~

## 4. 总结

**归纳以上的介绍，可以这样声明和使用类模板：**

1) 先写出一个实际的类。由于其语义明确，含义清楚，一般不会出错。

2) 将此类中准备改变的类型名(如int要改变为float或char)改用一个自己指定的虚拟类型名(如上例中的T)。

3) 在类声明前面加入一行，格式为：

​    template <class 虚拟类型参数>

如：

​    template <class T> //注意本行末尾无分号

​    class Compare

​    {…}; //类体

4) 用类模板定义对象时用以下形式：

​    类模板名<实际类型名> 对象名;

​    类模板名<实际类型名> 对象名(实参表列);

如：

​    Compare<int> cmp;

​    Compare<int> cmp(3,7);

5) 如果在类模板外定义成员函数，应写成类模板形式：

   template <class 虚拟类型参数>

   函数类型 类模板名<虚拟类型参数>::成员函数名(函数形参表列) {…}

**关于类模板的几点说明：**

1) 类模板的类型参数可以有一个或多个，每个类型前面都必须加class，如：

​    template <class T1,class T2>

​    class someclass

​    {…};

在定义对象时分别代入实际的类型名，如：

​    someclass<int,double> obj;

2) 和使用类一样，使用类模板时要注意其作用域，只能在其有效作用域内用它定义对象。

3) 模板可以有层次，一个类模板可以作为基类，派生出派生模板类。

# 八.关联容器

| map      | 关联数组：保存关键字-值对，关键字不可重复        |
| -------- | ------------------------------------------------ |
| set      | 关键字即值，即只保存关键字的容器，关键字不可重复 |
| multimap | 关键字可重复出现的map                            |
| multiset | 关键字可重复出现的set                            |

## 1.关联容器操作

- set中保存的值就是关键字,是const的
- map中，元素是关键字-值对，每个元素是一个pair对象，包含一个关键字和一个关联的值，pair的关键字部分是const的，pair的first成员是一个迭代器，指向具有给定关键字的元素，second成员是一个bool值，指出元素是插入成功还是已经存在于容器中。

### 1.1 关联容器迭代器

对于map

~~~c++
map<string int> count;
auto beg=count.begin();//beg为pair类型，*beg是指向一个pair<const string,size_t>对象的引用
cout<<beg->first;//打印元素的关键字
cout<<beg->second;//打印元素的值
++beg->second;//将值加1
~~~

对于set

~~~c++
set<int> ist={0,1,2,4,5};
auto beg=ist.begin();
if(beg != ist.end())
{
    cout<<*beg++<<endl;//打印关键字
}
~~~

### 1.2 添加元素

对set

~~~c++
vector<int> ict={1,2,3,1,2,3};
set<int> ict2;
ict2.insert(ict.begin(),ict.end());//ict2中只有3个元素
~~~

对map

~~~c++
//map中插入word的四种方法
map<string,size_t> count;
count.insert({word,1});
count.insert(make_pair(word,1));
count.insert(pair<string,size_t>(word,1));
count.insert(map<string,size_t>::value_type(word,1));
~~~

### 1.3 insert返回值

insert的返回值为pair，pair的first成员是一个迭代器，指向具有给定关键字的元素，second成员是一个bool值，指出元素是插入成功还是已经存在于容器中，成功则bool值为true，否则false

~~~c++
map<string,size_t> count;
string word;
while(cin>>word)
{
    auto res=count.insert({word,1});
    if(!res.second)//查看是否插入成功
        ++res.first->second; //不成功，则计数器加1
}
~~~

### 1.4 删除元素

使用erase删除元素，与顺序容器使用一样，通过传递给erase一个迭代器来删除一个元素或一个迭代器对来删除一个范围的元素。

返回值：删除元素的数量，不存在则为0，存在1个则为1，对于multimap可返回2或以上

### 1.5 访问元素

~~~c++
set<int> ist={0,1,2,3,4,5,6};
ist.find(1);//返回一个迭代器，指向key==1的元素，此返回值为1
ist.find(11);//返回0
ist.lower_bound(k);//返回一个迭代器，指向第一个关键字不小于k的元素
ist.upper_bound(k);//返回一个迭代器，指向第一个关键字大于k的元素
ist.count(k)；//返回关键字等于k的元素的数量，不允许重复的容器，返回值为0或1
~~~

# 九.动态内存

## 1.new和delete

~~~c++
//new delete基础
#include<iostream>
using namespace std;

class T
{
private:
	int a;
public:
	T(int x)
	{
		a = x;
	}
	~T()
	{
		cout << a << endl;
	}
};

//1. malloc free  c中的函数
//   new delete   操作符  基本语法
//2. new 基础类型变量 分配数组变量 分配类对象
int main()
{
	//分配基础类型
	int *p = (int *)malloc(sizeof(int));
	*p = 20;
	free(p);
	int *p2 = new int;//分配内存
	*p2 = 30;
	delete p2;
	int *p1 = new int(10);//分配内存并初始化
	cout << *p1 << endl;
	delete p1;
	

	//分配数组类型
	int *p4 = (int *)malloc(sizeof(int)*10);//int array[10]
	p[0] = 2;
	free(p);
	int *p5 = new int[10];
	p5[1] = 9;
	delete [] p5;//注意：数组不要把[] 忘记

	//分配对象  new 能执行类的构造函数 delete能执行类的析构函数
	T *t1 = (T *)malloc(sizeof(T));
	free(t1);
	T *t2 = new T(10);
	delete t2;
	system("pause");
	return 0;
}
~~~

**molloc free和new delete的主要区别：**

- new可以为对象分配内存并初始化（调用构造函数），molloc只能分配内存
- delete会调用类的析构函数，而free不会

# 十. 重载运算

### **基本语法**

- 运算符函数是一种特殊的成员函数或友元函数
- 语法形式：

~~~c
类型 类名：：operator op(参数表)     //例：Test operator+(Test &obj1, Test &obj2)
~~~

- 一个运算符被重载后，原有意义没有失去，只是定义了相对特定类的一个新运算符

**运算符重载的两种方法**：用成员或全局函数重载运算符

区别：成员函数具有this指针，全局函数没有this指针

**二元运算符：**

~~~c
obj1 op obj2
重载为成员函数，解释为obj1.operator op(obj2)
作操作数由obj1通过this指针传递，右操作数由参数obj2传递
//例：  t1 + t2 解释为t1.operator-(t2) 函数为Test operator-(Test &t2)
重载为全局函数，解释为operator op(obj1,obj2)
左右操作数都是由参数传递
//例：  t1 + t2 解释为operator-(t1,t2) 函数为Test operator-(Test &t1，Test &t2)
~~~

**全局函数、类成员函数方法实现运算符重载步骤**
1.写出重载函数的名称operator +、-、*...
2.根据操作数，写出函数参数
3.根据业务，完成函数返回值（看函数是返回引用 指针还是元素）

### 案例（重载+ - ++ -- << >>）

~~~c++
#include<iostream>
using namespace std;

//一般情况下 ，重载操作符 + - ++ --用成员函数重载  << >>只能用全局函数重载，返回值为iostream中的域
//不能用全局函数重载 = () [] ->
//++ -- << >> [] 返回值既可以作左值也可以作右值，需返回一个引用（因为含有当左值的可能）
class Test
{
	friend Test operator+(Test &obj1, Test &obj2);//用友元函数实现私有变量的访问
    friend Test& operator++(Test &obj);
	friend Test operator++(Test &obj, int);
	friend ostream& operator<<(ostream &out, Test &obj);
	friend istream& operator>>(istream &cin, Test &obj);
public:
	Test(int);
	void print()
	{
		cout << "m_a: " << m_a << endl;
	}
	//用成员函数重载操作符 -
	Test operator-(Test &obj)
	{
		Test tmp(this->m_a - obj.m_a);
		return tmp;
	}
	//用成员函数重载操作符 前置--
	Test& operator--()
	{
		this->m_a--;
		return *this;
	}
	//用成员函数重载操作符 后置--
	Test operator--(int)
	{
		Test tmp(this->m_a);
		this->m_a--;
		return tmp;
	}
private:
	int m_a;
};

Test::Test(int a)
{
	this->m_a = a;
}
//用全局函数重载操作符 +
Test operator+(Test &obj1, Test &obj2)//用全局函数重载操作符
{
	Test tmp(obj1.m_a + obj2.m_a);
	return tmp;
}
//用全局函数重载操作符 前置++
Test& operator++(Test &obj)
{
	obj.m_a++;
	return obj;
}
//用全局函数重载操作符 后置++
Test operator++(Test &obj, int)//添加占位符 区分重载前置++
{
	//先使用 再++
	Test tmp = obj;
	obj.m_a++;
	return tmp;
}
ostream& operator<<(ostream &out, Test &obj)
{
	out << obj.m_a << endl;
	return out;
}
istream& operator >> (istream &cin, Test &obj)
{
	cin >> obj.m_a;
	return cin;
}
/*
全局函数、类成员函数方法实现运算符重载步骤
1.写出重载函数的名称operator +、-、*...
2.根据操作数，写出函数参数
3.根据业务，完成函数返回值（看函数是返回引用 指针还是元素）
*/
void main()
{
	Test t1(10), t2(20);
	//用全局函数重载操作符 +
	Test t3 = t1 + t2;//对运算符+ 进行重载，本质是一个函数
	t3.print();
	//用成员函数重载操作符 -
	Test t5(30), t6(20);
	Test t4 = t5 - t6;
	t4.print();
	//用全局函数重载操作符 前置++
	++t1;
	t1.print();
	//用成员函数重载操作符 前置--
	--t2;
	t2.print();
	//用全局函数重载操作符 后置++
	t3++;
	t3.print();
	//用成员函数重载操作符 后置--
	t4--;
	t4.print();
	//用成员函数重载操作符 <<
	cout << t5<<t4;//使用链式输出，返回的是Ostream的引用
	//用成员函数重载操作符 >>
	cin >> t6;
	cout << t6;
	system("pause");
}
~~~

### 重载=操作符

~~~c++
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
using namespace std;

class Test
{
public:
	char *p_str;
	int p_len;
public:
	Test(const char *p)
	{
		p_len = strlen(p);
		p_str = (char *)malloc(p_len + 1);//注意+1 strlen计算的长度不包含 '\0'
		strcpy(p_str, p);
	}
	Test(const Test& obj)//深拷贝，重新为对象分配新的内存空间，上图蓝色线条所示，手动编写拷贝构造函数代码，解决浅拷贝出现的问题
	{
		p_len = obj.p_len;
		p_str = (char *)malloc(p_len + 1);//注意+1 strlen计算的长度不包含 '\0'
		strcpy(p_str, obj.p_str);
	}
	~Test()
	{
		if (p_str != NULL)
		{
			free(p_str);
			p_str = NULL;
			p_len = 0;
		}
	}
	void print()
	{
		cout << p_str << endl;
	}
	/*
	重载=操作符步骤：=操作符，从右往左赋值
	1.先释放旧内存
	2.根据obj分配内存大小
	3.用obj赋值
	4.注意返回值，应为类的引用
	*/

	Test& operator=(Test &obj)
	{
		//先释放旧内存
		if (this->p_str != NULL)
		{
			delete[] p_str;
			this->p_len = 0;
		}
		// 根据obj分配内存大小
		this->p_len = obj.p_len;
		this->p_str = new char[this->p_len+1];//注意需要加1，p_len不包括'\0',字符串中含有'\0'
		//用obj赋值
		strcpy(p_str,obj.p_str);
		return *this;
	}
};

int main() {

	Test p("zxfll");
	p.print();
	Test t1 = p;
	t1.print();
	Test t2("123");
	Test t3("456");
	t2.print();
	t2 = t1 = t3;//等号操作符，默认的为浅拷贝
	t2.print();
	system("pause");
	return 0;
}
~~~

### 案例（重载[] ==）

~~~c++
//MyArray.h
#pragma once
class Array
{
public:
	Array(int len);
	Array(const Array &obj);
	int getData(int index);
	void setData(int index,int data);
	int getLen();
	~Array();
	int& operator[](int index);
	Array& operator=(Array &obj);
	bool operator==(Array &obj);
private:
	int m_len;
	int *m_data;
};
~~~

~~~c++
//MyArray.cpp
#include"MyArray.h"
#include<iostream>

Array::Array(int len)
{
	m_len = len;
	m_data = new int[m_len];
}
Array::Array(const Array &obj)
{
	m_len = obj.m_len;
	m_data = new int[m_len];
	for (int i = 0;i < m_len;i++)
	{
		m_data[i] = obj.m_data[i];
	}
}
int Array::getData(int index)
{
	return m_data[index];
}
void Array::setData(int index, int data)
{
	m_data[index] = data;
}
int Array::getLen()
{
	return m_len;
}
Array::~Array()
{
	if (m_data != NULL)
	{
		delete[] m_data;
		m_len = 0;
	}
}
//重载操作符[]
int& Array::operator[](int index)//函数返回值当左值，需要返回一个引用
{
	return this->m_data[index];
}
//重载操作符=
//释放旧内存，分配新内存，赋值
Array&  Array::operator=(Array &obj)
{
	if (this->m_data != NULL)
	{
		delete[] m_data;
		m_len = 0;
	}
	this->m_len = obj.m_len;
	this->m_data = new int[m_len];
	for (int i=0;i < m_len;i++)
	{
		m_data[i] = obj.m_data[i];
	}
	return *this;
}
//重载操作符==
bool Array::operator==(Array &obj)
{
	if (this->m_len != obj.m_len)
	{
		return false;
	}
	else
	{
		for (int i = 0;i < m_len;i++)
		{
			if (m_data[i] != obj[i])
			{
				return false;
			}
		}
		return true;
	}
}
~~~

~~~c++
//Myarraytest.cpp
#include<iostream>
#include"MyArray.h"
using namespace std;

int main()
{
	Array a1(5);
	for (int i = 0;i < a1.getLen();i++)
	{
		a1[i] = i;//重载操作符[]
	}
	for (int i = 0;i < a1.getLen();i++)
	{
		cout << a1[i] << " ";
	}
	Array a4(5);
	for (int i = a1.getLen();i <= 0;i--)
	{
		a1[i] = i;//重载操作符[]
	}
	Array a2(3),a3(2);
	a2 = a3=a1;//重载操作符=
	for (int i = 0;i < a2.getLen();i++)
	{
		cout << a2[i] << " ";
	}
	for (int i = 0;i < a3.getLen();i++)
	{
		cout << a3[i] << " ";
	}

	//重载操作符==
	if (a1 == a4)
	{
		cout << "a1,a4相等\n" << endl;
	}
	else
	{
		cout << "a1,a4不相等\n" << endl;
	}
	system("pause");
	return 0;
}
~~~

结果：

![32](F:\学习专用\note\pic\32.png)

### 案例（重载()）

~~~C++
#include<iostream>
using namespace std;

class Test
{
public:
	int operator()(int a,int b)
	{
		m_a = a;
		m_b = b;
		return m_a + m_b;
	}
private:
	int m_a;
	int m_b;
};

void main()
{
	Test t1;
	cout << t1(3, 4);//重载()
	system("pause");
}
~~~

案例（字符串类 运算符重载）

~~~c++
//MyString.h
#pragma once
#include<iostream>
using namespace std;
class MyString
{
private:
	char *m_str;
	int m_len;
public:
	MyString();
	MyString(const char *str);
	MyString(const MyString &obj);
	~MyString();
	char* fun()
	{
		return m_str;
	}
public://重载<<
	friend ostream& operator<<(ostream &out, const MyString &obj);
public://重载= == + 
	MyString& operator=(const MyString &obj);
	MyString& operator=(const char *str);
	bool operator==(const MyString &obj) const;
	bool operator==(const char *str) const;
	friend MyString operator+(const MyString &obj1, const MyString &obj2);
public://重载[] <
	char& operator[](int index) const;
	bool operator<(const char *str)const;
};
//cout << m1 << endl;
~~~

~~~
//MyString.cpp
#define _CRT_SECURE_NO_WARNINGS
#include"MyString.h"
#include<iostream>
MyString::MyString(const char *str)
{
	m_len = strlen(str);
	m_str = new char[strlen(str) + 1];//注意：分配内存时，使用的是[] 而不是()
	strcpy(m_str,str);
}
MyString::MyString(const MyString &obj)
{
	m_len = obj.m_len;
	m_str = new char[m_len + 1];
	strcpy(m_str, obj.m_str);
}
MyString::MyString()
{
	m_len = 0;
	m_str = new char[m_len+1];
	strcpy(m_str, "");
	
}
MyString::~MyString()
{
	if (m_str != NULL)
	{
		delete [] m_str;
		m_len = 0;
	}
}
ostream& operator<<(ostream &out, const MyString &obj)
{
	out << obj.m_str;
	return out;
}
MyString& MyString::operator=(const MyString &obj)
{
	if (m_str != NULL)
	{
		delete [] m_str;
		m_len = 0;
	}
	m_len = obj.m_len;
	m_str = new char[m_len + 1];
	strcpy(m_str,obj.m_str);
	return *this;
}
MyString& MyString::operator=(const char  *str)
{
	if (m_str != NULL)
	{
		delete[] m_str;
		m_len = 0;
	}
	if (str == NULL)
	{
		m_len = 0;
		m_str = new char[m_len + 1];
		strcpy(m_str,"");
	}
	else
	{
		m_len = strlen(str);
		m_str = new char[m_len + 1];
		strcpy(m_str,str);
	}
	return *this;
}
bool MyString::operator==(const MyString &obj)const
{
	if (strcmp(this->m_str, obj.m_str)==0)
	{
		return true;
	}
	else
	{
		return false;
	}
}
bool MyString::operator==(const char *str)const
{
	if (str == NULL)
	{
		if (m_len == 0)
			return true;
	}
	else
	{
		if (strcmp(this->m_str, str) == 0)
			return true;
	}
	return false;
}
MyString operator+(const MyString &obj1,const MyString &obj2)
{
	/*int len = m_len + obj.m_len;
	char *str = strcat(m_str,obj.m_str);
	if (m_str != NULL)
	{
		delete[] m_str;
		m_len = 0;
	}
	m_len = len;
	m_str = new char[len+1];
	strcpy(m_str, str);
	return *this;*/
	char *str = strcat(obj1.m_str, obj2.m_str);
	MyString tmp(str);
	return tmp;
}
char& MyString::operator[](int index)const
{
	return m_str[index];
}
bool MyString::operator<(const char *str)const
{
	if (strcmp(m_str, str) < 0)
		return true;
	return false;
}

~~~

~~~c++
//MyString_Test.cpp
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
#include"MyString.h"
using namespace std;
void main()
{

	MyString m;
	MyString m1("asd"),m3("123");
	cout << m1 << endl;
	MyString m2 = m1;
	cout << m2 << endl;
	m = m1;
	cout << m << endl;
	m = "123";
	cout << m << endl;
	if (m1 == m3)
	{
		cout << "m1==m3" << endl;
	}
	else
	{
		cout << "m1!=m3" << endl;
	}
	if (m1 == "asd2")
	{
		cout << "相等" << endl;
	}
	else
	{
		cout << "不相等" << endl;
	}
	if (m1 < "aaaa")
	{
		cout << "m1<aaaa" << endl;
	}
	else
	{
		cout << "m1>aaaa" << endl;
	}
	cout << m[2] << endl;
	//MyString m4 = m2 + m1;
	cout << m1;	
	system("pause");
}
~~~

**结果：**

![33](F:\学习专用\note\pic\33.png)

### 运算符重载的限制

![31](F:\学习专用\note\pic\31.png)

重载运算符函数可以对运算符作出新的解释，但原有基本语义不变：

- 不改变运算符的优先级
- 不改变运算符的结合性
- 不改变运算符所需要的操作数
- 不能创建新的运算符

### 为什么不要重载&&和||操作符

因为重载&&和||操作符，无法实现短路规则，即从左往右执行

例：

重载&&操作符，t1&&(t1+t2),若t1为假，则不会执行（t1+t2），而实际上是先运行了（t1+t2），不符合短路规则，所以不对&&和||进行重载

# 十一. 异常处理机制

~~~c++
#include<iostream>
using namespace std;

//1. 抛出异常后，跨函数
//2. 接收异常后 可以不处理 再抛出异常
//3. catch异常时，严格按照抛出异常的类型进行catch
void divide(int a,int b)
{
	if (b == 0)
	{
		throw a;//抛出异常 int类型 异常 ，类似于return
	}
	cout << "divide 结果： " << a / b << endl;
}
void main()
{
	try
	{
		divide(12, 0);
	}
	catch(int x)//接收异常的类型 int型 严格匹配抛出异常类型
	{
		cout << x << "被零除"<<endl;
	}
	catch (...)//catch其他类型的异常
	{
		cout << "其他类型的异常" << endl;
		throw;//向上抛出异常，接收异常后不处理
	}
	system("pause");
}
~~~

~~~c++
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
using namespace std;
//throw char *型异常
void divide(char *str1, char *str2)
{
	if (str1[0] == 'a')
	{
		throw "首字母为a";
	}
	else
	{
		strcpy(str2,str1);
		cout << str2 << endl;
	}
}

void main()
{
	char buf[] = "aello";
	char buf2[1024];
	try
	{
		divide(buf,buf2);
	}
	catch (char *str)
	{
		cout <<str<< " char *型异常!" << endl;
	}
	catch (...)
	{
		cout << "其他类型的异常" << endl;
	}
	system("pause");
}
~~~

## 11.1 异常处理的基本思想

![40](F:\学习专用\note\pic\40.png)

- C++的异常处理机制使得异常的**引发**和异常的**处理**不必在同一个函数中，这样底层的函数可以着重解决具体问题，而不必过多的考虑异常的处理。上层调用者可以再适当的位置设计**对不同类型异常**的处理。
- 异常超脱于函数机制，决定了其对函数的跨越式回跳。

![41](F:\学习专用\note\pic\41.png)

- 异常跨越函数

## 11.2 栈解旋(unwinding)

异常被抛出后，从进入try块起，到异常被抛掷前，这期间在栈上的构造的所有对象，都会被自动析构。析构的顺序与构造的顺序相反。这一过程称为栈的解旋(unwinding)。

~~~c++
#include<iostream>
using namespace std;
//异常被抛出后，从进入try块起，到异常被抛掷前，这期间在栈上的构造的所有对象，都会被自动析构。析构的顺序与构造的顺序相反。这一过程称为栈的解旋(unwinding)。
class test
{
public:
	test(int a = 10)
	{
		this->a = a;
		cout << "构造函数" << endl;
	}
	~test()
	{
		cout << "析构函数" << endl;
	}
private:
	int a;
};
void divide()
{
	test t1(2), t2(4);
	cout << "发生异常 " << endl;
	throw 1;
}

void main()
{
	try
	{
		divide();
	}
	catch (int s)
	{
		cout << "int型异常" << endl;
	}
	catch (...)
	{
		cout << "其他类型的异常" << endl;
		throw;//向上抛出异常，接收异常后不处理
	}
	system("pause");
}
~~~

结果：

![42](F:\学习专用\note\pic\42.png)

##  11.3 异常接口声明

~~~c++
#include<iostream>
using namespace std;
//不抛出任何类型的异常
void divide() throw（）
{
	cout << "发生异常 " << endl;
	throw 1;
}
//可以抛出任何类型的异常
void divide()
{
	cout << "发生异常 " << endl;
	throw 1;
}
//只能抛出int型和char型异常
void divide() throw（int ，char ）
{
	cout << "发生异常 " << endl;
	throw 1;
}

void main()
{
	try
	{
		divide();
	}
	catch (int s)
	{
		cout << "int型异常" << endl;
	}
	catch (...)
	{
		cout << "其他类型的异常" << endl;
		throw;//向上抛出异常，接收异常后不处理
	}
	system("pause");
}
~~~

## 11.4 异常变量的生命周期

~~~c++
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
using namespace std;

class test
{
public:
	test()
	{
		cout << "构造函数" << endl;
	}
	~test()
	{
		cout << "析构函数" << endl;
	}
	test(test &obj)
	{
		cout << "copy构造函数" << endl;
	}
};
void divide()
{

	cout << "异常生命周期测试" << endl;
	throw test();//产生匿名对象，执行构造函数
}

void main()
{
	try
	{
		divide();
	}
	//结论1：使用一个变量 则将执行copy构造函数，析构先析构t1，再析构匿名对象
	catch (test t1)//执行拷贝构造函数
	{
		cout << "test类类型异常" << endl;
	}
	//结论2： 使用引用，会将抛出的匿名对象 扶正
	//catch (test &t1)//将匿名对象 扶正
	//{
	//	cout << "test类类型异常" << endl;
	//}
    //结论3：类对象异常，一般使用引用
	catch (...)
	{
		cout << "其他类型的异常" << endl;
	}
	system("pause");
}
~~~

## 11.5 异常的层次结构（继承在异常中的应用）

~~~c++
/*	异常是类 – 创建自己的异常类
	异常派生
	异常中的数据：数据成员
	按引用传递异常
	在异常中使用虚函数
案例：设计一个数组类 MyArray，重载[]操作，
数组初始化时，对数组的个数进行有效检查
1）	index<0 抛出异常eNegative类
	2）index = 0 抛出异常 eZero类
	3）index>1000抛出异常eTooBig类
	4）index<10 抛出异常eTooSmall类
	5）eSize类是以上类的父类，实现有参数构造、并定义virtual void printErr()输出错误。
*/
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
	using namespace std;

class MyArray
{
public:
	MyArray(int len);
	~MyArray()
	{
		if (m_data != NULL)
		{
			delete[] m_data;
			m_data = NULL;
			m_len = 0;
		}
	}
	int getLen()
	{
		return m_len;
	}
	int &operator[](int index)//注意返回的是引用，返回值做左值
	{
		return m_data[index];
	}
	class eSize
	{
	public:
		eSize(int len)
		{
			e_len = len;
		}
		virtual void print()
		{
			cout << "e_len: " << e_len << endl;
		}
	protected:
		int e_len;
	};
	class eNegative :public eSize
	{
	public:
		eNegative(int len) :eSize(len)
		{
			;
		}
	};
	class eZero :public eSize
	{
	public:
		eZero(int len) :eSize(len)
		{
			;
		}
	};
	class eTooBig :public eSize
	{
	public:
		eTooBig(int len) :eSize(len)
		{
			;
		}
	};
	class eTooSmall :public eSize
	{
	public:
		eTooSmall(int len) :eSize(len)
		{
			;
		}
	};
private:
	int m_len;
	int *m_data;
};

MyArray::MyArray(int len)
{

	if (len<0)
	{
		throw eNegative(len);
	}
	if (len == 0)
	{
		throw eZero(len);
	}
	if (len>100)
	{
		throw eTooBig(len);
	}
	if (len<10)
	{
		throw eTooSmall(len);
	}
	this->m_len = len;
	m_data = new int[m_len];
}

void main()
{
	try
	{
		MyArray my(1000);
		for (int i = 0;i < my.getLen();i++)
		{
			my[i] = i;
		}

	}
	catch (MyArray::eSize &t1)
	{
		t1.print();
	}
	catch (...)
	{
		cout << "其他类型的异常" << endl;
	}
	system("pause");
}
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
using namespace std;

class MyArray
{
public:
	MyArray(int len);
	~MyArray()
	{
		if (m_data != NULL)
		{
			delete[] m_data;
			m_data = NULL;
			m_len = 0;
		}
	}
	int getLen()
	{
		return m_len;
	}
	int &operator[](int index)
	{
		return m_data[index];
	}
	class eSize 
	{
	public:
		eSize(int len)
		{
			e_len = len;
		}
		virtual void print()
		{
			cout << "e_len: " << e_len << endl;
		}
	protected:
		int e_len;
	};
	class eNegative:public eSize
	{
	public:
		eNegative(int len):eSize(len)
		{
			;
		}
	};
	class eZero :public eSize
	{
	public:
		eZero(int len) :eSize(len)
		{
			;
		}
	};
	class eTooBig :public eSize
	{
	public:
		eTooBig(int len) :eSize(len)
		{
			;
		}
	};
	class eTooSmall :public eSize
	{
	public:
		eTooSmall(int len) :eSize(len)
		{
			;
		}
	};
private:
	int m_len;
	int *m_data;
};

MyArray::MyArray(int len)
{

	if (len<0)
	{
		throw eNegative(len);
	}
	if (len==0)
	{
		throw eZero(len);
	}
	if (len>100)
	{
		throw eTooBig(len);
	}
	if (len<10)
	{
		throw eTooSmall(len);
	}
	this->m_len = len;
	m_data = new int[m_len];
}

void main()
{
	try
	{
		MyArray my(1000);
		for (int i = 0;i < my.getLen();i++)
		{
			my[i] = i;
		}

	}
	catch (MyArray::eSize &t1)
	{
		t1.print();
	}
	catch (...)
	{
		cout << "其他类型的异常" << endl;
	}
	system("pause");
}
~~~

## 11.6 标准异常库

![43](F:\学习专用\note\pic\43.png)

~~~c++

#include<iostream>
#include<stdexcept>//标准异常库
using namespace std;

class teacher
{
public:
	teacher(int age)
	{
		if (age > 100)
		{
			throw out_of_range("年纪过大");
		}
	}
private:
	int age;
};

void main()
{
	try
	{
		teacher(120);
	}
	catch (out_of_range &e)
	{
		cout << e.what() << endl;//what()函数为标准库中exception类中的函数，out_of_range为其子类
	}
	catch (...)
	{
		cout << "其他类型的异常" << endl;
	}
	system("pause");
}
~~~

# 十二.c++输入和输出流

C++输入输出包含以下三个方面的内容：

- 对系统指定的标准设备的输入和输出。即从**键盘输入数据**，输出到显示器屏幕。这种输入输出称为标准的输入输出，简称**标准I/O**。
- ​    以外存磁盘文件为对象进行输入和输出，即从**磁盘文件输入数据**，数据输出到磁盘文件。以外存文件为对象的输入输出称为文件的输入输出，简称**文件I/O**。
- ​    对内存中指定的空间进行输入和输出。通常指定一个字符数组作为存储空间(实际上可以利用该空间存储任何信息)。这种输入和输出称为字符串输入输出，简称串I/O。



![44](F:\学习专用\note\pic\44.png)

![45](F:\学习专用\note\pic\45.png)

**标准I/O**

![46](F:\学习专用\note\pic\46.png)

## 12.1 标准输入流

~~~c++
/*
标准输入流对象cin，重点掌握的函数
cin.get() //一次只能读取一个字符
cin.get(一个参数) //读一个字符
cin.get(三个参数) //可以读字符串
cin.getline() 从缓冲区接受字符，可以接受空格
cin.ignore() 忽略缓冲区中紧接的字符
cin.peek()//读一下缓冲区，用于网络编程
cin.putback() //将取出的字符放回缓冲区
*/
#include<iostream>
using namespace std;

void main01()
{
	int i;
	long j;
	char buf[1024];
	cin >> i;
	cin >> j;
	cin >> buf;//遇到空格停止接受数据
	cout << "i" << i << "j " << j << "buf " << buf << endl;
	system("pause");
}

void main02()
{
	char ch;
	while ((ch = cin.get()) != EOF)//一直接受数据，直到遇到EOF（ctil+z）
	{
		cout << ch << endl;
	}
	system("pause");
}
//cin.get() //一次只能读取一个字符,如果缓冲区没有数据，则程序阻塞
void main03()
{
	char a, b, c;
	cout << "cin.get()如果缓冲区没有数据，则程序阻塞" << endl;
	cin.get(a);
	cin.get(b);
	cin.get(c);
	cout << a << b << c << "缓冲区只提取了3个字符，现在缓冲区任然有数据，所以不会阻塞" << endl;
	cin.get(a).get(b).get(c);
	cout << a << b << c << endl;
	system("pause");
}
//cin.getline() 接受字符，可以接受空格
void main04()
{
	char buf[128];
	char buf1[128];
	char buf2[128];
	cout << "输入一个形为aa bb cc dd的字符串" << endl;
	cin >> buf;//遇到空格停止接受数据
	cin >> buf1;//接受缓冲区第一个不是空格的字符
	cin.getline(buf2,256);//接受剩下的字符串，可包含空格
	cout << buf << endl;
	cout << buf1 << endl;
	cout << buf2 << endl;
	system("pause");
}
//cin.ignore() 忽略字符
void main05()
{
	char buf1[128];
	char buf2[128];
	cout << "输入一个形为aa  bb cc dd的字符串" << endl;
	cin >> buf1;//接受缓冲区第一个不是空格的字符
	cin.ignore(2);//手动忽略了缓冲区中紧接的两个字符
	cin.getline(buf2, 128);//接受剩下的字符串，可包含空格
	cout << "buf1:"<<buf1 << endl;
	cout << "buf2:"<<buf2 << endl;
	system("pause");
}
//输入的整数和字符串分开处理
void main()
{
	cout << "请输入一串字符：";
	char c = cin.get();
	if ((c>='0')&&(c<='9'))//处理数字
	{
		int n;
		cin.putback(c);//将字符放回缓冲区
		cin >> n;
		cout << "number is " << n << endl;
	}
	else//处理字符串
	{
		char buf[256];
		cin.putback(c);
		cin.getline(buf, 256);
		cout << "buf: " << buf << endl;
	}
	system("pause");
}
~~~

结果04：

![47](F:\学习专用\note\pic\47.png)

结果05：

![48](F:\学习专用\note\pic\48.png)

## 12.2 标准输出流

~~~c++
/*
标准输出流对象cout
cout.flush() 刷新缓冲区
cout.put()
cout.write()
cout.width()
cout.fill()
cout.setf(标记)
*/
/*
manipulator(操作符、控制符)
flush
endl
oct
dec
hex
setbase
setw
setfill
setprecision
…
*/

#include<iostream>
using namespace std;

void main()
{
	char *p = "hlooe  asa";
	cout << "hello" << endl;
	cout.put('h').put('e') << endl;
	cout.write(p, strlen(p))<<endl;
	int a = 21;
	cout.setf(ios::showbase);//显示基数符号(0x或)
	cout << "dec:" << a << endl; //默认以十进制形式输出a
	cout.unsetf(ios::dec); //终止十进制的格式设置
	cout.setf(ios::hex); //设置以十六进制输出的状态
	cout << "hex:" << a << endl; //以十六进制形式输出a
	cout.unsetf(ios::hex); //终止十六进制的格式设置
	cout.setf(ios::oct); //设置以八进制输出的状态
	cout << "oct:" << a << endl; //以八进制形式输出a
	cout.unsetf(ios::oct);
	char *pt = "China"; //pt指向字符串"China"
	cout.width(10); //指定域宽为
	cout << pt << endl; //输出字符串
	cout.width(10); //指定域宽为
	cout.fill('*'); //指定空白处以'*'填充
	cout << pt << endl; //输出字符串
	double pi = 22.0 / 7.0; //输出pi值
	cout.setf(ios::scientific); //指定用科学记数法输出
	cout << "pi="; //输出"pi="
	cout.width(14); //指定域宽为
	cout << pi << endl; //输出pi值
	cout.unsetf(ios::scientific); //终止科学记数法状态
	cout.setf(ios::fixed); //指定用定点形式输出
	cout.width(12); //指定域宽为
	cout.setf(ios::showpos); //正数输出“+”号
	cout.setf(ios::internal); //数符出现在左侧
	cout.precision(6); //保留位小数
	cout << pi << endl; //输出pi,注意数符“+”的位置
	system("pause");
}
~~~

## 12.3 文件I/O流

- v  文件输入流 ifstream
- v  文件输出流 ofstream
- v  文件输入输出流 fstream
- v  文件的打开方式
- v  文件流的状态
- v  文件流的定位：文件指针（输入指针、输出指针）
- v  文本文件和二进制文件

![49](F:\学习专用\note\pic\49.png)

~~~c++
#include<iostream>
using namespace std;
#include"fstream"
void main()
{
	char *fname = "f:/test1.txt";
	ofstream fout(fname);//建立一个 输出流对象 和文件关联 默认方式，如果已有此名字的文件，则将其原有的内容全部清除
    //ofstream fout(fname,ios::app);//在文件的末尾添加
	fout << "回家啊谁的空间啊" << endl;
	fout << "dsadsadsa" << endl;
	//读文件
	ifstream fin(fname);//建立一个输入流对象 和文件关联
	char ch;
	while (fin.get(ch))
	{
		cout << ch;
	}
	fin.close();
	system("pause");
}
~~~

~~~c++
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
using namespace std;
#include"fstream"

class Teacher
{
public:
	Teacher()
	{
		age = 20;
		strcpy(name, "");
	}
	Teacher(int a,char *name)
	{
		age = a;
		strcpy(this->name,name);
	}
	void print()
	{
		cout << "age: " << age << "name: " << name << endl;
	}
private:
	int age;
	char name[32];
};
void main()
{
	char *fname = "f:/test1.txt";
	ofstream fout(fname,ios::binary);//建立一个 输出流对象 和文件关联 
	Teacher t1(31,"小王");
	Teacher t2(32, "小张");
	fout.write((char *)&t1,sizeof(t1));
	fout.write((char *)&t2, sizeof(t2));
	fout.close();

	//读文件
	ifstream fin(fname);//建立一个输入流对象 和文件关联
	Teacher tmp;
	fin.read((char *)&tmp, sizeof(Teacher));
	tmp.print();
	fin.read((char *)&tmp, sizeof(Teacher));
	tmp.print();
	fin.close();
	system("pause");
}
~~~

结果：

![50](F:\学习专用\note\pic\50.png)

#  十三. STL

STL的从广义上讲分为三类：algorithm（算法）、container（容器）和iterator（迭代器），容器和算法通过迭代器可以进行无缝地连接。

**使用STL的好处**

1）STL是C++的一部分，因此不用额外安装什么，它被内建在你的编译器之内。

2）STL的一个重要特点是**数据结构和算法的分离**。尽管这是个简单的概念，但是这种分离确实使得STL变得非常通用。

## 13.1 容器

![51](F:\学习专用\note\pic\51.png)

### 13.1.1 STL的string

~~~c++
//#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
using namespace std;
#include "string"
#include<algorithm>//算法的头文件
//字符串初始化
void main01()
{
	string t1 = "dasdsa";
	string t2("dadasdas");
	string t3 = t2;//通过copy构造函数 初始化对象t3
	string t4(3, 's');
	cout << t1 << endl;
	cout << t2 << endl;
	cout << t3 << endl;
	cout << t4<< endl;
	system("pause");
}
//字符串遍历
void main02()
{
	string s = "dasdasd";
	//数组方式
	for (int i = 0;i < s.length();i++)
	{
		cout << s[i] << " ";
	}
	cout << endl;
	//迭代器方式
	for (string::iterator i=s.begin();i!=s.end();i++)
	{
		cout << *i << " ";
	}
	cout << endl;

	try 
	{
		//使用at()函数
		for (int i = 0;i < s.length()+3;i++)
		{
			cout << s.at(i) << " ";//可以抛出异常,s[i]不可以抛出异常
		}
	}
	catch (...)
	{
		cout << "异常" << endl;
	}
	/*
	try
	{
		for (int i = 0;i < s.length()+6;i++)
		{
			cout << s[i] << " ";//err 越界
		}
	}
	catch (...)
	{
		cout << "异常" << endl;
	}
	*/
	system("pause");
}
//字符指针和string的转换
void main03()
{
	string s1 = "dasdas";
	//1. string ==>char *
	printf("s1:%s\n", s1.c_str());
	//2. char *==>string
	string s2("sadas");
	//string中的内容复制到char *中
	char buf[32] = {0};
	s2.copy(buf, s2.length(), 0);//从0开始拷贝s2.length()长度的字符给buf
	cout << "buf: " << buf << endl;
	system("pause");
}
//连接字符串
void main04()
{
	string a1 = "aaaa";
	string a2 = "bbbb";
	a1 = a1 + a2;
	cout << a1 << endl;
	string s1("11111");
	s1.append(a1);
	cout << s1 << endl;
	system("pause");
}
//string的查找和替换                       !!!!!重点
void main05()
{
	string a1 = "who hello sa who are boy who";
	//第一次出现who的index
	int index = a1.find("who");//查找who字符，默认为从0开始查找，第一次出现的位置的下标
	cout << "index: " << index << endl;
	 //求所有who字符的位置的下标
	int index2 = a1.find("who");//没有找到返回-1
	while (index2 != -1)
	{
		cout << "index2: " << index2 <<endl;
		index2 += 1;
		index2 = a1.find("who",index2);//从index2位置继续往后查找
	}
	//替换
	int index3 = a1.find("who");//没有找到返回-1
	while (index3 != -1)
	{
		cout << "index3: " << index3 << endl;
		a1.replace(index3, 3, "WHO");//replace函数,删除从index3位置开始的后3个位置，再从index3位置插入字符串“WHO”
		index3 += 1;
		index3 = a1.find("who", index3);//从index2位置继续往后查找
	}
	cout << "a1替换后的结果： "<<a1 << endl;
	//替换案例
	string s2 = "aa ss cc";
	s2.replace(3, 2, "SS");
	cout << "s2: "<<s2 << endl;
	system("pause");
}
//截断（区间删除）和插入
void main()
{
	string s1 = "dasd adaaa ssad";
	string::iterator it = find(s1.begin(),s1.end(),'a');//查找第一个出现'a'的位置
	if (it != s1.end())
	{
		s1.erase(it);//删除'a'
	}
	cout << "删除后的s1为： " << s1 << endl;
	s1.erase(s1.begin(), s1.end());
	cout << "全部删除后s1： " << s1 << endl;
	string s = "dsa sadadaa da awf";
	string::iterator at = find(s.begin(), s.end(), 'a');//整个字符串中进行查找
	while (at != s.end())//删除字符串中所有的'a'字符
	{
		s.erase(at);
		at = find(at, s.end(), 'a');//从位置at开始查找
	}
	cout << "s: " << s << endl;
	
	string s2 = "dadsa";
	s2.insert(0, "AAA");//在位置0插入“AAA”,头插法
	s2.insert(s2.length(), "CCC");//尾插法
	cout << "s2: " << s2 << endl;
	system("pause");
}
//大小写转换
void main07()
{
	string s1 = "AAAdsd";
	transform(s1.begin(),s1.end(),s1.begin(),toupper);//从首元素位置开始转换，转换为大写
	cout << "s1: " << s1 << endl;
	string s2 = "ASDASasa";
	transform(s2.begin(),s2.end(),s2.begin(),tolower);//转换为小写
	cout << "s2: " << s2 <<endl;
	system("pause");
}
~~~

结果05：

![52](F:\学习专用\note\pic\52.png)

### 13.1.2 vector容器

 ~~~c++
//#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
using namespace std;
#include "vector"
#include<algorithm>//算法的头文件

void main01()
{
	vector<int> arr;
	for (int i = 0;i < 5;i++)
	{
		arr.push_back(i+1);
	}
	cout << "arr大小： "<<arr.size() << endl;
	cout << "arr的首元素： " << arr.front() << endl;
	cout << "arr的尾元素： " << arr.back() << endl;
	arr.pop_back();
	cout<<"删除尾元素后,尾元素为： "<< arr.back() << endl;
	//修改首元素和尾元素
	arr.front() = 10;
	arr.back() = 40;
	cout << "修改首元素： " << arr.front() << endl;
	cout << "修改尾元素： " << arr.back() << endl;
	//修改全部元素
	for (vector<int>::iterator i = arr.begin(); i != arr.end();i++)
	{
		*i = *i + 10;
		cout << *i << endl;
	}

	system("pause");
}
//vector初始化
void main02()
{
	vector<int> v1(10);//v1为10个0
	vector<int> v2;
	v2.push_back(10);
	v2.push_back(20);
	vector<int> v3 = v2;//类模版的拷贝构造函数
	vector<int> v4(3, 9);//初始化v4为999
	vector<int> v5(v1.begin(),v1.begin()+3);
	system("pause");
}
//vector遍历
void main03()
{
	vector<int> v1(10);//提前分配v1的内存，v1为10个0
	//使用数组方式遍历
	for (int i = 0;i < v1.size();i++)
	{
		v1[i] = i + 10;//修改v1中的元素的初始值
		cout << v1[i] << endl;
	}
	vector<int> v2;
	v2.push_back(1);
	v2.push_back(2);
	v2.push_back(3);
	for (int i = 0;i < v2.size();i++)
	{
		cout << v2[i] << endl;
	}
	//使用迭代器方式遍历,正向和逆向
	/*迭代器种类
	typedef typename _Mybase::iterator iterator;
	typedef typename _Mybase::const_iterator const_iterator;只读迭代器

	typedef _STD reverse_iterator<iterator> reverse_iterator;
	typedef _STD reverse_iterator<const_iterator> const_reverse_iterator;
	*/
	for (vector<int>::iterator i = v2.begin(); i != v2.end();i++)
	{
		cout << *i << endl;
	}
	//反向遍历，reverse_iterator迭代器
	for (vector<int>::reverse_iterator i2 = v2.rbegin(); i2 != v2.rend();i2++)//注意此处任然是++，
	{
		cout << *i2 << endl;
	}
	system("pause");
}
//push_back()强化
void main04()
{
	vector<int> v2(10);//定义v2为10个0
	v2.push_back(1);//向后添加元素，未修改元素初始值
	v2.push_back(2);
	v2.push_back(3);
	cout << v2.size() << endl;
	system("pause");
}
//区间删除，删除指定元素                  ！！！！重点
void main()
{
	vector<int> v1(10);
	for (int i = 0;i < v1.size();i++)
	{
		v1[i] = i + 10;
	}
	for (int i = 0;i < v1.size();i++)
	{
		cout << v1[i] << " ";
	}
	cout << endl;
	v1.erase(v1.begin(), v1.begin() + 3);//删除前3个元素
	for (int i = 0;i < v1.size();i++)
	{
		cout << v1[i] <<" ";
	}
	cout << endl;
    v1.erase(v1.begin());//在头部删除一个元素
	v1[1] = 20;
	v1[3] = 20;
	v1[5] = 20;
	for (int i = 0;i < v1.size();i++)
	{
		cout << v1[i] << " ";
	}
	cout << endl;
	for (vector<int>::iterator it = v1.begin();it != v1.end();)//删除指定元素20
	{
		if (*it == 20)
		{
			it=v1.erase(it);//删除元素后，会返回删除元素的下一个位置
		}
		else
		{
			it++;
		}
	}
	for (int i = 0;i < v1.size();i++)
	{
		cout << v1[i] << " ";
	}
	cout << endl;
	system("pause");
}
 ~~~

结果：

![54](F:\学习专用\note\pic\54.png)

**begin(),rbegin(),end(),rend()函数图解：**

![53](F:\学习专用\note\pic\53.png)

### 13.1.3 deque 双端数组

![55](F:\学习专用\note\pic\55.png)

~~~c++
//push_back() push_front() pop_back() pop_front() distance()
#include<iostream>
using namespace std;
#include "deque"
#include<algorithm>//算法的头文件

void print(deque<int> &d)
{
	for (deque<int>::iterator it=d.begin();it!=d.end();it++)
	{
		cout << *it << " ";
	}
	cout << endl;
}
//deque双向数组操作，基本与vector相同
void main()
{
	deque<int> d1;
	d1.push_back(11);
	d1.push_back(22);
	d1.push_back(33);
	d1.push_front(-11);
	d1.push_front(-22);
	d1.push_front(-33);
	print(d1);
	cout << "头部元素"<<d1.front() << endl;
	cout << "尾部元素"<<d1.back()<< endl;
	//查找元素的位置
	deque<int>::iterator it = find(d1.begin(), d1.end(), -22);
	if (it != d1.end())
	{
		cout << "-22的下标是： " << distance(d1.begin(), it) << endl;
	}
	else
	{
		cout << "未找到-22" << endl;
	}
	//删除头部，尾部元素
	d1.pop_back();
	d1.pop_front();
	print(d1);

	system("pause");
}
~~~

### 13.1.4 stack栈

![56](F:\学习专用\note\pic\56.png)

~~~c++
//栈：先进后出
//push() top() pop() size() empty()
#include<iostream>
using namespace std;
#include "stack"
#include<algorithm>//算法的头文件

class Teacher
{
public:
	int age;
	void print()
	{
		cout << "age: " << age << endl;
	}
};
void main01()
{
	stack<int> st;
	//入栈
	for (int i = 0;i < 10;i++)
	{
		st.push(i + 1);
	}
	cout << st.size() << endl;//栈大小
	//出栈
	while ( !st.empty())
	{
		int tmp=st.top();//获取栈顶元素
		cout << tmp << " ";
		st.pop();//出栈
	}
	cout << endl;

	system("pause");
}
//stack算法 和数据类型的分离
void main02()
{
	stack<Teacher> st;
	Teacher t1, t2, t3;
	t1.age = 30;
	t2.age = 31;
	t3.age = 32;
	st.push(t1);
	st.push(t2);
	st.push(t3);
	while (!st.empty())
	{
		Teacher tmp=st.top();
		tmp.print();
		st.pop();
	}
	system("pause");
}
//stack放入指向类的指针
void main()
{
	stack<Teacher *> st;
	Teacher t1, t2, t3;
	t1.age = 30;
	t2.age = 31;
	t3.age = 32;
	st.push(&t1);
	st.push(&t2);
	st.push(&t3);
	while (!st.empty())
	{
		Teacher *tmp = st.top();
		tmp->print();
		st.pop();
	}
	system("pause");
}
~~~

### 13.1.5 queue队列

![57](F:\学习专用\note\pic\57.png)

~~~c++
//先进先出
//push() pop() size() front() back() 
#include<iostream>
using namespace std;
#include "queue"
#include<algorithm>//算法的头文件
class Teacher
{
public:
	int age;
	void print()
	{
		cout << "age: " << age << endl;
	}
};
//队列基本操作
void main01()
{
	queue<int> q;
	q.push(3);
	q.push(30);
	q.push(300);
	cout << "队尾元素： " << q.back() << endl;
	cout << "队列大小： " << q.size() << endl;
	while (!q.empty())
	{
		int tmp = q.front();
		cout << tmp << endl;
		q.pop();
	}
	system("pause");
}

//queue算法 和数据类型的分离
void main()
{
	queue<Teacher> q1;
	Teacher t1, t2, t3;
	t1.age = 30;
	t2.age = 31;
	t3.age = 32;
	q1.push(t1);
	q1.push(t2);
	q1.push(t3);
	while (!q1.empty())
	{
		Teacher tmp = q1.front();
		tmp.print();
		q1.pop();
	}
	system("pause");
}
~~~

### 13.1.6 List双向链表

~~~c++

#include<iostream>
using namespace std;
#include "list"
#include<algorithm>//算法的头文件
//list基本操作，插入元素（尾部，头部，中间插入）
void main01()
{
	list<int> l;
	for (int i = 0;i < 10;i++)
	{
		l.push_back(i);
	}
	cout << "l的大小：" << l.size() << endl;
	
	for (list<int>::iterator it = l.begin();it != l.end();it++)
	{
		cout << *it << " ";
	}
	cout << endl;
	list<int>::iterator it2 = l.begin();
	it2++;
	it2++;
	//it2=it2+6;//err 不支持随机访问容器
	l.insert(it2, 100);//结论： 向前插入，在2号位置插入元素，原来的3号位置变成4号，原4号变成5号，依次后移
	for (list<int>::iterator it = l.begin();it != l.end();it++)
	{
		cout << *it << " ";
	}
	system("pause");
}
//删除元素操作
void main()
{
	list<int> l;
	for (int i = 0;i < 10;i++)
	{
		l.push_back(i);
	}
	for (list<int>::iterator it = l.begin();it != l.end();it++)
	{
		cout << *it << " ";
	}
	cout << endl;
	list<int>::iterator it1 = l.begin();
	list<int>::iterator it2 = l.begin();
	it2++;
	it2++;
	l.erase(it1,it2);//删除元素为左闭右开区间，不会删除it2迭代器所指向的位置2
	for (list<int>::iterator it = l.begin();it != l.end();it++)
	{
		cout << *it << " ";
	}
	cout << endl;
	system("pause");
}

~~~

### 13.1.7 优先级队列priority_queue

~~~c++

#include<iostream>
using namespace std;
#include "queue"
#include<algorithm>//算法的头文件
//priority_queue 最大优先级 队列插入元素后按从大到小自动排列
void main()
{
	priority_queue<int> q;//默认为最大优先级，相当于priority_queue<int, vector<int>, less<int> > p1;
	q.push(100);
	q.push(21);
	q.push(456);
	q.push(222);
	cout << "最大优先级队列 首元素： " << q.top() << endl;
	while (q.size()>0)
	{
		cout << q.top() << " ";
		q.pop();
	}
	cout << endl;
	system("pause");
}
//最小优先级？？
~~~

### 13.1.8 set容器

~~~c++
/*
	一、容器set / multiset的使用方法；
红黑树的变体，查找效率高，插入不能指定位置，插入时自动排序。
	二、functor的使用方法；
类似于函数的功能，可用来自定义一些规则，如元素比较规则。
	三、pair的使用方法。
对组，一个整体的单元，存放两个类型(T1, T2，T1可与T2一样)的两个元素。
*/
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
using namespace std;
#include<set>
#include<algorithm>//算法的头文件
//1.set是一个集合容器，其中所包含的元素是唯一的，集合中的元素按一定的顺序排列。元素插入过程是按排序规则插入，所以不能指定插入位置。
//2.set采用红黑树变体的数据结构实现，红黑树属于平衡二叉树。在插入操作和删除操作上比vector快。
//3.set不可以直接存取元素。（不可以使用at.(pos)与[]操作符）。
//4.multiset与set的区别：set支持唯一键值，每个元素值只能出现一次；而multiset中同一值可以出现多次。
//5.不可以直接修改set或multiset容器中的元素值，因为该类容器是自动排序的。如果希望修改一个元素值，必须先删除原有的元素，再插入新的元素。
void main01()
{
	set<int> s;
	for (int i = 0;i < 5;i++)
	{
		int a = rand();
		s.insert(a);
	}
	s.insert(12);
	s.insert(12);//插入的元素不能重复
	for (set<int>::iterator it = s.begin();it != s.end();it++)
	{
		cout << *it << " ";
	}
	cout << endl;
	//删除元素
	while (s.size() > 0)
	{
		set<int>::iterator it = s.begin();
		s.erase(it);
	}
	cout << "删除后s的大小： " << s.size() << endl;
	system("pause");
}
//仿函数
struct greater
{
	bool operator()(const int &t1, const int &t2)
	{
		if (t1>t2)
		{
			return true;
		}
		else
		{
			return false;
		}
	}
};
//从大到小排序
void main02()
{
	set<int,greater> s;
	for (int i = 0;i < 5;i++)
	{
		int a = rand();
		s.insert(a);
	}
	s.insert(12);
	s.insert(12);//插入的元素不能重复
	for (set<int>::iterator it = s.begin();it != s.end();it++)
	{
		cout << *it << " ";
	}
	cout << endl;
	//删除元素
	while (s.size() > 0)
	{
		set<int>::iterator it = s.begin();
		s.erase(it);
	}
	cout << "删除后s的大小： " << s.size() << endl;
	system("pause");
}

class Teacher
{
public:
	char name[64];
	int age;
	Teacher(char *p,int age)
	{
		strcpy(name, p);
		this->age = age;
	}
};
//仿函数
struct func
{
	bool operator()(const Teacher &t1, const Teacher &t2)
	{
		if (t1.age>t2.age)
		{
			return true;
		}
		else
		{
			return false;
		}
	}
};
//对自定义的类型进行排序
void main03()
{
	set<int, less<int> > ss;
	Teacher t1("wang", 20), t2("li", 30), t3("zhang", 25);
	set<Teacher,func> s;
	s.insert(t1);
	s.insert(t2);
	s.insert(t3);
	for (set<Teacher, func>::iterator it = s.begin();it != s.end();it++)
	{
		cout << it->age << "\t" << it->name << endl;
	}
	system("pause");
}
//pair的使用
void main04()
{
	Teacher t1("wang", 20), t2("li", 30), t3("zhang", 25),t4("das",25);//t4能否插入成功
	set<Teacher, func> s;
	pair<set<Teacher, func>::iterator, bool> pair1 = s.insert(t1);//insert的返回值是pair类型
	if (pair1.second == true)
	{
		cout << "插入t1成功" << endl;
	}
	else
	{
		cout << "插入失败" << endl;
	}
	s.insert(t2);
	s.insert(t3);
	pair<set<Teacher, func>::iterator, bool> pair4 = s.insert(t4);//insert的返回值是pair类型
	if (pair4.second == true)
	{
		cout << "插入t1成功" << endl;
	}
	else
	{
		cout << "插入失败" << endl;
	}
	for (set<Teacher, func>::iterator it = s.begin();it != s.end();it++)
	{
		cout << it->age << "\t" << it->name << endl;
	}
	system("pause");
}
//find() lower_bound() upper_bound()
void main()
{
	set<int> s;
	for (int i = 0;i < 5;i++)
	{
		s.insert(i);
	}
	int num = s.count(2);//返回元素为2的个数
	cout << "元素为2的个数为： " << num << endl;
	set<int>::iterator it = s.find(3);//返回元素为3的迭代器的位置
	cout << "*it: " << *it << endl;
	set<int>::iterator it1 = s.lower_bound(3);//返回第一个大于等于元素3的迭代器
	cout << "*it1: " << *it1 << endl;
	set<int>::iterator it2 = s.upper_bound(3);//返回第一个大于元素3的迭代器
	cout << "*it2: " << *it2 << endl;
	system("pause");
}
~~~

13.1.9 map容器

~~~c++
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
using namespace std;
#include<map>
#include<string>
#include<algorithm>//算法的头文件
void main01()
{
	map<int, string> m1;
	//插入元素，方法1
	m1[1] = "student1";
	m1[2] = "student2";
	//方法2
	m1.insert(pair<int,string>(3,"student3"));
	m1.insert(pair<int, string>(4, "student4"));
	//方法3
	m1.insert(make_pair(5,"student5"));
	m1.insert(make_pair(6, "student6"));
	//方法4
	m1.insert(map<int,string>::value_type(7,"student7"));
	m1.insert(map<int, string>::value_type(8, "student8"));
	cout << "m1大小： " << m1.size() << endl;
	//遍历
	for (map<int, string>::iterator it = m1.begin();it != m1.end();it++)
	{
		cout << it->first << "\t" << it->second << endl;
	}
	//删除容器
	m1.erase(m1.begin(),m1.end());
	cout << "删除容器后m1大小： " << m1.size() << endl;
}
//插入的四种方法的异同
//方法1 若key已经存在，则覆盖之前的key，value值
//后三种方法 返回pair<iterator,bool>类型， 若key已经存在，则报错
void main02()
{
	map<int, string> m1;
	//插入元素，方法1
	m1[1] = "student1";
	m1[1] = "student11";
	//方法2 
	pair<map<int,string>::iterator,bool> pair3 = m1.insert(pair<int, string>(3, "student3"));
	if (pair3.second != true)
	{
		cout << "插入失败" << endl;
	}
	else
	{
		cout << pair3.first->first << "\t" << pair3.first->second << endl;
	}
	m1.insert(pair<int, string>(4, "student4"));
	//方法3
	m1.insert(make_pair(5, "student5"));
	m1.insert(make_pair(6, "student6"));
	//方法4
	pair<map<int, string>::iterator, bool> pair7 =m1.insert(map<int, string>::value_type(7, "student7"));
	if (pair7.second != true)
	{
		cout << "插入失败" << endl;
	}
	else
	{
		cout << pair7.first->first << "\t" << pair7.first->second << endl;
	}
	pair<map<int, string>::iterator, bool> pair8=m1.insert(map<int, string>::value_type(7, "student77"));
	if (pair8.second != true)
	{
		cout << "插入失败" << endl;
	}
	else
	{
		cout << pair8.first->first << "\t" << pair8.first->second << endl;
	}
	cout << "m1大小： " << m1.size() << endl;
	//遍历
	for (map<int, string>::iterator it = m1.begin();it != m1.end();it++)
	{
		cout << it->first << "\t" << it->second << endl;
	}
	//删除容器
	m1.erase(m1.begin(), m1.end());
	cout << "删除容器后m1大小： " << m1.size() << endl;
}
void main03()
{
	map<int, string> m1;
	m1[1] = "student1";
	m1[2] = "student2";
	m1.insert(pair<int, string>(3, "student3"));
	m1.insert(pair<int, string>(4, "student4"));
	m1.insert(make_pair(5, "student5"));
	m1.insert(make_pair(6, "student6"));
	m1.insert(map<int, string>::value_type(7, "student7"));
	m1.insert(map<int, string>::value_type(8, "student8"));
	map<int, string>::iterator it = m1.find(20);
	if (it == m1.end())
	{
		cout << "未找到key为20的元素" << endl;
	}
	else
	{
		cout << it->first << "\t" << it->second << endl;
	}
	//equal_range使用 
	//返回的第一个迭代器为>=输入值的位置 
	//返回的第二个迭代器为>输入值的位置 形成一个pair 
	pair<map<int, string>::iterator, map<int, string>::iterator> mypair = m1.equal_range(3);//返回两个迭代器
	if (mypair.first == m1.end())//第一个迭代器，异常处理
	{
		cout << "第一个迭代器 >= 3的位置不存在" << endl;
	}
	else
	{
		cout << mypair.first->first << "\t" << mypair.first->second << endl;
	}
	if (mypair.second == m1.end())
	{
		cout << "第二个迭代器 >3的位置不存在" << endl;
	}
	else
	{
		cout << mypair.second->first << "\t" << mypair.second->second << endl;
	}
}
void main()
{
	//main01();
	//main02();
	main03();
	system("pause");
}
~~~

结果02：

![58](F:\学习专用\note\pic\58.png)

结果03：

![59](F:\学习专用\note\pic\59.png)

### 13.1.9 multimap容器

~~~c++
//Multimap 案例 :
//1个key值可以对应多个valude  =分组 
//公司有销售部 sale （员工2名）、技术研发部 development （1人）、财务部 Financial （2人） 
//人员信息有：姓名，年龄，电话等组成
//通过 multimap进行 信息的插入、保存、显示
//分部门显示员工信息 

#include<iostream>
using namespace std;
#include<map>
#include<string>
#include<algorithm>//算法的头文件

class Person
{
public:
	Person(int a,string b,int c)
	{
		age = a;
		name = b;
		phone = c;
	}
public:
	int		age;
	string	name;
	int		phone;
};
void main01()
{
	multimap<string, Person> map1;
	Person p1(20, "小王", 123456), p2(23, "小夏", 321654), p3(18, "小黑", 000000), p4(40, "小张", 1111111), p5(25,"小飞",8888888);
	map1.insert(make_pair("sale", p1));
	map1.insert(make_pair("sale", p2));
	map1.insert(make_pair("development", p3));
	map1.insert(make_pair("financial", p4));
	map1.insert(make_pair("financial", p5));
	//遍历信息
	for (multimap<string, Person>::iterator it = map1.begin();it != map1.end();it++)
	{
		cout << it->first << "\t" << it->second.name << endl;
	}
	cout << "sale部门人数： " << map1.count("sale") << endl;
	//打印development部门的人员名字
	cout << "打印development部门的人员名字" << endl;
	int number = map1.count("development");
	multimap<string, Person>::iterator it = map1.find("development");
	while (it != map1.end()&&number>0)
	{
		cout << it->first << "\t" << it->second.name << endl;	
		it++;
		number--;
	}
	//按照条件修改 将age=25的名字改为"age25"
	cout << "按照条件修改 将age=25的名字改为\"age25\"" << endl;
	for (multimap<string, Person>::iterator it = map1.begin();it != map1.end();it++)
	{
		if (it->second.age == 25)
		{
			it->second.name = "age25";
		}
	}
	//遍历信息
	for (multimap<string, Person>::iterator it = map1.begin();it != map1.end();it++)
	{
		cout << it->first << "\t" << it->second.name << endl;
	}
}
void main()
{
	main01();
	system("pause");
}
~~~

结果：

![60](F:\学习专用\note\pic\60.png)

### 13.1.10 容器共性机制研究 

![61](F:\学习专用\note\pic\61.png)

**结论：**

**理论提高：**所有容器提供的都是**值（value）语意**，而非引用（reference）语意。**容器执行插入元素的操作时，内部实施拷贝动作。**所以STL容器内存储的元素必须**能够被拷贝**（必须提供拷贝构造函数）。

-  除了queue与stack外，每个容器都提供可返回迭代器的函数，运用返回的迭代器就可以访问元素。
-   通常STL不会丢出异常。要求使用者确保传入正确的参数。
-  每个容器都提供了一个默认构造函数跟一个默认拷贝构造函数。 如已有容器vecIntA。  vector<int> vecIntB(vecIntA);  //调用拷贝构造函数，复制vecIntA到vecIntB中。

 与大小相关的操作方法(c代表容器)：

- c.size();   //返回容器中元素的个数
- c.empty();   //判断容器是否为空

 比较操作(c1,c2代表容器)：

- c1 == c2     判断c1是否等于c2
- c1 != c2      判断c1是否不等于c2
- c1 = c2        把c2的所有元素指派给c1 //每个类节点需含有重载=操作符

~~~c++

#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
using namespace std;
#include<vector>
#include<algorithm>//算法的头文件

class Person
{
public:
	Person(int a, char *b)
	{
		age = a;
		name = new char[strlen(b) + 1];
		strcpy(name, b);
	}
	Person(const Person &obj)
	{
		age = obj.age;
		name = new char[strlen(obj.name) + 1];
		strcpy(name, obj.name);
	}
	~Person()
	{
		if (name != NULL)
		{
			delete[] name;
			name = NULL;
			age = 0;
		}
	}
	Person &operator=(const Person &obj)
	{
		//先清除旧内存
		if (name != NULL)
		{
			delete[] name;
			name = NULL;
			age = 0;
		}
		//分配内存
		name = new char[strlen(obj.name)+1];
		//赋值
		age = obj.age;
		strcpy(name,obj.name);
		return *this;
	}
public:
	int		age;
	char	*name;

};
void play()
{
	Person p1(30, "小王");

	vector<Person> v1;//存入类节点 拷贝构造函数 重载=操作符
	v1.push_back(p1);
	vector < Person> v2;
	v2 = v1;//类节点，需重载=
	cout<<v1[0].age<<"\t"<<v1[0].name<<endl;
	cout << v2[0].age << "\t" << v2[0].name << endl;
}
void main()
{
	play();
	system("pause");
}
~~~

### 13.1.11 各容器使用时机

![62](F:\学习专用\note\pic\62.png)

-  Vector的使用场景：比如软件历史操作记录的存储，我们经常要**查看历史记录**，比如上一次的记录，上上次的记录，但却不会去删除记录，因为记录是事实的描述。
-  deque的使用场景：比如**排队购票系统**，对排队者的存储可以采用deque，支持头端的快速移除，尾端的快速添加。如果采用vector，则头端移除时，会移动大量的数据，速度慢。

vector与deque的比较：

 一：vector.at()比deque.at()效率高，比如vector.at(0)是固定的，deque的开始位置却是不固定的。

二：如果有大量释放操作的话，vector花的时间更少，这跟二者的内部实现有关。

 三：deque支持头部的快速插入与快速移除，这是deque的优点。

-  list的使用场景：比如公交车乘客的存储，随时可能有乘客下车，支持频繁的不确实位置元素的移除插入。
- set的使用场景：比如对手机游戏的个人得分记录的存储，存储要求从高分到低分的顺序排列。 
-  map的使用场景：比如按ID号存储十万个用户，想要快速要通过ID查找对应的用户。二叉树的查找效率，这时就体现出来了。如果是vector容器，最坏的情况下可能要遍历完整个容器才能找到该用户。

## 13.2 算法

#### 算法概述

算法部分主要由头文件<algorithm>，<numeric>和<functional>组成。

<algorithm>是所有STL头文件中最大的一个，其中常用到的功能范围涉及到比较、交换、查找、遍历操作、复制、修改、反转、排序、合并等等。

 <numeric>体积很小，只包括几个在序列上面进行简单数学运算的模板函数，包括加法和乘法在序列上的一些操作。

  <functional>中则定义了一些模板类，用以声明函数对象。

  STL提供了大量实现算法的模版函数，只要我们熟悉了STL之后，许多代码可以被大大的化简，只需要通过调用一两个算法模板，就可以完成所需要的功能，从而大大地提升效率。

#### 算法分类

- - 非可变序列算法 指不直接修改其所操作的容器内容的算法

  - - 计数算法       count、count_if
    - 搜索算法       search、find、find_if、find_first_of、…
    - 比较算法       equal、mismatch、lexicographical_compare

  - 可变序列算法 指可以修改它们所操作的容器内容的算法

  - - 删除算法       remove、remove_if、remove_copy、…
    - 修改算法       for_each、transform
    - 排序算法       sort、stable_sort、partial_sort、

### 13.2.1 函数对象和谓词

**函数对象：** 

重载函数调用操作符的类，其对象常称为函数对象(function object)，即它们是行为类似函数的对象。一个类对象，表现出一个函数的特征，就是通过“对象名+([参数列表](http://baike.baidu.com/view/8423750.htm))”的方式使用一个类对象，如果没有上下文，完全可以把它看作一个函数对待。

这是通过[重载](http://baike.baidu.com/view/126530.htm)类的operator()来实现的。

“在标准库中，函数对象被广泛地使用以获得弹性”，标准库中的很多算法都可以使用函数对象或者函数来作为自定的回调行为；

**谓词：**

一元函数对象：函数参数1个；

二元函数对象：函数参数2个；

一元谓词 函数参数1个，函数返回值是bool类型，可以作为一个判断式

​                       谓词可以使一个仿函数，也可以是一个回调函数。

二元谓词 函数参数2个，函数返回值是bool类型

~~~c++

#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
using namespace std;
#include<vector>
#include<algorithm>//算法的头文件

template<typename T>
class Show
{
public:
	Show()
	{
		a = 0;
	}
	void operator()(T &t)
	{
		a++;
		cout << t << endl;
	}
	void print()
	{
		cout << "a: "<<a<< endl;
	}
private:
	int a;
};

template<typename T>
void Func(T &t)
{
	cout << t << endl;
}
void Func2(int &t)
{
	cout << t << endl;
}
//函数对象定义，以及函数对象和普通函数的异同
void main01()
{
	int a = 10;
	Show<int> show;
	show(a);//函数对象的执行很像一个函数//仿函数
	Func<int>(a);//模板函数使用
	Func2(a);//普通函数使用
}
//函数对象是属于类对象，能突破函数的概念，能保持调用状态信息《=====函数对象的好处
//for_each算法中，对象做函数参数
//for_each算法中，函数对象当返回值
//结论要点：分清楚 stl算法返回的只是迭代器 还是 谓词(函数对象) 是stl算法的重要点
void main02()
{
	vector<int> v1;
	v1.push_back(10);
	v1.push_back(100);
	v1.push_back(1000);
	for_each(v1.begin(), v1.end(), Show<int>());//匿名函数对象 匿名仿函数
	for_each(v1.begin(), v1.end(), Func<int>);
	for_each(v1.begin(),v1.end(),Func2);//通过回调函数
	Show<int> show;
	/*
	template<class _InIt,
	class _Fn1> inline
	_Fn1 for_each(_InIt _First, _InIt _Last, _Fn1 _Func)
	{	// perform function for each element
	_DEBUG_RANGE_PTR(_First, _Last, _Func);
	_For_each_unchecked(_Unchecked(_First), _Unchecked(_Last), _Func);
	return (_Func);
	}
	*/
	cout << "通过for_each的返回值看函数调用次数" << endl;
	show=for_each(v1.begin(), v1.end(), show);//for_each算法函数对象的传递是元素值传递，不是引用传递
	show.print();	
}
void main()
{
	//main01();
	main02();
	system("pause");
}
~~~

~~~c++

#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
using namespace std;
#include<vector>
#include<set>
#include<algorithm>//算法的头文件

template<typename T>
class Show
{
public:
	Show(T &t)
	{
		a = t;
	}
	bool operator()(T &t)
	{
		return t%a == 0;
	}
	void print()
	{
		cout << "a: " << a << endl;
	}
private:
	T a;
};

void main01()
{
	vector<int> v(10);
	for (int i = 0;i < 10;i++)
	{
		v[i] = i + 10;
	}

	/*
	template<class _InIt,
	class _Pr> inline
	_InIt find_if(_InIt _First, _InIt _Last, _Pr _Pred)
	{	// find first satisfying _Pred
	_DEBUG_RANGE_PTR(_First, _Last, _Pred);
	return (_Rechecked(_First,
	_Find_if_unchecked(_Unchecked(_First), _Unchecked(_Last), _Pred)));
	}
	*/
	//find_if返回一个迭代器
	int at = 3;
	vector<int>::iterator it=find_if(v.begin(),v.end(),Show<int>(at));
	if (it == v.end())
	{
		cout << "容器中没有能被3整除的元素" << endl;
	}
	else
	{
		cout << "第一个被3整除的元素为： " << *it << endl;
	}
}
template<typename T>
class AddF
{
public:
	T &operator()(T &t1,T &t2)
	{
		return t = t1 + t2;
	}
	T t;
};
void main02()
{
	//v3=v1+v2
	vector<int> v1, v2, v3(10);
	v1.push_back(10);
	v1.push_back(20);
	v1.push_back(30);
	v2.push_back(40);
	v2.push_back(50);
	v2.push_back(60);
	/*
	_OutIt transform(_InIt1 _First1, _InIt1 _Last1,
	_InTy (&_First2)[_InSize], _OutIt _Dest, _Fn2 _Func)
	{	// transform [_First1, _Last1) and [_First2, ...), array input
	_DEPRECATE_UNCHECKED(transform, _Dest);
	return (_Transform_no_deprecate(_First1, _Last1,
	_Array_iterator<_InTy, _InSize>(_First2), _Dest, _Func));
	}
	//transform返回值为运算结果迭代器的开始位置
	*/
	transform(v1.begin(),v1.end(),v2.begin(),v3.begin(),AddF<int>());//注意：v1.begin(),v1.end()此区域中的元素，必须与v2中元素个数相同，不然报错
	for (vector<int>::iterator it = v3.begin();it != v3.end();it++)
	{
		cout << *it << endl;
	}
}
void Func2(int &t)
{
	cout << t <<" ";
}
bool Compare(const int &t1,const int &t2)//二元谓词
{
	return t1 < t2;
}
void main03()
{
	vector<int> v(10);
	for (int i = 0;i < v.size();i++)
	{
		v[i] = rand();
	}
	for_each(v.begin(),v.end(), Func2);//遍历
	cout << endl;
	/*
	template<class _RanIt,
	class _Pr> inline
	void sort(_RanIt _First, _RanIt _Last, _Pr _Pred)
	{	// order [_First, _Last), using _Pred
	_DEBUG_RANGE(_First, _Last);
	_Sort_unchecked(_Unchecked(_First), _Unchecked(_Last), _Pred);
	}
	*/
	sort(v.begin(),v.end(),Compare);//排序
	for_each(v.begin(), v.end(), Func2);//遍历
	cout << endl;
}
class NotCompare
{
public:
	bool operator()(const string &t1, const string &t2)
	{
		string str1_;
		str1_.resize(t1.size());
		transform(t1.begin(), t1.end(), str1_.begin(), tolower);
		string str2_;
		str2_.resize(t2.size());
		transform(t2.begin(), t2.end(), str2_.begin(), tolower);
		return str1_ < str2_;
	}
};
void main04()
{
	set<string> s1;
	s1.insert("aaa");
	s1.insert("bb");
	s1.insert("cccc");
	set<string>::iterator it = s1.find("BB");//默认区分大小写，修改为不区分大小写
	if (it != s1.end())
	{
		cout << "找到为BB的元素" << endl;
	}
	else
	{
		cout << "未找到BB" << endl;
	}
	set<string,NotCompare> s2;
	s2.insert("aaa");
	s2.insert("bb");
	s2.insert("cccc");
	set<string, NotCompare>::iterator  it2 = s2.find("BB");//默认区分大小写，修改为不区分大小写
	if (it2 != s2.end())
	{
		cout << "找到为BB的元素" << endl;
	}
	else
	{
		cout << "未找到BB" << endl;
	}
}
void main()
{
	//main01();//一元谓词，一元谓词 函数参数1个，函数返回值是bool类型
	//main02();//二元函数对象
	//main03();//二元谓词，函数参数2个，函数返回值是bool类型
	main04();
	system("pause");
}
~~~

### 13.2.2 预定义函数对象和函数适配器

1）预定义函数对象基本概念：标准模板库**STL**提前定义了很多预定义函数对象，#include
<functional> 必须包含。

**2）算术函数对象**
预定义的函数对象支持加、减、乘、除、求余和取反。调用的操作符是与type相关联的实例
加法：plus<Types>
plus<string> stringAdd;
sres = stringAdd(sva1, sva2);
减法：minus<Types>
乘法：multiplies<Types>
除法divides<Tpye>
求余：modulus<Tpye>
取反：negate<Type>

3**）关系函数对象** 

等于equal_to<Tpye>

equal_to<string> stringEqual;

sres = stringEqual(sval1,sval2);

不等于not_equal_to<Type>

大于 greater<Type>

大于等于greater_equal<Type>

小于 less<Type>

小于等于less_equal<Type>

**4）逻辑函数对象**

逻辑与 logical_and<Type>

logical_and<int> indAnd;

ires = intAnd(ival1,ival2);

dres=BinaryFunc( logical_and<double>(),dval1,dval2);

逻辑或logical_or<Type>

逻辑非logical_not<Type>

**函数适配器**

STL中已经定义了大量的函数兑现，但有时候需要对函数返回值进行进一步的简化计算，或者填上多余的参数，不能直接带入算法。函数适配器实现了这一功能，将一种函数对象转化为另一种符合要求的函数对象。函数适配器分为4大类：绑定适配器，组合适配器，指针函数适配器和成员函数适配器。

![64](F:\学习专用\note\pic\64.png)

![63](F:\学习专用\note\pic\63.png)

**2）常用函数函数适配**

标准库提供一组函数适配器，用来特殊化或者扩展一元和二元函数对象。常用适配器是：

1绑定器（binder）: binder通过把二元函数对象的一个实参绑定到一个特殊的值上，将其转换成一元函数对象。C＋＋标准库提供两种预定义的binder适配器：bind1st和bind2nd，前者把值绑定到二元函数对象的第一个实参上，后者绑定在第二个实参上。

2取反器(negator) : negator是一个将函数对象的值翻转的函数适配器。标准库提供两个预定义的ngeator适配器：not1翻转一元预定义函数对象的真值,而not2翻转二元谓词函数的真值。

常用函数适配器列表如下：

bind1st(op, value)

bind2nd(op, value)

not1(op)

not2(op)

~~~c++
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
using namespace std;
#include<vector>
#include<set>
#include<string>
#include<algorithm>//算法的头文件，其中常用到的功能范围涉及到比较、交换、查找、遍历操作、复制、修改、反转、排序、合并等等
#include<functional>//定义了一些模板类，用以声明函数对象

void Func(string &str)
{
	cout << str << " ";
}
//1. plus<type> 预定义函数对象，嫩实现不同类型数据的加法运算,实现了数据类型和算法的分离==>通过函数对象实现
//2. sort()排序函数对象，默认为从小到大排序
//3. equal_to()函数对象 比较两个参数是否相等
//4. count_if()求某数据出现的次数，和equal_to一起使用
//5. 函数适配器的使用，bind2nd（），把预定义函数对象和第二个参数进行绑定
void main01()
{
	/*
	template<class _Ty = void>
	struct plus
	{	// functor for operator+
	typedef _Ty first_argument_type;
	typedef _Ty second_argument_type;
	typedef _Ty result_type;

	constexpr _Ty operator()(const _Ty& _Left, const _Ty& _Right) const
	{	// apply operator+ to operands
	return (_Left + _Right);
	}
	};
	*/
	
	plus<int> add;
	int a = 10;
	int b = 20;
	int c = add(10, 20);
	cout << c << endl;
	plus<string> addstr;
	string s1("aaa");
	string s2 = "bbb";
	string s3 = addstr(s1,s2);
	cout << s3 << endl;

	vector<string> v;
	v.push_back("dsa");
	v.push_back("bbb");
	v.push_back("bbb");
	v.push_back("bbb");
	v.push_back("ccc");
	v.push_back("ddd");
	/*
	template<class _RanIt> inline
	void sort(_RanIt _First, _RanIt _Last)
	{	// order [_First, _Last), using operator<
	_STD sort(_First, _Last, less<>());//从小到大
	}
	*/
	sort(v.begin(), v.end(), greater<>());//从大到小排序
	for_each(v.begin(), v.end(), Func);//遍历容器
	sort(v.begin(),v.end());//排序默认为sort(_First, _Last, less<>()),从小到大
	for_each(v.begin(),v.end(),Func);//遍历容器
	
	//查询bbb出现的次数
	/*
	template<class _Ty = void>
	struct equal_to
	{	// functor for operator==
	typedef _Ty first_argument_type;
	typedef _Ty second_argument_type;
	typedef bool result_type;

	constexpr bool operator()(const _Ty& _Left, const _Ty& _Right) const
	{	// apply operator== to operands
	return (_Left == _Right);
	}
	};
	*/
	//equal_to<>()有两个参数 left参数来自容器，right参数来自s
	//bind2nd函数适配器，把预定义函数对象和第二个参数进行绑定
	//bind1st函数适配器，把预定义函数对象和第一个参数进行绑定
	string s = "bbb";
	int num=count_if(v.begin(),v.end(),bind2nd(equal_to<string>(),s));
	cout << num << endl;
}
template<typename T>
struct CompareOver
{
	CompareOver(T t)
	{
		m_a = t;
	}
	 bool operator()(const T &t)
	{
		 return t>m_a;
	}
private:
	int m_a;
};
void main02()
{
	vector<int> v;
	for (int i = 0;i < 10;i++)
	{
		v.push_back(i + 10);
	}
	for (vector<int>::iterator it = v.begin();it != v.end();it++)
	{
		cout << *it << " ";
	}
	cout << endl;
	//求数组中大于13的个数,通过谓词方式
	int a = 13;
	int number = count_if(v.begin(), v.end(), CompareOver<int>(a));
	cout << "数组中大于13的个数为： " << number << endl;
	//求数组中小于15的个数，通过预定义函数对象
	//less<int>() 有两个函数参数 left参数来自容器的元素，right参数为b(通过适配器绑定)
	int b = 15;
	int number2 = count_if(v.begin(), v.end(), bind2nd(less<int>(), b));
	cout << "数组中小于15的个数为： " << number2 << endl;
	//求奇数的个数
	/*
	template<class _Ty = void>
	struct modulus
	{	// functor for operator%
	typedef _Ty first_argument_type;
	typedef _Ty second_argument_type;
	typedef _Ty result_type;

	constexpr _Ty operator()(const _Ty& _Left, const _Ty& _Right) const
	{	// apply operator% to operands
	return (_Left % _Right);//通过%2方式可求奇数
	}
	};
	*/
	int number3 = count_if(v.begin(), v.end(), bind2nd(modulus<int>(), 2));
	cout << "数组中为奇数的个数： " << number3 << endl;
	//求偶数的个数 取反器（not1）
	int number4 = count_if(v.begin(), v.end(),not1( bind2nd(modulus<int>(), 2)));
	cout << "数组中为偶数的个数： " << number4 << endl;
}
void main()
{
	main01();
	//main02();
	system("pause");
}
~~~

### 13.2.3 STL的容器算法迭代器的设计理念

![65](F:\学习专用\note\pic\65.png)

**结论：**！！！！！！！！！！！！！！！！！！！！！！！

1）  STL的容器通过**类模板**技术，实现数据类型和容器模型的分离。

2）  STL的**迭代器技术**实现了**遍历容器**的统一方法；也为STL的算法提供了统一性

3）  STL算法：通过**函数对象**实现了**自定义数据类型**的算法运算。STL的算法也提供了统一性

核心思想：函数对象本质就是回调函数，回调函数的思想：就是任务的编写者和任务的调用者有效解耦合。函数指针作函数参数

4）  具体例子：transform算法的输入，通过迭代器first和last指向的元算作为输入；通过result作为输出；通过函数对象来做自定义数据类型的运算。

### 13.2.4 常用的遍历算法

#### for_each()

for_each: 用指定函数一次对指定范围内所有元素进行迭代访问,或修改容器中元素

#### transform()

与for_each类似，遍历所有元素，可将结果返回到其他容器中

~~~c++
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
using namespace std;
#include<vector>
#include<set>
#include<string>
#include<algorithm>//算法的头文件，其中常用到的功能范围涉及到比较、交换、查找、遍历操作、复制、修改、反转、排序、合并等等
#include<functional>//定义了一些模板类，用以声明函数对象

void Func(int &t)
{
	t++;
	cout << t << " ";
}
void play(vector<int> &obj)
{
	for (vector<int>::iterator it = obj.begin();it != obj.end();it++)
	{
		cout << *it << " ";
	}
	cout << endl;
}
struct show
{
	void operator()(int t)
	{
		cout << t << " ";
	}
};
//for_each()
/*
template<class _InIt,
class _Fn1> inline
_Fn1 for_each(_InIt _First, _InIt _Last, _Fn1 _Func)
{	// perform function for each element
_DEBUG_RANGE_PTR(_First, _Last, _Func);
_For_each_unchecked(_Unchecked(_First), _Unchecked(_Last), _Func);
return (_Func);
}
*/
void main01()
{
	vector<int> v;
	for (int i = 0;i < 10;i++)
	{
		v.push_back(i + 10);
	}
	for_each(v.begin(),v.end(), Func);//调用普通函数
	cout << endl;
	for_each(v.begin(), v.end(), show());//调用函数对象
	cout << endl;
	play(v);//使用迭代器遍历
}
int increase(int &t)
{
	t = t + 10;
	cout << t << " ";
	return t;
}
//transform()
/*
template<class _InIt,
class _OutIt,
class _Fn1> inline
_OutIt transform(_InIt _First, _InIt _Last,
_OutIt _Dest, _Fn1 _Func)
{	// transform [_First, _Last) with _Func
_DEPRECATE_UNCHECKED(transform, _Dest);
return (_Transform_no_deprecate(_First, _Last, _Dest, _Func));
}
*/
void main02()
{
	vector<int> v;
	for (int i = 0;i < 10;i++)
	{
		v.push_back(i + 10);
	}
	//使用回调函数
	transform(v.begin(), v.end(), v.begin(), increase);
	cout << endl;
	//使用预定义的函数对象 取反
	transform(v.begin(), v.end(), v.begin(), negate<int>());
	play(v);
	//使用函数适配器
	vector<int> v1(v.size());
	transform(v.begin(),v.end(),v1.begin(),bind2nd(multiplies<int>(),10));//把结果返回给v1容器
	play(v1);
}
void main()
{
	//main01();
	main02();
	system("pause");
}
~~~

区别：

for_each()   速度快  不灵活

transform()  速度慢   非常灵活

结论：

一般情况下，for_each所使用的函数对象，参数是引用，没有返回值

transform所使用的函数对象，参数一般不使用引用，而且需要有返回值

### 13.2.5 常用的查找算法

~~~c++
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
using namespace std;
#include<vector>
#include<set>
#include<string>
#include<algorithm>//算法的头文件，其中常用到的功能范围涉及到比较、交换、查找、遍历操作、复制、修改、反转、排序、合并等等
#include<functional>//定义了一些模板类，用以声明函数对象
//adjacent_find() 在iterator对标识元素范围内，查找一对相邻重复元素，找到则返回指向这对元素的第一个元素的迭代器。否则返回past-the-end。
void main01()
{
	vector<int> v;
	v.push_back(2);
	v.push_back(1);
	v.push_back(4);
	v.push_back(3);
	v.push_back(2);
	v.push_back(2);
	v.push_back(6);
	vector<int>::iterator it = adjacent_find(v.begin(),v.end());
	if (it!=v.end())
	{
		cout << "找到相邻重复元素: " << *it<<endl;
	}
	else
	{
		cout << "没有找到相邻重复元素: " << endl;
	}
	int index = distance(v.begin(),it);
	cout << index << endl;
}
//binary_search 二分查找法，在有序序列中查找value,找到则返回true。注意：在无序序列中，不可使用 
//优点：速度快 在一个数组含有1024个有序元素中查找，只需要查找10次
void main02()
{
	vector<int> v;
	for (int i = 0;i < 10000;i++)
	{
		v.push_back(i);
	}
	bool sa=binary_search(v.begin(),v.end(),2334);
	if (sa == true)
	{
		cout << "找到2334" << endl;
	}
	else
	{
		cout << "未找到" << endl;
	}
}
//count()  利用等于操作符，把标志范围内的元素与输入值比较，返回相等的个数。
void main03()
{
	vector<int> v;
	for (int i = 0;i < 10000;i++)
	{
		v.push_back(i);
	}
	int num = count(v.begin(), v.end(), 2345);
	cout << num << endl;
}
bool great(int num)
{
	if (num > 23)
	{
		return true;
	}
	else
	{
		return false;
	}
}
//count_if() 
void main04()
{
	vector<int> v;
	for (int i = 0;i < 100;i++)
	{
		v.push_back(i);
	}
	int num = count_if(v.begin(), v.end(), great);
	cout << num << endl;
}
//find() find_if()
void main05()
{
	vector<int> v;
	for (int i = 0;i < 100;i++)
	{
		v.push_back(i);
	}
	vector<int>::iterator it = find(v.begin(),v.end(),40);
	cout << "*it: " << *it << endl;
	//第一个大于23的位置
	vector<int>::iterator it2 = find_if(v.begin(), v.end(), great);
	cout << "*it2: " << *it2 << endl;
}
void main()
{
	//main01();
	//main02();
	//main03();
	//main04();
	main05();
	system("pause");
}
~~~

### 13.2.6 常用的排序算法

~~~c++
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
using namespace std;
#include<vector>
#include<set>
#include<string>
#include<deque>
#include<algorithm>//算法的头文件，其中常用到的功能范围涉及到比较、交换、查找、遍历操作、复制、修改、反转、排序、合并等等
#include<functional>//定义了一些模板类，用以声明函数对象
template<typename T>//打印模板函数
void print(T t)
{
	for (T::iterator it = t.begin();it != t.end();it++)
	{
		cout << *it<<" ";
	}
	cout << endl;
}
//merge() 合并两个有序序列，存放到另一个序列
void main01()
{
	deque<int> d1,d2;
	d1.push_back(1);
	d1.push_back(3);
	d1.push_back(5);
	d1.push_back(8);
	d2.push_back(4);
	d2.push_back(6);
	d2.push_back(9);
	d2.push_back(14);
	d2.push_back(17);
	d2.push_back(19);
	deque<int> d3(d1.size() + d2.size());
	merge(d1.begin(),d1.end(),d2.begin(),d2.end(),d3.begin());
	print<deque<int>>(d3);//1 3 4 5 6 ...
}
class Teacher
{
public:
	int age;
	char name[32];
	Teacher(int a,char *b)
	{
		strcpy(name,b);
		age = a;
	}
	friend ostream &operator<<(ostream &out,Teacher &obj);
};
ostream &operator<<(ostream &out, Teacher &obj)
{
	out << "age: " << obj.age << "name: " << obj.name << endl;
	return out;
}
bool compare(Teacher &obj1,Teacher &obj2)
{
	return obj1.age < obj2.age;
}
//sort() 以默认升序的方式重新排列指定范围内的元素。若要改排序规则，可以输入比较函数。
void main02()
{
	vector<Teacher> v;
	Teacher t1(20,"a");
	Teacher t2(30, "bb");
	Teacher t3(25, "vv");
	v.push_back(t1);
	v.push_back(t2);
	v.push_back(t3);
	//根据自定义函数对象 进行自定义数据类型的排序
	sort(v.begin(),v.end(), compare);
	print<vector<Teacher>>(v);
}
//random_shuffle() 随机排序
void main03()
{
	vector<int> v;
	for (int i = 0;i < 10;i++)
	{
		v.push_back(i);
	}
	print<vector<int>>(v);
	random_shuffle(v.begin(), v.end());
	print<vector<int>>(v);
	string s = "dasavzfa";
	random_shuffle(s.begin(), s.end());
	print<string>(s);
}
void main()
{
	//main01();
	//main02();
	main03();
	system("pause");
}
~~~

### 13.2.7 常用的拷贝和替换算法

~~~c++
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
using namespace std;
#include<vector>
#include<set>
#include<string>
#include<deque>
#include<algorithm>//算法的头文件，其中常用到的功能范围涉及到比较、交换、查找、遍历操作、复制、修改、反转、排序、合并等等
#include<functional>//定义了一些模板类，用以声明函数对象
template<typename T>//打印模板函数
void print(T t)
{
	for (T::iterator it = t.begin();it != t.end();it++)
	{
		cout << *it << " ";
	}
	cout << endl;
}
//replace(beg,end,oldValue,newValue):将指定范围内的所有等于oldValue的元素替换成newValue
void main01()
{
	vector<int> v;
	v.push_back(2);
	v.push_back(3);
	v.push_back(5);
	v.push_back(3);
	v.push_back(2);
	print<vector<int>>(v);
	replace(v.begin(),v.end(),2,20);
	print<vector<int>>(v);
}
bool Geater(int num)
{
	return num > 2;
}
//replace_if : 将指定范围内所有操作结果为true的元素用新值替换。
//用法举例：
//replace_if(vecIntA.begin(), vecIntA.end(), GreaterThree, newVal)
//其中 vecIntA是用vector<int>声明的容器
//GreaterThree 函数的原型是 bool GreaterThree(int iNum)
void main02()
{
	vector<int> v;
	v.push_back(2);
	v.push_back(3);
	v.push_back(5);
	v.push_back(3);
	v.push_back(2);
	print<vector<int>>(v);
	replace_if(v.begin(),v.end(), Geater,10);//将大于2的值都改为10
	print<vector<int>>(v);
}
//swap() 交换两个容器的元素
void main03()
{
	vector<int> v;
	v.push_back(2);
	v.push_back(3);
	v.push_back(5);
	v.push_back(3);
	v.push_back(2);
	vector<int> v1;
	v1.push_back(6);
	v1.push_back(6);
	v1.push_back(6);
	v1.push_back(6);
	v1.push_back(6);
	swap(v,v1);
	print<vector<int>>(v);
}
void main()
{
	//main01();
	//main02();
	main03();
	system("pause");
}
~~~

### 13.2.8 常用集合运算

~~~c++
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
using namespace std;
#include<vector>
#include<set>
#include<string>
#include<deque>
#include<algorithm>//算法的头文件，其中常用到的功能范围涉及到比较、交换、查找、遍历操作、复制、修改、反转、排序、合并等等
#include<functional>//定义了一些模板类，用以声明函数对象
template<typename T>//打印模板函数
void print(T t)
{
	for (T::iterator it = t.begin();it != t.end();it++)
	{
		cout << *it << " ";
	}
	cout << endl;
}
//set_union:  构造一个有序序列，包含两个有序序列的并集。
//set_intersection : 构造一个有序序列，包含两个有序序列的交集。
//set_difference : 构造一个有序序列，该序列保留第一个有序序列中存在而第二个有序序列中不存在的元素。
void main()
{
	vector<int> v;
	v.push_back(2);
	v.push_back(3);
	v.push_back(5);
	v.push_back(3);
	v.push_back(2);
	vector<int> v2;
	v2.push_back(1);
	v2.push_back(3);
	v2.push_back(4);
	v2.push_back(98);
	v2.push_back(2);
	vector<int> v3;
	v3.resize(v.size()+v2.size());
	set_union(v.begin(), v.end(), v2.begin(), v2.end(), v3.begin());
	print<vector<int>>(v3);
	system("pause");
}
~~~

 



































































