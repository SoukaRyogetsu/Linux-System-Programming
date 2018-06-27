# 1.管道通信
## 1.标准流管道
像文件操作有标准io 流一样，管道也支持文件流模式。用来创建连接到另一进程的管道，是通过函数popen 和pclose。  
函数原型：   
#include <stdio.h>  
FILE* popen(const char* command, const char* open_mode);  
int pclose(FILE* fp);  
```.c
写一个print.c
#include<stdio.h>
#include<unistd.h>
int main(){
	printf("I am print process\n");
	return 0;
}
```
然后 gcc print.c -o print
```.c
写一个popen_r.c
#include<stdio.h>
#include<unistd.h>
int main(){
	FILE *fp;
	fp=popen("./print","r");
	if(NULL==fp){
		perror("popen");
		return -1;
	}
	char buf[128]={0};
	fgets(buf,sizeof(buf),fp);
	printf("%s",buf);
	pclose(fp);
	return 0;
}
```
直接编译运行 popen_r ，打印 I am print process  
同理，写一个 scan.c 和 popen_r.c
```.c
#include<stdio.h>
#include<unistd.h>
int main(){
	FILE *fp;
	fp=popen("./scan","w");
	if(NULL==fp){
		perror("popen");
		return -1;
	}
	char buf[128]="3 4";
	fputs(buf,fp);
	pclose(fp);
	return 0;
}

#include<stdio.h>
int main(){
	int i,j;
	scanf("%d%d",&i,&j);
	printf("sum=%d\n",i+j);
	return 0;
}
```
## 2.无名管道
无名管道的特点： 
1.只能在亲缘关系进程间通信（父子或兄弟）  
2.半双工（固定的读端和固定的写端）  
3.他是特殊的文件，可以用read、write 等，只能在内存中  
管道函数原型：  
#include <unistd.h>  
int pipe(int fds[2]);  
管道在程序中用一对文件描述符表示，其中一个文件描述符有可读属性，一个有可写的属性。fds[0]是读，fds[1]是写。  
函数pipe 用于创建一个无名管道，如果成功，fds[0]存放可读的文件描述符，fds[1]存放可写文件描述符。  
通过调用pipe 获取这对打开的文件描述符后，一个进程就可以从fds[0]中读数据，而另一个进程就可以往fds[1]中写数据。  
当然两进程间必须有继承关系，才能继承这对打开的文件描述符。  
```.c
#include<stdio.h>
#include<string.h>
#include<unistd.h>
#include<sys/wait.h>
#include<sys/types.h>
int main(){
	int fds[2];
	int fds1[2];
	pipe(fds);
	pipe(fds1);
	if(!fork()){//child
		while(1){
			close(fds[1]);
			close(fds1[0]);
			char buf[111]={0};
			char buf1[111]={0};
			memset(buf1,0,sizeof(buf1));
			printf("child send:");
			scanf("%s",buf1);
			write(fds1[1],buf1,strlen(buf1));
			read(fds[0],buf,sizeof(buf));
			printf("child receive: %s\n\n",buf);
		}
	}
	else{
		while(1){
			close(fds[0]);
			close(fds1[1]);
			char buf[111]={0};
			char buf1[111]={0};
			memset(buf1,0,sizeof(buf1));
			read(fds1[0],buf,sizeof(buf));
			printf("parent send:");
			scanf("%s",buf1);
			printf("parent receive: %s\n",buf);
			write(fds[1],buf1,strlen(buf1));
		}
	}
	wait(NULL);
}
```
## 3.有名管道
就是之前用 mkfifo 创建的管道。
```.c
//写一个 mkfifo.c
#include<stdio.h>
#include<sys/stat.h>
#include<unistd.h>
#include<sys/types.h>
int main(int argc,char **argv){
	mkfifo(argv[1],0666);
	return 0;
}
//写一个 unlink.c
#include<stdio.h>
#include<unistd.h>
int main(int argc,char **argv){
    unlink(argv[1]);
    return 0;
}
```
# 2.共享内存
```.c
#include<stdio.h>
#include<sys/types.h>
#include<sys/ipc.h>
#include<sys/shm.h>
#include<string.h>
int main(){
  	int shmid=shmget(1000,4096,IPC_CREAT|0600);
  	printf("shmid=%d\n",shmid);
  	char *p=(char*)shmat(shmid,NULL,0);
//	memset(p,0,20);//第一次取消这一行与下两行注释，第四行保持注释
//	strcpy(p,"How are you");
//  char buf[20]={0};
//  strcpy(buf,p);//第二次取消这一行，前三行保持注释
  	puts(buf);
  	return 0;
}
```
第一次创建了一个共享内存，并写入一个字符串，第二次可以直接从该段共享内存内取值。  
可以使用 ipcs 指令查看系统共享内存。  
可以使用 ipcrm -m [shmid] 指令删除某共享内存。
```.c
//在一个共享内存处写入一个整数0，父子进程对该整数各进行一千万次加一操作，输出最终结果
#include<stdio.h>
#include<unistd.h>
#include<sys/types.h>
#include<sys/ipc.h>
#include<sys/shm.h>
#include<sys/wait.h>
#define N 10000000
int main(){
	int shmid=shmget(1000,4096,IPC_CREAT|0600); //第一次创建，第二次获取
	int *p=shmat(shmid,NULL,0);
	*p=0;
	int i;
	if(!fork()){
		for(i=0;i<N;i++)
			*p=*p+1;
	}
	else{
		for(i=0;i<N;i++)
			*p=*p+1;
		wait(NULL);
		printf("%d\n",*p);
	}
	return 0;
}
```
很大情况下，不会输出两千万，电脑CPU比较强的话概率出现两千万。
```.c
//写一个 shmctl.c
#include<stdio.h>
#include<unistd.h>
#include<sys/ipc.h>
#include<sys/shm.h>
int main(){
    int shmid=shmget(IPC_PRIVATE,4096,0600|IPC_CREAT); //IPC_PRIVATE每次都是创建
    shmctl(shmid,IPC_RMID,NULL);
```
int shmctl(int shmid, int cmd, struct shmid_ds *buf);  
函数 shmctl 是共享内存的控制函数，可以用来删除共享内存段。   
参数 shmid 是共享内存段标识通常应该是shmget 的成功返回值   
参数 cmd 是对共享内存段的操作方式，可选为IPC_STAT,IPC_SET,IPC_RMID。通常为IPC_RMID，表示删除共享内存段。
参数 buf 是表示共享内存段的信息结构体数据，通常为NULL。

如果我们删除共享内存的时候，有进程正在用，能删的掉吗？  
会删除成功(返回值0)，但是共享内存不会消失,不能有新进程加入，但最后一个进程结束时，共享内存关闭。
