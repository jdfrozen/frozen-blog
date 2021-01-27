# 多播

#### 多播地址：
多播组是D类E地址( 224.0.0.0--239.255.255.255 )
#### TTL：
路由（Routing ) 和TTL （Time to Live ，生存时间)
#### 加入多播组：
join_adr.imr_multiaddr.s_addr="" 多播纽地址信息";
join_adr.imr_interface.s_addr=" 加入多播组的主机地址信息";
setsockopt(recv_sock ，IPPROTO_IP ， IP_ADD_MEMBER5HIP, (void*)&join_adr)，sizeof(join_adr));
```c
/* Sender只需创建UDP套接字，并向多播地址发送数据 */
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>

#define TTL 64
#define BUF_SIZE 30
void error_handling(char *message);

int main(int argc, char *argv[]){
	int send_sock;
	struct sockaddr_in mul_adr;
	int time_live = TTL;
	FILE *fp;
	char buf[BUF_SIZE];
	if (argc != 3) {
		printf("Usage: %s <GroupIP> <PORT> \n",argv[0]);
		exit(1);
	}
	send_sock = socket(PF_INET,SOCK_DGRAM,0);
	memset(&mul_adr,0,sizeof(mul_adr));
	mul_adr.sin_family = AF_INET;
	mul_adr.sin_addr.s_addr = inet_addr(argv[1]);		//Multicast IP
	mul_adr.sin_port = htons(atoi(argv[2]));			//Multicast Port
	setsockopt(send_sock,IPPROTO_IP,IP_MULTICAST_TTL,(void*)&time_live,sizeof(time_live));	//指定套接字TTL
	if((fp = fopen("news.txt","r")) == NULL){error_handling("fopen() error!");}
	/* 实际传输数据的区域,基于UDP套接字传输数据,*/
	while(!feof(fp)){
		fgets(buf,BUF_SIZE,fp);	//从fp中读取数据,每次读取一行,每次最多读取BUF_SIZE-1个字符,读取的数据保存到buf
		sendto(send_sock,buf,strlen(buf),0,(struct sockaddr*)&mul_adr,sizeof(mul_adr));
		sleep(2);				//给传输数据提供一定的时间间隔
	}
	fclose(fp);
	close(send_sock);
	return 0;
}

void error_handling(char *message)
{
	fputs(message,stderr);
	fputc('\n',stderr);
	exit(1);
}
```
./hc 224.1.1.2 9100
```c
/* Receiver为了接收传向任意多播地址的数据，需要经过加入多播组的过程 */
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
	if (argc != 3) {
		printf("Usage: %s <GroupIP> <PORT> \n",argv[0]);
		exit(1);
	}
	recv_sock = socket(PF_INET,SOCK_DGRAM,0);
	memset(&adr,0,sizeof(adr));
	adr.sin_family = AF_INET;
	adr.sin_addr.s_addr = htonl(INADDR_ANY);
	adr.sin_port = htons(atoi(argv[2]));

	if (bind(recv_sock,(struct sockaddr*)&adr,sizeof(adr)) == -1){error_handling("bind() error!");}
	join_adr.imr_multiaddr.s_addr = inet_addr(argv[1]);	//初始化多播组地址
	join_adr.imr_interface.s_addr = htonl(INADDR_ANY);	//初始化待加入主机的IP地址
	setsockopt(recv_sock,IPPROTO_IP,IP_ADD_MEMBERSHIP,(void*)&join_adr,sizeof(join_adr));			//利用可选项IP_ADD_MEMBERSHIP加入多播组
	while(1){
		str_len = recvfrom(recv_sock,buf,BUF_SIZE-1,0,NULL,0);  //接收多播数据
		if (str_len < 0)
			break;
		buf[str_len] = 0;
		fputs(buf,stdout);
	}
	close(recv_sock);
	return 0;
}

void error_handling(char *message)
{
	fputs(message,stderr);
	fputc('\n',stderr);
	exit(1);
}
```
./hc 224.1.1.2 9100
