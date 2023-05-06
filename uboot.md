# bootloder相关的一些特点和概念
bootloder是硬件启动的引导程序，是运行操作系统的前提，进行硬件的各种初始化配置
bootloder不属于操作系统，在移植系统时，一般首先移植bootloder。
**板子上电运行的第一段代码就是bootloder**
## bootloder的操作模式
1 自启动模式 上电之后用户不干预他，自动进入内核
2 交互模式 一般是用串口接收用户命名进行一些环境变量的配置
**bootloder分为很多种，提到bootloder不只是说uboot,只不过目前uboot（通用bootloder）是用的最多的**
*有能力的话，甚至可以自己用汇编语言和C语言写一个bootloder*
# UBOOT一些命令
***输入help查看所有命令***
##<font color=red>命令分类
### 环境设置
#### printenv
    显示所有环境变量
#### saveenv
    将当前定义的所有的环境变量值存储到flash
#### setenv
    设置新的环境变量
    例如 setenv my=123 这样就会在环境变量的清单里新加入一个叫my的变量，且
    该变量值为123
#### 删除变量
    如果想删除my这个变量，需要先输入setenv my 
    注意此处my后面是空格，然后输入saveenv 这样环境变量的清单里就没有my这一项了
### 数据传输
#### tftp 通过网络下载程序
    例如 tftp 内存地址 文件名

### 存储器访问
#### Nor flash与nand flash的区别
    前者可以按字节访问，后者只能按块访问，不能只操作某一块的某一个字节
所以一般nor flash做启动引导，nand flash用于存储大文件
#### protect 对Nor flash写保护
    例如 protect on 0 10000 对区间[0x0,0x10000]进行写保护
    protect off 0 10000对上述区间取消写保护
#### erase 擦除Nor flash
    例如 erase all 擦除FLASH所有的扇区
    erase 0 10000把FLASH区间[0x0,0x10000]擦除
#### Nor flash的特点
    假如nor flash里面数据是1111如果你想把1变0，可以直接操作
    但是如果数据是0000你想把0直接变成1，这在硬件上是不允许的，需要先擦除

#### Nand flash相关命令
    用的少，这地方
    nand read addr off size
    nand write addr off size
    nand erase erase [clean] [off size]
#### movi命令
### 重点命令
#### bootcmd
    把需要手动输入的命令以变量值的形式赋给他，不同命令间用分号分隔
#### bootm
    bootm kernel-addr ramdisk-addr dtb-addr
    引导内核且为内核传参，三个参数分别是内核的地址 根文件系统地址（这个参数可以不加） 设备树地址
### 加载运行
    go addr 执行内存中的二进制代码，简单的跳转到指定的地址
    在uboot中如果想运行一些裸机程序可以用这个命令来做
#uboot 编译烧录及相关配置
    整个工程是通过顶层目录下的makefile开始递归的调用各级子目录下的makefile，最后链接成uboot镜像
##顶层目录下的Makefile的作用
    负责uboot整体配置和编译
    一般需要在该makefile里指定交叉编译工具链
##编译生成的文件
    u-boot.map u-boot映像的符号表 方便源码跟踪
    u-boot u-boot映像的ELF格式 一般该文件用于调试uboot
    u-boot.bin u-boot映像原始的二进制格式 烧录用 重点是这个文件
    u-boot.srec u-boot映像的s-record格式
其实此处这么写有些笼统，具体看厂商，有些厂商会把一些硬件的初始化代码剥离出来，然后生成bin文件，需要在uboot.bin前面加上这些文件才会正常启动
## 镜像下载烧录
    初次或开发板代码损坏不能正常启动时，可以采用JTAG工具烧录
    专用的烧录工具如h-jtag或者DNW等
    u-boot已经能工作，升级或修正u-boot时可以用uboot的命令来完成烧录
    或者烧录到sd卡里
一般自己移植uboot会在board目录下新建一个，这样方便管理
# u-boot目录结构
##平台相关
    arch,board,include....
##平台无关
    common,net,fs,drivers...
##工具和文档
    tools,doc
***此处需要重点掌握，抽时间研究一下每个目录的作用***
# U-BOOT启动流程
**这部分非常重要,一定要死死记住**
***u-boot是上电的第一个程序，绝对不能出问题，要求绝对稳定***
# u-boot启动流程和代码细节这部分非常复杂，建议找专门的优质资料学习
# u-boot移植方法
注意不要在windows下的共享文件夹进行linux的各种编译，因为windows不支持软连接文件（符号文件），而编译过程中会生成大量的软链接文件
## Beyond
    善用这种代码比对软件，移植的时候对比官网的代码看看到底改了哪里，思考
    为什么这样改，集中精力做重点
## 第一道难关 找到正确版本的官方源码再配置编译
### 指定交叉编译链
    这个看厂家，，有的芯片你不签NDA啥资料都没有，比如海思，那交叉编译链和SDK就别想了
### 指定cpu和board(参考最类似的配置比如origen)
    看厂家，如果厂家直接仿照官方评估板来做，那恭喜uboot几乎不用改啥。
    有几率你找了一份配置编译完成烧录之后，发现串口信息不打印，啥反应也没有，这时候就只能自己改了(进入第二道难关)
## 第二道难关 实现串口信息输出
### 跟踪代码运行(uboot源码内部写入汇编或c来控制led亮灭)
### JTAG接口调试器调试
## 第三道难关 网卡移植（实现TFTP NFS方便开发调试）
    寄存器地址
    参数设置  主要侧重改这两方面
## 第四道难关 FLASH移植（实现能下载软件到FLASH，产品能离线运行）
大概思想是这样，，，具体的细节很难说，这部分详细的还得看内核移植的部分
想要深入研究还得找一套优质教程，，，








