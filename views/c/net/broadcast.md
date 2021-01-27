# 广播

#### 直接广播：
192.1 2.34所有主机传输数据
192.1 2.34.255
#### 本地广播：
255.255.255.2555
#### 实现：
默认生成的套接字会阻止广播
修改：
int so_brd = 1;
setsockopt(send_sock,SOL_SOCKET,SO_BROADCAST,(void*)&so_brd,sizeof(so_brd));
```c
/* 基于广播的Sender */
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>
#define BUF_SIZE 30
void error_handling(char *message);
int main(int argc, char *argv[]){
	int send_sock;
	struct sockaddr_in broad_adr;
	FILE *fp;
	char buf[BUF_SIZE];
	if (argc != 3) {
		printf("Usage: %s <GroupIP> <PORT> \n",argv[0]);
		exit(1);
	}
	send_sock = socket(PF_INET,SOCK_DGRAM,0);
	memset(&broad_adr,0,sizeof(broad_adr));
	broad_adr.sin_family = AF_INET;
	broad_adr.sin_addr.s_addr = inet_addr(argv[1]);
	broad_adr.sin_port = htons(atoi(argv[2]));
    int so_brd = 1;
	setsockopt(send_sock,SOL_SOCKET,SO_BROADCAST,(void*)&so_brd,sizeof(so_brd));
	if((fp = fopen("news.txt","r")) == NULL){error_handling("fopen() error!");}
	/* 实际传输数据的区域,基于UDP套接字传输数据,*/
	while(!feof(fp)){
		fgets(buf,BUF_SIZE,fp);	//从fp中读取数据,每次读取一行,每次最多读取BUF_SIZE-1个字符,读取的数据保存到buf
		sendto(send_sock,buf,strlen(buf),0,(struct sockaddr*)&broad_adr,sizeof(broad_adr));		//更改UDP套接字可选项，使其能够发送广播数据
		sleep(2);				//给传输数据提供一定的时间间隔
	}
	close(send_sock);
	return 0;
}
void error_handling(char *message){
	fputs(message,stderr);
	fputc('\n',stderr);
	exit(1);
}
```
./hc 255.255.255.255 9100
```c
/* 基于广播的Receiver */
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>
#define BUF_SIZE 30

void error_handling(char *message);
int main(int argc,char *argv[]){
	int recv_sock;
	int str_len;
	char buf[BUF_SIZE];
	struct sockaddr_in adr;
	struct ip_mreq join_adr;
	if (argc != 2) {
		printf("Usage: %s <GroupIP> <PORT> \n",argv[0]);
		exit(1);
	}
	recv_sock = socket(PF_INET,SOCK_DGRAM,0);
	memset(&adr,0,sizeof(adr));
	adr.sin_family = AF_INET;
	adr.sin_addr.s_addr = htonl(INADDR_ANY);
	adr.sin_port = htons(atoi(argv[1]));
	if (bind(recv_sock,(struct sockaddr*)&adr,sizeof(adr)) == -1){error_handling("bind() error!");}
	while(1){
		str_len = recvfrom(recv_sock,buf,BUF_SIZE-1,0,NULL,0);	//接收数据
		if (str_len < 0)
			break;
		buf[str_len] = 0;
		fputs(buf,stdout);
	}
	close(recv_sock);
	return 0;
}
void error_handling(char *message){
	fputs(message,stderr);
	fputc('\n',stderr);
	exit(1);
}

```
./hs 9100
