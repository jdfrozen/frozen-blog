# Select

### 调用过程：
![image.png](http://cdn.jdfrozen.cn/1607497400648-438b3732-70db-4275-bdbb-1f01c32ddf51.png)
### 变量fd_set：
![image.png](http://cdn.jdfrozen.cn/1607497462114-2c591d6f-4514-40cd-8a64-1d0aae206968.png)
### 设置文件描述符：
```c
void FD_CLR(int fd, fd_set *set);
int  FD_ISSET(int fd, fd_set *set);
void FD_SET(int fd, fd_set *set);
void FD_ZERO(fd_set *set);
```
![image.png](http://cdn.jdfrozen.cn/1607497515000-9a56377e-ddc2-4da7-b184-92d6bc7a7be8.png)
### Select调用
```c
int select(int nfds, fd_set *readfds, fd_set *writefds,
                  fd_set *exceptfds, struct timeval *timeout);
```
变量变化：   
![	](http://cdn.jdfrozen.cn/1607497586803-798f3149-a09f-4603-ba1b-409016376ccf.png)

### Select例子
```c
#include<stdio.h>
#include<stdlib.h>
#include<sys/types.h>
#include<sys/time.h>
#include<unistd.h>
#include<string.h>

const int LEN = 1024;
int fds[2];          //只监测标准输入与输出这两个文件描述符
int main(int argc, char* argv[]){
    int std_in = 0;
    int std_out = 1;
    int fds_max = 1;
    fd_set reads, writes;
    struct timeval timeout;
    fds[0] = std_in;
    fds[1] = std_out;
    while(1){
        FD_ZERO(&reads);
        FD_ZERO(&writes);
        FD_SET(std_in, &reads);          //标准输入关注的是读事件
        FD_SET(std_out, &writes);       //标准输出关注的是写事件
        timeout.tv_sec = 5;
        timeout.tv_usec = 0;
        switch(select(fds_max + 1, &reads, &writes, NULL, &timeout)){
            case 0:
                printf("select time out ......\n");
                break;
            case -1:
                perror("select");
                break;
            default:
                if(FD_ISSET(fds[0], &reads)){
                    char buf[LEN];
                    memset(buf, '\0', LEN);
                    fgets(buf,LEN,stdin);
                    printf("echo: %s\n", buf);
                    if(strncmp(buf, "quit", 4) == 0){
                        exit(0);
                    }
                }
                if(FD_ISSET(fds[1], &writes)){
                    char* buf = "write is ready.......\n";
                    printf("%s", buf);
                    sleep(5);
                }
                break;
        }
    }
}

```
