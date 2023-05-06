# C++结合QT的基础笔记
*学习难点就是C++为了实现各种用法，定义的概念的太多，搞清每个概念就行*

##  面向过程与面向对象
    面向过程：思想核心是功能分解，自顶向下，逐层细化 
    例如C语言就是这样面向过程的语言，程序=算法+数据结构

    面向对象编程：算法与数据结构可以看作一个整体称作对象
    程序=对象+对象+对象
**此处的数据结构指的是数据的存储方式，指的是数据类型：char short**
C语言是弱语法语言（很多警告可以忽略），而C++是强语法语言（很多在C语言中是警告在C++中直接报错）
##  C++的三大特性
    1封装：将客观事物抽象为一个类（将数据和方法打包在一起加以权限的区分）
    2继承：表示类之间的关系，使得对象可以继承另外一个对象的特征和能力
    3多态：一个接口多种方法，程序在运行时才决定调用的函数
## C++对C的拓展
### ::作用域运算符
表明数据 方法的归属性问题
```c++
int a=10;//这是全局变量
void test()
{
    int a=20;//这是局部变量
    cout<<"a="<<a<<endl;//这种情况下会优先选择局部变量
}
//在执行test函数时，会遵循就近原则把局部变量的优先级高于全局变量
//在C语言里也遵循这个原则
```
```c++
int a=10;//这是全局变量
void test()
{
    int a=20;//这是局部变量
    cout<<"a="<<::a<<endl;//利用作用域实现调用全局变量
}
//作用域运算符在C++里可以用这种方式调用全局变量
```
### 命名空间
在大型项目中难免出现变量名字重复的问题，所以C++引入了命名空间

*在C语言中是通过static静态变量的方式来解决，C++中通过命名空间的方式来解决（也支持static静态变量的方式）*
*static静态变量的方法就是假如在A文件中定义了x变量，加入static的前缀使x变量只在A文件内有效*
**C++的命名空间可以用于符号常量 变量 函数 结构 枚举 类和对象等等**
#### 命名空间的使用方式
##### 正确的定义方式
```c++
//定义一个名字为A的命名空间
namespace A{
    int a=100;
}
//定义一个名字为B的命名空间
namespace B{
    int a=200;
}
void test02()
{
    //错误用法
    cout<<"a="<<a<<endl;
    //这时候直接执行test02，会直接报错显示a没定义过
    //所以用的时候必须指定命名空间
    //正确用法
     cout<<"a="<<A::a<<endl;
}
```
##### 不允许在局部定义命名空间
```c++
//错误的定义方式
void test()
{
    namespace A{
        int a;
    }
}
//这种也是直接报错，命名空间只能在全局范围（函数外面）中定义
//在函数内部定义不允许
```
##### 可以嵌套定义命名空间
```c++
//命名空间允许嵌套定义
namespace A{
    int a=10;
        namespace B{
            int a=20;
        }
}
//嵌套定义完之后使用一定要指定到最底层
void test03()
{
    cout<<"A中的a="<<A::a<<endl;
    cout<<"B中的a="<<A::B::a<<endl;
}
```
##### 给命名空间添加新成员
```c++
//定义一个名字为A的命名空间
namespace A{
    int a=100;
}
//这个时候如果想给命名空间添加新成员
//直接写成下面这样就行
namespace A{
    int b=200;
}
//如果想在某个函数内部添加新成员也是可以的
//前提是该命名空间已经在全局定义过了，否则报错
void test04()
{
    namespace A{
        inr c=300;
    }
}
```
*在命名空间内引用命名空间自己内定义的变量不需要指明变量属于哪个命名空间*
```c++
namespace A{
    int a=10;
    void fun{
        cout<<"a="<<a<<endl;//这样写是可以的
    }
}
//但是在命名空间外引用一定要说明命名空间
//下面这种就是错误的
void test03()
{
    cout<<"A中的a="<<a<<endl;
}
```
##### 命名空间中的函数可以在外面定义
```c++
namespace A{
    int a=10;
}
//这样写就行，在函数定义的时候给他指明命名空间
void A::fun()
{
    cout<<"A中的a="<<a<<endl;
}
void test06()
{
   A::fun();
}
```
##### 命名空间也可以是没有名字的
*尽量少定义无名命名空间*
```c++
namespace {
    int a=10;
}
void test06()
{
   cout<<"a="<<a<<endl;//无名自然也不需要在a前面写命名空间的名字
}
//上述代码这么写是允许的
//无名命名空间意味着空间内部的成员只能在本文件内访问，相当于C语言的static
```
##### 可以给命名空间取别名
*很少这么用*
```c++
namespace very_long_name{
    int a=10;
    void fun(){cout<<"a="<<a<<endl;}
}
void test06()
{
    namespace very_long_name=short_name;//这个namespace是必须写的，否则报错
    very_long_name::fun();
    short_name::fun();
}
//上述写法是允许的
```
##### using使用整个命名空间
*这个用的比较多，虽然我不喜欢这么写*
```c++
//这种用法方便用户进行输入，有些用户不喜欢多输入单词
namespace very_long_name{
    int a=10;
}
void test06()
{

    using namespace very_long_name;//使用特定的命名空间
    //从这开始出现的变量，先从very_long_name命名空间中寻找
    //找不到就去其他地方如全局或者局部变量中寻找
    cout<<"a="<<a<<endl;

}
//上述写法是允许的
```
用using来完成上述操作付出的代价是容易造成冲突
```c++
namespace very_long_name{
    int a=10;
}
void test06()
{
    int  a=200;
    using namespace very_long_name;
    cout<<"a="<<a<<endl;
}
//这个时候两个地方都有a
//会遵循就近原则输出200

//如果要想输出命名空间的a，需要加入命名空间前缀
void test07()
{
    using namespace very_long_name;
    cout<<"a="<<a<<endl;//输出局部变量a
    cout<<"a="<<very_long_name::a<<endl;//输出全局变量a
}
//到头来还得加前缀，方便了个寂寞
```
##### using指明使用具体的命名空间成员
```c++
namespace very_long_name{
    int a=10;
    void fun(){ cout<<"a="<<a<<endl;}
}
void test06()
{

    using very_long_name::a;//注意此处不需要加namespace前缀，就能指明只使用a
    cout<<"a="<<a<<endl;//直接这样写就能使用命名空间中的a
    very_long_name::fun();//但是使用其他没指定的成员的时候，还是得说明作用域否则报错

}
```
下面这种情况不会遵循就近原则，会直接报错
```c++
namespace very_long_name{
    int a=10;
    void fun(){ cout<<"a="<<a<<endl;}
}
void test06()
{
    int a=200;
    using very_long_name::a;//注意此处不需要加namespace前缀，就能指明只使用a
    cout<<"a="<<a<<endl;//这时候不会遵循就近原则 会直接报错
}
```
*但是下面这种情况并不会报错，此处有疑问*
*貌似是因为全局变量前面默认隐藏了一个作用域::*
```c++
namespace very_long_name{
    int a=10;
    void fun(){ cout<<"a="<<a<<endl;}
}
int a=200;
void test06()
{
    using very_long_name::a;//注意此处不需要加namespace前缀，就能指明只使用a
    cout<<"a="<<a<<endl;//等同于 cout<<"a="<<::a<<endl;
}//这种写法不会和全局变量冲突
```
##### using遇到函数重载的情况
此处等学到重载的时候再说

##### 不同命名空间中的同名成员，使用时注意二义性
```c++
namespace A{
    int a=10;
}
namespace B{
    int a=10;
}
void test06()
{

    using namespace A;
    using namespace B;
    cout<<"a="<<a<<endl;//这样没有具体说明属于哪个命名空间会产生二义性
    //除非在a前面指明命名空间，否则直接报错
}
```
##### 标准命名空间std
cout cin endl都是命名空间std的成员 可以这样写
```c++
using namespace std;
void test06()
{
    cout<<"a="<<a<<endl;//这样就能直接使用cout和endl了
}
```
或者这样直接指明
```c++
void test06()
{
    std::cout<<"a="<<a<<std::endl;
}
```
### C++的引用(重要)
**能用引用不要用指针，切记，切记**
变量名实质上是一段连续内存空间的别名，是一个门牌号，程序中通过变量来申请并命名内存空间，通过变量的名字可以来使用存储空间
*引用的核心作用就是给已有变量取一个别名（一段连续的内存空间可以取不止一个别名）*
**C++中的&有时候是表示引用，有时候是表示取地址**
**引用本身所占的内存空间和指针一样**
#### 引用的使用流程
```c++
int num=10;//假如要给num取别名，肯定先定义num
int &a=num;//&要和别名结合，表示这是个引用，引用表示完之后必须紧接着初始化
//int *a =&mum; 这么写才是取地址
```
**取别名，比如上述代码的a，并不会给a开辟空间。a完全等价于num,num能完成的操作，a也能完成，a和num表示同一段内存空间**

#### 引用一旦初始化就不能再次修改别名
通俗的说就是一个引用（别名）绑定一个变量，不能重新绑定给别的变量，一夫一妻制
```c++
int num=10;
int &a=num;

int data=20;
a=data;//该语句不能表示data的别名为a,而是表示将data的值赋值给a(num)
```
#### 引用给数组取别名
##### 方式1
```c++
void test01()
{
    int arr[5]={10,20,30,40,50};
    int (&my_arr)[5]=arr; //这样就表示my_arr是整个数组的别名
    //如果不加哪个小括号就变成定义了一个数组，每个数组元素是引用的数组
    //因为&的优先级没有[]高
    for(i=0;i<5;i++)
    {
        cout<<my_arr[i]<<" ";
    }
    cout<<endl;
}
```
##### 方式2：配合typedef
```c++
void test01()
{
    int arr[5]={10,20,30,40,50};
//1 用typedef给数组类型取个别名
    typedef int type_arr[5];//表示type_arr就是一个数据类型
                            //该数据类型内包含5个元素，且每个元素是int类型
//type_arr new_arr={1,2,3,4,5};就表示定义了有5个元素的cint数组
//2 按照之前说的来
    type_arr &my_arr=arr;
    for(i=0;i<5;i++)
    {
        cout<<my_arr[i]<<" ";
    }
    cout<<endl;
}
```
上述代码中涉及到typedef的用法，详细请参考C语言的typedef的使用细节，抽空复习一下
#### 引用作为函数参数
```c++
void my_swap1(int &a,int &b)//此时的形参是表示引用
{
    int tmp=a;
    a=b;
    b=tmp;
}
void my_swap2(int *a,int *b)//此时形参表示传入指针
{
    int tmp=*a;
    *a=*b;
    *b=tmp;
}
void test()
{
    int data1=10,data2=20;
    //my_swap2(&data1,&data2);//这样写能用指针的方式使其交换数值
    my_swap1(data1,data2);
    //这样写直接把变量的别名（原本的名也算别名）传进去利用引用交换数据
}
```
通过引用参数产生的效果与按地址传递的效果是一样的，引用的语法更简单
*函数调用时传递的实参不必加&符号*
#### 引用作为函数返回值
*本质是给函数的返回值取别名*
```c++
int& my_data(void)//int&就表示该函数的返回值是一个int型的引用（别名）
{
    int num=100;
    return num;//变量名本身就是个别名，自然也算作引用
}
void test05()
{
    int &ret=my_data();//定义一个int型的引用，紧接着初始化
    //此时ret就作为函数返回值num的别名
}
```
上述的代码，看起来没有问题，但其实是一种危险操作，具体说明如下
```c++
int& my_data(void)//int&就表示该函数的返回值是一个int型的引用（别名）
{
    int num=100;//num是个局部变量
    return num;//
}
void test05()
{
    int &ret=my_data();//定义一个int型的引用，紧接着初始化
    cout<<"num="<<ret<<endl;
}
int main()
{
    test05();
}
//此时上述代码会输出空
//因为num是一个局部变量，用完马上就释放了，
//ret代表的num内存空间就是释放之后的内存空间就是空
//这是非法访问内存，会报错
```
**函数的返回值是引用时，不要返回给一个局部变量**
##### 函数返回值作为左值时，那么函数返回值的类型必须是引用
*此处多数用在运算符重载*
必须如以下代码这么操作时，函数才能作为左值
```c++
int& my_data(void)
{
    static int num=10;//注意此处的static的生命周期是比局部变量长的
    cout<<"num="<<num<<endl;

    return num;
}
void test()
{
    my_data()=1000;//该语句是把1000赋给函数的返回值
    my_data();//此时函数会输出1000说明上一句是把1000给函数的返回值
}
```
#### 指针的引用
此处知识用的比较少
*实质上就是给一个指针变量来取一个别名*
如果要修改一个指针的指向，传统C语言的思路是用二级指针
```c++
void my_str1(char **p)//此处要求传入的参数为二级指针,等价于p保存了str的地址
{
    *p=(char *)calloc(1,32);
    strcpy(*p,"hello world");
}
void test07()
{
    char *str=NULL;//定义一个指针变量并初始化为空指针
//假如此时有个需求 
//要求你用一个函数从堆区给str申请一个内存空间并赋值hello world
    my_str1(&str);
    cout<<"str="<<*str<<endl;
    free(str);//用完这片内存空间马上释放
    
}
```
**引用的方式来完成**
```c++
void my_str2((char * )&p)//此处写成char * &p也可 因为优先级的作用
{
    p=(char *)calloc(1,32);//此时p代表指针变量本身的内存空间
    strcpy(p,"hello world");
}
void test08()
{
    char *str=NULL;//定义一个指针变量并初始化为空指针
//假如此时有个需求 
//要求你用一个函数从堆区给str申请一个内存空间并赋值hello world
    my_str2(str);
    cout<<"str="<<*str<<endl;
    free(str);//用完这片内存空间马上释放
    
}
```
#### 常量引用(常引用)
*在C++类传参基本都是常引用，尤其是在类的拷贝 复制构造函数*
先看应用场景
```c++
typedef struct 
{
    int num;
    char name[32];
} STU;
void myPrintfSTU1(STU tmp)
{//此时的tmp需要给他开辟一个结构体大小的空间
    cout<<"num="<<tmp.num<<endl;
    cout<<"name="<<tmp.name<<endl;
}
void test()
{
    //需求是要求你定义一个函数来遍历lucy的成员
    STU lucy={100,"lucy"};
    myPrintfSTU1(lucy);
}
//上述代码看起来没啥问题，但是函数的形参是一个普通结构体变量，占用空间太大
```
输出数据是读操作，功能函数占用空间大，这是不合理的代码，改成下面引用的方式就可解决
```c++
typedef struct 
{
    int num;
    char name[32];
} STU;
void myPrintfSTU2(const STU &tmp)//这时候形参是一个结构体的引用（别名）
{//tmp是lucy的别名，不需要为他开辟空间
    cout<<"num="<<tmp.num<<endl;
    cout<<"name="<<tmp.name<<endl;
    //形参写成上述形式便是常引用的含义，写成这样可以避免函数内部对传入参数的修改
    //tmp.num=2000;//此时这条语句报错，因为有const修饰
    //这样既降低了传参的开销也避免了函数内部不小心对参数内容的修改
}
void test()
{
    //需求是要求你定义一个函数来遍历lucy的成员
    STU lucy={100,"lucy"};
    myPrintfSTU2(lucy);
}
```
#### 常量的引用(与上述章节不是一个概念)
首先在此处需要明确两个概念
**1 常量的引用是指给一个常量取别名，比如给10这个数字取一个别名(引用)**
**2 尽管可以写成int a=10,但是10并不是一个int类型而是const int型**
```c++
//假如你需要给10这个常数取一个别名
void test()
{
    int &num=10;//会报错，因为10是const int类型不是int型
    //此处也不能将const int强转为int.因为本身常量就是一个不能被修改的量
    //如果能修改就是变量了，强转在此处没用，依旧报错
    const int &num =10;//这样才行，就能完成对常量的引用
}

```
#### 引用的本质(了解性)
引用的本质在C++内部实现是一个指针常量，以下代码举例
```c++
int data=10;
int &a=data;//在编译器内部转化为int *const a=&data;一个只读的指针变量
//这样就实现了引用只能绑定给一个变量，不能修改的功能
a=2000;//在编译器内部转化为 *a=2000
```
## C++对C的增强

### C++对全局变量的检测增强
C语言是弱语法语言，导致很多地方判断不严谨
下面的代码在C语言中可以编译通过，但是C++不行
```c
int a=10;//C语言会把有赋值的看作定义
int a;//如果前面和他有个同名的变量，且这里没有赋值，C语言会把此处看作声明
void test()
{
    printf("a=%d",a);
}
```
### C++中所有的变量和函数必须有类型
下面的代码在C语言中可以编译通过，但是C++不行
```c
void fun1(a)//a没有写类型，在C语言中代表可以是输入任意类型
{//不建议这么写
    printf("a=%d\n",a);
}
void fun2(a)
{
    printf("%s\n",a);
}
void main()
{
    fun1(100);
    fun2("string");
}//上述代码会打印出 100和string
```
**C语言中如果函数没有参数，建议写void代表无参数**
### C++是更严格的类型转换
*C++中不同类型的变量一般是不能直接赋值的，需要相应的强转*
以下代码在C语言中可以正常编译，C++报错
```c
typedef enum COLOR(GREEN,REG,BLUE) color;
int main()
{
    color mycolor=GREEN;//用枚举类型的初衷就是限定取值范围，
    //此处的GREEN应该是个固定的数值，不应该随便赋值
    mycolor=10;
}
```
### C++对结构体的增强
C语言中用结构体必须加struct关键字，C++不需要（当然你加上也不报错）
**C中的结构体只能定义成员变量，不能定义成员函数，C++都可以**
### C++布尔类型关键字
标准c++的bool类型有两种内建的常量true(转换为整数1)和false(转换为整数0)
*只有这两个值*
```c++
void test()
{
    cout<<sizeof(false)<<endl;//为1，布尔类型占一个字节大小
    bool flag=true;
    flag=100;//给布尔类型赋值时，非零值自动转换为true(1),0自动转换为false
}
```
*在C99标准之前是没有布尔关键字的，但是C99标准有，只需包含stdbool.h就可以用布尔类型了*
### 三目运算符增强
**C语言的三目运算符返回值为数据值，右值（不能给他赋值）**
```c
int a=10;
int b=20;
(a>b?a:b)=100;//这样写会直接报错，右值不能被赋值
```
**但是C++的三目运算符返回的是变量本身(引用)，为左值（可以给他赋值）**
### C语言中的const
以下代码为在C中的情况
```c
const int a=10;//不要把这个a看作常量 
//a的本质是变量，是只读变量 他是有内存空间的,会被分配内存空间
```
**这就导致可以通过内存地址的方式间接修改这个只读变量的数据**
```c
void test()
{
    const int data=100;//局部变量的内存地址是在栈区，栈区是可读可写的
    printf("data=%d\n",data);
    int *p =(int *)&data;
    *p=2000;
    printf("data=%d\n",data);
    //上述代码在输出data=100之后会紧接着输出data=2000
}
```
但要注意下面这种情况不能修改
```c
const int data=10;//此时定义的是全局变量
void test()
{
    int *p =(int *)&data;
    *p=2000;
    printf("data=%d\n",data);
    //此时会输出10，并没有修改成功，某些严格的编译器会直接报错
    //当全局变量被const修饰的时候，内存空间是在文字常量区开辟，该内存空间是只读的
}
```
*C语言中的const修饰全局变量的时候，默认是外部链接的*
*外部链接的含义是：别的源文件可以使用他，加入extern声明即可*
### C++中的const
**C++中定义的const可以当作常量看待**
**在C++中，是否为const常量分配内存空间取决于如何使用他**


#### C++中的const默认是内部链接
**在C++中,如果在函数外面定义定义一个const，那么他的作用域只有当前文件内有效，默认是内部链接，但是C++的其他标识符一般默认为外部链接**
```c++
//在test.cpp中定义一个全局变量
const int a=10;//这么定义一个全局变量
```
```c++
//在main.cpp中进行如下操作
extern const int a;//这么写是声明
void test()
{
    cout<<"a="<<a<<endl;//该代码会直接报错
}
```
*但是下面这么写是可以编译通过的*
```c++
//在main.cpp中进行如下操作
extern const int a=999;//C++会把这种赋值认为这是一个定义，
void test()
{
    cout<<"a="<<a<<endl;//此时输出999
    //因为函数外面定义的const变量作用于当前文件，所以此处的a是有定义的
}
```
**如果必须把全局的const只读变量用在其他源文件，那么在定义的时候必须加入extern前缀，转换为外部链接**
```c++
//在test.cpp中定义一个全局变量
extern const int a=10;//这么定义一个全局变量
```
```c++
//在main.cpp中进行如下操作
extern const int a;
void test()
{
    cout<<"a="<<a<<endl;
}
```
#### 讨论C++中的const内存空间存在与否的问题
**对于基本的数据类型，例如const int a=10;这种，编译器会把他放到符号表中，不分配内存，但是当对其取地址时，或者定义的时候加入extern，又会给他分配内存**
下面的代码说明了C++的这种特性
```c++
//这时候不开辟内存空间
const int a=10;
//这时候才开辟内存空间
int *p=(int*)&a;
*p=200;
cout<<"a="<<a<<endl;//此时从符号表中取出a的值
cout<<"*p="<<*p<<endl;//此时从内存空间中取出值
//此时上述代码会输出 a=10 *p=200
//出现这种情况的本质原因时，更新内存空间不会更新符号表
```
**对于基础数据类型，如果用一个变量初始化const变量，例如const int a=b;那么也会给a分配内存**
```c++
int b=10;
const int a=b;//此时不会存储到符号表会直接开辟内存空间
int *p=(int*)&a;
*p=200;
cout<<"a="<<a<<endl;
cout<<"*p="<<*p<<endl;
//此时上述代码会输出 a=200 *p=200
//a已经被分配了内存，所以可以修改他的值
```
**对于自定义的数据类型（结构体 类），比如类里面的对象，也会分配内存空间，不放在符号表**
```c++
struct person{
    int num;
    char name[25];
}
void test()
{
    const person lucy={100,"Lucy"};//此处定义一个只读的结构体变量
    person *p=(person *)&lucy;//必须这样写，其他也类似，C++有严格的类型要求
    cout<<"num="<<lucy.num<<endl;
    p->num=2000;
    cout<<"*p.num="<<p.num<<endl;
    cout<<"num="<<lucy.num<<endl;
}//上述代码会依次输出num=100 p,num=200 num=2000
```
#### const与#define的区别
**编写C++代码时尽量用const代替#define**
1 const有类型，可以进行编译器类型安全检查，#define无类型，不能进行类型检查
2 const有作用域，而#define不重视作用域，#define不管是在所有函数之外还是在某个函数之内，**作用域都是从定义处到整个文件结束**。如果定义在指定作用域下有效的常量，那么#define就不能用
**3 宏不能作为命名空间的成员**例如在以下代码中
```c++
namespace A{
    const int a=100;
    #define B 200
}
void test()
{
    cout<<"a="<<A::a<<endl;//这条语句可以正常输出 命名空间可以识别到a
    cout<<"B="<<A::B<<endl;//这条语句会报错，命名空间识别不到这个define
    //也说明这个宏不属于命名空间，而是属于当前文件
     cout<<"B="<<B<<endl;//这样可以输出这个宏
}
```

## C的缺陷与写C++时如何避免缺陷
**后续慢慢补充**
### 易产生链接错误
由于C语言的const是外部链接，如果在不同文件中定义多个同名的const只读变量，那么编译器会产生链接错误。
### 允许用变量定义数组
C99支持用变量定义数组，如下代码可以在支持C99的gcc中正常编译
```c
int a=10;
int array[a];
//不推荐这种写法
```
### 编写C++时尽量用const代替#define
加入你在代码中编写#define max 1024;由于在预处理阶段，所有为max的地方都会被替换为1024，所以max从未被编译器看过。**当我们的代码出现问题时，提示的报错信息可能只包含1024这个信息，假如你不知道1024代表什么，那这个报错信息会让你困惑**

***一定要用const替换，写成这样 const int max=1024;***
## 内联函数
**内联函数继承了宏函数的效率，没有普通函数调用时的开销，又可以像普通函数那样，可以进行参数 返回值类型的安全检查，也可以作为成员函数**
*宏函数概念：在C语言中我们经常把一些短并且执行频繁的计算（切记是计算）写成宏，
这种写法被称作宏函数（但并不是真正的函数）*
### 宏函数的弊端
1 宏函数(**带参数的宏**)说到底也还是宏，是在预处理阶段完成。尽管看起来是个函数调用，但实际上不是
导致会隐藏一些错误信息（就是前几个章节提到的max和1024的问题）**这个问题C和C++都有**
2 由于预处理阶段不允许访问类的成员，所以*宏函数不能作为类的成员函数*。**(C++特有问题)**
*上述弊端用以下代码举例*
```c++
#define ADD(x+y) x+y//宏函数例子
inline int add(int x,int y)//内联函数例子
{
    return x+y;
}
void test()
{
    int data1=ADD(10,20)*10;//期望结果应该是300
    int data2=add(10,20)*10；//期望结果是300
    cout<<"data1="<<data1<<endl;//输出210
    cout<<"data2="<<data2<<endl;//输出300
}
//上述宏函数没有得到预期的结果原因是 宏只做替换她不会检查参数完整性
//实际上ADD（10，20）直接被替换为10*20，
//替换完的表达式就是10*20+10=210
```
**哪怕宏函数写的严谨一些，解决参数的优先级（完整性问题），还是有缺点**
```c++
#define MIN(x,y) ((x)<(y)?(x):(y))//这种宏函数把每个参数括起来，保证了优先级
inline min(int x,int y)
{
    return x<y?x:y;
}
void test()
{
    int a=1;
    int b=3;
    cout<<"MIN="<<MIN(++a,b)<<endl;//会输出3 而不是2
    cout<<"min="<<min(++a,b)<<endl;//会输出2
}
//上述代码出现这个问题是因为宏函数只做替换
//会把++a当作X然后进行生硬的替换，替换的结果如下
//(++a)<b?(++a):b这样执行了两次++a就是3
```
### 类内部的内联函数
在类里面定义的函数，无论前面加不加inline前缀，都会自动变成内联函数
### 内联函数的了解性知识
1 *内联函数的替换是发生在编译阶段*而不是预处理阶段
2 **内联函数也是占用空间的，用空间换时间**
3 内联函数并不是何时何地都有效
### 编译器如何处理内联函数
**内联函数不是万能的**
内联函数并不是何时何地都有效，为了了解内联函数何时有效，我们需要知道编译器遇到内联函数是怎么处理的。
对于任何类型的函数，编译器会将函数类型（包括函数名字，参数类型，返回值类型）放入符号表中。同样当编译器看到内联函数，并且对内联函数的函数体进行分析没有发现错误时也会把内联函数放入*符号表*。当调用一个内联函数的时候，编译器首先确保传入参数类型是正确匹配的，或者如果不完全匹配，但是可以将其转换为正确类型，并且返回值在目标表达式里匹配正确类型，或者可以转换为目标类型，内联函数就会直接**替换**函数调用，这就**消除**了函数调用的开销。假如内联函数是成员函数，对象this指针也会被放入合适位置。类型检查和类型转换以及在合适位置放入对象this指针，这些都是预处理器无法完成的。
### 内联函数的限制
1 不能存在**任何**形式的循环语句
2 不能存在过多的条件判断语句
3 函数体不能过于庞大，不能对函数进行**取址操作**
**内联函数只是给编译器一个建议，编译器不一定会接受这个建议**如果没有将函数声明为
内联函数，那么编译器也可能将此函数做内联编译。一个好的编译器将会内联*小的，简单的函数*。
### 内联函数使用的注意事项
#### 需配合定义和声明使用
必须在定义的时候加入inline前缀，函数声明的时候也得加入inline前缀，否则编译器不会把她当作内联函数处理而是当作普通函数。如下所示
```c++
inline int test(void) //函数定义有inline前缀
{
    cout<<"Hello C++"<<endl;
}
inline int test(void)；//函数声明也得有inline前缀
```
## 函数的默认参数（重要）
C++在*声明*函数原型时可以为一个或者多个参数指定默认(缺省)的参数值，当函数调用时如果没有传递该参数，编译器会自动用默认值代替。
**尽管在定义函数时也可以指定默认参数，但是尽量在声明处指定而不是在定义处指定**
### 定义函数时添加默认参数
```c++
int test(int x=10,int y=20)
{
    return x+y;
}
cout<<"data="<<test()<<endl;//不传参时直接输出30
cout<<"data="<<test(100)<<endl;//输出120
cout<<"data="<<test(100，200)<<endl;//输出300 
//函数传参时，默认参数就无效了
```
### 声明函数时添加默认参数
*建议在声明时添加默认参数*
在test.cpp中编写如下代码
```c++
int test(int x,int y)
{
    return x+y;
}
```
在main.cpp中编写如下代码
```c++
extern int test(int x=10.int y=20);
cout<<test()<<endl;//直接输出30
```
*此处的注意事项请查看注意点章节*
### 默认参数的注意点
#### 一个默认参数后面必须都是默认参数
**默认参数从左往右，如果一个参数设置了默认参数，那么这个参数之后的参数都必须设置默认参数**
```c++
int fun1(int x,int y=10,int z)
{
    return x+y+z;
}
void test1()
{
    cout<<fun1()<<endl;//此处如果不为Z设置默认参数会报错缺失默认参数
}
//必须像下面这样写才行
int fun2(int x,int y=10,int z=20)
{
    return x+y+z;
}
void test2()
{
    cout<<fun2(30)<<endl;//此时X为30，其他为默认参数，这样不会报错
}
```
#### 分文件编写声明和定义时
**如果函数声明和函数定义分文件写，那么声明和定义不能同时设置默认参数**
下面这种情况可以正常通过编译
在test.cpp中编写如下代码
```c++
int test(int x,int y=20，int z=30)
{
    return x+y+z;
}
```
在main.cpp中编写如下代码
```c++
extern int test(int x.int y=20,int z=30);
cout<<test(10)<<endl;//直接输出60
```
**这种情况以声明处的默认参数为准**
```c++
int test(int x,int y=20，int z=30)
{
    return x+y+z;
}
```
在main.cpp中编写如下代码
```c++
extern int test(int x.int y=200,int z=300);
cout<<test(10)<<endl;//直接输出510
```
**声明处不写明默认参数，编译器也不会用定义处的默认参数**
```c++
int test(int x,int y=20，int z=30)
{
    return x+y+z;
}
```
在main.cpp中编写如下代码
```c++
extern int test(int x.int y,int z);
cout<<test(10)<<endl;//直接报错提示缺少参数
```
## 函数的占位参数(了解性)
C++在声明和定义的时候，可以设置占位参数。*占位参数只有参数类型声明，没有参数名*
**一般情况下，在函数体内部无法使用占位参数**
**后面重载的相关知识才会用到**
### 占位参数的例子
```c++
void test(int a,int b,int)//只写类型int,不写形参的名字就是占位参数
{
    cout<<a+b<<endl;//在函数内部也无法使用占位参数
}
```
**占位参数也是参数，函数调用时也得传递参数**
```c++
void test(int a,int b,int)//只写类型int,不写形参的名字就是占位参数
{
    cout<<a+b<<endl;//在函数内部也无法使用占位参数
}
test(10,20);//直接报错提示缺少参数
test(10,20，30);//这样才行
```
### 占位参数可以设置默认值
```c++
void test(int a,int b,int=20)
{
    cout<<a+b<<endl;//在函数内部也无法使用占位参数
}
```
## 函数重载(重要)
*这部分知识体现了C++的多态*
**同一个函数名在不同场景下有不同的意义(和多音字一样)，这就叫重载**
### 算作函数重载的依据
 **同一个作用域内**参数个数不同，参数类型不同，参数顺序不同，这几种都可以算作重载
 **注意**：函数的返回值类型不同，函数形参名不同不能作为函数重载的依据，如下代码所示
```c++
void test(int a,int b)
{
    cout<<"int int fun"<<endl;
}
void test()
{
    cout<<"void fun"<<endl;
}
void test(int a,char b)
{
    cout<<"int char fun"<<endl;
}
void test(char a,int b)
{
    cout<<"char int fun"<<endl;
}
//上面这些都可以算作函数重载都能正常编译

//下面这个返回值类型改变的不算函数重载，所以直接报错
int test(char a,int b)//返回值类型改变
{
    cout<<"char int fun"<<endl;
}
//下面这个只是参数名改变的也不算函数重载，所以直接报错
void test(char b,int a)//形参名改变了之前是 char a,int b
{
    cout<<"char int fun"<<endl;
}
```
*本质上这些重载的函数其实还是算作不同的函数（函数入口地址不同），C++区分不同函数是根据函数名+参数类型来判断*
### 函数重载与默认参数的冲突
函数重载与缺省参数结合使用，非常容易产生二义性。如下所示
```c++
void test(int a,int b=10)
{
    cout<<"int int fun"<<endl;
}
void test(int a)
{
    cout<<"int fun"<<endl;
}
int main(void)
{
    test(20);//此时只传递一个参数，可以理解为函数本来就是一个参数
    //也可以理解为有两个参数，另一个是默认参数
    //编译器不清楚该调用哪个函数 产生歧义
```
##  C++和C的混合编程
*很多芯片厂商都只会提供C库不会提供C++,所以这部分知识是个流程性的技巧性知识*
大体流程如下
首先在c_code.c源文件中编写你要的C语言代码
```c
#include "c_code.h"
void fun1()
{
    printf("hello fun1\n");
}
void fun2()
{
    printf("hello fun2\n");
}
```
*接下来重点是对应头文件的编写方式*
在c_code.h头文件中编写如下代码.**下面这种写法是最严谨的写法**
```c++
#ifndef C_CODE_H
#define C_CODE_H

#include <stdio.h>
#if __cplusplus//注意是大写C，__cplusplus也是固定的命名
extern "C"{//把C语言的函数声明放在这个大括号里面，就会用C编译器来编译链接
#endif
        void fun1();
        void fun2();
#if __cplusplus
}
#endif
#endif//这就完成混合编程
```
**如果编程环境是IDE那么经过上述操作之后，编译器会自动把C代码用C编译器编译，C++用C++编译器编译，但是如果你的环境不是IDE，是工具链的形式还需要做别的操作**
### 工具链下进行混合编程
比如现在我有三个文件 main.cpp fun.c fun.h
**fun.h此时不需要写成上文的形式**
**先对C文件单独gcc编译**
gcc -c fun.c -o fun.o
**再编译链接C++**
g++ main.cpp fun.o -o main
这样就能生成main这个可执行文件了
## 类和对象(非常重要)
C语言的结构体只有变量，C++的结构体既有变量也有函数（方法）
**类的封装性体现在两个方面**：一个是属性(变量)和函数(行为 方法)合成一个整体，二是给属性和函数增加访问权限(公有 私有 保护)
**类是一个抽象概念，会占用空间但不会分配空间，这就好比int占用4字节空间，但是只有在int a的时候才会给a分配空间而不会给int分配空间**
### 类的访问权限
在**类的内部**（作用域范围内）没有访问权限之分，所有**成员可以互相访问**
在类的外部（作用域范围外）访问权限才有意义，public privat(私有) protected(保护)
在类的外部，只有public修饰的成员才能被访问，在没有涉及继承与派生时，private和protected是同等级的，外部不允许访问
### 设计一个类
*一般是把数据设置为私有，方法设置为公有，这样用户就能借助公有方法间接操作私有数据*
```c++
class person
{
    private:
        int my_money;//是私有数据
    protected:
        int age;//是保护数据
    public:
        void dese()
        {
            cout<<"我啥都有"<<endl;
        }
}
//然后用类实例化一个对象，就类似结构体定义个变量
void test()
{
    person lucy;
    lucy.mu_money=200;//这句直接报错 私有变量类的外部不能访问
    cout<<"有多少钱"<<lucy.my_money<<endl;//还是报错不能访问私有变量
    lucy.age=23;//这句直接报错 保护变量类的外部不能访问
    cout<<"年龄多少"<<lucy.age<<endl;//还是报错不能访问保护变量

    lucy.dese();//这个可以正常执行
}
```
**定义一个类的时候，不要给成员初始化，因为类的初衷是抽象出事物的特征和行为，你让他初始化就不抽象了**
### 类默认是私有的
```c++
class person
{
        int my_money;//此时不加权限的前缀
        int age;
    public:
        void dese()
        {
            cout<<"我啥都有"<<endl;
        }
}
//不加权限的前缀还是会报错，因为默认是私有的
void test()
{
    person lucy;
    lucy.mu_money=200;//这句直接报错 私有变量类的外部不能访问
    cout<<"有多少钱"<<lucy.my_money<<endl;//还是报错不能访问私有变量
    lucy.age=23;//这句直接报错 私有变量类的外部不能访问
    cout<<"年龄多少"<<lucy.age<<endl;//还是报错不能访问私有变量

    lucy.dese();//这个可以正常执行
}
```
### 练习：设计一个person类
设计一个person类，person类具有name和age属性，提供初始化函数(init),并提供对
name和age的读写函数(set,get)。但必须保证年龄的有效范围是0-100否则无法给年龄赋值。并提供方法能输出姓名和年龄。
**设计一个类的时候，类内部的函数功能应该尽量单一，比如一个立方体设置长宽高，应该把设置长宽高作为三个不同的函数，而不是用一个函数一下子设置全部参数**
```c++
class person{
    private:
        char name[32];
        int age;
    public:
    //初始化函数 同时给名字和年龄赋值
        void initPerson(char *input_name,int input_age)
        {
            strcpy(name,input_name);//注意包含string.h或#include <cstring>
            if(input_age>0&&input_age<100)
            {
                age=input_age;
            }
            else
                cout<<"age err"<<endl;
        }
    //对名字进行写操作
        void serName(char *input_name)
        {
            strcpy(name,input_name);
        }
    //对名字进行读操作
        char* getName(void)
        {
            return name;
        }

    //对年龄进行写操作
        void serAge(int input_age)
        {
            if(input_age>0&&input_age<100)
            {
                age=input_age;
            }
            else
                cout<<"age err"<<endl;
        }
    //对年龄进行读操作
        int getAge(void)
        {
            return age;
        }
    //显示姓名和年龄
        void showPerson(void)
        {
            cout<<name<<endl;
            cout<<age<<endl;
        }
};
```
**类的难点主要是如何设计一个类**
### 类的大小
**成员函数，不占用类的空间，成员数据占用类的空间**
```c++
class data
{
private:
    //成员数据 占类的空间大小
    int num;
public:
    //成员函数不占用类的空间大小
    void setNum(int data)
    {
        num = data;
    }
    int getNum(void)
    {
        return num;
    }
}
void test()
{
    printf("%d\n",sizeof(data));//输出4，int类型的大小
}
```
所有的函数都是存放在代码区的，成员数据具体是在内存空间的哪个位置，主要看实例化对象，如果实例化对象是局部变量，那就存放在栈区。如果是用new申请的那就在堆区，如果实例化的对象是全局变量就在全局区，如果用const修饰的全局变量那就在文字常量区。
### 分文件实现类
*成员函数在类内部声明，在类外部定义*
就像下面这样
```c++
class data
{
private:
    int num;
public:
    //在类内部 声明
    void setNum(int data)；//在QT中光标移动到分号后面按下alt+回车实现自动类外定义
    int getNum(void)；
}
//在类 外部 定义函数 函数名前面指明作用域
    void data::setNum(int data)
    {
        num = data;
    }
    int data::getNum(void)
    {
        return num;
    }
```
**类的 定义一般都放在头文件，成员函数的定义放在CPP文件**
比如设计一个类，类的名字叫data，一般把类的定义放在data.h，把成员函数的实现放在data.cpp。
*在QT里有个方便的功能，点击添加源文件的选项，选择class开头的选项，然后在弹窗内填写类的名字，就会根据类的名字自动生成.h和cpp文件*
## 对象的构造函数与析构函数
当我们创建一个对象的时候，这个对象应该有一个初始化状态，当对象销毁之前应该销毁自己创建的一些数据。**构造函数(完成对象初始化)和析构函数(对象结束时完成清理工作)**，*会被编译器自动调用*完成对象初始化和清理工作（*对象的初始化和清理工作是编译器强制我们做的，即使你不提供初始化操作和清理操作，编译器也会给你增加默认的操作，只是这个操作不会做任何事*）
### 构造函数的定义
*构造函数：实例化对象的时候系统自动调用，完成成员数据的赋值*
**构造函数语法**：构造函数函数名和类名 相同，没有返回类型，连void都不能有，但是可以有参数，可以重载。
------------------------------------------------------分割线-----------------------------------------------------------
*析构函数：对象释放的时候系统自动调用*
**析构函数语法**：析构函数的函数名是在类名前面加~组成。没有返回类型，连void都不能有，**不能有参数，不能重载**。
```c++
class data
{
public:
    int num;
public:
//构造函数 无参的构造函数
    data()
    {
        num=0;
        cout<<"无参的构造函数"<<endl;>
    }
//构造函数 有参的构造函数
    data(int n)
    {
        num=n;
        cout<<"有参数的构造函数"<<num<<endl;
    }
//析构函数
    ~data()
    {
        cout<<"析构函数"<<endl;
    }
}
void test()
{//实例化对象时，自动调用构造函数
    data ob;
//函数结束的时候，局部对象ob被释放，自动调用析构函数
}
int main(void)
{
    cout<<"第一步"<<endl;
    test();
    cout<<"第二步"<<endl;
    return 0;
}
//上述代码输出 第一步 无参的构造函数 析构函数 第二步
```
*为什么调用无参的构造函数下面的章节进行讨论*
### 构造函数的分类及调用
按照参数分类 无参构造函数和有参构造函数
按类型分类   普通构造函数和拷贝构造函数(复制构造函数)
#### 无参构造函数的调用
**在实例化对象时不加小括号或者小括号内无参数即可调用**
```c++
class data
{
public:
    int num;
public:
//构造函数 无参的构造函数
    data()
    {
        num=0;
        cout<<"无参的构造函数num="<<num<<eendl;>
    }
//构造函数 有参的构造函数
    data(int n)
    {
        num=n;
        cout<<"有参数的构造函数"<<num<<endl;
    }
//析构函数
    ~data()
    {
        cout<<"析构函数"<<endl;
    }
}
void test()
{//调用无参构造函数或者 系统默认的构造函数（空函数）
    data ob;//这种叫隐式调用
    data ob2=data();//这种叫显式调用 运行效果和隐式一样
}
int main(void)
{
    cout<<"第一步"<<endl;
    test();
    cout<<"第二步"<<endl;
    return 0;
}
//上述代码输出 第一步 无参的构造函数num=0 析构函数 第二步
```
#### 有参构造函数的调用
```c++
class data
{
public:
    int num;
public:
//构造函数 无参的构造函数
    data()
    {
        num=0;
        cout<<"无参的构造函数num="<<num<<eendl;>
    }
//构造函数 有参的构造函数
    data(int n)
    {
        num=n;
        cout<<"有参数的构造函数"<<num<<endl;
    }
//析构函数
    ~data()
    {
        cout<<"析构函数"<<endl;
    }
}
void test()
{
    data ob3(10);//这种叫隐式调用
    data ob4=data(10);//这种叫显式调用 运行效果和隐式一样
}
int main(void)
{
    cout<<"第一步"<<endl;
    test();
    cout<<"第二步"<<endl;
    return 0;
}
//上述代码输出 第一步 有参的构造函数10 析构函数 第二步
```
#### 隐式转换调用有参构造
**这种方式要求只有一个成员数据**例如上面代码中只有一个num数据
*这种方式尽量少用*
```c++
class data
{
public:
    int num;
public:
//构造函数 无参的构造函数
    data()
    {
        num=0;
        cout<<"无参的构造函数num="<<num<<eendl;>
    }
//构造函数 有参的构造函数
    data(int n)
    {
        num=n;
        cout<<"有参数的构造函数"<<num<<endl;
    }
//析构函数
    ~data()
    {
        cout<<"析构函数"<<endl;
    }
}
void test()
{
    data ob4=20;//转换成data ob4(10);
}
int main(void)
{
    cout<<"第一步"<<endl;
    test();
    cout<<"第二步"<<endl;
    return 0;
}
//上述代码输出 第一步 有参的构造函数20 析构函数 第二步
```
#### 匿名对象
*匿名对象的生命周期非常短，执行完匿名对象的语句立即释放*
```c++
class data
{
public:
    int num;
public:
//构造函数 无参的构造函数
    data()
    {
        num=0;
        cout<<"无参的构造函数num="<<num<<eendl;>
    }
//构造函数 有参的构造函数
    data(int n)
    {
        num=n;
        cout<<"有参数的构造函数"<<num<<endl;
    }
//析构函数
    ~data()
    {
        cout<<"析构函数"<<endl;
    }
}
void test()
{//下面的就叫匿名对象 没有名字直接调用
    data(20);//执行完当前语句立即结束生命
    cout<<"-----"<<endl;
}
int main(void)
{
    cout<<"第一步"<<endl;
    test();
    cout<<"第二步"<<endl;
    return 0;
}
//上述代码输出 第一步 有参的构造函数20 析构函数 -----  第二步
```
**注意**：在同一作用域下构造和析构的顺序相反（先构造的最后释放）
### 拷贝构造函数
本质上依旧是构造函数，是*对象与对象之间的一个初始化工作*，**是自身对象的常引用**
```c++
class data
{
public:
    int num;
public:
    data(const data& ob)//这就是拷贝构造函数，拷贝构造函数写法都是这样
    {
        cout<<"这是拷贝构造函数"<<endl;
    }
//析构函数
    ~data()
    {
        cout<<"析构函数"<<endl;
    }
}
//上述代码输出
```
#### 系统提供的拷贝构造函数
系统也会提供一个拷贝构造函数，不是空函数，是**执行单纯的整体赋值操作(浅拷贝)**
**如果用户实现了自己的拷贝构造函数，那么系统将调用用户实现的拷贝构造函数**
```c++
class data
{
public:
    int num;
public:
//构造函数 有参的构造函数
    data(int n)
    {
        num=n;
        cout<<"有参数的构造函数"<<num<<endl;
    }
//析构函数
    ~data()
    {
        cout<<"析构函数"<<endl;
    }
}
void test()
{
    data ob1(10);
    cout<<"ob1.num="<<ob1.num<<endl;
    //下面这种情况会调用默认的拷贝构造函数
    data ob2=ob1;
    cout<<"ob2.num="<<ob2.num<<endl;
}
int main(void)
{
    test();
    return 0;
}
//上述代码输出如下
//有参数的构造函数
//ob1.num=10
//ob2.num=10
//析构函数
//析构函数
```
上述代码的输出中，输出信息构造函数和析构函数没有成对出现，因为调用了系统默认的拷贝构造函数
**调用用户实现的拷贝构造函数，若没有赋值操作，此时就会出现下面这种情况**
```c++
class data
{
public:
    int num;
public:
    data(const data& ob)//此时函数内部没有实现赋值操作
    {
        cout<<"这是拷贝构造函数"<<endl;
    }
//析构函数
    ~data()
    {
        cout<<"析构函数"<<endl;
    }
}
void test()
{
    data ob1(10);
    cout<<"ob1.num="<<ob1.num<<endl;

    data ob2=ob1;
    cout<<"ob2.num="<<ob2.num<<endl;
}
int main(void)
{
    test();
    return 0;
}
//上述代码输出
//有参数的构造函数
//ob1.num=10
//这是拷贝构造函数
//ob2.num=642245
//析构函数
//析构函数
```
*由于用户实现的拷贝函数不存在赋值操作，所以此时打印出的ob2.num是个随机值*
#### 拷贝构造函数的调用
**也是分显式与隐式调用,还有比较特殊的等号隐式转换调用**
```c++
class data
{
public:
    int num;
public:
    data(const data& ob)//此时函数内部没有实现赋值操作
    {
        num=ob.num;
        cout<<"这是拷贝构造函数"<<endl;
    }
//析构函数
    ~data()
    {
        cout<<"析构函数"<<endl;
    }
}
void test()
{
    data ob1(10);
    cout<<"ob1.num="<<ob1.num<<endl;
    //隐式调用
    data ob2(ob1);
    //显式调用
    data ob3=data(ob1);
    //等号隐式转换调用 这种方式也适用于多个成员
    data ob4=ob1;//这种方式最常用
    cout<<"ob2.num="<<ob2.num<<endl;
}
int main(void)
{
    test();
    return 0;
}
```

#### 拷贝构造函数调用的时机
**旧对象初始化新对象，才会调用拷贝构造函数**（新旧是从定义上是新定义还是旧定义）
例如如下几种情况都可以正常调用
```c++
data ob1(10);

data ob2(ob1);//调用拷贝
data ob3=data(ob1);//调用拷贝 ob3在当前语句刚刚定义满足 新 的要求
data ob4=ob1;//调用拷贝 ob4在当前语句刚刚定义满足 新 的要求
```
**注意**下面这种情况是不能调用拷贝构造函数的
```c++
data ob1(10);
data ob2;//注意此处的ob2已经定义，尽管没赋值
ob2=ob1;//因为ob2之前就定义过了，在本条语句中是旧的，所以无法调用拷贝构造函数
//这句话仅仅表示赋值，但是不会调用拷贝构造函数
```
### 构造函数的调用规则
C++的这部分问题其实也是变相的督促用户实现自己的构造函数和析构函数
*系统会对任何一个类提供至少3个成员函数：默认构造函数(就是默认无参构造 是空函数)，默认析构函数(空函数)，默认拷贝构造函数(浅拷贝)*
**如果用户提供了有参构造函数，那么会屏蔽系统提供的默认构造函数（默认是无参构造函数），但是不会屏蔽默认的拷贝构造函数**
---------下面的代码就是因为屏蔽默认构造函数导致报错--------------
```c++
class data
{
public:
    int num;
public:
//构造函数 有参的构造函数
    data(int n)
    {
        num=n;
        cout<<"有参数的构造函数"<<num<<endl;
    }
//析构函数
    ~data()
    {
        cout<<"析构函数"<<endl;
    }
}
void test()
{
    data ob1;//正常来说会调用系统提供的默认无参构造函数
    //但是因为用户提供了有参构造导致屏蔽了默认的
    //代码会直接报错，提示找不到无参构造函数
}
int main(void)
{
    test();
    return 0;
}
```
**如果用户提供了拷贝构造函数，将屏蔽系统的默认构造函数 默认拷贝构造函数**
如果不注意上述的这几条屏蔽的关系将会导致一些报错
**对于构造函数:用户一般要实现 无参构造 有参构造 拷贝构造 析构函数**
### 深拷贝与浅拷贝
同一类型的对象之间可以赋值，使得两个对象的成员变量的值相同，两个对象仍然是独立的两个对象，这就叫*浅拷贝*。一般情况下，浅拷贝没有任何副作用，但是**当类中有指针，并且指针指向动态分配的内存空间，析构函数做了动态内存的释放处理，这将导致内存问题**
深拷贝就是为了解决浅拷贝的问题而存在的，只有当类中有指针的时候才会体现出二者的差别
```c++
class person()
{//shift_TAB 左缩进
privata:
    char *name;//存放字符串的一般是指针的方式，数组形式的太小会溢出，太大浪费
    int num;
public:
    //实现用户自己的无参构造函数
    person()
    {
        name=NULL;//让指针初始化指向NULL
        num=0;
        cout<<"无参构造"<<endl;
    }
    //实现用户的有参构造函数
    person(char *input_name,int input_num)
    {
        //由于此处还没有学习new相关的知识，所以此处还是用C语言的方式开辟空间
        //name初始化是NULL，所以应该先给name开辟空间
        name=(char *)calloc(1,strlen(input_name)+1);
    //因为strlen计算长度是不算最后的\0所以需要手动加1留出存放\0的位置
    //calloc的方式申请内存空间可能会失败所以需要判断是否申请成功
        if(name==NULL)
        {
            cout<<"构造失败"<<endl;
        }
        strcpy(name,input_name);//把名字存进去
        num=input_num;
        cout<<"有参构造"<<endl;
    }
    //实现析构函数
    ~person()
    {
        //此时的析构函数是要释放内存空间的
        if(name!=NULL)
        {
             cout<<"空间已被释放"<<endl;
            free(name);
            name=NULL;//释放完之后，重新指向空
        }
        cout<<"析构函数"<<endl;
    }
    //输出信息
    void showPerson(void)
    {
        cout<<"name="<<name<<"num="<<num<<endl;
    }
};//千万别忘了类的结尾 加分号
void test1()
{
    person lucy("lucy",18);
    lucy.showPerson();
    //下面是浅拷贝的问题
    person bob=lucy;//此处会调用系统默认的拷贝构造函数 浅拷贝
}
int main(void)
{
    test1();
    return 0;
}
//上述代码会输出
//无参构造
//name=lucy num=18
//空间已被释放
//析构函数
//空间已被释放
//析构函数
```
如此一来，上述代码就对同一块堆区释放了两次空间
**用户需要自己实现拷贝构造函数来避免上述的问题**
在类中添加如下代码即可
```c++
person(const person& ob)
{
    name=(char *)calloc(1,strlen(ob.name)+1);
    if(name==NULL)
    {
        cout<<"空间申请失败"<<endl;
    }
    strcpy(name,ob.name);
    num-ob.num;
}
```
上述代码便是深拷贝，不但实现了成员变量的拷贝，也避免了浅拷贝的问题
**对一个类的实例而言，无参构造 有参构造 拷贝函数只会调用一种，构造函数可以重载但是只能调用一个**

*一般情况下，在调用用户自己的拷贝构造函数之后不会再次调用有参拷贝函数。因为拷贝构造函数负责创建一个新对象并将其初始化为已有对象的副本，而有参拷贝函数则是用于将一个已存在的对象赋值给另一个已存在的对象。在调用拷贝构造函数时，已经完成了对象初始化的过程，不需要再次调用有参拷贝函数来完成对象的赋值操作。但是，在一些特殊情况下，例如类中存在虚函数或者基类的情况下可能会调用有参拷贝函数，这种情况下需要根据具体情况进行分析。*

## 初始化列表(参数列表)
**初始化列表只能在构造函数中使用**
例子如下
```c++
class data
{
private:
    int m_a;
    int m_b;
    int m_c;
public:
    data(int a,int b,int c):m_a(a),m_b(b),m_c(c)
    {//上面这种写法就叫初始化列表
    //在函数后面加冒号，然后写法 成员名（形参名）就能让abc分别给三个参数赋值
        cout<<"有参构造"<<endl;
    }
    ~data()
    {
        cout<<"析构函数"<<endl;
    }
    //显示数据的函数
    void showData()
    {
        cout<<m_a<<" "<<m_b" "<<m_c<<endl;
    }
};
```
## 类的对象作为另一个类的成员(涉及参数列表)
在类中定义的数据成员一般都是基本的数据类型，但是类中的成员也可以是对象，叫做*对象成员*。**当创建一个对象的时候，C++必须确保调用了 所有 子对象的 构造函数**。如果所有的子对象有默认构造函数，编译器可以自动调用他们。若没有默认的构造函数，C++提供了专门的语法来解决该问题即**构造函数初始化列表**，当调用构造函数时，首先按各对象成员在类定义中的顺序(和参数列表的顺序无关)依次调用他们的构造函数，对这些对象初始化，最后再调用本身的函数体，也就是说，先调用对象成员的构造函数，再调用本身的构造函数，析构函数和构造函数调用的顺序相反，先构造，后析构。

例子如下
```c++
class A
{
private:
    int m_a;
public:
    A()
    {
        cout<<"A无参构造函数"<<endl;
    }
    A(int a)
    {
        m_a=a;
        cout<<"A有参构造函数"<<endl;
    }
    ~A()
    {
        cout<<"A析构函数"<<endl;
    }
};
class B
{
private:
    int m_b;
public:
    B()
    {
        cout<<"B无参构造函数"<<endl;
    }
    B(int b)
    {
        m_b=b;
        cout<<"B有参构造函数"<<endl;
    }
    ~B()
    {
        cout<<"B析构函数"<<endl;
    }
};
class data
{
private://对象成员
    A oba;
    B obb;
    int num;
public:
    data()
    {
        cout<<"data无参构造函数"<<endl;
    }
    //有参构造
    data(int a,int b,int input_num)
    {
        num=input_num;
        cout<<"data有参构造函数"<<endl;
    }
    ~data()
    {
        cout<<"data析构函数"<<endl;
    }
};
void test()
{//先调用对象成员的构造函数 再调用自己的构造函数 
    data ob1;
    //先析构自己 再析构对象成员
}
int main(void)
{
    test();
    return 0;
}//上述代码输出
//A无参构造函数
//B无参构造函数
//data无参构造函数
//data析构函数
//B析构函数
//A析构函数
```
*上述是data ob1无参的例子*
下面是有参的例子
```c++
class A
{
private:
    int m_a;
public:
    A()
    {
        cout<<"A无参构造函数"<<endl;
    }
    A(int a)
    {
        m_a=a;
        cout<<"A有参构造函数"<<endl;
    }
    ~A()
    {
        cout<<"A析构函数"<<endl;
    }
};
class B
{
private:
    int m_b;
public:
    B()
    {
        cout<<"B无参构造函数"<<endl;
    }
    B(int b)
    {
        m_b=b;
        cout<<"B有参构造函数"<<endl;
    }
    ~B()
    {
        cout<<"B析构函数"<<endl;
    }
};
class data
{
private://对象成员
    A oba;
    B obb;
    int num;
public:
    data()
    {
        cout<<"data无参构造函数"<<endl;
    }
    //有参构造
    data(int a,int b,int input_num)
    {
        num=input_num;
        cout<<"data有参构造函数"<<endl;
    }
    ~data()
    {
        cout<<"data析构函数"<<endl;
    }
};
void test()
{//此时系统调用无参构造函数
    data ob2(10,20,30);
}
int main(void)
{
    test();
    return 0;
}//上述代码输出
//A无参构造函数
//B无参构造函数
//data有参构造函数
//data析构函数
//B析构函数
//A析构函数
```
**如果你想调用成员对象的有参构造函数你通过上面这种方式和下面这种方式都是不能完成的**
```c++
class data
{
private://对象成员
    A oba;
    B obb;
    int num;
public:
    data()
    {
        cout<<"data无参构造函数"<<endl;
    }
    //有参构造
    data(int a,int b,int input_num)
    {
        oba.m_a=a;//这样会直接报错这个m_a和m_b是私有变量，你在别的类里无法访问
        obb.m_b=b;
        num=input_num;
        cout<<"data有参构造函数"<<endl;
    }
    ~data()
    {
        cout<<"data析构函数"<<endl;
    }
};
```
**初始化列表可以解决这个问题，能显式调用对象成员的构造函数**
如下所示
```c++
class data
{
private://对象成员
    A oba;
    B obb;
    int num;
public:
    data()
    {
        cout<<"data无参构造函数"<<endl;
    }
    //有参构造
    data(int a,int b,int input_num):oba(a),obb(b),data(input_num)//对象名（形参名）
    {
        cout<<"data有参构造函数"<<endl;
    }
    ~data()
    {
        cout<<"data析构函数"<<endl;
    }
};
```
## explicit关键字(了解性)
C++提供了关键字explicit，禁止通过构造函数进行隐式转换（这种方式容易被看作赋值操作）。**声明为explicit的构造函数不能在隐式转换中使用。**
*注意*：该关键字用于修饰构造函数，防止隐式转换。是针对单参数的构造函数(或者除了第一个参数外其余参数都有默认值的多参构造函数而言)

例子如下所示
```c++
class data
{
private:
    int num;
public:
    data(int a):num(a)
    {
        cout<<"有参构造函数"<<endl;
    }
    ~data()
    {
        cout<<"析构函数"<<endl;
    }
    //输出num值
    void showNum(void)
    {
        cout<<"num="<<num<<endl;
    }
};
int main()
{
    //隐式转换
    data test=10;
    test.showNum();
}//上述代码输出num=10
```
若改为下面的代码就无法进行隐式转换调用
```c++
class data
{
private:
    int num;
public:
    explicit data(int a):num(a)
    {
        cout<<"有参构造函数"<<endl;
    }
    ~data()
    {
        cout<<"析构函数"<<endl;
    }
    //输出num值
    void showNum(void)
    {
        cout<<"num="<<num<<endl;
    }
};
int main()
{
    //隐式转换
    data test=10;//此时直接报错
    test.showNum();
}
```
## 动态对象创建(动态内存分配)
*尽管在C++中依旧可以使用C语言的malloc free calloc realloc等动态内存分配的函数，但是这些函数有缺陷，他们不能帮我们完成对象的初始化工作*
### C语言动态内存分配的缺点
1 用户必须确定对象的长度，才能开辟空间
2 malloc返回一个void指针，c++不允许将void赋值给其他任何指针，必须强制转换
3 malloc可能申请内存失败，所以必须判断返回值来确保内存分配成功
4 用户在使用对象之前必须记住对他初始化，构造函数不能显式调用初始化(构造函数是由编译器调用)，用户可能忘记调用初始化函数。
5 **最大的缺点是malloc calloc realloc不会调用构造函数，free也不会调用析构函数**
**综上所述，C语言的动态内存分配函数太复杂，不方便用，在C++中推荐使用运算符new delete来从堆区申请和释放空间**
### new操作
C++中解决动态内存分配的方案是把创建一个对象所需要的操作都结合在一个被称为new的运算符里。**当用new创建一个对象的时候，它会在堆区为对象分配内存并调用构造函数完成初始化**
### new给基本类型申请空间
```c++
void test1()
{
    //基本类型
    int *p=NULL;
    //传统的申请方法
 //   p=(int *)calloc(1,sizeof(int));
    //new的写法
    p=new int;
    *p=100;
    cout<<"*p="<<*p<<endl;
    //释放空间
    delete p;//注意C语言那一套和C++这一套不能混合使用，比如你用new开辟了P的空间
    //不能用free来释放她，得用delete

}//上述代码会输出 *p=100
//上述代码可以写成p=new int(100)
```
### new给基本类型数组申请空间
给一个5个元素的数组申请空间，传统写法写成这样
```c++
void test()
{
    int *arr=NULL;
    arr=(int *)calloc(5,sizeof(int));
}
```
用new写成这样
```c++
void test()
{
    int *arr=NULL;
    arr=new int[5];//申请空间的时候，内容没有初始化，此时的值是随机的
    //释放数组的时候必须加[]
    delete [] arr;
    //中括号里不需要加入数字，他会自动计算个数
}
```
用new申请完之后可以紧接着初始化
```c++
void test()
{
    int *arr=NULL;
    arr=new int[5]{1,2,3,4,5};
    //释放数组的时候必须加[]
    delete [] arr;
    //中括号里不需要加入数字，他会自动计算个数
}
```
*char和其他类型也是类似的操作*
```c++
void test()
{
    char *arr=NULL;
    arr=new char[32]{'f','u','c','k'};
    delete [] arr;
}//new的同时初始化是有局限性的，只能单个元素单个元素的进行初始化，不能整体初始化
```
*上述代码中的初始化方式若写为arr=new char[32]{"fuck"};会直接报错，虽然现在某些智能IDE也支持这种操作，但还是不要这样写*

写成下面这样即可整体初始化
```c++
void test()
{
    char *arr=NULL;
    arr=new char[32];
    strcpy(arr,"fuck");//这样整体赋值，注意包含头文件
    delete [] arr;
}
```
### new给类申请空间(重点)
**new从堆区申请的都是指针，都是地址**
```c++
class Person
{
privata:
    chat m_name[32];
    int m_num;
public:
//这里是个有参构造函数，会自动屏蔽系统的无参构造函数
    Person(char *name,int num)
    {
        strcpy(m_name,name);
        m_num=num;
        cout<<"有参构造函数"<<endl;
    }
    ~Person()
    {
        cout<<"析构函数"<<endl;
    }
};
void test()
{
    Person *p=new Person;//如果这么写会直接报错提示找不到无参构造函数
    //这是因为用户实现了有参构造函数，所以自动屏蔽掉系统的无参构造函数
}
```
上述这个问题有两种解决方式 1 手动实现无参构造函数
```c++
class Person
{
privata:
    chat m_name[32];
    int m_num;
public:
//手动实现无参构造函数
    Person()
    {
        cout<<"无参构造函数"<<endl;
    }
//这里是个有参构造函数，会自动屏蔽系统的无参构造函数
    Person(char *name,int num)
    {
        strcpy(m_name,name);
        m_num=num;
        cout<<"有参构造函数"<<endl;
    }
    ~Person()
    {
        cout<<"析构函数"<<endl;
    }
};
void test()
{//此处的new按照Oerson类申请空间，如果申请成功，会自动调用构造函数
    Person *p=new Person;
    delete p;//此处会先调用析构函数 再 释放堆区空间
}//上述代码会直接输出 无参构造函数 析构函数
```
2 手动调用有参构造
```c++
class Person
{
privata:
    chat m_name[32];
    int m_num;
public:
//手动实现无参构造函数
    Person()
    {
        cout<<"无参构造函数"<<endl;
    }
//这里是个有参构造函数，会自动屏蔽系统的无参构造函数
    Person(char *name,int num)
    {
        strcpy(m_name,name);
        m_num=num;
        cout<<"有参构造函数"<<endl;
    }
    ~Person()
    {
        cout<<"析构函数"<<endl;
    }
    showPerson()
    {
        cout<<"name is"<<m_name<<"num="<<m_num<<endl;
    }
};
void test()
{
    Person *p=new Person("lucy",18);
    p->showPerson();//由于P是指针所以调用得用这个运算符
    //如果p是普通对象久使用点.运算符
    delete p;
}//上述代码会直接输出 有参构造函数 析构函数
```
### 对象数组
对象数组本质上还是数组，但是数组中的每一个元素都是类的对象
```c++
class Person
{
privata:
    chat m_name[32];
    int m_num;
public:
//手动实现无参构造函数
    Person()
    {
        cout<<"无参构造函数"<<endl;
    }
//这里是个有参构造函数，会自动屏蔽系统的无参构造函数
    Person(char *name,int num)
    {
        strcpy(m_name,name);
        m_num=num;
        cout<<"有参构造函数"<<endl;
    }
    ~Person()
    {
        cout<<"析构函数"<<endl;
    }
    showPerson()
    {
        cout<<"name is"<<m_name<<"num="<<m_num<<endl;
    }
};
void test()
{
    Person arr1[3];//这个就叫对象数组 每个元素是Person类型的对象
}
int main()
{
    test();
    return 0;
}//上述代码会输出
//无参构造函数
//无参构造函数
//无参构造函数
//析构函数
//析构函数
//析构函数
```
*定义对象数组的时候，系统会为数组中的每一个元素自动调用构造函数*。**上述代码中并没有给每个元素传参，所以调用无参构造函数**

**如果想让对象数组中的元素调用有参构造，必须人为显式调用（人为使用有参构造）**
```c++
class Person
{
privata:
    chat m_name[32];
    int m_num;
public:
//手动实现无参构造函数
    Person()
    {
        cout<<"无参构造函数"<<endl;
    }
//这里是个有参构造函数，会自动屏蔽系统的无参构造函数
    Person(char *name,int num)
    {
        strcpy(m_name,name);
        m_num=num;
        cout<<"有参构造函数"<<endl;
    }
    ~Person()
    {
        cout<<"析构函数"<<endl;
    }
};
void test()
{//此处的new按照Oerson类申请空间，如果申请成功，会自动调用构造函数
    Person *p=new Person;
    delete p;//此处会先调用析构函数 再 释放堆区空间
}//上述代码会直接输出 无参构造函数 析构函数
```
2 手动调用有参构造
```c++
class Person
{
privata:
    chat m_name[32];
    int m_num;
public:
//手动实现无参构造函数
    Person()
    {
        cout<<"无参构造函数"<<endl;
    }
//这里是个有参构造函数，会自动屏蔽系统的无参构造函数
    Person(char *name,int num)
    {
        strcpy(m_name,name);
        m_num=num;
        cout<<"有参构造函数"<<endl;
    }
    ~Person()
    {
        cout<<"析构函数"<<endl;
    }
    showPerson()
    {
        cout<<"name is"<<m_name<<"num="<<m_num<<endl;
    }
};
void test()
{
    Person *p=new Person("lucy",18);
    p->showPerson();//由于P是指针所以调用得用这个运算符
    //如果p是普通对象久使用点.运算符
    delete p;
}//上述代码会直接输出 有参构造函数 析构函数
```
### 对象数组
对象数组本质上还是数组，但是数组中的每一个元素都是类的对象
```c++
class Person
{
privata:
    chat m_name[32];
    int m_num;
public:
//手动实现无参构造函数
    Person()
    {
        cout<<"无参构造函数"<<endl;
    }
//这里是个有参构造函数，会自动屏蔽系统的无参构造函数
    Person(char *name,int num)
    {
        strcpy(m_name,name);
        m_num=num;
        cout<<"有参构造函数"<<endl;
    }
    ~Person()
    {
        cout<<"析构函数"<<endl;
    }
    showPerson()
    {
        cout<<"name is"<<m_name<<"num="<<m_num<<endl;
    }
};
void test()
{
    Person arr1[3]={Person("lucy",18)};
    //初始化部分调用有参构造，未初始化部分调用无参构造函数
    arr1[0].showPerson();
}
int main()
{
    test();
    return 0;
}//上述代码会输出
//有参构造函数
//无参构造函数
//无参构造函数
//name is lucy num=18
//析构函数
//析构函数
//析构函数
```
### new申请对象数组
```c++
void test()
{
    //第一种方式
    Person *arr=NULL;
    arr = new Person[3];//此时没指定参数，所以会调用无参构造函数
    delete [] arr;
    //第二种方式
    Person *arr1=NULL;
    //初始化的元素会调用有参构造 没有初始化的部分调用无参构造
    arr1=new Person[3]{Person("lucy",18),Person("bob",20)};
    delete [] arr1;
}
```
**申请空间之后的访问方式与C语言相同**
```c++
void test()
{
    Person *arr1=NULL;
    arr1=new Person[3]{Person("lucy",18),Person("bob",20)};
    (*(arr1+0)).showPerson();//显示第一个对象的信息(arr1+0)是个地址，*表示取出地址中的值
    //此时*(arr1+0)是个普通对象，普通对象的成员函数可以用点运算符访问
    arr1[0].showPerson();//与上面的写法等价
    (arr1+1)->shownPerson();//表示输出第二个元素的人物信息
    delete [] arr1;
}
```
### delete释放void*
如果对一个void*指针执行delete操作，这将可能成为一个程序错误，*除非指针指向的内容是非常简单的*，**因为他将不执行析构函数，导致可用内存减少**
```c++
void test2()
{
    Person *p=new Person("lucy",18);
    p->showPerson();
    void *p1=p;//这行代码不会报错因为void *类型是万能指针类型可以接一切指针类型

    delete p1;
}//上述代码会输出
//有参构造
//name=lucy num=18
```
*上述代码并不会运行析构函数，这就是bug*
delete发现p1指向的类型是void空类型，无法从void中寻找析构函数，但是delete可以为p释放空间，因为p指向的类型是Person类型，这是类，他是有析构函数的。

## 静态成员
在类定义中，它的成员(包括成员变量和成员函数)这些成员可以用关键字static声明为静态的，称为静态成员。不管这个类创建了多少个对象，**静态成员只有一个拷贝，这个拷贝被所有属于这个类的对象共享**(静态成员特点)
**静态成员属于 类，而不是 对象，是所有对象公有的一份东西。无论是静态成员数据还是静态成员函数都是可以添加权限的(比如设置为私有或者公有)**
### 静态成员变量
在一个类中，若将一个成员变量声明为static,这种成员称为静态成员变量。**静态变量是在编译阶段就分配空间，对象还没有创建时，就已经分配空间**
**静态成员变量必须在类中声明，在类外定义。** 静态数据成员不属于某个对象，在为对象分配空间中不包括静态成员所占空间。*静态数据成员可以通过类名或者对象名来引用*
```c++
class Data
{
//此处为了方便演示定义为public 这点不是必须
public:
    int num;
    static int a;//静态成员变量(类内声明)

};
int Data::a=100;//静态成员变量类外定义 一定要加作用域并初始化 定义可以不加static
void test1()
{
    //a是静态成员变量，属于类，就算没有实例化对象 也能访问
    cout<<"a="<<Data::a<<endl;
    //静态成员不代表是只读的，还可以给他修改赋值
    Data::a=200;
    cout<<"a="<<Data::a<<endl;
    //也可以通过对象名访问
    Data ob1;
    ob1.a=300;
    cout<<"a="<<Data::a<<endl;//注意看这两种输出方式
    cout<<"a="<<ob1.a<<endl;
}//上述代码会输出
//100
//200
//300
//300
```
*只要你发现一个类是通过类名::的方式直接访问而不是通过对象名.(注意这有个点运算符)，那应该意识到这个变量就是静态成员变量，普通成员变量只能通过对象名访问*
### 静态成员函数
```c++
class Data
{
private:
    int num;
    static int a;//静态成员变量(类内声明)
public:
    static int geta(void)//这就叫静态成员函数
    {
        return a;
    }//静态成员函数属于类不属于对象

};
int Data::a=100;
void test1()
{
//静态成员函数属于类 可以通过类名直接访问
cout<<Data::geta()<<endl;
//静态成员函数也可以通过对象名访问，对象共享静态成员函数
Data ob1;
cout<<ob1.geta()<<endl;
//普通成员函数依赖对象，只能先实例化对象再访问

}
```
*静态成员函数目的就是操作静态成员数据*
**静态成员函数不能访问普通成员变量**  例子如下
```c++
class Data
{
private:
    int num;
    static int a;//静态成员变量(类内声明)
public:
    static int geta(void)//这就叫静态成员函数
    {
        return num;//直接报错 num是普通成员变量
    }

};
int Data::a=100;
```
**普通成员函数能访问静态变量也能访问普通成员变量**
只不过普通成员函数访问静态成员数据必须先实例化一个对象来访问，否则不行
```c++
class Data
{
private:
    int num;
    static int a;//静态成员变量(类内声明)
public:
    int test(void)//此处是普通成员函数
    {
        return a;
    }

};
int Data::a=100;
void test1()
{
cout<<Data::test()<<endl;//这行代码直接报错 test是普通成员函数
//普通成员函数必须先实例化对象来访问 静态成员数据
Data ob1;
cout<<ob1.test()<<endl;
}
```
### const修饰静态成员
如果一个类的成员，既要实现共享 *(static)* ,又要实现不可改变 *(const)*，这就需要用**const static**修饰。定义静态const数据成员时，最好在类内部初始化
```c++
class Data
{
//此处为了方便演示定义为public 这点不是必须
public:
    int num;
    const static int a;//静态成员变量(类内声明)
public:
    int geta(void)//此处是普通成员函数
    {
        return a;
    }

};
const int Data::a=100;//此处一定要加const前缀
void test1()
{
    cout<<Data::a<<endl;//正常输出100
    //尝试修改之后再输出
    Data::a=200;
    cout<<Data::a<<endl;//报错提示只读
}
```
### 静态成员统计对象个数
静态成员的用处之一
*静态成员是对象共享的，可以作为对象间信息沟通的渠道*
```c++
class Data
{
public:
    Data()
    {
        cout<<"无参构造函数"<<endl;
        count++;//实例化一个对象必定调用构造函数，所以在这里然他递增
    }
    Data(const Data &ob)
    {
        cout<<"拷贝构造函数"<<endl;
        count++;//拷贝构造函数或者有参构造也代表实例化对象 所以在这里递增
    }
    ~Data()//当一个对象释放的时候就代表对象少了一个，可以减去一个
    {
        cout<<"析构函数"<<endl;
        count--;
    }
    static int count;//静态成员
};
int Data::count=0;//类内声明，类外定义，初始化为0
int main(void)
{
    Data ob1;
    Data ob2;
    //下面这种写法叫C++的复合语句 抽空查查详细资料
    {
        Data ob3;
        Data ob4;
        cout<<"对象个数="<<Data::count<<endl;
    }
     cout<<"对象个数="<<Data::count<<endl;
     return 0;
}//上述代码会输出
//无参构造
//无参构造
//无参构造
//无参构造
//对象的个数=4
//析构函数
//析构函数
//对象个数=2
//析构函数
//析构函数
```
### 单例模式设计(重要)
单例模式是一种常用的软件设计模式，在它的核心结构中只包含一个被称为单例的特殊类。
通过单例模式可以保证系统中一个类只有一个实例，而且该实例易于外界访问，从而方便对实例个数控制并节约系统资源。**如果希望在系统中某个类的对象只能存在一个，单例模式是最好的解决方案。简而言之，这种类只能实例化一个对象。**
Singleton(单例)：在单例类的内部实现只生成一个实例，同时它提供一个静态的成员方法
（专业术语叫做getInstance()工厂方法）让用户可以访问它的唯一实例，为了防止在外部对其实例化，**将其默认构造函数和拷贝构造函数设计为私有**，在单例类的内部定义了一个singleton类型的静态对象，作为外部共享的唯一实例。
### 单例模式设计之打印机案例
单例模式设计就类似于公司里的打印机，整个公司里就一台打印机，是公有的，大家都可以用。
*按照下面的步骤来设计一个打印机的单例模式*
步骤一：在单例类内部定义一个Singleton类型的静态对象，作为外部共享的唯一实例。
步骤二：提供一个静态方法，让用户可以访问它的唯一实例。
步骤三：为了防止在外部实例化其他对象，将其默认构造函数和拷贝构造函数设计为私有。

```c++
class Printer
{
private:
    //定义一个静态的 对象指针变量 保存唯一实例
    static Printer *p;//此处是静态变量依旧需要在类外定义
public://提供一个方法让用户获得单例的指针 该方法完成步骤二的目的
    static Printer* getP(void)
    {
        return p;
    }
//第四步 写一个功能函数 前三步每个单例模式都要有
    void printText(char *str)
    {
        cout<<str<<endl;
        count++;
    }
    int count;
private:
    //第三步防止该类实例化其他对象 将构造函数 全部私有化
    Printer(){count=0;}
    Printer(const Printer &ob){count=0;}
    
}
Printer *Printer::p=new Printer;//这部分知识点的关键就是这个地方 第一步完成
int main()
{
    //Printer ob;//直接报错 单例模式禁止实例化多个对象
//打印任务1
    Printer *p1=Printer::p;
    p1->printText("test1");
    p1->printText("test2");
    p1->printText("test3");
//打印任务2
    Printer *p2=Printer::p;
    p2->printText("test3");
    p2->printText("test4");
    p2->printText("test5");
//输出打印任务数量
    cout<<"count="<<p2->count<<endl;
    return 0;
}//上述代码会输出
//text1
//text2
//text3
//text4
//text5
//count=6
```
## this指针(重要)
C++的封装性：将数据 和 方法封装在一起
C++中的数据和方法 是分开存储的
每个对象 拥有独立的数据
每个对象 共享同一块方法
**任何一个对象调用一个方法（非静态成员函数）的时候都会在这个方法中产生一个this指针**
为了能更好的了解this指针的作用，请看下面代码
```c++
class Data
{
public://此处只是为了方便演示所以设置为公有 别的也可以
    int m_num;
    void setNum(int num)
    {
        m_num=num;
    }
};
void test()
{
    Data ob1;
    ob1.ssetNum(10);
    cout<<"ob1.num="<<ob1.num<<endl;

    Data ob2;
    ob2.ssetNum(20);
    cout<<"ob2.num="<<ob2.num<<endl;
}//上述代码会打印
//ob1.num=10
//ob2.num=20
```
为什么ob1 ob2两个不同的对象调用相同的setNum方法却能做到互不影响，就是因为this指针。详细的过程是*当一个对象调用setNum方法时，会在setNum方法中产生一个this指针，this指针指向所调用方法的对象*
### this指针的注意点
**C++的数据和操作也是分开存储，并且每一个普通成员函数（排除内联函数和静态成员函数）只会诞生一份函数实例，也就是说多个同类型的对象会共用一块代码**

1 *this指针是隐含在对象成员函数内的一种指针*
2 成员函数通过this指针即可知道操作的是哪个对象的数据
3 *this指针无需定义，直接使用即可*
4 静态成员函数内部没有this指针，静态成员函数不能操作非静态成员变量
### C++内部对普通函数的处理
此处仅为了解，下面是一个简单的例子
在C++中编写如下代码
```c++
class Test
{
public:
    Test(int a)
    {
        m_a=a;
    }
    int getA()
    {
        return m_a;
    }
    static void print()
    {
        cout<<"class test"<<endl;
    }
private:
    int m_a;
};
Test a(10);
a.getA();
Test::print();
```
上面的代码会被转化成下面的形式
```c++
struct Test{
    int m_a;
}
void Test_initalize(Test *pThis,int a)
{
    pThis->m_a=a;
}
int Test_getA(Test *pThis)
{
    return pThis->m_a;
}
void Test_print()
{
    cout<<"class test"<<endl;
}
Test a;
Test_initalize(&a,10);
Test_getA(&a);
Test_print();
```
这也能看出this指针的作用
### this指针的用途
当形参和成员变量同名时，可用this指针来区分
```c++
class Data
{
public:
    int num;
    //形参和成员名相同
    void setNum(int num)
    {
        //形参num 成员num this->num
        //num=num;//这么写不会报错，但是打印出来的num也不是你想要的num
        //num会是一个随机值,形参的num作用范围是这个函数，根据就近原则
        //num=num编译器会把num的值赋给他自己
        this->num=num;//这样就正常赋值了
    }
}
```
在类的非静态成员函数中**返回对象本身**，可使用return *this
下面的例子是我们想自己实现一个cout功能的函数
```c++
class MyCout
{
public:
    void mycout(char *str)
    {
        cout<<str;
    }
};
int main()
{
    MyCout ob1;
    ob1.mycout("hello");//
    ob1.mycout("C++");
    //上述代码这么写虽然能输出想要的结果但是得分两次写到函数里不能像cout一样
    //可以用<<来完成多次字符串传输
}
```
可以利用this指针改善这个问题
```c++
class MyCout
{
public:
    MyCouy& mycout(char *str)//注意此处是返回引用 MyCout&表示引用，这样节省空间
    {
        cout<<str;
        return *this;//*this表示当前调用该函数的对象
    }
};
int main()
{
    MyCout ob;
    ob.mycout("hello");//因为该函数返回的是当前对象所以这条语句等价于ob这个对象
    //这样就能完成多次赋值
    ob.mycout("hello").mycout("C++");//这行代码会输出hello c++
}//此处认真体会，这是this的精妙之处
```
*有些程序员喜欢定义函数的形参写成和成员变量相同的名字，此处看个人习惯*

***疑问：如果在上述代码中把MyCout&换成MyCout*，return *this换成rreturn this，是不是每次返回调用该函数的对象都会给他重新开辟空间呢？***
## const修饰成员函数
用const修饰成员函数时，const修饰this指针指向的内存区域，成员函数体内不可以修改本类中的任何普通成员变量，*除非成员变量前用mutable修饰。*

const修饰成员函数一般写成下面的样子
```c++
class Data
{
private:
    int data;
    mutable int num;
public:
    void printData(void)const //注意 是在小括号后面写const
    {
        cout<<data<<endl;//写成cout<<this->data<<endl;也可以
        num=20;//num加了mutable所以能修改
    }
    Data()
    {
        cout<<"无参构造函数"<<endl;
    }
    Data(int a)
    {
        num=a;
        cout<<"有参构造函数"<<endl;
    }
    ~Data()
    {
        cout<<"析构函数"<<endl;
    }
};
int main()
{
    Data ob(10);
    ob.printData();
}//上述代码会输出
//有参构造函数
//10
//析构函数
```
## const修饰对象(常对象)
const修饰对象叫常对象
```c++
class Data
{
private:
    int data;
    int num;
public:
    void setdata(int data)
    {
        this->data=data;
    }
    void showdata()
    {
        cout<<"data="<<this->data<<endl;
    }
};
int main()
{
    const Data ob(10);//这就叫常对象 定义之后不能修改成员
    ob.showdata();//还是报错不能修改
    ob.setdata(200);//此时直接报错 显示不能修改
}
```
**因为编译器认为只要是普通的成员函数都有修改成员变量的可能，所以哪怕有的函数没对变量进行修改也依然提示报错**
用const修饰成员函数的方式编写就不报错了
```c++
class Data
{
private:
    int data;
    int num;
public:
    void setdata(int data)
    {
        this->data=data;
    }
    void showdata() const//这样写就相当于告诉编译器这个函数不会修改变量
    {
        cout<<"data="<<this->data<<endl;
    }
};
int main()
{
    const Data ob(10);//这就叫常对象 定义之后不能修改成员
    ob.showdata();//此时可以正常输出
}
```
**常对象只能调用const修饰的函数**
加入const修饰之后，哪怕函数里面修改也不行
```c++
class Data
{
private:
    int data;
    int num;
public:
    void setdata(int data) const
    {
        this->data=data;
    }
    void showdata() const//这样写就相当于告诉编译器这个函数不会修改变量
    {
        cout<<"data="<<this->data<<endl;
    }
};
int main()
{
    const Data ob(10);//这就叫常对象 定义之后不能修改成员
    ob.setdata(200);//直接报错提示不能修改
}
```
## 友元(重要)
类的主要特点之一是数据隐藏，安全性好，类的私有成员无法在类的作用域之外访问。但是有时候需要在类的外部访问类的私有成员。*为了解决这个问题提出了友元函数* *可以把一个全局函数 某个类中的成员函数 甚至整个类声明为友元*
**友元函数是一种特权函数，他破坏了类的封装性**
### 友元语法
**添加friend关键字修饰(只出现在声明处)**，其他类 类成员函数 全局函数都可以声明为友元。
**友元函数不是类的成员，没有this指针，友元函数可以访问对象任意成员属性，包括私有属性**
```c++
class Room
{
friend void friend_visit(Room &room);//这行代码放在哪里都行，类外面也行
private:
    string bedroom;//string类使用需要#include <string>
public:
    string sittingroom;//卧室
public:
    Room()
    {
        this->bedroom="卧室";
        this->sittingroom="客厅";
    }
};
//好朋友访问我的房间 房价包括客厅和卧室
void friend_visit(Room &room)//该函数不是类的成员
{
    cout<<"好朋友访问你的"<<room.sittingroom<<endl;
    cout<<"好朋友访问你的"<<room.bedroom<<endl;
}
void test1()
{
    Room myroom;
    friend_visit(myroom);
}//上述代码会输出
//好朋友访问你的客厅
//好朋友访问你的卧室
```
### 某个成员函数作为另一个类的友元
```c++
class goodFriend
{
public:
    void visit(Room &room)//此时这行代码会报错 不识别Room这个类
    {
        cout<<"好朋友访问你的"<<room.sittingroom<<endl;
        cout<<"好朋友访问你的"<<room.bedroom<<endl;
    }
};
class Room
{
friend void friend_visit(Room &room);//这行代码放在哪里都行，类外面也行
private:
    string bedroom;//string类使用需要#include <string>
public:
    string sittingroom;//卧室
public:
    Room()
    {
        this->bedroom="卧室";
        this->sittingroom="客厅";
    }
};
```
为了解决不识别Room类这个问题，可以把这个类先在前面声明
就像下面这样
```c++
class Room;//先在前面声明
class goodFriend
{
public:
    void visit1(Room &room)
    {//此时还会报错 不识别sittingroom bedroom
        cout<<"好朋友访问你的"<<room.sittingroom<<endl;
        cout<<"好朋友访问你的"<<room.bedroom<<endl;
    }
};
class Room
{
private:
    string bedroom;
public:
    string sittingroom;
public:
    Room()
    {
        this->bedroom="卧室";
        this->sittingroom="客厅";
    }
};
```
在C++中，尽管向前声明了Room类（就是先在前面声明Room类），*只能说明Room类存在不能描述Room类里面的成员，所以会出现上述错误*

为了解决上述问题，我们可以把visit1这个成员函数*定义在所有类的外面，声明在类里面*
```c++
class Room;//先在前面声明
class goodFriend
{
public:
    void visit1(Room &room);
};
class Room
{
private:
    string bedroom;
public:
    string sittingroom;
public:
    Room()
    {
        this->bedroom="卧室";
        this->sittingroom="客厅";
    }
};
void goodFriend::visit1(Room &room)
{
    cout<<"好朋友访问你的"<<room.sittingroom<<endl;
    cout<<"好朋友访问你的"<<room.bedroom<<endl;
}
```
但是尽管上面的问题得到了解决，但是编译器会报错不能访问bedroom这个私有数据，可以访问sittingroom这个公有数据。
```c++
class Room;//先在前面声明
class goodFriend
{
public:
    void visit1(Room &room);
    void visit2(Room &room);
};
class Room
{//为了让visit2访问Room的私有成员，可以把visit2作为Room的友元函数
friend void goodFriend::visit2(Room &room);//一定要加goodFriend这个作用域
private:
    string bedroom;
public:
    string sittingroom;
public:
    Room()
    {
        this->bedroom="卧室";
        this->sittingroom="客厅";
    }
};
//此处定义visit1 visit2只是为了方便对比 便于学习
void goodFriend::visit1(Room &room)
{
    cout<<"好朋友visit1访问你的"<<room.sittingroom<<endl;
}
void goodFriend::visit2(Room &room)
{
    cout<<"好朋友visit2访问你的"<<room.sittingroom<<endl;
    cout<<"好朋友visit2访问你的"<<room.bedroom<<endl;
}
int main()
{
    Room myroom;
    goodFriend.visit1(myroom);
    goodFriend.visit2(myroom);//注意尽管visit2是Room的友元函数
    //但是依旧是goodFriend的成员函数
}//好朋友visit1访问你的客厅
//好朋友visit2访问你的客厅
//好朋友visit2访问你的卧室
```
### 一个类整体作为另一个类的友元
**这种操作便能实现一个类的所有成员函数 访问 另一个类的私有数据，注意一般在C++里没有用数据访问数据的写法，尽量不要像C语一样data1=data2这种方式来访问**
```c++
class Room;//这个声明还是得写上
class goodFriend
{
public:
    void visit1(Room &room);
    void visit2(Room &room);
};
class Room
{//要将goodFriend作为Room的友元类只需 这样写
friend class goodFriend;//虽然不加class这个前缀也对，但是尽量不要
//上述操作便能实现goodFriend所有成员函数都可以访问Room私有数据
private:
    string bedroom;
public:
    string sittingroom;
public:
    Room()
    {
        this->bedroom="卧室";
        this->sittingroom="客厅";
    }
};
void goodFriend::visit1(Room &room)
{
    cout<<"好朋友visit1访问你的"<<room.sittingroom<<endl;
}
void goodFriend::visit2(Room &room)
{
    cout<<"好朋友visit2访问你的"<<room.sittingroom<<endl;
    cout<<"好朋友visit2访问你的"<<room.bedroom<<endl;
}
```
**在C++的类里面，如果成员变量前面没写权限，没标明是公有还是私有，那么编译器是按照默认私有来操作**
## 练习：实现一个数组类















