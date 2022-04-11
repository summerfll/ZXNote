# 【UNIX环境高级编程】线程

## 1.引言

一个进程中的所有线程都可以访问该进程的组成部件，如文件描述符和内存

## 2.线程标识

进程ID在整个系统中是唯一的，但线程ID只有在它所属的进程环境中才有效。

线程ID用pthread_t数据类型表示。

**比较两个线程ID：**

~~~c
#include<pthread.h>
int pthread_equal(pthread_t tid1,pthread_t tid2);
~~~

**获得自身的线程ID**

~~~c
#include<pthread.h>
pthread_t pthread_self(void);//返回值：调用线程的线程ID
~~~

## 3.线程创建

新增线程通过pthread_create函数创建

~~~c
#include<pthread.h>
int pthread_create(pthread_t *restrict tidp,const pthread_attr_t *restrict attr,void *(*start_rtn)(void *),void *restrict arg);//返回值：成功返回0，否则返回出错编号
~~~

当pthread_create成功返回时，由tidp指向的内存单元被设置为新创建线程的线程ID。attr参数用于定制各种不同的线程属性。新创建的线程从start_rtn函数的地址开始运行，该函数只有一个无类型指针参数arg，如果需要向start_rtn函数传递的参数不止一个，那么需要把这些参数放到一个结构中，然后把这个结构的地址作为arg参数传入。

线程创建时不能保证哪个线程先运行：是新创建的线程还是调用线程。

**例程：**

创建一个线程并打印进程ID，新线程的线程ID及初始线程ID

~~~c
#include"apue.h"
#include<pthread.h>
pthread_t ntid;
void printids(const char *s)
{
    pid_t pid;
    pthread_t tid;
    pid=getpid();
    tid=pthread_self();
    printf("%s pid %u tid %u (0x%x)\n",s,(unsigned int)pid,(unsigned int)tid,(unsigned int)tid);
}
void *thr_fn(void *arg)
{
    printids("new thread: ");
    return((void *)0);
}
int main()
{
    int err;
    err=pthread_create(&ntid,NULL,thr_fn,NULL);
    if(err != 0)
        err_quit("can't create:%s\n",strerror(err));
    printids("main thread: ");
    sleep(1);
    exit(0);
}

~~~

运行结果：

![43](F:\学习专用\学习笔记\图片\43.png)

说明：

- Linux系统编译程序指令：gcc pthread.c -lpthread -o pthread
- 此例程中需要处理主线程和新线程之间的竞争，sleep(1)用于使主线程休眠1s，防止新线程还未执行完程序主线程就已经退出程序。

## 4.线程终止

单个线程通过下列三种方式退出，在不终止整个进程的情况下停止它的控制流

- 线程只是从启动例程种返回，返回值是线程的退出码
- 线程可以被同一进程中的其他线程取消
- 线程调用pthread_exit

~~~c
#include<pthread.h>
void pthread_exit(void *rval_ptr);
~~~

rval_ptr是一个无类型指针，与传给启动例程的单个参数类似。

进程中的其他线程可以通过调用pthread_join函数访问这个指针

~~~c
#include<pthread.h>
int pthread_join(pthread_t thread,void **rval_ptr);//返回值：成功返回0，否则返回出错编号
~~~

通过调用pthread_join自动把线程置于分离状态，如果线程已经处于分离状态，pthread_join调用就会失败，返回EINVAL

**例程**：

获取已终止的线程的退出码

~~~c
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

~~~

运行结果：

![44](F:\学习专用\学习笔记\图片\44.png)

当一个线程通过调用pthread_exit退出或从启动例程中返回，进程中的其他线程可以通过调用pthread_join函数获得该线程的退出状态。

线程可以通过调用pthread_cancel函数来请求取消同一进程中的其他线程

~~~c
#include<pthread.h>
int pthread_cancel(pthread_t tid);//返回值：成功返回0，否则返回错误编号
~~~

pthread_cancel并不等待线程终止，它仅仅提出请求

线程可以安排它退出时需要调用的函数，这与进程可以调用atexit函数安排进程退出时需要调用的函数类似。这样的函数为线程清理处理程序。

处理程序记录在栈中，它们的执行顺序与它们注册时的顺序相反

~~~c
#include<pthread.h>
void pthread_cleanup_push(void (*rtn)(void *),void *arg);
void pthread_cleanup_pop(int execute);
~~~

当线程执行以下动作时调用清理函数，调用参数为arg，清理函数rtn的调用顺序是由pthread_cleanup_push函数安排

- 调用pthread_exit时
- 相应取消请求时
- 用非零execute参数调用pthread_cleanup_pop时

**线程函数和进程函数相似之处**

![45](F:\学习专用\学习笔记\图片\45.png)

## 5.线程同步

### 5.1互斥量

可以通过使用pthread的互斥接口保护数据，确保同一时间只有一个线程访问数据。

互斥量本质上说是一把锁，在访问共享资源前对互斥量进行加锁，在访问完成后释放互斥量上的锁。对互斥量进行加锁后，任何其他试图再次对互斥量加锁的线程将会被阻塞直到当前线程释放该互斥锁。

互斥量用pthread_mutex_t数据类型来表示，在使用互斥量前，必须首先对它进行初始化，可以把他置为常量PTHREAD_MUTEX_INITIALLZER(只对静态分配的互斥量)。也可调用pthread_mutex_init函数进行初始化。如果动态的分配互斥量（如malloc函数）,那么在释放前需要调用pthread_mutex_destroy

~~~c
#include<pthread.h>
int pthread_mutex_init(pthread_mutex_t *restrict mutex,const pthread_mutexattr_t *restrict attr);
int pthread_mutex_destroy(pthread_mutex_t *mutex);//返回值：成功返回0，否则返回错误编号
~~~

要默认的属性初始化互斥量，只需把attr设置为NULL。

对**互斥量进行加锁**，需要调用pthread_mutex_lock，如果互斥量已经上锁，调用线程将阻塞直到互斥量被解锁。

对**互斥量解锁**，需要调用pthread_mutex_unlock

~~~c
#include<pthread.h>
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_trylock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);//返回值：成功返回0，否则返回错误编号
~~~

pthread_mutex_trylock尝试对互斥量加锁，线程不被阻塞，调用pthread_mutex_trylock时互斥量处于未锁柱状态，则锁柱互斥量，返回0，否则不能锁柱，返回EBUSY

**实例：**

使用互斥量保护数据结构

~~~c
#include<stdlib.h>
#include<pthread.h>
struct foo{
    int f_count;
    pthread_mutex_t f_lock;
};
struct f00 *foo_alloc(void)
{
    struct foo *fp;
    if((fp = malloc(sizeof(struct foo))) != NULL)//用malloc动态分配互斥量和计数变量
    {
        fp->f_count = 1;
        if(pthread_mutex_init(&fp->f_lock,NULL)!=0)//初始化互斥量
        {
            free(fp);
            return(NULL);
        }
    }
    return(fp);
}
void foo_hold(struct foo *fp)
{
    pthread_mutex_lock(&fp->f_lock);//对互斥量加锁并修改计数变量的值
    fp->count++;
    pthread_mutex_unlock(&fp->f_lock);
}

void foo_rele(struct foo *fp)
{
    pthread_mutex_lock(&fp->f_lock);
    if(--fp->f_count == 0){
        pthread_mutex_unlock(&fp->f_lock);
        pthread_mutex_destroy(&fp->f_lock);
        free(fp);//释放内存
    }else{
        pthread_mutex_unlock(&fp->f_lock);
    }
}
~~~

### 5.2避免死锁

**死锁：**

如果线程试图对同一个互斥量加锁两次，那么它自身就会陷入死锁状态。例：程序中使用多个互斥量，如果允许一个线程一直占用第一个互斥量，并且在试图锁住第二个互斥量时处于阻塞状态，但拥有第二个互斥量的线程也在试图锁住第一个互斥量，这就发生死锁。即：两个线程都在请求另一个线程拥有的资源，所有两个线程都无法向前运行，于是产生**死锁**。

可以通过小心地控制互斥量加锁地顺序来避免死锁地发生。例：假设需要对两个互斥量A和B同时加锁，如果所有线程总是在对互斥量B加锁之前锁住互斥量A，那么使用者两个互斥量不会产生死锁。类似地，如果所有线程总是在锁住互斥量A之前锁住互斥量B，那么也不会产生死锁。只有在一个线程试图以与另一个线程相反地顺序锁住互斥量时，才可能出现死锁。

### 5.3读写锁

读写锁类似于互斥量，不过读写锁允许更高地并行性。

**读写锁地三种状态：**

- 读模式下加锁
- 写模式下加锁
- 不加锁

一次只有一个线程可以占有写模式地读写锁，但是多个线程可以同时占有读模式地读写锁

（1）当读写锁是写加锁状态时，在解锁前，所有试图对这个锁加锁的线程都会被阻塞。

（2）当读写锁是读加锁时，所有试图以读模式对它进行加锁的线程都可以实现，但如果线程以写模式对此加锁，它必须阻塞直到所有线程释放读锁。

（3）当读写锁是读加锁时，如果有另外的线程试图以写模式加锁，读写锁会阻塞随后的读模式锁请求，避免读模式锁长期占用

读写锁也叫共享-独占锁，当读写锁以读模式锁住时，它是以共享模式锁住的，当它以写模式锁住时，它是以独占模式锁住的。

**读写锁初始化：**

~~~c
#include<pthread.h>
int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock,const pthread_rwlockattr_t *restrict attr);
int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);//两者返回值：成功返回0，否则返回错误编号
~~~

若希望读写锁有默认属性，attr为NULL

pthread_rwlock_init为读写锁分配资源，pthread_rwlock_destroy将释放这些资源

**读模式下加锁，写模式下加锁，解锁：**

~~~c
#include<pthread.h>
int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);//读模式下加锁
int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);//写模式下加锁
int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);//解锁
//所有的返回值：成功返回0，否则返回错误编号
~~~

### 5.4条件变量

条件变量给多个线程提供一个会合的场所。条件变量和互斥量一起使用，允许线程以无竞争的方式等待特定的条件发生。

条件本身是由互斥量保护的，线程在改变条件状态前必须首先锁住互斥量，其他线程在获得互斥量前不会察觉这种变化，因为必须锁住互斥量后才能计算条件。

**条件变量初始化：**

pthread_cond_t 数据类型代表的条件变量以两种方式初始化，（1）把常量PTHREAD_COND_INITIALIZER赋给静态分配的条件变量，（2）条件变量是动态分配的，可用pthread_cond_init函数初始化

~~~c
#include<pthread.h>
int pthread_cond_init(pthread_cond_t *rectrict cond,pthread_condattr_t *restrict attr);
int pthread_cond_destroy(pthread_cond_t *cond);//两返回值：成功返回0，否则返回错误编号
~~~

pthread_cond_init中attr一般默认为NULL

使用pthread_cond_wait等待条件变为真，如果在给定的时间内条件不满足，会生成一个错误码

~~~c
#include<pthread.h>
int pthread_cond_wait(pthread_cond_t *restrict cond,pthread_mutex_t *restrict mutex);
int pthread_cond_timedwait(pthread_cond_t *restrict cond,pthread_mutex_t *restrict mutex,const struct timespec *restrict timeout);//两返回值：成功返回0，否则返回错误编号
~~~

传递给pthread_cond_wait的互斥量对条件进行保护，调用者把锁住的互斥量传给函数。

pthread_cond_signal函数唤醒等待该条件的某个线程，pthread_cond_broadcast函数唤醒等待该条件的所有线程

~~~c
#include<pthread.h>
int pthread_cond_signal(pthread_cond_t *cond);
int pthread_cond_broadcast(pthread_cond_t *cond);//两返回值：成功返回0，否则返回错误编号
~~~



**例程：**

使用条件变量和互斥量对线程进行同步

~~~c
#include<pthread.h>
struct msg{
    struct msg *m_next;
};
struct msg *workq;
pthread_cond_t qready = PTHREAD_COND_INITIALIZER;
pthread_mutex_t qlock = PTHREAD_MUTEX_INITIALIZER;

void process_msg(void)
{
    struct msg *mp;
    for(;;)
    {
        pthread_mutex_lock(&qlock);
        while(workq == NULL)
            pthread_cond_wait(&qready,&qlock);
        mp = workq;
        workq = mp->m_next;
        pthread_mutex_unlock(&qlock);
    }
}
void enqueue_msg(struct msg *mp)
{
    pthread_mutex_lock(&qlock);
    mp->m_next = workq;
    workq=mp;
    pthread_mutex_unlock(&qlock);
    pthread_cond_signal(&qready);
}
~~~













