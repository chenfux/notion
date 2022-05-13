# cv-qualifiers

Created: March 30, 2022 6:47 PM
Last Edited Time: April 6, 2022 11:13 AM

**3.9.3 CV-qualifiers**

1. 基本类型(3.9.1)和复合类型(3.9.2)的完整和不完整对象类型或void类型都有 *const-qualified、volatile-qualified 、const-volatile-qualified* 三种相应的cv限定形式 *.*
    
    对象创建时指定的 cv-qualifiers 属于对象类型的一部分 [Note: 也就是说 int, const int, volatile int, const volatile int 是不同的类型 —end node]
    
    类型的 cv-qualifiers 版本和 cv-unqualified 版本属于不同的类型, 但是它们具有相同的表示和对齐要求
    
2. 复合类型不会因为它的组成类型带有 cv-qualifiers 而变成cv限定. cv-qualifiers 应用于数组类型时, 影响的是数组的元素类型, 而不是数组类型本身.
3. 被const限定的类对象的每个 non-static, non-mutable, non-reference 数据成员也都是被const限定的, 被volatile限定的类对象的每个 non-static, non-reference 数据成员也都是被volatile限定的, 被 const-volatile 限定的类成员也类似.
4. [3.9.3/4] 定义了一种基于 cv-qualifiers 的偏序关系, 用于描述一个类型比另一个类型具有更多的cv限定(*more cv-qualified*).
    
            *no cv-qualifier < const
            no cv-qualifier < volatile
            no cv-qualifier < const volatile
            const                  < const volatile
            volatile               < const volatile*
    
    根据cv限定的排序规则, cv限制更少类型T的可以向cv限制更多的类型T进行隐式转换
    
    *no cv-qualifier* T ⇒ const T
    *no cv-qualifier* T ⇒ volatile T
    *no cv-qualifier T ⇒* const volatile T
    const T ⇒ const volatile T
    volatile T => const volatile T
    

cv**限定指针**

1. 限定指针指向的目标为const, 这种指针称为常量指针, 指针可以重新指向新的地址, 但是不能通过指针修改指向的值.
    
          *int n = 0, m = 0;*
    
    *const int* p1 = &n; //ok*
    
    *int const* p2 = &n; //ok*
    
    *const int* p3;          //ok*
    
    *int& r = *p1;             //error, *p1 的类型是 const int*
    
    **p1 = 123;                //error, *p1 的类型是 const int*
    
    *p1 = &m;                   //ok*
    
         *const int& cr = *p1; //ok*
    
2. 限定指针本身为const, 这种指针称为指针常量, 指针不可以重新指向新的地址.

    
          *int n = 0, m = 0;*
    
    *int* const p1 = &n;   //ok*
    
    *int* const p2;            //error, p2不能修改指向地址, 所以必须初始化.*
    
    *int& r = *p1;               //ok*
    
    **p1 = 123;                  //ok*
    
    *const int& cr = *p1; //ok*
    
    *p1 = &m;                    //error, p1 不可以重新指向新的地址*
    
3. 同时限定指针本身和指针所指向的目标都为 const

    
          *int n = 0, m = 0;*
    
    *const int *cosnt p1 = &n;   //ok*
    
    *int const *const p2 = &m; //ok*
    
         **p1 = 1;                                     //error*
    
         **p2 = 2;                                    //error*
    
         *p1 = &m;                                  //error*
    
         *p2 = &n;                                   //error*
    

cv**限定引用**

引用从行为上看起来像是它所引用的目标的一个别名, 所以cv限定引用的情况下, 实际上是对它引用的目标进行cv限定, 而并不是对引用本身来进行限定.

1. cv限定不能应用到引用本身

    
            *int n = 0;
            int & const cr = n; //error: 'const' qualifiers cannot be applied to 'int&'
            int & volatile vr = n; //error: 'volatile' qualifiers cannot be applied to 'int&'*
    

1. 引用必须比引用的目标具有更多的cv限定(*more cv-qualified*)
    
    
          *int n0 = 0;*
    
    *const int n1 = 0;*
    
    *volatile int n2 = 0;
    
    int& r0 = n0;*
    
    *int& r1 = n1;            //error*
    
    *int& r2 = n2;           //error
    
    const int& r3 = n0;*
    
    *const int& r4 = n1;*
    
    *const int& r6 = n2; //error, 
    
    const volatile int& r7 = n0;*
    
    *const volatile int& r8 = n1;*
    
    *const volatile int& r9 = n2;*
    
2. 当右值绑定到左值引用的时候, 左值引用应该带有 const 限定

根据[12.2 Temporary objects/3], 临时对象的销毁应该是它们被创建的那个表达式所在的完整表达式求值过程中的最后一个步骤.  如果允许以临时对象来初始化非const左值引用, 会导致初始化结束之后临时对象被销毁, 从而使引用的目标无效, 又由于这些引用变量可能在其声明周期结束之前被继续访问, 这时对这些引用变量的访问属于非法内存访问.
    
    允许const左值引用绑定到右值首先就可以避免对引用的临时对象的修改问题, 其次为了解决临时对象的销毁问题,  标准允许临时对象被绑定于const左值引用时, 临时对象的声明周期将被延长至其绑定的引用变量的生存周期结束. 
    
          *int & n = 100;        //error*
    
    *const int & n = 0;*
    
    *double d = 0.123;*
    
    *int& r = d;               //error*
    
    *const int& r = d;    //const int& r = int(d)*
    
    *struct A;
    struct B
    {
        B(const A& x);
    };
    
    A a;
    B &b = a;            //error
    const B &b = a; // const B &b = B(a)*
    
3. 限定函数形参(仅const)

    
    这种情况其实更多的可以算作是惯例, 语法上并不会强制一个接受引用为参数的函数必须使用const限定形参. 但是原则上来说, 只要函数内部不修改形参, 并且形参被声明为引用,这时就可以对形参加上const限定, 以表明函数内对参数不做修改.
    
    另外就是使用const修饰引用形参就可以接收右值的实参,或者是一些带有const限定的实参(如果想要向函数传递一个带有const volatile 限定的实参,那么形参也需要通过const volatile修饰).
    
4. cv限定数据成员和成员函数

const 数据成员只能在初始化列表中进行初始化
const 成员函数与同名的非const成员函数构成重载关系
const 成员函数不能修改任何数据成员
const 对象仅能调用const成员函数

1. cv限定与typedef
    
    
    *typedef char* pchar_t;
    char c1 = 'A', c2 = 'B';*
    *const pchar_t p = &c1;  // ⇒ char* const p = &c1;
    *p = 'C';                          //ok
    p = &c2;                         //error, assignment of read-only variable 'p'
    typedef char& rchar_t;*
    const rchar_t r = 'A';     //error, invalid initialization of non-const reference of type 'rchar_t {aka char&}' from an rvalue of type 'char'
    *typedef char  arr_char16_t[16];
    typedef char* arr_charp16_t[16];
    const arr_char16_t arr = {1};
    const arr_charp16_t arrp = {"hello world"};
    arr[0] = 1000;                        //error: assignment of read-only location 'arr[0]'
    arrp[0] = "HELLO WORLD"; //error, assignment of read-only location 'arrp[0]'
    arrp[0][0] = 'H';*
    
    *typedef char  char_t;
    const char_t c = 'A';
    c = 'B';                            //error: assignment of read-only variable 'c'*
    
    *typedef const pchar_t cpchar_t; //*char* const
    *typedef pchar_t const pcchar_t; //*char* const
    *char str[] = "hello world";
    cpchar_t p1 = str;
    pcchar_t p2 = str;
    p1[0] = 'H';
    p2[0] = 'H';
    p1 = str; //error: assignment of read-only variable 'p1'
    p2 = str; //error: assignment of re*ad-only variable 'p2'
    
2. volatile与常量折叠(const folding)
    
    
    常量折叠(const folding), 应该算是一种编译器的优化手段: 编译器在编译过程中会预先将const变量记录到某些"常量表"中, 后续所有对这个const变量的访问实际上都是读取这个常量表中的缓存值.
    
     *const int x1 = 1;
     int& x2 = const_cast<int&>(x1);
     x2  = 2;
     
     printf("%d\n", x1);    //  1
     printf("%d\n", x2);    // 2
    
     printf("%p\n", &x1 );  //0x7ffeeaa5b3d4
     printf("%p\n", &x2 );  //0x7ffeeaa5b3d4*
    
    输出的***x1***和***x2***的地址都是相同的, 但问题是***x1***和***x2***的值是不同的, 这就是由于利用 ***const_cast*** 对 ***x1***的值做了修改, 实际上***x1***指向的内存中确实被修改为新值, 但是由于常量折叠的优化, 导致对x1的访问仍然使用缓存, 所以导致x1读取为旧值. 要解决这种情况, 可以使用**volatile**关键字来修饰变量, 这个关键字要求每次解析变量的时候都从内存中取值.
    
     
    
    volatile *const int x1 = 1;*
     volatile *int& x2 = const_cast<*volatile *int&>(x1);
     x2  = 2;
     
     printf("%d\n", x1);    //  2
     printf("%d\n", x2);    //  2
    
     printf("%p\n", &x1 );  //0x7ffeeaa5b3d4
     printf("%p\n", &x2 );  //0x7ffeeaa5b3d4*
    

1. const修饰函数返回值
2. const限定与mutable
3. 多级指针与const限定

**参考**

ISO/IEC 14882:2011 - 3.9.3 CV-qualifiers

ISO/IEC 14882:2011 - **12.2 Temporary objects**

ISO/IEC 14882-2011 - 8.3.2 References

C++语言99个常见编程错误 - 常见错误5

C++语言99个常见编程错误-常见错误20
****