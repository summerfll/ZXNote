# 【UNIX环境高级编程】线程控制

## 1.线程属性

使用pthread_attr_t 结构修改线程默认属性，并把这些属性与创建的线程联系起来。

**属性初始化：**

~~~c
#include<pthread.h>
int pthread_attr_init(pthread_attr_t *attr);//返回值：成功0，出错返回错误编号
~~~

**去除初始化：**

~~~c
#include<pthread.h>
int pthread_attr_destroy(pthread_attr_t *attr);//返回值：成功0，出错返回错误编号
~~~

**分离线程：**

使用pthread_attr_setdetachstate函数把线程属性detachstate设置为以下两个合法值之一：

- 设置为PTHREAD_CREATE_DETACHED,以分离状态启动线程
- 设置为PTHREAD_CREATE_JOINABLE,正常启动线程

~~~c
#include<pthrad.h>
int pthread_attr_getdetachstate(const pthread_attr_t *restrict attr,int *detachstate);
int pthread_attr_setdetachstate(pthread_sttr_t *attr,int detachstate);//返回值：成功0，出错返回错误编号
~~~

**例程：以分离状态创建线程**

~~~c
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
~~~

## 2.同步属性

### 2.1互斥量属性

**初始化：**

~~~c
#include<pthread.h>
int pthread_mutexattr_init(pthread_mutexattr_t *attr);//返回值：成功0，出错返回错误编号
~~~

**回收：**

~~~c
#include<pthread.h>
int pthread_mutexattr_destroy(pthread_mutexattr_t *attr);//返回值：成功0，出错返回错误编号
~~~

#### 互斥量**进程共享属性和类型属性**

**进程共享属性：**pthread_mutexattr_getpshared查询进程共享属性，pthread_mutexattr_setpshared修改进程共享属性

~~~c
#include<pthread.h>
int pthread_mutexattr_getpshared(const pthread_mutexattr_t *restrict attr,int *restrict pshared);
int pthread_mutexattr_setpshared(const pthread_mutexattr_t *attr,int *pshared);//返回值：成功0，出错返回错误编号
~~~

**类型属性：**pthread_mutexattr_gettype查询互斥量类型属性，pthread_mutexattr_settype修改互斥量类型属性

~~~c
#include<pthread.h>
int pthread_mutexattr_gettype(const pthread_mutexattr_t *restrict attr,int *restrict type);
int pthread_mutexattr_settype(pthread_mutexattr_t *attr,int type);//返回值：成功0，出错返回错误编号
~~~

### 2.2读写锁属性

**读写锁属性初始化：**

~~~c
#include<pthread.h>
int pthread_rwlockattr_init(pthread_rwlockattr_t *attr);//返回值：成功0，出错返回错误编号
~~~

**回收：**

~~~c
#include<pthread.h>
int pthread_rwlockattr_destroy(pthread_rwlockattr_t *attr);//返回值：成功0，出错返回错误编号
~~~

读写锁支持唯一属性是进程共享属性，与互斥量的进程共享属性相同。

**读取设置读写锁的共享属性：**

~~~c
#include<pthread.h>
int pthread_rwlockattr_getpshared(const pthread_rwlockattr_t *restrict attr,int *restrict pshared);
int pthread_rwlockattr_setpshared(const pthread_rwlockattr_t *attr,int *pshared);//返回值：成功0，出错返回错误编号
~~~

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

~~~c
#include<pthread.h>
int pthread_key_create(pthread_key_t *keyp,void (*destructor)(void *));//返回值：成功0，出错返回错误编号
~~~

创建的键存放在keyp指向的内存单元，这个键可以被进程中的所有线程使用，但每个线程把这个键与不同的线程私有数据地址进行关联。

**取消键与线程私有数据值得关联：****取消键**

~~~c
#include<pthread.h>
int pthread_key_delete(pthread_key_t *key);//返回值：成功0，出错返回错误编号
~~~

**键和线程私有数据关联：**

~~~c
#include<pthread.h>
void *pthread_getspecific(pthread_key_t key);//返回值：线程私有数据值，若没有值与键关联返回NULL
int pthread_setspecific(pthread_key_t key,const void *value);//返回值：成功0，出错返回错误编号
~~~

## 5.取消选项

有两个线程属性并没有包含在pthread_attr_t 结构中，他们是**可取消状态**和**可取消类型**

这两个属性影响着线程在相应pthread_cancel函数调用时所呈现的行为。

### **可取消状态属性**

可以是PTHREAD_CANCEL_ENABLE,也可以是PTHREAD_CANCEL_DISABLE

pthread_setcancelstate修改可取消状态

~~~c
#include<pthread.h>
int pthread_setcancelstate(int state,int *lodstate);//返回值：成功0，出错返回错误编号
~~~

### 可取消类型

也称为延迟取消，调用pthread_cancel后，在线程到达取消点前，并不会出现真正的取消。pthread_setcanceltype来修改取消类型

~~~c
#include<pthread.h>
int pthread_setcanceltype(int type,int *oldtype);//返回值：成功0，出错返回错误编号
~~~

## 6.线程和信号

每个线程都有自己的信号屏蔽字，但是信号的处理是进程中所有线程共享的。

**阻止信号发送：**

~~~c
#include<pthread.h>
int pthread_sigmask(int how,const sigset_t *restrict set,sigset_t *restrict oset);//返回值：成功0，出错返回错误编号
~~~

**等待信号发生：**

~~~c
#include<pthread.h>
int sigwait(const sigset_t *restrict set,int *restrict signop);//返回值：成功0，出错返回错误编号
~~~

set参数指出线程等待的信号集，signop指向的整数将作为返回值，表明发送信号的数量

## 7.线程和fork

**清除锁状态：**

通过调用pthread_atfork函数建立fork处理函数

~~~c
#include<pthread.h>
int pthread_atfork(void (*prepare)(void),void (*parent)(void),void (*child)(void));//返回值：成功0，出错返回错误编号
~~~

pthread_atfork可安装三个帮助清理锁的函数：

- prepare fork处理程序由父进程在fork创建子进程前调用，获取父进程定义的所有锁
- parent fork处理程序是在fork创建了子进程以后，但在fork返回之前在父进程环境中调用，对prepare fork处理程序获得的所有锁进行解锁
- child fork处理程序在fork返回之前在子进程环境中调用，释放prepare fork处理程序获得的所有锁。

**例程：**

~~~c
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
~~~

**运行结果：**

![46](F:\学习专用\学习笔记\图片\46.png)

（1）程序中定义了两个互斥量，lock1和lock2，prepare fork处理程序获取这两把锁，child fork处理程序在子进程中释放锁，parent fork在父进程中释放锁

（2）prepare fork处理程序在调用fork后运行，child fork处理程序在fork调用返回到子进程之前运行，parent fork处理程序在fork调用返回给父进程前运行、





