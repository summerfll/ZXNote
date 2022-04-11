

# 【UNIX环境高级编程】文件I/O

大多数文件I/O只需要5个函数：

**open、read、write、lseek以及close**

**不带缓冲的I/O：**

每个**read和write**都调用内核中的一个系统调用

## 1.文件描述符

对于内核而言，所有打开的文件都通过文件描述符引用。当打开一个文件时，内核向进程返回一个文件描述符。当读或写一个文件时，使用open或create返回的文件描述符标识该文件，将其作为参数传给read或write。

UNIX系统shell使用**文件描述符0**与进程的标准**输入**相关联，**1**与标准**输出**相关联，**2**与标准**出错输出**相关联。

## 2.open函数

调用open函数打开或创建一个文件

~~~c
#include<fcntl.h>
int open(const char *pathname,int oflag,.../*mode_t mode*/);//返回值：成功返回文件描述符，出错返回-1
~~~

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

~~~c
#include<fcntl.h>
int creat(const char *pathname,mode_t mode);//返回值：成功返回为只写打开的文件描述符，出错返回-1
~~~

此函数等效于：open(pathname,O_WRONLY|O_CREAT|O_TRUNC,mode);

## 4.close函数

调用close函数关闭一个打开的文件

~~~c
#include<unistd.h>
int close(int file);//成功返回0，出错-1
~~~

关闭一个文件时还会释放该进程加在该文件上的所有记录锁。

当一个进程终止时，内核自动关闭它所有打开的文件。很多程序都利用这一功能不显式的关闭打开文件。

## 5.lseek函数

### **偏移量**

每个打开的文件都有一个与其相关联的“当前文件偏移量”。它通常是一个非负整数，用以度量从文件开始处计算的字节数。通常，读、写操作都从当前文件偏移量处开始，并使偏移量增加所读写的字节数。系统默认下，打开文件，未指定O_APPEND选项时，该偏移量为0

调用lseek函数能显式地为一个打开地文件设置其**偏移量**

~~~c
#include<unistd.h>
off_t lseek(int file,off_t offset,int whence);//成功返回新的文件偏移量，出错-1
~~~

对参数offset地解释与参数whence有关

- 若whence是SEEK_SET，则将该文件地偏移量设置为距文件开始处offset个字节
- 若whence是SEEK_CUR，则将该文件地偏移量设置为其当前值加offset，offset可正可负
- 若whence是SEEK_END，则将该文件地偏移量设置为文件长度加offset，offset可正可负

例：

~~~c
off_t currpo;
currpo = lseek(fd,0,SEEK_CUR);//打开文件地当前偏移量
~~~

## 6.read函数

调用read函数从打开文件中读数据

~~~c
#include<unistd.h>
int read(int file,char *buf,unsigned nbytes);//成功返回读到地字节数，若已到文件结尾返回0，出错-1
~~~

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
ssize_t write(int file,const void *buf,size_t nbytes);//成功返回已写字节数，出错返回-1
```

write出错的常见原因：磁盘已写满，或超过一个给定进程的文件长度限制。

普通文件，写操作从文件的当前偏移量处开始，如果在打开文件时，指定了O_APPEND选项，则每次写操作之前，将文件偏移量设置在文件的当前结尾处，成功写之后，该文件偏移量增加实际写的字节数。

## 8.I/O的效率

例程：标准输入复制到标准输出

~~~c
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
~~~

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

~~~c
#include<unistd.h>
ssize_t praed(int file,void *buf,size_t nbytes,off_t offset);//返回值：读到的字节数，若已到文件结尾则返回0，出错-1
ssize_t pwrite(int file,const void *buf,size_t nbytes,off_t offset;//返回值：成功返回已写的字节数，出错-1
~~~

调用pread相当于顺序调用lseek和read，但也有区别：

- 调用pread，无法中断其定位和读操作
- 不更新文件指针

调用pwrite相当于顺序调用lseek和write，不过也有相同的区别

### 10.3 创建一个文件

同时选择open函数地O_CREAT和O_EXCL选项，该文件已经存在则open失败。检查该文件是否存在以及创建该文件这两个操作作为一个一个原子操作执行。

## 11.dup和dup2函数

这两个函数用于复制一个现存地文件描述符

~~~c
#include<unistd.h>
int dup(int file);
int dup2(int file,int file2);//两个函数返回值：成功返回新的文件描述符，出错-1
~~~

dup返回的新文件描述符是当前可用文件描述符中的最小数值。dup2可用file2参数指定新的文件描述符数值。若果file2已经打开，则先将其关闭，若file等于file2，则dup2返回file2，而不关闭它。

这些函数返回的新文件描述符于参数file共享同一个文件表项。

假定进程执行：

newfd=dup（1）；

![38](F:\学习专用\学习笔记\图片\38.png)

当此函数开始执行，新的文件描述符很有可能为3（0，1，2已经被shell打开），两个文件描述符指向同一文件表项，共享同一文件状态标志(读、写等)和同一当前文件偏移量

复制一个描述符的另一种方法是使用ftcntl函数

调用dup(file)等效于fcntl（file,F_DUPFD,0）

调用dup2(file,file2)等效于close(file2);fcntl(file,F_DUPFD,file2);

## 12.fcntl函数

fcntl函数可以改变已打开的文件的性质

~~~c
#include<fcntl.h>
int fcntl(int file,int cmd,.../*int arg*/);//返回值：成功则依赖cmd，出错-1
~~~

**fcntl函数5种功能：**

- 复制一个现有的描述符（cmd=F_DUPFD）
- 获得/设置文件描述符标记（cmd=F_GETFD或F_SETFD）
- 获得/设置文件状态标志（cmd=F_GETFL或F_SETFL）
- 获得/设置异步I/O所有权（cmd=F_GETOWN或F_SETOWN）
- 获得/设置记录锁（cmd=F_GETLK、F_SETLK或F_SETLKW）

















































































