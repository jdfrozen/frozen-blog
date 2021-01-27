# SOCKET选项

getsockopt()函数用于获取任意类型、任意状态套接口的选项当前值，并把结果存入optval
```c
#include <sys/socket.h>
int getsockopt(int sockfd, int level, int optname, void *optval, socklen_t *optlen);
/*
sockfd：一个标识套接口的描述字。
level：选项定义的层次。例如，支持的层次有SOL_SOCKET、IPPROTO_TCP等。
optname：需获取的套接口选项。
optval：指针，指向存放所获得选项值的缓冲区。
optlen：指针，指向optval缓冲区的长度值。
*/
```
setsockopt()函数用于任意类型、任意状态套接口的设置选项值。尽管在不同协议层上存在选项，但本函数仅定义了最高的“套接口”层次上的选项。
```c
#include <sys/socket.h>
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);
/*
sockfd：标识一个套接口的描述字。
level：选项定义的层次；支持SOL_SOCKET、IPPROTO_TCP、IPPROTO_IP和IPPROTO_IPV6等。
optname：需设置的选项。
optval：指针，指向存放选项值的缓冲区。
optlen：optval缓冲区长度。
*/
```
**

| **level级别：SOL_SOCKET** |  |  |  |
| --- | --- | --- | --- |
| **optname(选项名)** | **选项值数据类型** | **访问** | **说明** |
| SO_ACCEPTCONN | bool | get | 如为TRUE（真） ，表明套接字处于监听模式_ |
| SO_BROADCAST | bool | get/set | 如TRUE，表明套接字已配置成对广播消息进行发送_ |
| SO_CONNECT_TIME | int | get | 返回套接字建立连接的时间，以秒为单位，如尚未连接，返回0xffffffff |
| SO_DEBUG | bool | get/set | 如果TRUE，就允许调试输出 (W32不支持) |
| SO_DONTLINGER | bool | get/set | 如果是TRUE，则禁用SO_LINGER_ |
| SO_LINGER | struct linger | get/set | 设置或获取当前的拖延值_ |
| SO_DONTROUTE | bool | get/set | 如果TRUE,便直接向网络接口发送消息，毋需查询路由表 |
| SO_ERROR | bool | get | 返回错误状态_ |
| SO_EXCLUSIVEADDRUSE | bool | get/set | 如果TRUE，套接字绑定那个本地端口就不能重新被另一个进程使用_ |
| SO_KEEPALIVE | bool | get/set | 如果TRUE，套接字就会进行配置，在会话过程中发送”保持活动”消息_ |
| SO_MAX_MSG_SIZE | unsigned int | get | 对一个面向消息的套接字来说，一条消息的最大长度_ |
| SO_OOBINLINE | bool | get/set | 如果是TRUE，带外数据就会在普通数据流中返回 (W32不支持) |
| SO_PROTOCOL_INFO | WSAPROTOCOL_INFO | get | 套接字绑定的那种协议的特征_ |
| SO_RCVBUF | int | get/set | 面向接收操作，为每个套接字分别获取或设置缓冲区长度_ |
| SO_REUSEADDR | bool | get/set | 如果是TRUE，套接字就可与一个正由其他套接字使用的地址绑定到一起，或与处在TIME_WAIT状态的地址绑定到一起 |
| SO_SNDBUF | bool | get/set | 设置分配给套接字的数据发送缓冲区的大小 |
| SO_TYPE | int | get | 返回指定套接字的类型（如SOCK_DGRAM和SOCK_STREAM等等）_ |
| SO_SNDTIMEO | int | get/set | 获取或设置套接字上的数据发送超时时间（以毫秒为单位）_ |
| SO_RCVTIMEO | int | get/set | 获取或设置与套接字上数据接收对应的超时时间值（以毫秒为单位） |
| SO_UPDATE_ACCEPT_CONTEXT | SOCKET | get/set | 更新SOCKET状态  |

| **level级别：**IPPROTO_IP**** |  |  |  |
| --- | --- | --- | --- |
| **optname(选项名)** | **选项值数据类型** | **访问** | **说明** |
| IP_OPTIONS | char[] | get/set | 设置或获取IP头内的IP选项 |
| IP_HDRINCL | bool | get/set | 如果是TRUE,IP头就会随即将发送的数据一起提交，并从读取的数据中返回 |
| IP_TOS | int | get/set | IP服务类型 |
| IP_TTL
 | int | get/set | IP协议的“存在时间” （TTL） |
| IP_MULTICAST_IF | unsigned long | get/set | 获取或设置打算从它上面发出多播数据的本地接口 |
| IP_MULTICAST_TTL | int | get/set | 为套接字获取或设置多播数据包的存在时间 |
| IP_MULTICAST_LOOP | bool | get/set | 如果TRUE，发至多播地址的数据将原封不动地“反射”或“反弹”回套接字的进入缓冲区 |
| IP_ADD_MEMBERSHIP | struct ip_mreq | set | 在指定的IP组内为套接字赋予成员资格 |
| IP_DROP_MEMBERSHIP | struct ip_mreq | set | 将套接字从指定的IP组内删去（撤消成员资格） |
| IP_DONTFRAGMENT | bool | get/set | 如果是TRUE，就不对IP数据报进行分段 |
| **level级别：**IPPROTO_TCP**** |   |   |   |
| **optname(选项名)** | **选项值数据类型** | **访问** | **说明** |
| TCP_NODELAY | bool | get/set | 若为TRUE, 就会在套接字上禁用Nagle算法 (只适用于流式套接字)  |

```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<sys/socket.h>

void error_handling(char* message);

int main(int argc,char* argv)
{
    //声明socket
    int tcp_sock,udp_sock;
    //socket类型
    int sock_type;
    //socket缓冲区大小
    int snd_buf,rev_buf;
    //可选项字节数
    socklen_t optlen;
    //getsockopt返回的状态，0表示获取成功
    int state;

    optlen=sizeof(sock_type);
    tcp_sock=socket(PF_INET,SOCK_STREAM,0);
    udp_sock=socket(PF_INET,SOCK_DGRAM,0);
    printf("SOCK_STREAM: %d\n",SOCK_STREAM);
    printf("SOCK_DGRAM: %d\n",SOCK_DGRAM);

    state=getsockopt(tcp_sock,SOL_SOCKET,SO_TYPE,&sock_type,&optlen);
    if(state)
        error_handling("getsockopt one error");
    printf("socket type one: %d\n",sock_type);
    state=getsockopt(udp_sock,SOL_SOCKET,SO_TYPE,&sock_type,&optlen);
    if(state)
        error_handling("getsockopt two error");
    printf("socket type two: %d\n",sock_type);

    optlen=sizeof(snd_buf);
    state=getsockopt(tcp_sock,SOL_SOCKET,SO_SNDBUF,&snd_buf,&optlen);
    if(state==0)
        printf("socket输出缓冲区大小是: %d\n",snd_buf);
    optlen=sizeof(rev_buf);
    state=getsockopt(tcp_sock,SOL_SOCKET,SO_RCVBUF,&rev_buf,&optlen);
    if(state==0)
        printf("socket输入缓冲区大小是: %d\n",rev_buf);

    printf("更改输入和输出缓冲区...\n");

    snd_buf=1024*3;
    rev_buf=1024*6;

    optlen=sizeof(snd_buf);
    state=setsockopt(tcp_sock,SOL_SOCKET,SO_SNDBUF,&snd_buf,optlen);
    if(state==0)
    {
        state=getsockopt(tcp_sock,SOL_SOCKET,SO_SNDBUF,&snd_buf,&optlen);
        printf("更改成功!\n");
        printf("更改后的输出缓冲区大小为:%d\n",snd_buf);
    }
    optlen=sizeof(rev_buf);
    state=setsockopt(tcp_sock,SOL_SOCKET,SO_RCVBUF,&rev_buf,optlen);
    if(state==0)
    {
        state=getsockopt(tcp_sock,SOL_SOCKET,SO_RCVBUF,&rev_buf,&optlen);
        printf("更改后的输入缓冲区大小为:%d\n",rev_buf);
    }
    return 0;
}

void error_handling(char* message)
{
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}

```
