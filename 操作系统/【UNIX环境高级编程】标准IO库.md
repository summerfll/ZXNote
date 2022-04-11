#  【UNIX环境高级编程】标准I/O库

## 1.流和FILE对象

所有I/O函数都是针对文件描述符的。当打开一个文件时，即返回一个文件描述符，然后改文件描述符就用于后续的I/O操作。对于标准I/O库，他们的操作则是围绕流进行的。当用标准I/O库打开或创建一个文件时，我们已使一个流于一个文件相关联。

**流的定向**决定了所读、写的字符是单字节还是多字节的。当一个流最初被创建时，他并没有定向。如果在为定向的流上使用一个多字节I/O函数，则将改流的定向设置为**宽定向**。若在未定义的流上使用一个单字节I/O函数，则将改流的定向设置为字节定向。

通过freopen可清楚一个流的定向，fwide函数设置流的定向。

~~~c
#include<stdio.h>
#include<wchar.h>
int fwide(FILE *fp,int mode);//返回值：流是宽定向返回正值，字节定向返回负值，为定向返回0
~~~

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

~~~c
#include<stdio.h>
void setbuf(FILE *restrict fp,char *restrict buf);
int setvbuf(FILE *restrict fp,char *restrict buf,int mode,size_t size);//返回值：成功0，出错-1
~~~

这些函数一定要在流已被打开后调用，而且也应该在对流执行任何一个其他操作之前调用。

可以使用setbuf函数打开或关闭缓冲机制。此时，将buf设置为NULL

使用setvbuf，可以指定所需的缓冲类型。

mode参数：

_IOFBF  全缓冲

_IOLBF  行缓冲

_IONBF  不带缓冲

若指定一个不带缓冲的流，则忽略buf和size。如果指定全缓冲或行缓冲，则buf和size额选择的指定一个缓冲区及其长度。若果该流是带缓冲的，而buf为NULL，则标准I/O库将自动为该流分配适当长度的缓冲区。

**强制冲洗一个流：**

~~~c
#include<>
~~~

此函数使该流所有未写的数据都被传送至内核。

## 4.打开流

三个函数打开标准I/O流

~~~c
#include<stdio.h>
FILE *fopen(const char *restrict pathname,const char *restrict type);
FILE *freopen(const char *restrict pathname,const char *restrict type,FILE *restrict fp);
FILE *fdopen(int file,const char *type);//三函数返回值：成功返回文件指针，出错返回NULL
~~~

区别：

- fopen打开一个指定的文件
- freopen在指定的流上打开一个指定的文件，如若该流已经打开，则先关闭该流。此函数一般用于将一个指定的文件打开为一个预定义的流：标准输入、标准输出或标准出错
- fdopen获取一个现有的文件描述符（可从open、dup、、、函数获取此文件描述符），并使一个标准的I/O流与该描述符相结合。此函数通常用于创建管道和网络通信通道函数返回的描述符。

type参数指定对该I/O流的读、写方式，如图

![42](F:\学习专用\学习笔记\图片\42.png)

用fclose关闭一个打开的流：

~~~c
#include<stdio.h>
int fclose(FILE *fp);
~~~

在该文件被关闭之前，冲洗缓冲区中的输出数据，丢弃缓冲区中的任何输入数据，如果标准I/O库已经为该流自动分配了一个缓冲区，则释放此缓冲区。

当一个进程正常终止时（直接调用exit函数，或从main返回），则所有带未写缓冲数据的标准I/O流都会被冲洗，所有打开的标准I/O流都会被关闭

## 5.读和写流

一旦打开了流，可在三种不同类型的非格式化I/O中进行选择，对其进行读、写操作：

（1）每次一个字符的I/O，一次读或写一个字符，如果流是带缓冲的，则标准I/O函数回处理所有缓冲。

（2）每次一行I/O。如想要一个读或写一行，则使用fgets和fputs。每行都以一个换行符终止。

（3）直接I/O。fread和fwrite函数支持这种类型的I/O。每次I/O操作读或写某种数量的对象，而每个对象具有指定的长度。

### 5.1输入函数

一次读一个字符的三个函数：

~~~c
#include<stdio.h>
int getc(FILE *fp);
int fgetc(FILE *fp);
int getchar(void);//三函数返回值：成功返回下一个字符，已到达文件结尾或出错返回EOF
~~~

函数getchar等价于getc(stdin)。

### 5.2输出函数

对应上面每个输入函数都有一个输出函数

~~~~~c
#include<stdio.h>
int putc(int c,FILE *fp);
int fputc(int c,FILE *fp);
int putchar(int c);//三函数返回值：成功返回c，出错返回EOF
~~~~~

函数putshar（c）等效于putc（c,stdout）

## 6.每次一行I/O

下面两个函数提供每次输入一行的功能

~~~c
#include<stdio.h>
char *fgets(char *restrict buf,int n,FILE *restrict fp);
char *gets(char *buf);//返回值：成功返回buf，若到达文件结尾或出错返回NULL
~~~

两个函数都指定了缓冲区的地址，读入的行将送人其中，gets从标准输入读，而fgets则从指定的流读。

对于fgets，必须指定缓冲区的长度n。此函数一直读到下一个换行符为止。

gets是一个不建议使用的函数，其问题是调用者在使用gets时不能指定缓冲区的长度。这样可能造成缓冲区溢出，写到缓冲区之后的存储空间中，产生不可预料的后果。gets与fgets的另一个区别是：gets不讲换行符存入缓冲区。

fputs和puts提供每次输出一行的功能。

~~~c
#include<stdio.h>
int fputs(const char *restrict str,FILE *restrict fp);
int puts(const char *str);//返回值：成功返回非负值，出错返回EOF
~~~

函数fputs将一个以null符终止的字符串写到指定的流，尾端的终止符null不写出。

puts将一个以null符终止的字符串写到标准输出，终止符不写出

## 7.标准I/O的效率

例程：用getc和putc将标准输入复制到标准输出

~~~c
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
~~~

用fgets和fputs将标准输入复制到标准输出

~~~c
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
~~~

## 8.二进制I/O

进行二进制I/O操作，一次读或写整个结构。下列两个函数可以执行二进制I/O操作

~~~c
#include<stdio.h>
size_t fread(void *restrict ptr,size_t size,size_t nobj,FILE *restrict fp);
size_t fwrite(const void *restrict ptr,size_t size,size_t nobj,FILE *restrict fp);//返回值：读或写的对象数
~~~

两函数常见用法：

（1）读或写一个二进制数组，例：将一个浮点数组的第2-5个元素写至一个文件上

~~~c
float data[10];
if(fwrite(&data[2],sizeof(float),4,fp) != 4)
    err_sys("fwrite error");
~~~

size参数为每个数组元素的长度，nobj参数为欲写的元素数

（2）读或写一个结构，例

~~~c
struct{
    short c;
    long a;
    char name[LINE];
}item;
if(fwrite(&item,sizeof(item),1,fp) != 1)
    err_sys("fwrite error");
~~~

size指定结构的长度，nobj为要写的对象数

## 9.定位流

三种方法定位标准I/O流

（1）ftell和fseek函数。他们都假定文件的位置可以存放在一个长整型中

（2）ftell和fseeko函数。可以使文件偏移量不必一定使用长整型。

（3）fgetpos和fsetpos函数，他们使用一个抽象数据类型fpos_t记录文件的位置。

需要移植到非UNIX系统上运行的应用程序应当使用fgetpos和fsetpos。

~~~c
#include<stdio.h>
long ftell(FILE *fp);//返回值：成功返回当前文件位置指示，出错-1
int fseek(FILE *fp,long offset,int whence);//返回值：成功0，出错-1
void rewind(FILE *fp);
~~~

fgetpos和fsetpos这两个函数是c函数引进的

~~~c
#include<stdio.h>
int fgetpos(FILE *restrict fp,fpos_t *restrict pos);
int fsetpos(FILE *fp,const fpos_t *pos);//两函数返回值：成功返回0，出错非0
~~~



fgetpos将文件位置指示器的当前值存入由pos指向的对象中。在以后调用fsetpos时，可以使用此值将流重新定位至该位置。

## 10.格式化I/O

### 10.1格式化输出

执行格式化输出处理的4个printf函数

~~~c
#include<stdio.h>
int printf(const char *restrict format,...);
int fprintf(FILE *restrict fp,const char *restrict format,...);//两个函数返回值：成功返回输出字符数，出错返回负值
int sprintf(char *restrict buf,const char *restrict format,...);
int snprintf(char *restrict buf,size_t n,const char *resrtict format,...);//返回值：成功返回存入数组的字符数，出错返回负值
~~~

printf将格式化数据写到标准输出，fprintf写至指定的流，sprintf将格式化的字符送入数组buf中，snprintf在该数组的尾端自动加一个null字节，但该字节不包括在返回值中。

### 10.2 格式化输入

执行格式化输入处理的是三个scanf函数

~~~c
#include<stdio.h>
int scanf(const char *restrict format,...);
int fscanf(FILE *restrict fp,const char *restrict format,...);
int sscanf(const char *restrict buf,const char *restrict format,...);//三函数返回值：指定的输入项数，输入出错或在任意变换前已到达文件结尾则返回EOF
~~~

scanf族用于分析输入字符串，并将字符序列转换成指定类型的变量。

## 11.实现细节

每个标准I/O流都有一个与其相关联的文件描述符，可以对一个流调用fileno函数以获得其文件描述符

~~~c
#include<stdio.h>
int fileno(FILE *fp);//返回值：与该流相关联的文件描述符
~~~

## 12.临时文件

ISO C标准I/O库提供了两个函数以帮助创建临时文件。

~~~c
#include<stdio.h>
char *tmpnam(char *ptr);//返回值：指向唯一路径名的指针
FILE *tmpfile(void);//返回值：成功返回文件指针，出错返回NULL
~~~

tmpnam函数产生一个与现有文件名不同的一个有效路径名字符串。每次调用它时，都产生一个不同的路径名，最多调用TMP_MAX，至少为25

若ptr是NULL，则所产生的路径名存放在一个静态区中，指向该静态区的指针作为函数值返回。

tmpfile创建一个临时二进制文件，在关闭该文件或程序结束时将自动删除这种文件。



