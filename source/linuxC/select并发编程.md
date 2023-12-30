---
title: select并发编程
date: 2022-02-23 14:27:30
description: linux select
tags: select 编程
---

### 1. 函数说明

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

### 2. setsockopt 函数

```c
/*
1.closesocket（一般不会立即关闭而经历TIME_WAIT的过程）后想继续重用该socket：
BOOL bReuseaddr=TRUE;
setsockopt(s,SOL_SOCKET ,SO_REUSEADDR,(const char*)&bReuseaddr,sizeof(BOOL));

2. 如果要已经处于连接状态的soket在调用closesocket后强制关闭，不经历
   TIME_WAIT的过程：
   BOOL bDontLinger = FALSE;
   setsockopt(s,SOL_SOCKET,SO_DONTLINGER,(const char*)&bDontLinger,sizeof(BOOL));

3.在send(),recv()过程中有时由于网络状况等原因，发收不能预期进行,而设置收发时限：
int nNetTimeout=1000;//1秒
//发送时限
setsockopt(socket，SOL_S0CKET,SO_SNDTIMEO，(char *)&nNetTimeout,sizeof(int));
//接收时限
setsockopt(socket，SOL_S0CKET,SO_RCVTIMEO，(char *)&nNetTimeout,sizeof(int));

4.在send()的时候，返回的是实际发送出去的字节(同步)或发送到socket缓冲区的字节
(异步);系统默认的状态发送和接收一次为8688字节(约为8.5K)；在实际的过程中发送数据
和接收数据量比较大，可以设置socket缓冲区，而避免了send(),recv()不断的循环收发：
// 接收缓冲区
int nRecvBuf=32*1024;//设置为32K
setsockopt(s,SOL_SOCKET,SO_RCVBUF,(const char*)&nRecvBuf,sizeof(int));
//发送缓冲区
int nSendBuf=32*1024;//设置为32K
setsockopt(s,SOL_SOCKET,SO_SNDBUF,(const char*)&nSendBuf,sizeof(int));

5. 如果在发送数据的时，希望不经历由系统缓冲区到socket缓冲区的拷贝而影响
   程序的性能：
   int nZero=0;
   setsockopt(socket，SOL_S0CKET,SO_SNDBUF，(char *)&nZero,sizeof(nZero));

6.同上在recv()完成上述功能(默认情况是将socket缓冲区的内容拷贝到系统缓冲区)：
int nZero=0;
setsockopt(socket，SOL_S0CKET,SO_RCVBUF，(char *)&nZero,sizeof(int));

7.一般在发送UDP数据报的时候，希望该socket发送的数据具有广播特性：
BOOL bBroadcast=TRUE;
setsockopt(s,SOL_SOCKET,SO_BROADCAST,(const char*)&bBroadcast,sizeof(BOOL));

8.在client连接服务器过程中，如果处于非阻塞模式下的socket在connect()的过程中可
以设置connect()延时,直到accpet()被呼叫(本函数设置只有在非阻塞的过程中有显著的
作用，在阻塞的函数调用中作用不大)
BOOL bConditionalAccept=TRUE;
setsockopt(s,SOL_SOCKET,SO_CONDITIONAL_ACCEPT,(const char*)&bConditionalAccept,sizeof(BOOL));

9.如果在发送数据的过程中(send()没有完成，还有数据没发送)而调用了closesocket(),以前我们
一般采取的措施是"从容关闭"shutdown(s,SD_BOTH),但是数据是肯定丢失了，如何设置让程序满足具体
应用的要求(即让没发完的数据发送出去后在关闭socket)？
struct linger {
	u_short l_onoff;
	u_short l_linger;
};
linger m_sLinger;
m_sLinger.l_onoff=1;//(在closesocket()调用,但是还有数据没发送完毕的时候容许逗留)
// 如果m_sLinger.l_onoff=0;则功能和2.)作用相同;
m_sLinger.l_linger=5;//(容许逗留的时间为5秒)
setsockopt(s,SOL_SOCKET,SO_LINGER,(const char*)&m_sLinger,sizeof(linger));
*/
```

### 3. 服务端代码

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
#include <sys/select.h>
#include <sys/time.h>

static int g_var = 0;
static int ser_fd = 0;
static int cli_fd= 0 ;

int main()
{
    int iRet = 0;
    int yes = 1;
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

    struct timeval tm;
    fd_set rfd, allfd;
    fd_set wfd;

    tm.tv_sec=5;
    tm.tv_usec = 0;

    
    FD_ZERO(&rfd);
    FD_SET(ser_fd, &rfd);
    
    while (1)
    {
        int lx_fd;
        allfd = rfd;
		
        /*FD_SETSIZE 根据平台的值来的，此平台 1024*/
        iRet = select (FD_SETSIZE, &allfd, NULL, NULL, NULL);
        printf("select iRet:[%d], msg:[%s]\n", iRet, strerror(errno));
        if (iRet < 0)
        {
            perror("select error.");
            return -1;
        }

        /*开始遍历fd_set位*/
        for (lx_fd = 0; lx_fd < FD_SETSIZE; lx_fd++)
        {
            /*有fd_set置1的位*/
            if (FD_ISSET(lx_fd, &allfd))
            {
                if (lx_fd == ser_fd)
                {
                    int c_len = 0;
                    cli_fd = accept(ser_fd, (struct sockaddr *)&cli_addr, &c_len);
                    if (cli_fd < 0)
                    {
                        perror("accept fail");
                        exit(-1);
                    }
                    FD_SET(cli_fd, &rfd); /*将客户端的描述符加入到fd_set集合中*/
                    printf("fd_%d accept addr IP:[%s], port:[%ld]\n", lx_fd, inet_ntoa(cli_addr.sin_addr), ntohs(cli_addr.sin_port));
                }else
                {
                    char rbuff[1024] = {0};
                    int rlen = 0;
                    memset(rbuff, 0, sizeof(rbuff));
                    rlen = read(lx_fd, rbuff, sizeof(rbuff));
                    if (rlen > 0)
                    {
                        rlen = strlen(rbuff);
                        rbuff[rlen-1] = '\0';
                        printf("fd_%d read buf:[%s]\n", lx_fd, rbuff);
                    }else if (rlen == 0)
                    {
                        close(lx_fd);
                        printf("cloe fd_%s\n", lx_fd);
                        /*关闭fd_set位*/
                        FD_CLR(lx_fd, &rfd);
                    }
                }
            }
        }

    }
    close(ser_fd);
    close(cli_fd);

    return 0;
}
```

### 4. 客户端代码

```c
#include <stdio.h>  
#include <stdlib.h>  
#include <string.h>
#include <unistd.h>  
#include <fcntl.h>  
#include <errno.h>  
#include <netinet/in.h>       /*socket address struct*/  
#include <arpa/inet.h>        /*host to network convertion*/  
#include <sys/socket.h>  
#include <signal.h>  
#define MAX_TRANSPORT_LENTH 512  

int main()
{
    struct sockaddr_in addr;
    memset(&addr,0,sizeof(addr));
    addr.sin_family =  AF_INET;
    addr.sin_addr.s_addr = inet_addr("192.168.181.130");
    addr.sin_port = htons(58888);

    int sock;
    sock = socket(AF_INET,SOCK_STREAM,0);
    if(sock == -1)
    {
        perror("Create socket failed");
        exit(-1);
    }

    int ret;
    ret = connect(sock,(struct sockaddr *)&addr,sizeof(addr));
    if(ret == -1)
    {
        perror("Connect socket failed");
        exit(-1);
    }

    char wbuff[1024] = {0};
    while(1)
    {
        memset(wbuff, 0, sizeof(wbuff));
        printf("start input:\n");
        fgets(wbuff, sizeof(wbuff), stdin);
        write(sock, wbuff, sizeof(wbuff));
        sleep(1);
    }

    return 0;
}
```

