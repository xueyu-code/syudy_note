# 文件IO及进程重制版笔记
## 常见IO函数的分类与区别
- 不同系统的文件IO（系统IO 系统调用）有区别
- 而标准IO是任何平台都通用的
- 标准IO吞吐量大，系统IO响应速度快
- 标准IO与系统IO不可混用（此处存疑）
## Linux系统文件类型
*一共有七种bcd-lsp*
- b:块设备文件
- c:字符设备文件
- d：目录文件
- \- 普通文件
- l:链接文件
- s:套接字文件
- p:管道文件
## 标准IO
**概念**：是C库中定义的一组用于输入输出的函数
### 默认打开的3个流
- stdin 标准输入
- stdout 标准输出
- stderr 标准错误
### 缓冲区
#### 缓冲机制的目的
通过缓冲机制减少系统调用
#### 全缓冲
全缓冲：跟文件相关
#### 行缓冲
行缓冲：跟终端相关
#### 不缓冲
没有缓冲区，标准错误就是没有缓冲区，这样能立即输出错误信息
#### 缓冲区刷新条件
缓冲区刷新条件：
- 反斜杠n
- 程序正常退出
- 缓冲区满刷新
- 强制刷新fflush
   - fflush(NULL) 强制刷新所有打开的标准输出
   - fflush(stdout)强制刷新标准输出流

```c
#include <stdio.h>
int main()
{
      printf("test");
      while(1);
}//上述代码并不会打印信息
```
**写成下面几种方式可以完成刷新**
```c
int main()
{
    printf("test");
    fflush(NULL);
    while(1);

   printf("hello");
   fflush(stdout);
   while(1);

   printf("test\n");
   while(1);

   printf("err");
   while(1);
}
```
### 打开关闭文件相关API
#### fopen
```c
FILE *fopen(const char *path, const char *mode)
功能：打开文件
参数：
    path：打开的文件
    mode：打开的方式
        r/rb：只读，当文件不存在时报错，文件流定位到文件开头
        r+/r+b：可读可写，当文件不存在时报错，文件流定位到文件开头
        w/wb：只写，文件不存在创建，存在清空
        w+/w+b：可读可写，文件不存在创建，存在清空
        a/ab：追加(在末尾写),文件不存在创建，存在追加，文件流定位到文件末尾
        a+/a+b：读和追加，文件不存在创建，存在追加，
        读文件流定位到文件开头，写文件流定位到文件末尾
            注：当a的方式打开文件时，写只能在末
            尾进行追加，定位操作是无法改变写的位置，但是可以改变读的位置
返回值：成功：文件流
      失败：NULL，并且会设置错误码
```
#### freopen
#### fclose关闭文件
```c
int fclose(FILE* stream);
功能：关闭文件
参数：stream：文件流
```
### 杂项API
#### perror
```c
void perror(const char *s);
功能：根据errno值打印对应的错误信息
参数：
	 s:你想打印的提示语句
返回值：空
```
```c
#include <stdio.h>
int main(int argc,char *argv[])
{
    FILE *read=fopen(argv[1],"r");
   if (NULL==read)
   {
      perror("Open ERR!!!");
      return -1;
   }
}
//注意此处if条件判断的写法
//如果你写成read==NULL
//这种写法中如果你不小心写为read=NUll;
//那么系统不会报错，就会把一个判断语句变成一个赋值语句
//但是如果写为NULL==read，由于C语言中不能给一个常量赋值
//所以你如果不小心丢掉一个等于号变为NULL=read
//此时编译器会报错提醒
//综上所述这种写法更加安全 更好
```
### 读写操作API
#### fgetc fputc
**每次读写一个字符**
```c
int  fgetc(FILE * stream)
功能：从文件中读取一个字符
参数：stream：文件流
返回值：成功：读到的字符
      失败或读到文件末尾：EOF(-1)
```
```c
int fputc(int c, FILE * stream)
功能：向文件中写入一个字符
参数：c：要写的字符
   stream：文件流
返回值：成功：写的字符的ASCII
      失败：EOF
```
#### fgets fputs
**每次读写一串字符**
```c
char * fgets(char *s,  int size,  FILE * stream);
功能：从文件中每次读取一行字符串
参数：s：存放字符串的地址
         size：一次读取的字符个数         
         stream：文件流
 返回值：成功：s的地址
       失败或读到文件末尾：NULL
特性：每次实际读取的字符个数为size-1个，会在末尾自动添加\0
       每次读一行，遇到\n后不再继续，读下一行
```
### 判断是否读到文件末尾
```c
int  feof(FILE * stream);
功能：判断文件有没有到结尾
返回：到达文件末尾，返回非零值
```
```c
if(feof(fp))//判断当前位置后面还有没有字符
{
    printf("end\n");
}
```
### 检测文件有没有出错
```c
int ferror(FILE * stream);
功能:检测文件有没有出错
返回：文件出错，返回非零值

if(ferror(fp))
{
    printf("err\n");
    return 0;
}
```
#### 一般出错的情况
- 读写文件的权限不够
- 打开文件失败
- 没有打开文件 直接操作
## 文件读写顺序对代码的影响
```c
// int main(int argc, char *argv[])
// {
//    char s[32]={};
//    int count=1;
//    if (argc<2)
//    {
//       printf("请输入文件名\n");
//    }
//    FILE *pp=fopen(argv[1],"r");
//    char a=fgetc(pp);
//       while(a!=EOF)
//       {
//          if (a=='\n')
//          {
//             count++;
            
//          }
//          a=fgetc(pp);
//       }
//    printf("%d\n",count);
//    fclose(pp);
```
以a或者a+操作文件是不支持fseek操作的