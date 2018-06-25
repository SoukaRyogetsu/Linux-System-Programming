# 1.进程标识
```.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
int main(){
    printf("uid:%d gid:%d euid:%d egid:%d\n",getuid(),getgid(),geteuid(),getegid());
    return 0;
}

真实用户ID和真实组ID可以通过函数getuid()和getgid()获得。
与真实ID对应，进程还具有有效用户ID和有效组ID的属性，有效用户id和有效组id通过函数geteuid()和getegid()获得。
```
有效用户id有什么用？
我们将上述程序编译链接为可执行文件 getid ，在命令行输入
```
chmod u+s getid
```
然后把当前用户切换为 nagi ，执行 getid，发现 nagi 的信息为：uid=1001,gid=1001,euid=1000,guid=1001  
nagi 的有效用户id变了，这意味着运行 getid 的过程中，nagi 用户可以当作 ichinichi 用户(uid=1000)进行一系列 getid 所允许的操作。
容易联想到 chmod g+s getid 的功能，但是没有所谓的 o+s,不过倒是有下面这种写法
```
chmod o+t dir1
```
只能对目录操作，执行完毕后任何用户都能对该目录写入文件，然而，只能删除自己写入的文件。类似一个垃圾场，
任何人都能往里面扔垃圾，任何人也都能看别人扔了什么，垃圾场负责人(root)可以清空垃圾，不过你自己不能私吞别人的垃圾，只能清理掉自己扔的垃圾。  

注：要想更改文件所属，可以在命令行输入
```
sudo chown root:root file
```
这样，file 文件的所属用户和所属组均变为root用户
# 2.进程管理
```
ps  查看系统中的进程 
top  动态显示系统中的进程（按 q 退出）
renice  改变正在运行进程的优先级
kill  向进程发送信号（包括后台进程）
bg  将挂起的进程放到后台进行

eg：ps -elf  查看进程父子关系
eg：ps -aux  查看进程cpu和内存使用率
eg：ps -elf|grep a.out  查看 a.out 进程是否存在
eg：free  查看内存和交换区使用情况
eg：sudo fdisk -l  查看硬盘使用情况
eg：kill -9 12356  杀死pid=12356的进程
eg：renice -n 19 12356  将pid=12356的进程优先级降低19
```
注：写一个死循环，运行过程中按 Ctrl+Z 挂起进程，输入 bg 将进程后台运行，利用 jobs 显示Linux中的任务列表及任务状态，包括后台运行的任务。
然后用 fg 可将后台作业(在后台运行的或者在后台挂起的作业)放到前台终端运行。
# 3.fork
```.c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
int main(){
	int i=10;
	pid_t pid;
	pid=fork();
	if(pid==0){
		printf("I am child,pid=%d,ppid=%d\n",getpid(),getppid());
		while(1);
	}
	else{
	   	printf("I am parent,pid=%d,ppid=%d,childpid=%d\n",getpid(),getppid(),pid);
		sleep(1);
	}
	return 0;
}
程序运行结果：
I am parent,pid=3669,ppid=1361,childpid=3670
I am child,pid=3670,ppid=3669

#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<string.h>
#include<fcntl.h>
int k=10;
int main(){
	int i=10;
	int fd=open("file",O_RDWR|O_CREAT,0664);
	char *p=(char*)malloc(20);
	strcpy(p,"hello");
	int pid=fork();
	if(pid==0){
		char buf[128]={0};
		read(fd,buf,5);
		printf("I am child,i=%d,p=%s,&p=%p,k=%d,buf=%s\n",i,p,p,k,buf);
		while(1);
	}
	else{
	   	printf("I am parent,i=%d,p=%s,&p=%p,k=%d\n",i,p,p,k);
		i=5;
		k=20;
	   	strcpy(p,"world");
		printf("I am parent,i=%d,p=%s,&p=%p,k=%d\n",i,p,p,k);
		lseek(fd,5,SEEK_SET);
		printf("lseek OK\n");
		sleep(1);
	}
	return 0;
}
程序运行结果：
I am parent,i=10,p=hello,&p=0x1729010,k=10
I am parent,i=5,p=world,&p=0x1729010,k=20
lseek OK
I am child,i=10,p=hello,&p=0x1729010,k=10,buf=world

#include <unistd.h>
pid_t fork(void);
在linux中fork函数时非常重要的函数，它从已存在进程中创建一个新进程。新进程为子进程，而原进程为父进程。
它和其他函数的区别在于：它执行一次返回两个值。其中父进程的返回值是子进程的进程号，而子进程的返回值为0。
因此可以通过返回值来判断是父进程还是子进程。
fork 函数创建子进程的过程为：使用fork 函数得到的子进程是父进程的一个复制品，它从父进程继承了进程的地址空间，
包括进程上下文、进程堆栈、内存信息、打开的文件描述符、信号控制设定、进程优先级、进程组号、当前工作目录、根目录、
资源限制、控制终端，而子进程所独有的只有它的进程号、资源使用和计时器等。
通过这种复制方式创建出子进程后，原有进程和子进程都从函数fork 返回，各自继续往下运行。
但是原进程的fork 返回值与子进程的fork 返回值不同。
在原进程中，fork 返回子进程的pid,而在子进程中，fork 返回0,如果fork 返回负值，表示创建子进程失败。
```
