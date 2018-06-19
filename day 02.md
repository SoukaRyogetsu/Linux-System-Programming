# 1.tree
```  
tree  列出目录下的所有文件。
tree -a  列出目录下的所有文件(包括隐藏文件)。
tree -d  仅显示目录名。
tree -h  列出目录下的所有文件并显示大小。
```  
# 2.chmod  
```  
chmod [who][+|-|=][mode] [文件名]  修改指定文件名中who的权限增加/去除/赋值为mode。
eg：chmod u+x echo.sh  给当前用户添加 echo.sh 的可执行权限。
eg：chmod o-w echo.sh  去除其他用户对 echo.sh 的写入权限。
eg：chmod +r echo.sh  给所有用户增添 echo.sh 的读取权限。
chomod [八进制] [文件] 数字设定法设置权限。
eg： chmod 664 echo.sh  设置 echo.sh 当前用户可读可写， 同组用户可读可写，其他用户只读(rw-rw-r--)。
注：使用 vim echo.sh 在当前目录下创建一个shell脚本，写完后先按"Esc"键表示退出编辑，再依次按":","x"返回到命令行界面。
注：使用 ./echo.sh 运行脚本，"./"不能少。
注：新建文件默认权限666，，文件夹默认权限777，由于掩码关系(使用 umask 查看掩码)，表现为664、774。
```  
# 3.find  
```  
find [起始目录] [查找条件] [操作]
eg：find /usr/include/ -name stdio.h  在系统include文件下按名字查找 stdio.h 文件。
eg：find . -name "file"  在当前目录下查找名为"file"的文件。
eg：find . -name "file" -a -type d  在当前目录下查找名为"file"的目录。
eg：find . -name "file" -a -type f  在当前目录下查找名为"file"的文件(不包含目录)。
eg：find /usr/include/ -name stdio.h -o -name stdlib.h  在系统include文件下按名字查找 stdio.h 文件和 stdlib.h 文件。
eg：find . -name "*.c"  在当前目录下查找所有后缀为 .c 的文件。
eg：find . -name "tes?.c"  在当前目录下查找所有满足文件名为 tes?.c 的文件，其中，? 可以代表任意 "一个" 字符。
eg：find . -name "tes[1-3].c"   在当前目录下查找文件名 tes1.c 或 tes2.c 或 tes3.c 的文件。
eg：find . -empty  在当前目录下查找空目录或空文件。
eg：find . -perm 664  在当前目录下查找权限为 664 的文件。
eg：find . -empty |xargs ls -l  在当前目录下查找空目录或空文件并显示详细信息。
eg：find /usr/include/ -name stdio.h |xargs ls -l  在系统include文件下按名字查找 stdio.h 文件并显示详细信息。
注：显然，查找条件可以写的很复杂。
注：Linux认为目录(文件夹)也是文件，同一目录内不允许文件名和子目录重名，但Windows可以。
```
# 4.df
```
df -h  查看磁盘空间，以人们易读的GB、MB、KB 等格式显示磁盘使用情况。
```
# 5.du
```
du -h  以人们易读的GB、MB、KB 等格式显示每一级目录使用情况。
du -h --max-depth=0  只显示当前目录下，不显示子目录。
eg：du -h .  
eg：du -h --max-depth=0 test2
```
# 6.cat
```
cat [选项] [文件]  查看文件的内容
cat -b  对非空行输出行号。
cat -n  对每一行输出行号。
cat -s  不输出多行空行，用处大概是在网上抄的代码格式太烂，空行太多，用这个规范一下，瞎猜的。
cat -e  在每行结束处显示$，用处大概是在网上抄代码编译不通可以看看是否有看不到的特殊字符。
eg：cat -b file.c
注：别忘了写文件名。
注：Ctrl + A 光标移到行首，Ctrl + E 移到行尾。
注：用vim写一个main.c步骤：首先， vim main.c ，其次，gcc main.c ，最后， ./a.out 。 
```
# 7. > <
```
> 重定向标准输出
eg：./aout > file  将 a.out 的标准输出写入 file ，不会把标准错误输出写进去。
eg：./aout 2> file1  将 a.out 的标准错误输出写入 file1 。
eg：./a.out > file2 2>&1  将 a.out 的标准错误输出和标准输出写入 file2 。
注：标准错误输出位于非缓冲区，打印到屏幕的速度比标准输出要快。哪怕标准错误输出在标准输出的后面，在屏幕上也可能显示在前面。
eg：echo hello >> file2  将 "hello" 写入 file2 末尾。
eg：cat >file3<file2  将 file2 内容写入 file3 。
eg：cat file3>file4  将 file3 内容写入 file4 。
注：cat 可以自动新建文件
```
# 8.创建空文件的几种方法
```
echo > a.txt (a.txt 有大小，为一个字节)
touch b.txt
cat > c.txt  按 ctrl+c 组合键退出；或 ctrl+d 。
vi d.txt  进入之后，输入 ":wq" 退出。
```
# 9.head tail
```
head/tail -[数字] [文件]  显示文件的前(后)几行。
eg：head -5 main.c  显示当前目录下 main.c 的前5行。
eg：history | tail -10  显示最近输入的10条命令。
eg：history | tail -20 >t_his.txt  将最近输入的20条命令写入 t_his.txt 中。
```
# 10.more less
```
eg：more/less The_Holy_Bible.txt  打开当前目录下《圣经》的前几页，滑动看文件。
```
# 11.sort
```
eg：sort name  对文件 name 的行按照ASCII码表进行升序排序，但不会重新写回文件。
eg：sort name > name2  将 name 排序后的结果写入 name2 。
```
# 12.file
```
file [文件]  查看文件内容类型。
eg：file The_Holy_Bible.txt  
注：上一行命令输出信息 "The_Holy_Bible.txt: ASCII text, with very long lines, with CRLF line terminators" 。
```
# 13.uniq
```
unic -c  在输出行前面加上每行在输入文件中出现的次数.
unic -d  仅显示重复行。
unic -u  仅显示不重复的行。
eg：uniq name2  将 name2 文件中相邻的重复的内容删去。
eg：sort name2 |uniq >name3  将 name2 文件中重复的内容删去并写入 name3 。
eg： uniq -c name2
eg： uniq -d name2
```
# 13.wc
```
wc [文件]  统计指定文件中的行数、字数、字节数。
wc -c 统计字节数。
wc -l 统计行数。
wc -m 统计字符数。这个标志不能与 -c 标志一起使用。
wc -w 统计字数。一个字被定义为由空白、跳格或换行字符分隔的字符串。
eg：ls|wc -1  统计当前目录下文件及子目录数目。
eg：find . -name "*"|wc -1  统计当前目录下所有文件及所有子目录连同内文件数目。
eg：wc -l The_Holy_Bible.txt  统计《圣经》行数(32432)。
```
# 14.iconv
```

```
