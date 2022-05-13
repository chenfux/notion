# C++11标准之右值引用(ravalue reference)

Created: April 6, 2022 11:33 AM
Last Edited Time: April 6, 2022 11:40 AM

[C++11标准之右值引用（ravalue reference）](https://www.cnblogs.com/opangle/archive/2012/11/19/2777131.html)

**1、右值引用引入的背景**

临时对象的产生和拷贝所带来的效率折损，一直是C++所为人诟病的问题。但是C++标准允许编译器对于临时对象的产生具有完全的自由度，从而发展出了Copy Elision、RVO（包括NRVO）等编译器优化技术，它们可以防止某些情况下临时对象产生和拷贝。下面简单地介绍一下Copy Elision、RVO，对此不感兴趣的可以直接跳过：

（1） Copy Elision

Copy Elision技术是为了防止某些不必要的临时对象产生和拷贝，例如：

```cpp
struct A {
    A(int) {}
    A(const A &) {}
};
A a = 42;
```

理论上讲，上述A a = 42;语句将分三步操作：第一步由42构造一个A类型的临时对象，第二步以临时对象为参数拷贝构造a，第三步析构临时对象。如果A是一个很大的类，那么它的临时对象的构造和析构将造成很大的内存开销。我们只需要一个对象a，为什么不直接以42为参数直接构造a呢？Copy Elision技术正是做了这一优化。

*【说明】：你可以在A的拷贝构造函数中加一打印语句，看有没有调用，如果没有被调用，那么恭喜你，你的编译器支持Copy Elision。但是需要说明的是：A的拷贝构造函数虽然没有被调用，但是它的实现不能没有访问权限，不信你将它放在private权限里试试，编译器肯定会报错。*

（2） 返回值优化（RVO，Return Value Optimization）

返回值优化技术也是为了防止某些不必要的临时对象产生和拷贝，例如：

```cpp
struct A {
    A(int) {}
    A(const A &) {}
};
A get() {return A(1);}
A a = get();
```

理论上讲，上述A a = get();语句将分别执行：首先get()函数中创建临时对象（假设为tmp1）,然后以tmp1为参数拷贝构造返回值（假设为tmp2），最后再以tmp2为参数拷贝构造a，其中还伴随着tmp1和tmp2的析构。如果A是一个很大的类，那么它的临时对象的构造和析构将造成很大的内存开销。返回值优化技术正是用来解决此问题的，它可以避免tmp1和tmp2两个临时对象的产生和拷贝。

*【说明】： a）你可以在A的拷贝构造函数中加一打印语句，看有没有调用，如果没有被调用，那么恭喜你，你的编译器支持返回值优化。但是需要说明的是：A的拷贝构造函数虽然没有被调用，但是它的实现不能没有访问权限，不信你将它放在private权限里试试，编译器肯定会报错。*

*b）除了返回值优化，你可能还听说过一个叫具名返回值优化（Named Return Value Optimization,NRVO）的优化技术，从程序员的角度而言，它其实跟RVO同样的逻辑。只是它的临时对象具有变量名标识，例如修改上述get()函数为：*

```cpp
A get() {
    A tmp(1); // #1
    // do something
    return tmp;
}
A a = get(); // #2
```

*想想上述修改后A类型共有几次对象构造？虽然#1处看起来有一次显示地构造，#2处看起来也有一次显示地构造，但如果你的编译器支持NRVO和Copy Elision，你会发现整个A a = get();语句的执行过程，只有一次A对象的构造。如果你在get()函数return语句前打印tmp变量的地址，在A a = get();语句后打印a的地址，你会发现两者地址相同，这就是应用了NRVO技术的结果。*

（3） Copy Elision、RVO无法避免的临时对象的产生和拷贝

虽然Copy Elision和NVO（包括NRVO）等技术能避免一些临时对象的产生和拷贝，但某些情况下它们却发挥不了作用，例如：

```cpp
template <typename T>
void swap(T& a, T& b) {
    T tmp(a);
    a = b;
    b = tmp;
}
```

我们只是想交换a和b两个对象所拥有的数据，但却不得不使用一个临时对象tmp备份其中一个对象，如果T类型对象拥有指向（或引用）从堆内存分配的数据，那么深拷贝所带来的内存开销是可以想象的。为此，C++11标准引入了右值引用，使用它可以使临时对象的拷贝具有move语意，从而可以使临时对象的拷贝具有浅拷贝般的效率，这样便可以从一定程度上解决临时对象的深度拷贝所带来的效率折损。

**2、C++03标准中的左值与右值**

要理解右值引用，首先得区分左值（lvalue）和右值（rvalue）。

C++03标准中将表达式分为左值和右值，并且“非左即右”：

*Every expression is either an lvalue or an rvalue.*

区分一个表达式是左值还是右值，最简便的方法就是看能不能够对它取地址：如果能，就是左值；否则，就是右值。

*【说明】：由于右值引用的引入，C++11标准中对表达式的分类不再是“非左即右”那么简单，不过为了简单地理解，我们暂时只需区分左值右值即可，C++11标准中的分类后面会有描述。*

**3、右值引用的绑定规则**

右值引用（rvalue reference，&&）跟传统意义上的引用（reference，&）很相似，为了更好地区分它们俩，传统意义上的引用又被称为左值引用（lvalue reference）。下面简单地总结了左值引用和右值引用的绑定规则（函数类型对象会有所例外）：

（1）非const左值引用只能绑定到非const左值；（2）const左值引用可绑定到const左值、非const左值、const右值、非const右值；（3）非const右值引用只能绑定到非const右值；（4）const右值引用可绑定到const右值和非const右值。

测试例子如下：

```
struct A { A(){} };
A lvalue;                             // 非const左值对象
const A const_lvalue;                 // const左值对象
A rvalue() {return A();}              // 返回一个非const右值对象
const A const_rvalue() {return A();}  // 返回一个const右值对象

// 规则一：非const左值引用只能绑定到非const左值
A &lvalue_reference1 = lvalue;         // ok
A &lvalue_reference2 = const_lvalue;   // error
A &lvalue_reference3 = rvalue();       // error
A &lvalue_reference4 = const_rvalue(); // error

// 规则二：const左值引用可绑定到const左值、非const左值、const右值、非const右值
const A &const_lvalue_reference1 = lvalue;         // ok
const A &const_lvalue_reference2 = const_lvalue;   // ok
const A &const_lvalue_reference3 = rvalue();       // ok
const A &const_lvalue_reference4 = const_rvalue(); // ok

// 规则三：非const右值引用只能绑定到非const右值
A &&rvalue_reference1 = lvalue;         // error
A &&rvalue_reference2 = const_lvalue;   // error
A &&rvalue_reference3 = rvalue();       // ok
A &&rvalue_reference4 = const_rvalue(); // error

// 规则四：const右值引用可绑定到const右值和非const右值，不能绑定到左值
const A &&const_rvalue_reference1 = lvalue;         // error
const A &&const_rvalue_reference2 = const_lvalue;   // error
const A &&const_rvalue_reference3 = rvalue();       // ok
const A &&const_rvalue_reference4 = const_rvalue(); // ok

// 规则五：函数类型例外
void fun() {}
typedef decltype(fun) FUN;  // typedef void FUN();
FUN       &  lvalue_reference_to_fun       = fun; // ok
const FUN &  const_lvalue_reference_to_fun = fun; // ok
FUN       && rvalue_reference_to_fun       = fun; // ok
const FUN && const_rvalue_reference_to_fun = fun; // ok
```

*【说明】：（1） 一些支持右值引用但版本较低的编译器可能会允许右值引用绑定到左值，例如g++4.4.4就允许，但g++4.6.3就不允许了，clang++3.2也不允许，据说VS2010 beta版允许，正式版就不允许了，本人无VS2010环境，没测试过。*

*（2）右值引用绑定到字面值常量同样符合上述规则，例如：int &&rr = 123;，这里的字面值123虽然被称为常量，可它的类型为int，而不是const int。对此C++03标准文档4.4.1节及其脚注中有如下说明：*

*If T is a non-class type, the type of the rvalue is the cv-unqualified version of T.    In C++ class rvalues can have cv-qualified types (because they are objects). This differs from ISO C, in which non-lvalues never have cv-qualified types.*

*因此123是非const右值，int &&rr = 123;语句符合上述规则三。*

**4、C++11标准中的表达式分类**

右值引用的引入，使得C++11标准中对表达式的分类不再是非左值即右值那么简单，下图为C++11标准中对表达式的分类：

![https://pic002.cnblogs.com/images/2012/335314/2012111913234588.jpg](https://pic002.cnblogs.com/images/2012/335314/2012111913234588.jpg)

简单解释如下：

（1）lvalue仍然是传统意义上的左值；

（2）xvalue（eXpiring value）字面意思可理解为生命周期即将结束的值，它是某些涉及到右值引用的表达式的值（An xvalue is the result of certain kinds of expressions involving rvalue references），例如：调用一个返回类型为右值引用的函数的返回值就是xvalue。

（3）prvalue（pure rvalue）字面意思可理解为纯右值，也可认为是传统意义上的右值，例如临时对象和字面值等。

（4）glvalue（generalized value）广义的左值，包括传统的左值和xvalue。

（5）rvalue除了传统意义上的右值，还包括xvalue。

上述lvalue和prvalue分别跟传统意义上的左值和右值概念一致，比较明确，而将xvalue描述为『某些涉及到右值引用的表达式的值』，某些是哪些呢？C++11标准给出了四种明确为xvalue的情况：

```
[ Note: An expression is an xvalue if it is:
  -- the result of calling a function, whether implicitly or explicitly, whose return type is an rvalue reference to object type,
  -- a cast to an rvalue reference to object type,
  -- a class member access expression designating a non-static data member of non-reference type in which the object expression is an xvalue, or
  -- a .* pointer-to-member expression in which the first operand is an xvalue and the second operand is a pointer to data member.
  In general, the effect of this rule is that named rvalue references are treated as lvalues and unnamed rvalue references to objects are treated as xvalues; rvalue references to functions are treated as lvalues whether named or not. --end note ]
[ Example:
    struct A {
        int m;
    };
    A&& operator+(A, A);
    A&& f();
    A a;
    A&& ar = static_cast<A&&>(a);
  The expressions f(), f().m, static_cast<A&&>(a), and a + a are xvalues. The expression ar is an lvalue.
--end example ]
```

简单地理解就是：具名的右值引用（named rvalue reference）属于左值，不具名的右值引用（unamed rvalue reference）就属于xvalue，而引用函数类型的右值引用不论是否具名都当做左值处理。看个例子更容易理解：

```cpp
A rvalue(){ return A(); }
A &&rvalue_reference() { return A(); }
fun();              // 返回的是不具名的右值引用，属于xvalue
A &&ra1 = rvalue(); // ra1是具名右值应用，属于左值
A &&ra2 = ra1;      // error，ra1被当做左值对待，因此ra2不能绑定到ra1（不符合规则三）
A &la = ra1;        // ok，非const左值引用可绑定到非const左值（符合规则一）
```

**5、move语意**

现在，我们重新顾到1-(3)，其中提到move语意，那么怎样才能使临时对象的拷贝具有move语意呢？下面我们以一个类的实现为例：

```
class A {
public:
    A(const char *pstr = 0) { m_data = (pstr != 0 ? strcpy(new char[strlen(pstr) + 1], pstr) : 0); }

    // copy constructor
    A(const A &a) { m_data = (a.m_data != 0 ? strcpy(new char[strlen(a.m_data) + 1], a.m_data) : 0); }

    // copy assigment
    A &operator =(const A &a) {
        if (this != &a) {
            delete [] m_data;
            m_data = (a.m_data != 0 ? strcpy(new char[strlen(a.m_data) + 1], a.m_data) : 0);
        }
        return *this;
    }

    // move constructor
    A(A &&a) : m_data(a.m_data) { a.m_data = 0; }

    // move assigment
    A & operator = (A &&a) {
        if (this != &a) {
            m_data = a.m_data;
            a.m_data = 0;
        }
        return *this;
    }

    ~A() { delete [] m_data; }

private:
    char * m_data;
};
```

从上例可以看到，除了传统的拷贝构造(copy constructor)和拷贝赋值(copy assigment)，我们还为A类的实现添加了移动拷贝构造(move constructor)和移动赋值(move assigment)。这样，当我们拷贝一个A类的（右值）临时对象时，就会使用具有move语意的移动拷贝构造函数，从而避免深拷贝中strcpy()函数的调用；当我们将一个A类的（右值）临时对象赋值给另一个对象时，就会使用具有move语意的移动赋值，从而避免拷贝赋值中strcpy()函数的调用。这就是所谓的move语意。

**6、std::move()函数的实现**

了解了move语意，那么再来看1-(3)中的效率问题：

```cpp
template <typename T> // 如果T是class A
void swap(T& a, T& b) {
    T tmp(a);  // 根据右值引用的绑定规则三可知，这里不会调用move constructor，而会调用copy constructor
    a = b;     // 根据右值引用的绑定规则三可知，这里不会调用move assigment，而会调用copy assigment
    b = tmp;   // 根据右值引用的绑定规则三可知，这里不会调用move assigment，而会调用copy assigment
}
```

从上例可以看到，虽然我们实现了move constructor和move assigment，但是swap()函数的例子中仍然使用的是传统的copy constructor和copy assigment。要让它们真正地使用move语意的拷贝和复制，就该std::move()函数登场了，看下面的例子：

```cpp
void swap(A &a, A &b) {
    A tmp(std::move(a)); // std::move(a)为右值，这里会调用move constructor
    a = std::move(b);    // std::move(b)为右值，这里会调用move assigment
    b = std::move(tmp);  // std::move(tmp)为右值，这里会调用move assigment
}
```

我们不禁要问：我们通过右值应用的绑定规则三和规则四，知道右值引用不能绑定到左值，可是std::move()函数是如何把上述的左值a、 b和tmp变成右值的呢？这就要从std::move()函数的实现说起，其实std::move()函数的实现非常地简单，下面以libcxx库中的实现（在<type_trait>头文件中）为例：

```cpp
template <class _Tp>
inline typename remove_reference<_Tp>::type&& move(_Tp&& __t) {
    typedef typename remove_reference<_Tp>::type _Up;
    return static_cast<_Up&&>(__t);
}
```

其中remove_reference的实现如下：

```cpp
template <class _Tp> struct remove_reference        {typedef _Tp type;};
template <class _Tp> struct remove_reference<_Tp&>  {typedef _Tp type;};
template <class _Tp> struct remove_reference<_Tp&&> {typedef _Tp type;};
```

从move()函数的实现可以看到，move()函数的形参（Parameter）类型为右值引用，它怎么能绑定到作为实参（Argument）的左值a、b和tmp呢？这不是仍然不符合右值应用的绑定规则三嘛！简单地说，如果move只是个普通的函数（而不是模板函数），那么根据右值应用的绑定规则三和规则四可知，它的确不能使用左值作为其实参。但它是个模板函数，牵涉到模板参数推导，就有所不同了。C++11标准文档14.8.2.1节中，关于模板函数参数的推导描述如下：

Template argument deduction is done by comparing each function template parameter type (call it P) with the type of the corresponding argument of the call (call it A) as described below. (14.8.2.1.1)    If P is a reference type, the type referred to by P is used for type deduction. If P is an rvalue reference to a cvunqualified template parameter and the argument is an lvalue, the type "lvalue reference to A" is used in place of A for type deduction. (14.8.2.1.3)

大致意思是：模板参数的推导其实就是形参和实参的比较和匹配，如果形参是一个引用类型（如P&），那么就使用P来做类型推导；如果形参是一个cv-unqualified（没有const和volatile修饰的）右值引用类型（如P&&），并且实参是一个左值（如类型A的对象），就是用A&来做类型推导（使用A&代替A）。

```cpp
template <class _Tp> void f(_Tp &&) { /* do something */ }
template <class _Tp> void g(const _Tp &&) { /* do something */ }
int x = 123;
f(x);   // ok，f()模板函数形参为非const非volatile右值引用类型，实参x为int类型左值，使用int&来做参数推导，因此调用f<int &>(int &)
f(456); // ok，实参为右值，调用f<int>(int &&)
g(x);   // error，g()函数模板参数为const右值引用类型，会调用g<int>(const int &&)，通过右值引用规则四可知道，const右值引用不能绑定到左值，因此会导致编译错误
```

了解了模板函数参数的推导过程，已经不难理解std::move()函数的实现了，当使用左值（假设其类型为T）作为参数调用std::move()函数时，实际实例化并调用的是std::move<T&>(T&)，而其返回类型T&&，这就是move()函数左值变右值的过程（其实左值本身仍是左值，只是被当做右值对待而已，被人“抄了家”，变得一无所有）。

*【说明】： move()函数改名为rval()可能会更好些，但是move()这个名字已经被使用了好些年了（C++FAQ: Maybe it would have been better if move() had been called rval(), but by now move() has been used for years.）。*

**7、完整的示例**

至此，我们已经了解了不少右值引用的知识点了，下面给出了一个完整地利用右值引用实现move语意的例子：

```
#include <iostream>
#include <cstring>

#define PRINT(msg) do { std::cout << msg << std::endl; } while(0)

template <class _Tp> struct remove_reference        {typedef _Tp type;};
template <class _Tp> struct remove_reference<_Tp&>  {typedef _Tp type;};
template <class _Tp> struct remove_reference<_Tp&&> {typedef _Tp type;};

template <class _Tp>
inline typename remove_reference<_Tp>::type&& move(_Tp&& __t) {
    typedef typename remove_reference<_Tp>::type _Up;
    return static_cast<_Up&&>(__t);
}

class A {
public:
    A(const char *pstr) {
        PRINT("constructor");
        m_data = (pstr != 0 ? strcpy(new char[strlen(pstr) + 1], pstr) : 0);
    }
    A(const A &a) {
        PRINT("copy constructor");
        m_data = (a.m_data != 0 ? strcpy(new char[strlen(a.m_data) + 1], a.m_data) : 0);
    }
    A &operator =(const A &a) {
        PRINT("copy assigment");
        if (this != &a) {
            delete [] m_data;
            m_data = (a.m_data != 0 ? strcpy(new char[strlen(a.m_data) + 1], a.m_data) : 0);
        }
        return *this;
    }
    A(A &&a) : m_data(a.m_data) {
        PRINT("move constructor");
        a.m_data = 0;
    }
    A & operator = (A &&a) {
        PRINT("move assigment");
        if (this != &a) {
            m_data = a.m_data;
            a.m_data = 0;
        }
return *this;
    }
    ~A() { PRINT("destructor"); delete [] m_data; }
private:
    char * m_data;
};

void swap(A &a, A &b) {
    A tmp(move(a));
    a = move(b);
    b = move(tmp);
}

int main(int argc, char **argv, char **env) {
    A a("123"), b("456");
    swap(a, b);
    return 0;
}
```

输出结果为：

```
constructor
constructor
move constructor
move assigment
move assigment
destructor
destructor
destructor
```

**8、花絮**

C++11标准引入右值引用的提案是由Howard Hinnant提出的，它的最初提案[N1377](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2002/n1377.htm)在02年就提出来了，中间经历了多次修改N1385、N1690、N1770、N1855、N1952、[N2118](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2118.html)。包括最终版本[N2118](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2118.html)在内，Howard Hinnant的提案中都使用了右值引用直接绑定到左值的例子，并且由Howard Hinnant、Bjarne Stroustrup和Bronek Kozicki三人08年10月共同署名的《A Brief Introduction to Rvalue References》文章中也有右值引用直接绑定到左值的例子，但奇怪的是C++11标准文档中却不允许右值引用直接绑定到左值，其中的原因不得而知，但由此不难理解为什么早些编译器版本（如g++ 4.4.4）允许右值引用绑定到左值，而最新的编译器却会报错。

另外，介绍一下Howard Hinnant及其维护的标准库：Howard Hinnant是C++标准委员会Library Working Group老大，[libcxx](http://libcxx.llvm.org/)和l[ibcxxabi](http://libcxxabi.llvm.org/)的维护者，苹果公司的高级软件工程师。libcxx库中大量地使用右值引用，想了解更多的右值引用的应用实例，可以瞅瞅libcxx的代码。

参考文档：

[ISO/IEC 14882:2011](http://www.open-std.org/jtc1/sc22/wg21/)

[A Brief Introduction to Rvalue References](http://www.artima.com/cppsource/rvalue.html)

[C++11 FAQ](http://www.stroustrup.com/C++11FAQ.html#rval)

转载 [https://www.cnblogs.com/opangle/archive/2012/11/19/2777131.html](https://www.cnblogs.com/opangle/archive/2012/11/19/2777131.html)