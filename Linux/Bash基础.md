# Bash基础

## 1、标准输入、输出与标准错误输出

| **文件描述符** | **名称** | **通用缩写** | **默认值** |
| -------------- | -------- | ------------ | ---------- |
| 0              | 标准输入 | stdin        | 键盘       |
| 1              | 标准输出 | stdout       | 屏幕       |
| 2              | 标准错误 | stderr       | 屏幕       |

### 1.1 输出重定向 

| 语法   | 说明                                                   |
| ------ | ------------------------------------------------------ |
| >      | 把标准输出重定向到一个新文件，”>” 会覆盖原有的内容。   |
| >>     | 把标准输出重定向到一个文件中，不覆盖原有的内容(追加)。 |
| 2 >    | 把标准错误重定向到一个文件中                           |
| 2 >>   | 把标准错误重定向到一个文件中(追加)                     |
| 2 > &1 | 把标准输出和错误重定向到一个文件(追加)                 |

### 1.2 输入重定向

| 语法         | 说明                                      |
| ------------ | ----------------------------------------- |
| <            | filename文件作为标准输入                  |
| << delimiter | 从标准输入中读入，知道遇到delimiter分界符 |

### 1.3 绑定重定向

| **语法** | **说明**                        |
| -------- | ------------------------------- |
| > &m     | 把标准输出重定向到文件描述符m中 |
| < &-     | 关闭标准输入                    |
| > &-     | 关闭标准输出                    |

## 2、变量

### 2.1 环境变量

通过使用printenv可以显示当前的环境变量

~~~shell
[root@IDC-D-1699 ~]# printenv
HOSTNAME=IDC-D-1699
TERM=xterm
SHELL=/bin/bash
HISTSIZE=1000
SSH_CLIENT=111.200.23.36 31752 22
QTDIR=/usr/lib64/qt-3.3
QTINC=/usr/lib64/qt-3.3/include
SSH_TTY=/dev/pts/3
USER=root
MAIL=/var/spool/mail/root
PATH=/usr/local/java/jdk1.8.0_101/bin:/usr/lib64/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
PWD=/root
JAVA_HOME=/usr/local/java/jdk1.8.0_101
LANG=zh_CN.UTF-8
HISTCONTROL=ignoredups
SHLVL=1
HOME=/root
LOGNAME=root
~~~

使用$可获取变量名称

~~~shell
[root@IDC-D-1699 ~]# a=$PWD;echo $a
/root
[root@IDC-D-1699 ~]# a=$PATH;echo $a
/usr/local/java/jdk1.8.0_101/bin:/usr/lib64/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
~~~

通过``符号和$()也可以执行Linux命令并获取运行结果

~~~shell
[root@IDC-D-1699 ~]# a=`pwd`;echo $a
/root
[root@IDC-D-1699 ~]# a=$(pwd);echo $a
/root
[root@IDC-D-1699 ~]# a=`path`;echo $a
/usr/local/java/jdk1.8.0_101/bin:/usr/lib64/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
~~~

### 2.2 本地变量

Shell不需要明确定义类型，事实上Shell变量的值都是字符串，比如我们定义var=45，其实var的值是字符串而非整数，shell变量不需要事先定义就可以使用，如果使用没有定义的变量，这字符串取值为空字符串。

变量名称=”变量Value”，“=”的两边不能有空格，否则shell解释成名称和命令参数。

获取变量使用 $变量名称

~~~shell
[root@IDC-D-1699 ~]# a="abc";echo $a
abc
~~~

#### 2.2.1 内容代换

可以使用*、?、[]对内容代换

| **匹配符** | **说明**                           |
| ---------- | ---------------------------------- |
| *          | 匹配0个多个任意字符                |
| ?          | 匹配一个任意字符                   |
| []         | 匹配方括号中任意一个字符的一次出现 |

#### 2.2.2 命令代换

将命令替换为命令输出，所有的shell支持使用反引号的方式进行命令替换，命令替换可以嵌套，需要注意的是如果使用反引号的形式，在内部反引用前必须使用反斜杠转移

| **匹配符** | **说明**        |
| ---------- | --------------- |
| ``  |例如echo ``pwd` `|
| $()        | 例如echo $(pwd) |

#### 2.2.3 算术代换

| **匹配符** | **说明**             |
| ---------- | -------------------- |
| $(())      | 例如 echo $((4 + 6)) |

## 3、符号

### 3.1 转义字符

'\’用作转义字符

### 3.2 单引号

单引号内的所有字符都保持它本身字符的意思，而不会被bash进行解释

### 3.3 双引号

除了$、``、/外，双引号内所有的字符保持字符本身的含义

## 4、逻辑判断

### 4.1 if

在shell中用if，then，elif，else，fi这几条命令实现分支控制，这种流程控制语句本质上也是由若干个逻辑判断组成，需要注意的是

- if/then结束都离不开fi
- if和[]注意用空格隔开，]后面紧跟;
- []内的条件与都有一个空格隔开

~~~shell
if [ -f $a ];then 
        echo "hello world!" 
fi
~~~

### 4.2 case

case结构用于多种情况的条件判断，类似于其它语言的switch/case，但从语法结构上有很大的不同，常用格式

~~~shell
case 字符串 in
    模式)
        语句
        ;;
    模式2 | 模式3)
         语句
        ;;
    *)
     默认执行的 语句
         ;;
esac
~~~

~~~shell
#!/bin/bash
read -p "请输入要查查询的区号:" num
case $num in
   *)echo -n "中国";;&
     03*)echo -n "河南省";;&
        ??71)echo "郑州市";;
        ??72)echo "安阳市";;
        ??73)echo "新乡市";;
        ??73)echo "许昌市";;
     07*)echo -n "江西省";;&
        ??91)echo "南昌市";;
        ??92)echo "九江市";;
        ??97)echo "赣州市";;
esac
~~~

注意

当程序指定到条件语句;;&时，不会停止，直到执行到;;esac

不管是if还是case，他们的结尾都很有意思，if的结尾是fi，而case的结尾是easc，首位和尾部正好相反

## 5、循环

### 5.1 for

~~~shell
#!/bin/bash
for i in $(ls)
do
    echo item: $i
done
~~~

### 5.2 while

~~~shell
#!/bin/bash
num=$1
while [ $num -le 20 ]
do
	echo num:$num
	num=$(($num+1))
done
~~~

### 5.3 until

~~~shell
#!/bin/bash
num=$1
until [ $num -lt 20 ]
do
	echo num:$num
	let num=num-1
done
~~~

## 6、比较运算

### 6.1 比较符

| **比较符**      | **说明**                            | **举例**               |
| --------------- | ----------------------------------- | ---------------------- |
| -e filename     | 如果filename存在，则为真            | [ -e /var/log/syslog ] |
| -d filename     | 如果filename为目录，则为真          | [ -d /tmp/mydir ]      |
| -f filename     | 如果filename常规文件，则为真        | [ -f /usr/bin/grep ]   |
| -L filename     | 如果filename为符号链接，则为真      | [ –L /usr/bin/grep ]   |
| -r filename     | 如果filename可读，则为真            | [ –r /var/log/syslog ] |
| -w filename     | 如果filename可写，则为真            | [ –w /varmytmp.txt ]   |
| -x filename     | 如果filename可执行，则为真          | [ –x /usr/bin/grep ]   |
| -s filename     | 如果filename不是空白文件，则为真    |                        |
| -u filename     | 如果filename有SUID属性，则为真      |                        |
| -g filename     | 如果filename有SGID属性，则为真      |                        |
| -k filename     | 如果filename有stickybit属性，则为真 |                        |
| file1 –nt file2 | 如果file1比file2新，则为真          |                        |
| file1 –ot file2 | 如果file1比file2旧，则为真          |                        |

### 6.2 字符串比较运算符

| **比较符**   | **说明**                     |
| ------------ | ---------------------------- |
| -z string    | 如果string长度为零，则为真   |
| -n string    | 如果string长度不为零，则为真 |
| str1 = str2  | 如果str1与str2相同，则为真   |
| str1 != str2 | 如果str1与str2不相同，则为真 |

### 6.3 算数比较符

| **比较符** | **说明**   |
| ---------- | ---------- |
| -eq        | 等于       |
| -ne        | 不等于     |
| -lt        | 小于       |
| -le        | 小于或等于 |
| -gt        | 大于       |
| -ge        | 大于或等于 |