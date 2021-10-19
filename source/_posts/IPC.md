### 1. 进程通信

-   什么是进程间通信？

-   什么是线程间通信？

    **进程通信：在用户空间实现进程通信是不可能的，通过linux内核通信。**

    <details>
    <summary>进程代码说明</summary>
    <pre>
    <code>
    #include "dong.h"
    int main()
    {
        int i;
        pid_t pid;
        pid = fork();
        if (pid < 0)
        {
            printf("fork error. msg:[%s]\n", strerror(errno));
            return -1;
        }
        if (pid == 0)
        {
            for (i = 0; i < 10; i++)
                printf("I'm child i:[%d]\n", i);
        }
        for (i = 0; i < 10; i++)
            printf("I'm parent i:[%d]\n", i);
        return 0;
    }
    </code>
    </pre>
    </details>

    >   注：打印出的返回值有可能不是父进程运行完子进程才运行，想要实现这种，需要通过**内核**通信，子进程获取父进程在内核中写入的状态才能限制父进程先运行。

    **线程间通信：在用户空间就能实现，可以通过全局变量通信。**

    <details>
    <summary>线程代码说明</summary>
    <pre>
    <code>
    #include "dong.h"
    int thread_val = 0;  /*线程全局变量能够实现线程间的数据信息共享*/
    void func(void *arg)
    {
        int i;
        while (thread_val == 0);   /*直到thread_val=1才会继续下面的运行*/
        for (i = 0; i < 10; i++)
            printf("I'm child thread i:[%d]\n", i);
        return ;
    }
    int main()
    {
        int i;
        int iRet;
        pthread_t thread;
        iRet = pthread_create(&thread, NULL, (void *)&func, (void *)NULL);
        for (i = 0; i < 10; i++)
            printf("I'm parent thread i:[%d]\n", i);
        thread_val = 1;
        sleep(3);   /*内核创建线程没有那么快，休眠一下可以让子线程继续跑*/
        return 0;
    }
    </code>
    </pre>
    </details>

### 2. 进程间通信使用的目的

1.  [数据传输](https://wiki.mbalib.com/wiki/数据传输)：一个进程需要将它的[数据](https://wiki.mbalib.com/wiki/数据)[发送](https://wiki.mbalib.com/wiki/发送)给另一个进程，发送的数据量在一个字节到几[兆字节](https://wiki.mbalib.com/wiki/兆字节)之间。
2.  共享数据：多个进程想要操作共享数据，一个进程对共享数据的修改，别的进程[应该](https://wiki.mbalib.com/wiki/应该)立刻看到。
3.  通知事件：一个进程需要向另一个或一组进程发送消息，通知它（它们）发生了某种事件（如进程终止时要通知父进程）。
4.  资源共享：多个进程之间共享同样的[资源](https://wiki.mbalib.com/wiki/资源)。为了做到这一点，[需要](https://wiki.mbalib.com/wiki/需要)内核提供锁和同步机制。
5.  进程控制：有些进程希望完全[控制](https://wiki.mbalib.com/wiki/控制)另一个进程的执行（如Debug进程），此时控制进程希望能够拦截另一个进程的所有陷入和异常，并能够及时知道它的状态改变。

　　进程通过与内核及其它进程之间的互相通信来[协调](https://wiki.mbalib.com/wiki/协调)它们的[行为](https://wiki.mbalib.com/wiki/行为)。如[Linux](https://wiki.mbalib.com/wiki/Linux)支持多种进程间通信（IPC）机制，[信号](https://wiki.mbalib.com/wiki/信号)和管道是其中的两种。除此之外，Linux还支持System V的IPC机制（用首次出现的Unix版本命名）。

### 3. 进程间几种通信方式

-   管道通信：无名管道、有名管道（文件系统中有名）。

    -   管道可用于具有亲缘关系进程间的通信，有名管道除了具有管道所具有的功能外，它还允许无亲缘关系进程间的通信。

-   信号通信：信号（通知）通信包括：信号的发送、信号的接收和信号的处理。

    -   [信号](https://wiki.mbalib.com/wiki/信号)是一种软中断，它是比较复杂的通信方式，用于通知进程有某事件发生，一个进程收到一个信号与处理器收到一个中断请求效果上可以说是一致的.

-   IPC(Inter-Process Communication)通信：共享内存、消息队列和信号(灯)量。

    -   消息队列是[消息](https://wiki.mbalib.com/wiki/消息)的链接表，它克服了上两种通信方式中信号量有限的缺点，具有写权限的进程可以按照一定的规则向消息队列中添加新[信息](https://wiki.mbalib.com/wiki/信息)；对消息队列有读权限的进程则可以从消息队列中读取[信息](https://wiki.mbalib.com/wiki/信息)。
    -   可以说这是最有用的进程间通信方式。它使得多个进程可以访问同一块内存空间，不同进程可以及时看到对方进程中对共享内存中[数据](https://wiki.mbalib.com/wiki/数据)的更新。这种方式需要依靠某种同步操作，如互斥锁和信号量等。 
    -   信号量（semaphore）：主要作为进程之间及同一种进程的不同线程之间得同步和互斥手段。

    >   **以上是单机模式下的进程通信(只有一个Linux内核)**

-   Socket通信：存在于一个网络中两个进程之间的通信(**两个linux内核**)
    -   这是一种更为一般的进程间通信机制，它可用于[网络](https://wiki.mbalib.com/wiki/网络)中不同机器之间的进程间通信，应用非常广泛。

#### 3.1 管道

- 进程1---------->管道(内核)----------->进程2

- 管道文件是一个特殊的文件，是由队列来实现的。

- 功能介绍：

    ```
    函数形式：int pipe(int fd[2]);
    功   能：创建管道，为系统调用
    参   数：就是得到的文件描述符。可见有两个文件描述符：fd[0]和fd[1],管道有一个读端fd[0]，一个写端fd[1],这个规定不能变。
    返 回值：成功是0，出错是-1。
    ```

    

#### 3.2 信号

-   函数 void (\*signal(int sig, void (\*handler)(int))) (int);

    void (\*handler)(int) 是个函数指针  

    信号处理函数一般具有： void handler(int sig){}; sig表示信号值

-   exit(0) 发出的信号是 SIGCHILD的信号；

-   wait(NULL) 父进程回收子进程，可以使用signal信号+函数来处理，不然会一直阻塞；

-   sigaction的使用比signal更具有移植性；

```c
#include "dong.h"

void func(int signum)
{
    printf("recv signal:[%d]\n", signum);
    return ;
}

void func1(int signum)
{
    printf("exit sign:[%d]\n", signum);
    wait(NULL);

    return ;
}

int main()
{
    int i = 0;

    pid_t pid;

    signal(SIGUSR1, func); /*处理kill信号*/
    signal(SIGCHLD, func1); /*处理exit(0)的信号*/

    pid = fork();
    if (pid < 0)
    {
        printf("fork error.\n");
        return -1;
    }

    if (pid > 0)
    {
        while (1)
        {
            printf("parent process thing, i:[%d]\n", i);
            sleep(1);
            i++;
        }
        
    }else
    {
        sleep(10);
        kill(getppid(),0);
        exit(0);
    }
    
    return 0;
}
```



#### 3.3 共享内存

-   ipcs 查看创建的IPC对象：

    ipcs -m 查看创建的内存   ipcs -q 查看创建的队列  ipcs -s 查看创建的信号灯

-   函数 key_t ftok(const char *pathname, int proj_id); 

    返回值：key 参数：路径，字符。 

-   函数 int shmget(key_t key, size_t size, int shmflg); 

    返回值：shmid 参数：ftok返回的key，或者 IPC_PRIVATE 只能用于有亲缘关系；共享内存的大小；shmflag表示权限 IPC_CREAT|IPC_EXCL|0666;

    shmget创建共享内存成功后，再次创建会报错 EEXIST;

```c
#include "dong.h"

int main()
{
    int shmid;
    char cmd[128] = {0};

    key_t key;
    key = ftok(".", 'x');
    if (key < 0)
    {
        printf("ftok error. msg:[%s]\n", strerror(errno));
        return -1;
    }

    shmid = shmget(key, 128, IPC_CREAT|IPC_EXCL|0666);
    if (errno == EEXIST)
    {
        shmid = shmget(key, 0, 0); /*重复了，需要重新获取id号*/
    }

    if (shmid < 0)
    {
        printf("shmget error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    printf("shmid:[%d]\n", shmid);
    system("ipcs -m");
    
    
    system("ipcs -m");
    return 0;
}
```

-   函数 int shmdt(const void *shmaddr); 

    shmaddr 映射后的地址，将进程里的地址映射删除 （**就是将用户空间的共享内存地址删除**）；

-   函数 int shmctl(int shmid, int cmd, struct shmid_ds *buf); 删除内核空间的共享内存地址  成功：0 出错-1

    shmid:要操作的共享内存标识符

    cmd: IPC_STAT (获取对象属性)

    ​         IPC_SET(设置对象属性)

    ​         IPC_RMID(删除对象)

    buf:指定IPC_STAT/IPC_SET是用以保存/设置属性
