# 1.Makefile
日常工作中我们常使用Makefile实现多文件编译和增量编译。  
先写两个.c  
```.c
//vim maic.c
#include <stdio.h>
void print();
int main(){
   printf("I am printf\n");
   print();
}
//vim print.c
#include <stdio.h>
void print(){
   printf("I am print\n");
   return;
}
```
再写Makefile 
首先： vim Makefile  ( M 要大写)
下面写 Makefile 里的内容
```
main:main.c print.c
    gcc main.c print.c -o main
```
注: gcc 前面一定要是一个 TAB 键，不能是四个空格。  
保存回到命令行，输入 make ，系统自动执行 gcc main.c print.c -o main  
再输入 make ，系统提示 main 已经是最新，不执行命令。  
这是因为 main 文件生成的时间比 main.c print.c 的最后更新时间都要晚，没有必要重新编译。  
也就是说，Makefile能根据文件时间自动发现更新过的文件而减少编译的工作量。  

改写Makefile
```
main:main.o print.o
    gcc main.o print.o -o main
main.o:main.c
    gcc -c main.c
print.o:print.c
    gcc -c print.c
```
这样，如果我们只改写 main.c 或者 print.c 中的一个，那么另一个将不会被编译，节省时间。

写一个清理，利用没有依赖项的伪目标 .PHONY
```
main:main.o print.o
    gcc main.o print.o -o main
main.o:main.c
    gcc -c main.c
print.o:print.c
    gcc -c print.c
.PHONY:clean
clean:
    rm -rf print.o main.o main
```
保存退到命令行，输入 make clean 即把 print.o main.o main 三文件删除
然而事实上，.PHONY:clean 完全可以省去不写，直接写一个 clean: ,因为只要不写依赖性，默认是伪目标。
如果不用系统默认的文件名Makefile，而是用户随便起的一个名字，
如：vim Makefile11  
则make 后面必须要加上-f Makefile11 ，如：  
make –f Makefile11 clean //表示执行clean: 开始的命令段。  
make –f Makefile11 main.exe //表示执行main.exe: 开始的命令段。  

改写 Makefile 文件  
用户自定义变量  
Makefile 可以自定义变量，然后用 $(自定义变量，名字自己取) 可以引用
```
OBJS:=main.o print.o
ELF:=main
$(ELF):$(OBJS)
    gcc $(OBJS) -o $(ELF)
main.o:main.c
    gcc -c main.c
print.o:print.c
    gcc -c print.c
.PHONY:clean
clean:
    rm -rf $(OBJS) $(ELF)
```
测试，可以正常运行。

自动变量
$^  表示当前规则的所偶依赖文件
$@  当前规则的目标文件
```
OBJS:=main.o print.o
ELF:=main
$(ELF):$(OBJS)
    gcc $^ -o $@
main.o:main.c
    gcc -c $^
print.o:print.c
    gcc -c $^
.PHONY:clean
clean:
    rm -rf $(OBJS) $(ELF)
```
Makefile 判定依赖文件为 .o 时，会自动寻找同名的 .c，然后自动编译成 .o，是一种隐含规则。
所以 Makefile 文件可简写一下
```
OBJS:=main.o print.o
ELF:=main
$(ELF):$^
    gcc $^ -o $@
clean:
    rm -rf $(OBJS) $(ELF)
```
这样退出再编译，会显示使用 cc 编译的，而不是 gcc ，如果想用 gcc ，可以使用预定义变量
预定义变量
改写上面的 Makefile
```
OBJS:=main.o print.o
ELF:=main
CC:=gcc
$(ELF):$^
    gcc $^ -o $@
clean:
    rm -rf $(OBJS) $(ELF)
```
还有一个问题，如果我们用 gdb 加载这样写出来的 main ，是没有调试信息的。  
如果我们要加调试信息，也要预定义
```
OBJS:=main.o print.o
ELF:=main
CC:=gcc
CFALGS:=-g -Wall
$(ELF):$^
    gcc $^ -o $@
clean:
    rm -rf $(OBJS) $(ELF)
```
重新编译生成 main ，命令行输入 gdb main 可以调试。  。

如果我们想把整个目录下的 .c 一起编译， 且不同时刻目录下 .c 文件数目名字都不一样，又只想写一个 Makefile应万变，这时需要使用模式规则
```
SOURCES:$(wildcard *.c)
OBJS:=$(patsubst %.c,%.o,$(SOURCES))
ELF:=main
CC:=gcc
CFALGS:=-g -Wall
$(ELF):$^
    gcc $^ -o $@
clean:
    rm -rf $(OBJS)
```
wildcard 搜索当前目录下的文件名，展开成一列所有符合由其参数描述的文件名，文件间以空格间隔。  
SOURCES = $(wildcard *.cpp) 把当前目录下所有 ".cpp" 文件存入变量 SOURCES 里。  
字符串替换函数：$(patsubst 要查找的子串,替换后的目标子串，源字符串)。  
将源字符串(以空格分隔)中的所有要查找的子串替换成目标子串。  
如OBJS = $(patsubst %.cpp,%.o,$(SOURCES)) 把SOURCES 中'.cpp' 替换为'.o' 。  

这样，我们就把多个 .c 编译成一个可执行二进制。
注：一个可执行二进制并不等价于一个进程，二者无内在联系

# 2.基于文件指针的文件操作
基于文件指针的文件操作，就是标准C
linux 中对目录和设备的操作都是文件操作，文件分为普通文件，目录文件，链接文件和设备文件。
打开文件模式 ab+ 与 rb+ 的区别

写一个 fopen.c
```.c
#include <stdio.h>
int main(int argc,char** argv){
    if(argc!=2){
        printf("error args\n");
        return -1;
    }
    FILE *fp;
    fp=fopen(argv[1],"rb+");
    if(NULL==fp){
        perror("fopen");
        return -1;
    }
    char buf[128]={0};
    fread(buf,sizeof(char),5,fp);
    puts(buf);
    fwrite("hello",sizeof(char),5,fp);
    fclose(fp);
    return 0;
}
```
然后 vim file ，内容是"helloworld" ，编译程序，file 内容变为 "hellohello"。
这意味着文件读取前五个字符后，直接从光标处开始覆盖性质的写。
将 file 内容再改回来，然后将模式改为 "ab+" ，编译程序， file 内容变为 "hellohello(换行)hello"。
换行是因为系统自动在文末增加换行符，第二行出现"hello" 说明 "ab+" 模式进入写状态时光标自动移动到文末。

sscanf与sprintf
复习一下
```.c
include <stdio.h>                                                                                 
typedef struct{
    int num;
    char name[20];
    float score;
}stu;
int main(){
    stu s={1001,"nagi",98.5};
    char buf[128]={0};
    sprintf(buf,"%d %s %5.2f",s.num,s.name,s.score);
    puts(buf);
    return 0;
}         
```
```.c
#include <stdio.h>                                                                                 
typedef struct{
    int num;
    char name[20];
    float score;
}stu;
int main(){
    stu s={0};
    char buf[128]="1001 nagi 98.5";
    sscanf(buf,"%d%s%f",&s.num,s.name,&s.score);
    printf("%d %s %5.2f\n",s.num,s.name,s.score);
    return 0;
}
```
写一个 chmod.c  
```.c
#include <stdio.h>
#include <sys/stat.h>                                                           
int main(int argc,char** argv){
    if(argc!=2){
        printf("error args\n");
        return -1; 
    }   
    int ret;
    ret=chmod(argv[1],0777);
    if(ret==-1){
        perror("chmod");
        return -1; 
    }   
    return 0;
}
```
```
gcc chmod.c
touch file
./a.out file
然后在命令行 ls -l 查看到 file 文件的权限变为 777
```
可以使用 echo $? 判断前面函数的返回值，为0则正常


写一个 getcwd.c 打印当前目录路径
```.c
#include <unistd.h>
#include <stdio.h>
int main(){
    char path[512]={0};
    getcwd(path,sizeof(path));
    puts(path);                                                                
}
```
或者这样写
```.c
#include <unistd.h>
#include <stdio.h>                                                             
int main(){
    char *path;
    path=getcwd(NULL,0);
    printf("path=%s\n",path);
    return 0;  
}
```
编译输出当前目录的绝对路径  
还有一种方法改变当前目录进行路径输出 
```.c
#include <unistd.h>
#include <stdio.h>
int main(int argc,char **argv){
    if(argc!=2){
        printf("error args\n");
        return -1;          
    }   
    int ret;
    ret=chdir(argv[1]);
    if(ret==-1){
        perror("chdir");
        return -1; 
    }   
    char *path;
    path=getcwd(NULL,0);
    printf("path=%s\n",path);                                                  
    return 0;  
}
```
一些命令和接口函数名字一样的，是用了系统调用。
写一个 mkdir.c 
```.c
#include "func.h"                                                              
int main(int argc,char **argv){
    args_check(argc,2);
    int ret;
    ret=mkdir(argv[1],0777);
    if(-1==ret){
        perror("mkdir");
        return -1; 
    }   
    return 0;
}
```
写一个 rmdir.c 
```.c
#include "func.h"                                                              
int main(int argc,char **argv){
    args_check(argc,2);
    int ret;
    ret=rmdir(argv[1]);
    if(-1==ret){
        perror("rmdir");
        return -1; 
    }   
    return 0;
}
```
其中，头文件 func.h
```.c
#include <stdio.h>                                                             
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>
#define args_check(a,b){if(a!=b) {printf("error args\n");return -1;}}
```

遍历当前目录内文件
```.c
#include "func.h"     
#include <dirent.h>  //之后就把这个写入 func.c 了
int main(int argc,char **argv){
    args_check(argc,2);
    DIR *dir;
    dir=opendir(argv[1]);
    if(dir==NULL){
        perror("opdir");
        return -1; 
    }   
    struct dirent *p; 
    while((p=readdir(dir))!=NULL){
        printf("ino=%ld, d_reclen=%d d_type=%d d_name=%s\n",p->d_ino,p->d_reclen,p->d_type,p->d_name);
    }   
    closedir(dir);
}
```
对当前目录进行深度优先搜索
```.c
#include "func.h"
void printdir(char *path,int width){
    DIR *dir;   
    dir=opendir(path);    
    if(dir==NULL){
        perror("opendir");
        return ; 
    }
    struct dirent *p;
    char buf[512]={0};
    while((p=readdir(dir))!=NULL){
        if(!strcmp(p->d_name,".")||!strcmp(p->d_name,".."))
        {
        }
        else {  
            printf("%*s%s\n",width,"",p->d_name);   
            if(p->d_type==4){
                sprintf(buf,"%s%s%s",path,"/",p->d_name);//路径拼接
                printdir(buf,width+4);
            }
        }
    }
    closedir(dir);
}
int main(int argc,char **argv){
    args_check(argc,2);
    printdir(argv[1],4);
    return 0;
}        
```
