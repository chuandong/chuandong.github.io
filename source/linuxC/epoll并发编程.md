---
title: epoll并发编程
date: 2022-03-17 15:37:56
description: epoll
tags: epoll
---

### 1. 函数说明

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

### 2. epoll的工作模式

```
epoll对文件描述符的操作有两种模式：LT（level trigger）和ET（edge trigger）。LT模式是默认模式，LT模式与ET模式的区别如下：
　　LT模式：当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序可以不立即处理该事件。下次调用epoll_wait时，会再次响应应用程序并通知此事件。
　　ET模式：当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序必须立即处理该事件。如果不处理，下次调用epoll_wait时，不会再次响应应用程序并通知此事件。

1. LT模式
LT(level triggered)是缺省的工作方式，并且同时支持block和no-block socket.在这种做法中，内核告诉你一个文件描述符是否就绪了，然后你可以对这个就绪的fd进行IO操作。如果你不作任何操作，内核还是会继续通知你的。

2. ET模式
ET(edge-triggered)是高速工作方式，只支持no-block socket。在这种模式下，当描述符从未就绪变为就绪时，内核通过epoll告诉你。然后它会假设你知道文件描述符已经就绪，并且不会再为那个文件描述符发送更多的就绪通知，直到你做了某些操作导致那个文件描述符不再为就绪状态了(比如，你在发送，接收或者接收请求，或者发送接收的数据少于一定量时导致了一个EWOULDBLOCK 错误）。但是请注意，如果一直不对这个fd作IO操作(从而导致它再次变成未就绪)，内核不会发送更多的通知(only once)

ET模式在很大程度上减少了epoll事件被重复触发的次数，因此效率要比LT模式高。epoll工作在ET模式的时候，必须使用非阻塞套接口，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。
```

### 3. 服务端程序

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

### 4. 客户端程序

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
    char rbuff[1024] = {0};
    int  rlen = 0;
    while(1)
    {
        memset(wbuff, 0, sizeof(wbuff));
        memset(rbuff, 0, sizeof(rbuff));

        printf("start input:\n");
        fgets(wbuff, sizeof(wbuff), stdin);
        write(sock, wbuff, sizeof(wbuff));
        recv(sock, rbuff, sizeof(rbuff), 0);
        rlen = strlen(rbuff);
        rbuff[rlen-1] = '\0';
        printf("recv:[%s]\n", rbuff);
        sleep(1);
    }

    return 0;
}
```

