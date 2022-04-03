# Effective C Plus Plus  --Third Edition

C++关键字 "explicit"。此关键字用于修饰带参数的类的构造函数，它的作用在于表明此构造函数是显示的而不是隐式的。这可以阻止进行隐式转换。



### 1. View C++ as a federation of languages

所谓联邦就是说C++是由一系列的次语言组成的，主要有以下几点：

1. C。C++说到底是基于C语言的，如指针等。但C语言的缺点，如没有模板，没有异常处理等反映了其局限性。
2. Object-Oriented C++。面向对象的C++其实就是最原始的诉求，即”包含类的C语言“。其特性就是继承、多态、虚函数等。
3. Template C++。泛型编程的一部分。
4. STL。是一个template程序库，各种容器、迭代器、算法等。

### 2. Prefer consts，enums，and inlines to #define

其实本质上就是说“用编译器替代预处理器是一种更佳的方案”。

总结到最后是两点：

1.对于单纯常量，最好以consts对象或enums替换#define。**可详细了解enum的使用方式和工作原理。**

```c++
// use enum
class A{
    private:
        enum {Num = 5};
        int score[Num];
        /*
        // a int type variable is not valid to describe a array
        int tempNum = 10;
        int B[tempNum];
        */
};
```

如上所示我们知道tempNum是不能这样修饰数组的，const int就可以，同样enum枚举出来的也行！理论基础就是“一个属于枚举类型的数值可权当int被使用”，但我们无法取enum的地址同时enum不会产生不必要的内存分配。

2.对于形似函数的宏(macros)，最后改用inline函数进行替换。

```c++
一个形似函数的宏：
#define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))
用函数加以替换：
template<typename T>
inline void CALL_WITH_MAX(const T& a, const T& b){
    f(a > b ? a : b);
}
```

我们需要为宏中的每个实参加上小括号，但即使如此在下例中也会出现问题：

```c++
int a = 5, b = 0;
CALL_WITH_MAX(++a, b);
CALL_WITH_MAX(++a, b+10);

注意我们在真实引用宏时的操作实际上是把 ++a看做一个整体的！
所以对于第一次调用->    f((++a) > (b) ? (++a) : b)
实际上，a累加了两次！！ 显然这并不是我们想象的那样。
```

具体测试如下图所示，可以看见最后输出了7，也就是a累加了两次。~~(为实参去掉括号是一样的哈)~~

![1639742699365](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1639742699365.png)

修改为inline函数即可：

![1639742906511](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1639742906511.png)

### 3. Use const whenever possible

对于常指针、常量、指向常量的的常指针等复习点可以看到下面：

```c++
/*
此文件用于对指针进行一些小测试
*/

#include <bits/stdc++.h>

using namespace std;

int main(){
    int x = 5;
    int y = 6;


    // case1, a normal pointer
    int* p = &x;
    cout << "value of pointer1 is:" << *p << endl << "location of pointer1 is:" << p << endl;
    // amend value
    *p = 7;
    cout << "value of pointer1 is:" << *p << endl;
    // amend location
    p = &y;
    cout << "location of pointer1 is:" << p << endl;


    // case 2: a const int variable which the pointer points to!
    const int* p2 = &x;
    cout << "value of pointer2 is:" << *p2 << endl << "location of pointer2 is:" << p2 << endl;
    // the following code is wrong because the type of *p2 is const int, it's a read-only variable!
    // *p2 = 10;
    // cout << "value is:" << *p2 << endl << "location is:" << p2 << endl;
    
    // however you can still change the location the pointer points to!
    // so the "const int" only means the type of the variable which the pointer points to is read-only
    p2 = &y;
    cout << "value of pointer2 is:" << *p2 << endl << "location of pointer2 is:" << p2 << endl;


    // case 3: a const pointer!
    int* const p3 = &x;
    cout << "value of pointer3 is:" << *p3 << endl << "location of pointer3 is:" << p3 << endl;
    // you can change the value, because the type of the variable which pointer p3 points to is only a int 
    *p3 = 10;
    cout << "value of pointer3 is:" << *p3 << endl << "location of pointer3 is:" << p3 << endl;

    // you can't change the location of which the pointer p3 points to anymore
    // p3 = &y;
    // cout << "value is:" << *p3 << endl << "location is:" << p3 << endl;


    // case 4: a const pointer points to a const variable!
    const int* const p4 = &x;
    // you just can't change anything! Because all things are const
    cout << "value of pointer4 is:" << *p4 << endl << "location of pointer4 is:" << p4 << endl;

    return 0;
}
```

把const写在类型之前或者类型之后且星号之前都是指所指代的对象是常量。如：

```C++
const Widget* pw;
Widget const* pw;
都是一样的，Widget是一个作者示例类罢了，pw指pointer to Widget
```

返回常对象的函数可以避免程序员一些低级的错误，比如“a * b = c”，加入我们的operater*返回的是常对象就无法对其进行修改，编译器会直接报错。这可能看起来比较傻**<u>[但也并非无法解释，或许你只想做一个比较操作但却少打了一个‘=’？]</u>**，但或许错误就这么不经意的发生了。



***const成员函数：***

我们常用const成员函数来处理常对象，我们不可忽视的一点就是两个成员函数如果只是常量性不同，也是一种“重载”！用一张图来简单地显示这种重载现象：

![1639821355761](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1639821355761.png)



bitwise const概念阵营的人认为一个const成员函数内不能修改任何non-static元素。与之相对应的就是logical const。但前者在逻辑上会存在一些问题，如下代码所示：

```c++
class A{
    private:
        int x;
        int *p;
    public:
        void change() const {   // 这是一个常函数
            *p = 10;
        }
};
```

你会发现上述代码是可以通过编译的，但显然指针指向的值被改变了，这是反直觉的！又如一个更隐蔽的：

```c++
class Text{
    private:
        char* text;
    public:
        Text(char* s):text(s){};
        char& operator[](int position) const {
            return text[position];
        }
};

int main(){
    char str[6] = "Hello";
    const Text txt(str);
    char* ch = &txt[0];
    *ch = 'J';
    return 0;
}
```

明明还是一个const常成员函数，但是我们最后仍旧修改了text的第一个元素，虽然是在主函数内进行的。

由此引申出来的logical constness就是可以在用户侦测不到的情况下改变对象的一部分属性。但显然如下的操作是**不允许**的：

```c++
class Text{
    private:
        char* text;
        bool alpha = false;
    public:
        Text(char* s):text(s){};
        void change() const{
            if(!alpha)
                alpha = true;
        }
};
```

因此这里就用到了一个与const相关的变动场，利用关键字“mutable”使得常成员函数中可以修改部分属性，如下所示就可以通过编译：

```c++
class Text{
    private:
        char* text;
        mutable bool alpha = false;
    public:
        Text(char* s):text(s){};
        void change() const{
            if(!alpha)
                alpha = true;
        }
};
```



**在const和non-const函数中避免重复**

我们刚刚谈过这两者实现了C++语言的“重载”特性，但我们应该避免大量的重复操作，这样会产生低效的代码。

如下图所示：

![1639838667183](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1639838667183.png)

显然我们比较简单的一种方式就是将我们要实现的三个功能都封装成用户函数，里面直接调用即可。但显然。。。这仍旧是重复的，并不是最优的方案，**我们的最佳选择是让non-const函数去调用const成员函数即可！**

![1639839137241](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1639839137241.png)

如上图所示，我们需要熟悉static_cast和const_cast的操作。static_cast帮助我们进行了第一次安全转型，从non-const对象转换为const对象，而const_cast则帮助我们进行第二次转换将返回类型的对象变回char&，取消其常量性。这样的代码可能有点丑，但实践证明这是最高效的，所以不妨学习着使用它！



### *4. Make sure the objects are initialized before they're used

通常来说，C part of C++的初始化要花费一定成本因此不保证一定会发生初始化。而C++部分会发生。可以联系C++基础知识里的int a[10]输出的是一堆随机初始化的东西。但加入你初始化容器如vector<int> vec(10)就会发现里面是10个0，因此我们较容易得到上面的结论。



当然了我们也都知道初始化是一个很重要的步骤否则程序会莫名发生一些离奇的错误。

接下来我们要辨析一下初始化(initialization)和赋值(assignment)的区别，如下图所示我们称为赋值：

![1639840266264](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1639840266264.png)

初始化动作都会发生在进入构造函数主体之前，所以我们更推荐另一个版本的更高效的构造函数：

![1639841122932](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1639841122932.png)

除此之外我们强烈建议将初值列**(构造函数里的元素)**列出的成员变量按照类中的成员声明顺序排序。



non-local static



### 5.Know what functions C++ silently writes and calls

当你写了一个空类时你的编译器会自动帮你补全诸如构造函数、析构函数等成员函数当且仅当在你调用他们时。例如如下一段编译并不会报错的代码段：

```c++
class Empty{};

int main(){
    Empty e1;    // default构造函数
    Empty e2(e1);    // 拷贝构造函数
    e2 = e1;		// copy assignment 赋值操作符
    return 0;
}
```

虽然你没有显示地声明该有的成员函数，但编译器仍旧自动帮你生成了。

对于拷贝赋值操作符有一些值得注意的点，编译器自动生成的无非就是把类中non-static成员变量拷贝到目标对象。但在一些情况下编译器是会拒绝生成的，比如修改引用、修改常量等，下图示例修改引用：

![1639893853756](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1639893853756.png)

还有一种情况就是base_class若为copy assignment设为private成员函数，那么衍生类同样拒绝为这些derived_class生成拷贝赋值函数。因为想象中衍生类的拷贝赋值函数应该可以操作基类成分，但显然他又是无法调用基类的拷贝赋值函数。



### 6.Explicitly disallow the use of compiler-generated functions you do not want

比如对一个独一无二的类我们并不希望其他人做出一份拷贝，所以我们不允许拷贝构造或者是拷贝赋值的行为发生。这种情况下我们用到的一个伎俩就是将拷贝构造或者拷贝赋值函数声明到private中且无需具体实现它。

但即使这么做导致假如我们在其他成员函数或者友元函数中执行拷贝操作时发生的是**链接(ld)错误**，如下图：

![1639902529559](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1639902529559.png)

我们企图用一种更佳的办法或者更精妙的技巧使得编译时就会直接报错，我们可执行文件生成过程可以稍微提一下“预编译->编译->汇编->链接”，所以尽早发现错误总归是好的。

我们通过代码来看这个技巧：

```c++
#include <iostream>
#include <stdio.h>
using namespace std;

class Base{
    public:
        Base(){};
    private:
        Base& operator=(const Base& obj);
        Base(const Base& obj);
};

class Text:public Base{
    private:
        int sz;
    public:
        Text(int s):Base(), sz(s){};
        void copy(const Text& tx){
            *this = tx;
        }
};

int main(){
    Text tx1(1);
    const Text tx2(2);
    tx1.copy(tx2);
    return 0;
}
```

我们事先声明了一个基类，在其private中将拷贝操作均加以声明但没有定义。此时将我们需要的类作为其衍生类继承它，此时假如我们再用成员函数(抑或友元函数)进行拷贝操作的话会得到如下编译错误：

![1639903251514](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1639903251514.png)

第一个error是说Text类的拷贝操作是缺失的，但实际上我们知道编译器是会帮我们自动生成的，那为何现在不行？所以再看第二个错误，Base基类的拷贝赋值函数是私有的。所以联系起来理由就明确了，我们编译器实际上还是会生成相应拷贝赋值函数的，但此函数是需要去调用基类拷贝赋值函数的，此时就出了大问题因为私有性导致没法调用，最后总之我们的编译器就是自动生成失败了所以报了以上两个错。-------**这里就体现了继承的特性，当我们对子类拷贝赋值或者构造时，相应的基类也要同时进行拷贝操作！**



### 7.Declare destructors virtual  in polymorphic base classes

带多态性质的基类需要将析构函数声明为虚函数，避免内存泄漏等问题。看以下代码：

![1639910099406](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1639910099406.png)

我们并未将基类析构函数设置为virtual。此时我们创建一个Text对象，但指针却是指向基类的，这是完全可行的！但问题在于我们释放内存时，可以通过输出观察只对基类进行了释放，导致了Text对象内存泄漏。但只要将析构函数设置为虚函数，上述现象就不会发生！



但无端把类中的析构函数设置为虚函数并不是值得推崇的方法，因为虚函数机制导致类中会多出一根虚指针指向一张虚表，此虚表中是函数指针，我们取出调用即可！这样会一定程度上造成可移植性的问题。

不要去继承带有non-virtual析构函数的类！比如string，STL容器等等，这会有大麻烦！

有时候让一个抽象类包含一个纯虚析构函数也是一种不错的方案，在需要注意的是要为纯虚析构函数提供一个定义，否则一步步析构到这个基类时会因为析构函数未定义而报错。代码如下所示：

```c++
class Object{
    virtual ~Object() = 0;
};

Object::~Object(){}
```



### 8.Prevent exceptions from leaving destructors

析构函数不要吐出异常，对此我们有两种方案：

![1639913981285](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1639913981285.png)

如上图所示，我们选择直接将程序终止或者第二种方式吞掉异常。看起来第二种并不是特别合理，违反了我们的处理异常的初衷，但这在某些时刻也是很有作用的！

第二点需要记住的是，如果客户需要对某个操作函数运行期间抛出的异常做出反应，那么class应该提供一个普通函数(而非在析构函数中)执行该操作。 但实际上我们在析构函数中还是会加一个双保险操作，比如以关闭数据库连接为例，我们提供一个普通函数close执行断开连接，然后设置closed bool值为true，但我们在析构函数中仍旧会先判断bool值，如果为false，我们仍旧执行断开的操作，这就是“双保险”。



### 9.Never call virtual functions during construction or destruction

看下面一段在构造函数期间调用虚函数的代码并分析其所引发的问题：

```c++
class Transaction{
    public:
        Transaction();
        virtual void logTransaction() const{};
};

Transaction::Transaction(){
    // .. 操作1
    // .. 操作2
    logTransaction();    // 调用虚函数
}

class BuyTransaction:public Transaction{
    public:
        BuyTransaction();
        virtual void logTransaction() const{};
};

BuyTransaction::BuyTransaction(){
    Transaction();
    // logTransaction();
}
```

Transaction是基类，表示一笔交易，logTransaction代表我们登记一笔交易，我们根据交易类型的不同进行不同的操作，因此设置为虚函数，但关键问题就在于**我们在构造函数中调用了“虚函数”**。

我们的derived class在进行base class初始化时，调用base class的构造函数，**此时base class构造函数调用的登记交易函数是自身的那个**！通俗来说，base class构造期间，virtual函数不是virtual函数，所以登记交易函数用错了。因此我们才说不要在构造函数调用虚函数。

我们要了解，当这个derived class先执行base class的构造函数时，此时derived class里的成员变量都是未定义的，编译器只会默认进来的是一个base class对象。同样当derived class执行析构时，derived class里的成员变量也都会变为未定义，进入base class的析构函数时又会默认当前进来的就是一个base class对象。

那么像这种一定要在构造抑或析构函数中执行的操作就改为non-virtual，把基类所需要的数据从derived class往上传递进行构造，然后各自执行相应的non-virtual成员函数即可。



### 10. Have assignment operators return a reference to *this

联想我们的连续赋值操作：

```c++
int x, y, z;
x = y = z = 5;

int p, q, j;
p = (q = (j = 10));
```

更准确地说，下面这个表达式是上面的解析式！这样一来就会会发现我们列举的这条实际上是已经被采用的准则，因为只有(y = 10)返回一个对象，才能继续对q赋值啊。

实际上，也确实是如此的，诸如string、vector、shared_ptr等也都是这么做的。更广泛地来说，我们不仅是在赋值操作时返回一个指向自身的指针，对所有赋值操作运算符，比如+=、-=、*=等都应如此。

于是，我们在设计类重载操作符时，应有如下普适的代码：

```c++
class Widget{
    private:
        // ... 
    public:
        // rhs指的是右手端 right hand side
        Widget& operator+(const Widget& rhs){
            // 把this里的成员变量都设置成和rhs一样
            return *this;
        }
        Widget& operator+=(const Widget& rhs){
            // 同上
            return *this;
        }
        // etc.
};
```

当然了，参数可以任由你设定，我只是默认两个相同类型的对象进行赋值操作运算了，核心是返回*this。



### 11. Handle assignment to self in operator=

自我赋值的行为看起来有点愚蠢，但谁也不能完全避免它。尤其是在“继承”这个强大特性的加持下，一个基类指针也可以指向衍生类对象，或许有时候难免会反应不过来两个指针实际上指的是同一对象！

那么我们就需要先探讨，operator=操作符在一般情况下我们是如何操作的，看如下代码：

```c++
class Bitmap{};
class Widget{
    private:
    	Bitmap* ptr;
   	public:
    	Widget& operator=(const Widget& rhs){
            delete ptr;
            ptr = new Bitmap(*rhs.ptr); //rhs.ptr只是一根指针，解引用成Bitmap对象后，用拷贝构造
            return *this;   //见条款10
        }
};
```

此时问题来了，如果rhs和*this是同一个对象怎么办？我们在delete掉ptr后rhs的ptr也已经成为一个空指针了，此时this和rhs对象的bitmap一起被删了，但我们仍在对一根空指针解引用，那么显然犯了忌讳。

对上述问题一个较为容易的修改是：

```c++
Widget& operator=(const Widget& rhs){
			if(*this == rhs) return *this;
    
            delete ptr;
            ptr = new Bitmap(*rhs.ptr); //rhs.ptr只是一根指针，解引用成Bitmap对象后，用拷贝构造
            return *this;   //见条款10
        }
```

这至少会帮我们避免一定的安全性问题，但避免不了“异常安全性”问题。这里我们考虑比如因为内存不够或Bitmap拷贝构造时抛出异常的问题，这样一来这根ptr指针还是会成为一根“问题指针”。

当今我们更多的把注意力放在“异常安全性”上，这往往会收到“自我赋值安全性”的回报，所以我们的代码如下：

```c++
Widget& operator=(const Widget& rhs){
			Bitmap* porig = ptr;
            ptr = new Bitmap(*rhs.ptr); //rhs.ptr只是一根指针，解引用成Bitmap对象后，用拷贝构造
    		delete porig;
            return *this;   //见条款10
        }
```

我们为原Bitmap做了一份附件porig，删除原Bitmap然后指向新创建的复件。这一版最主要的是修改上一版的一个小bug，即(*this == rhs)认同测试失败时，且new操作失败，则this对象的ptr指针最后指向被删除的一块地址。但改版后的不会在new失败时产生异常。



### 12.Copy all parts of an object

前面我们讲过编译器有时候会为你的代码自动生成缺省的copy assignment或者copy操作符相关代码。当然你完全可以随心所欲，自己写一个operater=或者拷贝构造的函数，只要语法没问题编译器就不会报错。

书中较为形象得描述了“编译器的报复性行为”，含义如下：

当你为你的类新增一个成员变量时，编译器既有的copy assignment或者copying操作符并不会报错，但显然这不是你想要的结果，这就是“报复性行为”。此时要做的就是修正copy assignment和copy操作符函数，将这个新增的成员变量也进行拷贝即可。

还有一点值得注意的是derived class的拷贝函数。我们很容易写出以下形式的代码：

![1640662898877](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1640662898877.png)

![1640662912890](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1640662912890.png)

我们确实将衍生类的priority成员变量进行了拷贝，但基类被我们遗忘，此时编译器就会用默认不带实参默认构造函数对此类中的base class即Customer成分进行初始化，这也许并不是我们想要的结果。

所以较好的办法就是别忘了调用一下base class的拷贝操作，如下所示：

![1640663051266](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1640663051266.png)



最后一点，千万不要让所谓的“精简代码”在两个具有非常近似的本体的copy函数之间出现，逐一分析：

1. 让copy assignment调用拷贝构造函数，我们让一个已经构造好的对象去执行拷贝构造函数好像不是很说得通。
2. 让拷贝构造函数执行copy assignment操作，让一个未成形的对象执行赋值操作，也不是很说得过去。

我们确实把赋值系列操作进行封装然后让两个函数共同调用，但上面讲的互相调用没有尝试的必要！



### 13.Use objects to manage resources

我们都知道要去释放堆上new/malloc出来的资源，用析构函数来返还类的资源，现在假设存在以下代码：

```c++
class Investment{
	public:
		Investment *createInvestment();
		void p(){
			Investment *ptr = createInvestment();
			// some operations
			delete ptr;
		}
};
```

**我们常常将类似于createInvestment的函数称为“工厂函数”[Factory Function]**，他返回了一个类对象指针。很显然我们在p函数中调用了工厂函数并且要用这个指针做一系列操作最后释放资源。

现在的问题就在于你确保你的operations一定正确吗？或者说你的函数一定会经过delete操作最后结束吗，恐怕未必，你可能提前return掉了。或者说另外形势下，delete在循环体内部，那么你如何确保你的循环break之类操作的时候不会忘记delete呢？最后，你的代码要交给他人维护的，他人不一定注重这个点，那么还是有错误风险。

此时一种较好的方式就是用shared_ptr，下面会略微介绍一下。**【书中的auto_ptr是C++98标准的，但现在是C++17啦！！】**，就是C++的智能指针。**这些指针(还有一个unique_ptr全部包含在C++的<memory>头文件中)！**

![1640706912806](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1640706912806.png)

可以看上图，我们并没有手动对ptr这一智能指针进行释放，但结果很明显，delete了两次，自然是自动释放了。用auto_ptr也可以，但编译会有一些warning，unique_ptr也是没有任何warning的，他和shared_ptr是C++新标的解决方案，更受推崇！

auto_ptr有一个bug，就是如果多个auto_ptr指向同一对象，那么多次delete是会报错的。例：

```c++
double* pa = new double;
*pa = 2.5;
auto_ptr<double> ptr(pa);
delete pa;
return 0;
// 程序return后会继续调用auto_ptr的delete，但此内存已被手动delete掉，会有异常产生。
```

auto_ptr还有一个特性，如果对他们调用拷贝构造或者拷贝赋值则此原ptr会变为null而拷贝所得的指针会获得资源的唯一控制权。如以下代码所示：

![1640788291057](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1640788291057.png)

我们刚刚说的auto_ptr一系列的问题显然并不能满足所有的行为，所以才有了“智能指针”，这种指针追踪共有多少对象指向该笔资源，只有在无人使用时才会删除改对象。同时shared_ptr的复制行为也是正常的，多个指针会指向同一对象，且不会变为null。

由于这类指针的析构函数做的都是“delete ptr”操作，所以在array身上用此类指针并不好！



### 14.Think carefully about copying behavior in resource-managing classes

比如我们的锁对象Mutex，为了确保一个被锁住的资源会被解锁最后释放我们建立一个类管理。

```c++
class Lock{
    private:
    	Mutex* mutexPtr;
    public:
    	explicit Lock(Mutex* ptr):mutexPtr(ptr){
            lock(mutexPtr);
        };
    	~Lock(){
          unlock(mutexPtr);  
        };
};


// main operations
Mutex m;
{
    Lock l1(&m);
    // ...   operations
    // 最后的析构函数会自动进行释放
}
```

对于这样的资源管理类，我们会好奇这么一个问题，其拷贝函数如何处理？两种解决方案：

1.不允许拷贝。详见条款6。

2.对底层资源实施“计数法”，就像我们对shared_ptr实施的那样。我们将类里的资源，就好比上面那个Lock类里的mutexPtr改为 shared_ptr<Mutex> mutexPtr即可，但我们知道智能指针在引用数为0时会把资源进行删除。这并不是我们想要的，我们只是要解锁罢了，这里是这样的：

![1640792266851](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1640792266851.png)

可以从上图看出类的析构函数并未被声明，因为其默认会调用shared_ptr的删除器。

3.复制底部资源。也就是说复制管理对象时我们要进行深度拷贝。

4.转移底部资源的控制权。这其实就是auto_ptr的存在意义，就是拷贝后原资源就变为null了，正是我们所谓的“控制权转移”。



### 15.Provide access to raw resources in resource-managing classes

虽然用了"智能指针"对资源进行了管理，但我们不可避免地要用到原始指针。

```c++
class Investment{
    public:
        Investment* createInvestment();
        void p(){
            shared_ptr<Investment> ptr(createInvestment());
            deal(ptr);
        }
        void deal(const Investment* ptr){}
};
```

如上图我们在"13"的基础上**(没有补充完整工厂函数，因为我们只需要认识到编译错误即可)**。现在我们就提到了题设条件，deal函数不可避免地用到原始资源的指针。加入我们执行gcc编译，得到下图结果：

![1641647782525](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1641647782525.png)

错误很明显，指针有问题。我们不能直接这样转换。接下来我们就提供几种可以转换为原始资源的方式。

1.用shared_ptr的get方法，可以返回原始指针，如上图就是Investment*。下图所示，只有一个小warning罢了，编译已经可以通过了。

![1641647833331](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1641647833331.png)

2.智能指针同样重载了"解索引 *"和指向"->"，我们可以通过这两种方式来访问成员函数等。

![1641648135512](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1641648135512.png)







### 16.Use the same form in corresponding uses of new and delete

这个比较简单，如下例：

```c++
string* stringArray = new string[100];
delete stringArray;
```

显然我们应该写的是  "delete []stringArray"，因为上式只删掉了一个string对象，但stringArray包含了100个string对象，所以很有可能有99个string对象没有被适当地调用析构函数。

其实规则很简单就是new用了'[]'的话，delete也得跟上'[]'。

还有一条建议就是尽量少用"typedef"声明数组。如下列代码：

```c++
typedef string stringArray[4];
// 定义了一个类型是stringArray，其中包含了4个元素，元素类型是string。

int main(){
    string* arr = new stringArray;
    // 相当于  string* arr = new string[4];
    delete arr;
    return 0;
}
```

![1641649668393](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1641649668393.png)

**编译并没有报错，但显然这代码是有问题的。**因为我们的arr指向一个 包含4个string的数组，我们显然应该写成"delete []arr"。

因此就有了我们上面的建议，别typedef一个数组，很容易出错而且编译器不告诉你，运行的时候会有bug。我们完全可以用 vector，或者直接string* arr = new string[4]来显示说明。



### 17.Store newd objects in smart pointers in standalone statements

题意为"用单独的语句来存储新建对象"。看起来可能有点怪，我们同样根据一个例子看：

```c++
#include <iostream>
#include <memory>
using namespace std;

class Object{
    public:
        Object(){};
};

int calx(){
    int a = 0;
    return a;
}

void deal(shared_ptr<Object> ptr, int x){
    return;
}

int main(){
    deal(shared_ptr<Object> ptr(new Object), calx());
}
```

我的编译器已经能直接报错，说ptr需要事先声明，但书中表示这种写法也有可以通过编译的版本。但我们要理解的其实就是为什么不能这么写，原因如下：

1.C++执行步骤弹性大，并不是按照我们想象的先执行"new"， 再调用shared_ptr构造函数，最后执行calx函数。

2.如若我们率先new，再执行calx函数，但calx是个问题函数，那么创建出来的这个对象就发生了"内存泄露现象"

所以就按照编译器提示我们的操作，如果拿智能智能装new出来的对象，那么单独写一句进行声明，如下：

```c++
int main(){
    shared_ptr<Object> ptr(new Object);
    deal(ptr, calx());
}
```

编译器就不会再报错。



### 18.Make interfaces easy to use correctly and hard to use incorrectly

对于这么大的一个课题，我们会分几个小细则阐述：

1.明智而审慎地导入新类型对预防"接口被误用"有奇效。比如下例，设计一个日期类：

```c++
#include <iostream>
using namespace std;

class Date{
    public:
        Date(int m, int d, int y):month(m), day(d), year(y){};
    
    private:
        int month, day, year;
};

int main(){
    return 0;
}
```

这是一个按美国人习惯设计的日期类，显然编程角度并不存在任何问题。但作为接口被客户使用的时候，你怎么确保他不会把月份和日期参数反过来抑或是写一个无效日期？

用到了我们条款的思想，用一种新类型，就是单独把年月日设计成一个结构体或类传参，这样对于你搞返参数顺序的行为就会被编译器警报。简单修改后代码如下：

```c++
struct Year{
    int y;
    explicit Year(int _y):y(_y){};
};

class Date{
    public:
        Date(int m, int d, const Year& y):month(m), day(d), year(y){};
    
    private:
        int month, day;
        const Year year;
};
```

2.通过限制值的方式阻止不合理的事情发生，就好比月份产生了大于等于13的数，天数不合理等等。比如对于Month，我们设置一个类，并预先定义所有有效的月份。

```c++
class Month{
    private:
        int month;
    public:
        explicit Month(int m):month(m){};
        static Month Jan(){return Month(1);};
        // etc
};

Month p = Month::Jan();   // 用这种方法表示一月
```

3.限制类型内能做什么事，不能做什么事。

比如**条款"3"**，多用const会避免客户对自定义类型做出一些"不可理喻"的事情。

接口可以做到先发制人而不是要求客户不要忘记去做某事。比如在条款13提到的利用智能指针将工厂函数获取到的原始资源指针进行转换。此时我们的先发制人准则就要求我们，在工厂函数阶段就返回智能指针防止客户忘记。

再又如智能指针删除器设置等，最后我们的工厂函数会被转换为下列形式：

![1641779051034](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1641779051034.png)





### 19. Treat class design as type design

比较繁琐复杂，建议直接参考电子书籍。P114  - P116.



### 20. Prefer pass-by-reference-to-const to pass-by-value

缺省情况下C++都是以pass-by-value的方式传递对象至函数。除非另外指定，否则函数参数或者调用者使用的函数返回值都是都是一份复件，由对象调用拷贝构造函数产出。

书中设计了如下类：

![1641827421859](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1641827421859.png)

然后我们进行如下代码的操作：

```c++
bool validatestudent(Student s);
Student Plato;
bool ans = validatestudent(Plato);
```

仔细思考全过程，大体上Plato会先调用拷贝构造函数初始化s，函数退出后Student s就执行析构函数。

再进入到类内部进行深入分析，每次构造Student对象需要调用 1次Student拷贝函数，Person拷贝函数，四次string的拷贝函数，析构的话也是对应的，只不过应该是反顺序的。这无疑是耗时耗成本的。

但如果按照我们的条款建议将参数改为  "const Student& s"。则不会调用任何构造和析构函数。

设置const是必要的，因为我们只用引用传参的话，传进来的不是我们刚刚所提及的"副本"，是实打实的一个内存中的东西，所以在函数中对其进行修改是很危险的！因此置为const则不会出现弊病。

同时通过这种方式传参可以避免"参数切割"问题。下面略微讲一下此问题，因为概念比较模糊：

```c++
class Base
{
  public:
    virtual void print(void) const { std::cout<<"Base::print()"<<std::endl; }
};

class Derived : public Base
{
  public:
    virtual void print(void) const { std::cout<<"Derived::print()"<<std::endl; }
};

void passByValue(Base obj)
{
  obj.print();
}

void passByReferenceToConst(const Base& obj)
{
  obj.print();
}
```

两个类，虚函数print以方便重载，外部有两个函数分别展示了用pass-by-value和pass-by-const-reference的效果。对于pass-by-value方式的那个函数，我们始终调用的是Base的print函数，但后者则可以当传入衍生类Derived的时候调用衍生类Derived::print。所以意思就是，在pass-by-value方案下，即使传入衍生类，也会去调用Base class的拷贝构造函数，就好像衍生类的特性被切除了一样。运行截图：

![1641829381574](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1641829381574.png)



但这个条款并不是必然，对于一些内置类型，它们可以成为合格的pass-by-value的候选人。还有一类就是STL容器的迭代器和函数对象，千万不要以为你自己设计的class抑或struct只占很小的空间就企图用pass-by-value！



### 21. Don't try to return a reference when you must return an object







### 22.Declare member data private

其实这一点在学校"面向对象编程"的课上老师就会提及，为了继承的话有时候我们也会把member data置于protected属性里，接下来我们来看看究竟是为何要这么做，显然不是为了不让客户直接调用它这么简单。

1.省去客户的麻烦。用户接口全部保留成函数，则不需要判断需不需要括号"()"这一混淆点。

2.对封装有很大的好处。封装确保了你日后改变它的权利，夸张地说public = 不封装 = 不可改变(或者说非常困难，因为实际工程里并不是我们这么简单的class而已)。

protected变量实际上也是缺乏封装性的，因为取消这么一个变量会导致大量的derived class受损。所以我们的结论是protected变量并不比public拥有更好的封装性。

对于封装性的解释，我们看一个小示例：

![1641830828079](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1641830828079.png)

相信图中的内容不难看懂，就是空间换时间的思想。对于追求空间的，那么每次求均值就行了；反之追求时间的，那么每次把均值提前计算好用空间进行保存，调用函数时直接返回即可。

这为我们提供了"所有实现"的可能提供了弹性。



### 23. Prefer non-member non-friend functions to member functions

同样以一个浏览器的系列操作为例：

![1641951855163](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1641951855163.png)

我们构造了一个浏览器类后，里面有很多相关的操作。现在我们想要有一个函数能集合调用这些操作，那么考虑两种办法，一是仍旧声明成员函数进行调用，二就是构造非成员函数将对象以参数形式传入。

相对而言上述的第二种方式能提供更好的"封装性"，因为它不能增加"能够访问class内private成分"的函数数量。封装并非唯一考虑标准，在**条款"24"**我们将会对隐式转换做一些解释。

但我们的"non-member function is better"并不意味着这个函数不能是别的类的成员函数，我们把上图中的clearBrowser函数设为其他工具类的静态成员函数。

C++中一种较为自然的方式就是利用命名空间"namespace"，如下图所示：

![1641954882243](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1641954882243.png)

对于自己定义的命名空间可能不太熟悉，可以查阅 [知乎](https://zhuanlan.zhihu.com/p/126481010)。我们也截取书中对C++架构的一些解释：

![1641954978930](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1641954978930.png)

<u>**因为一个巨大的浏览器处理方法的类，会需要用到很多很多方法，用户并不会对所有方法感兴趣。如果我们把这些方法都声明为"成员函数"，那么编译的时候class就是一个整体全部编译进去，但如果我们以下图形式组织：**</u>

![1641955151975](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1641955151975.png)

那么我们只会将客户用到的函数编译进来会增强机能扩充性，因为class本身对于客户而言是不能扩展的。



### 24. Declare non-member functions when type conversions should apply to all parameters

我们来考虑有理数相乘的例子，我们的有理数类用分数的形式表示，所以类里有分子和分母两个成员变量。同时我们重载operator*操作符以方便相乘，看以下代码：

```c++
#include <iostream>
using namespace std;

class Rational{
    private:
        int dominator;
        int numerator;
    public:
        Rational(int dom = 1, int num = 0):dominator(dom), numerator(num){};
        int dom() const{return dominator;};
        int num() const{return numerator;};
        const Rational operator*(const Rational& rhs){
            Rational ans(dom() * rhs.dom(), num() * rhs.num());
            return ans;
        };
};

int main(){
    Rational oneEighth(8, 1);
    Rational oneHalf(2, 1);
    Rational result = oneEighth * oneHalf;

    return 0;
}
```

上面的代码编译完全没有问题，但我们考虑有理数和一个int相乘的情况。这是合理的，甚至可以说比int转换为double更合理，但如此合理的事编译器会不会有问题仍待观察：

![1642046592790](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1642046592790.png)

好了，我么用 Rational * int可行，但反过来确不可行。前者是因为我们会把int值，也即2，当作参数列表的参数用于构造一个Rational对象。所以对于前者我们其实可以想成编译器做了如下操作：

```c++
const Rational temp(2);
oneEighth.operator*(temp);
```

但后者不可行的原因就是int没有重载和Rational对象相乘的函数，同时全局下也没有Rational operator*(int, const Rational& rhs)这样的函数。**我们可以把 lhs即左手端视为this隐含指针，你想让这个指针主动隐式转换，这不合理，也是不被允许的，所以别指望 2 * Rational的时候2这个this指针隐式变成Rational对象。**



因此，为了解决"一致性"的问题，我们将这种需要隐式类型转换的重载函数作为非成员函数。

```c++
const Rational operator*(const Rational& lhs, const Rational& rhs){
    return Rational(lhs.dom() * rhs.dom(), lhs.num() * rhs.num());
}
```

同时我们无须多此一举将此函数设为友元函数，因为正如代码所示的，完全可以用public接口下的函数实现。



### 25.  Consider support for non-throwing swap*

P142

我们常规情况下std下的模板swap定义如下所示：

```c++
namespace std{
    template<typename T>
    void swap(T& a, T& b){
        T tmp(a);
        a = b;
        b = tmp;
    }
}
```

我们先拷贝构造一个临时对象，然后又会调用赋值函数。确实没有任何毛病，但对于效率而言可能还需要有另外的考虑。

比如我们有如下类：

```c++
class WidgetImpl{
    public:
        //  ... some member functions
    private:
        int a, b ,c;
        vector<double> v;
        // ... some member data
};

class Widget{
    public:
        Widget(const Widget& rhs);
        Widget& operator=(const Widget& rhs){
            // ...
            * pImpl = * (rhs.pImpl);
            // ..
        }
    private:
        WidgetImpl* pImpl;
}
```

置换两个Widget对象实际上我们只要置换指针即可，但缺省的swap算法并不知道。不仅会复制三个Widget对象，还会复制三个WidgetImpl对象，缺乏效率。

对此我们有一个词叫做"特化"，比如如下代码就是一种特化：

```c++
namespace std{
    template<>
    void swap<Widget>(Widget& a, Widget& b){
        swap(a.pImpl, b.pImpl);
    }
}
```

但上面代码有两个问题：1.我们通常不被允许改变std空间内的内容，2.pImpl是私有成员变量因此无法直接访问。

对此我们的策略如下图所示：

![1642144244705](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1642144244705.png)

在Widget声明成员函数swap，然后我们直接在特化版本中调用此成员函数即可。











### 26. Postpone variable definition as long as possible

很容易理解，只要定义了一个变量并且有其专属的构造和析构函数，那显然当程序控制流到达到达定义式后就要承担构造成本，当变量离开作用域后就得承担其析构函数的成本。考虑一个加密函数，如果待加密内容太短则不进行加密。如果函数长这样：

![1642145583644](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1642145583644.png)

不难理解这个问题吧。显然我们如果在判断之后定义encrypted不就可以了吗？

问题再稍微复杂一些，加入我们加密步骤在一个函数中进行：

```c++
void encrypt(std::string& s);
```

我们encryptPassword函数函数中的步骤会被我们改成：

```c++
std::string encryptPassword(const std::string& password){
		// 判断步骤
		std::string encrypt;
		encrypt = password;
		encrypt(encrypt);
}
```

如果再回去看一看**条款4**，你应该采取更好的初始化过程，用一个拷贝构造即可。而上面我们首先调用了default构造，其次用了拷贝赋值。

最后我们采取最高效的拷贝构造的方式即可：

```c++
std::string encryptPassword(const std::string& password){
		// 判断步骤
		std::string encrypt(password);
		encrypt(encrypt);
}
```

所以我们应该尽可能往后推迟变量的定义，甚至知道可以赋给他初值为止。

这里还有一个值得思考的问题是循环体内的变量定义，我们应该在外部定义好不断去里面赋值还是不断在里面重新定义？我们仔细分析一下就知道了：

```c++
Solution A:
Widget s;
for(int i = 0 ; i < n ; i ++){
    // 赋值
}


Solution B:
for(int i = 0 ; i < n ; i ++){
    Widget s(...);
}
```

对于A策略：

1次构造 + 1次析构 + n次赋值

对于B策略:

n次构造 + n次析构

那么我们就需要做一个代价分析，如果构造+析构成本大于赋值，那么显然应该选择策略A。但A的控制域会比较广。如果赋值成本较大，那么显然应该选B策略。 我们一般情况下还是会选B策略。



### 27. Minimize casting

看完条款我才发现我在用的原来是"老式转型"，比如下式:

```c++
int a;
double b = (double)a;

double c = double(a);
```

在Python中我更中意上面代码的第二种转型方式，现在了解C++新型转型方式。

![1642391382832](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1642391382832.png)

新型转型方式的优点有 1.更易被检查出来，2.提供了更窄的转型范围意味着越能诊断出错误这对于你的程序来说是一件好事。但旧式转型并非一无是处，其简便性是显而易见的，比如如下代码：

```c++
#include <iostream>
#include <vector>
using namespace std;

class obj
{
private:
    int size;
public:
    obj(int sz):size(sz){};
};

void doSomething(const obj& t1){
    return;
}

int main(){
    doSomething(15);   // OK
    doSomething(obj(15));   //OK
    doSomething(static_cast<obj>(15));
    return 0;
}
```

代码没有困难的地方，但提供了三种完全不会有警示的通过编译的转型方式。第一种在类简单时问题不大，复杂一点时并不是很好的办法，作者在书中也指出了在构造函数时不妨采用这种老式转型方式。

不要乱用static_cast来进行随意转型，有时候会出现你意想不到的问题，如下例所示：

```c++
class Window{
    public:
        virtual void onResize(){};
};

class SpecialWindow : public Window{
    public:
        virtual void onResize(){
            static_cast<Window>(*this).onResize();
        };
};
```

我们的衍生类SpecialWindow继承了Window，并且想要在onResize函数中调用一下基类的方法。我们采用static_cast方式转为Window类型进行调用，编译能通过，代码没问题，但逻辑问题是存在的。

通过上述方式实际上是"在转型过程中建立了一个'*this'的base成分的副本，并用此副本对象调用onResize"，这样产生的问题是如果在onResize中对对象成员变量有改动的话并不会反应在 *this 对象上，这显然不是我们想要的。

**所以一个较简单的策略就是将问题代码改为"Window::onResize()"。**



要避免连串的转型操作。

![1642471374075](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1642471374075.png)

因为这样生成的代码又长又乱而且基础不稳定，假如何时要修改继承体系都是非常麻烦的，建议用虚函数virtual来实现。



### 28. Avoid Returning handles to object internals

首先解释什么是"handles"句柄，我们认为References、指针和迭代器都是所谓的handles(号码牌，用于获取某个对象)。我们考虑''**条款3**''出现的情况，一个const成员函数返回了私有成员变量的reference，那么实际上此成员函数的实际调用者就可以改动那个私有变量，这就和logicwise的const理念相对立。当然我们作为设计者也是不希望这样的情况发生，这会大大降低封装性。

上述情况对一些被声明为private和protected的成员函数也适用，不要企图去返回函数指针以提供给客户更大的权限，但返回函数指针的行为并不常见。

所以对于上面问题的一个简易的解法就是返回  "const typed&"，这样返回的对象就不会被允许任意修改了。但实际上还是会存在一些其他的问题，考虑如下情况：

![1642472879554](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1642472879554.png)

也就是说boundingBox(*pgo)会返回一个临时对象，然我们真正获得的Point指针也是作用于这个临时对象的某点上的，当这个临时对象被析构后我们获得的指针也沦为"野指针"，所以就有了"dangling handles"一说。

所以我们说将handles进行返回总归是危险的，因此是需要避免的。但这并非说这个规则是一成不变的，比如"operator []"我们仍旧是需要返回reference来指向"容器内的数据"。



### 29. Strive for exception-safe code ***

因为书中采用的例子有Image数据结构，我并不打算写个结构体，直接用截图方式来展示此例：

![1642562227080](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1642562227080.png)

Mutex是互斥量，可以参考自己的**"OS面经-APUE实践"**，因为我们想要把这个类应用于多线程环境下，那么让程序顺序执行是必要的，因此需要锁。否则可能此线程的更改会被另外线程破坏。

但此代码中带有异常安全性的问题有二：

1.不泄露任何资源。但一旦在"new Image"中出错，互斥量就不会被解锁，锁资源就一直被占用。

2.不允许数据败坏。仍旧是刚刚的"new Image"出粗，那么bgImage就会指向一个被删除的资源。而统计的更改次数也已经增加。

**解决问题'1'可以参考条款"14"**，用智能指针来维护一把锁，并在初始化时指定其"删除器"为unlock函数即可。



### 30. Understand the ins and outs of inlining

inline看起来动作像函数，却不受函数调用的花费成本比"宏"好得多，见条款二，是一个非常有效的工具。

但inline函数出现的地方都会被本体替换，所以过多设计inline会造成程序膨胀、体积过大。C++中，基本上类中的成员函数是默认的inline的，但虚函数不然。因为我们刚刚说了，inline出现的地方会被本体替换，但对于虚函数谁又能在运行前说的准调用的是哪一个函数呢，自然也就没法直接进行替换了，因此也不会成为inline了。

inline在C++中大部分是编译期行为。除了上述的虚函数，编译器也拒绝将带有复杂循环、递归的函数变为inline。以上这些是说，是否真正能变成inline函数取决于你的环境和编译器，索性一些编译器会对无法成为inline的函数提出警告。

有时候编译器有意愿内联某个函数但有可能还是会去生成某个函数本体，比如某些情况下你要取某内联函数的地址那么编译器仍旧会去生成一个outlined函数本体。与此类似的是"编译器通常不对通过函数指针进行调用"实施inlining。然而即使你从未使用函数指针，仍旧可能发生inline函数未被成功inlined。

对于程序库设计者而言这是个严峻的课题，因为inline代表着把函数本体编入代码中，那么当你更新inline函数时必须重新编译才行。而对于一个函数进行修改，我们仅需重新链接就好，迅速很多。

而对于程序开发者而言，你**不应该**对**"调试器对inline函数无能为力"**这一点感到奇怪。因为你无法在一个不存在的函数内设置断点。

我们给出的一条较好建议就是**"不要事先将函数设置为inline。我们尝试去遵循'80-20法则'，即80%的程序执行时间是花费在20%的代码上的，我们在程序设计完毕后，竭尽所能找出这20%的代码进行优化才是正道。"**



### 31. Minimiza compilation dependencies between files *** 





### 32. Make sure public inheritance models "is-a"

public继承就代表了"衍生类"就是一种"基类"，即适用于base class的事同样都适用于derived class。因此这样的继承不能仅凭经验主义来断定，接下来的例子会告诉你为什么：

Rectangle类和Square类，正方形是一种矩形，这是天经地义的，是数学告诉我们的正确的定理。那么考虑我们有如下一个类：![1642856273054](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1642856273054.png)主要观察我们的makeBigger函数，传入引用，修改新的宽。现在我们的Square类会使用这个函数，那么假设有如下代码：

```c++
class Square : public Rectangle{};
Square s;

// ...
assert(s.height() == s.width());
makeBiggrt(s);
assert(s.height() == s.width());
```

显然第二句assert断言失败了，因为我们正方形的宽在前面被增加了10，这不就矛盾了吗，长!=宽，又从何而来正方形呢？

鸟和企鹅的继承。![1642856544787](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1642856544787.png)我们在基类中实现了fly虚函数，企鹅继承鸟类后理应也有一个fly，但显然，企鹅不会飞。**第一种解决方案是细化继承的类别：**![1642856649850](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1642856649850.png)这样一来，只有"会飞的鸟"类才拥有飞这一成员函数。第二种解决方案就是为企鹅的fly报错：![1642856716426](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1642856716426.png)但我们在条款18提及过让错误在编译时期产生是一个更佳的选择，于是我们还是尽量让Penguine没有fly这一成员函数较好，就比如方案1就是一种办法。



### 33. Avoid hiding inherited names

在函数中我们常遇到变量作用域的问题，简单的例子如下所示:

```c++
int x;
void someFunc(){
    double x;
    std::cin >> x;
}
```

那么在函数中我们读入的x是赋值给局部作用域里的"双精度浮点数"x的而不是全局作用域(global)的整形。

这个不难理解，我们现在将问题转移到类的继承中来，我们经常会在衍生类(derived class)中对基类函数进行重载，所以我们要考虑的主题就出现了，我们是否在继承时"遮掩了"继承而来的名字呢？

考虑如下类:

```c++
class Base{
  public:
    virtual void func1() = 0; 
    virtual void func2();
    void func3();
    // ....
  private:
    int x;
};

class Derived : public Base{
    public:
      virtual void func1();
      void func4(){
        // ...
        func2();
        // ...
      };
      // .....
};
```

func2的寻找从func4局部域开始寻找，然后去Derived作用域寻找，再去Base作用域寻找(且找到)，如果仍旧没找到就去包含Base的namespace作用域寻找，最后去global作用域寻找。这个类并未产生我们想讨论的"遮盖名称"的问题，我们是探讨了如何寻找的问题，接着我们来看一个出现问题的代码：

```c++
class Base{
  public:
    virtual void func1() = 0; 
    virtual void func1(int);
    virtual void func2();
    void func3();
    void func3(double);
    // ....
  private:
    int x;
};

class Derived : public Base{
    public:
      virtual void func1();
      void func3();
      void func4();
      // .....
};


Derived d;
int x;
// ...
d.func1();
d.func1(x);    // 错误
d.func2();
d.func3();
d.func3(x);    // 错误
```

我们如上代码有两处错误，即带参的两个成员函数并没有按照预期进行，仿佛我们没去继承一样。实则不然，此时即发生了我们一开始讨论的"遮盖名称"的问题，我们Derived类作用域内的名称"func3、func1"已经遮盖了基类中带参的两个函数，因此报错。

我们一种修改方式就是利用"using"关键字:

```c++
class Derived : public Base{
    public:
      using Base::func1;
      using Base::func3;
      virtual void func1();
      void func3();
      void func4();
      // .....
};
```

同时，比如你在使用private继承时可能并不想继承基类的所有成员函数**【这种现象不能在public发生，我们在条款32中声明了public继承应该是一种"is-a"关系，不存在不想要就不继承】**，此时我们用到的策略是"**转交函数**"：

```c++
class Derived : private Base{
    public:
      virtual void func1(){
        Base::func1();  
      };
      void func3();
      void func4();
      // .....
};

Derived t;
int x;
// ....
t.func1(x);    // 会报错
```





### 34. Differentiate between inheritance of interface and inheritance of implementation

在严密检查public继承后，我们可以发现继承分为两部分即"函数接口"继承和"函数实现"继承。

**对于 pure virtual函数而言，所有derived class都需要具象化这个函数，所以纯虚函数只是函数接口的继承。**

对于impure virtual而言就有所不同了，诚然，derived classes会继承这个函数接口，但除此之外还可能包含override(重写)。我们当然也可以不重写，那么会直接调用base class的vitual函数。

但是，同时允许impure virtual函数指定声明函数和函数缺省行为却是有危险的！如下例所示：

```c++
#include <iostream>
using namespace std;

class Airport{};

class Airplane{
    public:
        virtual void fly(const Airport& destination);
}

class ModelA : public Airplane{};
class ModelB : public Airplane{};

int main(){
    return 0;
}
```

机场类，飞机类且有一个共有的fly函数，被A类和B类飞机所共享。但假设现在新购了一种C类飞机，并且其飞行行为可能略有不同但却忘记了重写fly：

```c++

class ModelC : public Airplane{};

Airport PDX;
ModelC* x = new ModelC;
x->fly(PDX);
```

这样的话C类飞机就会去调用缺省飞行行为，会得到意想不到的结果。一种改动方法就是借鉴我们上面的pure virtual function，将fly函数声明为纯虚函数，即derived class只是继承接口必须得自己实现才行，又为了避免A类和B类飞机代码重复，在base class 的protected域增加一个默认飞行行为，如下图所示：

![1643289974968](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1643289974968.png)



还有一种情况，一些人反对用不同的函数分别提供接口和缺省行为。即，我们最好别又多个defaultFly出来。然而，对于接口和实现的继承应该分开这一点他们又是赞同的。对于这种情况我们采用的方法是：pure virtual函数必须在derived classes中重新实现但其也可以有自己的实现，如下代码所示：

```c++
class Airport{};

class Airplane{
    public:
        virtual void fly(const Airport& destination) = 0;
};

void Airplane::fly(const Airport& destination){
    return;
}

class ModelA : public Airplane{
    public:
        virtual void fly(const Airport& destination){
            Airplane::fly(destination);
        }
};
```

如此一来，fly便被分割成了两个要素，纯虚函数表现出了"接口继承"的属性，即derived classes必须加以重新声明。而定义部分则体现了"实现继承"属性，可以提供缺省行为供衍生类调用。

对于基类中的non-virtual 函数，我们认为这是一种"不变性"凌驾于"特异性"的表现，well，你进行override没人算你错，but why not declare it with virtual ？所以，原则上来讲，它绝不应该在derived classes中被重新定义。



### 35. Consider alternatives to virtual functions

诚然，虚函数(包括纯虚函数)是一种很好的机制，但正如我们标题所说，有时候要变通得去考虑一下其替代品。

假设我们设计一个游戏，有人物类，包含了一个成员函数是用于返回健康状况的：

```c++
class GameCharacter{
  public:
    virtual int HealthConditions() const;
};
```



流派一提出的思想是虚函数都应该声明为private，**因此其方法是"non-virtual interface(NVI)"**。也就是提供给用户的public接口通过调用private域的虚函数进行实现。其中我们还可以在调用前完成事前事后工作。

我们把non-virtual函数称为virtual函数的外覆器(wrapper)。修改后的代码如下图所示：

![1643335616547](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1643335616547.png)



NVI手法对于类似析构函数(见条款7)必须为public这样的情况并不适用，而对于那些在Derived classes中想要调用Base class的virtual函数行为则需要把虚函数放在protected域内！



**藉由Function Pointers 实现 Strategy模式。**

```c++
class GameCharacter;
int defaultHealthCal(const GameCharacter& gc);
class GameCharacter{
    public:
        typedef int (*HealthCalcFunc)(const GameCharacter& gc);
        explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCal):
            healthFunc(hcf){};
        int healthValue() const{
            return healthFunc(*this);
        };
    private:
        HealthCalcFunc healthFunc;
};
```

我们在类外部定义一个需要传入游戏角色对象的函数用于计算健康状况。类内部定义了函数指针，并且将函数指针类型的healthFunc做成了私有成员变量，构造函数就是为这个函数指针赋值，然后在healthValue进行调用。

用这种方式我们可以在类外部定义计算不同人物健康状况的函数，构造时传入不同的函数指针即可。

此策略的缺点在于如果这个non-member函数需要访问对象中的non-public成分时需要弱化类的封装，将non-member声明为friend抑或提供public接口进行对那个成分的访问。



##### **藉由tr1::function完成Strategy策略。****   ---------------待学习

库包含在<tr1/functional>中。

function对象可持有任何可调用物(callable entity)。包括了函数指针、函数对象或成员函数指针。

```c++
class GameCharacter2;
int defaultHealthCal2(const GameCharacter2& gc);
class GameCharacter2{
    public:
        typedef function<int (const GameCharacter2&)> HealthCalcFunc;
        explicit GameCharacter2(HealthCalcFunc hcf = defaultHealthCal2):
            healthFunc(hcf){};
        int healthValue() const{
            return healthFunc(*this);
        };
    private:
        HealthCalcFunc healthFunc;
};
```

含义解析和优势如下图所示：

![1643375924828](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1643375924828.png)





**古典的strategy模式。**

![1643377257622](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1643377257622.png)

左边就是一个以GameCharacter为基类的有许多角色衍生类的继承体系。右边则是则是以HealthCalcFunc为基类的继承体系，每一个GameCharacter对象都会包含一个来自HealthCalcFunc对象的指针。

结构模式代码如下：

```c++
class GameCharacter3;
class HealthCalcFunc{
    public:
        // ....
        virtual int calc(const GameCharacter3& gc) const;
        // ...
};

HealthCalcFunc defaultHealthCalc3;

class GameCharacter3{
    public:
        explicit GameCharacter3(HealthCalcFunc* phcf = &defaultHealthCalc3):
            pHealthCalc(phcf){};

        int healthValue() const{
            return pHealthCalc->calc(*this);
        }
    private:
        HealthCalcFunc* pHealthCalc;
};
```



### 36. Never redefine an inherited non-virtual function

这个问题不用多加讨论，我们在讨论"虚函数"机制和"public"继承以及其他一些条款时已经加以阐述过好多次。如果你想要重载，那么请用vitual或者上一个条款提到的virtual的替代品。

从public继承是"is-a"角度来看，non-virtual函数的设计是一种不变性凌驾特异性的表现，说明基类实现的衍生类一定都是照搬的，我们不能去进行修改。

总而言之，不要去修改继承来的non-virtual函数，就是这么简单。



### 37. Never redefine a function‘s inherited default parameter value

由于上一个条款的存在，我们安全的将此条款叙述为"不要试图去修改继承而来的virtual函数的默认参数值"。考虑我们写了这么一个糟糕的违反此条例的代码：

```c++
#include <iostream>
using namespace std;

class Shape{
    public:
        enum Shapecolor{Red, Green, Blue};
        virtual void draw(Shapecolor color = Red) const = 0;
};

class Rectangle : public Shape{
    public:
        virtual void draw(Shapecolor color = Green) const;
};

class Circle : public Shape{
    public:
        virtual void draw(Shapecolor color) const;
};

int main(){
    Shape* ps;
    Shape* pc = new Circle;
    Shape* pr = new Rectangle;
    
    pr->draw();
    return 0;
}
```

我们的pr会正常执行Rectangle的draw函数，因为virtual函数是动态绑定的。但问题就出现在参数上，颜色参数实际上是静态绑定的行为，虽然我们在Rectangle里修改为了绿色，但真实颜色仍旧是基类声明的红色。不信？看下面的略加修改的代码：

```c++
class Rectangle : public Shape{
    public:
        virtual void draw(Shapecolor color = Green) const{
            string ans = "None";
            switch (color)
            {
            case 0:
                ans = "Red";
                break;

            case 1:
                ans = "Green";
                break;
            
            case 2:
                ans = "Blue";
                break;
            
            default:
                break;
            }
            cout << "real color is " << ans << endl;
        };
};
```

看下图：我想现在大家就知道了，真的很神奇，我也不敢相信！！

![1643707024731](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1643707024731.png)

这种奇怪的编译方式是由运行期效率决定的，动态绑定缺省参数不是一件好事，很耗时。

**对以上问题的解决方式仍旧是条款35提及的NVI策略。**

![1643764362265](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1643764362265.png)

用这样的方式清楚地使得缺省的颜色参数总是为Red。



### 38. Model "has-a" or "is-implemented-in-terms-of" through composition

composition就是复合的意思，当我们某对象时其中包含了其他对象我们就称其为"复合"形式。很简单的C++初学时会让你设计人这么一个类，里面包含了比如身份证、电话、住址等等就是复合的一种体现。

对于标题里的两个属于的解释如下图所示：

![1643764786625](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1643764786625.png)





has-a关系很好解释，还是我们刚刚的例子，人这个类，**有**电话、住址等信息。我们也不会和is-a关系搞混。因为我们没人会说**人是一个电话、一个住址等**。

实现域则稍微复杂些。思考我们想要造个轮子，实现自己的Set类，因为我们的需求或者老师强迫要求std::set或许并不能满足你的需求。有一种思路就是底层依靠linked list链表进行实现，这个轮子我们或许不用自己造，std::list是一个能帮到你的工具。

所以我们直接写出如下代码：

```c++
template<typename T>
class Set : public List<T>
```

不对！因为public继承表达了一种is-a关系，对List成立的必然对Set成立。链表可以插入重复元素，Set不行，这是由集合的数学概念决定的。

可行的做法就是把List<T>放入到Set类当中，用作一种实现手段，正是我们标题里的"is-implemented-in-terms-of"。这样一来你的设计思想和实现方式不会出错而且还能在成员函数中大量利用到List的functions。下例：

![1643765699916](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1643765699916.png)



### 39. Use private inheritance judiciously

假如我们设计了一个人的类，并被一个学生类以public方式继承，类外有eat函数、study函数，参数分别是const Person& 和 const Student&，这样即使我们在调用eat时传参const Student&编译器也会帮我们做隐式转换。

但我们更换为"private"继承时，一切就变了，但显然Student不能吃是荒谬的。

private继承实际上用上一条款的术语说就是"is-implemented-in-terms-of"（一种实现模式），因为根据private继承其Base里的所以内容都会变为private，所有东西都已经对Derived不可见了！

**用条款34说的话就是只继承了实现方式却没有继承接口**。

而这和条款38给出的复合定义也是一致的，但我们要求尽量使用composition而不是private继承。

书中给出了一个案例是，我们想要为某Widget设计一个可以记录每个成员函数被调用的工具函数，于是我们设计了一个Timer类，内含virtual void onTick成员函数，时钟每滴答一次此函数就被调用。至于如何将Timer和Widget结合起来。

1.假设是继承必须是private的，因为Widget不是一种Timer，因此无法使用public继承。一经private继承，则原先的onTick函数则在Widget中变为private属性。

2.可以用composition模式，如下图所示：

![1644042042381](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1644042042381.png)



我么来谈谈设计1的缺点：

显然我们的Widget不应该能进行onTick的操作，但即使是private继承也无法阻止你在derived classes中对virtual函数进行重新定义。**（当然，可以利用上面不知道哪条条款题到过的关键字final）**

设计2的优点就是可以降低文件编译依存性，正如条款31提到的。因为按照设计1，我们可能会需要引入和Timer相关的头文件。

那究竟什么情况适用private继承呢？情况如下图所述：

![1644042439616](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1644042439616.png)

```c++
class Empty{};
class HoldsAnInt{
private:
    int x;
    Empty e;
};
```

其实我在**C++基础知识.md**中提到过，空类也会有1Byte的大小。而齐位规则还会为类HoldsAnInt做一些padding，这里的话会补充为sizeod(int)整数倍，所以会浪费很多空间。

所以我们利用"EBO"法则(空白基类最优化)，即去private继承一个空基类，而不是去含有一个对象。值得注意一点就是此法则通常只适用于单一继承。



### 40. Use multiple inheritance judiciously

多重继承容易产生歧义，比如我们想要设计一个从图书馆借电子物品的系统，代码如下：

```c++
#include <iostream>
using namespace std;

class BorrowableItem{
    public:
        void checkout();    // 离开时进行检查
};

class ElectronicGadget{
    public:
        bool checkout();
};

class MP3Player : public BorrowableItem , public ElectronicGadget{
    public:

};

int main(){
    MP3Player mp;
    mp.checkout();
    return 0;
}
```

如果你执行"g++ -o xxx xxx.cpp"会报错，编译期间就报错了，会提示ambiguous，就是checkout指代不明。即使你讲某个类的checkout放入private域，问题仍旧存在。

解决方法就是明确指出你要用哪个基类的checkout方法，例如下：

```c++
mp.BorrowableItem::checkout();
```

**（不要用我的代码直接g++，因为我的checkout都为定义，会产生链接错误）。**



多重继承还会常遇见的一个问题就是"钻石型继承"问题，就是Derived class到Base Class的路径有两条甚至更多。典型的图如下所示，相信在学oop的时候也一定上机写过这样的题：

![1644111705610](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1644111705610.png)

想象File有成员变量filename，那么IOFile有几个filename？哦，理论上来就，一个文件名才是符合逻辑的。但你的继承方式实际上导致了C++更加偏向于为IOFile的每个Base Class(即InputFile & OutputFile各创建一份filename)。

对上述问题的解决方案就是：执行虚继承。新的结构图如下所示：

![1644111918731](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1644111918731.png)

简单的虚继承使用方式和结果展示可以参考此链接"[虚继承](http://c.biancheng.net/view/2280.html)"。但虚继承会大大增加访问时间，所产生的对象也会体积更大(因为编译器要在幕后动一点手脚才会让使用者更加舒服地使用)。

多重继承的一个很经典的用途在于：用public继承某个interface class，用private继承某个协助实现的class，两者相结合。

**将书上的案例简述。**

我们设计一个纯虚类IPerson，提供了人所必需的一些接口，对于纯虚类我们无法生成对象进行操作，因此要用指针或者引用来进行操作。如下图所示：

![1644113516175](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1644113516175.png)

makePerson就是一个工厂函数，可以返回一个指向IPerson的智能指针。需要传递的参数是一个数据库ID，数据库也被我们编成一个类，可以用数据库ID初始化。

然后我们设计一个class与数据库相关，可以提供人这个类所需要的实质信息，设计如下：

![1644113706508](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1644113706508.png)

public域的成员函数就是提供人名和生日，private域用于进行格式化输出，在头部尾部分别加一个字符。不同的类，不同的实现者可能对格式化的要求不同，所以此接口也应是virtual的可修改的。

**CPerson和PersonInfo的关系也是"is-implemented-in-terms-of"，那么选复合or private 继承？上一条款说能选复合尽量选复合，但对于要重新定义PersonInfo里virtual函数(格式化输出)这种行为还是要采用private继承。**

所以，最后的CPerson类设计如下：

![1644114013382](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1644114013382.png)