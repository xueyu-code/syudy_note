# Linux设备驱动编写
## 编写驱动的相关知识
    驱动源码（大部分是c文件少部分是汇编）可以编译进内核(在内核启动时自动启动)
    或者以模块的形式(.ko灵活度高，模块化易于管理，减少内核体积)
    内核中的驱动越多，执行时间（开机时间）越长
**编写驱动代码的前提是你的内核源码要先编译，你的模块装载的时候内核首先要启动起来
因为ko是运行在内核空间的（但是装载这一步是在用户态）**
## 大致分为6个部分
### 1 头文件
    一般必须有的是
    #include <linux/init.h>
    #include <linux/odule.h>
### 2 驱动模块的装载和卸载函数入口的声明
    module_init(hello_init);//这里填写了一个hello驱动的函数名
    module_exit(hello_exit);
### 3 实现模块装载和卸载函数入口
    static int _init hello_int(void)
    {
        //一般是做系统申请资源
        return 0;  //装载函数的返回值一定要是int，
        //入口参数一定要是void,_init是给编译器用的（可以不写）
    }


    static void _exit hello_exit(void)
    {
        //一般是做系统释放资源
        //卸载函数的返回值一定是void型
    }
### 4 GPL声明
    MODULE_LICENSE("GPL");//有方法跳过这个GPL声明

### 5 编译为ko文件
    编写Makwfile规则，注意这个Makefile里的内核路径所指向的内核必须是已经编译通过的一个
    一般写成如下这种
```Makefile
ifeq ($(KERNELRELEASE),) #如果KERNELRELEASE为空就执行all对应的命令,默认进来第一次KERNELRELEASE会为空
KERNEL_DIL=/home/linux_kernel #这个地方填写你的内核代码路径
CUR_DIR= $(shell pwd) #调用pwd命令获取当前路径
ROOTFS_DIR= /opt/rootfs #根文件系统路径
all :
    make -C $(KERNEL_DIR) M=$(CUR_DIR) modules 
#-C是进入内核路径的意思，M后面指定当前路径并编译为模块
clean : 
        make -C $(KERNEL_DIR) M=$(CUR_DIR) clean
install :
    cp -raf *.ko $(ROOTFS_DIR)/drv_module
    #把编译出来的ko文件都拷贝到根文件系统目录下的drv_module

else
    obj-m += hello.o #编译为对应的名字，这个决定到底编译那个驱动
#此处可以添加多条相似指令，来同时编译多个模块
endif
```
### 6 装载模块
    insmod hello.ko //装载模块且自动执行驱动中的init函数
    lsmod//查看系统中装载过那些模块
    rmmod hello //卸载模块且自动执行驱动的exit函数，卸载命令不需要加.ko的后缀
    如果执行卸载命令报错说没有文件夹，请自行手动创建一个
## ko模块的开发
### 参数传递
    在代码中如何处理参数：
    module_param(name,type,perm)
    name:表示参数的名字
    type:表示参数的类型，charp,int
    perm:表示参数文件的权限一般是0644 /sys/modules是该文件的存放位置
    也可以用宏的形式来写详细的可以参考内核源码中的用法
**假如在代码中定义了一个变量参数test=18,但是传参让test=23那么优先使用传过来的参数**
***给的权限太大可能会报错***
### 符号导出
    这个是方便模块与模块间交互，类似于软件中动态库的概念
    应用场景：
        你有两个驱动文件，一个是hello.c一个是math.c，math.c里实现一些算法，
        可以不写模块的init和exit函数只写功能，然后hello去调用这些功能
    代码实现：
        EXPORT_SYMBOL(func);//func是供其他模块调用的函数名，这个export_sumbol是个宏
**注意需要建立math.h头文件来声明func函数以防止编译出现函数声明的问题**
***注意模块装载的顺序，在上面的例子里如果先装载hello.ko会直接报错，因为此时还没有装载math.ko，所以要先装载math.ko然后装载hello.ko***




