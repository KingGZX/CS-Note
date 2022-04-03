# For Tencent C++ Basic

## 习题和面经大杂烩

### 1.new操作符问题

```c++
int *p1 = new int[10]; 
int *p2 = new int[10](); 
```

如上代码片段，问初始化问题。

new用于申请一片地址空间，对于内置类型(int char等)除非后面显式地跟上'()'**[默认构造函数]**，才会对这片内存进行初始化且初始化为0。所以像p1是随机值。因此答案为：

**p1申请的空间里的值是随机值，p2申请的空间里的值已经初始化。**

值得注意的是如果是自定义类型(struct class之类的话)，即使不显式地用'()'也会自动调用默认构造函数。

用自己的程序进行验证：

```c++
#include<iostream>
using namespace std;

void show(int* arr, int size){
   for (int i = 0; i < size; i ++){
      cout << arr[i] << " ";
   }
   cout << endl;
}

int main ()
{
   int *p1 = new int[10]; 
   int *p2 = new int[10]();
   int p3[10];
   show(p1, 10);
   show(p2, 10);
   show(p3, 10);
   return(0);
}
```

验证结果：

![image-20211125144443366](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20211125144443366.png)



### 2.指针加问题

如下题所示：

```c++
unsigned char *p1; 
unsigned long *p2; 
p1 = (unsigned char *)0x801000; 
p2 = (unsigned long *)0x810000; 
```

请问p1 + 5 和 p2 + 5后的输出。

**在这里要明白的一点就是'1'只是一个单位量，实际计算时我们真正的操作是：**

**p1 = p1 + 5 * sizeof(unsigned char)，p2 = p2 + 5 * sizeof(unsigned long)。**

可以用自己的程序进行验证：

```c++
#include <iostream>
#include <stdio.h>
using namespace std;

int main ()
{
   unsigned char *p1; 
   unsigned long *p2; 
   p1 = (unsigned char *)0x801000; 
   p2 = (unsigned long *)0x810000;
   printf("%llu\n", sizeof(unsigned char));
   printf("%llu\n", sizeof(unsigned long));
   // printf("%llu\n", sizeof(long long));
   printf("%p %p", p1 + 5, p2 + 5);
   return(0);
}
```

验证结果如下：

![image-20211125150354934](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20211125150354934.png)



### 3.C++虚函数深入学习

首先我们知道一个类的成员函数是不占类的内存的，类的内存是由其数据区+(如有虚函数)一根虚指针构成的。

而**如果一个类什么都没有，其内存占1Byte;但一旦有成员函数或者其他数据这一个Byte就会被忽略。**

如下面的示例程序所示：

```c++
#include <iostream>
#include <stdio.h>
using namespace std;

class A{
   private:
      int a;
      int b;
   public:
      void show();
};

class B{
   private:
      int a;
      int b;
   public:
      virtual void show();    // 会比A多一根指针，而且无论虚函数的数量，总是只多一根指针。
      // 指针 vptr  virtual pointer
      
};

class C{
    
};

void func1(void){
   
}

int main ()
{
   printf("size of pointer : %lld\n", sizeof(int *));
   void (*ptr)();
   ptr = func1;
   printf("size of function pointer : %lld\n", sizeof(ptr));
   printf("size of A : %llu\nsize of B : %llu\nsize of C : %llu\n", sizeof(A), sizeof(B), sizeof(C));
   return(0);
}
```

我这里打印了四个值，指针大小，函数指针大小和两个类的大小。

我是64bit的机器，结果应该分别是 8、8、8、16。其实主要是验证类B比类A多了一根虚指针。

下图所示是侯捷老师的课程截图：

![image-20211125155553281](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20211125155553281.png)

可以看到有虚函数的类就会多出一根虚指针，**<u>而且无论虚函数个数都只有一根指针</u>**！如类A有俩虚函数但只有一个0x409004的指针。这跟指针就是vptr->virtual pointer，虚函数指针。

其指向一个虚函数表->vtbl，表中会去索引第n项，第n项是一个函数指针，解索引后就是相应函数了，因为虚函数也是类的成员函数啊，所以函数有一个参数就是我们说的this指针。

上述分析完就可以知道上图的最下方两个表达式含义了->     **(* p->vptr[n])(p) ||  (*(p->vptr[n]))(p)**  。



### 5.堆栈区，静态区域等的理解

首先我们最最最清楚的 malloc/free   new/delete 都是在堆上操作的，程序员需要手动释放他！

一个函数的局部变量是存放于栈区的->联想栈帧结构，所以一个函数声明结束后那块栈就会被释放！

初始化的静态变量static或者全局变量是存放在静态区域的，未经初始化的在附近的另一块区域。程序结束后由系统释放。

据上，看一道题:

```c
#include<stdio.h>
char *myString()
{
    char buffer[6] = {0};
    char *s = "Hello World!";
    for (int i = 0; i < sizeof(buffer) - 1; i++)
    {
        buffer[i] = *(s + i);
    }
    return buffer;
}
int main(int argc, char **argv)
{
    printf("%s\n", myString());
    return 0;
}
```

注意这里的buffer[6]是一块分配在栈区的内存，最后我们返回的是此块内存的首地址。但是，当myString函数结束后这块栈区会被回收，再访问该地址时，此地址已经变成一个野指针，得到的就是一个随机值了，也有可能程序崩溃。



### 6.指针

```c++
int x[6][4], (*p)[4];
p = x;
```

问*(p + 2)指向哪里。为了更好地产生对比效果，我们以下图为例，其中的a也是一个二维数组 大小[4] [4]：

![1641262876538](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1641262876538.png)

所以再回到题目，p即指向int [4]的指针，所以 p + 2 = p + (sizeof(int [4]) * 2)，sizeof(int [4])刚好是二维数组一行的的大小，两倍的话就是两行，所以就是从二维数组第一个开始加上两行，**就会到达 x[2] [0]**。



而对于图中的指针数组，意为数组中的每一个元素都是一个指针。



### 7.C++模板类

如图所示题：

![1641263244016](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1641263244016.png)

选C的原因是，在“**编译时**”就能检查数据类型。

不选A的原因是， 模板声明的参数类型T可以是int、double等内置类型，也可以是自定义结构类型，如果在自定义结构类型中再使用模板，就可以在外部改变struct内部的类型，而不需要去原数据结构中做改动，相当于实现了动态扩展和缩小。 



### 8.结构体对齐规则

应该查看CSAPP，复习自己的CSAPP--ch03(三)笔记。

本题中：

```c++
class A{
    int a;
    short b;
    int c;
    char d;
};
class B{
    short b;
    int c;
    char d;
};
```

sizeof(A) = 4 + 2 + (padding 2) + 4 + 1 + (padding 3) = 16

sizeof(B) = 2 + (padding 2) + 4 + 1 + (padding 3) = 12

至于为什么有填补位padding，可以参照下面的规则：

![1641264626430](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1641264626430.png)



### 9.



### 10.include写法的区别

如下图所示：

![1641267220859](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1641267220859.png)





### 11.仍旧是虚函数

```c++
#include<stdio.h>
class CParent {
    public: virtual void Intro(){
        printf( "I'm a Parent, " ); Hobby();
    }
    virtual void Hobby(){
        printf( "I like football!" );
    }
}; 
class CChild : public CParent { 
    public: virtual void Intro(){
        printf( "I'm a Child, " ); Hobby();
    }
    virtual void Hobby(){
       printf( "I like basketball!\n" );
    }
}; 
int main( void ){
    CChild *pChild = new CChild(); 
    CParent *pParent = (CParent *) pChild; 
    pParent->Intro(); 
    return(0);
}

```

可以看见pParent是指向基类的指针，且是由子类指针转换而来的，C++认为这样是安全的。且当我们调用虚函数是仍旧会去调用衍生类中的声明的那个，所以最后就是   “I'm a Child， I like basketball!”。



### 12.C++什么类不能被继承

1.将自身的构造函数与析构函数放在private作用域。这样一来衍生类无法调用基类的构造函数就会报错。



2.将自身作为一个已存在类的友元类。   引用别人的案例： [原博地址](https://blog.csdn.net/qq_34796146/article/details/108065409)

```c++
class CFinalClassMixin {//从这个类中继承的类都不能再被继承
  friend class Cparent;
private:
  CFinalClassMixin() {}
  ~CFinalClassMixin(){}
};
class Cparent: virtual public CFinalClassMixin, public CXXX {
public:
  Cparent() {}
  ~Cparent(){}
};
class CChild : public Cparent {
public:
  CChild() {};//编译错误
  ~CChild() {};//编译错误
};

```

这里要注意几点：

1.要申明友元类，友元类可以访问私有成员函数。

2.要执行虚继承，虚继承的一个特点就是虚基类的构造函数由最终子类完成。按上述代码，最终子类实际上应该是CChild，但此类并不是虚基类的友元类，因此仍旧无法调用构造函数。

为了更加泛化，或者说很多类都想成为其友元类，每个都往上加非常繁琐，我们用模板的方法操作：

```c++
template<class T>
class CFinalClassMixin {//从这个类中继承的类都不能再被继承
friend class T;
private:
  CFinalClassMixin() {}
  ~CFinalClassMixin(){}
};
class Cparent: virtual public CFinalClassMixin<Cparent>, public CXXX {
public:
  Cparent() {}
  ~Cparent(){}
};
```

虚继承机制可以让从多个衍生类继承的下来的类只保留基类的一份拷贝。

```c++
class A{};

class B : virtual public A{};

class C : virtual public A{};

class D : public B, public C{};
```

就如上述代码，D中可以直接对A初始化，最后只有一个基类对象。 否则很有可能B和C构造的基类A对象不同导致保留两份拷贝，会产生二义性。



3.final关键字，可以禁止类被继承。同时，题外话，还可以阻断虚函数的重载。

```c++
// 用于禁止类继承时
class A final{

}；
    
// 用于禁止重载（只适用于虚函数）  --------用结构体的方式解释，来自"知乎"
struct Base
{
    virtual void foo();
};

struct A : Base
{
    void foo() final; // Base::foo 被覆盖而 A::foo 是最终覆盖函数
    void bar() final; // 错误：非虚函数不能被覆盖或是 final
};

struct B final : A // struct B 为 final
{
    void foo() override; // 错误：foo 不能被覆盖，因为它在 A 中是 final
    // override还能检测出另外一些错误，比如你在子类中想重写父类方法，但可能子类的函数
    // 参数写错，从父类的int被你误写为double，那么编译器将报错告诉你这不是重写，
    // 因为两者不被认为是同一函数。 又比如父类的是常函数，但子类漏了const，也无法override。
};

struct C : B // 错误：B 为 final
{
};
```

