# 基于文件描述符的文件操作 
内核为每个进程维护一个已打开文件的记录表，文件描述符是一个较小的正整数（0—1023），它代表记录表的一项，通过文件描述符和一组基于文件描述符的文件操作函数，就可以实现对文件的读、写、创建、删除等操作。基于文件描述符的文件操作并非ANSI C 的函数。  
标准输入文件描述符     STDIN_FILENO    0  
标准输出文件描述符     STDOUT_FILENO   1  
标准错误输出文件描述符  STDERR_FILENO  2  
## 1.open
```.c
#include "func.h"                                                                            
int main(int argc,char **argv){
    int fd=open(argv[1],O_RDWR|O_CREAT,0664);
    if(fd==-1){
        perror("open failed\n");
        return -1; 
    }   
    close(fd);
    return 0;  
}

#include <sys/types.h> //头文件
#include <sys/stat.h>
#include <fcntl.h>
int open(const char *pathname, int flags); //文件名打开方式  
int open(const char *pathname, int flags, mode_t mode);//文件名打开方式权限  
flags 和 mode 都是一组掩码的合成值，flags 表示打开或创建的方式，mode 表示文件的访问权限。  
flags 的可选项有：
O_RDONLY  以只读的方式打开
O_WRONLY  以只写的方式打开
O_RDWR  以读写的方式打开
O_CREAT  如果文件不存在，则创建文件
O_TRUNC 如果文件存在，将文件的长度截至0，即把文件清空
O_APPEND 已追加的方式打开文件，每次调用write 时，文件指针自动先移到文件尾
```
## 2.read & write
```.c
#include "func.h"
int main(int argc,char **argv){
    int fd=open(argv[1],O_RDWR|O_CREAT,0664);
    printf("fd=%d\n",fd);
    int buf[128]={0};
    read(fd,buf,sizeof(buf)); //将 fd 文件里的东西读入buf
    puts(buf);
    clode(fd);
    return 0;
}

#include "func.h"
int main(int argc,char **argv){
    int fd=open(argv[1],O_RDWR|O_CREAT|O_APPEND,0664);
    printf("fd=%d\n",fd);
    int buf[128]="hellonagi";
    write(fd,buf,strlen(buf)); //将 buf 里的东西从 fd 末尾写入文件
    //int val=100；
    //write(fd,&val,strlen(int)); //也能写非字符进去，只不过要用 :%!xxd 查看
    clode(fd);
    return 0;
}

#include <unistd.h>
ssize_t read(int fd, void *buf, size_t count);//文件描述词缓冲区长度
ssize_t write(int fd, const void *buf, size_t count);  //count代表要写几个东西
对于read 和write 函数，出错返回-1，读取完了之后，返回0，其他情况返回读写的个数。
```
## 3.ftruncate
```.c
#include "func.h"                                                                            
int main(int argc,char **argv){
    int fd; 
    fd=open(argv[1],O_RDWR|O_CREAT,0664);  
    ftruncate(fd,24);
    close(fd);
    return 0;
}

#include <unistd.h>
int ftruncate(int fd, off_t length);
函数 ftruncate 会将参数 fd 指定的文件大小改为参数 length 指定的大小。
参数 fd 为已打开的文件描述符，而且必须是以写入模式打开的文件。
如果原来的文件大小比参数 length 大，则超过的部分会被删去。
如果原来的文件大小比参数 length 小，则超过的部分会进行填充。
返回值执行成功则返回0，失败返回-1。
```
## 4.lseek
```.c
#include "func.h"   
int main(int argc,char **argv){
    int fd=open(argv[1],O_RDWR|O_CREAT,0664);  
    off_t ret=lseek(fd,1024,SEEK_SET);
    printf("ret=%ld\n",ret);   //off_t是长整型
    write(fd,"ab",1);  //在第1024个位置上写了一个'a'
    close(fd);
    return 0;
}

函数 lseek 将文件指针设定到相对于whence，偏移值为offset 的位置
#include <sys/types.h>
#include <unistd.h>
off_t lseek(int fd, off_t offset, int whence);//fd 文件描述符
whence 可以是下面三个常量的一个
SEEK_SET 从文件头开始计算
SEEK_CUR 从当前指针开始计算
SEEK_END 从文件尾开始计算
利用该函数可以实现文件空洞（对一个新建的空文件，可以定位到偏移文件开头1024 个字节的地
方，在写入一个字符，则相当于给该文件分配了1025 个字节的空间，形成文件空洞）
```
## 5.fstat
```.c
#include "func.h"                                                                            
int main(int argc,char **argv){
    int fd=open(argv[1],O_RDWR|O_CREAT,0664);  
    struct stat *p=(struct stat *)calloc(1,sizeof(struct stat));
    fstat(fd,p);
    printf("size=%ld\n",p->st_size);  //获取文件长度
    free(p);
    close(fd);
    return 0;
}

可以通过fstat 和stat 函数获取文件信息，调用完毕后，文件信息被填充到结构体struct 变量中，函数原型为：
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
int stat(const char *file_name, struct stat *buf); //文件名stat 结构体指针
int fstat(int fd, struct stat *buf); //文件描述词stat 结构体指针
结构体stat 的定义为：
struct stat {
dev_t st_dev; /*如果是设备，返回设备表述符，否则为0*/
ino_t st_ino; /* i 节点号*/
mode_t st_mode; /* 文件类型*/ 无符号短整型
nlink_t st_nlink; /* 链接数*/
uid_t st_uid; /* 属主ID */
gid_t st_gid; /* 组ID */
dev_t st_rdev; /* 设备类型*/
off_t st_size; /* 文件大小，字节表示*/
blksize_t st_blksize; /* 块大小*/
blkcnt_t st_blocks; /* 块数*/
time_t st_atime; /* 最后访问时间*/
time_t st_mtime; /* 最后修改时间*/
time_t st_ctime; /* 最后权限修改时间*/
};
```
## 6.mmap
```.c
#include "func.h"                                                                            
int main(int argc,char **argv){
    int fd=open(argv[1],O_RDWR|O_CREAT,0664);
    printf("fd=%d\n",fd);
    char *p; 
    p=(char*)mmap(NULL,5,PROT_READ|PROT_WRITE,MAP_SHARED,fd,0);
    if((char*)-1==p){
        perror("mmap");
        return -1; 
    }   
    printf("p[0]=%c\n",p[0]);
    p[0]='H';
    munmap(p,5);
    return 0;
}

很多，一时总结不过来，先贴个百度百科
https://baike.baidu.com/item/mmap/1322217?fr=aladdin
```
## 7.dup
```.c
#include "func.h"                                                                            
int main(int argc,char **argv){
    int fd=open(argv[1],O_RDWR|O_CREAT,0664);   
    char buf[128]={0};
    int fd2=dup(fd);  
    printf("fd2=%d,fd=%d",fd2,fd);  //fd2=4,fd=3
    close(fd);  //关闭fd
    read(fd2,buf,sizeof(buf));  //依旧可读
    puts(buf);
    close(fd2);
    return 0;
}

#include "func.h"                                                                            
int main(int argc,char **argv){
    int fd=open(argv[1],O_RDWR|O_CREAT,0664);
    printf("%d",fd);
    fflush(stdout);
    close(1);
    int fd2=dup(fd);  //fd2=1
    //int fd2=dup2(fd,1);  //上面两行可写成这一行
    printf("fd2=%d\n",fd2);
    printf("you can't see me\n");  //这两行打不到屏幕，重定向输入到一个新建的文件内
    close(fd2);
    return 0;
}

系统调用函数 dup 和 dup2 可以实现文件描述符的复制，经常用来重定向进程的stdin(0),stdout(1),stderr(2)。
dup 返回新的文件描述符（没有使用的文件描述符的最小编号）。
这个新的描述符是旧文件描述符的拷贝。这意味着两个描述符共享同一个数据结构。
dup2 允许调用者用一个有效描述符(oldfd)和目标描述符(newfd)，函数成功返回时，目标描述符将变成旧描述符的复制品。
此时两个文件描述符现在都指向同一个文件，并且是函数第一个参数（也就是oldfd）指向的文件。
原型为：
#include <unistd.h> //头文件
int dup(int oldfd);
int dup2(int oldfd, int newfd);
文件描述符的复制是指用另外一个文件描述符指向同一个打开的文件，它完全不同于直接给文件描
述符变量赋值
```
## 8.pipe
```.c
mkfifo 1pipe 建立一个管道
```
