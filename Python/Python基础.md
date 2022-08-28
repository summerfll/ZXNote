# Python基础

## 一、数据类型

Python的数据类型有六个：

- Numbers：数字类型，包括 `int（有符号整数）`、`long（长整型）`、`float(浮点型)`、`complex(复数)`
- Boolean：布尔类型，包括`True`、`False`
- String：字符串类型
- List：列表
- Tuple：元组
- Dictionary：字典

### 1、数字类型

Python3 支持 int、float、bool、complex（复数）。在Python 3里，只有一种整数类型 int，表示为长整型，没有 python2 中的 Long。内置的 type() 函数可以用来查询变量所指的对象类型。   

```python
a, b, c, d = 20, 5.5, True, 4+3j
print(type(a), type(b), type(c), type(d))
```

输出类型结果如下：

```python
<class 'int'> <class 'float'> <class 'bool'> <class 'complex'>
```

### 2、字符串类型

Python中的字符串用单引号 ' 或双引号 " 括起来，同时使用反斜杠 \ 转义特殊字符。  

```python
str = 'hello'

print(str)				#hello
print(str[0])			#h
print(str[0:-1])		#hell
print(str[0:2])			#he
print(str[2:])			#llo
print(str * 2)			#hellohello
print(str + " ruyue")	#hello ruyue
```

#### 2.1 字符串常见方法

查找方法`find()`和`index()`：查看参数是否在字符串中，是则返回所在索引，否则返回-1

```python
str = 'hello'
print(str.find('h'))                # 0
print(str.find('h', 1, 5))          # -1 后续2个可选参数为起始位置、结束位置

# index() 方法用法与find()一致，但是在没有找到数据时报错
```

次数方法`count()`：获取参数在字符串中存在的个数

```python
str = 'hello'
print(str.count('h'))               # 1
print(str.count('h', 1, 5))        # 0
```

替换方法`replace()`：

```python
str1 = 'hello'
print(str1.replace('l', 'k'))       # hekko

str2 = 'hello'
print(str2.replace('l', 'k', 1))    # heklo 第三个参数代表替换次数
```

分割方法`split()`：

```python
str = 'he ll o'
print(str.split(' '))       # ['he', 'll', 'o']
```

其他常见方法：

```python
rfind()             类似find，从右侧查找
rindex()            类似index，从右侧查找
partition()         将字符串分割为三部分：参数前，参数，参数后
capitalize()        首字符大写
title()             每个单词首字母大写
startswith()        检查字符串是否以参数开头，返回布尔
endswith()          检查字符串是否以参数结尾，返回布尔
lower()             所有大写转小写
upper()             所有小写转大写
lstrip()            清除左边空白
rstrip()            清除右边空白
```

### 3、列表

~~~python
list = ['a', 'b', 'c']

# 添加元素
list.append('d')            # 尾部添加
list.extend(['e', 'f'])     # 逐个添加
list.insert(6, 'g')         # 指定位置添加

# 删除元素
del list[0]                 # 删除指定位置
list.pop()                  # 删除末尾元素
list.remove('g')            # 删除具体元素

# 修改元素
list[0] = 'A'

# 查找元素,index/count 等方法与字符串的对应方法用法一致。也可以使用循环，配合 in 与 not in 查找
list.index('a', 1, 3)   # 左闭右开区间

# 排序
list.sort(reverse=True)             # 默认由小到大排序，添加False参数，则由大到小排列

# 逆置
list.reverse()
~~~

### 4、元组

元组（tuple）与列表类似，不同之处在于元组是不可变的。  

~~~python
tuple = ( 'abcd', 786 , 2.23, 'overnote', 70.2  )
tinytuple = (123, 'overnote')
 
print (tuple)             # 输出完整元组
print (tuple[0])          # 输出元组的第一个元素
print (tuple[1:3])        # 输出从第二个元素开始到第三个元素
print (tuple[2:])         # 输出从第三个元素开始的所有元素
print (tinytuple * 2)     # 输出两次元组
print (tuple + tinytuple) # 连接元组
~~~

### 5、字典

~~~python
dict["key"]                 # 访问元素
dict.get("key")             # 访问元素
dict["key"] = "value"       # 修改元素，若key不存在，则新增键值对
del dict["key"]             # 删除元素
del dict                    # 删除字典
dict.clear()                # 清空字典
len(dict)                   # 字段键值对个数
dict.keys()                 # key列表
dict.values()               # 值列表
dict.items()                # 将字典转换为列表 [('name', 'overnote'), ('code', 1)]
~~~

### 6、数据类型转换

~~~python
int(x [,base ])         将x转换为一个整数
float(x)	            将x转换为一个浮点数
complex(real [,imag ])	创建一个复数，real为实部，imag为虚部
str(x)	                将对象 x 转换为字符串
repr(x)	                将对象 x 转换为表达式字符串
eval(str)	            用来计算在字符串中的有效Python表达式,并返回一个对象
tuple(s)	            将序列 s 转换为一个元组
list(s)	                将序列 s 转换为一个列表
chr(x)	                将一个整数转换为一个Unicode字符
ord(x)	                将一个字符转换为它的ASCII整数值
hex(x)	                将一个整数转换为一个十六进制字符串
oct(x)	                将一个整数转换为一个八进制字符串
bin(x)	                将一个整数转换为一个二进制字符串
~~~

## 二、流程控制

### 1、条件控制

~~~python
    if score>=90 and score<=100:
        print('本次考试，等级为A')
    elif score>=80 and score<90:
        print('本次考试，等级为B')
    elif score>=70 and score<80:
        print('本次考试，等级为C')
    elif score>=60 and score<70:
        print('本次考试，等级为D')
    elif score>=0 and score<60:
        print('本次考试，等级为E')
~~~

### 2、循环语句

#### 2.1 for循环

```python
languages = ["C", "C++", "Perl", "Python"] 
for x in languages:
    print (x)
```

#### 2.2 for range

```python
for i in range(5):
    print(i)
```

#### 2.3 while循环

```python
n = 100
 
sum = 0
counter = 1
while counter <= n:
    sum = sum + counter
    counter += 1
 
print("1 到 %d 之和为: %d" % (n,sum))
```

## 三、函数

### 3.1 缺省参数

如果在调用参数时，缺省参数的值没有传入，则取其默认值：

```python
def show(name, age=35):
    print("name: %s" % name)
    print("age %d" % age)

show(name="miki")  # 在函数执行过程中 age去默认值35
```

注意：带有默认值的参数一定要位于参数列表的最后面。  

### 3.2 不定长参数

参数的数量无法预估，则可以使用不定长参数：

```python
def fn(a, b, *args, **kwargs):# *args：单value **kwargs：key=value类型参数
    for key, value in kwargs.items():
        print(f"key={key} value={value}")
    i = 0
    for a in args:
        print(f"args{i} = {a}")
        i += 1
    

fn(1, 2, 3, 4, 5, m=6, n=7, p=8)      
# key=m value=6
# key=n value=7
# key=p value=8
# args0 = 3
# args1 = 4
# args2 = 5
```

注意：缺省参数需要放在 *args 后面， 但如果有**kwargs的话，该参数必须是放在最后的。

## 四、面向对象

### 4.1、类与对象

~~~python
# 创建类
class Person(object):

    #  默认初始化方法，系统自动调用，不需要初始化一些数据则可以生路
    def __init__(self):    
        self.surename = "李"
    # 删除对象时，默认调用 `__del__()`方法：
	def __del__(self):
        print("__del__方法被调用")
        
    def hello(self):
        print("my surname is %s" % self.surename)

    def say(self):
        print("my age is %d" % self.age)


# 创建对象
p = Person()
p.hello()

# 给对象添加新的属性
p.age = 13
p.say()
~~~

### 4.2、封装

~~~python
# 在属性名和方法名 前面 加上两个下划线 __ 即成为私有权限
class Prentice(School, Master):
    def __init__(self):
        self.kongfu = "猫氏煎饼果子配方"
        # 私有属性，可以在类内部通过self调用，但不能通过对象访问
        self.__money = 10000  

    # 私有方法，可以在类内部通过self调用，但不能通过对象访问
    def __print_info(self):
        print(self.kongfu)
        print(self.__money)
~~~

### 4.3、继承

~~~python
# 父类
class A(object):
    def __init__(self):
        self.num = 10

    def print_num(self):
        print(self.num + 10)
# 子类
class B(A):
    pass


b = B()
print(b.num) 
b.print_num() #20
~~~

#### 4.3.1、多继承

多继承可以继承多个父类，也继承了所有父类的属性和方法，如果多个父类中有同名的 属性和方法，则默认使用第一个父类的属性和方法

~~~python
class Master(object):
    def __init__(self):
        self.kongfu = "古法煎饼果子配方"  # 实例变量，属性

    def make_cake(self):                    # 实例方法，方法
        print("[古法] 按照 <%s> 制作了一份煎饼果子..." % self.kongfu)

    def dayandai(self):
        print("师傅的大烟袋..")

class School(object):
    def __init__(self):
        self.kongfu = "现代煎饼果子配方"

    def make_cake(self):
        print("[现代] 按照 <%s> 制作了一份煎饼果子..." % self.kongfu)

    def xiaoyandai(self):
        print("学校的小烟袋..")

class Prentice(Master, School):  # 多继承，继承了多个父类（Master在前）
    pass

damao = Prentice()
print(damao.kongfu) # 执行Master的属性
damao.make_cake() # 执行Master的实例方法
~~~

### 4.4、重写

~~~python
class Master(object):
    def __init__(self):
        self.kongfu = "古法煎饼果子配方" 

    def make_cake(self): 
        print("[古法] 按照 <%s> 制作了一份煎饼果子..." % self.kongfu)


class School(object):
    def __init__(self):
        self.kongfu = "现代煎饼果子配方"

    def make_cake(self):
        print("[现代] 按照 <%s> 制作了一份煎饼果子..." % self.kongfu)


class Prentice(School, Master):  # 多继承，继承了多个父类
    def __init__(self):
        self.kongfu = "猫氏煎饼果子配方"

    def make_cake(self):
        print("[猫氏] 按照 <%s> 制作了一份煎饼果子..." % self.kongfu)


# 如果子类和父类的方法名和属性名相同，则默认使用子类的
# 叫 子类重写父类的同名方法和属性
damao = Prentice()
print(damao.kongfu) # 子类和父类有同名属性，则默认使用子类的
damao.make_cake() # 子类和父类有同名方法，则默认使用子类的
~~~

### 4.5 super()使用

~~~python
class Master(object):
    def __init__(self):
        self.kongfu = "古法煎饼果子配方"  # 实例变量，属性

    def make_cake(self):  # 实例方法，方法
        print("[古法] 按照 <%s> 制作了一份煎饼果子..." % self.kongfu)


# 父类是 Master类
class School(Master):
    def __init__(self):
        self.kongfu = "现代煎饼果子配方"

    def make_cake(self):
        print("[现代] 按照 <%s> 制作了一份煎饼果子..." % self.kongfu)
        super().__init__()  # 执行父类的构造方法
        super().make_cake()  # 执行父类的实例方法


# 父类是 School 和 Master
class Prentice(School, Master):  # 多继承，继承了多个父类
    def __init__(self):
        self.kongfu = "猫氏煎饼果子配方"

    def make_cake(self):
        self.__init__()  # 执行本类的__init__方法，做属性初始化 self.kongfu = "猫氏...."
        print("[猫氏] 按照 <%s> 制作了一份煎饼果子..." % self.kongfu)

    def make_all_cake(self):
        # 方式1. 指定执行父类的方法（代码臃肿）
        # School.__init__(self)
        # School.make_cake(self)
        #
        # Master.__init__(self)
        # Master.make_cake(self)
        #
        # self.__init__()
        # self.make_cake()

        # 方法2. super() 带参数版本，只支持新式类
        # super(Prentice, self).__init__() # 执行父类的 __init__方法 
        # super(Prentice, self).make_cake()
        # self.make_cake()

        # 方法3. super()的简化版，只支持新式类
        super().__init__()  # 执行父类的 __init__方法 
        super().make_cake()  # 执行父类的 实例方法
        self.make_cake()  # 执行本类的实例方法
        
damao = Prentice()
damao.make_cake()
damao.make_all_cake()
~~~

### 4.6、多态

Python的多态，就是弱化类型，重点在于对象参数是否有指定的属性和方法，如果有就认定合适，而不关心对象的类型是否正确

~~~python
class F1(object):
    def show(self):
        print('F1.show')

class S1(F1):
    def show(self):
        print('S1.show')

class S2(F1):
    def show(self):
        print('S2.show')

def Func(obj):  
    # python是弱类型，即无论传递过来的是什么，obj变量都能够指向它，这也就没有所谓的多态了（弱化了这个概念）
    print(obj.show())

s1_obj = S1()
Func(s1_obj) 

s2_obj = S2()
Func(s2_obj)
~~~

## 五、异常

~~~python
try:
    print('-----test--1---')
    open('123.txt','r') # 如果123.txt文件不存在，那么会产生 IOError 异常
    print('-----test--2---')
    print(num)# 如果num变量没有定义，那么会产生 NameError 异常

except (IOError,NameError): 
    #如果想通过一次except捕获到多个异常可以用一个元组的方式

else:
    print('没有捕获到异常，真高兴')
~~~

~~~python
import time
try:
    f = open('test.txt')
    try:
        while True:
            content = f.readline()
            if len(content) == 0:
                break
            time.sleep(2)
            print(content)
    except:
        #如果在读取文件的过程中，产生了异常，那么就会捕获到
        #比如 按下了 ctrl+c
        pass
    finally:
        #finally语句可以在异常发生后强制执行：
        f.close()
        print('关闭文件')
except:
    print("没有这个文件")
~~~

### 5.1、异常抛出

raise语句来引发一个异常。异常/错误对象必须有一个名字，且它们应是Error或Exception类的子类

~~~python
class ShortInputException(Exception):
    '''自定义的异常类'''
    def __init__(self, length, atleast):
        #super().__init__()
        self.length = length
        self.atleast = atleast

def main():
    try:
        s = input('请输入 --> ')
        if len(s) < 3:
            # raise引发一个你定义的异常
            raise ShortInputException(len(s), 3)
    except ShortInputException as result:#x这个变量被绑定到了错误的实例
        print('ShortInputException: 输入的长度是 %d,长度至少应是 %d'% (result.length, result.atleast))
    else:
        print('没有异常发生.')

main()
~~~

## 六、python模块

~~~python
import math
print(math.sqrt(2))
------------
from math import *来实现
print(sqrt(2))
~~~

### 6.1、包

多个相关联的模块组成一个包，以便于维护和使用，同时能有限的避免命名空间的冲突

~~~python
package
  |- subpackage1
      |- __init__.py
      |- a.py
  |- subpackage2
      |- __init__.py
      |- b.py
  |- main.py
~~~

导入方式

```python
import subpackage1.a # 将模块subpackage.a导入全局命名空间，例如访问a中属性时用subpackage1.a.attr
from subpackage1 import a #　将模块a导入全局命名空间，例如访问a中属性时用a.attr_a
from subpackage.a import attr_a # 将模块a的属性直接导入到命名空间中，例如访问a中属性时直接用attr_a
```

当我们导入这个包的时候，__init__.py文件自动运行。帮我们导入了这么多个模块，我们就不需要将所有的import语句写在一个文件里了，也可以减少代码量。
不需要一个个去导入module了。
__init__.py 中还有一个重要的变量，叫做 __all__。我们有时会使出一招“全部导入”：

from PackageName import *

~~~python
#subpackage1/__init__.py
import os
__all__ = ['os','a']

#subpackage1/a.py
def func_a():
    print("a")
    
#subpackage2/__init__.py
import sys
__all__ = ['sys','b']

#subpackage2/a.py
def func_b():
    print("b")
    
#main.py
from subpackage2 import *
from subpackage1 import *
print(a.func_a())
print(b.func_b())
print(os.path)
print(sys.path)
~~~

