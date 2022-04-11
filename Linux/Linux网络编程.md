# Linux网络编程

## 1.socket

### 1.1基础知识

#### 网络字节序

TCP/IP协议规定，据流应采用大端字节序，即低地址高字节。

转换函数

~~~c
#include <arpa/inet.h>
uint32_t htonl(uint32_t hostlong);      uint32_t ntohl(uint32_t netlong);
uint16_t htons(uint16_t hostshort);     uint16_t ntohs(uint16_t netshort);
~~~

h表示host，n表示network，l表示32位长整数，s表示16位短整数。

如果主机是小端字节序，这些函数将参数做相应的大小端转换然后返回，如果主机是大端字节序，这些函数不做转换，将参数原封不动地返回。

#### IP地址转换函数

~~~c
#include <arpa/inet.h>
	int inet_pton(int af, const char *src, void *dst);
	const char * inet_ntop(int af, const void *src, char *dst, socklen_t size);
~~~

支持IPv4和IPv6

可重入函数

其中inet_pton和inet_ntop不仅可以转换IPv4的in_addr，还可以转换IPv6的in6_addr。

因此函数接口是void * addrptr。

#### sockaddr数据结构

```c
struct sockaddr {
	sa_family_t sa_family; 		/* address family, AF_xxx /
	char sa_data[14];			     / 14 bytes of protocol address */
};
```
~~~c
struct sockaddr_in {
	sa_family_t sin_family; 			/* Address family */  	地址结构类型
	in_port_t sin_port;					 		/* Port number */		端口号
	struct in_addr sin_addr;					/* Internet address */	IP地址
};
struct in_addr{
    uint32_t s_addr;
} 
~~~

~~~C
//IPv6
struct sockaddr_in6 { 
    sa_family_t     sin6_family;   /* AF_INET6 */ 
    in_port_t       sin6_port;     /* port number */ 
    uint32_t        sin6_flowinfo; /* IPv6 flow information */ 
    struct in6_addr sin6_addr;     /* IPv6 address */ 
    uint32_t        sin6_scope_id; /* Scope ID (new in 2.4) */ 
};

struct in6_addr { 
    unsigned char   s6_addr[16];   /* IPv6 address */ 
};
~~~

### 1.2网络套接字函数

#### socket模型创建流程（TCP）

![223](F:\学习专用\note\pic\223.png)

#### 1.2.1socket函数

~~~c
#include <sys/types.h> /* See NOTES */
#include <sys/socket.h>
int socket(int domain, int type, int protocol);//成功：返回套接字描述符，失败：-1，并设置errno
~~~

domain:

​         AF_INET 这是大多数用来产生socket的协议，使用TCP或UDP来传输，用**IPv4的地址**

​         AF_INET6 与上面类似，不过是来用**IPv6的地址**

​         AF_UNIX **本地协议**，使用在Unix和Linux系统上，一般都是当客户端和服务器在同一台及其上的时候使用

type:

​         **SOCK_STREAM** 这个协议是按照顺序的、可靠的、数据完整的基于字节流的连接。这是一个使用最多的socket类型，这个socket是使用**TCP**来进行传输。

​         **SOCK_DGRAM** 这个协议是无连接的、固定长度的传输调用。该协议是不可靠的，使用**UDP**来进行它的连接。

​         SOCK_SEQPACKET该协议是双线路的、可靠的连接，发送固定长度的数据包进行传输。必须把这个包完整的接受才能进行读取。

​         SOCK_RAW socket类型提供单一的网络访问，这个socket类型使用ICMP公共协议。（ping、traceroute使用该协议）

​         SOCK_RDM 这个类型是很少使用的，在大部分的操作系统上没有实现，它是提供给数据链路层使用，不保证数据包的顺序

protocol:

​         传0 表示使用默认协议。

#### 1.2.2bind函数

~~~c
#include <sys/types.h>    /* See NOTES */
#include <sys/socket.h>
int bind(int sockfd, const struct sockaddr * addr, socklen_t addrlen);//成功返回0，失败返回-1, 设置errno
~~~

sockfd：
​	socket文件描述符
addr:
​	构造出的IP地址加端口号
addrlen:
​	sizeof(addr)长度

bind()的作用是将参数sockfd和addr绑定在一起**，**使sockfd这个用于网络通讯的文件描述符监听addr所描述的地址和端口号。

#### 1.2.3listen函数

同时发起的连接数

~~~c
#include <sys/types.h> /* See NOTES */
#include <sys/socket.h>
int listen(int sockfd, int backlog);//成功返回0，失败返回-1
~~~

sockfd:
​	socket文件描述符
backlog:
​	排队建立3次握手队列和刚刚建立3次握手队列的链接数和

典型的**服务器程序可以同时服务于多个客户端**，当有客户端发起连接时，服务器调用的accept()返回并接受这个连接，如果有大量的客户端发起连接而服务器来不及处理，尚未accept的客户端就处于连接等待状态，**listen()声明sockfd处于监听状态，并且最多允许有backlog个客户端处于连接待状态**，如果接收到更多的连接请求就忽略。

#### 1.2.4accept函数

~~~c
#include <sys/types.h> 		/* See NOTES */
#include <sys/socket.h>
int accept(int sockfd, struct sockaddr * addr, socklen_t * addrlen);//成功返回一个新的socket文件描述符，用于和客户端通信，失败返回-1，设置errno
~~~

sockdf:

​       socket文件描述符

addr:

​       **传出参数**，返回**链接**客户端地址信息，**含IP地址和端口号**

addrlen:

​       传入传出参数（值-结果）,传入sizeof(addr)大小，函数返回时返回真正接收到地址结构体的大小

三方握手完成后，服务器调用**accept()接受连接**，如果服务器调用accept()时还没有客户端的连接请求，就阻塞等待直到有客户端连接上来。

#### 1.2.5connect函数

 ~~~c
#include <sys/types.h> 					/* See NOTES */
#include <sys/socket.h>
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);//成功返回0，失败返回-1，设置errno
 ~~~

sockdf:

​         socket文件描述符

addr:

​         传入参数，指定服务器端地址信息，含IP地址和端口号

addrlen:

​         传入参数,传入sizeof(addr)大小

###   1.3TCP模型

**server.c**

1. socket() 创建套接字描述符
2. bind() 绑定服务器IP和端口号
3. listen() 设置客户端同时连接的最大值
4. accept() 阻塞等待客户端连接
5. read() 读取客户端发送的数据
6. write() 将数据发送给客户端
7. close() 关闭连接

**client.c**

1. socket() 创建套接字描述符
2. bind() 可以依赖“隐式绑定”
3. connect() 连接服务器IP和端口号
4. write() 给服务器发送数据
5. read() 读取服务器发回的数据
6. close() 关闭连接

#### 例程

客户端发送小写字母给服务器，服务器收到小写字母后转换为大写字母，发回给客户端

server.c

~~~c
#include<unistd.h>
#include<stdlib.h>
#include<stdio.h>
#include <arpa/inet.h>
#include<string.h>
#include <sys/types.h> 					/* See NOTES */
#include <sys/socket.h>
#include<ctype.h>

#define SERV_PORT 7777
int main()
{
    int lfd,cfd;
    char buf[BUFSIZ],client_ip[32];
    lfd=socket(AF_INET,SOCK_STREAM,0);//创建套接字描述符
    if(lfd==-1)
    {
        perror("socket err");
        exit(1);
    }
    struct sockaddr_in serv;//初始化结构体
    bzero(&serv,sizeof(serv));
    serv.sin_family=AF_INET;
    serv.sin_port=htons(SERV_PORT);
    serv.sin_addr.s_addr=htonl(INADDR_ANY);//自动获取当前网卡上的有效ip
    
    int ret=bind(lfd,(struct sockaddr *)&serv,sizeof(serv));//将套接字描述符与自己的ip和端口绑定
    if(ret==-1)
    {
        perror("bind err");
        exit(1);
    }
    
    int err=listen(lfd,128);
    if(err==-1)
    {
        perror("listen err");
        exit(1);
    }
    
    struct sockaddr_in client_addr;
    int client_len=sizeof(client_addr);
    cfd=accept(lfd,(struct sockaddr *)&client_addr,&client_len);//阻塞等待客户端连接
    if(cfd==-1)
    {
        perror("accept err");
        exit(1);
    }
    printf("clinet ip:%s client port:%d\n",inet_ntop(AF_INET,&client_addr.sin_addr.s_addr,client_ip,sizeof(client_ip)),ntohs(client_addr.sin_port));//打印已连接的客户端
    while(1)
    {
        int n=read(cfd,buf,sizeof(buf));
        printf("recv data:%s",buf);
        for(int i=0;i<n;i++)
            buf[i]=toupper(buf[i]);
        write(cfd,buf,n);
    }
    close(lfd);
    close(cfd);
    return 0;
}
~~~

client.c

~~~c
#include<unistd.h>
#include<stdlib.h>
#include<stdio.h>
#include <arpa/inet.h>
#include <sys/types.h> 					/* See NOTES */
#include <sys/socket.h>
#include<string.h>

#define SERV_ADDR "127.0.0.1"
#define SERV_PORT 7777
int main(int argc,char *argv[])
{
	int cfd;
     char buf[BUFSIZ];
	cfd=socket(AF_INET,SOCK_STREAM,0);
    if(cfd==-1)
    {
        perror("socket err");
        exit(1);
    }
    struct sockaddr_in serv;
    bzero(&serv,sizeof(serv));//将结构体清零
    serv.sin_family=AF_INET;
    serv.sin_port=htons(SERV_PORT);
    inet_pton(AF_INET,SERV_ADDR,&serv.sin_addr.s_addr);
    
	int ret=connect(cfd,(struct sockaddr *)&serv,sizeof(serv));//与服务器建立连接
    if(ret==-1)
    {
        perror("connect err");
        exit(1);
    }
    while(1)
    {
        fgets(buf,sizeof(buf),stdin);//从标准输入中读取字符,hello\n---fgets()--->"hello\n\0"
        write(cfd,buf,strlen(buf));//将数据发送给服务器
        int len=read(cfd,buf,sizeof(buf));//接收服务器发回的消息
        write(STDOUT_FILENO,buf,len);//将数据打印在终端
    }
	close(cfd);
    return 0;
}
~~~

![224](F:\学习专用\note\pic\224.png)

## 2.TCP协议

### 2.1TCP通信时序

![225](F:\学习专用\note\pic\225.png)

### 2.2半关闭

当TCP链接中客户端发送FIN请求关闭，服务器端回应ACK后（客户端端进入FIN_WAIT_2状态），服务器没有立即发送FIN给客户端时，客户端处在半链接状态，此时客户端可以接收服务器发送的数据，但是客户端已不能再向服务器发送数据。

### 2.3TCP状态转换

![229](F:\学习专用\note\pic\229.png)

注意：实线为主动打开，虚线为被动打开，细线为同时打开

**实际程序中抓取的状态：**netstat -apn | grep xxxx ---端口号，查看端口号状态

![230](F:\学习专用\note\pic\230.png)

![231](F:\学习专用\note\pic\231.png)

### 2.4端口复用

在server的TCP连接没有完全断开之前不允许重新监听是不合理的。因为，TCP连接没有完全断开指的是connfd（127.0.0.1:6666）没有完全断开，处于TIME_WAIT状态，而我们重新监听的是lis-tenfd（0.0.0.0:6666），虽然是占用同一个端口，但IP地址不同，connfd对应的是与某个客户端通讯的一个具体的IP地址，而listenfd对应的是wildcard address。

解决这个问题的方法是使用setsockopt()设置socket描述符的选项SO_REUSEADDR为1，表示允许创建端口号相同但IP地址不同的多个socket描述符。

在server代码的**socket()和bind()调用之间**插入如下代码：

 ~~~c
int opt = 1; 
setsockopt(listenfd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
 ~~~

有关setsockopt可以设置的其它选项请参考UNP第7章。

### 2.5C/S 架构TCP

![232](F:\学习专用\note\pic\232.png)

### 2.6TCP异常断开

**心跳检测机制**

在TCP网络通信中，经常会出现客户端和服务器之间的非正常断开，需要实时检测查询链接状态。常用的解决方法就是在程序中加入心跳机制。

Heart-Beat线程

这个是最常用的简单方法。在接收和发送数据时个人设计一个守护进程(线程)，定时发送Heart-Beat包，客户端/服务器收到该小包后，立刻返回相应的包即可检测对方是否实时在线。

该方法的好处是通用，但缺点就是会改变现有的通讯协议！大家一般都是使用业务层心跳来处理，主要是灵活可控。

UNIX网络编程不推荐使用SO_KEEPALIVE来做心跳检测，还是在业务层以心跳包做检测比较好，也方便控制。

**设置TCP属性**

O_KEEPALIVE保持连接检测对方主机是否崩溃，避免（服务器）永远阻塞于TCP连接的输入。

## 3.多进程并发服务器

使用多进程并发服务器时要考虑以下几点：

1. 父进程最大文件描述个数(父进程中需要close关闭accept返回的新文件描述符)

2. 系统内创建进程个数(与内存大小相关)

3. 进程创建过多是否降低整体服务性能(进程调度)

![226](F:\学习专用\note\pic\226.png)

### 案例：

~~~c
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>
#include<strings.h>
#include <sys/types.h> /* See NOTES */
#include <sys/socket.h>
#include <arpa/inet.h>
#include<ctype.h>
#include<sys/wait.h>
#include<signal.h>

#define SERV_PORT 7777
void sys_err(char *str)
{
        perror(str);
        exit(1);
}
void wait_child(int arg)
{
    while(waitpid(0,NULL,WNOHANG)>0);//非阻塞回收子进程
    return;
}
int main()
{
    int lfd,cfd;
    pid_t pid;
    char buf[BUFSIZ],cli_ip[BUFSIZ];
    struct sockaddr_in serv_addr,cli_addr;
    lfd=socket(AF_INET,SOCK_STREAM,0);
    if(lfd==-1)
    {
        sys_err("socket err");
    }
    bzero(&serv_addr,sizeof(serv_addr));
    bzero(&cli_addr,sizeof(cli_addr));
    
    socklen_t cli_len=sizeof(cli_addr);
    serv_addr.sin_family=AF_INET;
    serv_addr.sin_port=htons(SERV_PORT);
    serv_addr.sin_addr.s_addr=htonl(INADDR_ANY);
    //inet_pton(AF_INET,"192.168.1.220",&serv_addr.sin_addr.s_addr);
    int ret=bind(lfd,(struct sockaddr*)&serv_addr,sizeof(serv_addr));
    if(ret==-1)
    {
        sys_err("bind err");
    }
    int err=listen(lfd,128);
    if(err==-1)
    {
        sys_err("listen err");
    }
    while(1)
    {
        cfd=accept(lfd,(struct sockaddr*)&cli_addr,&cli_len);
        printf("client ip:%s port:%d\n",
               inet_ntop(AF_INET,&cli_addr.sin_addr.s_addr,cli_ip,sizeof(cli_ip)),ntohs(cli_addr.sin_port));
        if(cfd==-1)
        {
            sys_err("accept err");
        }
        pid=fork();
        if(pid<0)
        {
            sys_err("fork err");
        }
        else if(pid==0)//子进程用于与客户端进行通信
        {
            close(lfd);//关闭没用的套接字
            break;
        }
        else//父进程用于监听客户端连接请求，创建子进程对连接请求进行处理，并回收子进程
        {
            close(cfd);
            signal(SIGCHLD,wait_child);//注册信号捕捉函数
        }
    }
	if(pid==0)
    {
        while(1)
        {
            int n=read(cfd,buf,sizeof(buf));
            if(n==0)//此时客户端断开连接
            {
                close(cfd);
                return 0;
            }
            else if(n==-1)
            {
                sys_err("read err");
            }
            else
            {
                for(int i=0;i<n;i++)
                    buf[i]=toupper(buf[i]);
                write(cfd,buf,n);
            }
        }
    }
        
    return 0;
}
~~~

client.c 同上

![227](F:\学习专用\note\pic\227.png)

## 4.多线程并发服务器

在使用线程模型开发服务器时需考虑以下问题：

1. 调整进程内最大文件描述符上限

2. 线程如有共享数据，考虑线程同步

3. 服务于客户端线程退出时，退出处理。（退出值，分离态）

4. 系统负载，随着链接客户端增加，导致其它线程不能及时得到CPU

### 案例

~~~c
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>
#include<strings.h>
#include <sys/types.h> /* See NOTES */
#include <sys/socket.h>
#include <arpa/inet.h>
#include<ctype.h>
#include<sys/wait.h>
#include<signal.h>
#include<pthread.h>

#define SERV_PORT 7777
void sys_err(char *str)
{
        perror(str);
        exit(1);
}
struct info//将已连接的套接字描述符和连接的客户端，放在一个结构体中进行管理
{
    struct sockaddr_in cli_addr;
    int confd;
};
void *do_func(void *arg)//子线程与客户端通信
{
    char buf[BUFSIZ];
    struct info *ts=(struct info *)arg;
    while(1)
    {
        int n=read(ts->confd,buf,sizeof(buf));
        write(STDOUT_FILENO,buf,n);//打印客户端发送的数据到终端
        if(n==0)//此时客户端断开连接
        {
            break;
        }
        else if(n==-1)
        {
            sys_err("read err");
        }
        else
        {
            for(int i=0;i<n;i++)
                buf[i]=toupper(buf[i]);
            write(ts->confd,buf,n);
            
        }
    }
    close(ts->confd);
}
int main()
{
    int lfd,cfd;
    int i;
    char cli_ip[BUFSIZ];
    pthread_t tid;
    struct sockaddr_in serv_addr,cli_addr;
    struct info ts[256];
    lfd=socket(AF_INET,SOCK_STREAM,0);
    if(lfd==-1)
    {
        sys_err("socket err");
    }
    bzero(&serv_addr,sizeof(serv_addr));
    bzero(&cli_addr,sizeof(cli_addr));
    
    socklen_t cli_len=sizeof(cli_addr);
    serv_addr.sin_family=AF_INET;
    serv_addr.sin_port=htons(SERV_PORT);
    serv_addr.sin_addr.s_addr=htonl(INADDR_ANY);
    //inet_pton(AF_INET,"192.168.1.220",&serv_addr.sin_addr.s_addr);
    int ret=bind(lfd,(struct sockaddr*)&serv_addr,sizeof(serv_addr));
    if(ret==-1)
    {
        sys_err("bind err");
    }
    int err=listen(lfd,128);
    if(err==-1)
    {
        sys_err("listen err");
    }
    while(1)
    {
        cfd=accept(lfd,(struct sockaddr*)&cli_addr,&cli_len);
        printf("client ip:%s port:%d\n",
               inet_ntop(AF_INET,&cli_addr.sin_addr.s_addr,cli_ip,sizeof(cli_ip)),ntohs(cli_addr.sin_port));
        if(cfd==-1)
        {
            sys_err("accept err");
        }
        ts[i].cli_addr=cli_addr;
        ts[i].confd=cfd;
        pthread_create(&tid,NULL,do_func,(void *)&ts[i]);//创建子线程
        pthread_detach(tid);//线程分离，防止线程僵尸产生
        i++;
    }        
    return 0;
}
~~~

client.c 同上

![228](F:\学习专用\note\pic\228.png)

## 5.多路I/O转接服务器

多路IO转接服务器也叫做多任务IO服务器。该类服务器实现的主旨思想是，不再由应用程序自己监视客户端连接，取而代之由内核替应用程序监视文件。

![233](F:\学习专用\note\pic\233.png)

### 5.1select

-    select能监听的文件描述符个数受限于FD_SETSIZE,一般为1024，单纯改变进程打开的文件描述符个数并不能改变select监听文件个数
-  解决1024以下客户端时使用select是很合适的，但如果链接客户端过多，select采用的是轮询模型，会大大降低服务器响应效率，不应在select上投入更多精力

![234](F:\学习专用\note\pic\234.png)

~~~c
#include <sys/select.h>
/* According to earlier standards */
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>
int select(int nfds, fd_set *readfds, fd_set *writefds,
			fd_set *exceptfds, struct timeval *timeout);//返回值：成功：所监听的所有的监听集合中，满足条件的总数，失败：-1 设置errno
~~~

​         nfds:      监控的文件描述符集里最大文件描述符加1，因为此参数会告诉内核检测前多少个文件描述符的状态

​         readfds：   监控有读数据到达文件描述符集合，传入传出参数

​         writefds：  监控写数据到达文件描述符集合，传入传出参数

​         exceptfds： 监控异常发生达文件描述符集合,如带外数据到达异常，传入传出参数

​         timeout：   定时阻塞监控时间，3种情况

​                                     1.NULL，永远等下去

​                                     2.设置timeval，等待固定时间

​                                     3.设置timeval里时间均为0，检查描述字后立即返回，轮询

~~~c
struct timeval {
    long tv_sec; /* seconds */
    long tv_usec; /* microseconds */
};
void FD_CLR(int fd, fd_set *set);          //把文件描述符集合里fd清0
int FD_ISSET(int fd, fd_set *set);          //测试文件描述符集合里fd是否置1
void FD_SET(int fd, fd_set *set);  //把文件描述符集合里fd位置1
void FD_ZERO(fd_set *set);         //把文件描述符集合里所有位清0
~~~

### 5.2poll

![235](F:\学习专用\note\pic\235.png)

~~~c
#include <poll.h>
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
	struct pollfd {
		int fd; /* 文件描述符 */
		short events; /* 监控的事件 */
		short revents; /* 监控事件中满足条件返回的事件 */
	};
~~~

	POLLIN			普通或带外优先数据可读,即POLLRDNORM | POLLRDBAND
	POLLRDNORM		数据可读
	POLLRDBAND		优先级带数据可读
	POLLPRI 		高优先级可读数据
	POLLOUT			普通或带外数据可写
	POLLWRNORM		数据可写
	POLLWRBAND		优先级带数据可写
	POLLERR 		发生错误
	POLLHUP 		发生挂起
	POLLNVAL 		描述字不是一个打开的文件
	
	nfds 			监控数组中有多少文件描述符需要被监控
	
	timeout 		毫秒级等待
		-1：阻塞等，#define INFTIM -1 				Linux中没有定义此宏
		0：立即返回，不阻塞进程
		>0：等待指定毫秒数，如当前系统时间精度不够毫秒，向上取值
###     5.3epoll--！！

![237](F:\学习专用\note\pic\237.png)

epoll是Linux下多路复用IO接口select/poll的**增强版本**

- 1)突破文件描述符个数限制
- 2)**无须遍历整个被侦听的描述符集**，只要遍历那些被内核IO事件异步唤醒而加入Ready队列的描述符集合就行了。

目前epoll是linux大规模并发网络程序中的热门首选模型。

#### 5.3.1基础API

创建一个epoll句柄，参数size用来告诉内核监听的文件描述符的个数，跟内存大小有关。

~~~c++
#include<sys/epoll.h>
int epoll_create(int size);//size为监听的文件描述符的个数
~~~

控制某个epoll监控的文件描述符上的事件：注册、修改、删除。

~~~c
#include<sys/epoll.h>
int epoll_ctl(int epfd,int op,int fd,struct epoll_event *event);
struct epoll_event {
    __uint32_t events; /* Epoll events */
    epoll_data_t data; /* User data variable */
};
typedef union epoll_data {
    void *ptr;
    int fd;
    uint32_t u32;
    uint64_t u64;
} epoll_data_t;
~~~

- ​     epfd：  为epoll_creat的句柄
- ​      op：    表示动作，用3个宏来表示：

​            EPOLL_CTL_ADD (注册新的fd到epfd)，

​            EPOLL_CTL_MOD (修改已经注册的fd的监听事件)，

​            EPOLL_CTL_DEL (从epfd删除一个fd)；

- ​        event： 告诉内核需要监听的事件

​		 **EPOLLIN** ：  表示对应的文件描述符可以读（包括对端SOCKET正常关闭）

​		**EPOLLOUT**：  表示对应的文件描述符可以写

​		 **EPOLLERR**：  表示对应的文件描述符发生错误

等待所监控文件描述符上有事件的产生，类似于select()调用。

~~~c
	#include <sys/epoll.h>
	int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);//返回值： 成功返回有多少文件描述符就绪，时间到时返回0，出错返回-1
~~~

​        events：    用来存内核得到事件的集合，

​        maxevents： 告之内核这个events有多大，这个maxevents的值不能大于创建epoll_create()时的size，

​        timeout：   是超时时间

​            -1： 阻塞

​            0： 立即返回，非阻塞

​            \>0： 指定毫秒

####      5.3.2 案例

server.c

~~~c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include<unistd.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <sys/epoll.h>
#include <errno.h>
#include<ctype.h>

#define SERV_PORT 7777
#define OPEN_MAX 5000//监听的文件描述符的个数
#define MAXLINE 8192//传输数据大小限制
void sys_err(char *str)
{
        perror(str);
        exit(1);
}
int main()
{
    int lfd,cfd,sockfd,n;//lfd为监听描述符，cfd为已连接的描述符，sockfd为正在通信的描述符
    ssize_t nready, efd, res;
    socklen_t cli_len;
    char buf[BUFSIZ],cli_ip[BUFSIZ];
    struct sockaddr_in serv_addr,cli_addr;
    struct epoll_event tep, ep[OPEN_MAX];       //tep: epoll_ctl参数  ep[] : epoll_wait参数
    lfd=socket(AF_INET,SOCK_STREAM,0);
    if(lfd==-1)
		sys_err("socket err");
    int opt = 1;
    setsockopt(lfd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));      //端口复用
    bzero(&serv_addr,sizeof(serv_addr));
    bzero(&cli_addr,sizeof(cli_addr));
    
    
    serv_addr.sin_family=AF_INET;
    serv_addr.sin_port=htons(SERV_PORT);
    serv_addr.sin_addr.s_addr=htonl(INADDR_ANY);
    //inet_pton(AF_INET,"192.168.1.220",&serv_addr.sin_addr.s_addr);
    
    int ret=bind(lfd,(struct sockaddr*)&serv_addr,sizeof(serv_addr));
    if(ret==-1)
		sys_err("bind err");
    int err=listen(lfd,128);
    if(err==-1)
        sys_err("listen err");
    efd=epoll_create(OPEN_MAX); //创建epoll模型，efd指向红黑色根节点
    if(efd==-1)
        sys_err("epoll_create err");
    tep.events=EPOLLIN;tep.data.fd=lfd;//指定lfd的监听事件为读
    res=epoll_ctl(efd,EPOLL_CTL_ADD,lfd,&tep);//将lfd及对应的结构体设置到树上,efd可找到该树
    if(res==-1)
        sys_err("epoll_ctl err");
    for(;;)
    {
        /*epoll为server阻塞监听事件, ep为struct epoll_event类型数组, OPEN_MAX为数组容量, -1表永久阻塞*/
        nready=epoll_wait(efd,ep,OPEN_MAX,-1);
        if(nready==-1)
            sys_err("epoll_wait err");
        for(int i=0;i<nready;i++)
        {
            if(!(ep[i].events&EPOLLIN))//如果不是“读”事件，继续循环，“读”事件，即有请求连接，或有数据写入
                continue;
            if(ep[i].data.fd==lfd) //判断满足事件的fd是不是lfd，即是否为请求连接
            {
                cli_len=sizeof(cli_addr);
                cfd=accept(lfd,(struct sockaddr*)&cli_addr,&cli_len);
                if(cfd==-1)
                    sys_err("accept err");
                printf("client ip:%s\tport:%d\n",                   inet_ntop(AF_INET,&cli_addr.sin_addr.s_addr,cli_ip,sizeof(cli_ip)),ntohs(cli_addr.sin_port));
                tep.events=EPOLLIN;tep.data.fd=cfd;
                res=epoll_ctl(efd,EPOLL_CTL_ADD,cfd,&tep);
                if(res==-1)
                    sys_err("epoll_ctl err");
            }
            else//不是lfd,有数据发送给服务器
            {
                sockfd=ep[i].data.fd;
                n=read(sockfd,buf,MAXLINE);
                if(n==0)//读到0，说明客户端关闭连接
                {
                    res=epoll_ctl(efd,EPOLL_CTL_DEL,sockfd,NULL);//将该文件描述符从红黑树中删除
                    if(res==-1)
                        sys_err("epoll_ctl err");
                    close(sockfd);
                    printf("client[%d] closed connection\n", sockfd);
                }
                else if(n<0)
                {
                    perror("read n < 0 error: ");
                    res = epoll_ctl(efd, EPOLL_CTL_DEL, sockfd, NULL);
                    close(sockfd);
                }
                 else
                 {
                     for (i = 0; i < n; i++)
                         buf[i] = toupper(buf[i]);   //转大写,写回给客户端
                     write(STDOUT_FILENO, buf, n);
                     write(sockfd, buf, n);
                 }          
            }
        }
    }
    close(lfd);
    close(efd);
    return 0;
}
~~~

client.c 同上

 ![236](F:\学习专用\note\pic\236.png)

#### 5.3.3epoll进阶

##### 5.3.3.1事件模型

**LT模式**（水平触发）：当epoll_wait检测到**描述符事件发生**并将此事件通知应用程序，应用程序可以**不立即处理该事件**。下次调用epoll_wait时，会再次响应应用程序并通知此事件，**只要有描述符事件就会通知应用程序**。

**ET模式**（边沿触发）：当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序必须**立即处理该事件**。如果不处理，下次调用epoll_wait时，不会再次响应应用程序并通知此事件，即**只有描述符事件发生时，才会通知应用程序**。

## 6.线程池

![238](F:\学习专用\note\pic\238.png)

1)   为什么需要线程池

大多数的网络服务器，包括Web服务器都具有一个特点，就是单位时间内必须处理数目巨大的连接请求，但是处理时间却是比较短的。在传统的多线程服务器模型中是这样实现的：一旦有个请求到达，就创建一个新的线程，由该线程执行任务，任务执行完毕之后，线程就退出。这就是”即时创建，即时销毁”的策略。尽管与创建进程相比，创建线程的时间已经大大的缩短，但是如果提交给线程的任务是执行时间较短，而且执行次数非常频繁，那么服务器就将处于一个不停的创建线程和销毁线程的状态。这笔开销是不可忽略的，尤其是线程执行的时间非常非常短的情况。**不停地创建和销毁线程，需要消耗大量CPU**

2)   线程池原理

在**应用程序启动之后，就马上创建一定数量的线程，放入空闲的队列**中。这些线程都是处于**阻塞**状态，这些线程只占一点内存，不占用CPU。当任务到来后，线程池将选择一个空闲的线程，将任务传入此线程中运行。当所有的线程都处在处理任务的时候，线程池将自动**创建一定的数量的新线程**，用于处理更多的任务。执行任务完成之后线程并不退出，而是继续在线程池中等待下一次任务。当**大部分线程处于阻塞状态时，线程池将自动销毁一部分的线程，回收系统资源**。

3)   线程池的作用

需要大量的线程来完成任务，且完成任务的时间比较短；对性能要求苛刻的应用；

## 7.UDP协议

### TCP和UDP优缺点

**TCP:面向连接的可靠的数据包**

​	优点：稳定：1.数据稳定---丢包回传（丢包率千分之97）

​				2.速率稳定---固定传输路径

​				3.流量稳定---滑动窗口机制

​	缺点：效率低、速度慢

​	使用场景：大文件，重要文件传输

**UDP:无连接的不可靠的数据报**

​	优点：效率高、速度快

​	缺点：不稳定：数据，速率，流量

使用场景：对实时性要求较高，视频，网络电话

### 案例

server.c

~~~c
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>
#include<sys/socket.h>
#include<sys/types.h>
#include<arpa/inet.h>
#include<ctype.h>
#include<string.h>
void sys_err(char *str)
{
    perror(str);
    exit(1);
}
#define SERV_PORT 7777
int main()
{
    int sockfd,n;
    char buf[BUFSIZ],cli_ip[64];
    struct sockaddr_in serv_addr,cli_addr;
    socklen_t cli_len;
    sockfd=socket(AF_INET,SOCK_DGRAM,0);
    if(sockfd==-1)
        sys_err("socket err");
    bzero(&serv_addr,sizeof(serv_addr));
    bzero(&cli_addr,sizeof(cli_addr));
    serv_addr.sin_family=AF_INET;
    serv_addr.sin_port=htons(SERV_PORT);
    serv_addr.sin_addr.s_addr=htonl(INADDR_ANY);
    int ret=bind(sockfd,(struct sockaddr*)&serv_addr,sizeof(serv_addr));
    if(ret==-1)
        sys_err("bind err");
    while(1)
    {
        cli_len =sizeof(cli_addr);
        n=recvfrom(sockfd,buf,BUFSIZ,0,(struct sockaddr*)&cli_addr,&cli_len);
        if(n==-1)
            sys_err("recvfrom err");
         printf("clinet ip:%s client port:%d\n",inet_ntop(AF_INET,&cli_addr.sin_addr.s_addr,cli_ip,sizeof(cli_ip)),ntohs(cli_addr.sin_port));//打印已连接的客户端
        for(int i=0;i<n;i++)
            buf[i]=toupper(buf[i]);
        n=sendto(sockfd,buf,n,0,(struct sockaddr*)&cli_addr,sizeof(cli_addr));
        if(n==-1)
            sys_err("sendto err");
        write(STDOUT_FILENO,buf,n);
    }
    close(sockfd);
    return 0;
}
~~~

client.c

~~~c
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>
#include<sys/socket.h>
#include<sys/types.h>
#include<arpa/inet.h>
#include<ctype.h>
#include<string.h>

#define SERV_IP "127.0.0.1"
#define SERV_PORT 7777
void sys_err(char *str)
{
    perror(str);
    exit(1);
}

int main()
{
    int sockfd,n;
    char buf[BUFSIZ];
    struct sockaddr_in serv_addr;
    socklen_t cli_len;
    sockfd=socket(AF_INET,SOCK_DGRAM,0);
    if(sockfd==-1)
        sys_err("socket err");
    bzero(&serv_addr,sizeof(serv_addr));
    serv_addr.sin_family=AF_INET;
    serv_addr.sin_port=htons(SERV_PORT);
    inet_pton(AF_INET,SERV_IP,&serv_addr.sin_addr.s_addr);
    while(1)
    {
        fgets(buf,sizeof(buf),stdin);
        n=sendto(sockfd,buf,strlen(buf),0,(struct sockaddr*)&serv_addr,sizeof(serv_addr));
        if(n==-1)
            sys_err("sendto err");
        n=recvfrom(sockfd,buf,BUFSIZ,0,NULL,0);
        if(n==-1)
            sys_err("sendto err");
        write(STDOUT_FILENO,buf,n);
    }
    close(sockfd);
    return 0;
}
~~~

![239](F:\学习专用\note\pic\239.png)

### C/S 架构UDP

![240](F:\学习专用\note\pic\240.png)