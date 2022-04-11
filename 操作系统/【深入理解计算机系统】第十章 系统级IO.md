# 【深入理解计算机系统】第十章 系统级I/O

## 1.UNIX I/O

所有的I/O设备（如磁盘，网络和终端）都被模型化为文件，而所有的输入和输出都被当作相应文件的读和写来执行。这种将设备映射为文件的方式，允许Linux内核引出一个简单、低级的应用接口，称为Unix I/O，这使得所有的输入和输出都能以一种统一且一致的方式来执行：

- **打开文件**，内核返回一个非负整数的文件描述符，通过对此文件描述符对文件进行所有操作。
- **Linux shell创建的每个进程开始时都有三个打开的文件**：标准输入（文件描述符0）、标准输出（1），标准出错（2）。头文件<unistd.h>定义了常量STDIN_FILENO、STDOUT_FILENO、STDERR_FILENO，他们可用来代替显式的描述符值
- **改变当前的文件位置**，文件开始位置为文件偏移量，应用程序通过seek操作，可设置文件的当前位置为k。
- **读写文件**，读操作：从文件复制n个字节到内存，从当前文件位置k开始，然后将k增加到k+n；写操作：从内存复制n个字节到文件，当前文件位置为k，然后更新k
- **关闭文件**。当应用完成对文件的访问后，通知内核关闭这个文件。内核会释放文件打开时创建的数据结构，将描述符恢复到描述符池中。

## 2 文件

每个Linux文件都有一个类型来表明它在系统中的角色：

- **普通文件**包含任意数据，应用程序常区分**文本文件**和**二进制文件**，文本文件是只含ASCII或Unicode字符的普通文件，二进制文件是所有其他的文件。对内核而言，文本文件和二进制文件没区别
- **目录**是包含一组链接的文件。其中每个链接都将一个文件名映射到一个文件，这个文件也可能是一个另一个目录。每个目录至少含有两个条目：“.”是到该目录自身的链接，“..”是到父目录的链接
- **套接字**是用来与另一个进程进行跨网络通信的文件。

**Linux系统目录层次结构的一部分：**

![48](F:\学习专用\学习笔记\图片\48.png)

### **路径名：**

- **绝对路径名**以一个斜杠开始，表示从根节点开始的路径。如上图所示的hello.c的绝对路径为/home/droh/hello.c
- **相对路径名**以文件名开始，表示从当前工作目录开始的路径。如上图，若/home/drop是当前工作目录，则hello.c的相对路径为./hello.c。如果/home/bryant是当前工作目录，那么相对路径名为../home/droh/hello.c

## 3.打开和关闭文件

### 打开文件

进程通过调用open函数打开一个已存在的文件或创建一个新文件：

~~~c
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>
int open(char *filename,int flags,mode_t mode);//返回值：成功返回文件描述符，出错-1
~~~

open函数将filename转换为一个文件描述符，并返回描述符数字。返回的描述符总是在进程中当前没有打开的**最小描述符**。

**flags参数**：

- O_RDONLY：只读
- O_WRONLY：只写
- O_RDWR：可读可写

例：fd = open("foo.txt",O_RDONLY，0);//以读的方式打开一个已存在的文件

**flags参数**也可为一个或更多位掩码的或，为写提供一些额外的指示：

- O_CREAT：若文件不存在，就创建它的一个截断的文件

- O_TRUNC：若文件已经存在，就截断它
- O_APPEND：在每次写操作前，设置文件位置到文件的结尾处

例：fd = open("foo.txt",O_WRONLY|O_APPEND,0);//打开一个文件并在末尾添加数据

**mode参数**指定新文件的访问权限位

作为上下文的一部分，每个进程都有一个umask，它是通过调用umsk函数设置的。

文件的访问权限设置为<u>**mode & ~umask**</u>

**例：**创建一个新文件，文件的拥有着有读写权限，所有其他用户有读权限

~~~c
#define DEF_MODE S_IRUSR|S_IWUSR|S_IRGRP|S_IWGRP|S_IROTH|S_IWOTH
#define DEF_UMASK S_IWGRP|S_IWOTH
umask(DEF_UMASK);
fd = open("foo.txt",O_CREAT|O_TRUNC|O_WRONLY,DEF_MODE);
~~~

**访问权限位设置**

![49](F:\学习专用\学习笔记\图片\49.png)

### 关闭文件

进程通过调用close函数关闭一个打开的文件

~~~c
#include<unistd.h>
int close(int fd);//返回值：成功返回0，出错-1
~~~

**注意：**关闭一个已关闭的描述符会出错

## 4.读和写文件

通过read和write函数执行输入和输出

~~~c
#include<unistd.h>
ssize_t read(int fd,void *buf,size_t n);//返回值：成功返回读的字节数，到达文件末尾返回0，出错返回-1
ssize_t write(int fd,const void *buf,size_t n);//返回值：成功返回写的字节数，出错返回-1
~~~

**注意：**size_t被定义为unsigned long,ssize_t 被定义为long

read函数从描述符为fd的当前文件位置复制最多n个字节到内存位置buf。返回值为-1表示有一个错误，返回0表示EOF（end-of-file），否则返回值表示实际传送的字节数

write函数从内存位置buf复制最多n个字节到描述符fd的当前文件位置。

例：使用read和write一次一个字节的从标准输入复制到标准输出

~~~c
#include"csapp.h"
int main(void)
{
    char c;
    while(read(STDIN_FILENO,&C,1)!=0)
        write(STDOUT_FILENO,&C,1);
    exit(0);
}
~~~

调用lseek函数，应用程序能显示地修改当前文件的位置。

**read和write传送的字节数少于要求的字节数情况：**

- **读时遇到EOF**（end-of-file），如该文件从当前位置开始只有20个字节，而我们要读50个字节
- **从终端读文本行**，如果打开文件是与终端相关联的（如键盘和显示器），那么每个read函数将一次传送一个文本行，返回的不足值等于文本行的大小
- **读和写网络套接字（socket）**。如果打开的文件对应于网络套接字，那么内部缓冲约束和较长的网络延迟会引起read和write返回值不足值，对Linux管道（pipe）调用read和write时，也可能出现不足值。

## 5.用RIO包健壮地读写

RIO（健壮地I/O）包，它会自动为你处理上文中所述的不足值。

RIO提供两类不同函数：

- **无缓冲的输入输出函数。**这些函数直接在内存和文件之间传送数据，没有应用级缓冲。它们对将二进制数据读写到**网络**和从网络中读写二进制数据较有用。
- **带缓冲的输入函数。**这些函数允许你高效地从文件中读取文本行和二进制数据这些文件的内容缓存在应用级缓冲区内，类似于为printf这样的标准I/O函数提供的缓冲区。

### RIO的无缓冲的输入输出函数

通过调用rio_readn和rio_writen函数，应用程序可以在内存和文件间直接传送数据

~~~c
#include<csapp.h>
ssize_t rio_readn(int fd,void *usrbuf,size_t n);//返回值：成功返回读的字节数，到达文件末尾返回0，出错返回-1
ssize_t rio_writen(int fd,void *usrbuf,size_t n);//返回值：成功返回写的字节数，出错返回-1
~~~

rio_readn函数从描述符为fd的当前文件位置复制最多n个字节到内存位置usrbuf。返回值为-1表示有一个错误，遇到EOF（end-of-file）只能返回一个不足值0,否则返回值表示实际传送的字节数

rio_writen函数从内存位置usrbuf复制n个字节到描述符fd的当前文件位置，绝不会返回不足值

### RIO的带缓冲的输入函数

假设我们要编写一个程序来计算文本文件中文本行的数量，该如何实现？

一种好的方法是调用一个包装函数rio_readlineb,它从一个内部读缓冲区复制一个文本行，当缓冲区变空时，会自动调用read重新填满缓冲区。对于既包含文本行也包含二进制数据的文件，提供一个rio_readn带缓冲区的版本rio_readnb，与rio_readlineb一样的读缓冲区中传送原始字节。

~~~c
#include"csapp.h"
void rio_readinitb(rio_t *rp,int fd);
ssize_t rio_readlineb(rio_t *rp,void *usrbuf,size_t maxlen);//返回值：成功返回读的字节数，到达文件末尾返回0，出错返回-1
ssize_t rio_readnb(rio_t *rp,void usrbuf,size_t n);//返回值：成功返回读的字节数，到达文件末尾返回0，出错返回-1
~~~

rio_readlineb函数从文件rp读出下一个文本行（包括结尾的换行符），将它复制到内存位置usrbuf，并用NULL字符来结束这个文本行。rio_readlineb函数最多读maxlen-1个字节，超过maxlen-1字节的文本行被截断，并用一个NULL字符结束。

rio_readnb函数从文件rp最多读n个字节到内存位置usrbuf

## 6.读取文件元数据

应用程序通过调用stat和fstat函数，检索到关于文件的信息（有时也称为文件的元数据）

~~~c
#include<unistd.h>
#include<sys/stat.h>
int stat(const char *filename,struct stat *buf);
int fstat(int fd,struct stat *buf);//返回值：成功返回0，出错-1
~~~

## 7.读取目录内容

**读取目录**

应用程序用readdir系列函数来读取目录的内容

~~~c
#include<sys/types.h>
#include<dirent.h>
DIR *opendir(const char *name);//返回值：若成功，则为处理的指针，若出错，则为NULL
~~~

函数opendir以路径名为参数，返回指向目录流的指针。

~~~c
#include<dirent.h>
struct dirent *readdir(DIR *dirp);//返回：若成功，则为指向下一个目录项的指针；若没有更多的目录项或出错，则为NULL
~~~

每次对readdir调用返回的都是指向流dirp中下一个目录项的指针。每个目录项结构如下：

~~~c
struct dirent{
    ino_t d_ino;//文件位置
    char d_name[256];//文件名
}
~~~

若出错，readdir返回NULL，并设置errno，检查自调用readdir以来errno是否被修改过可区分错误和流结束

**关闭目录**

~~~c
#include<dirent.h>
int closedir(DIR *dirp);//返回：成功0，错误-1
~~~

函数closedir关闭流并释放其所有资源

**例：读取目录内容**

~~~c
#include"csapp.h"
int main(int argc,char **argv)
{
    DIR *streamp;
    struct dirent *dep;
    streamp=opendir(argv[1]);
    errno=0;
    while((dep=readdir(streamp))!=NULL)//读取目录
    {
        printf("find file:%s\n",dep->d_name);//打印文件名
    }
    if(errno!=0)//检查errno是否被修改，区分错误和流结束
        unix_error("readdir error");
    closedir(streamp);//关闭流
    exit(0);   
}
~~~

## 8.共享目录

可以用许多不同的方式来共享Linux文件。内核用三个相关的数据结构来表示打开的文件：

- **描述符表**。每个进程都有它独立的描述符表，它的表项是由进程打开的文件描述符来索引的。每个打开的描述符表项指向文件表中的一个表项。
- **文件表**。打开文件的集合是由一张文件表来表示的，所有进程共享这张表。每个文件表的表项组成包括当前的文件位置、引用计数，以及一个指向v-node表中对应表项的指针。
- **v-node表**，同文件表一样，所有的进程共享这张v-node表。每个表项包含stat结构中的大多数信息，包括st_mode和st_size

**没有共享文件**

![50](F:\学习专用\学习笔记\图片\50.png)

描述符1和4通过不同的文件表项来引用两个不同的文件

**共享文件**

![51](F:\学习专用\学习笔记\图片\51.png)

多个描述符通过不同的文件表项来引用同一个文件

## 9.I/O重定向

dup2函数

~~~~c
#include<unistd.h>
int dup2(int oldfd,int newfd);//返回：成功返回非负描述符，出错-1
~~~~

dup2函数复制描述符表项oldfd到描述符表表项newfd，覆盖描述符表项oldfd以前的内容。如果newfd已经打开，dup2会先关闭newfd再复制oldfd

![52](F:\学习专用\学习笔记\图片\52.png)

**例：**

假设磁盘文件foo.txt由“foobar”组成，求程序输出？

~~~c
#include"csapp.h"
int main()
{
    int fd1,fd2;
    char c;
    fd1 = open("foo.txt",O_RDONLY,0);
    fd2 = open("foo.txt",O_RDONLY,0);
    read(fd2,&c,1);
    dup2(fd2,fd1);
    read(fd1,&c,1);
    printf("c=%c\n",c);
    exit(0);
}
~~~

结果：c=o

## 10.综合：该使用哪些I/O函数

#### **各种I/O包**

![53](F:\学习专用\学习笔记\图片\53.png)

- UNIX I/O模型是在操作系统内核中实现的。应用程序可通过open,close,lseek,read,write和stat等函数访问Unix I/O。
- 较高级别的RIO和标准I/O函数是基于Unix I/O函数实现的。
- RIO函数专门为read和write开发的健壮地包装函数，他们自动处理不足值。
- 标准I/O函数提供Unix I/O函数的一个更加完整的带缓冲的替代品，包括格式化的I/O例程，如printf和scanf

#### **使用I/O函数准则：**

- 只要有可能就使用标准I/O。
- 不要使用scanf或rio_readlineb来读取二进制文件。scanf或rio_readlineb是专门设计来读取文本文件的。用于读取二进制文件易发生不可测的失败。
- 对网络套接字的I/O使用RIO函数

