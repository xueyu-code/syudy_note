# Linux内核简单分析
**linux内核与linux系统不是一个概念**
## 版本命名方式
    linux系统采用A.B.C.D的版本号管理方式
    A表示linux的主版本号
    B表示linux的次版本号，B为偶数表示稳定版本，奇数表示开发中的版本
    C表示linux的发行版本号
    D表示更新版本号
## 内核的核心功能
    进程管理
    内存管理
    文件系统
    网络协议
    设备管理
**linux内核不是一个真正的实时系统，采用时间片轮转的方式实现的，，但是vxwork是真正的实时系统**
## 编译内核相关
    make uImage 编译内核 
    make dtbs 编译设备树 3.14版本之后的内核只编译内核是不够的，还需要编译设备树
    
***注意不要在windows下共享目录下进行编译，因为windows不支持符号文件***
**编译的时候一定要设置正确的交叉编译链，如果在命令行输入交叉编译链工具的名字能自动补全说明交叉编译工具安装成功**
### 编译后的文件位置
    默认是在内核源码目录下的arch/arm/boot 编译完的uImage dts都在这里面
## 各个内核Image 的不同点
    uImage 一般是用的最多的，里面有一些uboot的信息，专门针对uboot的，本身也是压缩的，不含调试信息
    zImage 是内核压缩的一种形式
    Image 是未压缩的
**上述3个文件都在内核/boot目录下**
kenel目录下会放一些核心的调度算法等源码
***移植一般只关心两个目录 arch 和 drivers***
## 抽空搜索一下内核各个目录是存放什么文件的，以及那些目录做到了和平台无关
## 嵌入式系统启动分析
### 第一阶段 U-boot启动
    以 Starting kernel.... 为分界线，该语句上面打印的log都是uboot阶段
#### 阶段一（汇编）
    设置为SVC模式，关闭中断，MMU,看门狗 //为了保持uboot能稳定初始化所以关掉这些
    基本硬件初始化//初始化 时钟 串口 flash 内存
    自搬移（自己转移自己）到内存//提高运行速度
    设置好栈 跳转到C阶段
#### 阶段二（C语言）
    大部分硬件初始化
    搬移内核到内存//提高运行速度
    运行内核
### 第二阶段 linux内核启动
    一般以VFS：Mounted root(nfs filesystem) on device 0:10 为分界线
    该信息上面都是内核启动的信息
#### 自解压内核
    decompess 代码位置 arch/arm/boot/compressed/head.s
#### 运行内核汇编部分
    head.s入口stext 位置arch/arm/kernel/head.s
    检测合法性（cpu类型，机器类型）
#### 运行内核C部分
    start_kernel 位置init/main.c
    cpu ,机器参数的安装 setup_arch
    中断 定时 终端 内存等最基本的初始化
    创建核心进程 kernel_init运行，启动多任务调度
#### 挂载rootfs
#### 运行第一个应用程序init(一般是linuxrc)
### 第三阶段 根文件系统启动
    到这个阶段结束才能运行应用程序
**板子上电的启动log一定要学会分析**
**内核代码暂时还没有研究，上述只是一个大致流程**
### 内核和uboot为什么都要对中断 内存等进行初始化
    uboot初始化是直接用的实际的物理地址，而内核是不能直接操作物理地址的，而是虚拟地址空间。这两个初始化是不一样的，所以内核为了安全性要自己初始化一遍
## 内核调试方法
**在内核移植过程中出现问题，此时就要进行内核调试了**
### 点灯法（此时手头没有仿真器 printk失败）
    内核移植过程中printk是有可能不打印信息的，因为此时串口没有移植好
    此时需要在汇编部分根据原理图 和芯片手册 编写汇编点灯程序，这样观察
    代码执行到哪里了,一般都是把点灯代码加到内核自解压或者内核汇编部分
### 看printk打印信息（最常用）
    内核中打印信息有多种方法 
    puts(内核解压前)
    printascii(console(串口终端)初始化前)
    printk(内核解压后，信息输出显示是在console初始化之后)主要用这个
#### proc查看和修改日志级别
    cat /proc/sys/kernel/printk 显示4 4 1 7
    意思就是只显示0 1 2 3 级别的信息（一般这四个级别是内核系统使用，开发者不用）
    echo "7 4 1 7" > /proc/sys/kernel/printk //输入这条指令进行更改打印级别
    效果是0-7，8个级别的信息全部打印
    cat /proc/sys/kernel/printk  //显示7 4 1 7 更改成功
    后面三个参数的意义感兴趣的可以查查
#### 内核信息打印级别
    #define KERN_EMERG  KERN_SOH "0"    /* system is unusable */  
    #define KERN_ALERT  KERN_SOH "1"    /* action must be taken immediately */  
    #define KERN_CRIT   KERN_SOH "2"    /* critical conditions */  
    #define KERN_ERR    KERN_SOH "3"    /* error conditions */  
    #define KERN_WARNING    KERN_SOH "4"    /* warning conditions */  
    #define KERN_NOTICE KERN_SOH "5"    /* normal but significant condition */  
    #define KERN_INFO   KERN_SOH "6"    /* informational */  
    #define KERN_DEBUG  KERN_SOH "7"    /* debug-level messages */ 
**一般通过proc改比较方便，用代码指定也行，在这里不做探讨**
***一般都是在这里start_kernel 位置init/main.c 根据需要写入要输出的信息，然后重新编译内核，在根据打印信息研究***
### OOP内核异常信息

    该错误经常是由空指针的问题引起的，在驱动开发里非常容易发生，比如在某个
    驱动函数中写入int *ptr=NULL;*ptr=0xff;
    这是空指针的一种错误操作(空指针还没有申请空间就往里面写数据肯定不行)，就会引起这种错误
    一般这种错误的log会打印出错的函数，如果没看出来，可以借助交叉编译的工具来定位具体的哪一行
    arm-none-linux-gnueabi-addr2line c024225c -e vmlinux -f 
    这条命令执行后会显示出错的具体位置，
    开头的是交叉编译工具链的名字 addr2line是这个小工具的名字
    vmlinux是编译完成之后未压缩的内核含有调试信息
    c024225c是PC出错的位置(log里会打印，注意看)
***一般这种现象是打印出一堆寄存器的值，内核也崩了***
<font color=red>注意修改内核或者驱动源码之后，确保你的makefile里的编译规则会编译改过的文件