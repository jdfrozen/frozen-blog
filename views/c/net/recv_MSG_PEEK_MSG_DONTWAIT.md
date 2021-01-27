# recv函数MSG_PEEK|MSG_DONTWAIT

MSG_DONTWAIT:调用函数时不阻塞
MSG_PEEK:只有recv能用,检测input缓冲区是否有数据,(缓冲区数据不变）
```c
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
void error_handling(char * messages);
int main(int argc,char *argv[]){
  int sock;
  struct sockaddr_in send_adr;
  if (argc != 3){
    printf("Usage: %s <IP> <port> \n",argv[0]);
    exit(1);
  }
  sock = socket(AF_INET,SOCK_STREAM,0);
  memset(&send_adr,0,sizeof(send_adr));
  send_adr.sin_family = AF_INET;
  send_adr.sin_addr.s_addr = inet_addr(argv[1]);
  send_adr.sin_port = htons(atoi(argv[2]));
  if (connect(sock,(struct sockaddr*)&send_adr,sizeof(send_adr)) == -1){error_handling("connect() error");}
  write(sock,"1234",strlen("1234"));
  close(sock);
  return 0;
}
void error_handling(char * messages){
    puts(messages);
    exit(1);
}
```
```c
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#define BUF_SIZE 30
void error_handling(char * messages);
int main(int argc,char *argv[])
{
  int acpt_sock,recv_sock;
  struct sockaddr_in acpt_adr,recv_adr;
  int str_len,state;
  socklen_t recv_adr_sz;
  char buf[BUF_SIZE];
  if (argc != 2){
    printf("Usage : %s <port>\n",argv[0]);
    exit(1);
  }

  //绑定服务端套接字
  acpt_sock = socket(AF_INET,SOCK_STREAM,0);
  memset(&acpt_adr,0,sizeof(acpt_adr));
  acpt_adr.sin_family = AF_INET;
  acpt_adr.sin_addr.s_addr = htonl(INADDR_ANY);
  acpt_adr.sin_port = htons(atoi(argv[1]));

  if (bind(acpt_sock,(struct sockaddr*)&acpt_adr,sizeof(acpt_adr)) == -1){error_handling("bind() error");}

  listen(acpt_sock,5);
  recv_adr_sz = sizeof(recv_adr);
  recv_sock = accept(acpt_sock,(struct sockaddr*)&recv_adr,&recv_adr_sz);

  //以非阻塞的方式验证待读入的数据是否存在,不存在则一直询问?,存在则退出
  while(1){
    str_len = recv(recv_sock,buf,sizeof(buf)-1,MSG_PEEK | MSG_DONTWAIT);
    if (str_len > 0)
      break;
  }
  buf[str_len] = 0;
  printf("Buffering %d bytes: %s \n",str_len,buf);//拿到缓冲区的数据,
  str_len = recv(recv_sock,buf,sizeof(buf)-1,0);//读入后会删除缓冲区
  buf[str_len] = 0;
  printf("Read again: %s \n",buf);
  close(acpt_sock);
  close(recv_sock);
  return 0;
}
void error_handling(char * messages){
    puts(messages);
    exit(1);
}
```
