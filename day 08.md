# 1.exec函数簇
```.c
//写一个add.c
#include<stdlib.h>
#include<stdio.h>
int main(int argc,char **argv){
	int i=atoi(argv[1]);
	int j=atoi(argv[2]);
	printf("sum=%d",i+j);
	return 0;
}

gcc add.c -o add

//写一个execl.c
#include<stdio.h>
#include<unistd.h>
#include<string.h>
int main(){
	execl("./add","add","3","4",NULL);
	printf("you cannot see me\n");
	return 0;
}
```
gcc execl.c  
./a.out  
屏幕打印 7
```.c
//写一个execv.c
#include<stdio.h>
#include<unistd.h>
#include<string.h>
int main(){
  char *args[4]={"add","3","4",NULL};
	execl("./add",args);
	printf("you cannot see me\n");
	return 0;
}
结果与execl.c相同
```
```.c
//写一个execvp.c
#include<stdio.h>
#include<unistd.h>
#include<string.h>
int main(){
  char *envp[2]={"PATH=/home/ichinichi/",NULL};
	execl("./add","add","3","4",NULL,envp);
	printf("you cannot see me\n");
	return 0;
}
//可以实现环境变量的改变，意义不是特别大
```
```.c
#include <unistd.h>
int execl( const char *pathname, const char *arg0, ... /* (char *)0 */ );
int execv( const char *pathname, char *const argv[] );
int execle( const char *pathname, const char *arg0, ... /* (char *)0, char *const envp[] */ );
int execve( const char *pathname, char *const argv[], char *const envp[] );
int execlp( const char *filename, const char *arg0, ... /* (char *)0 */ );
int execvp( const char *filename, char *const argv[] );
6个函数返回值：若出错则返回-1，若成功则不返回值
exec系列函数可以拉起一个进程
```
# 2.system
```
#include <stdlib.h> 
int system(const char *string);
system 函数通过调用shell 程序/bin/sh –c 来执行string 所指定的命令，该函数在内部是通过调用execve(“/bin/sh”,..)函数来实现的。
通过system 创建子进程后，原进程和子进程各自运行，相互间关联较少。
如果system 调用成功，将返回0。
```
```.c
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
int main(){
    system("sleep 5");
    printf("I am main\n");
    return 0;
}
```
编译执行a.out，然后 ps -elf 查看进程，发现除了 a.out 之外还生成了两个进程 sh -c sleep 5 和 sleep 5
说明 system 执行时会创建一个脚本解析器进程，脚本解析器再创建进程执行脚本。
```.c
继续写一个 system.c
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
int main(int argc,char**argv){
    system(argv[1]);
    printf("I am main\n");
    return 0;
}
```
编译生成a.out，然后，命令行输入
```
sudo chown root:root a.out
sudo chmod u+s a.out
./a.out "sleep 20"
```
然后 ps -elf 查看进程，发现只有 a.out 的有效用户ID变成了root， sh -c sleep 5 和 sleep 5 都是 ichinichi 用户。
注：system 是 fork + exec 
```.c
//写一个 sleep.c
#include<stdio.h>
int main(){
    sleep(100);
    printf("I am Trojan");
}
//写一个 fork.c
#include<stdio.h>
#include<unistd.h>
#include<stdio.h>
int main(){
    if(!fork()){
        printf("I am child\n");
        execl("./sleep","sleep",NULL); 
        while(1);
    }
    else{
        printf("I am parent\n");
        while(1);
    }
    return 0;
}
```
编译生成a.out，然后，命令行输入 chmod u+s a.out
切换用户为 nagi ，执行 a.out ， ps -elf 查看进程，发现父子进程均变为 ichinichi 。
如此利用 fork 和execl 实现了一个类似 system 的功能，当然，自己写的一点安全性没有。
# 3.僵尸进程与孤儿进程
用fork 函数启动一个子进程时，子进程就有了它自己的生命并将独立运行。  
如果父进程先于子进程退出，则子进程成为孤儿进程，此时将自动被PID 为1 的进程（即init）接管。  
孤儿进程退出后，它的清理工作有祖先进程init 自动处理。但在init 进程清理子进程之前，它一直消耗系统的资源，所以要尽量避免。  
如果子进程先退出，系统不会自动清理掉子进程的环境，而必须由父进程调用wait 或 waitpid 函数来完成清理工作，  
如果父进程不做清理工作，则已经退出的子进程将成为僵尸进程(defunct)  
在系统中如果存在的僵尸（zombie）进程过多，将会影响系统的性能，所以必须对僵尸进程进行处理。  
```.c
//造一个僵尸进程
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
int main(){
  	int i=10;
  	pid_t pid;
  	pid=fork();
	  if(pid==0){
    	 	printf("I am zombie\n");
   	}
  	else{
	    	printf("I am parent");
		    while(1);
	  }
	  return 0;
}
```
用 ps -elf 查看进程可以看到子进程状态标识为 Z
```.c
//造一个孤儿进程
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
int main(){
    int i=10;
    pid_t pid;
    pid=fork();
    if(pid==0){
        printf("I am orphan\n");
        while(1);
    }
    else{
        printf("I am parent\n");
    }
    return 0;
}
```
用 ps -elf 查看进程可以看到子进程的ppid已经变为1
# 4.daemon守护进程
Daemon 运行在后台也称作“后台服务进程”。它是没有控制终端与之相连的进程。它独
立与控制终端、会话周期的执行某种任务。那么为什么守护进程要脱离终端后台运行呢？
守护进程脱离终端是为了避免进程在执行过程中的信息在任何终端上显示并且进程也不会
被任何终端所产生的任何终端信息所打断。那么为什么要引入守护进程呢？由于在linux 中，
每一个系统与用户进行交流的界面称为终端，每一个从此终端开始运行的进程都会依赖这
个终端，这个终端就称为这些进程的控制终端。当控制终端被关闭时，相应的进程都会自
动关闭。但是守护进程却能突破这种限制，它被执行开始运转，直到整个系统关闭时才退
出。末尾的字母 d 通常就是指daemon。
```.c
//先看一下进程组id(pgid)
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<sys/types.h>
#include<sys/wait.h>
int main(){
  	if(!fork()){
		    printf("I am child,mypid=%d,ppid=%d,pgid=%d\n",getpid(),getppid(),getpgid(0));
   	}
  	else{
	    	printf("I am parent,mypid=%d,pgid=%d\n",getpid(),getpgid(0));
  	  	wait(NULL);
  	}
	  return 0;
}
```
输出：
I am parent,mypid=2844,pgid=2844  
I am child,mypid=2845,ppid=2844,pgid=2844  
如果一个进程的pid等于pgid，说明它是该进程组组长。  
```.c
//写一个 setpgid.c
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<sys/types.h>
#include<sys/wait.h>
int main(){
	if(!fork()){
		printf("I am child,mypid=%d,ppid=%d,pgid=%d\n",getpid(),getppid(),getpgid(0));
		setpgid(0,0);
		printf("I am child,mypid=%d,ppid=%d,pgid=%d\n",getpid(),getppid(),getpgid(0));
		while(1);
		return 0;
	}
	else{
		printf("I am parent,mypid=%d,pgid=%d\n",getpid(),getpgid(0));
		wait(NULL);
		return 0;
	}
}
```
输出：
I am parent,mypid=2863,pgid=2863  
I am child,mypid=2864,ppid=2863,pgid=2863  
I am child,mypid=2864,ppid=2863,pgid=2864  
setpgid将参数pid指定进程所属的组识别码设为参数pgid指定的组识别码。如果参数pid 为0，则会用来设置目前进程的组识别码，如果参数pgid为0，则由pid指定的进程ID将用作进程组ID。一个进程只能为它自己或它的子进程设置进程组ID。
