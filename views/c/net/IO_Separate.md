# IO分离流

#### 问题：
![](http://cdn.jdfrozen.cn/1607599538107-710af187-db1b-4557-a3ad-aa17ae85af86.png)
![](http://cdn.jdfrozen.cn/1607599548766-1c8bd6b8-b1f9-4d66-972d-31fca43327a2.png)

### dup & dup2
文件描述符的复制函数:
![](http://cdn.jdfrozen.cn/1607600170727-82973e1f-4026-4fb9-9eb7-1076ad502447.png)
#### 标准IO半关闭实现：
```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>
#define BUF_SIZE 1024
 
int main(int argc,char *argv[])
{
	int sock;
	char buf[BUF_SIZE];
	struct sockaddr_in serv_addr;
 
	FILE * readfp;
	FILE * writefp;
 
	sock = socket(PF_INET,SOCK_STREAM,0);
	memset(&serv_addr,0,sizeof(serv_addr));
	serv_addr.sin_family = AF_INET;
	serv_addr.sin_addr.s_addr = inet_addr(argv[1]);
	serv_addr.sin_port = htons(atoi(argv[2]));
 
	connect(sock,(struct sockaddr*)&serv_addr,sizeof(serv_addr));
	readfp = fdopen(sock,"r");
	writefp = fdopen(sock,"w");
 
	while(1)
	{
		if(fgets(buf,sizeof(buf),readfp) == NULL)	//收到EOF时,fgets函数返回NULL指针,退出循环.
			break;
		fputs(buf,stdout);
		fflush(stdout);
	}
 
	fputs("FROM CLIENT: Thank you! \n",writefp);	//发送最后的字符串
	fflush(writefp);
	fclose(writefp); fclose(readfp);
	return 0;
}
```
```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>
#define BUF_SIZE 1024
 
int main(int argc,char *argv[])
{
	int serv_sock,clnt_sock;
	FILE * readfp;
	FILE * writefp;
 
	struct sockaddr_in serv_adr, clnt_adr;
	socklen_t clnt_adr_sz;
	char buf[BUF_SIZE] = {0,};
	
	serv_sock = socket(PF_INET,SOCK_STREAM,0);
	memset(&serv_adr,0,sizeof(serv_adr));
	serv_adr.sin_family = AF_INET;
	serv_adr.sin_addr.s_addr = htonl(INADDR_ANY);
	serv_adr.sin_port = htons(atoi(argv[1]));
 
	bind(serv_sock,(struct sockaddr*)&serv_adr, sizeof(serv_adr));
	listen(serv_sock,5);
	clnt_adr_sz = sizeof(clnt_adr);
	clnt_sock = accept(serv_sock,(struct sockaddr*)&clnt_adr,&clnt_adr_sz);
 
	readfp = fdopen(clnt_sock,"r");			//创建读模式FILE指针
	writefp = fdopen(dup(clnt_sock),"w");	//针对dup函数的返回值生成FILE指针。
 
	fputs("FROM SERVER: HI~ client? \n",writefp);
	fputs("I love all of the world \n",writefp);
	fputs("You are awesome! \n",writefp);
	fflush(writefp);		
 
	shutdown(fileno(writefp),SHUT_WR);		//针对fileno函数返回的文件描述符调用shutdown函数。服务器端进入半关闭状态，并向客户端发送EOF。
	fclose(writefp);						
	
	fgets(buf,sizeof(buf),readfp);			//使用readfp接收数据
	fputs(buf,stdout);
	fclose(readfp);
	return 0;
}
```


