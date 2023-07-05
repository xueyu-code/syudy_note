# 文件I/O相关
## I/O体系结构
    设备类型
    总线系统
            总线类型：PCI USB ISA AMBA SBUS等
    文件系统
大体就分为上面三部分
## I/O与kernel的关系
IO的操作是基于文件系统的，linux内核源码中的fs目录就是与文件系统相关的代码，里面包含了各类文件系统的代码。linux在各类文件系统之上又封装了VFS(虚拟文件系统)，用户能看到的也是虚拟文件系统。**C语言的IO读写相关的API会访问系统封装的读写函数**这部分内容，暂时了解即可。
## I/O操作过程
这个操作过程有个大致的了解，明白概念就行，不是这部分知识的重点。
### 打开文件
一个应用程序通过要求**内核**打开对应文件，宣告他要访问一个I/O设备，内核返回一个非负整数，称作描述符，这是文件的唯一标识。
### 改变文件位置
对于每个打开的文件，内核保持一个文件位置k，初始为0，这个位置是从文件头开始的偏移量。执行seek操作，可以设置当前位置为k。
### 读写文件
读操作：从文件拷贝n(n>0)个字节到存储器
写操作：从存储器拷贝n(n>0)个字节到文件
### 关闭文件
通知内核关闭文件，作为响应，内核释放*文件打开时创建的数据结构*

## Linux的I/O操作
### 文件及流的标准输入输出
ANSI C的C库(跨平台)的输入输出方式，所以称做*标准输入输出*，注意这部分操作的都是文件流，类似缓存，不是真正磁盘上的文件。
*抽时间可以了解一下系统分层和系统调用表的知识，目前这部分只做了解。*
**下面列举一些常用标准I/O函数，都是stdio.h中的**
#### fopen
FILE *fopen(const char *filename,const char *mode)
#### fread
size_t fread(void *ptr,siez_t size,size_t nitems,FILE *stream)
#### fwrite
size_t fwrite(const void *ptr,siez_t size,size_t nitems,FILE *stream)
#### fclose
int fclose(FILE *stream)
#### fflush
int fflush(FILE *stream)
#### fseek
int fseek(FILE *stream,long int offset,int whence)
#### fgetc,getc,getchar
int fgetc(FILE *stream)
int getc(FILE *stream)
int getchar()
#### fputc,putc,putchar
int fputc(int c,FILE *stream)
#### fgets,gets
char *fgets(char *s,int n,FILE *stream)
char *gets(char *s)
#### 练习:拷贝文件
要求：将file.in的内容拷贝到file.out里
思路：
- 1 打开两个文件
- 2 循环读取file.in的内容到char (fgetc)
- 3 循环写入char内容到file.out (fputc)
- 4 关闭两个文件fclose
示例：
```c
#include <stdio.h>
#include <string.h>
int main(int argc, char *argv[])
{
    char c;
    FILE *pin,*pout;
    pin=fopen("file.in","r");
    pout=fopen("file.out","w+");
    while ((c=fgetc(pin))!=EOF)//每读取一个字符后，游标Seek会自动往下走一个位置
    {
        fputc(c,pout);
    }
    fclose(pin);
    fclose(pout);
    return 0;
}
```
**一般来说我们是采用标准I/O即ANSI C来开发，这样代码的移植性好。但是如果我们不希望系统的缓存(系统的自动缓存机制)来打扰我们，或者对实时性要求高的时候(例如socket应用)，就要选用POSIX C的底层方式开发**
### 底层输入输出
这部分紧密结合操作系统，是linux系统原生的I/O操作 *(Low-Level Input/Output)* 。是基于POSIX C（嵌入式大部分会和这种不跨平台的打交道）。**这部分详细的知识可以阅读GNU官网(gnu.org)的GNU Lib C的参考手册，GNU官网还有GDB和GRUB的手册。**
#### 打开和关闭文件
下面只是介绍常用API，全部详细内容需要自己查看手册。man手册也可以查出来对应函数的介绍。
##### open操作
用到的头文件
(这些头文件都可以在/usr/include路径下找到，/usr/include/sys也存放了一些)
```c
#include <sys/types.h>//定义了一些flags宏的定义
#include <sys/stat.h>//perms(权限)宏的定义
#include <fcntl.h>
```
函数原型
```c
int open(const char *pathname,int flags,int perms)
```
参数含义:
***pathname**:被打开的文件名，传入一个字符串或者一个数组名(此处存疑)
**flags**：文件打开的方式，主要包含下面几种宏(在types.h中定义)
```c
O_RDONLY  //只能读
O_WRONLY  //只能写
O_RDWR  // 读写打开
O_CREAT //建立文件
O_TRUNC //覆盖操作
O_APPEND //追加内容
```
也可以不用宏代表，直接填写宏对应的数字
**perms**：表示被打开文件的存储权限，前缀为S_I，括号里的内容自由组合
```c
S_I(R/W/X)(USR/GRP/OTH) //可以直接用权限对应的数字表示比如777 666
//括号里的内容都可以作为参数的一部分 前缀固定为S_I
S_IRUSR//该参数就表示usr可以进行读操作
```
返回值
```c
成功返回文件描述符
失败返回-1
```
例子
```c
#include <stdio.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>
//exit会用到stdlib.h这个头文件，最好包含进去
int main(int argc,char *argv[])
{
    if(argc<2)//说明后面没跟参数
    {
        printf("please input filename\n");
        exit(1);//1表示异常退出
    }else
    {
        int fd;//定义一个文件描述符
        //flags这里用|连接两种模式表示没有该文件就创建一个
        fd=open(argv[1],O_RDWR|O_CREAT,0777);//注意这里表示权限的数字是4位
        if (fd==-1)
        {
            printf("ERR!!!!\n");
            exit(1);
        }else
        {
            printf("open ok,fd is %d\n",fd);
        }
    }
    return 0;
}
```
##### umask导致的问题
*此处存疑，我没有遇到这个问题，可能是因为我是root用户吧*
如果在代码中把0777换成666，生成的文件权限不会是666权限，这是因为*umask权限掩码*的存在，实际的权限是你设定的权限减去umask的值。乌班图默认的umask数值是0002，新创建的文件将默认为644权限，新创建的目录将默认为755权限。
```sh
umask 0000
#从左往右第一位表示GID/PID用来屏蔽用户或者是组
#第二位表示usr权限
#第三位表示grup所在的组权限
#第四位表示other其他用户权限
```
输入上述指令修改掩码全部为0，
```sh
umask
```
执行后会输出umask的值，
**所以在代码里要填四位，以免搞混**
##### 代码中设置umask
在代码中直接调用函数umask即可
```c
    #include <sys/types.h>//需要用到这两个头文件
    #include <sys/stat.h>
    umask(0000);//设置为0000
```
```sh
man umask 
```
可以man手册看一下设置umask函数的详细介绍。
##### close操作
用到的头文件
```c
#include <unistd.h>
```
函数原型
```c
int close(int fd)
```
参数含义:
**fd**:想要关闭的文件所对应的文件描述符
返回值
```c
成功返回0
失败返回-1
```
#### 读写文件
##### read操作
用到的头文件
```c
#include <unistd.h>
```
函数原型
```c
ssize_t read(int fd,void *buf,size_t count)
```
参数含义:
**fd**:被打开的文件所对应的文件描述符
**buf**：指定存储器读出数据的缓冲区（一般是内存的地址空间）
**count**:指定读出的字节数
返回值
```c
成功 返回读到的字节数
0 已经到达文件尾
-1 失败
```
示例
```c
#include <stdio.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
int main(int argc,char *argv[])
{
    if(argc<2)//说明后面没跟参数
    {
        printf("please input filename\n");
        exit(1);//1表示异常退出
    }else
    {
        int fd;//定义一个文件描述符
        fd=open(argv[1],O_RDWR|O_CREAT,0666);
        if (fd==-1)
        {
            printf("ERR!!!!\n");
            exit(1);
        }else
        {
            printf("open ok,fd is %d\n",fd);
            //进行读文件
            char buf[1024];//此处只是定义，但是没有给他开辟空间
            memset(buf,0,1024);//开辟空间
//memset需要引用头文件string.h，至于为什么用memset请参考libc的手册内存部分
            int return_num=read(fd,buf,1024);
            //1024这个地方也可以换成自己想要读出的个数(小于1024)
            if (return_num!=-1)//不等于-1代表成功读出
            {//输出读取的内容
                printf("%s\n",buf);
            }else
            {
                printf("Read ERR!!\n");
            }
            close(fd);
        }
    }
    return 0;
}
```
##### write操作
用到的头文件
```c
#include <unistd.h>
```
函数原型
```c
ssize_t write(int fd,void *buf,size_t count)
```
参数含义:
**fd**:被打开的文件所对应的文件描述符
**buf**：指定存储器写入数据的缓冲区
**count**:指定写入的字节数

返回值
```c
成功 返回写入的字节数
-1 失败
```
#### 设置文件位置
##### lseek函数
用到的头文件
```c
#include <unistd.h>
#include <sys/types.h>
```
函数原型
```c
off_t lseek(int fd,off_t offset,int whence)
```
参数含义:
**fd**:文件描述符
**buf**:偏移量，单位是字节，**可正可负**
**whence**:当前位置的基点
```c
SEEK_SET 文件头
SEEK_CUR 当前位置
SEEK_END 文件尾
```
返回值
```c
成功 返回相对于文件头的偏移量
-1 失败
```
*示例*
```c
#include <stdio.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
int main(int argc,char *argv[])
{
    if(argc<2)//说明后面没跟参数
    {
        printf("please input filename\n");
        exit(1);//1表示异常退出
    }else
    {
        int fd;//定义一个文件描述符
        fd=open(argv[1],O_RDWR|O_CREAT,0666);
        if (fd==-1)
        {
            printf("ERR!!!!\n");
            exit(1);
        }else
        {
            printf("open ok,fd is %d\n",fd);
            //进行写文件
            char buf[]="write test";
            int return_num=write(fd,buf,strlen(buf));
            if (return_num!=-1)//不等于-1代表成功写入
            {//输出写入的内容
                printf("%s\n",buf);
            }else
            {
                printf("Write ERR!!\n");
            }
            //利用lseek得到整个文件的长度
            int file_len=lseek(fd,0,SEEK_END);//文件尾部偏移0个位置就是文件长度
            printf("len is %d\n",file_len);
            //重新偏移然后读出来,看看效果如何
            lseek(fd,5,SEEK_SET);//偏移5个位置
            char buf2[1024];
            memset(buf2,0,1024);
            int read_num=read(fd,buf2,1024);
            if (read_num!=-1)
            {
                printf("%s\n",buf2);
            }else
            {
                printf("read ERR!!\n");
            }
            close(fd);
        }
    }
    return 0;
}
```
#### 练习:2个文件内容拷贝
*要求*：将文件A的内容拷贝到文件B里
*思路*：
- 打开两个文件
- 循环读取A的内容到char
- 循环把char内容写入到B
- 关闭两个文件

*示例*
```c
#include <stdio.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
int main(int argc, char *argv[])
{
    int fd_in, fd_out;
    char c;
    fd_in=open("A",O_RDONLY,0666);
    fd_out=open("B",O_WRONLY|O_CREAT,0666);
    while (read(fd_in,&c,1))
    {
        write(fd_out,&c,1);
    }
    close(fd_in);
    close(fd_out);
    return 0;
}
```
#### 文件符号和流
#### 快速聚集I/O
#### 内存映射I/O
#### 同步I/O操作
#### 文件锁定
假设有两个进程A和B，在A对某个文件的操作过程中，这个文件应该是不能被其他进程访问的，否则最后的文本信息就混乱了。文件锁定可以*文件整锁（整个文件都不允许被其他进程操作）*也可以*记录锁（文件部分锁定，如只锁定文件的第2-3行）*。文件锁定分为两大类，强制锁与建议锁。
**强制锁概念**
```
给某个文件上锁之后，是由内核空间来支持的。
而且在上锁之后，所有的内核函数访问这个文件的时候，都要判断一下这个锁的状态。
```
**建议锁概念**
```
是由用户空间支持的。但是这种只是对这个文件打一个上锁的标识。
函数每次访问这个文件是否要判断这个锁由用户的代码控制。
这种方式有漏洞，因为你判断这个文件的锁状态是在用户的进程代码里控制的，
假设你在A进程代码里判断了，但是B进程通过open或者其他内核函数还是能访问这个文件
```
**fcntl函数**
```
这个函数不仅能实现文件加锁，还能实现修改和复制文件描述符等多种功能
```
*依赖头文件*
```c
#include <unistd.h>
#include <fcntl.h>
```
*函数原型*
```c
int fcntl(int fd,int cmd,struct flock *arg);
```
*参数含义*
**fd**:文件描述符
**cmd**:控制命令，该参数不止下面三个，但是和文件锁相关的只有下面三个
- F_GETLK  得到锁
- F_SETLK  设置锁
- F_SETLKW 设置锁并等待返回

**(struct flock *arg)**：加锁结构体的指针
```c
//加锁结构体的内容
struct flock
{
    short l_type;//锁类型 F_RDLCK申请 读锁（也叫共享锁）
//共享锁 例如一个进程锁定了某个文件，但是另一个进程也可以打开它就是不能写入
                //F_WRLCK 申请 写锁（也叫独占锁）
//独占锁 例如一个进程锁定某个文件后，别的进程连打开这个文件都不行
                //F_UNLCK 释放锁
    short l_whence;//锁区域开始地址的 相对位置 类似lseek中的whence参数
                    //SEEK_SET 相对文件起始位置
                    //SEEK_CUR 相对文件当前位置
                    //SEEK_END 相对文件结束位置
    long l_start;//锁区域开始的地址偏移量，和I_whence共同设置锁区域
    long l_len;//锁的长度，0表示锁到文件末
    long l_pid;//拥有锁的进程ID
}
```
**默认用的是建议锁，如果用强制锁，需要手动挂载**
##### 文件锁练习
*要求；*用文件锁的相关函数，设置一个文件为独占模式，然后在控制台通过echo修改文件，测试文件锁的效果
*示例1*
```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/stat.h>
#include <fcntl.h>
int main()
{
    int fd=open("hello.txt",O_RDWR|O_CREAT,0666);
    if (fd>0)//fd>0表示正确打开
    {
        //定义锁定结构体
        struct flock lock;
        lock.l_type=F_WRLCK;//设置为独占锁(排他锁 互斥锁)
        lock.l_whence=SEEK_SET;
        lock.l_start=0;
        lock.l_len=0;
        lock.l_pid=getpid();//getpid函数可以得到当前进程的PID
        //锁定文件
        int lock_num=fcntl(fd,F_SETLK,&lock);
        printf("return value of lock %d\n",lock_num);
        while(1);//为了演示这个锁定的作用，我们不能让这个锁消失
                //所以不能结束这个进程，否则锁会自动释放
                //让他在这里死循环
    }
    return 0;
}
```
*现象：*运行代码之后1号控制台显示程序一直在运行，进程没结束。重新打开另一个控制台，输入
```sh
echo file lock >> hello.txt
cat hello.txt
```
此时发现文件已经被修改，原因是**默认是建议锁，open这种内核级的函数操作的时候不会去判断这个文件的状态，需要用户手动写代码控制**
*示例2(强制锁)*
如果我们不在上述的代码中添加判断操作，就需要开启强制锁。
```sh
man fcntl #Mandatory locking部分
```
经过查阅man手册得知开启强制锁，需要重新挂载文件系统，并修改操作文件的权限
```sh
sudo mount -oremount,mand / #重新挂载根目录使其支持强制锁
sudo chmod g+s,g-x hello.txt
```
开启强制锁需要在文件系统加载的时候指定开启强制锁，并设置对应文件的group-id。(此处存疑)
设置完之后，此时再执行修改文件的echo命令，会进入阻塞状态，退出程序才能正常修改。
**有空整理一下echo命令相关的知识**
#### 中断驱动输入

### 文件系统接口
### 管道及FIFO(先入先出队列)
### Socket
这部分是比较特殊的I/O。这部分先放到网络编程中学习。
### 底层终端接口(tty)

### perror错误机制
系统级调用函数失败之后会设置外部变量errno的值来指明错误原因，不同的错误原因所对应的错误码也不同。*可以用perror函数将最新的errno值输出*
*依赖头文件*
```c
#include <stdio.h>
```
*函数原型*
```c
void perror(const char *s)
```
*示例*
```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/stat.h>
#include <fcntl.h>
int main()
{
    int fd=open("hello",O_RDONLY,0666);
    if (fd<0)//fd<0表示不能正常打开
    {
        perror("open err");//这里只是相当于传一个标题的作用
                            //具体的错误原因系统会自动输出
    }
    return 0;
}
```
此时在执行目录下没有hello文件，执行该程序之后，会显示
```sh
open err: No such file or directory
```
### 文件和目录
#### 文件和目录的维护
例如 修改权限或者删除目录。
*这部分的函数名字和shell命令一样，其实shell在执行过程中也会调用libc库*
##### chmod
*依赖头文件*
```c
#include <sys/stat.h>
```
*函数原型*
```c
int chmod(const char *path,mode_t mode)
```
*返回值*
```
成功返回0
失败返回-1
```
*man查阅的小细节*
如果你直接输入
```sh
man chmod
```
那么此时man显示的是shell命令的chmod介绍，从标题也能看出来
```sh
User Commands #用户命令
```
如果想查看libc中*chmod函数*的介绍应该这样，先输入
```sh
man chmod
```
翻到下面找到这样的信息
```sh
SEE ALSO
       chmod(2)

       Full documentation at: <http://www.gnu.org/software/coreutils/chmod>
       or available locally via: info '(coreutils) chmod invocation'
```
他会提示你如果想查阅更多信息要么输入2参数，要么去官网，接着输入
```sh
man 2 chmod #这个才是函数的介绍
```
*示例*
```c
#include <stdio.h>
#include <sys/stat.h>
#include <sys/types.h>
int main()
{        
         umask(0000);                                                  
         int rt=chmod("hello",0777);//修改hello文件的权限为777
         if(rt==0)
                 printf("OK\n");
 } 
```
##### chown
##### unlink
*示例*
```c
#include <stdio.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>
int main()
{                                           
         int rt=unlink("hello");
         if(rt==0)
                 printf("OK\n");
 } 
```
执行这个代码之后，hello文件就会被删除。
减少链接数，在linux下面删除文件其实就是靠链接数，如果链接数为0，说明文件删除了。
##### mkdir/rmdir

#### 扫描目录
例如扫描目录并打印
##### opendir
*依赖头文件*
```c
#include <sys/types.h>
#include <dirent.h>
```
*函数原型*
```c
DIR *opendir(const char *name)
```
*返回值*
```
失败返回 NULL
成功返回 DIR这个描述文件夹的结构体
```
##### readdir
*依赖头文件*
```c
#include <sys/types.h>
#include <dirent.h>
```
*函数原型*
```c
struct dirent *readdir(DIR *dirp);
```
*dirent结构(目前只了解)*
```c
           struct dirent {
               ino_t          d_ino;       /* Inode number */
               off_t          d_off;       /* Not an offset; see below */
               unsigned short d_reclen;    /* Length of this record */
               unsigned char  d_type;      /* Type of file; not supported
                                              by all filesystem types */
               char           d_name[256]; /* Null-terminated filename */
           };
```

##### closedir
*依赖头文件*
```c
#include <sys/types.h>
#include <dirent.h>
```
*函数原型*
```c
int closedir(DIR *dirp);
```
*返回值*
```
成功返回0
失败返回-1
详细讲解，请查看man手册
```
### 文件夹练习
*要求*：扫描当前路径下的所有文件和文件夹，并打印出这个目录下所包含的文件与文件夹名字

*示例*
```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <dirent.h>
#include <sys/stat.h>
#include <fcntl.h>
int main()
{
    DIR *dp;//类比文件操作，先定义一个描述文件夹的指针
    struct dirent *ep;//readdir函数需要这个参数
    //打开当前文件夹
    dp=opendir("./");
    if (dp!=NULL)//证明打开成功
    {
        //开始读取
        while (ep=readdir(dp))
        {
            puts(ep->d_name);
        }
        
    }else
    {
        perror("opendir erroe");
    }
    closedir(dp);
    return 0;
}
```
### proc文件系统
传统的文件系统是用于块设备上的信息存储，而proc文件系统包含如下信息
```
内存管理
系统进程特征数据
文件系统
设备驱动程序
系统总线
电源管理
终端
系统控制参数
```
**我们不止可以在这个proc文件夹下查看各类底层信息，还能在这修改系统参数**
在linux系统下输入
```sh
df -T 此处填写文件夹路径
```
这样就能看对应文件夹是什么文件系统了，
*在根目录下并不是所有文件夹的文件系统都是ext4*
**proc文件夹是虚拟文件系统（proc文件系统），进入该文件夹发现里面包含很多数字命名的子文件夹，这些表示当前运行的进程，每个进程文件夹里包含进程信息**









## 涉及到的数据结构
### FD(重点)
对于内核而言，所有打开文件都由文件描述符引用，**打开这个文件的唯一标识，这个标识可以标识出一个文件访问入口地址。**
文件描述符是一个非负整数，当打开一个现存文件或者创建一个新文件时，内核都向进程返回一个文件描述符。当读写一个文件时，用open或者create返回的文件描述符标识该文件，将其作为参数传送给read或write。在POSIX应用程序中，文件描述符为
- STDIN_FILENO  (0 标准输入)
- STDOUT_FILENO (1 标准输出)
- STDERR_FILENO (2 标准出错输出)
这几个常数定义在*unistd.h*中。文件描述符的范围是0-openmax，32位linux系统是65535，64位系统更多。
### File(了解性)
该结构体下面包含了众多信息，篇幅有限所以不一一列举。包含文件对应的目录结构 文件大小 文件模式等信息
### Files Structure(了解性)
该结构体下面包含了众多信息，篇幅有限所以不一一列举。包含 打开的fd集合 下一个空闲的fd 最大的fd集合容量等信息


一个数(a)与另一个数(b)相与如果能得到另一个数证明a是1 



