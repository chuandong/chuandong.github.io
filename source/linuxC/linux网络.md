---
title: linux网络
date: 2022-01-19 09:22:10
description: 网络
tags: linux 网络
---

#### 1. 网络基础知识

##### 1.1 struct sockaddr 和struct sockaddr_in 这两个结构体

*sockaddr结构体的缺陷是：sa_data把目标地址和端口信息混在一起了；*

```
结构体：sockaddr
头文件：<sys/socket.h>
结构体说明：
struct sockaddr{
    sa_family sin_family;  /*地址族*/
    char sa_data[14];  /*14字节，包含套接字中的目标地址和端口信息*/ 
}
```

*sockaddr_in结构体解决了sockaddr的缺陷，把port和addr分开储存在两个变量中；*

```
结构体：sockaddr_in
头文件：<netinet/in.h> <arpa/inet.h>
结构体说明：
struct sockaddr_in{
	sa_family_t    sin_family;  /*地址族*/
	uint16_t       sin_port;    /*16位TCP/UDP端口号*/
	struct in_addr sin_addr;    /*32位IP地址*/
	char           sin_zero[8]; /*不使用*/
}

struct in_addr{
	In_addr_t      s_addr;      /*32位IPv4地址*/
}
```


> 1. sin_port和sin_addr都必须是网络字节序（NBO)，一般可视化的数字都是主机字节序（HBO）；
> 2. htons()作用是将端口号或INADDR_ANY由主机字节序转换为网络字节序的整数值；（host to net）（ ser_addr.sin_port = htons(0) 0表示随端口       ser_addr.sin_addr.s_addr = htons(INADDR_ANY) INADDR_ANY表示任意地址）
> 3. inet_addr()作用是将一个IP字符串转化为一个网络字节序的整数值，用于sockaddr_in.sin_addr.s_addr;（inet_addr("127.0.0.1")）
> 4. inet_ntoa()作用是将一个sin_addr结构体输出成IP字符串（network to ascii）;(printf("%s", inet_ntoa(ser_addr.sin_addr)))

###### 1.1.1  setsockopt 函数

> 1. closesocket（一般不会立即关闭而经历TIME_WAIT的过程）后想继续重用该socket：
>    BOOL bReuseaddr=TRUE;
>    setsockopt(s,SOL_SOCKET ,SO_REUSEADDR,(const char*)&bReuseaddr,sizeof(BOOL));
>
> 2. 如果要已经处于连接状态的soket在调用closesocket后强制关闭，不经历
>    TIME_WAIT的过程：
>    BOOL bDontLinger = FALSE;
>    setsockopt(s,SOL_SOCKET,SO_DONTLINGER,(const char*)&bDontLinger,sizeof(BOOL));
>
> 3. 在send(),recv()过程中有时由于网络状况等原因，发收不能预期进行,而设置收发时限：
>    int nNetTimeout=1000;//1秒
>    //发送时限
>    setsockopt(socket，SOL_S0CKET,SO_SNDTIMEO，(char *)&nNetTimeout,sizeof(int));
>    //接收时限
>    setsockopt(socket，SOL_S0CKET,SO_RCVTIMEO，(char *)&nNetTimeout,sizeof(int));
>
> 4. 在send()的时候，返回的是实际发送出去的字节(同步)或发送到socket缓冲区的字节
>    (异步);系统默认的状态发送和接收一次为8688字节(约为8.5K)；在实际的过程中发送数据
>    和接收数据量比较大，可以设置socket缓冲区，而避免了send(),recv()不断的循环收发：
>    // 接收缓冲区
>    int nRecvBuf=32*1024;//设置为32K
>    setsockopt(s,SOL_SOCKET,SO_RCVBUF,(const char*)&nRecvBuf,sizeof(int));
>    //发送缓冲区
>    int nSendBuf=32*1024;//设置为32K
>    setsockopt(s,SOL_SOCKET,SO_SNDBUF,(const char*)&nSendBuf,sizeof(int));
>
> 5. 如果在发送数据的时，希望不经历由系统缓冲区到socket缓冲区的拷贝而影响
>    程序的性能：
>    int nZero=0;
>    setsockopt(socket，SOL_S0CKET,SO_SNDBUF，(char *)&nZero,sizeof(nZero));
>
> 6. 同上在recv()完成上述功能(默认情况是将socket缓冲区的内容拷贝到系统缓冲区)：
>    int nZero=0;
>    setsockopt(socket，SOL_S0CKET,SO_RCVBUF，(char *)&nZero,sizeof(int));
>
> 7. 一般在发送UDP数据报的时候，希望该socket发送的数据具有广播特性：
>    BOOL bBroadcast=TRUE;
>    setsockopt(s,SOL_SOCKET,SO_BROADCAST,(const char*)&bBroadcast,sizeof(BOOL));
>
> 8. 在client连接服务器过程中，如果处于非阻塞模式下的socket在connect()的过程中可
>    以设置connect()延时,直到accpet()被呼叫(本函数设置只有在非阻塞的过程中有显著的
>    作用，在阻塞的函数调用中作用不大)
>    BOOL bConditionalAccept=TRUE;
>    setsockopt(s,SOL_SOCKET,SO_CONDITIONAL_ACCEPT,(const char*)&bConditionalAccept,sizeof(BOOL));
>
> 9. 如果在发送数据的过程中(send()没有完成，还有数据没发送)而调用了closesocket(),以前我们
>    一般采取的措施是"从容关闭"shutdown(s,SD_BOTH),但是数据是肯定丢失了，如何设置让程序满足具体
>    应用的要求(即让没发完的数据发送出去后在关闭socket)？
>    struct linger {
>    	u_short l_onoff;
>    	u_short l_linger;
>    };
>    linger m_sLinger;
>    m_sLinger.l_onoff=1;//(在closesocket()调用,但是还有数据没发送完毕的时候容许逗留)
>    // 如果m_sLinger.l_onoff=0;则功能和2.)作用相同;
>    m_sLinger.l_linger=5;//(容许逗留的时间为5秒)
>    setsockopt(s,SOL_SOCKET,SO_LINGER,(const char*)&m_sLinger,sizeof(linger));

##### 1.2 服务器端代码-原始版本

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <errno.h>

int main(int argc, char *argv[])
{
    int serfd, clifd;
    int on = 1;
    char buff[1024] = {0};
    
    struct sockaddr_in ser_addr, cli_addr;
    
    if (argc < 2)
    {
        printf("Please input port.");
        return -1;
    }
    
    serfd = socket(AF_INET, SOCK_STREAM, 0);
    if (serfd < 0)
    {
        printf("socket error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    if (setsockopt(serfd, SOL_SOCKET, SO_REUSEADDR, &on, sizeof(int)) < 0)
    {
        printf("socketopt error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    bzero(&ser_addr, sizeof(ser_addr));
    ser_addr.sin_family = AF_INET;
    ser_addr.sin_port = htons(atoi(argv[1]));
    ser_addr.sin_addr.s_addr = htons(INADDR_ANY);
    
    if (bind(serfd, (struct sockaddr *)&ser_addr, sizeof(struct sockaddr)) < 0)
    {
        printf("bind error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    if (listen(serfd, 10) < 0)
    {
        printf("listen error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    int len = sizeof(struct sockaddr);
    clifd  = accept(serfd, (struct sockaddr *)&cli_addr, &len);
    if (clifd < 0)
    {
        printf("accept error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    while (1)
    {
        memset(buff, 0, sizeof(buff));
        len = 0;
        len = read(clifd, buff, sizeof(buff));
        if (len > 0)
        {
            printf("read buff is:[%s]\n", buff);
        }
        else if(len == 0)
        {
            printf("client close. msg:[%s]\n", strerror(errno));
            close(serfd);
            close(clifd);
            return 0;
        }
        else 
        {
            printf("read error. msg:[%s]\n", strerror(errno));
            close(serfd);
            close(clifd);
            return -1;
        }
        
        write(clifd, buff, strlen(buff));
    }
    
    close(serfd);
    close(clifd);
    return 0;
}
```

##### 1.3 客户端代码

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <errno.h>

int main(int argc, char *argv[])
{
    int clifd;
    
    struct sockaddr_in cli_addr;
    
    clifd = socket(AF_INET, SOCK_STREAM, 0);
    if (clifd < 0)
    {
        printf("socket error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    bzero(&cli_addr, sizeof(cli_addr));
    
    cli_addr.sin_family = AF_INET;
    cli_addr.sin_port = htons(atoi(argv[1]));
    cli_addr.sin_addr.s_addr = inet_addr("127.0.0.1");
    
    if (connect(clifd, (struct sockaddr *)&cli_addr, sizeof(struct sockaddr)) < 0)
    {
        printf("connect error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    char buff[1024];
    int len = 0;
    while(1)
    {
        memset(buff, 0, sizeof(buff));
        scanf("%s", buff);
        len = strlen(buff);
        buff[len] = '\0';
        write(clifd, buff, strlen(buff));
        printf("write buff:[%s]\n", buff);
        memset(buff, 0, sizeof(buff));
        read(clifd, buff, sizeof(buff));
        printf("read buff:[%s]\n", buff);
    }
    close(clifd);
    return 0;
}
```

##### 1.4 服务器端代码-多线程版本

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <errno.h>

void *thread_func(void *arg)
{
    char buff[1024] = {0};
    int clifd = *(int *)arg;
    int len = 0;
    
    while (1)
    {
        memset(buff, 0, sizeof(buff));
        len = read(clifd, buff, sizeof(buff));
        if (len > 0)
        {
            printf("read buff is:[%s]\n", buff);
        }
        else if(len == 0)
        {
            printf("client close. msg:[%s]\n", strerror(errno));
            return (void *)0;
        }
        else 
        {
            printf("read error. msg:[%s]\n", strerror(errno));
            return (void *)-1;
        }
        
        write(clifd, buff, strlen(buff));
    }
    return (void *)0;
}

int main(int argc, char *argv[])
{
    int serfd, clifd;
    int on = 1;
    char buff[1024] = {0};
    
    struct sockaddr_in ser_addr, cli_addr;
    
    if (argc < 2)
    {
        printf("Please input port.");
        return -1;
    }
    
    serfd = socket(AF_INET, SOCK_STREAM, 0);
    if (serfd < 0)
    {
        printf("socket error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    if (setsockopt(serfd, SOL_SOCKET, SO_REUSEADDR, &on, sizeof(int)) < 0)
    {
        printf("socketopt error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    bzero(&ser_addr, sizeof(ser_addr));
    ser_addr.sin_family = AF_INET;
    ser_addr.sin_port = htons(atoi(argv[1]));
    ser_addr.sin_addr.s_addr = htons(INADDR_ANY);
    
    if (bind(serfd, (struct sockaddr *)&ser_addr, sizeof(struct sockaddr)) < 0)
    {
        printf("bind error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    if (listen(serfd, 10) < 0)
    {
        printf("listen error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    
    while (1)
    {
        int len = sizeof(struct sockaddr);
        clifd  = accept(serfd, (struct sockaddr *)&cli_addr, &len);
        if (clifd < 0)
        {
            printf("accept error. msg:[%s]\n", strerror(errno));
            return -1;
        }
        
        pthread_t pthid;
        if (pthread_create(&pthid, NULL, &thread_func, (void *)&clifd) != 0)
        {
            printf("pthread_create error. msg:[%s]\n", strerror(errno));
            return -1;
        }
        
    }
    
    return 0;
}
```

##### 1.5 服务器端代码-多进程版本

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>
#include <errno.h>

int do_func(int arg)
{
    char buff[1024] = {0};
    int clifd = arg;
    int len = 0;
    
    while (1)
    {
        memset(buff, 0, sizeof(buff));
        len = read(clifd, buff, sizeof(buff));
        if (len > 0)
        {
            printf("read buff is:[%s]\n", buff);
        }
        else if(len == 0)
        {
            printf("client close. msg:[%s]\n", strerror(errno));
            return 0;
        }
        else 
        {
            printf("read error. msg:[%s]\n", strerror(errno));
            return -1;
        }
        
        write(clifd, buff, strlen(buff));
    }
    return 0;
}

int main(int argc, char *argv[])
{
    int serfd, clifd;
    int on = 1;
    int iRet;
    char buff[1024] = {0};
    
    struct sockaddr_in ser_addr, cli_addr;
    
    if (argc < 2)
    {
        printf("Please input port.");
        return -1;
    }
    
    serfd = socket(AF_INET, SOCK_STREAM, 0);
    if (serfd < 0)
    {
        printf("socket error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    if (setsockopt(serfd, SOL_SOCKET, SO_REUSEADDR, &on, sizeof(int)) < 0)
    {
        printf("socketopt error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    bzero(&ser_addr, sizeof(ser_addr));
    ser_addr.sin_family = AF_INET;
    ser_addr.sin_port = htons(atoi(argv[1]));
    ser_addr.sin_addr.s_addr = htons(INADDR_ANY);
    
    if (bind(serfd, (struct sockaddr *)&ser_addr, sizeof(struct sockaddr)) < 0)
    {
        printf("bind error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    if (listen(serfd, 10) < 0)
    {
        printf("listen error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    
    while (1)
    {
        int len = sizeof(struct sockaddr);
        clifd  = accept(serfd, (struct sockaddr *)&cli_addr, &len);
        if (clifd < 0)
        {
            printf("accept error. msg:[%s]\n", strerror(errno));
            return -1;
        }
        
        pid_t pid;
        int status;
        
        pid = fork();
        if (pid == 0)
        {
            if (do_func(clifd) < 0)
            {
                printf("do_func error.\n");
                return -1;
            }
        }else if (pid > 0)
        {
            //wait(&status); /*wait会阻塞*/
        }
        else
        {
            printf("fork error. msg:[%s]\n", strerror(errno));
            return -1;
        }
        
    }
    
    return 0;
}
```

##### 1.6 服务器端代码-设置非阻塞版

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>
#include <fcntl.h>
#include <errno.h>

int main(int argc, char *argv[])
{
    int serfd, clifd;
    int on = 1;
    char buff[1024] = {0};
    
    struct sockaddr_in ser_addr, cli_addr;
    
    if (argc < 2)
    {
        printf("Please input port.");
        return -1;
    }
    
    serfd = socket(AF_INET, SOCK_STREAM, 0);
    if (serfd < 0)
    {
        printf("socket error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    if (setsockopt(serfd, SOL_SOCKET, SO_REUSEADDR, &on, sizeof(int)) < 0)
    {
        printf("socketopt error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    bzero(&ser_addr, sizeof(ser_addr));
    ser_addr.sin_family = AF_INET;
    ser_addr.sin_port = htons(atoi(argv[1]));
    ser_addr.sin_addr.s_addr = htons(INADDR_ANY);
    
    if (bind(serfd, (struct sockaddr *)&ser_addr, sizeof(struct sockaddr)) < 0)
    {
        printf("bind error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    if (listen(serfd, 10) < 0)
    {
        printf("listen error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    int len = sizeof(struct sockaddr);
    clifd  = accept(serfd, (struct sockaddr *)&cli_addr, &len);
    if (clifd < 0)
    {
        printf("accept error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    int flags = fcntl(clifd, F_GETFL, 0);
    fcntl(clifd, F_SETFL, flags |O_NONBLOCK);  /*将文件描述符设置成非阻塞*/
    
    while (1)
    {
        memset(buff, 0, sizeof(buff));
        len = 0;
        len = read(clifd, buff, sizeof(buff));
        
        
        if (len > 0)
        {
            printf("read buff is:[%s]\n", buff);
        }
        else if(len == 0)
        {
            printf("client close. msg:[%s]\n", strerror(errno));
            close(serfd);
            close(clifd);
            return 0;
        }
        
        if (len < 0 && errno != EAGAIN)
        {
            printf("read error. msg:[%s]\n", strerror(errno));
            return -1;
        }
        
        write(clifd, buff, strlen(buff));
    }
    
    return 0;
}
```

#### 2. select 

##### 2.1 函数说明

```c
#include <sys/select.h>
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>

/*对fd_set的理解：fd_set可以理解为一个集合，那么集合就会有一个数量，在<sys/select.h>总定义了一个常量FD_SETSIZE，默认为1024，也就是说在这个集合内默认最多有1024个文件描述符，但是通常你用不了这么多，你通常只是关心maxfds个描述符。也就是说你现在有maxfds个文件描述符在这个集合里，那么我怎么知道集合里的哪个文件描述符有消息来了呢？你可以将fd_set中的集合看成是二进制bit位，一位代表着一个文件描述符。0代表文件描述符处于睡眠状态，没有数据到来；1代表文件描述符处于准备状态，可以被应用层处理。我觉得select函数可以分下面几步进行理解
1）在你开始监测这些描述符时，你先将这些文件描述符全部置为0；
2）当你需要监测的描述符置为1；
3）使用select函数监听置为1的文件描述符是否有数据到来；
4）当状态为1的文件描述符有数据到来时，此时你的状态仍然为1，但是其他状态为1的文件描述如果没有数据到来，那么此时会将这些文件描述符置为0；
5）当select函数返回后，可能有一个或者多个文件描述符为1，那么你怎么知道是哪个文件描述符准备好了呢？其实select并不会告诉你说，我哪个文件描述符准备好了，他只会告诉你他的那些bit为位哪些是0，哪些是1。也就是说你需要自己用逻辑去判断你要的那个文件描是否准备好了；
*/
typedef struct fd_set
{
	u_int fd_count;
    int fd_array[FD_SETSIZE];
}

/*用来清除集合里面的某个文件描述符*/
void FD_CLR(int fd, fd_set *set);

/*用来检测指定的某个描述符是否有数据到来。- 那么假如在我们的程序中有5个客户端已经连接上了服务器，这个时候突然有一条数据过来了。select返回了，但是此时你并不知道是哪个客户发过来的消息，因为你每个客户发过来的消息都是一样重要的。所以你没法去只针对一个套接字使用FD_ISSET，你需要做的是用一个循环去检测（FD_ISSET）到底是哪一个客户发过来的消息，因为如果此时你监测一个套接字的话，其他客户的信息你会丢失。这个也是select的一个缺点，你需要去检测所有的套接字，看看这个套接字到底是谁来的数据。*/
int  FD_ISSET(int fd, fd_set *set);

/*用于在文件描述符集合中增加一个新的文件描述符，将相应的位置置为1*/
void FD_SET(int fd, fd_set *set);

/*将指定集合里面所有的描述符全部置为0，在对文件描述符集合进行设置前，必须对其进行初始化，如果不清空，由于在系统分配内存空间后，通常并不作清空处理，所以结果是不可知的*/
void FD_ZERO(fd_set *set);



int select(int maxfds, fd_set *readfds, fd_set *writefds,
                  fd_set *exceptfds, struct timeval *timeout);
struct timeval {
    long    tv_sec;         /* seconds */
    long    tv_usec;        /* microseconds */
};

/*1.maxfds：是一个整数值，是指集合中所有文件描述符的范围，即所有文件描述符的最大值加1，不能错。在linux系统中，select的默认最大值为1024。设置这个值的目的是为了不用每次都去轮询这1024个fd，假设我们只需要几个套接字，我们就可以用最大的那个套接字的值加上1作为这个参数的值，当我们在等待是否有套接字准备就绪时，只需要监测maxfd+1个套接字就可以了，这样可以减少轮询时间以及系统的开销。
2.readfds：首先需要明白，fd_set是什么数据类型，有一点像int，又有点像struct，其实，fd_set声明的是一个集合，也就是说，readfs是一个容器，里面可以容纳多个文件描述符，把需要监视的描述符放入这个集合中，当有文件描述符可读时，select就会返回一个大于0的值，表示有文件可读。
3.writefds：和readfs类似，表示有一个可写的文件描述符集合，当有文件可写时，select就会返回一个大于0的值，表示有文件可写。
4.exceptfds 用来监视文件错误异常文件。
5.timeout：这个参数一出来就可以知道，可以选择阻塞，可以选择非阻塞，还可以选择定时返回。当将timeout置为NULL时，表明此时select是阻塞的；当将tineout设置为timeout->tv_sec = 0，timeout->tv_usec = 0时，表明这个函数为非阻塞；当将timeout设置为非0的时间，表明select有超时时间，当这个时间走完，select函数就会返回。从这个角度看，个人觉得可以用select来做超时处理，因为你如果使用recv函数的话，你还需要去设置recv的模式，麻烦的很。
*/
/*select的使用流程
	1）先调用宏FD_ZERO将指定的fd_set清零；
	2）然后调用宏FD_SET将需要测试的fd加入fd_set；
	3）接着调用函数select监测fd_set中的所有fd；
	4）最后用宏FD_ISSET检查某个fd在函数select调用后，相应位是否仍然为1，然后做相应的逻辑处理。
*/

/*select的缺点:
1）每次调用 select ，都需要把 fd 集合从用户态拷贝到内核态，这个开销在 fd 很多时会很大。
2）同时每次调用 select 都需要在内核遍历传递进来的所有 fd ，这个开销在 fd 很多时也很大。
3）每次在 select() 函数返回后，都要通过遍历文件描述符来获取已经就绪的 socket 。
4）select 支持的文件描述符数量太小了，默认是 1024 。
*/

/*当使用select等待客户端发送数据时，如果客户端断开了连接，无论是主动close还是程序挂掉了，这时候select都会返回1，而且通过FD_ISSET可以看到是与该客户端相连的socket触发产生的，如果此时服务端仍然调用read读取信息，会返回0。而且如果这时候不处理该socket，select会不断的返回，并且read始终返回0。所以通过select应该无法判断触发返回的原因是有数据还是对方断开了，需要调用read函数，可以判断原因。
*/
```



##### 2.2 select带外数据处理

> 1.  传输层的TCP协议有带外数据的概念，带外数据又称为紧急数据，它比普通数据有更高的优先级，一般会立即发送，而不会排队等待。
> 2. 在TCP协议头部结构中有URG标志位和16位的紧急指针，若URG标志位被设置，表示紧急指针有效，此时紧急指针将指向紧急数据的下一个字节。
> 3. 带外数据只有一个字节大小，因为服务器将读取到的带外数据存入一个特殊的缓冲区，这个缓冲区只有一个字节的大小，并且带外数据会将TCP字节流截断，可以用带MSG_OOB标志的recv系统调用来读取带外数据。
> 4. 在网络编程中，select能监听可读，可写和异常事件，但能监听的异常事件只有一种，那就是socket上接收到带外数据。socket上接收到普通数据和带外数据都将使select返回，但前者处于可读状态，后者处于异常状态。



```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <errno.h>

int main(int argc, char *argv[])
{
    int serfd, clifd;
    int on = 1;
    char buff[1024] = {0};
    
    struct sockaddr_in ser_addr, cli_addr;
    
    if (argc < 2)
    {
        printf("Please input port.");
        return -1;
    }
    
    serfd = socket(AF_INET, SOCK_STREAM, 0);
    if (serfd < 0)
    {
        printf("socket error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    if (setsockopt(serfd, SOL_SOCKET, SO_REUSEADDR, &on, sizeof(int)) < 0)
    {
        printf("socketopt error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    bzero(&ser_addr, sizeof(ser_addr));
    ser_addr.sin_family = AF_INET;
    ser_addr.sin_port = htons(atoi(argv[1]));
    ser_addr.sin_addr.s_addr = htons(INADDR_ANY);
    
    if (bind(serfd, (struct sockaddr *)&ser_addr, sizeof(struct sockaddr)) < 0)
    {
        printf("bind error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    if (listen(serfd, 10) < 0)
    {
        printf("listen error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    int len = sizeof(struct sockaddr);
    clifd  = accept(serfd, (struct sockaddr *)&cli_addr, &len);
    if (clifd < 0)
    {
        printf("accept error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    fd_set rfd, exceptfd;
    FD_ZERO(&rfd);
    FD_ZERO(&exceptfd);
    int num;
    
    while (1)
    {
        FD_SET(clifd, &rfd);
        num = select(clifd+1, &rfd, NULL, &exceptfd, NULL);
        if (num == -1)
        {
            printf("select error. msg:[%s]\n", strerror(errno));
            return -1;
        }
        
        if (FD_ISSET(clifd, &rfd))
        {
            memset(buff, 0, sizeof(buff));
            len = 0;
            len = read(clifd, buff, sizeof(buff));  /*处理普通数据*/
            if (len > 0)
            {
                printf("read buff is:[%s]\n", buff);
            }
            else if(len == 0)
            {
                printf("client close. msg:[%s]\n", strerror(errno));
                close(serfd);
                close(clifd);
                return 0;
            }
            else 
            {
                printf("read error. msg:[%s]\n", strerror(errno));
                return -1;
            }
            
            write(clifd, buff, strlen(buff));
        }
        
        if (FD_ISSET(clifd, &exceptfd))
        {
            memset(buff, 0, sizeof(buff));
            len = 0;
            /*处理外带数据需要用MSG_OOB*/
            len = recv(clifd, buff, sizeof(buff), MSG_OOB);
            if (len < 0)
            {
                printf("recv error. msg:[%s]\n", strerror(errno));
                return -1;
            }
            
            printf("接收的外带数据为:[%s]\n", buff);
        }
    }
    
    close(serfd);
    close(clifd);
    
    return 0;
}
```

##### 2.3  select处理多个客户端

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <errno.h>

int main(int argc, char *argv[])
{
    int serfd, clifd;
    int on = 1;
    char buff[1024] = {0};
    
    struct sockaddr_in ser_addr, cli_addr;
    
    if (argc < 2)
    {
        printf("Please input port.");
        return -1;
    }
    
    serfd = socket(AF_INET, SOCK_STREAM, 0);
    if (serfd < 0)
    {
        printf("socket error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    if (setsockopt(serfd, SOL_SOCKET, SO_REUSEADDR, &on, sizeof(int)) < 0)
    {
        printf("socketopt error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    bzero(&ser_addr, sizeof(ser_addr));
    ser_addr.sin_family = AF_INET;
    ser_addr.sin_port = htons(atoi(argv[1]));
    ser_addr.sin_addr.s_addr = htons(INADDR_ANY);
    
    if (bind(serfd, (struct sockaddr *)&ser_addr, sizeof(struct sockaddr)) < 0)
    {
        printf("bind error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    if (listen(serfd, 10) < 0)
    {
        printf("listen error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    int num, len;
    int maxfd = serfd;
    fd_set allfd;
    fd_set rfd;
    FD_ZERO(&allfd);
    FD_ZERO(&rfd);
    FD_SET(serfd, &rfd);
    
    while (1)
    {
        allfd = rfd;   /*保存原来的BIT位*/
        num = select(maxfd+1, &allfd, NULL, NULL, NULL);
        if (num == -1)
        {
            printf("select error. msg:[%s]\n", strerror(errno));
            return -1;
        }
        
        printf("select num:[%d]\n", num);
        
        if (FD_ISSET(serfd, &allfd))
        {
            len = sizeof(struct sockaddr);
            clifd  = accept(serfd, (struct sockaddr *)&cli_addr, &len);
            if (errno == EAGAIN)
            {
                continue;
            }
            if (clifd < 0)
            {
                printf("accept error. msg:[%s]\n", strerror(errno));
                return -1;
            }
            FD_SET(clifd, &rfd);
            maxfd = clifd > maxfd ? clifd : maxfd;
        }
        
        printf("maxfd:[%d], serfd:[%d], clifd:[%d]\n", maxfd, serfd, clifd);
        int i;
        for (i = 0; i < maxfd+1; ++i)   /*需要循环遍历哪个描述符BIT位置成1了,处理此数据*/
        {
            if (i != serfd && FD_ISSET(i, &allfd))
            {
                memset(buff, 0, sizeof(buff));
                len = 0;
                len = read(i, buff, sizeof(buff));  /*处理普通数据*/
                if (len > 0)
                {
                    printf("read buff is:[%s]\n", buff);
                }
                else if(len == 0)
                {
                    printf("client close. msg:[%s]\n", strerror(errno));
                    FD_CLR(i, &rfd);
                    close(serfd);
                    close(clifd);
                    return 0;
                }
                else 
                {
                    printf("read error. msg:[%s]\n", strerror(errno));
                    FD_CLR(i, &rfd);
                    return -1;
                }
                
                write(i, buff, strlen(buff)+1);
            }
        }
    }
    
    close(serfd);
    close(clifd);
    
    return 0;
}
```



#### 3. poll

##### 3.1 函数说明

```
#include <poll.h>

函数：int poll(struct pollfd *fds, nfds_t nfds, int timeout);
说明：
struct pollfd {
	int   fd;         /* file descriptor */  /* 需要被检测或选择的文件描述符*/
    short events;     /* requested events */ /* 对文件描述符fd上感兴趣的事件 */
    short revents;    /* returned events */  /* 文件描述符fd上当前实际发生的事件*/
};
1. fds：是一个struct pollfd结构类型的数组，用于存放需要检测其状态的Socket描述符；每当调用这个函数之后，系统不会清空这个数组，操作起来比较方便；特别是对于 socket连接比较多的情况下，在一定程度上可以提高处理的效率；这一点与select()函数不同，调用select()函数之后，select() 函数会清空它所检测的socket描述符集合，导致每次调用select()之前都必须把socket描述符重新加入到待检测的集合中；因 此，select()函数适合于只检测一个socket描述符的情况，而poll()函数适合于大量socket描述符的情况；
2. nfds：nfds_t类型的参数，用于标记数组fds中的结构体元素的总数量；
3. timeout：是poll函数调用阻塞的时间，单位：毫秒；
返回值说明：
1. >0：数组fds中准备好读、写或出错状态的那些socket描述符的总数量；

2. ==0：数组fds中没有任何socket描述符准备好读、写，或出错；此时poll超时，超时时间是timeout毫秒；换句话说，如果所检测的 socket描述符上没有任何事件发生的话，那么poll()函数会阻塞timeout所指定的毫秒时间长度之后返回，如果timeout==0，那么 poll() 函数立即返回而不阻塞，如果timeout==INFTIM，那么poll() 函数会一直阻塞下去，直到所检测的socket描述符上的感兴趣的事件发 生是才返回，如果感兴趣的事件永远不发生，那么poll()就会永远阻塞下去；
3. -1：  poll函数调用失败，同时会自动设置全局变量errno；
```

> 1. 文件描述符可以修改，可以使用命令查看一个进程可以打开的socket描述符上限：
>
>    - cat /proc/sys/fs/file-max
>
>    - 如有需要，可以修改配置文件，修改文件描述符的上线
>
>    - vi /etc/security/limits.conf  在文件尾部添加
>
>      \* soft nofile 65536
>
>      \* hard nofile 65536
>
> 2. `fd`是文件描述符，`events`是用户感兴趣的事件（类似于`select`中的`readfds`, `writefds`和`exceptfds`），由用户填写；`revents`是实际发生的事件，由内核填写。
>
> 3. `events`与`revents`是位掩码，其可以包含的标志位有
>
>    - `POLLIN`：存在数据可以读入（相当于`select`中的`readfds`）
>
>    - `POLLPRI`：存在其他条件满足（相当于`select`中的`exceptfds`）
>
>    - `POLLOUT`：存在数据可以写入（相当于`select`中的`writefds`）
>
>    - `POLLRDHUP`
>
>      流套接字对端关闭连接。
>
>      需定义`_GNU_SOURCE`宏。
>
>    - `POLLERR`
>
>      出错。
>
>      只可由`revents`包含，不可由`events`包含
>
>    - `POLLHUP`
>
>      挂起。
>
>      只可由`revents`包含，不可由`events`包含
>
>    - `POLLNVAL`
>
>      由于`fd`未打开，请求无效。

##### 3.2 poll处理多个客户端

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <poll.h>
#include <errno.h>

#define MAXFD 1024

int main(int argc, char *argv[])
{
    int serfd, clifd;
    int on = 1;
    int i = 0, j = 0;
    char buff[1024] = {0};
    
    struct sockaddr_in ser_addr, cli_addr;
    struct pollfd poll_fd[MAXFD];
    
    if (argc < 2)
    {
        printf("Please input port.");
        return -1;
    }
    
    serfd = socket(AF_INET, SOCK_STREAM, 0);
    if (serfd < 0)
    {
        printf("socket error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    if (setsockopt(serfd, SOL_SOCKET, SO_REUSEADDR, &on, sizeof(int)) < 0)
    {
        printf("socketopt error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    bzero(&ser_addr, sizeof(ser_addr));
    ser_addr.sin_family = AF_INET;
    ser_addr.sin_port = htons(atoi(argv[1]));
    ser_addr.sin_addr.s_addr = htons(INADDR_ANY);
    
    if (bind(serfd, (struct sockaddr *)&ser_addr, sizeof(struct sockaddr)) < 0)
    {
        printf("bind error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    if (listen(serfd, 10) < 0)
    {
        printf("listen error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    poll_fd[0].fd = serfd;
    poll_fd[0].events = POLLIN;
    
    for (j = 1; j < MAXFD; j++)
    {
        poll_fd[j].fd = -1;
    }
    
    int maxfd = 0;
    int num = 0;
    
    while (1)
    {
        num = poll(poll_fd, maxfd+1, -1);
        if (num < 0)
        {
            printf("poll error. msg:[%s]\n", strerror(errno));
            return -1;
        }
        
        if (poll_fd[0].revents & POLLIN)
        {
            int len = sizeof(struct sockaddr);
            clifd  = accept(serfd, (struct sockaddr *)&cli_addr, &len);
            if (clifd < 0)
            {
                printf("accept error. msg:[%s]\n", strerror(errno));
                return -1;
            }
            
            printf("clifd:[%d]\n", clifd);
            
            for (i = 1; i < MAXFD; i++)
            {
                if (poll_fd[i].fd < 0)
                {
                    poll_fd[i].fd = clifd;  /*将连接描述符加入到队列里*/
                    poll_fd[i].events = POLLIN;  /*设置事件为可读*/
                    break;
                }
                
            }
            
            if (clifd > maxfd)
                maxfd = clifd;
                
            if (--num <0)
                continue;
        }
        
        for (i = 1; i <= maxfd; i++)
        {
            if (poll_fd[i].fd == -1)
                continue;
            
            if (poll_fd[i].revents & POLLIN)
            {
                memset(buff, 0, sizeof(buff));
                int len = 0;
                len = read(poll_fd[i].fd, buff, sizeof(buff));
                if (len > 0)
                {
                    printf("read buff is:[%s]\n", buff);
                }
                else if(len == 0)
                {
                    printf("client close. msg:[%s]\n", strerror(errno));
                    poll_fd[i].fd = -1;
                    close(i);
                    return 0;
                }
                else 
                {
                    printf("read error. msg:[%s]\n", strerror(errno));
                    return -1;
                }
                
                write(poll_fd[i].fd, buff, strlen(buff));
            }
        }
        
    }
    
    close(serfd);
    close(clifd);
    
    return 0;
}
```

#### 4. epoll

##### 4.1 函数说明

```c
int epoll_create(int size)；
/*创建一个epoll的句柄，size用来告诉内核这个监听的数目一共有多大，这个参数不同于select()中的第一个参数，给出最大监听的fd+1的值，参数size并不是限制了epoll所能监听的描述符最大个数，只是对内核初始分配内部数据结构的一个建议。
当创建好epoll句柄后，它就会占用一个fd值，在linux下如果查看/proc/进程id/fd/，是能够看到这个fd的，所以在使用完epoll后，必须调用close()关闭，否则可能导致fd被耗尽。
*/
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)；
/*函数是对指定描述符fd执行op操作。
- epfd：是epoll_create()的返回值。
- op：表示op操作，用三个宏来表示：添加EPOLL_CTL_ADD，删除EPOLL_CTL_DEL，修改EPOLL_CTL_MOD。分别添加、删除和修改对fd的监听事件。
- fd：是需要监听的fd（文件描述符）
- epoll_event：是告诉内核需要监听什么事，struct epoll_event结构如下：
*/
struct epoll_event {
  __uint32_t events;  /* Epoll events */
  epoll_data_t data;  /* User data variable */
};
/*events可以是以下几个宏的集合：
EPOLLIN ：表示对应的文件描述符可以读（包括对端SOCKET正常关闭）；
EPOLLOUT：表示对应的文件描述符可以写；
EPOLLPRI：表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）；
EPOLLERR：表示对应的文件描述符发生错误；
EPOLLHUP：表示对应的文件描述符被挂断；
EPOLLET： 将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的。
EPOLLONESHOT：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里
*/
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
/*等待epfd上的io事件，最多返回maxevents个事件。
参数events用来从内核得到事件的集合，maxevents告之内核这个events有多大，这个maxevents的值不能大于创建epoll_create()时的size，参数timeout是超时时间（毫秒，0会立即返回，-1将不确定，也有说法说是永久阻塞）。该函数返回需要处理的事件数目，如返回0表示已超时。
*/
```

##### 4.2 epoll水平触发与边缘触发的代码--最简便

```c
#include <stdio.h>  
#include <stdlib.h>  
#include <string.h>
#include <unistd.h>  
#include <fcntl.h>  
#include <errno.h>
#include <unistd.h>
#include <sys/epoll.h>

#define MAXSIZE 10

int main(int argc, char *argv[])
{
    struct epoll_event eve;
    struct epoll_event rev[MAXSIZE];
    
    int efd = epoll_create(MAXSIZE);
    
    eve.data.fd = 0;   /*监听读事件*/
    //eve.events = EPOLLIN;  /*水平触发的时候, 根据输入，epoll times是一直打印，停不下来。。。。*/
    eve.events = EPOLLIN|EPOLLET; /*边缘触发的时候，根据输入一次，打印一次*/
    
    epoll_ctl(efd, EPOLL_CTL_ADD, 0, &eve);
    
    int num;
    int i;
    
    while(1)
    {
        num = epoll_wait(efd, rev, MAXSIZE, -1);
        
        
        for (i = 0; i < num; i++)
        {
            if (rev[i].data.fd == 0)
            {
                printf("epoll times:[%d]\n", i);
            }
        }
    }
    close(efd);
    
    return 0;
}
```

##### 4.3 epoll水平触发与边缘触发的代码--管道实现父子进程

```c
#include <stdio.h>  
#include <stdlib.h>  
#include <string.h>
#include <unistd.h>  
#include <fcntl.h>  
#include <errno.h>
#include <unistd.h>
#include <sys/epoll.h>


int main(int argc, char *argv[])
{
    int fd[2];
    char buf[1024] = {0};
    char rbuf[1024] = {0};
    
    pid_t pid;
    
    pipe(fd);
    
    pid = fork();
    
    /*子进程写，父进程读*/
    if (pid == 0)
    {
        close(fd[0]);
        
        int len = 0;
        while (1)
        {
            fgets(buf, sizeof(buf), stdin);
            len = strlen(buf);
            buf[len-1] = '\0';
            write(fd[1], buf, strlen(buf));
            printf("write buf:[%s]\n", buf);
        }
        close(fd[1]);
    }else if (pid > 0)
    {
        close(fd[1]);
        printf("parent process.\n");
        /*使用epoll*/
        int epollfd = epoll_create(10);
        struct epoll_event eve;
        struct epoll_event rev[10];
        eve.data.fd = fd[0];
        eve.events = EPOLLIN;  /*水平触发的时候, 根据子进程的输入，父进程的read buf是一直打印，停不下来。。。。*/
        /*eve.events = EPOLLIN|EPOLLET;*/ /*边缘触发的时候，根据子进程的输入一次，父进程打印一次*/
        if (epoll_ctl(epollfd, EPOLL_CTL_ADD, fd[0], &eve) < 0)
        {
            printf("epoll_ctl error. msg:[%s]\n", strerror(errno));
            return -1;
        }
        
        int num = 0, i = 0;
        while (1)
        {
            memset(rbuf, 0, sizeof(rbuf));
            num = epoll_wait(epollfd, rev, 10, -1);
            if (rev[0].data.fd == fd[0])
            {
                /*read(rev[0].data.fd, rbuf, sizeof(rbuf));*/ /*read阻塞 水平触发跟边缘触发是一样的效果*/
                printf("read buf:[%s]\n", rbuf);
            }
        }
        
    }else 
    {
        printf("fork error, msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    close(epollfd);
    
    return 0;
}
```

##### 4.4 epoll网络编程代码--默认水平触发

```c
#include <stdio.h>  
#include <stdlib.h>  
#include <string.h>
#include <unistd.h>  
#include <fcntl.h>  
#include <errno.h>  
#include <netinet/in.h>       /*socket address struct*/  
#include <arpa/inet.h>            /*host to network convertion*/  
#include <sys/socket.h>  
#include <sys/types.h>  
#include <signal.h>  
#include <sys/ioctl.h>  
#include <sys/epoll.h>
#include <sys/time.h>

#define MAXSIZE 2048
#define EPOLLEVENTS 100

//添加事件
static void add_event(int epollfd,int fd,int state)
{
    struct epoll_event ev;
    ev.events = state;
    ev.data.fd = fd;
    epoll_ctl(epollfd,EPOLL_CTL_ADD,fd,&ev);
}

//删除事件
static void delete_event(int epollfd,int fd,int state) 
{
    struct epoll_event ev;
    ev.events = state;
    ev.data.fd = fd;
    epoll_ctl(epollfd,EPOLL_CTL_DEL,fd,&ev);
}

//修改事件
static void modify_event(int epollfd,int fd,int state)
{     
    struct epoll_event ev;
    ev.events = state;
    ev.data.fd = fd;
    epoll_ctl(epollfd,EPOLL_CTL_MOD,fd,&ev);
}

//处理接收到的连接
static void handle_accpet(int epollfd,int listenfd)
{
    int clifd;     
    struct sockaddr_in cliaddr;     
    socklen_t  cliaddrlen;     
    clifd = accept(listenfd,(struct sockaddr*)&cliaddr,&cliaddrlen);     
    if (clifd == -1)         
       perror("accpet error:");     
    else 
    {         
        printf("accept a new client: %s:%d\n",inet_ntoa(cliaddr.sin_addr),cliaddr.sin_port);                       //添加一个客户描述符和事件         
        add_event(epollfd,clifd,EPOLLIN);     
    } 
}

//读处理
static void do_read(int epollfd,int fd,char *buf)
{
    int nread;
    nread = recv(fd,buf,MAXSIZE, 0);
    if (nread == -1)     
    {         
        perror("read error:");         
        close(fd); //记住close fd        
        delete_event(epollfd,fd,EPOLLIN); //删除监听 
    }
    else if (nread == 0)     
    {         
        fprintf(stderr,"client close.\n");
        close(fd); //记住close fd       
        delete_event(epollfd,fd,EPOLLIN); //删除监听 
    }     
    else {         
        printf("read message is : %s",buf);        
        //修改描述符对应的事件，由读改为写         
        modify_event(epollfd,fd,EPOLLOUT); 
        //send(fd, buf, strlen(buf), 0);    
    } 
}

//写处理
static void do_write(int epollfd,int fd,char *buf) 
{     
    int nwrite;   
    char wbuff[1024] = {0};
      
    printf("Please input send client:\n");
    fgets(wbuff, sizeof(wbuff), stdin);
    nwrite = send(fd,wbuff,strlen(wbuff), 0);     
    if (nwrite == -1){         
        perror("write error:");        
        close(fd);   //记住close fd       
        delete_event(epollfd,fd,EPOLLOUT);  //删除监听    
    }else{
        modify_event(epollfd,fd,EPOLLIN); 
    }    
    memset(buf,0,MAXSIZE); 
}

//事件处理函数
static void handle_events(int epollfd,struct epoll_event *events,int num,int listenfd,char *buf)
{
     int i;
     int fd;
     //进行遍历;这里只要遍历已经准备好的io事件。num并不是当初epoll_create时的FDSIZE。
     for (i = 0;i < num;i++)
     {
         fd = events[i].data.fd;
        //根据描述符的类型和事件类型进行处理
         if ((fd == listenfd) && (events[i].events & EPOLLIN))
            handle_accpet(epollfd,listenfd);
         else if (events[i].events & EPOLLIN)
            do_read(epollfd,fd,buf);
         else if (events[i].events & EPOLLOUT)
            do_write(epollfd,fd,buf);
     }
}

int main()
{
    int iRet = 0;
    int yes = 1;
    int ser_fd;
    struct sockaddr_in addr;
    struct sockaddr_in cli_addr;

    memset(&addr,0,sizeof(addr));
    memset(&cli_addr,0,sizeof(cli_addr));
    addr.sin_family =  AF_INET;
    addr.sin_addr.s_addr = INADDR_ANY;
    addr.sin_port = htons(58888);

    ser_fd = socket(AF_INET,SOCK_STREAM,0);
    if(ser_fd == -1)
    {
        perror("Create socket failed");
        exit(-1);
    }

    iRet = setsockopt(ser_fd, SOL_SOCKET, SO_REUSEADDR, &yes, sizeof(int));
    if (iRet)
    {
        perror("setsockopt error.");
        return -1;
    }

    int len = sizeof(addr);
    
    iRet = bind(ser_fd,(struct sockaddr *)&addr,sizeof(addr));
    if(-1 == iRet)
    {
        perror("Bind socket failed");
        exit(-1);
    }

    listen(ser_fd, 5);
    printf("listen success.\n");

    int epollfd;
    char buf[2048] = {0};
    struct epoll_event events[EPOLLEVENTS];

    epollfd = epoll_create(10);

    //添加监听描述符事件
    add_event(epollfd,ser_fd,EPOLLIN);
    int i = 0;
    //循环等待
    for ( ; ; ){
        i++;
        //该函数返回已经准备好的描述符事件数目
        printf("epoll wait. [%d]次\n", i);
        iRet = epoll_wait(epollfd,events,EPOLLEVENTS,-1);
        //处理接收到的连接
        printf("handle_events. [%d]次\n", i);
        handle_events(epollfd,events,iRet,ser_fd,buf);
    }

    return 0;
}
```

