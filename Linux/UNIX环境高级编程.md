UNIX环境高级编程

# 一. UNIX基础知识

## 1.UNIX体系结构

从严格意义上，可将操作系统定义为一种软件，它控制计算机硬件资源，提供程序运行环境，我们称这种软件为**内核**。

内核的接口称为**系统调用**。公用函数库构建在系统调用接口之上，应用软件既可使用公用数据库，也可使用系统调用。shell是一种特殊的应用程序，它为运行其他应用程序提供一个接口。

![39](F:\学习专用\学习笔记\图片\39.png)

广义上，操作系统包括内核和一些其他软件。这些软件包括系统实用程序、应用软件、shell以及公用函数库。	

## 2.shell

shell是一个命令行解释器，它读取用户输入，然后执行命令。用户通过用终端（交互式shell），有时通过文件（称为shell脚本，shell script）向shell进行输入。

## 3.文件和目录

### 3.1文件系统

UNIX文件系统是目录和文件组成的一个层次结构，目录的起点称为**根**，其名字是一个字符\。目录是一个包含许多目录项的文件，在逻辑上，认为每个目录项都包含一个文件名，同时还包含说明文件属性的信息。

### 3.2 文件名

目录中的各个名字称为文件名。

创建新目录时回自动创建两个文件名：.（点）和..（点-点）   点指当前目录，点-点指父目录

### 3.3 路径名

一个或多个以斜线分隔的文件名序列构成路径名

绝对路径：是从盘符开始的路径，形如C:\windows\system32\cmd.exe

相对路径：是从当前路径开始的路径，假如当前路径为C:\windows
要描述上述路径，只需输入
system32\cmd.exe

### 3.4 工作目录

每个进程都有一个工作目录，有时称其为当前工作目录。所有相对路径名都从工作目录开始。进程可以用chdir函数更改工作目录。

## 4.程序和进程

### 4.1 程序

程序是存放在磁盘上，处于某个目录中的一个可执行文件。使用6个exec函数中的一个由内核将程序读入存储器，并使其执行。

### 4.2 进程和进程ID

程序的执行实例称为进程，某些操作系统用任务表示正被执行的程序。UNIX系统确保每个进程都有一个唯一的数字标识符，称为进程ID。

### 4.3 进程控制

进程控制的主要函数：

fork、exec和waitpid

### 4.4 线程和线程ID

一个进程只有一个控制线程，同一时刻只执行一组机器指令。对于某些问题，不同部分使用一个控制线程，可使问题简化。

在一个进程内的所有线程共享同一地址空间、文件描述符、栈以及与进程相关的属性。因为他们能访问同一存储区，所有各线程在访问共享数据时需要采取同步措施以避免不一致性。

每个线程都有线程ID，它只在所属进程内作用

## 5.出错处理

当UNIX函数出错时，常返回一个负值，整型变量errno通常被设置为含有附加信息的一个值。

c标准定义了两个函数打印出错信息。

```c
#include<string.h>
char *strerror(int errnum);//返回值：指向信息字符串的指针
```

此函数将errnum（它通常就是errno值）映射为一个出错信息字符串，并返回此字符串的指针

perror函数基于errno的当前值，在标准出错上产生一个出错信息，然后返回。

```c
#include<stdio.h>
void perror(const char *msg);
```

首先输出由msg指向的字符串，然后是一个冒号，一个空格，接着是对应于errno值得出错信息，最后是一个换行符

## 6.信号！！

### 6.1信号的概念

![203](F:\学习专用\note\pic\203.png)

信号是通知进程已发生某些情况的一种技术。

**每个进程收到的所有信号，都是由内核负责发送的，内核处理。**

#### 6.1.1与信号相关的事件和状态

**信号产生**

- 按键产生，如：ctrl+c,ctrl+z,ctrl+\
- 系统调用产生，如：kill,raise,abort
- 软件条件产生，如：定时器alarm
- 硬件异常产生，如：非法访问内存（段错误），除0，内存对齐错误（总线错误）
- 命令产生，如：kill命令

**递达：**

递送并到达进程

**未决：**

产生和递达之间的状态。主要由于阻塞（屏蔽）导致该状态

**处理信号方式**： 

- **忽略**处理该信号，有些信号表示硬件异常
- 按系统**默认**方式处理，对于除以0得情况，系统默认是终止进程
- 提供一个函数pause，信号发生时则调用该函数**捕捉**信号。

#### 6.1.2信号的编号

可以使用kill –l命令查看当前系统可使用的信号有哪些。

​         1) SIGHUP          **2) SIGINT**         3) SIGQUIT      4) SIGILL         5) SIGTRAP

​        **6) SIGABRT**        **7) SIGBUS**        8) SIGFPE         **9) SIGKILL**       10) SIGUSR1

11) SIGSEGV    12) SIGUSR2    13) SIGPIPE      **14) SIGALRM**   **15) SIGTERM**

16) SIGSTKFLT  **17) SIGCHLD**    18) SIGCONT    **19) SIGSTOP**     20) SIGTSTP

21) SIGTTIN     22) SIGTTOU    23) SIGURG      24) SIGXCPU    25) SIGXFSZ

26) SIGVTALRM        27) SIGPROF     28) SIGWINCH 29) SIGIO 30) SIGPWR

31) SIGSYS        34) SIGRTMIN 35) SIGRTMIN+1      36) SIGRTMIN+2      37) SIGRTMIN+3

38) SIGRTMIN+4      39) SIGRTMIN+5      40) SIGRTMIN+6      41) SIGRTMIN+7      42) SIGRTMIN+8

43) SIGRTMIN+9      44) SIGRTMIN+10    45) SIGRTMIN+11    46) SIGRTMIN+12    47) SIGRTMIN+13

48) SIGRTMIN+14    49) SIGRTMIN+15    50) SIGRTMAX-14    51) SIGRTMAX-13    52) SIGRTMAX-12

53) SIGRTMAX-11    54) SIGRTMAX-10    55) SIGRTMAX-9      56) SIGRTMAX-8      57) SIGRTMAX-7

58) SIGRTMAX-6      59) SIGRTMAX-5      60) SIGRTMAX-4      61) SIGRTMAX-3      62) SIGRTMAX-2

63) SIGRTMAX-1      64) SIGRTMAX

#### 6.1.3信号4要素

编号，名称，事件，默认处理动作

可通过man 7 signal查看帮助文档获取

Signal          Value     Action   Comment

────────────────────────────────────────────

SIGHUP        1       Term    Hangup detected on controlling terminal or death of controlling process

SIGINT        2       Term    Interrupt from keyboard

SIGQUIT       3       Core    Quit from keyboard

SIGILL         4       Core    Illegal Instruction

SIGFPE        8       Core    Floating point exception

SIGKILL        9       Term    Kill signal

SIGSEGV        11      Core    Invalid memory reference

SIGPIPE            13      Term    Broken pipe: write to pipe with no readers

SIGALRM        14      Term    Timer signal from alarm(2)

SIGTERM      15      Term    Termination signal

SIGUSR1   30,10,16    Term    User-defined signal 1

SIGUSR2   31,12,17    Term    User-defined signal 2

SIGCHLD   20,17,18    Ign     Child stopped or terminated

SIGCONT   19,18,25    Cont    Continue if stopped

SIGSTOP   17,19,23    Stop    Stop process

SIGTSTP   18,20,24    Stop    Stop typed at terminal

SIGTTIN   21,21,26    Stop    Terminal input for background process

SIGTTOU   22,22,27   Stop    Terminal output for background process

**默认动作：**

​                  Term：终止进程

​                  Ign： 忽略信号 (默认即时对该种信号忽略操作)

​                  Core：终止进程，生成Core文件。(查验进程死亡原因， 用于gdb调试)

​                  Stop：停止（暂停）进程

​                  Cont：继续运行进程

**注意：**

**9) SIGKILL 和19) SIGSTOP信号，不允许忽略和捕捉，只能执行默认动作。甚至不能将其设置为阻塞。**

### 6.2信号产生

**终端按键产生信号**

​    Ctrl + c  → **2) SIGINT（终止/中断）**   "INT" ----Interrupt

​    Ctrl + z  → **20) SIGTSTP（暂停/停止）**  "T" ----Terminal 终端。

​    Ctrl + \  → **3) SIGQUIT（退出）** 

**硬件异常产生信号**

​     除0操作   → **8) SIGFPE (浮点数例外)**         "F" -----float 浮点数。

​    非法访问内存  → **11) SIGSEGV (段错误)**

​    总线错误  → 7) **SIGBUS** 

#### raise和abort函数

raise函数：给当前进程发送指定信号

~~~cpp
int raise(int sig);//成功0，失败非0
~~~

abort函数：给自己发送异常终止信号 6）SIGABRT信号，终止并产生core文件

~~~cpp
void abort(void);//无返回值
~~~

#### 6.2.1软件条件产生信号

**alarm函数**

设置定时器，指定秒后，内核会给当前进程发送14）SIGALRM信号，进程收到信号，默认动作终止。

**每个进程都有且只有唯一个定时器**

~~~cpp
#inlcude<stdio.h>//计算1s钟计算的数
int main()
{
	alarm(1);//定时1s
	int i=0;
	for(;;i++)
	{
        printf("%d\n",i);
	}
    return 0;
}
~~~

使用time命令查看系统执行的时间，程序运行的瓶颈在于IO,优化程序，首先优化IO

实际执行时间=系统时间+用户时间+等待时间

### 6.3信号捕捉

![205](F:\学习专用\note\pic\205.png)

~~~cpp
#include<signal.h>
typedef void(*sighandler_t)(int);
sighandler_t signal(int sig,sighandler_t hander);//出错：SIG_ERR
~~~

~~~cpp
#include<signal.h>
#include<stdlib.h>
#include<stdio.h>
typedef void(*sighandler_t)(int);
void do_signal(int sig)
{
    printf("catch SIGINT----------\n");
}
int main()
{
	sighandler_t it;
	it=signal(SIGINT,do_signal);//注册信号捕捉函数
	if(it==SIG_ERR)
	{
        perror("signal err: ");
        exit(1);
	}
    while(1);
    return 0;
}
~~~

![204](F:\学习专用\note\pic\204.png)

### 6.4 竞态条件

**pause函数**

调用该函数可以造成进程主动**挂起**，等待信号唤醒。调用该系统调用的进程将处于阻塞状态(主动放弃cpu) 直到有信号递达将其唤醒。

~~~~cpp
int pause(void);//返回值：-1并且设置errno为EINTR
~~~~

  返回值：

​		   ① 如果信号的默认处理动作是终止进程，则进程终止，pause函数么有机会返回。

​                   ② 如果信号的默认处理动作是忽略，进程继续处于挂起状态，pause函数不返回。

​                   ③ 如果信号的处理动作是**捕捉**，则【调用完信号处理函数之后，pause返回-1】

​                      errno设置为EINTR，表示“被信号中断”。想想我们还有哪个函数只有出错返回值。

​                   ④ pause收到的信号不能被屏蔽，如果被屏蔽，那么pause就不能被唤醒。

案例：使用pause和alarm实现sleep函数

~~~cpp
#include<stdio.h>
#include<signal.h>
#include<unistd.h>
#include<stdlib.h>
#include<errno.h>
void do_signal(int sig)
{
    ;
}
void mysleep(int sec)
 {
     signal(SIGALRM,do_signal);//注册信号捕捉函数，捕捉中断信号
     alarm(sec);//产生中断信号
     int ret=pause();//主动挂起，等信号，造成阻塞
    if(ret==-1&&errno==EINTR)
    {
        printf("pause------\n");
    }
 }
int main()
{
    int i=5;
    while(1)
    {
        mysleep(i);
        printf("------------\n");
    }
    return 0;
}
~~~

![206](F:\学习专用\note\pic\206.png)

## 7.时间值

UNIX系统一直使用两种不同时间值：

（1）日历时间，国际标准时间，系统基本数据类型time_t用于保存这种时间值

（2）进程时间，也称cpu时间，用以度量使用的中央处理机资源。系统基本数据类型clock_t用于保存这种时间值。

UNIX系统使用的三个进程时间值：

- 时钟时间，又称墙上时钟时间，他是进程运行的时间总量，其值与系统中同时运行的进程数有关。
- 用户CPU时间，执行用户指令所用的时间
- 系统CPU时间，该进程执行内核程序所经历的时间。每当一个进程执行一个系统服务，如read或write，则在内核内执行该服务所花费的时间就计入该进程的系统CPU时间

## 8.系统调用和库函数

**系统调用**：

所有的操作系统都提供多种服务的入口点，程序由此向内核请求服务。各种版本的UNIX实现都提供定义明确、数量有限、可直接进入内核的入口点，这些入口点称为**系统调用**。

系统调用和库函数的差别:系统调用通常提供一种最小接口，而库函数通常提供比较复杂的功能。从sbrk系统调用和malloc库函数之间的差别就能看出。

进程控制系统调用（fork、exec和wait）通常由用户应用程序直接调用。

![40](F:\学习专用\学习笔记\图片\40.png)

# 二. 标准I/O库

## 1.流和FILE对象

所有I/O函数都是针对文件描述符的。当打开一个文件时，即返回一个文件描述符，然后改文件描述符就用于后续的I/O操作。对于标准I/O库，他们的操作则是围绕流进行的。当用标准I/O库打开或创建一个文件时，我们已使一个流于一个文件相关联。

**流的定向**决定了所读、写的字符是单字节还是多字节的。当一个流最初被创建时，他并没有定向。如果在为定向的流上使用一个多字节I/O函数，则将改流的定向设置为**宽定向**。若在未定义的流上使用一个单字节I/O函数，则将改流的定向设置为字节定向。

通过freopen可清楚一个流的定向，fwide函数设置流的定向。

```c
#include<stdio.h>
#include<wchar.h>
int fwide(FILE *fp,int mode);//返回值：流是宽定向返回正值，字节定向返回负值，为定向返回0
```

mode参数：

- 负值，fwide将试图使指定的流是字节定向
- 正值，fwide将试图使指定的流是宽定向
- 0，不设置定向

fwide不改变已定向流的定向。

## 2.标准输入、输出和出错

三个标准I/O流通过预定义文件指针stdin、stout和stderr加以引用，在头文件<stdio.h>中

## 3.缓冲

标准I/O库提供缓冲的目的是尽可能减少使用read和write调用的次数。它也对每个I/O流自动进行缓冲管理，从而避免了应用程序需要考虑这一点所带来的麻烦。

**标准I/O提供了三种类型的缓冲：**

- [ ] 全缓冲，在填满标准I/O缓冲区后才进行实际I/O操作。对于驻留在磁盘上的文件通常是由标准I/O库实施全缓冲的。在一个流上执行第一次I/O操作时，相关标准I/O函数通常调用malloc获得需使用的缓冲区。
- [ ] 行缓冲，当在输入和输出中遇到换行符，标准I/O库执行I/O操作。这允许我们一个输出一个字符（用标准I/O fputc函数），但只有在写了一行之后才进行实际I/O操作。
- [ ] 不带缓冲，标准I/O库不对字符进行缓冲存储。例，如果用标准I/O函数fputc写15个字符到不带缓冲的流中，则该函数很可能用write系统调用函数将这些字符立即写至关联的打开文件上。

**ISO C要求下列缓冲特征：**

- 当且仅当标准输入和标准输出并不涉及交互式设备时，他们才是全缓冲的 。
- 标准出错绝不会是全缓冲的。

**很多系统默认使用下列类型的缓冲：**

- 标准出错是不带缓冲的
- 如若是涉及终端设备的其他流，则他们是行缓冲的，否则是全缓冲的。

**更改缓冲类型的函数：**

```c
#include<stdio.h>
void setbuf(FILE *restrict fp,char *restrict buf);
int setvbuf(FILE *restrict fp,char *restrict buf,int mode,size_t size);//返回值：成功0，出错-1
```

这些函数一定要在流已被打开后调用，而且也应该在对流执行任何一个其他操作之前调用。

可以使用setbuf函数打开或关闭缓冲机制。此时，将buf设置为NULL

使用setvbuf，可以指定所需的缓冲类型。

mode参数：

_IOFBF  全缓冲

_IOLBF  行缓冲

_IONBF  不带缓冲

若指定一个不带缓冲的流，则忽略buf和size。如果指定全缓冲或行缓冲，则buf和size额选择的指定一个缓冲区及其长度。若果该流是带缓冲的，而buf为NULL，则标准I/O库将自动为该流分配适当长度的缓冲区。

**强制冲洗一个流：**

```c
#include<>
```

此函数使该流所有未写的数据都被传送至内核。

## 4.打开流

三个函数打开标准I/O流

```c
#include<stdio.h>
FILE *fopen(const char *restrict pathname,const char *restrict type);
FILE *freopen(const char *restrict pathname,const char *restrict type,FILE *restrict fp);
FILE *fdopen(int file,const char *type);//三函数返回值：成功返回文件指针，出错返回NULL
```

区别：

- fopen打开一个指定的文件
- freopen在指定的流上打开一个指定的文件，如若该流已经打开，则先关闭该流。此函数一般用于将一个指定的文件打开为一个预定义的流：标准输入、标准输出或标准出错
- fdopen获取一个现有的文件描述符（可从open、dup、、、函数获取此文件描述符），并使一个标准的I/O流与该描述符相结合。此函数通常用于创建管道和网络通信通道函数返回的描述符。

type参数指定对该I/O流的读、写方式，如图

![42](F:\学习专用\学习笔记\图片\42.png)

用fclose关闭一个打开的流：

```c
#include<stdio.h>
int fclose(FILE *fp);
```

在该文件被关闭之前，冲洗缓冲区中的输出数据，丢弃缓冲区中的任何输入数据，如果标准I/O库已经为该流自动分配了一个缓冲区，则释放此缓冲区。

当一个进程正常终止时（直接调用exit函数，或从main返回），则所有带未写缓冲数据的标准I/O流都会被冲洗，所有打开的标准I/O流都会被关闭

## 5.读和写流

一旦打开了流，可在三种不同类型的非格式化I/O中进行选择，对其进行读、写操作：

（1）每次一个字符的I/O，一次读或写一个字符，如果流是带缓冲的，则标准I/O函数回处理所有缓冲。

（2）每次一行I/O。如想要一个读或写一行，则使用fgets和fputs。每行都以一个换行符终止。

（3）直接I/O。fread和fwrite函数支持这种类型的I/O。每次I/O操作读或写某种数量的对象，而每个对象具有指定的长度。

### 5.1输入函数

一次读一个字符的三个函数：

```c
#include<stdio.h>
int getc(FILE *fp);
int fgetc(FILE *fp);
int getchar(void);//三函数返回值：成功返回下一个字符，已到达文件结尾或出错返回EOF
```

函数getchar等价于getc(stdin)。

### 5.2输出函数

对应上面每个输入函数都有一个输出函数

```c
#include<stdio.h>
int putc(int c,FILE *fp);
int fputc(int c,FILE *fp);
int putchar(int c);//三函数返回值：成功返回c，出错返回EOF
```

函数putshar（c）等效于putc（c,stdout）

## 6.每次一行I/O

下面两个函数提供每次输入一行的功能

```c
#include<stdio.h>
char *fgets(char *restrict buf,int n,FILE *restrict fp);
char *gets(char *buf);//返回值：成功返回buf，若到达文件结尾或出错返回NULL
```

两个函数都指定了缓冲区的地址，读入的行将送人其中，gets从标准输入读，而fgets则从指定的流读。

对于fgets，必须指定缓冲区的长度n。此函数一直读到下一个换行符为止。

gets是一个不建议使用的函数，其问题是调用者在使用gets时不能指定缓冲区的长度。这样可能造成缓冲区溢出，写到缓冲区之后的存储空间中，产生不可预料的后果。gets与fgets的另一个区别是：gets不讲换行符存入缓冲区。

fputs和puts提供每次输出一行的功能。

```c
#include<stdio.h>
int fputs(const char *restrict str,FILE *restrict fp);
int puts(const char *str);//返回值：成功返回非负值，出错返回EOF
```

函数fputs将一个以null符终止的字符串写到指定的流，尾端的终止符null不写出。

puts将一个以null符终止的字符串写到标准输出，终止符不写出

## 7.标准I/O的效率

例程：用getc和putc将标准输入复制到标准输出

```c
#include"apue.h"
int main(void)
{
    int c;
    while((c=getc(stdin))!=EOF)
        if(putc(c,stdout)==EOF)
            err_sys("output error");
    if(ferror(stdin))
        err_sys("input error");
    exit(0);
    
}
```

用fgets和fputs将标准输入复制到标准输出

```c
#include"apue.h"
int main(void)
{   
    char buf[MAXLINE];   
    while(fgets(buf,MAXLINE，stdin)!=NULL)    
        if(fputc(buf,stdout)==EOF)      
            err_sys("output error"); 
    if(ferror(stdin))    
        err_sys("input error");   
    exit(0);    
}
```

## 8.二进制I/O

进行二进制I/O操作，一次读或写整个结构。下列两个函数可以执行二进制I/O操作

```c
#include<stdio.h>
size_t fread(void *restrict ptr,size_t size,size_t nobj,FILE *restrict fp);
size_t fwrite(const void *restrict ptr,size_t size,size_t nobj,FILE *restrict fp);//返回值：读或写的对象数
```

两函数常见用法：

（1）读或写一个二进制数组，例：将一个浮点数组的第2-5个元素写至一个文件上

```c
float data[10];
if(fwrite(&data[2],sizeof(float),4,fp) != 4)
    err_sys("fwrite error");
```

size参数为每个数组元素的长度，nobj参数为欲写的元素数

（2）读或写一个结构，例

```c
struct{
    short c;
    long a;
    char name[LINE];
}item;
if(fwrite(&item,sizeof(item),1,fp) != 1)
    err_sys("fwrite error");
```

size指定结构的长度，nobj为要写的对象数

## 9.定位流

三种方法定位标准I/O流

（1）ftell和fseek函数。他们都假定文件的位置可以存放在一个长整型中

（2）ftell和fseeko函数。可以使文件偏移量不必一定使用长整型。

（3）fgetpos和fsetpos函数，他们使用一个抽象数据类型fpos_t记录文件的位置。

需要移植到非UNIX系统上运行的应用程序应当使用fgetpos和fsetpos。

```c
#include<stdio.h>
long ftell(FILE *fp);//返回值：成功返回当前文件位置指示，出错-1
int fseek(FILE *fp,long offset,int whence);//返回值：成功0，出错-1
void rewind(FILE *fp);
```

fgetpos和fsetpos这两个函数是c函数引进的

```c
#include<stdio.h>
int fgetpos(FILE *restrict fp,fpos_t *restrict pos);
int fsetpos(FILE *fp,const fpos_t *pos);//两函数返回值：成功返回0，出错非0
```



fgetpos将文件位置指示器的当前值存入由pos指向的对象中。在以后调用fsetpos时，可以使用此值将流重新定位至该位置。

## 10.格式化I/O

### 10.1格式化输出

执行格式化输出处理的4个printf函数

```c
#include<stdio.h>
int printf(const char *restrict format,...);
int fprintf(FILE *restrict fp,const char *restrict format,...);//两个函数返回值：成功返回输出字符数，出错返回负值
int sprintf(char *restrict buf,const char *restrict format,...);
int snprintf(char *restrict buf,size_t n,const char *resrtict format,...);//返回值：成功返回存入数组的字符数，出错返回负值
```

printf将格式化数据写到标准输出，fprintf写至指定的流，sprintf将格式化的字符送入数组buf中，snprintf在该数组的尾端自动加一个null字节，但该字节不包括在返回值中。

### 10.2 格式化输入

执行格式化输入处理的是三个scanf函数

```c
#include<stdio.h>
int scanf(const char *restrict format,...);
int fscanf(FILE *restrict fp,const char *restrict format,...);
int sscanf(const char *restrict buf,const char *restrict format,...);//三函数返回值：指定的输入项数，输入出错或在任意变换前已到达文件结尾则返回EOF
```

scanf族用于分析输入字符串，并将字符序列转换成指定类型的变量。

## 11.实现细节

每个标准I/O流都有一个与其相关联的文件描述符，可以对一个流调用fileno函数以获得其文件描述符

```c
#include<stdio.h>
int fileno(FILE *fp);//返回值：与该流相关联的文件描述符
```

## 12.临时文件

ISO C标准I/O库提供了两个函数以帮助创建临时文件。

```c
#include<stdio.h>
char *tmpnam(char *ptr);//返回值：指向唯一路径名的指针
FILE *tmpfile(void);//返回值：成功返回文件指针，出错返回NULL
```

tmpnam函数产生一个与现有文件名不同的一个有效路径名字符串。每次调用它时，都产生一个不同的路径名，最多调用TMP_MAX，至少为25

若ptr是NULL，则所产生的路径名存放在一个静态区中，指向该静态区的指针作为函数值返回。

tmpfile创建一个临时二进制文件，在关闭该文件或程序结束时将自动删除这种文件。

# 三. 文件和目录

## 1.stat、fstat和lstat函数！

```c
#include<sys/stat.h>
int stat(const char *restrict pathname,struct stat *restrict buf);
int fstat(int file,struct stat *buf);
int lstat(const char *restrict pathname,struct stat *restrict buf);//三函数返回值：成功0，出错-1
```

一旦给出pathname，**stat就返回此命名文件有关的信息结构**。fstat函数获取已在描述符file上打开文件的有关信息。lstat函数类似于stat，但当命名的文件是一个符号链接时，lstat返回该符号链接的有关信息，**不是链接到的那个文件的信息**，使用stat函数返回的是映射到的文件的信息。

**注意：**

lstat不穿透（跟踪）符号链接

stat穿透（跟踪）符号链接

buf指针指向一个数据结构，基本形式：

![180](F:\学习专用\note\pic\180.png)

使用stat函数最多的是ls -l命令，获得文件的所有信息

### 1.1 案例

~~~c++
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>
#include<stdio.h>
#include<stdlib.h>
int main(int argc,char *argv[])
{
	if(argc<2)
	{
		perror("./out filename \n");
		exit(1);
	}
	struct stat st;
	int ret=stat(argv[1],&st);//将输入的第一个字符串的信息保存在st中
	if(ret==-1)
	{
		perror("stat");
		exit(1);
	}
	int size=(int)st.st_size;//获取size大小信息
	printf("%s size = %d\n",argv[0],size);
}
~~~

结果：

![179](F:\学习专用\note\pic\179.png)

## 2.文件类型

- 普通文件
- 目录文件
- 块特殊文件，提供对设备（磁盘）带缓冲的访问，每次访问以固定长度为单位进行
- 字符特殊文件，提供对设备不带缓冲的访问，每次访问长度可变
- FIFO，用于进程间通信，有时也称为**管道**
- 套接字，用于进程间网络通信
-  链接，这种文件类型指向另一个文件。



## 3.设置用户ID和设置组ID

与一个进程相关联的ID由6个或更多

![41](F:\学习专用\学习笔记\图片\41.png)

当执行一个程序文件时，有效用户ID等于实际用户ID，有效组ID等于实际组ID

## 4.link、unlink、remove和rename函数

创建一个指向现有文件的链接的方法是使用link函数

```c
#include<unistd.h>
int link(const char *oldpath,const char *newpath);//成功返回0，出错-1
```

若newpath已经存在，则出错。

```c
#include<unistd.h>
int unlink(const char *pathname);//成功返回0，出错-1
```

此函数删除目录项，用于创建临时文件，使用后则删除文件

## 5.符号链接

![173](F:\学习专用\note\pic\173.png)

符号链接是指向一个文件的间接指针。引入符号链接是为了避开硬链接的一些限制：

- 硬链接通常要求链接和文件位于同一文件系统中
- 只有超级用户才能创建指向目录的硬链接

# 四. 文件I/O！！！

大多数文件I/O只需要5个函数：

**open、read、write、lseek以及close**

**不带缓冲的I/O：**

每个**read和write**都调用内核中的一个系统调用

## 1.文件描述符

对于内核而言，所有打开的文件都通过文件描述符引用。当打开一个文件时，内核向进程返回一个文件描述符。当读或写一个文件时，使用open或create返回的文件描述符标识该文件，将其作为参数传给read或write。

UNIX系统shell使用**文件描述符0**与进程的标准**输入**相关联，**1**与标准**输出**相关联，**2**与标准**出错输出**相关联，文件描述符一共有1024个，前3个不能使用，用户使用从第4个开始。

STDIN_FILENO 标准输入

STDOUT_FILENO 标准输出

STDERR_FILENO 标准出错

## 2.open函数

调用open函数打开或创建一个文件

```c
#include<fcntl.h>
int open(const char *pathname,int oflag,.../*mode_t mode*/);//返回值：成功返回文件描述符，出错返回-1
```

pathname是要打开或创建文件的名字。oflag参数，可用下列一个或多个常量进行“或”运算。

在这三个常量中必须指定一个且只能指定一个

O_RDONLY   	只读打开

O_WRONLY   只写打开

O_RDWR        读、写打开

可选常量：

O_APPEND 每次写时都加到文件的尾端

O_CREAT    若文件不存在，则创建它

O_EXCL      如果同时指定了O_CREAT，而文件已经存在，则会出错，用此可测试一个文件是否存在，如果不存在，则创建文件，这使测试和创建两者成为一个原子操作。

O_TRUNC   如果此文件存在，而且为只写或读写成功打开，则将其长度截短为0

## 3.creat函数

调用creat函数创建一个新文件

```c
#include<fcntl.h>
int creat(const char *pathname,mode_t mode);//返回值：成功返回为只写打开的文件描述符，出错返回-1
```

此函数等效于：open(pathname,O_WRONLY|O_CREAT|O_TRUNC,mode);

## 4.close函数

调用close函数关闭一个打开的文件

```c
#include<unistd.h>
int close(int file);//成功返回0，出错-1
```

关闭一个文件时还会释放该进程加在该文件上的所有记录锁。

当一个进程终止时，内核自动关闭它所有打开的文件。很多程序都利用这一功能不显式的关闭打开文件。

## 5.lseek函数

### **偏移量**

每个打开的文件都有一个与其相关联的“当前文件偏移量”。它通常是一个非负整数，用以度量从文件开始处计算的字节数。通常，读、写操作都从当前文件偏移量处开始，并使偏移量增加所读写的字节数。系统默认下，打开文件，未指定O_APPEND选项时，该偏移量为0

调用lseek函数能显式地为一个打开地文件设置其**偏移量**

```c
#include<unistd.h>
off_t lseek(int file,off_t offset,int whence);//成功返回新的文件偏移量，出错-1
```

对参数offset地解释与参数whence有关

- 若whence是SEEK_SET，则将该文件地偏移量设置为距文件开始处offset个字节
- 若whence是SEEK_CUR，则将该文件地偏移量设置为其当前值加offset，offset可正可负
- 若whence是SEEK_END，则将该文件地偏移量设置为文件长度加offset，offset可正可负

例：

```c
off_t currpo;
currpo = lseek(fd,0,SEEK_CUR);//打开文件地当前偏移量
```

### 文件扩展

~~~cpp
int ftruncate(int file,off_t len);//设置文件的容量，成功0，失败-1
~~~

## 6.read函数

调用read函数从打开文件中读数据

```c
#include<unistd.h>
int read(int file,char *buf,unsigned nbytes);//成功返回读到地字节数，若已到文件结尾返回0，出错-1
```

实际读到字节数少于要求读地字节数：

- 读文件时，要求字节数大于文件的字节数。如，到达文件尾端之前还有30个字节，而要求读100字节，则read返回30
- 当从终端设备读时，通常一次最多读一行
- 当从网络读时，网络中的缓冲机构可能造成返回值小于所要求读的字节数
- 当从管道或FIFO读时，若管道包含的字节少于所需的数量，那么read将只返回实际可用的字节数
- 当从某些面向记录的设备（如磁盘）读时，一次最多返回一个记录
- 当某一信号造成中断，而已经读了部分数据量时。

## 7.write函数

调用write函数向打开的文件写数据

```c
#include<unistd.h>
ssize_t write(int file,const void *buf,size_t nbytes);//file为1为标准写（将信息写到终端），成功返回已写字节数，出错返回-1
```

write出错的常见原因：磁盘已写满，或超过一个给定进程的文件长度限制。

普通文件，写操作从文件的当前偏移量处开始，如果在打开文件时，指定了O_APPEND选项，则每次写操作之前，将文件偏移量设置在文件的当前结尾处，成功写之后，该文件偏移量增加实际写的字节数。

### 7.1 案例

~~~c++
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>
#include<stdlib.h>
#include<stdio.h>
#include<unistd.h>
int main()
{
	int fd;//文件描述符
	fd=open("hello.txt",O_RDONLY);//打开一个已存在的文件
	if(fd==-1)
	{
		perrno("打开文件失败");
		exit(1);
	}
	int fd1;
	fd1=open("read.txt",O_CREAT|O_RDWR);//打开一个已存在的文件
	if(fd1==-1)
	{
		perrno("打开文件失败");
		exit(1);
	}
	char buf[1024]={0};
	int count = read(fd,buf,sizeof(buf));//读取字符，返回读取到的字符个数，为0则已读取完毕，-1则读取失败
	if(count==-1)
	{
		perrno("读取失败");
		exit(1);
	}
	while(count)//判断文件是否读取完毕
	{
		int ret = write(fd1,buf,count);
		printf("读取到的字符数为： %d\n",ret);
		count = read(fd,buf,sizeof(buf));
	}
	close(fd);
	close(fd1);
}
~~~

## 8.I/O的效率

例程：标准输入复制到标准输出

```c
#include"apue.h"
#define BUFFSIZE 4096

int main()
{
    int n;
    char buf[BUFFSIZE];
    while((n=read(STDIN_FILENO,buf,BUFFSIZE))>0)
        if(write(STDOUT_FILENO,buf,n)!=n)
            err_sys("write error");
    if(n<0)
        err_sys("read error");
    exit(0);
}
```

注意点：

- 它从标准输入读，写至标准输出，这就假定在执行本程序前，这些标准输入，输出已由shell安排好。使得程序不必自行打开输入和输出文件。
- 很多应用程序假定标准输入是文件描述符0，标准输出是文件描述符1。本例程中使用<unsitd.h>中定义的俩个名字：STDIN_FILENO,STDOUT_FILENO
- 进程终止时，UNIX系统内核会关闭该进程的所有打开的文件描述符，故程序中未关闭输入和输出文件
- 对UNIX系统内核而言，文本文件和二进制代码文件无区别

## 9.文件共享

UNIX系统支持在不同进程间共享打开的文件。

内核使用三种数据结构表示打开的文件，他们之前的关系决定了在文件共享方面一个进程对另一个进程可能产生的影响。

![36](F:\学习专用\学习笔记\图片\36.png)



（1）每个进程在进程表中都有一个记录项，记录项中包含有一张打开文件描述符表，可将其视为一个矢量，每个描述符占用一项，与每个文件描述符相关联的是：

- 文件描述符标志
- 指向一个文件表项的指针

（2）内核为所有打开文件维持一张文件表，每个文件表项包含：

- 文件状态标志（读、写、添加。。）
- 当前文件偏移量
- 指向该文件v节点表项的指针

（3）每个打开文件（或设备）都有一个v节点结构，v节点包含了文件类型和对比文件进行各种操作的函数的指针。

**两个独立进程各自打开同一文件**

![37](F:\学习专用\学习笔记\图片\37.png)



如图一进程用文件描述符3打开该文件，二进程用文件描述符4打开该文件，每个进程都得到一个文件表项，但一个文件只有一个v节点表项，两个进程的v节点指针均指向v节点表项

进一步说明：

- 在完成每个write后，在文件表项中的当前文件偏移量即增加所写的字节数。
- 如果用O_APPEND标志打开一个文件，则相应标志也被设置到文件表项的文件状态标志中。

## 10.原子操作

多步组成的操作，如果该操作原子地执行，则要么执行完所有步骤，要么一步也不执行，不可能只执行所有步骤地一个子集。

### 10.1添加至一个文件

两个独立进程A和B都对同一文件进行添加操作，UNIX系统提供一种方法使这种操作成为原子操作，该方法是在打开文件时设置O_APPEND标志。这使内核每次对这种文件进行写之前，都将进程的当前偏移量设置到该文件的尾端处，于是每次写之前就不再调用lseek。

### 10.2 pread和pwrite函数

pread和pwrite是XSI扩展，该扩展允许原子性地定位搜索和执行I/O

```c
#include<unistd.h>
ssize_t praed(int file,void *buf,size_t nbytes,off_t offset);//返回值：读到的字节数，若已到文件结尾则返回0，出错-1
ssize_t pwrite(int file,const void *buf,size_t nbytes,off_t offset;//返回值：成功返回已写的字节数，出错-1
```

调用pread相当于顺序调用lseek和read，但也有区别：

- 调用pread，无法中断其定位和读操作
- 不更新文件指针

调用pwrite相当于顺序调用lseek和write，不过也有相同的区别

### 10.3 创建一个文件

同时选择open函数地O_CREAT和O_EXCL选项，该文件已经存在则open失败。检查该文件是否存在以及创建该文件这两个操作作为一个一个原子操作执行。

## 11.dup和dup2函数

这两个函数用于复制一个现存地文件描述符

```c
#include<unistd.h>
int dup(int fd);
int dup2(int oldfd,int newfd);//两个函数返回值：成功返回新的文件描述符，出错-1
```

dup返回的新文件描述符是当前可用文件描述符中的最小数值。dup2可用newfd参数指定新的文件描述符数值。**若newfd已经打开，则先将其关闭，若oldfd等于newfd，则dup2返回newfd，而不关闭它**。

这些函数返回的新文件描述符与参数oldfd共享同一个文件表项。

假定进程执行：

newfd=dup（1）；

![38](F:\学习专用\学习笔记\图片\38.png)

当此函数开始执行，新的文件描述符很有可能为3（0，1，2已经被shell打开），两个文件描述符指向同一文件表项，共享同一文件状态标志(读、写等)和同一当前文件偏移量

复制一个描述符的另一种方法是使用ftcntl函数

调用dup(file)等效于fcntl（file,F_DUPFD,0）

调用dup2(file,file2)等效于close(file2);fcntl(file,F_DUPFD,file2);

## 12.fcntl函数

fcntl函数可以改变已打开的文件的性质（打开文件时：只写，后期追加O_APPEND实现文件追加写）

```c
#include<fcntl.h>
int fcntl(int file,int cmd,.../*int arg*/);//返回值：成功则依赖cmd，出错-1
```

**fcntl函数5种功能：**

- 复制一个现有的描述符（cmd=F_DUPFD）
- 获得/设置文件描述符标记（cmd=F_GETFD或F_SETFD）
- 获得/设置文件状态标志（cmd=F_GETFL或F_SETFL）![181](F:\学习专用\note\pic\181.png)
- 获得/设置异步I/O所有权（cmd=F_GETOWN或F_SETOWN）
- 获得/设置记录锁（cmd=F_GETLK、F_SETLK或F_SETLKW）

# 五. 线程！！！

![211](F:\学习专用\note\pic\211.png)

**打开火狐浏览器产生的多线程，可看出进程ID和父进程ID均相同，LWP不同（LWP为线程号，不是线程ID，是cpu分配时间轮片的依据，而线程ID是进程中区分线程的依据）**

## 1.引言

线程是轻量级的进程，本质仞是进程

进程：独立地址空间，拥有PCB

线程：共享地址空间，每个线程有属于自己的PCB

区别：**是否共享地址空间**

Linux下：线程：最小执行单位

​		进程：最小分配资源单位，可看成只有一个线程的进程

![209](F:\学习专用\note\pic\209.png)

### **1.1Linux内核线程实现原理**

- 轻量级进程，有PCB，创建线程使用的底层函数和进程一样，都是clone()函数
- 从内核里看进程和线程是一样的，都有各自不同的PCB，但PCB中指向内存资源的**三级页表是相同**的
- 进程可以蜕变为线程
- 线程可以看做寄存器和栈的集合
- **在Linux下，线程是最小的执行单元，进程是最小的分配资源单位**

### **1.2三级页表**

![210](F:\学习专用\note\pic\210.png)

### 1.3线程共享资源

- 文件描述符
- 每个信号的处理方式
- 当前工作目录
- 用户ID和组ID
- 内存地址空间（.text/.data/.bss/heap/共享库）（无stack栈）

### 1.4线程非共享资源

- 线程id
- 处理器现场和栈指针（内核栈）
- 独立的栈空间（用户空间栈），函数运行所需要的空间
- errno变量
- 信号屏蔽字
- 调度优先级

线程优缺点

优点：1.提高程序并发性，2,开销小，3 .数据通信，共享数据方便

缺点：1.库函数，不稳定，2.调试，编写困难，3.对信号支持不好

## 2.线程标识

进程ID在整个系统中是唯一的，但线程ID只有在它所属的进程环境中才有效。

线程ID用pthread_t数据类型表示。

**比较两个线程ID：**

```c
#include<pthread.h>
int pthread_equal(pthread_t tid1,pthread_t tid2); 
```

**获得自身的线程ID**

```c
#include<pthread.h>
pthread_t pthread_self(void);//返回值：调用线程的线程ID
```

## 3.线程创建

新增线程通过pthread_create函数创建

```c
#include<pthread.h>
int pthread_create(pthread_t *tidp,const pthread_attr_t *restrict attr,void *(*start_rtn)(void *),void *arg);//返回值：成功返回0，否则返回出错编号
```

参数：    

 pthread_t：当前Linux中可理解为：typedef  unsigned long int  pthread_t;

​       参数1：传出参数，保存系统为我们分配好的线程ID

​       参数2：通常传NULL，表示使用线程默认属性。若想使用具体属性也可以修改该参数。

​       参数3：函数指针(返回值为void*,参数为 void *)，指向线程主函数(线程体)，该函数运行结束，则线程结束。

​       参数4：线程主函数执行期间所使用的参数。

**例程：**

创建一个线程并打印进程ID，新线程的线程ID及初始线程ID

```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<pthread.h>
void *start_func(void *arg)//新线程的主函数，该函数运行结束则线程结束
{
    printf("in pthread: thread id: %lu,pid:%u\n",pthread_self(),getpid());
    return NULL;
}
int main()
{
    pthread_t tid;
    printf("in man: thread id: %lu,pid:%u\n",pthread_self(),getpid());
    int ret=pthread_create(&tid,NULL,start_func,NULL);
    if(ret!=0)
    {
        printf("pthread_create err: ");
        exit(1);
    }
    sleep(1);
    return 0;//当前进程退出
}
```

运行结果：

![212](F:\学习专用\note\pic\212.png)

说明：

- Linux系统编译程序指令：gcc pthread.c -o pthread -pthread 
- 此例程中需要处理主线程和新线程之间的竞争，sleep(1)用于使主线程休眠1s，防止新线程还未执行完程序主线程就已经退出程序。

**案例：**

循环创建多个子线程

~~~cpp
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<pthread.h>
void *start_func(void *arg)//新线程的主函数，该函数运行结束则线程结束
{
    int i=(int)arg;
	sleep(i);
    printf("%dth pthread: thread id: %lu,pid:%u\n",i+1,pthread_self(),getpid());
    return NULL;
}
int main()
{
    pthread_t tid;
    int i;
    for(i=0;i<5;i++)
    {     
   		int ret=pthread_create(&tid,NULL,start_func,(void *)i);//注意此时创建子线程的最后一个参数是使用值传递
    	if(ret!=0)
    	{
      	  	 printf("pthread_create err: ");
       		 exit(1);
   		 }
    }
    printf("in man: thread id: %lu,pid:%u\n",pthread_self(),getpid());
    sleep(i);
    return 0;//当前进程退出
}
~~~

![213](F:\学习专用\note\pic\213.png)

**案例：**

线程间共享全局变量

~~~cpp
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<pthread.h>
int var=100;
void *start_func(void *arg)//新线程的主函数，该函数运行结束则线程结束
{
    var=300;//修改全局变量的值
    return NULL;
}
int main()
{
    pthread_t tid; 
	printf("before pthread: var:%d\n",var); 
    int ret=pthread_create(&tid,NULL,start_func,NULL);
    if(ret!=0)
    {
        printf("pthread_create err: ");
        exit(1);
    }
    sleep(1);
	printf("after pthread: var:%d\n",var);
    return 0;//当前进程退出
}
~~~

![214](F:\学习专用\note\pic\214.png)

子线程程修改全局变量值后主线程打印全局变量的值为修改后的值，证明共享全局变量

## 4.线程相关函数

### pthread_exit函数

单个线程通过下列三种方式退出，在不终止整个进程的情况下停止它的控制流

- 线程只是从启动例程种返回，返回值是线程的退出码
- 线程可以被同一进程中的其他线程取消
- 线程调用pthread_exit

```c
#include<pthread.h>
void pthread_exit(void *rval_ptr);
```

**比较exit,return,pthread_exit！！！！！**

return ：返回到调用那里去

pthread_exit： 将调用该函数的线程退出

exit： 将进程退出

### **pthread_join函数**

**阻塞等待线程退出**，获取线程退出状态，其作用，对应进程中waitpid()函数

```c
#include<pthread.h>
int pthread_join(pthread_t thread,void **rval_ptr);//返回值：成功返回0，否则返回出错编号
```

通过调用pthread_join自动把线程置于分离状态，**如果线程已经处于分离状态，pthread_join调用就会失败**，返回EINVAL

**例程**：

获取已终止的线程的退出码

```c
#include"apue.h"
#include<pthread.h>
void *thr_fn1(void *arg)
{
    printf("thread 1 returning\n");
    return((void *)111);
}

void *thr_fn2(void *arg)
{
    printf("thread 2 exiting\n");
    pthread_exit((void *)222);
}
int main()
{
    int err;
    pthread_t tid1,tid2;
    void *tret;
    err=pthread_create(&tid1,NULL,thr_fn1,NULL);
    if(err != 0)
        err_quit("can't create thread 1: %s\n",strerror(err));
    err=pthread_join(tid1,&tret);
    if(err != 0)
        err_quit("can't create thread 1: %s\n",strerror(err));
    printf("thread 1 exit code is %d\n",(int)tret);
    err=pthread_create(&tid2,NULL,thr_fn2,NULL);
    if(err != 0)
        err_quit("can't create thread 2: %s\n",strerror(err));
    err=pthread_join(tid2,&tret);
    if(err != 0)
        err_quit("can't create thread 2: %s\n",strerror(err));
    printf("thread 2 exit code is %d\n",(int)tret);
    exit(0);
}

```

运行结果：

![44](F:\学习专用\学习笔记\图片\44.png)

当一个线程通过调用pthread_exit退出或从启动例程中返回，进程中的其他线程可以通过调用pthread_join函数获得该线程的退出状态。

### pthread_detach函数

实现线程分离

~~~cpp
int pthread_detach(pthread_t thread);    成功：0；失败：错误号
~~~

进程若有该机制，将不会产生僵尸进程。僵尸进程的产生主要由于进程死后，大部分资源被释放，一点残留资源仍存于系统中，导致内核认为该进程仍存在。

### pthread_cancel函数

线程可以通过调用pthread_cancel函数来请求取消同一进程中的其他线程

```c
#include<pthread.h>
int pthread_cancel(pthread_t tid);//返回值：成功返回0，否则返回错误编号
```

pthread_cancel并不等待线程终止，它仅仅提出请求

 线程可以安排它退出时需要调用的函数，这与进程可以调用atexit函数安排进程退出时需要调用的函数类似。这样的函数为线程清理处理程序。

### **线程函数和进程函数对比**

![45](F:\学习专用\学习笔记\图片\45.png)

## 5.线程同步

### 5.1互斥量

可以通过使用pthread的互斥接口保护数据，**确保同一时间只有一个线程访问数据**。

互斥量本质上说是一把锁，在访问共享资源前对互斥量进行加锁，在访问完成后释放互斥量上的锁。对互斥量进行加锁后，任何其他试图再次对互斥量加锁的线程将会被阻塞直到当前线程释放该互斥锁。

互斥量用pthread_mutex_t数据类型来表示，在使用互斥量前，必须首先对它进行初始化，可以把他置为常量PTHREAD_MUTEX_INITIALLZER(只对静态分配的互斥量)。也可调用pthread_mutex_init函数进行初始化。如果动态的分配互斥量（如malloc函数）,那么在释放前需要调用pthread_mutex_destroy

```c
#include<pthread.h>
int pthread_mutex_init(pthread_mutex_t *restrict mutex,const pthread_mutexattr_t *restrict attr);
int pthread_mutex_destroy(pthread_mutex_t *mutex);//返回值：成功返回0，否则返回错误编号
```

要默认的属性初始化互斥量，只需把attr设置为NULL。

对**互斥量进行加锁**，需要调用pthread_mutex_lock，如果互斥量已经上锁，调用线程将阻塞直到互斥量被解锁。

对**互斥量解锁**，需要调用pthread_mutex_unlock

```c
#include<pthread.h>
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_trylock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);//返回值：成功返回0，否则返回错误编号
```

pthread_mutex_trylock尝试对互斥量加锁，线程不被阻塞，调用pthread_mutex_trylock时互斥量处于未锁柱状态，则锁柱互斥量，返回0，否则不能锁柱，返回EBUSY

**实例：**

```c
#include<stdlib.h>
#include<pthread.h>
#include<stdio.h>
#include<unistd.h>
void *play(void* arg)
{
    srand(time(0));
    while(1)
    {
        printf("hello ");
        sleep(rand()%5);//模拟长时间操作共享资源，导致cp易主，产生与时间有关的错误
        printf("mutex\n");
    }
}
int main()
{
   pthread_t tid;
    int ret=pthread_create(&tid,NULL,play,NULL);
    if(ret!=0)
    {
        printf("pthread_create err\n");
        exit(1);
    }
    srand(time(0));
    while(1)
    {
        printf("HELLO ");
        sleep(rand()%5);
        printf("MUTEX\n");
    }
    return 0;
}
```

结果：

![215](F:\学习专用\note\pic\215.png)

案例：

使用互斥锁对STDOUT进行加锁

~~~cpp
#include<stdlib.h>
#include<pthread.h>
#include<stdio.h>
#include<unistd.h>
 pthread_mutex_t mutex;//定义锁
void *play(void* arg)
{
    int i=3;
    srand(time(0));
    while(i--)
    {
        pthread_mutex_lock(&mutex);//加锁
        printf("hello ");
        sleep(rand()%5);//模拟长时间操作共享资源，导致cp易主，产生与时间有关的错误
        printf("mutex\n");
        pthread_mutex_unlock(&mutex);
    }
}
int main()
{
    pthread_t tid;
    int i=3;
    pthread_mutex_init(&mutex,NULL);//初始化锁，此时mutex==1
    int ret=pthread_create(&tid,NULL,play,NULL);
    if(ret!=0)
    {
        printf("pthread_create err\n");
        exit(1);
    }
    srand(time(0));
    while(i--)
    {
        pthread_mutex_lock(&mutex);
        printf("HELLO ");
        sleep(rand()%5);
        printf("MUTEX\n");
        pthread_mutex_unlock(&mutex);
    }
    pthread_join(tid,NULL);//阻塞回收子线程
     pthread_mutex_destroy(&mutex);//销毁锁
    return 0;
}
~~~

![216](F:\学习专用\note\pic\216.png)

### 5.2避免死锁

#### **死锁：**

1）.线程对同一个互斥量A加锁两次

![217](F:\学习专用\note\pic\217.png)

~~~cpp
#include<stdlib.h>
#include<pthread.h>
#include<stdio.h>
#include<unistd.h>
 pthread_mutex_t mutex;//定义锁
void *play(void* arg)
{
    int i=3;
    srand(time(0));
    while(i--)
    {
        pthread_mutex_lock(&mutex);//加锁
        printf("hello ");
        sleep(rand()%5);//模拟长时间操作共享资源，导致cp易主，产生与时间有关的错误
        printf("mutex\n");
        pthread_mutex_lock(&mutex);//---------------------线程对同一互斥量两次加锁，造成死锁
    }
}
int main()
{
    pthread_t tid;
    int i=3;
    pthread_mutex_init(&mutex,NULL);//初始化锁，此时mutex==1
    int ret=pthread_create(&tid,NULL,play,NULL);
    if(ret!=0)
    {
        printf("pthread_create err\n");
        exit(1);
    }
    srand(time(0));
    while(i--)
    {
        pthread_mutex_lock(&mutex);
        printf("HELLO ");
        sleep(rand()%5);
        printf("MUTEX\n");
        pthread_mutex_unlock(&mutex);
    }
    pthread_join(tid,NULL);//阻塞回收子线程
     pthread_mutex_destroy(&mutex);//销毁锁
    return 0;
}
~~~



2）线程1拥有A锁，请求获得B锁，线程2拥有B锁，请求获得A锁

![218](F:\学习专用\note\pic\218.png)

#### 避免死锁

对于死锁1：对一个互斥量加锁后，使用了共享资源便立即释放锁，便可避免死锁

对于死锁2：线程1拥有A锁，请求获得B锁，此时使用trylock函数，如果无法获得B锁，则放弃已有的A锁

### 5.3读写锁

读写锁类似于互斥量，不过读写锁允许更高地并行性，特性：**写独占，读共享，写的优先级更高！！！**

**一把读写锁地三种状态：**

- 读模式下加锁（读锁）
- 写模式下加锁（写锁）
- 不加锁

一次只有一个线程可以占有写模式地读写锁，但是多个线程可以同时占有读模式地读写锁

（1）当读写锁是写加锁状态时，在解锁前，所有试图对这个锁加锁的线程都会被阻塞。**写独占**

（2）当读写锁是读加锁时，所有试图以读模式对它进行加锁的线程都可以实现。**读共享**

（3）当读写锁是读加锁时，如果有另外的线程试图以写模式加锁，读写锁会阻塞随后的读模式锁请求，避免读模式锁长期占用，**写优先**

读写锁也叫共享-独占锁，当读写锁以读模式锁住时，它是以共享模式锁住的，当它以写模式锁住时，它是以独占模式锁住的。

**读写锁初始化：**

```c
#include<pthread.h>
int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock,const pthread_rwlockattr_t *restrict attr);
int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);//两者返回值：成功返回0，否则返回错误编号
```

若希望读写锁有默认属性，attr为NULL

pthread_rwlock_init为读写锁分配资源，pthread_rwlock_destroy将释放这些资源

**读模式下加锁，写模式下加锁，解锁：**

```c
#include<pthread.h>
int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);//读模式下加锁
int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);//写模式下加锁
int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);//解锁
//所有的返回值：成功返回0，否则返回错误编号
```

**案例：**

线程1,2进行读操作，线程3,4进行写操作

~~~cpp
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>
#include<pthread.h>
pthread_rwlock_t rwlock;//定义读写锁
int flag=10;
void *rd_play(void *arg)
{
    int i=(int)arg;
    while(1)
    {
        pthread_rwlock_rdlock(&rwlock);
        printf("===read====%dth pthread: pthread_ID = %lu flag = %d\n",i+1,pthread_self(),flag);
        pthread_rwlock_unlock(&rwlock);
        usleep(1000);//让cpu易主，执行其他操作
    }
}
void *wr_play(void *arg)
{
    int i=(int)arg;
   while(1)
    {
        pthread_rwlock_wrlock(&rwlock);
        ++flag;
        printf("----------write-------------%dth pthread: pthread_ID = %lu flag = %d\n",i+1,pthread_self(),flag);
        pthread_rwlock_unlock(&rwlock);
        usleep(1000);
    }
}
int main()
{
    
    pthread_t tid[4];
    pthread_rwlock_init(&rwlock,NULL);//初始化读写锁
    int i=0;
    for(;i<2;i++)
    {
        int ret=pthread_create(&tid[i],NULL,rd_play,(void *)i);
        if(ret!=0)
        {
            printf("pthread_create err: %s\n",strerror(ret));
            exit(1);
        }
    }
    for(;i<4;i++)
    {
        int ret=pthread_create(&tid[i],NULL,wr_play,(void *)i);
        if(ret!=0)
        {
            printf("pthread_create err: %s\n",strerror(ret));
            exit(1);
        }
    }
    for(int j=0;j<4;j++)
    {
        pthread_join(tid[i],NULL);
    }      
    pthread_rwlock_destroy(&rwlock);
    return 0;
}
~~~

![219](F:\学习专用\note\pic\219.png)

### 5.4条件变量

条件变量给多个线程提供一个会合的场所。条件变量和互斥量一起使用，允许线程以无竞争的方式等待特定的条件发生。

条件本身是由互斥量保护的，线程在改变条件状态前必须首先锁住互斥量，其他线程在获得互斥量前不会察觉这种变化，因为必须锁住互斥量后才能计算条件。

**条件变量初始化：**

pthread_cond_t 数据类型代表的条件变量以两种方式初始化，（1）把常量PTHREAD_COND_INITIALIZER赋给静态分配的条件变量，（2）条件变量是动态分配的，可用pthread_cond_init函数初始化

```c
#include<pthread.h>
int pthread_cond_init(pthread_cond_t *rectrict cond,pthread_condattr_t *restrict attr);
int pthread_cond_destroy(pthread_cond_t *cond);//两返回值：成功返回0，否则返回错误编号
```

pthread_cond_init中attr一般默认为NULL

使用pthread_cond_wait等待条件变为真，如果在给定的时间内条件不满足，会生成一个错误码

```c
#include<pthread.h>
int pthread_cond_wait(pthread_cond_t *restrict cond,pthread_mutex_t *restrict mutex);
int pthread_cond_timedwait(pthread_cond_t *restrict cond,pthread_mutex_t *restrict mutex,const struct timespec *restrict timeout);//两返回值：成功返回0，否则返回错误编号
```

传递给pthread_cond_wait的互斥量对条件进行保护，调用者把锁住的互斥量传给函数。

pthread_cond_signal函数唤醒等待该条件的某个线程，pthread_cond_broadcast函数唤醒等待该条件的所有线程

```c
#include<pthread.h>
int pthread_cond_signal(pthread_cond_t *cond);
int pthread_cond_broadcast(pthread_cond_t *cond);//两返回值：成功返回0，否则返回错误编号
```

#### **例程：**生产者消费者模型

线程同步典型的案例即为生产者消费者模型，而借助条件变量来实现这一模型，是比较常见的一种方法。假定有两个线程，一个模拟生产者行为，一个模拟消费者行为。两个线程同时操作一个共享资源（一般称之为汇聚），生产向其中添加产品，消费者从中消费掉产品。

![221](F:\学习专用\note\pic\221.png)

```c
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>
#include<pthread.h>
struct msg
{
    int mun;
    struct msg *next;
};
struct msg *head=NULL;
struct msg *mp=NULL;
//静态初始化一个条件变量和一个互斥量
pthread_cond_t has_product = PTHREAD_COND_INITIALIZER;
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
void *producter(void *arg)
{
    while(1)
    {
        mp=malloc(sizeof(struct msg));
        mp->mun=rand()%400+1;
        printf("---------producnted-----%d\n",mp->mun);
        pthread_mutex_lock(&mutex);//加锁，向全局区写数据
        mp->next=head;
        head=mp;
        pthread_mutex_unlock(&mutex);
        pthread_cond_signal(&has_product);//唤醒等待的线程
        sleep(rand()%3);
    }
}
void *consumer(void *arg)
{
    while(1)
    {
        pthread_mutex_lock(&mutex);//加锁，从全局区取数据
        while(head==NULL)
            pthread_cond_wait(&has_product,&mutex);//等待，直到被唤醒
        mp=head;
        head=mp->next;
        pthread_mutex_unlock(&mutex);
        printf("------consumer-----%d\n",mp->mun);
        free(mp);
        sleep(rand()%3);
    }
}
int main()
{
    pthread_t pid,cid;
    srand(time(0));
    pthread_create(&pid,NULL,producter,NULL);
    pthread_create(&cid,NULL,consumer,NULL);
    pthread_join(pid,NULL);
    pthread_join(cid,NULL);
    return 0;
}
```

![220](F:\学习专用\note\pic\220.png)

#### 例程：哲学家进餐模型

![222](F:\学习专用\note\pic\222.png)

5个哲学家共享一份面，需要一双筷子才能吃面，而他们每人只有1支筷子

振荡：如果每个人都攥着自己左手的锁，尝试去拿右手锁，拿不到则将锁释放。过会儿五个人又同时再攥着左手锁尝试拿右手锁，依然拿不到。如此往复形成另外一种极端死锁的现象——振荡。

​         避免振荡现象：只需5个人中，任意一个人，拿锁的方向与其他人相逆即可(如：E，原来：左：4，右：0 现在：左：0， 右：4)。

## 6.线程使用注意事项

1. 主线程退出其他线程不退出，主线程应调用pthread_exit

2. 避免僵尸线程

   ​	pthread_join

   ​	pthread_detach

3. malloc和mmap申请的内存可以被其他线程释放 

4. 应避免在多线程模型中调用fork除非，马上exec，子进程中只有调用fork的线程存在，其他线程在子进程中均pthread_exit

5. 信号的复杂语义很难和多线程共存，应避免在多线程引入信号机制

# 六. 线程控制

## 1.线程属性

使用pthread_attr_t 结构修改线程默认属性，并把这些属性与创建的线程联系起来。

**属性初始化：**

```c
#include<pthread.h>
int pthread_attr_init(pthread_attr_t *attr);//返回值：成功0，出错返回错误编号
```

**去除初始化：**

```c
#include<pthread.h>
int pthread_attr_destroy(pthread_attr_t *attr);//返回值：成功0，出错返回错误编号
```

**分离线程：**

使用pthread_attr_setdetachstate函数把线程属性detachstate设置为以下两个合法值之一：

- 设置为PTHREAD_CREATE_DETACHED,以分离状态启动线程
- 设置为PTHREAD_CREATE_JOINABLE,正常启动线程

```c
#include<pthrad.h>
int pthread_attr_getdetachstate(const pthread_attr_t *restrict attr,int *detachstate);
int pthread_attr_setdetachstate(pthread_sttr_t *attr,int detachstate);//返回值：成功0，出错返回错误编号
```

**例程：以分离状态创建线程**

```c
#include<pthread.h>
#include"apue.h"
int makethread(void *(*fn)(void *),void *arg)
{
    int err;
    pthread_t tid;
    pthread_attr_t attr;
    err=pthread_attr_init(&err);
    if(err != 0)
        return(err);
    err=pthread_attr_setdetach(&attr,PTHREAD_CREATE_DETACHED);
    if(err == 0)
        err = pthread_create(&tid,&attr,fn,arg);
    pthread_attr_destroy(&attr);
    return(err);
}
```

## 2.同步属性

### 2.1互斥量属性

**初始化：**

```c
#include<pthread.h>
int pthread_mutexattr_init(pthread_mutexattr_t *attr);//返回值：成功0，出错返回错误编号
```

**回收：**

```c
#include<pthread.h>
int pthread_mutexattr_destroy(pthread_mutexattr_t *attr);//返回值：成功0，出错返回错误编号
```

#### 互斥量**进程共享属性和类型属性**

**进程共享属性：**pthread_mutexattr_getpshared查询进程共享属性，pthread_mutexattr_setpshared修改进程共享属性

```c
#include<pthread.h>
int pthread_mutexattr_getpshared(const pthread_mutexattr_t *restrict attr,int *restrict pshared);
int pthread_mutexattr_setpshared(const pthread_mutexattr_t *attr,int *pshared);//返回值：成功0，出错返回错误编号
```

**类型属性：**pthread_mutexattr_gettype查询互斥量类型属性，pthread_mutexattr_settype修改互斥量类型属性

```c
#include<pthread.h>
int pthread_mutexattr_gettype(const pthread_mutexattr_t *restrict attr,int *restrict type);
int pthread_mutexattr_settype(pthread_mutexattr_t *attr,int type);//返回值：成功0，出错返回错误编号
```

### 2.2读写锁属性

**读写锁属性初始化：**

```c
#include<pthread.h>
int pthread_rwlockattr_init(pthread_rwlockattr_t *attr);//返回值：成功0，出错返回错误编号
```

**回收：**

```c
#include<pthread.h>
int pthread_rwlockattr_destroy(pthread_rwlockattr_t *attr);//返回值：成功0，出错返回错误编号
```

读写锁支持唯一属性是进程共享属性，与互斥量的进程共享属性相同。

**读取设置读写锁的共享属性：**

```c
#include<pthread.h>
int pthread_rwlockattr_getpshared(const pthread_rwlockattr_t *restrict attr,int *restrict pshared);
int pthread_rwlockattr_setpshared(const pthread_rwlockattr_t *attr,int *pshared);//返回值：成功0，出错返回错误编号
```

### 2.3条件属性

**条件属性初始化：**

```c
#include<pthread.h>
int pthread_condattr_init(pthread_condattr_t *attr);//返回值：成功0，出错返回错误编号
```

**回收：**

```c
#include<pthread.h>
int pthread_condattr_destroy(pthread_condattr_t *attr);//返回值：成功0，出错返回错误编号
```

条件属性支持唯一属性是进程共享属性。

**读取设置读写锁的共享属性：**

```c
#include<pthread.h>
int pthread_condattr_getpshared(const pthread_condattr_t *restrict attr,int *restrict pshared);
int pthread_condattrr_setpshared(const pthread_condattr_t *attr,int *pshared);//返回值：成功0，出错返回错误编号
```

## 3.重入

**线程安全：**

如果一个函数在同一时刻可以被多个线程安全的调用，称该函数是线程安全的。

如果一个函数对多线程来说是可重入的，则说这个函数是线程安全的，但并不能说明对信号处理程序来说该函数也是可重入的。

**异步-信号安全：**

如果函数对异步信号处理程序的重入是安全的，那么久可以说函数是异步-信号安全的

## 4.线程私有数据

线程私有数据（也称线程特定数据）是存储和查询与某个线程相关的数据的一种机制。每个线程可以独立的访问数据副本，而不需要担心与其他线程的同步访问问题。

**线程私有数据访问权的设立：键**

在分配线程私有数据前，需要创建与该数据关联的键，这个键用于获取对线程私有数据的访问权。

```c
#include<pthread.h>
int pthread_key_create(pthread_key_t *keyp,void (*destructor)(void *));//返回值：成功0，出错返回错误编号
```

创建的键存放在keyp指向的内存单元，这个键可以被进程中的所有线程使用，但每个线程把这个键与不同的线程私有数据地址进行关联。

**取消键与线程私有数据值得关联：****取消键**

```c
#include<pthread.h>
int pthread_key_delete(pthread_key_t *key);//返回值：成功0，出错返回错误编号
```

**键和线程私有数据关联：**

```c
#include<pthread.h>
void *pthread_getspecific(pthread_key_t key);//返回值：线程私有数据值，若没有值与键关联返回NULL
int pthread_setspecific(pthread_key_t key,const void *value);//返回值：成功0，出错返回错误编号
```

## 5.取消选项

有两个线程属性并没有包含在pthread_attr_t 结构中，他们是**可取消状态**和**可取消类型**

这两个属性影响着线程在相应pthread_cancel函数调用时所呈现的行为。

### **可取消状态属性**

可以是PTHREAD_CANCEL_ENABLE,也可以是PTHREAD_CANCEL_DISABLE

pthread_setcancelstate修改可取消状态

```c
#include<pthread.h>
int pthread_setcancelstate(int state,int *lodstate);//返回值：成功0，出错返回错误编号
```

### 可取消类型

也称为延迟取消，调用pthread_cancel后，在线程到达取消点前，并不会出现真正的取消。pthread_setcanceltype来修改取消类型

```c
#include<pthread.h>
int pthread_setcanceltype(int type,int *oldtype);//返回值：成功0，出错返回错误编号
```

## 6.线程和信号

每个线程都有自己的信号屏蔽字，但是信号的处理是进程中所有线程共享的。

**阻止信号发送：**

```c
#include<pthread.h>
int pthread_sigmask(int how,const sigset_t *restrict set,sigset_t *restrict oset);//返回值：成功0，出错返回错误编号
```

**等待信号发生：**

```c
#include<pthread.h>
int sigwait(const sigset_t *restrict set,int *restrict signop);//返回值：成功0，出错返回错误编号
```

set参数指出线程等待的信号集，signop指向的整数将作为返回值，表明发送信号的数量

## 7.线程和fork

**清除锁状态：**

通过调用pthread_atfork函数建立fork处理函数

```c
#include<pthread.h>
int pthread_atfork(void (*prepare)(void),void (*parent)(void),void (*child)(void));//返回值：成功0，出错返回错误编号
```

pthread_atfork可安装三个帮助清理锁的函数：

- prepare fork处理程序由父进程在fork创建子进程前调用，获取父进程定义的所有锁
- parent fork处理程序是在fork创建了子进程以后，但在fork返回之前在父进程环境中调用，对prepare fork处理程序获得的所有锁进行解锁
- child fork处理程序在fork返回之前在子进程环境中调用，释放prepare fork处理程序获得的所有锁。

**例程：**

```c
#include<pthread.h>
#include"apue.h"

pthread_mutex_t lock1=PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t lock2=PTHREAD_MUTEX_INITIALIZER;//初始化互斥量锁

void prepare(void)
{
    printf("preparing locks----------\n");
    pthread_mutex_lock(&lock1);
    pthread_mutex_lock(&lock2);//准备加锁
}
void parent(void)
{
    printf("parent unlocking locks------\n");
    pthread_mutex_unlock(&lock1);
    pthread_mutex_unlock(&lock2);//父处理程序解锁
}
void child(void)
{
    printf("child unlocking locks------\n");
    pthread_mutex_unlock(&lock1);
    pthread_mutex_unlock(&lock2);//子处理程序解锁
}
void *thr_fn(void *arg)
{
    printf("thread started ------\n");
    pause();
    return 0;
}
int main(void)
{
    int err;
    pid_t pid;
    pthread_t tid;
    #if defined(BSD)||defined(MACOS)
        printf("pthread_atfork is unsupported\n");
    #else
    	if((err=pthread_atfork(prepare,parent,child)) != 0)//创建fork处理程序
            err_exit(err,"can't install fork handlers");
    	err=pthread_create(&tid,NULL,thr_fn,0);
    	if(err != 0)
            err_exit(err,"can't create pthread");
    	sleep(2);
    	printf("parent about to fork ----\n");
    	if((pid = fork())<0)
            err_quit("fork failed");
    	else if(pid==0) 
            printf("child return fork\n");//子进程
    	else
            sleep(1);
            printf("parent return fork\n");//父进程
    #endif
    	exit(0);
}
```

**运行结果：**

![46](F:\学习专用\学习笔记\图片\46.png)

（1）程序中定义了两个互斥量，lock1和lock2，prepare fork处理程序获取这两把锁，child fork处理程序在子进程中释放锁，parent fork在父进程中释放锁

（2）prepare fork处理程序在调用fork后运行，child fork处理程序在fork调用返回到子进程之前运行，parent fork处理程序在fork调用返回给父进程前运行、

# 七. 进程环境

## 进程相关的概念！

### **1）.程序和进程的区别**

**程序**，是编译好的二进制文件，**在磁盘上**，不占用系统资源（cpu,内存，打开的文件，设备，锁）

**进程**，是一个抽象的概念，与操作系统原理联系紧密。进程是活跃的程序，**占用系统资源**（程序运行起来会产生一个进程）。 

### **2）.中央处理器工作原理**

![182](F:\学习专用\note\pic\182.png)

**MMU工作原理**

![183](F:\学习专用\note\pic\183.png)

### 3）.进程控制块PCB（进程描述符）

 每一个进程在内核中都有一个进程控制块来**维护进程相关的消息**，Linux内核的进程控制块是task_struct结构体

**主要内部成员：**

- **进程ID**      每个进程有唯一的id
- **进程的状态**    有就绪、运行、挂起、停止等状态

![184](F:\学习专用\note\pic\184.png)

- **描述虚拟地址空间的信息**
- 描述控制终端信息
- 当前工作目录
- umask掩码
- **文件描述符表**，包含很多指向file结构体的指针  ----------->file struct { 打开的文件的描述信息}
- 和信息相关的信息
- 用户id和组id
- 回话和进程组

### 4）.环境变量

定义进程的运行环境

**重要的环境变量：**

- PATH  可执行文件的搜索路径，通过echo $PATH查看
- SHELL 当前shell，通常为/bin/bash
- HOME 当前家目录
- LANG 当前语言 

环境字符串的形式通常如下：

name=value

ISO C定义了一个函数getenv，**可以用其取环境变量值**，但该标准又称环境的内容是由实现定义的。

```c
#include<stdlib.h>
char *getenv(const char *name);//指向与name关联的value的指针，未找到返回NULL
```

此函数返回一个指针，他指向name=value字符串中的value。我们应当使用getenv从环境中取一个指定环境变量的值，而不是直接访问environ

```c
#include<stdlib.h>
int putenv(char *str);//成功返回0，错误返回非0
int setenv(const char *name,const char*value,int rewrite);//成功返回0，错误返回-1
int unsetenv(const char *nam e);//成功返回0，错误返回-1
```

1）putenv取形式为name=value的字符串，将其放到环境表中，如果name已经存在，则先删除其原来的定义。

2）setenv将name设置为value。如果在环境中name已经存在，那么（a）若rewrite非0，则先删除其现有的定义，（b）若rewrite为0，则不删除其现有定义。

3）unsetenv删除name的定义。

## 1.	main函数

（1）c程序总是从main函数开始执行，main函数原型：int main(int argc,char *argv[]);----------argc是命令行参数的数目，argv是指向参数的各个指针所构成的数组

（2）在内核执行c程序时（使用exec函数），在调用main前先调用一个特殊的启动例程。启动例程从内核取得命令行参数和环境变量值，然后为调用main函数做好准备。

## 2.	进程终止

### 1）.	进程终止方式

**5种为正常终止：**

1）从main返回; 

2）调用exit；

3）调用_exit或_Exit; 

4）最后一个线程从其启动例程返回；

5）最后一个线程调用pthread_exit

**异常终止3种方式：**

1）调用abord；

2）接到一个信号并终止；

3）最后一个线程对取消请求做出响应

### 2）.	exit函数

有三个函数用于正常终止一个程序：_exit和 _Exit立即进入内核，exit则先执行一些清理处理（包括调用执行各终止处理程序，关闭所有标准I/O流等），然后进入内核。

```c
#include<stdlib.h>
void exit(int status);
void _Exit(int status);//exit和_Exit是ISO C说明的
#include<unistd.h>
void -exit(int status);//_exit是POSIX.1说明
```

三个exit函数都带一个整型参数，称为终止状态。1）这些函数不带终止状态，2）main执行一个无返回值的return，3）main没有声明返回类型为整型，这三种状态下进程的终止状态是未定义的。未定义时一般情况下默认终止状态为0

### 3）.	atexit函数

一个进程有多个**终止处理程序**，由exit函数自动调用，并调用atexit函数来登记这些函数。exit调用这些函数的顺序与它们登记时候的顺序**相反**

```c
#include<stdlib.h>
int atexit(void(*func)(void));//若成功返回0，出错返回非0
```

```c
/*例程：登记终止处理程序
*/
#include"apue.h"
static void exit_1(void);
static void exit_2(void);
int main()
{
    if(atexit(exit_1) != 0)//记录exit_1的终止处理程序
    	err_sys("can't register exit_1");
    if(atexit(exit_2) != 0)
        err_sys("can't register exit_2");
    if(atexit(exit_2) != 0)
        err_sys("can't register exit_2");
    return(0);
}
static void exit_1(void)
{
    printf("handle exit_1\n");
}
static void exit_2(void)
{
    printf("handle exit_2\n");
}
```

**输出结果：**

![1](F:\学习专用\学习笔记\图片\1.png)

**分析：**第二个终止处理程序被登记两次，所以也会被调用两次。

## 3.	命令行参数

当执行一个程序时，调用exec的进程可将命令行参数传递给该新程序。

```c
#include<apue.h>
int main(int argc,char *argv[])
{
    int i;
    for(i=1;i<argc;i++)
    {
        printf("argv[%d]:%s\n",i,argv[i]);
    }
    exit(0);
}
```

**输出结果：**

![2](F:\学习专用\学习笔记\图片\2.png)

## 4.	环境表

在历史上，大多数UNIX系统支持main函数带有三个参数，其中第三个参数是环境表的地址：

```c
int main(int argc,char *argv[],char *envp[]);
```

现在基本没有用到environ指针，main函数一般都只有两个参数，通常用getenv和putenv函数来访问特定的环境变量，而不是用environ变量。

## 5.	C程序的存储空间布局

**C程序一直由下列几部分组成：**

1) **正文段**。这是由CPU执行的**机器指令部分**。通常，正文段是可共享的，所以即使是频繁执行的程序（如文本编辑器，C编译器和shell等）在存储器中也需有一个副本，另外，正文段常常是只读的，以防止程序由于意外而修改其指令。

2) **初始化数据段**。通常将此段称为数据段，它包含了程序中需明确地赋初值的变量。例如，C程序中任何函数之外的声明，使变量的初值放在初始化数据段中。

```c
int max = 43;
```

3) **非初始化数据段**。通常将此段称为bss段，这一名称来源于早期汇编程序一个操作符，意思是“由符号开始的块”（block started by symbol），在程序开始执行之前，内核将此段中的数据初始化为0或空指针。函数外的声明，使变量存放在非初始化数据段中。

```c
long sum[100];
```

4) **栈**。自动变量以及每次函数调用时所需保存的信息都存放在此段中。每次函数调用时，其返回地址以及调用者的环境信息（如某些机器寄存器的值）都存放在栈中。然后，最近被调用的函数在栈上为其自动和临时分配存储空间。通过以这种方式使用栈，C递归函数可以工作。递归函数每次调用自身时，就用一个新的栈帧，因此一次函数调用实例中的变量集不会影响另一次函数调用实例中的变量。

5) **堆**。通常在堆中进行**动态存储分配**。由于历史上形成的惯例，堆位于非初始化数据段和栈之间。

对于32位Intel x86处理器上的Linux，正文段从0x08048000单元开始，栈底则在0xC0000000之下开始（在这种特定结构中，栈从高地址向低地址方向增长）。堆顶和栈顶之间的未用虚地址空间很大。

![1528093718325](C:\Users\Summer\AppData\Local\Temp\1528093718325.png)



未初始化的数据段的内容不存放在磁盘程序文件中。其原因是，内核在程序开始运行前将它们都设置为0。需要存放在磁盘程序文件中的段只有正文段和初始化数据段。

## 6. 	存储器分配

ISO C说明 了三个用于存储空间动态分配的函数

1)malloc   分配指定字节数的存储区。此存储区中的初始值不确定 

2）calloc 为指定数量具指定长度的对象分配存储空间，该空间的每一位都初始化为0

3）realloc 更改以前分配区的长度（增加或减少）

```c
#include<stdlib.h>
void *malloc(size_t size);
void *calloc(size_t nobj,xize_t size);
void *realloc(void *ptr,size_t newsize);//返回值：成功返回非空指针，出错返回NULL
void free(void *ptr);
```

注意：释放一个已经释放了的块，调用free时所用的指针不是三个alloc函数的返回值，如若一个进程调用malloc

函数，但却忘记调用free函数，那么该进程占用的存储器就会连续增加，这被称为**泄露**

## 7.	setjmp和longjmp函数

c中，goto语句不能跨越函数，执行这类跳转功能的是函数setjmp和longjmp。用于处理发生在深层嵌套函数调用中的出错情况。在栈上跳过若干调用帧，返回到当前调用路径的某个函数中。

```c
#include<setjmp.h>

int setjmp(jmp_buf env);//返回值：若直接调用则返回0，若从longjmp调用返回则返回非0值的longjmp中的val值

/*jmp_buf为特殊类型，是某种形式的数组，存放在调用longjmp时能用来恢复栈状态的所有信息*/

void longjmp(jmp_buf env,int val);//调用此函数则返回到语句setjmp所在的地方，其中env 就是setjmp中的 env，而val 则是使setjmp的返回值变为val

```

## 8.回收子进程！

**孤儿进程**

父进程先于子进程结束，则子进程成为孤儿进程，子进程的父进程成为**init进程**，称为init进程领养孤儿进程

~~~c++
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
int main()
{
    pid_t pid;
    pid=fork();
    if(pid==-1)
    {
        perror("fork err");
        exit(1);
    }
    else if(pid==0)
    {
        printf("子进程---其id为： %u,父进程id为： %u\n",getpid(),getppid());
        sleep(3);
         printf("子进程---其id为： %u,父进程id为： %u\n",getpid(),getppid());
    }
    else
    {
        printf("父进程---其id为： %u,父进程id为： %u\n",getpid(),getppid());
        sleep(1);
    }
    return 0;
}
~~~

![190](F:\学习专用\note\pic\190.png)

**僵尸进程**

子进程终止，父进程尚未回收，子进程残留资源（PCB）存放于内核中，成为僵尸进程。通过wait和waitpid函数可回收子进程

~~~cpp
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
int main()
{
    pid_t pid;
    pid=fork();
    if(pid==-1)
    {
        perror("fork err");
        exit(1);
    }
    else if(pid==0)
    {
        printf("子进程---其id为： %u,父进程id为： %u\n",getpid(),getppid());
    }
    else
    {
        while(1)
        {
             printf("父进程---其id为： %u,父进程id为： %u\n",getpid(),getppid());
       		 sleep(1);
        }
    }
    return 0;
}
~~~

![191](F:\学习专用\note\pic\191.png)

案例：用wait函数回收子进程

~~~cpp
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
int main()
{
    pid_t pid,waitp;
    pid=fork();
    if(pid==-1)
    {
        perror("fork err");
        exit(1);
    }
    else if(pid==0)
    {
        printf("子进程---其id为： %u,父进程id为： %u\n",getpid(),getppid());
    }
    else
    {
        waitp=wait(NULL);
        if(waitp==-1)
        {
            perror("wait err");
            exit(1);
        }
        while(1)
        {
             printf("父进程---其id为： %u,父进程id为： %u\n",getpid(),getppid());
       		 sleep(1);
        }
    }
    return 0;
}
~~~

![192](F:\学习专用\note\pic\192.png)



# 八.进程控制！！！

## 1.进程标识符

每一个进程都有一个非负整形表示的唯一进程ID。系统中的专用进程，ID为0进程一般为调度进程，也叫交换进程，或系统进程（为内核的一部分）。进程ID1通常为init进程，在自举过程结束时由内核调用。init通常读与系统有关的初始化文件，并将系统引导到一个状态（例如多用户）。进程ID 2是页守护进程。此进程负责支持虚拟存储系统的分页操作。

```cpp
#include<unistd.h>
pid_t getpid(void);//返回值：调用进程的进程ID
pid_t getppid(void);//返回值：调用进程的父进程ID
uid_t getuid(void);//返回值：调用进程的实际用户ID
uid_t geteuid(void);//返回值：调用进程的有效用户ID
gid_t getgid(void);//返回值：调用进程的实际组ID
gid_t getegid(void);//返回值：调用进程的有效组ID
```

# ![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)2.fork函数

```cpp
#include<unistd.h>
pid_t fork(void);//返回值：子进程中返回0，父进程中返回子进程ID，出错返回-1
```

由fork创建的新进程被称为子进程。**fork函数被调用一次，但返回两次**。子进程返回0，父进程返回值是新子进程的进程ID。

fork函数返回两个值

- 返回一个大于0的值给父进程
- 返回0给子进程
- 返回-1说明fork失败了

**（1）父、子进程的区别：**！！！！

- 相同点（0-3G虚拟地址空间）：全局变量，代码段，栈，堆，环境变量，用户id，宿主目录，进程工作目录，信息处理方式
- 不同点：进程id，fork返回值，父进程id（父进程的父进程id为shell的id），进程运行时间，闹钟（定时器），未决信号集

**注意:**

- 父子进程间遵守读时共享写时复制的原则（只需要读值时，共享一个物理内存，需要进行写操作时，会复制一块内存给子进程），这样可以节省内存开销
- 父子进程共享：1.文件描述符，2.mmap建立的映射区
- fork后父进程先执行还是子进程先执行不确定，取决于内核所使用的调度算法

**（2）fork的两种用法 ：**

- a.父子进程同时执行不同的代码段，父进程等待客户端的服务请求。子进程处理此请求。父进程等待下一个服务请求到达。
- b.一个进程要执行一个不同的程序，shell中常见，子进程从fork返回后立即调用exec

（3）fork（）函数通过系统调用创建一个与原来进程几乎完全相同的进程，这个新产生的进程称为子进程。一个进程调用fork（）函数后，系统先给新的进程分配资源，例如存储数据和代码的空间。然后把原来的进程的所有值都复制到新的新进程中，相当于克隆了一个自己。

### 案例：

~~~c++
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
int main()
{
    pid_t pid;
    printf("xxxxxxxxxxxxxxxx\n");
    pid=fork();
    if(pid==-1)
    {
        perror("fork err");
        exit(1);
    }
    else if(pid==0)
    {
        printf("子进程---其id为： %u,父进程id为： %u\n",getpid(),getppid());
    }
    else
    {
        printf("父进程---其id为： %u,父进程id为： %u\n",getpid(),getppid());
        sleep(1);
    }
    printf("yyyyyyyyyyyyyyyyy\n");
    return 0;
}
~~~

![185](F:\学习专用\note\pic\185.png)

~~~c++
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
int main()
{
    pid_t pid;
    int i=0;
    printf("xxxxxxxxxxxxxxxx\n");
    for(i;i<5;i++)//创建5个子进程
    {
        pid=fork();
        if(pid==-1)
        {
            perror("fork err");
            exit(1);
        }
        else if(pid==0)
        {
            break;//出口1，子进程出口
        }
    }//出口2 ，父进程出口
    if(i<5)
    {
        sleep(i);
        printf("子进程---其id为： %u,父进程id为： %u\n",getpid(),getppid());
    }
    else
    {
         sleep(i);
         printf("父进程---其id为： %u,父进程id为： %u\n",getpid(),getppid());
    }
    return 0;
}
~~~

![186](F:\学习专用\note\pic\186.png)

## 3.exec函数！

子进程调用exec函数以执行另一个程序，当进程调用一个exec函数时，该进程的用户空间代码和数据完全被新程序替换，从新程序的启动例程开始执行。但进程ID不变

### execlp函数

加载一个进程，借助PATH环境变量

~~~cpp
int execlp(const char *file,const char *arg,...);//成功无返回，失败-1
~~~

参数1：要加载的程序的名字。需要配合PATH环境变量使用，若PATH中所有目录搜索后没有参数1则出错

该函数通常用来调用系统程序，如ls,cat ,data,pwd

**案例：**

~~~cpp
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
int main()
{
    pid_t pid;
    pid=fork();
    if(pid==-1)
    {
        perror("fork err");
        exit(1);
    }
    else if(pid == 0)
    {
        execlp("ls","ls","-l","-a",NULL);//将ls-al 命令传给ls，子进程执行ls -al命令
    }
    else
    {
        printf("父进程\n");
        sleep(1);
    }
    return 0;
}
~~~

![187](F:\学习专用\note\pic\187.png)

### execl函数

加载一个进程，通过路径+程序名 来加载,主要用于执行用户自定义的程序

~~~cpp
int execl(const char *path,const char *arg,...);//成功无返回，失败-1
~~~

对比execlp，如加载"ls-al"

~~~cpp
execlp("ls","ls","-al",NULL);
execlp("/bin/ls","ls","-al",NULL);//参数1为绝对路径搜索
~~~

**案例：**

~~~
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
int main()
{
    pid_t pid;
    pid=fork();
    if(pid==-1)
    {
        perror("fork err");
        exit(1);
    }
    else if(pid == 0)
    {
        execl("/bin/ls","ls","-l","-a",NULL);//将ls-al 命令传给ls，子进程执行ls -al命令
    }
    else
    {
        printf("父进程\n");
        sleep(1);
    }
    return 0;
}
~~~

![188](F:\学习专用\note\pic\188.png)

**案例：执行自定义的hello程序**

~~~cpp
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
int main()
{
    pid_t pid;
    pid=fork();
    if(pid==-1)
    {
        perror("fork err");
        exit(1);
    }
    else if(pid == 0)
    {
        execl("/home/summer/zx/hello","hello",NULL);//将ls-al 命令传给ls，子进程执行ls -al命令
    }
    else
    {
        printf("父进程\n");
        sleep(1);
    }
    return 0;
}
~~~

![189](F:\学习专用\note\pic\189.png)

## 4.vfork函数

vfork用于创建一个新进程，与fork区别：

它不将父进程的地址空间完全复制到子进程，因为子进程会立即调用exec（或exit）。于是也就不会存访改地址空间。

**例：**

```cpp
#include "apue.h"
int glob = 6;
int main(void)
{
  int var;
  pid_t pid;
  var = 90;
  printf("before vfork\n");
  if((pid = vfork())<0)
    err_sys("vfork false");
  else if(pid == 0)
  {
      var++;
      glob++;
      printf("child pid: %d(0x%x)\n",getpid(),getpid());
      _exit(0);
  }
  //----------parent continue here
  printf("father pid:%d glob = %d var = %d\n",getpid(),glob,var);
}
```

运行结果

![img](https://img-blog.csdn.net/20180614102939286?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5neGlhZmxs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 5.wait与waitpid函数！！

调用wait或waitpid的进程可能发生的情况：

- 如果其所有子进程都还在运行，则阻塞
- 如果一个子进程已终止，正等待父进程获取其终止状态，则取得该子进程的终止状态立即返回。
- 如果它没有任何子进程，则立即出错返回

注意：

wait只能回收一个子进程

### wait函数

```cpp
#include <sys/types.h> /* 提供类型pid_t的定义 */
#include <sys/wait.h>
pid_t wait(int *status)
```

进程一旦调用了wait，就立即**阻塞**自己，由wait自动分析是否当前进程的某个子进程已经退出，如果让它找到了这样一个已经变成僵死的子进程，wait就会收集这个子进程的信息，并把它彻底销毁后返回；如果没有找到这样一个子进程，wait就会一直阻塞在这里，直到有一个出现为止。

参数status用来保存被收集进程退出时的一些状态，它是一个指向int类型的指针。但如果我们对这个子进程是如何死掉的毫不在意，只想把这个**僵死进程**消灭掉，（事实上绝大多数情况下，我们都会这样想），我们就可以设定这个参数为NULL，就象下面这样：

pid = wait(NULL);

**如果成功，wait会返回被收集的子进程的进程ID，如果调用进程没有子进程，调用就会失败，此时wait返回-1，同时errno被置为ECHILD。**

wait调用例程：

```cpp
/* wait1.c */
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>
#include <stdlib.h>
main()
{
        pid_t pc,pr;
        pc=fork();
 
        if(pc<0) /* 如果出错 */
              printf("error ocurred!\n");
       else if(pc==0){ /* 如果是子进程 */
              printf("This is child process with pid of %d\n",getpid());
              sleep(1); /* 睡眠1秒钟 */
            }
        else{ /* 如果是父进程 */
              pr=wait(NULL); /* 在这里等待 */
              printf("I catched a child process with pid of %d\n",pr);
        }
       exit(0);
}
```

运行结果

![img](https://img-blog.csdn.net/2018061410301947?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5neGlhZmxs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

可以明显注意到，在第2行结果打印出来前有1秒钟的等待时间，这就是我们设定的让子进程睡眠的时间，只有子进程从睡眠中苏醒过来，它才能正常退出，也就才能被父进程捕捉到。父进程的等待时间随子进程睡眠时间变化而变化。

**参数status：**

如果参数status的值不是NULL，wait就会把子进程退出时的状态取出并存入其中，这是一个整数值（int），指出了子进程是正常退出还是被非正常结束的（一个进程也可以被其他进程用信号结束），以及正常结束时的返回值，或被哪一个信号结束的等信息。由于这些信息被存放在一个整数的不同二进制位中，所以用常规的方法读取会非常麻烦，人们就设计了一套专门的宏（macro）来完成这项工作，下面我们来学习一下其中最常用的两个：

1、WIFEXITED(status) 为非0--->进程正常结束

​	WEXITSTATUS(status) 如上宏为真，使用此宏-->获得进程退出状态（exit的参数），如果子进程调用exit(5)退        出，WEXITSTATUS(status) 就会返回5

2、WIFSIGNALED(status)为非0--->进程异常终止

WTERMSIG(status)如上宏为真，使用此宏-->获取使进程终止的那个信号的编号 。

**案例：**

```cpp
![193](F:\学习专用\note\pic\193.png)/* wait2.c */
#include <sys/types.h>
#include <sys/wait.h>
#include<stdio.h>
#include<stdlib.h>
#include <unistd.h>
int main()
{
       int status;
       pid_t pc,pr;     
       pc=fork();
       if(pc<0) 
         	printf("error ocurred!\n");
       else if(pc==0){ 
            //execl("/home/summer/zx/abort","abort",NULL);//调用异常程序
       	    printf("This is child process with pid of %d\n",getpid());              
	    	exit(3); /* 子进程返回3 */
        }
       else{ 
            pr=wait(&status);
           	if(WIFEXITED(status)){ /* 如果WIFEXITED返回非零值 */
             	 printf("the child process %d exit normally.\n",pr);
            	 printf("child exit with %d.\n",WEXITSTATUS(status));
            }
           if(WIFSIGNALED(status))/* 程序异常退出 */
           {
                printf("the child process %d exit abnormally.\n",pr);
               printf("child killed by %d.\n",WTERMSIG(status));
           }      
        }
	return 0;
}
```

运行结果

![193](F:\学习专用\note\pic\193.png)

![194](F:\学习专用\note\pic\194.png)

### **waitpid函数**

```cpp
#include <sys/types.h> /* 提供类型pid_t的定义 */
#include <sys/wait.h>
pid_t waitpid(pid_t pid,int *statusp,int options)；//返回值： 成功pid 失败-1 返回0值：参3传WNOHANG，且子进程尚未结束
```

从本质上讲，系统调用waitpid和wait的作用是完全相同的，但waitpid多出了两个可由用户控制的参数pid和options，从而为我们编程提供了另一种更灵活的方式。

**1.pid：**从参数的名字pid和类型pid_t中就可以看出，这里需要的是一个进程ID。但当pid取不同的值时，在这里有不同的意义。

​         **pid > 0**时，只等待进程ID等于pid的子进程，不管其它已经有多少子进程运行结束退出了，只要指定的子进程还没有结束，waitpid就会一直等下去。

​         **pid == -1**时，等待任何一个子进程退出，没有任何限制，此时waitpid和wait的作用一模一样。

​         pid == 0时，等待同一个进程组中的任何子进程，如果子进程已经加入了别的进程组，waitpid不会对它做任何理睬。

​         pid<-1时，等待一个指定进程组中的任何子进程，这个进程组的ID等于pid的绝对值。

**2.statusp**：与wait函数中status相同

**3.options**：options提供了一些额外的选项来控制waitpid

WNOHANG参数调用waitpid，非阻塞回收子进程。

options为0 与wait相同，阻塞回收子进程

wait就是经过包装的waitpid，察看<内核源码目录>:

```cpp
static inline pid_t wait(int * wait_stat)
{
    return waitpid(-1,wait_stat,0);
}
```

**waitpid函数和wait函数的区别：**

**返回值和错误**

waitpid的返回值比wait稍微复杂一些，一共有3种情况：

​          1、当正常返回的时候，waitpid返回收集到的子进程的进程ID；

​          2、如果设置了选项WNOHANG，而调用中waitpid发现没有已退出的子进程可收集，则返回0；

​          3、如果调用中出错，则返回-1，这时errno会被设置成相应的值以指示错误所在；

当pid所指示的子进程不存在，或此进程存在，但不是调用进程的子进程，waitpid就会出错返回，这时errno被设置为ECHILD；

**不同功能**

watpid函数提供了wait函数没有提供的三个功能

1. waitpid可等待一个特定的进程，而wait则返回任一终止子进程的状态。
2. waitpid提供一个wait的**非阻塞版本**。有时用户希望取得一个子进程的状态，但不想阻塞。
3. waitpid支持作业控制（利用WUNTRACED和WCONTINUED选项）

```cpp
/* waitpid例程1 */
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>
main()
{
        pid_t pc, pr; 
        pc=fork();
        if(pc<0) /* 如果fork出错 */
               printf("Error occured on forking.\n");
        else if(pc==0){ /* 如果是子进程 */
               sleep(10); /* 睡眠10秒 */
              exit(0);
        }
        /* 如果是父进程 */
       do{
               pr=waitpid(pc, NULL, WNOHANG); /* 使用了WNOHANG参数，waitpid不会在这里等待，返回回收到的子进程id，没有回收到返回0*/
               if(pr==0){ /* 如果没有收集到子进程 */
                  printf("No child exited/n");
                   sleep(1);
               }
        }while(pr==0); /* 没有收集到子进程，就回去继续尝试 */
        if(pr==pc)
               printf("successfully get child %d/n", pr);
        else
               printf("some error occured/n");
}
```

**运行结果**

![img](https://img-blog.csdn.net/20180614103116967?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5neGlhZmxs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```cpp
 
/*waitpid例程2*/
#include<sys/types.h>
#include<sys/wait.h>
#include<unistd.h>
#include<stdio.h>

#include<errno.h>
#include<stdlib.h>
pid_t Fork(void)
{
    pid_t pid;//创建进程
    if((pid = fork())<0)
        perror("Fork error");
    return pid;
}
void main()
{
   int status;
   pid_t pid;
   printf("hello\n");
   pid=Fork();
   printf("%d\n",!pid);
   if(pid != 0 )
   {
       if(waitpid(-1,&status,0)>0)//阻塞回收子进程
       {
           if(WIFEXITED(status)!=0)
		   printf("%d\n",WEXITSTATUS(status));
       }
   }
    printf("bye\n");
    exit(2);
}
```

**进程图：**

![img](https://img-blog.csdn.net/20180614103147953?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5neGlhZmxs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**结果：**

![img](https://img-blog.csdn.net/2018061410315043?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5neGlhZmxs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 6.竞争条件

当多个进程都企图对共享数据进行某种处理，而最后的结果有取决于进程运行的顺序时，则这发生了**竞争条件**。

如果一个进程希望等待一个子进程终止，则它必须调用一种wait函数，如果一个进程要等待其父进程终止，则可使用下列形式的循环：

```c
while(getppid() != 1)
sleep(1);
```

这种形式的循环（称为轮询）的问题是浪费了CPU时间，调用着每隔1秒就被唤醒，然后进行条件测试。

为了避免竞争条件和轮询，在多个进程之间需要有某种形式的信号发送和接收的方法。在UNIX中可以使用信号机制，也可使用各种形式的进程件通信（IPC）。

```c
/*例程：竞争条件，内核尽可能的在两个进程之间进行多次切换*/
#include"apue.h"
static void charatatictime(char *);
int main()
{
    pid_t pid;
    if((pid = fork())<0){
        err_sys("fork error");
    }
    else if(pid==0){
        charatatictime("output from child\n");
    }else{
        charatatictime("output from parent\n");
    }
    exit(0);
}
static void charatatictime(char *str)
{
    char *ptr;
    int c;
    setbuf(stout,NULL);//将标准输出设置为不带缓冲的，则每个字符输出都需调用一个write
    for(ptr=str;(c = *ptr++) != 0；)
        putc(c,stout);
}
```

**输出结果：**

![3](F:\学习专用\学习笔记\图片\3.png)

## 7.进程间通信！！

**IPC方法**

Linux环境下，进程地址空间相互独立，每个金策划概念各自有不同的用户地址空间。任何一个进程的全局变量在另一个金策划概念中都看不到，所以进程之间不能相互访问，要交换数据必须通过内核，在内核开辟一块缓冲区，进程1把数据从用户空间拷贝到**内核缓冲区**，进程2再从内核缓冲区把数据读走，内核提供的这种机制称为进程间通信（IPC）

![195](F:\学习专用\note\pic\195.png)

**常用进程间通信方式：**

- 管道（使用最简单）
- 信号（开销最小）
- 共享映射区（无血缘关系）
- 本地套接字（最稳定）

### 1）管道

![199](F:\学习专用\note\pic\199.png)

~~~cpp
#include<unistd.h>
int pipe(int pipefd[2]);//向pipefd中输出两个文件描述符（读，写文件描述符），返回值：0成功，-1失败
~~~

作用于有血缘关系的进程之间，完成数据传递。特性：

- 其本质是一个伪文件（实为内核缓冲区）---伪文件：块设备，socket文件，字符设备，管道
- 由两个文件描述符引用，一个表示读端，一个表示写端
- 规定数据从管道的写端流入管道，从读端流出

![196](F:\学习专用\note\pic\196.png)

**管道原理：**

管道实为内核使用**环形队列**机制，借助内核缓冲区（4k）实现

**局限性：**

- 数据自己读不能自己写

- 数据一旦被读走，便不在管道中存在，不可反复读取

- 由于管道采用半双工通信方 式，数据只能在一个方向上流动

- 只能在有公共祖先的进程间使用管道  

  **案例：**

  ![197](F:\学习专用\note\pic\197.png)

~~~cpp
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
int main()
{
    int pipefd[2];
    int ret;
    pid_t pid;
    ret=pipe(pipefd);
    if(ret==-1)
    {
        perror("pipe err");
        exit(1);
    }
    pid=fork();
    if(pid<0)
    {
        perror("fork err");
        exit(1);
    }
    else if(pid==0)//子进程 读管道中数据
    {
        close(pipefd[1]);//关闭子进程中的写描述符
        char buf[1024];
        ret=read(pipefd[0],buf,sizeof(buf));//返回读取到的长度
        write(1,buf,ret);//将读到的数据打印到终端
    }
    else//父进程向管道中写数据
    {
        close(pipefd[0]);//关闭父进程中的读描述符
        char buf[]="hello pipe\n";
        write(pipefd[1],buf,sizeof(buf));//向管道中写数据
        sleep(1);//等待子进程读数据和打印数据
    }
    return 0;
}
~~~

![198](F:\学习专用\note\pic\198.png)

### 2）共享内存

![200](F:\学习专用\note\pic\200.png)

~~~cpp
#include<sys/mman.h>
void *mmap(void *addr,size_t length,int prot,int plags,int fd,off_t offset);//返回值：成功：返回创建的映射区首地址，失败：MAP_FAILED宏
int munmap(void *addr,size_t len);//释放内存,成功0，失败-1
~~~

参数：

addr: 建立映射区的首地址，有Linux内核指定，使用时，直接传递NULL

length: 预创建映射区的大小（跟文件大小有关）

prot: 映射区权限PROT_READ,PROT_WRITE,PROT_READ|PROT_WRITE

flags: 标志位参数（常用于设定更新物理区域、设置共享、创建匿名映射区）

​           MAP_SHARED: 会将映射区所做的操作反映到物理设备（磁盘）上

​	  MAP_PRIVATE:映射区所做的修改不会反映到物理设备

fd: 用来建立映射区的文件描述符

offset:映射文件的偏移

![201](F:\学习专用\note\pic\201.png)

~~~cpp
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<string.h>
#include<fcntl.h>
#include<sys/mman.h>
int main()
{
    int ret;
    int fd;
    fd=open("zxtxt.txt",O_RDWR|O_CREAT,0644);//以创建文件的方式打开文件
    if(fd==-1)
    {
        perror("open err");
        exit(1);
    }
    ftruncate(fd,20);//文件扩展
    char *p=NULL;
    p=mmap(NULL,11,PROT_READ|PROT_WRITE,MAP_SHARED,fd,0);//创建共享内存映射
    if(p==MAP_FAILED)
    {
        perror("mmap err:");
        exit(1);
    }
    strcpy(p,"hello word");//将hello mmap写入共享内存区
    ret=munmap(p,11);//方式共享内存映射
    if(ret==-1)
    {
        perror("munmap err");
        exit(1);
    }
    close(fd);
    return 0;
}
~~~

![202](F:\学习专用\note\pic\202.png)



# 九.守护进程

## 1.引言

守护进程也称**精灵进程**，是生存期较长的一种进程。它们通常在系统自举时启动，仅在系统关闭时才终止。因为它们**没有控制终端**，所以它们都是在**后台运行**的。守护进程的名字一般以d结尾

## 2.守护进程的特征

![47](F:\学习专用\学习笔记\图片\47.png)

各标题意义：父进程ID，进程ID，进程组ID，会话ID,终端名称，终端进程组ID，用户ID

系统进程依赖于操作系统实现。父进程ID为0的各进程通常是内核进程，它们作为系统自举过程的一部分而启动。进程1通常是init，它是一个系统守护进程，负责启动各运行层次特定的系统服务。

- 在Linux下，kenentd守护进程为在内核中运行计划执行的函数提供进程上下文。

​		             kapmd守护进程对很多计算机系统中具有的高级电源管理提供支持。

​	                     kswapd守护进程也称为页面调出守护进程，通过将脏页面以低速写到磁盘上从而使这些页面在需要时仍可回收。

- Linux内核使用bdflush和kupdated将高速缓冲中的数据冲洗到磁盘上。
- 大多数守护进程都以超级用户（用户ID为0）特权运行。没有一个守护进程具有控制终端，其终端名设置为问号（？），终端前台进程组ID设置为-1
- 大多数守护进程的父进程是init进程

## 3.编程规则！

编写守护进程需遵循一定的规则

- 首先要做的是调用umask将文件模式创建屏蔽字设置为0
- 调用fork，然后使父进程退出（exit），这样做实现下面几点：（1）如果该守护进程是作为一条简单shell命令启动的，那么父进程终止使得shell认为这条命令已经执行完毕，（2）子进程继承了父进程的进程组ID，但具有一个新的进程ID，这就保证了子进程不是一个进程组的组长进程。
- 调用setsid以创建一个新会话，使调用进程：（1）成为新会话的首进程，（2）成为一个新进程组的组长进程，（3）没有控制终端
- 将当前工作目录更改为根目录
- 关闭不再需要的文件描述符。这使守护进程不再持有从其父进程继承来的某些文件描述符
- 某些守护进程打开/dev/null使其具有文件描述符0、1和2，这样，任何一个试图读标准输入，写标准输出或标准出错的库例程都不会产生任何效果。

**简化编写一个守护进程!!!!：**

1. 指定文件掩码 umask()

2. 创建子进程 fork

3. 子进程创建新会话 setsid()

4. 改变进程的工作目录 chdir()

5. 将0/1/2文件描述符重定向 /dev/null   dup2()

6. 守护进程主逻辑

7. 退出

   #### **案例：**

   创建一个守护进程每隔1秒将一个字符串写到文件中

   ~~~cpp
   #include<stdio.h>
   #include<stdlib.h>
   #include<unistd.h>
   #include<fcntl.h>
   #include <sys/types.h>
   #include <sys/stat.h>
   int main()
   {
       pid_t pid,sid;
       int fd;
       pid=fork();//1.
       if(pid>0)
       {
           return 0;
       }
       sid=setsid();//2
       int ret=chdir("/home/summer/");//3
       if(ret==-1)
       {
           perror("chdir err : ");
       }
       umask(0002);//4
       close(STDIN_FILENO);//5
       open("/dev/null",O_RDWR);
       dup2(0,STDOUT_FILENO);
       dup2(0,STDERR_FILENO);
       fd=open("/home/summer/zx/zx.txt",O_RDWR|O_CREAT,0644);
       while(1)//6守护进程主逻辑
   	{
           char str[]="-------zx-----\n";
   		write(fd,str,sizeof(str));
           sleep(1);
   	}
       return 0;
   }
   ~~~

   ![207](F:\学习专用\note\pic\207.png)

![208](F:\学习专用\note\pic\208.png)

## 4.出错记录

与守护进程有关的一个问题是如何处理出错信息。

## 5.单实例守护进程

为了正常运行，某些守护进程实现为单实例的，在任意时刻只运行该守护进程的一个副本。



​		