# 【UNIX环境高级编程】文件和目录

## 1.stat、fstat和lstat函数

~~~c
#include<sys/stat.h>
int stat(const char *restrict pathname,struct stat *restrict buf);
int fstat(int file,struct stat *buf);
int lstat(const char *restrict pathname,struct stat *restrict buf);//三函数返回值：成功0，出错-1
~~~

一旦给出pathname，stat就返回此命名文件有关的信息结构。fstat函数获取已在描述符file上打开文件的有关信息。lstat函数类似于stat，但当命名的文件是一个符号链接时，lstat返回该符号链接的有关信息。

buf指针指向一个数据结构，基本形式：

~~~c
struct stat{
    mode_t st_mode;//file type & mode
    ino_t  st_ino;//i-node number
    dev_t st_dev;//device number
    uid_t st_uid;//user ID of owner
    gid_t st_gid;//group ID of owner
    ...
}

~~~

使用stat函数最多的是ls -l命令，获得文件的所有信息

## 2.文件类型

- 普通文件
- 目录文件
- 块特殊文件，提供对设备（磁盘）带缓冲的访问，每次访问以固定长度为单位进行
- 字符特殊文件，提供对设备不带缓冲的访问，每次访问长度可变
- FIFO，用于进程间通信，有时也称为**管道**
- 套接字，用于进程间网络通信
- 符号链接，这种文件类型指向另一个文件。



## 3.设置用户ID和设置组ID

与一个进程相关联的ID由6个或更多

![41](F:\学习专用\学习笔记\图片\41.png)

当执行一个程序文件时，有效用户ID等于实际用户ID，有效组ID等于实际组ID

## 4.link、unlink、remove和rename函数

创建一个指向现有文件的链接的方法是使用link函数

~~~c
#include<unistd.h>
int link(const char *existingpath,const char *newpath);//成功返回0，出错-1
~~~

若newpath已经存在，则出错。

~~~c
#include<unistd.h>
int unlink(const char *pathname);//成功返回0，出错-1
~~~

此函数删除目录项

## 5.符号链接

符号链接是指向一个文件的间接指针。引入符号链接是为了避开硬链接的一些限制：

- 硬链接通常要求链接和文件位于同一文件系统中
- 只有超级用户才能创建指向目录的硬链接

