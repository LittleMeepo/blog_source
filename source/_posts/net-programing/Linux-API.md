## Linux 常用网络API

#### IPv4 专用socket地址结构体

```c
struct sockaddr_in
{
    sa_family_t sin_family;     /* 地址族：AF_INET */
    u_int16_t sin_port;         /* 端口号，要用网络字节序表示 */
    struct in_addr sin_addr;    /* IPv4地址结构体，见下面 */
};
struct in_addr
{
    u_int32_t s_addr;           /* IPv4地址，要用网络字节序表示 */
};
```

所有专用socket地址（以及sockaddr_storage）类型的变量在实际使用时都需要转化为通用socket地址类型sockaddr（强制转换即可），因为所有socket编程接口使用的地址参数的类型都是sockaddr

#### 创建socket

```c
#include<sys/types.h>
#include<sys/socket.h>
int socket(int domain, int type, int protocol);
```

- domain：底层协议簇
  - PF_INET (Protocol Family of Internet，用于IPv4)
  - PF_INET6 (用于IPv6)
- type：服务类型
  - SOCK_STREAM 流服务
  - SOCK_UGRAM 数据报
- protocol：一般0
- 成功返回0，失败返回-1并设置errno

#### 命名socket

```c
#include<sys/types.h>
#include<sys/socket.h>
int bind(int sockfd, const struct sockaddr* my_addr, socklen_t addrlen);
```

成功返回0，失败返回-1并设置errno。两种常见的errno是EACCES（被绑定的地址是受保护的地址，仅超级用户能够访问，如普通用户绑定到知名服务端口）；EADDRINUSE（被绑定的地址正在使用中）

#### 监听socket

```c
#include<sys/socket.h>
int listen(int sockfd, int backlog);
```

backlog参数提示内核监听队列的最大长度。监听的队列的长度超过backlog，服务器将不受理新的客户连接，客户端也将收到ECONNREFUSED错误信息

成功返回0，失败返回-1并设置errno

#### 接受连接

从监听队列中接受一个连接：

```c
#include<sys/types.h>
#include<sys/socket.h>
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

成功返回一个新的连接socket，用于通信。失败返回-1并设置errno

####  发起连接

客户端主动与服务器建立连接：

```c
#include<sys/types.h>
#include<sys/socket.h>
int connect(int sockfd, const struct sockaddr *serv_addr, socklen_t addrlen);
```

成功返回0，并可通过sockfd进行通信，失败返回-1并设置errno。两种常见的errno是ECONNREFUSED（目标端口不存在）；ETIMEDOUT（连接超时）

#### 关闭连接

```c
#include<unistd.h>
int close(int fd);
```

#### TCP数据读写

```c
#include<sys/types.h>
#include<sys/socket.h>
ssize_t recv(int sockfd, void *buf, size_t len, int flags);
ssize_t send(int sockfd, const void *buf, size_t len, int flags);
```

recv读取sockfd上的数据，成功时返回实际读取到数据的长度，可能小于我们期望的长度len，失败返回-1并设置errno

send往sockfd上写入数据，成功时返回实际写入的数据的长度，失败返回-1并设置errno

flag参数为数据收发提供额外控制

#### 地址信息函数

```c
#include<sys/socket.h>
int getsockname(int sockfd, struct sockaddr* address, socklen_t* address_len);
int getpeername(int sockfd, struct sockaddr* address, socklen_t* address_len);
```

getsockname获取sockfd对应的本端socket地址

getpeername获取sockfd对应的远端socket地址

#### getaddrinfo & getnameinfo

```c
/******************************** 
 * Client/server helper functions
 ********************************/
/*
 * open_clientfd - Open connection to server at <hostname, port> and
 *     return a socket descriptor ready for reading and writing. This
 *     function is reentrant and protocol-independent.
 *
 *     On error, returns: 
 *       -2 for getaddrinfo error
 *       -1 with errno set for other errors.
 */
/* $begin open_clientfd */
int open_clientfd(char *hostname, char *port) {
    int clientfd, rc;
    struct addrinfo hints, *listp, *p;

    /* Get a list of potential server addresses */
    memset(&hints, 0, sizeof(struct addrinfo));
    hints.ai_socktype = SOCK_STREAM;  /* Open a connection */
    hints.ai_flags = AI_NUMERICSERV;  /* ... using a numeric port arg. */
    hints.ai_flags |= AI_ADDRCONFIG;  /* Recommended for connections */
    if ((rc = getaddrinfo(hostname, port, &hints, &listp)) != 0) {
        fprintf(stderr, "getaddrinfo failed (%s:%s): %s\n", hostname, port, gai_strerror(rc));
        return -2;
    }
  
    /* Walk the list for one that we can successfully connect to */
    for (p = listp; p; p = p->ai_next) {
        /* Create a socket descriptor */
        if ((clientfd = socket(p->ai_family, p->ai_socktype, p->ai_protocol)) < 0) 
            continue; /* Socket failed, try the next */

        /* Connect to the server */
        if (connect(clientfd, p->ai_addr, p->ai_addrlen) != -1) 
            break; /* Success */
        if (close(clientfd) < 0) { /* Connect failed, try another */  //line:netp:openclientfd:closefd
            fprintf(stderr, "open_clientfd: close failed: %s\n", strerror(errno));
            return -1;
        } 
    } 

    /* Clean up */
    freeaddrinfo(listp);
    if (!p) /* All connects failed */
        return -1;
    else    /* The last connect succeeded */
        return clientfd;
}
/* $end open_clientfd */

/*  
 * open_listenfd - Open and return a listening socket on port. This
 *     function is reentrant and protocol-independent.
 *
 *     On error, returns: 
 *       -2 for getaddrinfo error
 *       -1 with errno set for other errors.
 */
/* $begin open_listenfd */
int open_listenfd(char *port) 
{
    struct addrinfo hints, *listp, *p;
    int listenfd, rc, optval=1;

    /* Get a list of potential server addresses */
    memset(&hints, 0, sizeof(struct addrinfo));
    hints.ai_socktype = SOCK_STREAM;             /* Accept connections */
    hints.ai_flags = AI_PASSIVE | AI_ADDRCONFIG; /* ... on any IP address */
    hints.ai_flags |= AI_NUMERICSERV;            /* ... using port number */
    if ((rc = getaddrinfo(NULL, port, &hints, &listp)) != 0) {
        fprintf(stderr, "getaddrinfo failed (port %s): %s\n", port, gai_strerror(rc));
        return -2;
    }

    /* Walk the list for one that we can bind to */
    for (p = listp; p; p = p->ai_next) {
        /* Create a socket descriptor */
        if ((listenfd = socket(p->ai_family, p->ai_socktype, p->ai_protocol)) < 0) 
            continue;  /* Socket failed, try the next */

        /* Eliminates "Address already in use" error from bind */
        setsockopt(listenfd, SOL_SOCKET, SO_REUSEADDR,    //line:netp:csapp:setsockopt
                   (const void *)&optval , sizeof(int));

        /* Bind the descriptor to the address */
        if (bind(listenfd, p->ai_addr, p->ai_addrlen) == 0)
            break; /* Success */
        if (close(listenfd) < 0) { /* Bind failed, try the next */
            fprintf(stderr, "open_listenfd close failed: %s\n", strerror(errno));
            return -1;
        }
    }


    /* Clean up */
    freeaddrinfo(listp);
    if (!p) /* No address worked */
        return -1;

    /* Make it a listening socket ready to accept connection requests */
    if (listen(listenfd, LISTENQ) < 0) {
        close(listenfd);
	return -1;
    }
    return listenfd;
}
/* $end open_listenfd */
```

