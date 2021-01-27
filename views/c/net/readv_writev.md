# readv和writev

#### writev
```c
#include <stdio.h>
#include <sys/uio.h>
int main(int argc, const char * argv[]) {
    struct iovec vec[2];
    char buf1[] = "ABCDEFG";
    char buf2[] = "1234567";
    int str_len;
    vec[0].iov_base = buf1;
    vec[0].iov_len = 3;
    vec[1].iov_base = buf2;
    vec[1].iov_len = 4;
    str_len = writev(1, vec, 2); //1是系统标准输出文件描述符
    puts("");
    printf("Write bytes: %d \n", str_len);    return 0;
}
```
#### readv
```c
#include <stdio.h>
#include <sys/uio.h>
#define BUF_SIZE 100
int main(int argc, const char * argv[]) {
    struct iovec vec[2];
    char buf1[BUF_SIZE] = {};
    char buf2[BUF_SIZE] = {};
    int str_len;
    vec[0].iov_base = buf1;
    vec[0].iov_len = 5;
    vec[1].iov_base = buf2;
    vec[1].iov_len = BUF_SIZE;    //把数据放到多个缓冲中储存
    str_len = readv(0, vec, 2);  //2是从标准输入接收数据
    printf("Read bytes: %d \n", str_len);
    printf("First message: %s \n", buf1);
    printf("Second message: %s \n", buf2);
    return 0;
}
```
