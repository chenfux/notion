# Templates

Created: April 13, 2022 4:26 PM
Last Edited Time: April 13, 2022 7:00 PM

### 14 Templates

1 A *template* defines a family of classes or functions or an alias for a family of types.

*template-declaration:
    template < template-parameter-list > declaration
template-parameter-list:
    template-parameter
    template-parameter-list , template-parameter*

The *declaration* in a *template-declaration* shall

— declare or define a function or a class, or

— define a member function, a member class, a member enumeration, or a static data member of a class template or of a class nested within a class template, or

— define a member template of a class or class template, or

— be an *alias-declaration*.

A *template-declaration* is a *declaration*. A *template-declaration* is also a definition if its *declaration* defines a function, a class, or a static data member.

```cpp
template<class T> class X {
    static T s;
};
template<class T>   //`a static data member of a class template` 指的是这种情况.
T X<T>::s = 0;
```

2 A *template-declaration* can appear only as a namespace scope or class scope declaration. In a function template declaration, the last component of the *declarator-id* shall not be a *template-id*. [Note: That last component may be an identifier, an operator-function-id, a conversion-function-id, or a literal-operator-id. In a class template declaration, if the class name is a *simple-template-id*, the declaration declares a class template partial specialization (14.5.5). — end note ]

3 In a *template-declaration*, explicit specialization, or explicit instantiation the *init-declarator-list* in the declaration shall contain at most one declarator. When such a declaration is used to declare a class template, no declarator is permitted.

4 A template name has linkage (3.5). A non-member function template can have internal linkage; any other template name shall have external linkage. Specializations (explicit or implicit) of a template that has internal linkage are distinct from all specializations in other translation units. A template, a template explicit specialization (14.7.3), and a class template partial specialization shall not have C linkage. Use of a linkage specification other than C or C++ with any of these constructs is conditionally-supported, with implementation-defined semantics. Template definitions shall obey the one definition rule (3.2). [ Note: Default arguments for function templates and for member functions of class templates are considered definitions for the purpose of template instantiation (14.5) and must also obey the one definition rule. — end note ]

5 A class template shall not have the same name as any other template, class, function, variable, enumeration, enumerator, namespace, or type in the same scope (3.3), except as specified in (14.5.5). Except that a function template can be overloaded either by (non-template) functions with the same name or by other function templates with the same name (14.8.3), a template name declared in namespace scope or in class scope shall be unique in that scope.

6 A function template, member function of a class template, or static data member of a class template shall be defined in every translation unit in which it is implicitly instantiated (14.7.1) unless the corresponding specialization is explicitly instantiated (14.7.2) in some translation unit; no diagnostic is required.

### 14.1 Template parameters

1 The syntax for template-parameters is:

*template-parameter:
    type-parameter
    parameter-declaration
type-parameter:
    class ...opt identifieropt
    class identifieropt = type-id
    typename ...opt identifieropt
    typename identifieropt = type-id
    template < template-parameter-list > class ...opt identifieropt
    template < template-parameter-list > class identifieropt = id-expression*

[ Note: The > token following the *template-parameter-list* of a *type-parameter* may be the product of replacing a >> token by two consecutive > tokens (14.2). — end note ]

2 There is no semantic difference between **class** and **typename** in a *template-parameter*. **typename** followed by an *unqualified-id* names a template type parameter. **typename** followed by a *qualified-id* denotes the type in a *non-type*<sup>137</sup> *parameter-declaration*. A storage class shall not be specified in a *template-parameter* declaration. [ Note: A template parameter may be a class template. For example,

```cpp
template<class T> class myarray { /∗ ... ∗/ };
template<class K, class V, template<class T> class C = myarray>
class Map {
    C<K> key;
    C<V> value;
};
```

— end note ]

```cpp
struct X
{
    typedef int a;
};

template <typename X::a> //这实际上是声明了一个省略模板名称的非类型模板参数, 实际应该是像: template <typename X::a value>
                         //另外, 因为前面typename的存在, 要想这个声明是well-formed, X::a必须表示类型. []
class C {};

C<int> c; //error.
C<0>  c2; //ok
```

3 A *type-parameter* whose identifier does not follow an ellipsis defines its *identifier* to be a *typedef-name* (if declared with *class* or *typename*) or template-name (if declared with **template**) in the scope of the template declaration. [Note: Because of the name lookup rules, a *template-parameter* that could be interpreted as either a non-type *template-parameter* or a *type-parameter* (because its *identifier* is the name of an already existing class) is taken as a *type-parameter*. For example,

```cpp
class T { /∗...∗/ };
int i;

template<class T, T i> void f(T t) {
    T t1 = i;     // template-parameters T and i
    ::T t2 = ::i; // global namespace members T and i
}
```

Here, the template f has a *type-parameter* called T, rather than an unnamed non-type *template-parameter* of class T. — end note ]

在模板声明的作用域中, 如果一个类型参数的标识符后面没有跟着一个省略号, 则这个标识符被是一个typedef名(如果这个标识符声明带有class或typename), 或是一个模板名(如果这个标识符的声明带有一个template)

```cpp
template <typename T1, class T2, template <typename> class T3>
class X{
    T1 t1;
    T2 t2;
    T3<int> t3;
            //ok, T1和T2被视为typedef名, T3被视为模板名
};
```

4 A non-type *template-parameter* shall have one of the following (optionally cv-qualified) types:

— integral or enumeration type,

— pointer to object or pointer to function,

— lvalue reference to object or lvalue reference to function,

— pointer to member,

— std::nullptr_t.

5 [Note: Other types are disallowed either explicitly below or implicitly by the rules governing the form of template-arguments (14.3). — end note ] The top-level *cv-qualifiers* on the *template-parameter* are ignored when determining its type.

```cpp
template<int N> struct A;
template<const int N> struct A;
```

> https://stackoverflow.com/questions/9679116/is-there-any-difference-between-t-and-const-t-in-template-parameter/9679228
https://stackoverflow.com/questions/3361926/why-are-cv-qualifiers-in-template-parameters-ignored
> 

6 A non-type non-reference *template-parameter* is a prvalue. It shall not be assigned to or in any other way have its value changed. A non-type non-reference *template-parameter* cannot have its address taken. When a non-type non-reference *template-parameter* is used as an initializer for a reference, a temporary is always used.

```cpp
template<const X& x, int i> void f() {
    i++;            // error: change of template-parameter value

    &x;             // OK
    &i;             // error: address of non-reference template-parameter

    int& ri=i;      // error: non-const reference bound to temporary
    const int& cri = i;  // OK: const reference bound to temporary
}
```

7 A non-type *template-parameter* shall not be declared to have floating point, class, or void type.

```cpp
template<double d> class X;     // error
template<double* pd> class Y;   // OK
template<double& rd> class Z;   // OK
```

8 A non-type *template-parameter* of type “array of T” or “function returning T” is adjusted to be of type “pointer to T” or “pointer to function returning T”, respectively.

```cpp
template<int *a> struct R { /∗ ... ∗/ };
template<int b[5]> struct S { /∗ ... ∗/ };
int p;
R<&p> w;    // OK
S<&p> x;    // OK due to parameter adjustment
int v[5];
R<v> y;     // OK due to implicit argument conversion
S<v> z;     // OK due to both adjustment and conversion
```

9 A *default template-argument* is a *template-argument* (14.3) specified after = in a *template-parameter*. A default *template-argument* may be specified for any kind of *template-parameter* (type, non-type, template) that is not a template parameter pack (14.5.3). A default *template-argument* may be specified in a template declaration. A default template-argument shall not be specified in the *template-parameter-lists* of the definition of a member of a class template that appears outside of the member’s class. A default *template-argument* shall not be specified in a friend class template declaration. If a friend function template declaration specifies a default *template-argument*, that declaration shall be a definition and shall be the only declaration of the function template in the translation unit.

10 The set of default *template-arguments* available for use with a template declaration or definition is obtained by merging the default arguments from the definition (if in scope) and all declarations in scope in the same way default function arguments are (8.3.6).

可用于模板声明或定义的默认模板参数集合是通过合并作用域内所有来声明和定义中的默认参数来得到. （这同样适用于函数模板默认实参)

```cpp
template<class T1, class T2 = int> class A;
template<class T1 = int, class T2> class A;
```

is equivalent to

```cpp
template<class T1 = int, class T2 = int> class A;
```

11 If a *template-parameter* of a class template or alias template has a default *template-argument*, each subsequent(每个后续的) *template-parameter* shall either have a default *template-argument* supplied or be a template parameter pack. If a *template-parameter* of a primary class template(基本类模板, 也就是特化版本的类模板) or alias template is a template parameter pack, it shall be the last *template-parameter*. A template parameter pack of a function template shall not be followed by another template parameter unless that template parameter can be deduced or has a default argument (14.8.2).

```cpp
template<class T1 = int, class T2> class B;  // error
```

```cpp
// U cannot be deduced or specified
template<class... T, class... U> void f() { }
template<class... T, class U> void g() { }
```

12 A *template-parameter* shall not be given default arguments by two different declarations in the same scope.

```cpp
template<class T = int> class X;
template<class T = int> class X { /∗... ∗/ }; // error
```

13 When parsing a default *template-argument* for a non-type *template-parameter*, the first non-nested > is taken as the end of the *template-parameter-list* rather than a greater-than operator.

```cpp
template<int i = 3 > 4 >   // syntax error
class X { /∗...∗/ };

template<int i = (3 > 4) > // OK
class Y { /∗...∗/ };
```

14 A *template-parameter* of a template *template-parameter* is permitted to have a default *template-argument*. When such default arguments are specified, they apply to the template *template-parameter* in the scope of the template *template-parameter*. [Example:

```cpp
template <class T = float> struct B {};
template <template <class TT = float> class T> struct A {
    inline void f();
    inline void g();
};
template <template <class TT> class T> void A<T>::f() {
    T<> t; // error - TT has no default template argument
 }

template <template <class TT = char> class T> void A<T>::g() {
    T<> t; // OK - T<char>
}
```

15 If a *template-parameter* is a *type-parameter* with an ellipsis prior to(在...之前) its optional identifier or is a *parameter-declaration* that declares a parameter pack (8.3.5), then the *template-parameter* is a template parameter pack (14.5.3). A template parameter pack that is a *parameter-declaration* whose type contains one or more unexpanded parameter packs is a pack expansion. Similarly, a template parameter pack that is a *type-parameter* with a *template-parameter-list* containing one or more unexpanded parameter packs is a pack expansion. A template parameter pack that is a pack expansion shall not expand(被展开) a parameter pack declared in the same *template-parameter-list*.

```cpp
template <class... Types> class Tuple;  // Types is a template type parameter pack
                                        // but not a pack expansion
template <class T, int... Dims> struct multi_array; // Dims is a non-type template parameter pack
                                                    // but not a pack expansion
template<class... T> struct value_holder {
  template<T... Values> apply { };  // Values is a non-type template parameter pack
                                    // and a pack expansion
};
template<class... T, T... Values> struct static_array;  // error: Values expands template type parameter
                                                        // pack T within the same template parameter list
```

### 14.2 Names of template specializations

1 A template specialization (14.7) can be referred to by a *template-id*:

*simple-template-id:
    template-name < template-argument-listopt >

template-id:
    simple-template-id
    operator-function-id < template-argument-listopt >
    literal-operator-id < template-argument-listopt >

template-name:
    identifier

template-argument-list:
    template-argument ...opt
    template-argument-list , template-argument ...opt

template-argument:
    constant-expression
    type-id
    id-expression*

[Note: The name lookup rules (3.4) are used to associate the use of a name with a template declaration; that is, to identify a name as a *template-name*. — end note ]

2 For a *template-name* to be explicitly qualified by the template arguments, the name must be known to refer to a template.

这段定义意味着对模板特化时必须对该模板的模板声明可见.

```cpp
//template <typename T>
//struct X{};

template <typename T>
struct X<T*> //X是`template-name to be explicitly qualified by the template arguments`, 但它没有引用到一个模板X的声明
{
};

X<int*> x; //error
```

> [https://tieba.baidu.com/p/1076144275?traceid=&red_tag=2089621276](https://tieba.baidu.com/p/1076144275?traceid=&red_tag=2089621276)
> 

3 After name lookup (3.4) finds that a name is a *template-name* or that an *operator-function-id* or a *literal- operator-id* refers to a set of overloaded functions any member of which is a function template if this is followed by a <, the < is always taken as the delimiter of a *template-argument-list* and never as the less-than operator. When parsing a *template-argument-list*, the first non-nested >138 is taken as the ending delimiter rather than a greater-than operator. Similarly, the first non-nested >> is treated as two consecutive but distinct > tokens, the first of which is taken as the end of the *template-argument-list* and completes the *template-id*. [Note: The second > token produced by this replacement rule may terminate an enclosing *template-id* construct or it may be part of a different construct (e.g. a cast). — end note ]

```cpp
template<int i> class X { /* ... */ };

X< 1>2 > x1; // syntax error
X<(1>2)> x2; // OK

template<class T> class Y { /* ... */ };
Y<X<1>> x3;         // OK, same as Y<X<1> > x3;
Y<X<6>>1>> x4;      // syntax error
Y<X<(6>>1)>> x5;    // OK
```

> 138) A > that encloses the type-id of a dynamic_cast, static_cast, reinterpret_cast or const_cast, or which encloses the template-arguments of a subsequent template-id, is considered nested for the purpose of this description.
> 

4 When the name of a member template specialization appears after . or -> in a *postfix-expression* or after a *nested-name-specifier* in a *qualified-id*, and the object expression of the *postfix-expression* is type-dependent or the *nested-name-specifier* in the *qualified-id* refers to a dependent type, but the name is not a member of the current instantiation (14.6.2.1), the member template name must be prefixed by the keyword **template**. Otherwise the name is assumed to name a non-template.

需要以template标识为模板的几种情况:

- 当成员模板特化出现在前缀表达式的.或->之后, 并且前缀表达式的对象表达式是类型依赖的(type-dependent), 并且这个成员模板不是当前实例的成员, 或
- 当成员模板出现在一个限定id中的*nested-name-specifier*之后, 并且这个限定ID中的*nested-name-specifier*引用一个依赖类型, 并且这个成员模板不是当前实例的成员,
- 这种情况下这个成员模板必须以template为前缀标识该成员是一个模板, Otherwise the name is assumed to name a non-template.

```cpp
struct X {
    template<std::size_t> X* alloc();
    template<std::size_t> static X* adjust();

    template <typename T>
    struct S{};
};
template<class T> void f(T* p) {
    T* p1 = p->alloc<200>();            // ill-formed: < means less than
    T* p2 = p->template alloc<200>();   // OK: < starts template argument list
    T::adjust<100>();                   // ill-formed: < means less than
    T::template adjust<100>();          // OK: < starts template argument list
}
```

关于上面提到的"but the name is not a member of the current instantiation" 指的是这种情况:

```cpp
template <class T> class A {
    A* p1;      // A is the current instantiation
    A<T>* p2;   // A<T> is the current instantiation
    ::A<T>* p4; // ::A<T> is the current instantiation
    A<T*> p3;   // A<T*> is not the current instantiation
};
template <class T> class A<T*> {
    A<T*>* p1;  // A<T*> is the current instantiation
    A<T>* p2;   // A<T> is not the current instantiation

    template <typename TT>
    struct Vt{};

    typedef A<T>::Vt<int> value_type1;  //error, 应该是typedef typename A<T>::template Vt<int> value_type1;
    typedef A<T*>::Vt<int> value_type2  //ok,  不需要template.
};
```

> http://en.cppreference.com/w/cpp/language/dependent_name
> 

5 A name prefixed by the keyword **template** shall be a **template-id** or the name shall refer to a class template. [Note: The keyword **template** may not be applied to non-template members of class templates. —end note ] [ Note: As is the case with the **typename** prefix, the **template** prefix is allowed in cases where it is not strictly necessary; i.e., when the *nested-name-specifier* or the expression on the left of the -> or . is not dependent on a template-parameter, or the use does not appear in the scope of a template. —end note]

```cpp
template <class T> struct A {
    void f(int);
    template <class U> void f(U);
};
template <class T> void f(T t) {
    A<T> a;
    a.template f<>(t);      // OK: calls template
    a.template f(t);        // error: not a template-id
}
template <class T> struct B {
    template <class T2> struct C { };
};

// OK: T::template C names a class template:
template <class T, template <class X> class TT = T::template C> struct D { };
D<b<int> > db;
```

6 A *simple-template-id* that names a class template specialization is a *class-name* (Clause 9).

7 A *template-id* that names an alias template specialization is a *type-name*.

### 14.3 Template arguments

1 There are three forms of *template-argument*, corresponding to the three forms of *template-parameter*: type, non-type and template. The type and form of each *template-argument* specified in a *template-id* shall match the type and form specified for the corresponding parameter declared by the template in its *template-parameter-list*. When the parameter declared by the template is a template parameter pack (14.5.3), it will correspond to zero or more *template-arguments*.

```cpp
template<class T> class Array {
    T* v;
    int sz;
public:
    explicit Array(int);
    T& operator[](int);
    T& elem(int i) { return v[i]; }
};

Array<int> v1(20);
typedef std::complex<double> dcomplex;    // std::complex is a standard library template
Array<dcomplex> v2(30);
Array<dcomplex> v3(40);

void bar() {
    v1[3] = 7;
    v2[3] = v3.elem(4) = dcomplex(7,8);
}
```

2 In a *template-argument*, an ambiguity between a *type-id* and an expression is resolved to a *type-id*, regardless of(无论) the form of the corresponding *template-parameter* .139

```cpp
/*
template<class T> void f()
{
	std::function<T> f([]()->int{
		std::cout << "hello" << std::endl;
		return 0;
	});
	f();
}
*/
template<class T> void f();
template<int I> void f();
void g() {
    f<int()>(); // int() is a type-id: call the first f()
}
```

3 The name of a *template-argument* shall be accessible at the point where it is used as a *template-argument*. [Note: If the name of the *template-argument* is accessible at the point where it is used as a *template-argument*, there is no further access restriction in the resulting instantiation where the corresponding template-parameter name is used. — end note ]

```cpp
template<class T> class X {
    static T t;
};
class Y {
private:
    struct S { /∗ ... ∗/ };
    X<S> x; // OK: S is accessible
            // X<Y::S> has a static member of type Y::S
            // OK: even though Y::S is private
};
X<Y::S> y; // error: S not accessible
```

For a *template-argument* that is a class type or a class template, the template definition has no special access rights to the members of the template-argument.

```cpp
template <template <class TT> class T> class A {
    typename T<int>::S s;
};
template <class U> class B {
private:
    struct S { /∗ ... ∗/ };
};
A<B> b; // ill-formed: A has no access to B::S
```

4 When template argument packs or default *template-arguments* are used, a *template-argument* list can be empty. In that case the empty <> brackets shall still be used as the *template-argument-list*.

```cpp
template<class T = char> class String;
String<>* p; // OK: String<char>
String* q;  // syntax error
template<class ... Elements> class Tuple;

Tuple<>* t; // OK: Elements is empty
Tuple* u;   // syntax error
```

5 An explicit destructor call (12.4) for an object that has a type that is a class template specialization may explicitly specify the *template-arguments*.

```cpp
template<class T> struct A {
    ~A();
};
void f(A<int>* p, A<int>* q) {
    p->A<int>::~A();        // OK: destructor call
    q->A<int>::~A<int>();   // OK: destructor call
}
```

6 If the use of a *template-argument* gives rise to an ill-formed construct in the instantiation of a template specialization, the program is ill-formed.

7 When the template in a *template-id* is an overloaded function template, both non-template functions in the overload set and function templates in the overload set for which the *template-arguments* do not match the template-parameters are ignored. If none of the function templates have matching *template-parameters*, the program is ill-formed.

8 A *template-argument* followed by an ellipsis is a pack expansion (14.5.3).

### 14.3.1 Template type arguments

1 A *template-argument* for a *template-parameter* which is a type shall be a *type-id*.

```cpp
template <class T> class X { };
template <class T> void f(T t) { }
struct { } unnamed_obj;

void f() {
    struct A { };
    enum { e1 };
    typedef struct { } B;
    B b;
    X<A> x1;        //OK
    X<A*> x2;       //OK
    X<B> x3;        //OK
    f(e1);          //OK
    f(unnamed_obj); //OK
    f(b);           //OK
}
```

[ Note: A template type argument may be an incomplete type (3.9). — end note ]

3 If a declaration acquires a function type through a type dependent on a *template-parameter* and this causes a declaration that does not use the syntactic form of a function declarator to have function type, the program is ill-formed.

如果模板形参具有函数类型, 并且以这个类型来声明函数且不使用函数声明语法,  the program is ill-formed.
这种情况应该是对函数类型做特化处理

```cpp
template<class T> struct A {
  static T t;
};
typedef int function();
A<function> a;  // ill-formed: would declare A<function>::t
                // as a static member function
```

### 14.3.2 Template non-type arguments

1 A *template-argument* for a non-type, non-template *template-parameter* shall be one of:

— for a non-type template-parameter of integral or enumeration type, a converted constant expression (5.19) of the type of the template-parameter; or

— the name of a non-type template-parameter; or

— a constant expression (5.19) that designates the address of an object with static storage duration and external or internal linkage or a function with external or internal linkage, including function templates and function template-ids but excluding non-static class members, expressed (ignoring parentheses) as & id-expression, except that the & may be omitted if the name refers to a function or array and shall be omitted if the corresponding template-parameter is a reference; or

— a constant expression that evaluates to a null pointer value (4.10); or

— a constant expression that evaluates to a null member pointer value (4.11); or

— a pointer to member expressed as described in 5.3.1.

2 [ Note: A string literal (2.14.5) does not satisfy the requirements of any of these categories and thus is not an acceptable *template-argument*.

```
template<class T, const char* p> class X { /∗ ... ∗/ };

X<int, "Studebaker"> x1;    // error: string literal as template-argument

const char p[] = "Vivisectionist";
X<int,p> x2;                // OK
```

— end note ]

3 [ Note: Addresses of array elements and names or addresses of non-static class members are not acceptable *template-arguments*.

```
template<int* p> class X { };
int a[10];
struct S { int m; static int s; } s;
X<&a[2]> x3;   // error: address of array element
X<&s.m> x4;    // error: address of non-static member
X<&s.s> x5;    // error: &S::s must be used
X<&S::s> x6;   // OK: address of static member
```

4 [Note: Temporaries, unnamed lvalues, and named lvalues with no linkage are not acceptable *template-arguments* when the corresponding template-parameter has reference type.

```
template<const int& CRI> struct B { /∗ ... ∗/ };

B<1> b2; // error: temporary would be required for template argument

int c = 1;
B<c> b1; // OK
```

— end note ]

5 The following conversions are performed on each expression used as a non-type template-argument. ...

这一段定义了非类型模板实参向模板形参类型发生的转换, 需要时参考.

### 14.3.3 Template template arguments

1 A *template-argument* for a template *template-parameter* shall be the name of a class template or an alias template, expressed as(被表示为) *id-expression*. When the *template-argument* names a class template, only primary class templates(基本类模板) are considered when matching the template template argument with the corresponding parameter; partial specializations are not considered even if their parameter lists match that of the template template parameter.

2 Any partial specializations (14.5.5) associated with the primary class template are considered when a specialization based on the template template-parameter is instantiated. If a specialization is not visible at the point of instantiation, and it would have been selected had it been visible, the program is ill-formed; no diagnostic is required.

```
template<class T> class A {     // primary template
    int x;
};
template<class T> class A<T*> { // partial specialization
    long x;
};
template<template<class U> class V> class C {
    V<int>  y;
    V<int*> z;
};
C<A> c;
    // V<int> within C<A> uses the primary template,
    // so c.y.x has type int
    // V<int*> within C<A> uses the partial specialization,
    // so c.z.x has type long
```

```
template<class T> class A { /∗ ... ∗/ };
template<class T, class U = T> class B { /∗ ... ∗/ };
template <class ... Types> class C { /∗ ... ∗/ };
template<template<class> class P> class X { /∗ ... ∗/ };
template<template<class ...> class Q> class Y { /∗ ... ∗/ };
X<A> xa;    // OK
X<B> xb;    // ill-formed: default arguments for the parameters of a template argument are ignored
X<C> xc;    // ill-formed: a template parameter pack does not match a template parameter

Y<A> ya;    // OK
Y<B> yb;    // OK
Y<C> yc;    // OK
```

3 A *template-argument* matches a template *template-parameter*(call it P) when each of the template parameters in the *template-parameter-list* of the *template-argument’s* corresponding class template or alias template (call it A) matches the corresponding template parameter in the *template-parameter-list* of P. When P’s *template-parameter-list* contains a template parameter pack (14.5.3), the template parameter pack will match zero or more template parameters or template parameter packs in the *template-parameter-list* of A with the same type and form as the template parameter pack in P (ignoring whether those template parameters are template parameter packs)

当'模板的模板实参'对应的类模板或别名模板(称为A)的模板参数列表中的每个模板形参与模板的模板形参(称为P)的模板参数列表中的每个相应的模板形参匹配时, 认为A与P匹配. (就是说模板的模板参数是否匹配看的是模板的模板参数的模板参数列表是否匹配)
当P(模板的模板形参)的模板参数列表包含一个模板参数包时, 模板参数包将匹配0个或多个模板形参, 或模板参数包在A(模板的模板实参)的模板参数列表中与模板参数包在P的模板参数列表中具有相同的类型和形式(无论那些模板参数是否是模板参数包),

```
template <class T> struct eval;

template <template <class, class...> class TT, class T1, class... Rest>
struct eval<TT<T1, Rest...>> { };

template <class T1> struct A;
template <class T1, class T2> struct B;
template <int N> struct C;
template <class T1, int N> struct D;
template <class T1, class T2, int N = 17> struct E;

eval<A<int>> eA;          // OK: matches partial specialization of eval
eval<B<int, float>> eB;   // OK: matches partial specialization of eval
eval<C<17>> eC;           // error: C does not match TT in partial specialization  eval的模板形参TT的模板参数列表不支持非类型参数.
eval<D<int, 17>> eD;      // error: D does not match TT in partial specialization  eval的模板形参TT的模板参数列表不支持非类型参数.
eval<E<int, float>> eE;   // error: E does not match TT in partial specialization  eval的模板形参TT的模板参数列表不支持非类型参数.
```

### 14.4 Type equivalence

1 Two *template-ids* refer to the same class or function if

— their *template-names*, *operator-function-ids*, or *literal-operator-ids* refer to the same template and

— their corresponding type *template-arguments* are the same type and

— their corresponding non-type template arguments of integral or enumeration type have identical values and

— their corresponding non-type *template-arguments* of pointer type refer to the same external object or function or are both the null pointer value and

— their corresponding non-type *template-arguments* of pointer-to-member type refer to the same class member or are both the null member pointer value and

— their corresponding non-type *template-arguments* of reference type refer to the same external object or function and

— their corresponding template *template-arguments* refer to the same template.

```
template<class E, int size> class buffer { /∗ ... ∗/ };
buffer<char,2*512> x;
buffer<char,1024> y;
```

declares x and y to be of the same type, and

```
template<class T, void(*err_fct)()> class list { /∗ ... ∗/ };
list<int,&error_handler1> x1;
list<int,&error_handler2> x2;
list<int,&error_handler2> x3;
list<char,&error_handler2> x4;
```

declares x2 and x3 to be of the same type. Their type differs from the types of x1 and x4.

```
template<class T> struct X { };
template<class> struct Y { };
template<class T> using Z = Y<T>;
X<Y<int> > y;
X<Z<int> > z;
```

declares y and z to be of the same type. — end example ]

2 If an expression e involves a template parameter, **decltype(e)** denotes a unique dependent type. Two such *decltype-specifiers* refer to the same type only if their expressions are equivalent (14.5.6.1). [Note: however, it may be aliased, e.g., by a *typedef-name*. — end note ]

### 14.5 Template declarations

1 A *template-id*, that is, the *template-name* followed by a *template-argument-list* shall not be specified in the declaration of a primary template declaration.

```
template<class T1, class T2, int I> class A<T1, T2, I> { }; // error
template<class T1, int I> void sort<T1, I>(T1 data[I]); // error
```

[Note: However, this syntax is allowed in class template partial specializations (14.5.5). — end note ]

2 For purposes of name lookup and instantiation, default arguments of function templates and default arguments of member functions of class templates are considered definitions; each default argument is a separate(单独的) definition which is unrelated to the function template definition or to any other default arguments

3 Because an *alias-declaration* cannot declare a *template-id*, it is not possible to partially or explicitly specialize an alias template.

```
alias-declaration:
using identifier attribute-specifier-seqopt = type-id ;
```

### 14.5.1 Class templates

1 A class *template* defines the layout and operations for an unbounded set of related types. [ Example: a single class template List might provide a common definition for list of int, list of float, and list of pointers to Shapes. —endexample]

类模板为一个类型相关的无限集合定义了布局和操作

[ Example: An array class template might be declared like this:

```
template<class T> class Array {
  T* v;
  int sz;
public:
  explicit Array(int);
  T& operator[](int);
  T& elem(int i) { return v[i]; }
};
```

2 The prefix **template <class T>** specifies that a template is being declared and that a *type-name* T will be used in the declaration. In other words, Array is a parameterized type with T as its parameter. —end example ]

3 When a member function, a member class, a member enumeration, a static data member or a member template of a class template is defined outside of the class template definition, the member definition is defined as a template definition in which the *template-parameters* are those of the class template. The names of the template parameters used in the definition of the member may be different from the template parameter names used in the class template definition. The template argument list following the class template name in the member definition shall name the parameters in the same order as the one used in the template parameter list of the member. Each template parameter pack shall be expanded(被展开) with an ellipsis in the template argument list.

当一个类模板的成员函数, 成员类, 成员枚举, 静态数据成员,成员模板被定义在类模板的定义之外时,  这些成员定义被定义为模板定义, 其中模板形参就是类模板的形参.
类模板形参的名字在成员定义中可以与被用在类模板定义中的模板形参名不同。
在(类模板外部的)成员定义中,类模板名后面的模板实参列表命名的顺序应与成员的模板参数列表中使用的顺序相同.

```
template<class T1, class T2> struct A {
  void f1();
  void f2();
  void f3();
  void f4();
};

template<class T2, class T1> void A<T2,T1>::f1() { }    // OK
template<class T2, class T1> void A<T1,T2>::f2() { }    // error
template<class N, class M> void A<N,M>::f3() { }        // OK
void A::f4();                                           //error

template<class ... Types> struct B {
  void f3();
  void f4();
};
template<class ... Types> void B<Types ...>::f3() { }   // OK
template<class ... Types> void B<Types>::f4() { }   // error
```

4 In a redeclaration, partial specialization, explicit specialization or explicit instantiation of a class template,
the *class-key* shall agree in kind with the original class template declaration (7.1.6.3).

### 14.5.1.1 Member functions of class templates

1 A member function of a class template may be defined outside of the class template definition in which it is declared.

```
template<class T> class Array {
    T* v;
    int sz;
public:
    explicit Array(int);
    T& operator[](int);
    T& elem(int i) { return v[i]; }
};

```

declares three function templates. The subscript function might be defined like this:

```
template<class T> T& Array<T>::operator[](int i) {
    if (i<0 || sz<=i) error("Array: range error");
    return v[i];
}

```

2 The *template-arguments* for a member function of a class template are determined by the *template-arguments* of the type of the object for which the member function is called. [Example: the template-argument for **Array<T> :: operator [] ()** will be determined by the **Array** to which the subscripting operation is applied.

```
Array<int> v1(20);
Array<dcomplex> v2(30);
v1[3] = 7;              // Array<int>::operator[]()
v2[3] = dcomplex(7,8); // Array<dcomplex>::operator[]()

```

### 14.5.1.3 Static data members of class templates

1 A definition for a static data member may be provided in a namespace scope enclosing the definition of the static member’s class template.

```
template<class T> class X {
    static T s;
};
template<class T> T X<T>::s = 0;

```

2 An explicit specialization of a static data member declared as an array of unknown bound can have a different
bound from its definition, if any.

```
     template <class T> struct A {
       static int i[];
};
template <class T> int A<T>::i[4]; // 4 elements
template <> int A<int>::i[] = { 1 }; // OK: 1 element

```

### 14.5.1.4 Enumeration members of class templates

1 An enumeration member of a class template may be defined outside the class template definition.

```
template<class T> struct A {
    enum E : T;
};
A<int> a;
template<class T> enum A<T>::E : T { e1, e2 };
A<int>::E e = A<int>::e1;

```

### 14.5.2 Member templates

1 A template can be declared within a class or class template; such a template is called a member template. A member template can be defined within or outside its class definition or class template definition. A member template of a class template that is defined outside of its class template definition shall be specified with the *template-parameters* of the class template followed by the *template-parameters* of the member template.

```
template<class T> struct string {
    template<class T2> int compare(const T2&);
    template<class T2> string(const string<T2>& s) { /∗ ... ∗/ }
};
template<class T>
template<class T2>
int string<T>::compare(const T2& s) {
}

```

2 A local class shall not have member templates. Access control rules (Clause 11) apply to member template names. A destructor shall not be a member template. A normal (non-template) member function with a given name and type and a member function template of the same name, which could be used to generate a specialization of the same type, can both be declared in a class. When both exist, a use of that name and type refers to the non-template member unless an explicit template argument list is supplied.

相同名字的成员函数和成员函数模板如果都存在, 那么非模板版本的将被视为一个相同类型的特化, 并且使用这个名字和类型将优先引用非模板版本, 除非显式指定模板实参列表.

```
template <class T> struct A {
  void f(int);
  template <class T2> void f(T2);
};
template <> void A<int>::f(int) { }                 // non-template member
template <> template <> void A<int>::f<>(int) { }    // template member
  int main() {
    A<char> ac;
    ac.f(1);    // non-template
    ac.f(’c’);  // template
    ac.f<>(1);  // template
}
```

3 A member function template shall not be virtual.

```
template <class T> struct AA {
	template <class C> virtual void g(C); // error
	virtual void f();                       // OK
};
```

4 A specialization of a member function template does not override a virtual function from a base class.

```
class B {
    virtual void f(int);
};
class D : public B {
template <class T> void f(T); // does not override B::f(int)
void f(int i) { f<>(i); }    // overriding function that calls
                             // the template instantiation
};
```

5 A specialization of a conversion function template is referenced in the same way as a non-template conversion
function that converts to the same type.

```
struct A {
    template <class T> operator T*();
};
template <class T> A::operator T*(){ return 0; }
template <> A::operator char*(){ return 0; } // specialization
template A::operator void*();               // explicit instantiation
int main() {
  A a;
  int *ip;
  ip = a.operator int*();   // explicit call to template operator
                            // A::operator int*()
}

```

[Note: Because the explicit template argument list follows the function template name, and because conversion member function templates and constructor member function templates are called without using a function name, there is no way to provide an explicit template argument list for these function templates. —end note]

6 A specialization of a conversion function template is not found by name lookup. Instead, any conversion function templates visible in the context of the use are considered. For each such operator, if argument deduction succeeds (14.8.2.3), the resulting specialization is used as if found by name lookup.

7 A *using-declaration* in a derived class cannot refer to a specialization of a conversion function template in a base class.

8 Overload resolution (13.3.3.2) and partial ordering (14.5.6.2) are used to select the best conversion function among multiple specializations of conversion function templates and/or non-template conversion functions.

### 14.5.3 Variadic templates

1 A *template parameter pack* is a template parameter that accepts zero or more template arguments.

```
template<class ... Types> struct Tuple { };

Tuple<> t0;             // Types contains no arguments
Tuple<int> t1;          // Types contains one argument: int
Tuple<int, float> t2;   // Types contains two arguments: int and float
Tuple<0> error;         // error: 0 is not a type

```

2 A *function parameter pack* is a function parameter that accepts zero or more function arguments.

```
template<class ... Types> void f(Types ... args);
f();        // OK: args contains no arguments
f(1);       // OK: args contains one argument: int
f(2, 1.0);  // OK: args contains two arguments: int and double

```

3 A parameter pack is either a template parameter pack or a function parameter pack.

4 A *pack expansion* consists of a *pattern* and an ellipsis, the instantiation of which produces zero or more instantiations of the pattern in a list (described below). The form of the pattern depends on the context in which the expansion occurs. Pack expansions can occur in the following contexts:

包展开由一个模式和一个省略号组成,

— In a function parameter pack (8.3.5); the pattern is the parameter-declaration without the ellipsis.

— In a template parameter pack that is a pack expansion (14.1):

```
— if the template parameter pack is a parameter-declaration; the pattern is the parameter-declaration without the ellipsis;
— if the template parameter pack is a type-parameter with a template-parameter-list; the pattern is the corresponding type-parameter without the ellipsis.

```

— In an initializer-list (8.5); the pattern is an initializer-clause.

— In a base-specifier-list (Clause 10); the pattern is a base-specifier.

— In a mem-initializer-list (12.6.2); the pattern is a mem-initializer.

— In a template-argument-list (14.3); the pattern is a template-argument.

— In a dynamic-exception-specification (15.4); the pattern is a type-id.

— In an attribute-list (7.6.1); the pattern is an attribute.

— In an alignment-specifier (7.6.2); the pattern is the alignment-specifier without the ellipsis.

— In a capture-list (5.1.2); the pattern is a capture.

— In a sizeof... expression (5.3.3); the pattern is an identifier.

```
template<class ... Types> void f(Types ... rest);
template<class ... Types> void g(Types ... rest) {
    f(&rest ...); // “&rest ...” is a pack expansion; “&rest” is its pattern
}

```

5 A parameter pack whose name appears within the pattern of a pack expansion is expanded by that pack expansion. An appearance of the name of a parameter pack is only expanded by the innermost enclosing pack expansion. The pattern of a pack expansion shall name one or more parameter packs that are not expanded by a nested pack expansion; such parameter packs are called *unexpanded* parameter packs in the pattern. All of the parameter packs expanded by a pack expansion shall have the same number of arguments specified. An appearance of a name of a parameter pack that is not expanded is ill-formed.

出现在包扩展模式中的参数包的名字是一个被展开的包扩展.
参数包的名称仅由最内层的包装扩展来展开
包扩展的模式应该命名一个或多个没有被嵌套的包扩展展开过的参数包.
所有通过包扩展被展开的参数包都应该具有相同的实参个数.
未展开的参数包的名称的出现是ill-formed.

```
template<typename...> struct Tuple {};
template<typename T1, typename T2> struct Pair {};
template<class ... Args1> struct zip {
    template<class ... Args2> struct with {
        typedef Tuple<Pair<Args1, Args2> ... > type;
    };
};

typedef zip<short, int>::with<unsigned short, unsigned>::type T1;   // T1 is Tuple<Pair<short, unsigned short>, Pair<int, unsigned>>
typedef zip<short>::with<unsigned short, unsigned>::type T2;        // error: different number of arguments specified for Args1 and Args2
                                                                    // `All of the parameter packs expanded by a pack expansion shall have the same number of arguments specified.`

template<class ... Args>
void g(Args ... args) {             // OK: Args is expanded by the function parameter pack args
    f(const_cast<const Args*>(&args)...);   // OK: “Args” and “args” are expanded
    f(5 ...);                   // error: pattern does not contain any parameter packs
    f(args);                    // error: parameter pack “args” is not expanded       `An appearance of a name of a parameter pack that is not expanded is ill-formed.`
    f(h(args ...) + args ...);  // OK: first “args” expanded within h, second “args” expanded within f `An appearance of the name of a parameter pack is only expanded by the innermost enclosing pack expansion.`
                              // inner pack expansion is "args...", it is expanded first
                              // outer pack expansion is h(E1, E2, E3) + args..., it is expanded
                              // second (as h(E1,E2,E3) + E1, h(E1,E2,E3) + E2, h(E1,E2,E3) + E3)
}
```

```
template <typename... O>
struct out{};

template <typename... I>
struct in{};

template <typename... Args>
struct S
{
	typedef out<in<Args...>...> type;  //in< Args...>不是一个`unexpanded parameter packs`
                                        //`The pattern of a pack expansion shall name one or more parameter packs that are not expanded by a nested pack expansion; ...`
};
```

6 The instantiation of a pack expansion that is not a **sizeof...** expression produces a list E1, E2, ..., EN , where N is the number of elements in the pack expansion parameters. Each Ei is generated by instantiating the pattern and replacing each pack expansion parameter with its ith element. All of the Ei become elements in the enclosing list. [Note: The variety of list varies with the context: expression-list, base-specifier-list, template-argument-list, etc.—end note] When N is zero, the instantiation of the expansion produces an empty list. Such an instantiation does not alter the syntactic interpretation of the enclosing construct, even in cases where omitting the list entirely would otherwise be ill-formed or would result in an ambiguity in the grammar.

除`sizeof...`表达式以外的包扩展的实例化将产生一个E1, E2, E3, ... EN的列表, 这里N是包扩展形参的元素个数.
每个Ei是通过实例化模式并用它的第i个元素替换每个包扩展形参而生成的.
(例如上面例子中的 `typedef Tuple<Pair<Args1, Args2> ... > type;`  当模式Args1的包含的模板实参类型为short, int, 当模式Args2包含的模板实参类型为unsigned short, unsigned,
根据这段定义, E0就是将Args1用其第0号元素替换, Args2用其第0号元素替换, 得到Pair<short, unsigned short>, E1就是将Args1用其第1号元素替换, Args2用其第1号元素替换,得到 Pair<int, unsigned int>);

```
template<class... T> struct X : T... { };
template<class... T> void f(T... values) {
    X<T...> x(values...);
}

template void f<>(); // OK: X<> has no base classes
                    // x is a variable of type X<> that is value-initialized
```

7 The instantiation of a **sizeof...** expression (5.3.3) produces an integral constant containing the number
of elements in the parameter pack it expands.

### 14.5.4 Friends

### 14.5.5 Class template partial specializations

1 A *primary* class template declaration is one in which the class template name is an identifier. A template declaration in which the class template name is a *simple-template-id* is a *partial specialization* of the class template named in the *simple-template-id*. A partial specialization of a class template provides an alternative(代替的) definition of the template that is used instead of(代替...) the primary definition when the arguments in a specialization match those given in the partial specialization (14.5.5.1). The primary template shall be declared before any specializations of that template. A partial specialization shall be declared before the first use of a class template specialization that would make use of the partial specialization as the result of an implicit or explicit instantiation in every translation unit in which such a use occurs; no diagnostic is required.

2 Each class template partial specialization is a distinct template and definitions shall be provided for the members of a template partial specialization (14.5.5.3).

3 [ Example:

```
template<class T1, class T2, int I> class A             { }; // #1
template<class T, int I>            class A<T, T*, I>   { }; // #2
template<class T1, class T2, int I> class A<T1*, T2, I> { }; // #3
template<class T>                   class A<int, T*, 5> { }; // #4
template<class T1, class T2, int I> class A<T1, T2*, I> { }; // #5
```

The first declaration declares the primary (unspecialized) class template. The second and subsequent declarations declare partial specializations of the primary template. — end example ]

4 The template parameters are specified in the angle bracket enclosed list that immediately follows the keyword **template**. For partial specializations, the template argument list is explicitly written immediately following the class template name. For primary templates, this list is implicitly described by the template parameter list. Specifically, the order of the template arguments is the sequence in which they appear in the template parameter list. [ Example: the template argument list for the primary template in the example above is <T1, T2, I>. — end example ] [ Note: The template argument list shall not be specified in the primary template declaration. For example,

```
template<class T1, class T2, int I> class A<T1, T2, I> { }; // error
```

— end note ]

5 A class template partial specialization may be declared or redeclared in any namespace scope in which its definition may be defined (14.5.1 and 14.5.2).

```
template<class T> struct A {
    struct C {
         template<class T2> struct B { };
    };
};

// partial specialization of A<T>::C::B<T2>
template<class T>
template<class T2>
struct A<T>::C::B<T2*> { };

A<short>::C::B<int*> absip; // uses partial specialization
```

6 Partial specialization declarations themselves are not found by name lookup. Rather, when the primary template name is used, any previously-declared partial specializations of the primary template are also considered. One consequence is that a *using-declaration* which refers to a class template does not restrict the set of partial specializations which may be found through the using-declaration.

部分特化声明本身不是不是通过名称查找找到的. 而是当基本模板名被使用的时候, 任何之前声明过的基本模板的部分特化都被考虑. 这导致的结果是引用一个类模板的using声明不会限制可能通过using声明发现的部分特化集合.

```
namespace N {
    template<class T1, class T2> class A { };   // primary template
}
using N::A;                                     // refers to the primary template
namespace N {
    template<class T> class A<T, T*> { };       // partial specialization
}
A<int,int*> a;                                  // uses the partial specialization, which is found through
                                                // the using declaration which refers to the primary template
```

7 A non-type argument is non-specialized if it is the name of a non-type parameter. All other non-type arguments are specialized.

```
template <std::size_t n>
struct S{};

template <std::size_t n>
struct S<n> {}; //non-specialization
```

8 Within the argument list of a class template partial specialization, the following restrictions apply:

— A partially specialized non-type argument expression shall not involve a template parameter of the partial specialization except when the argument expression is a simple identifier.

```
template <int I, int J> struct A {};
template <int I> struct A<I+5, I*2> {}; // error
template <int I, int J> struct B {};
template <int I> struct B<I, I> {};     // OK
```

— The type of a template parameter corresponding to a specialized non-type argument shall not be
dependent on a parameter of the specialization.

模板特化的非类型实参(下例`1`)所对应的模板形参的类型(下例`t`的类型`T`)不应该依赖于模板特化的形参(`Tp`).

```
template <class T, T t> struct C {};
template <class Tp> struct C<Tp, 1>;                  // error

template< int X, int (*array_ptr)[X] > class A {};
int array[5];
template< int X > class A<X,&array> { };            // error
```

— The argument list of the specialization shall not be identical to the implicit argument list of the primary
template.

— The template parameter list of a specialization shall not contain default template argument values.140
默认模板参数应该加在基本模板上

— An argument shall not contain an unexpanded parameter pack. If an argument is a pack expansion (14.5.3),
it shall be the last argument in the template argument list.

### 14.5.5.1 Matching of class template partial specializations

模板黑魔法的基础 - 模板特化匹配.
这段就是针对类模板的SFINAE规则的标准依据(针对函数模板的SFINAE是依赖于另外的规则,可能需要依赖于重载解析): 优先对考虑匹配特化版本, 如果匹配失败(这里指的是没有匹配的特化, 不是指导致ill-formed的情况) 即对基本模板实例化.

1 When a class template is used in a context that requires an instantiation of the class, it is necessary to determine whether the instantiation is to be generated using the primary template or one of the partial specializations. This is done by matching the template arguments of the class template specialization with the template argument lists of the partial specializations.

— If exactly one matching specialization is found, the instantiation is generated from that specialization.

— If more than one matching specialization is found, the partial order rules (14.5.5.2) are used to determine whether one of the specializations is more specialized than the others. If none of the specializations is more specialized than all of the other matching specializations, then the use of the class template is ambiguous and the program is ill-formed.

— If no matches are found, the instantiation is generated from the primary template.

2 A partial specialization matches a given actual template argument list if the template arguments of the partial specialization can be deduced from the actual template argument list (14.8.2).

```
template<class T1, class T2, int I> class A             { };    // #1
template<class T, int I>            class A<T, T*, I>   { };    // #2
template<class T1, class T2, int I> class A<T1*, T2, I> { };    // #3
template<class T>                   class A<int, T*, 5> { };    // #4
template<class T1, class T2, int I> class A<T1, T2*, I> { };    // #5

A<int, int, 1>   a1;    // uses #1
A<int, int*, 1>  a2;    // uses #2, T is int, I is 1
A<int, char*, 5> a3;    // uses #4, T is char
A<int, char*, 1> a4;    // uses #5, T1 is int, T2 is char, I is 1
A<int*, int*, 2> a5;    // ambiguous: matches #3 and #5
```

3 A non-type template argument can also be deduced from the value of an actual template argument of a non-type parameter of the primary template. [ Example: the declaration of a2 above. — end example ]

4 In a type name that refers to a class template specialization, (e.g., A<int, int, 1>) the argument list shall match the template parameter list of the primary template. The template arguments of a specialization are deduced from the arguments of the primary template.

模板特化的模板实参都是从基本模板的实参推导而来

### 14.5.5.2 Partial ordering of class template specializations

1 For two class template partial specializations, the first is at least as specialized as the second if, given the following rewrite to two function templates, the first function template is at least as specialized as the second according to the ordering rules for function templates (14.5.6.2):

— the first function template has the same template parameters as the first partial specialization and has a single function parameter whose type is a class template specialization with the template arguments of the first partial specialization, and

— the second function template has the same template parameters as the second partial specialization and has a single function parameter whose type is a class template specialization with the template arguments of the second partial specialization.

```
template<int I, int J, class T> class X { };
template<int I, int J>          class X<I, J, int> { }; // #1
template<int I>                 class X<I, I, int> { }; // #2

template<int I, int J> void f(X<I, J, int>);            // A
template<int I>        void f(X<I, I, int>);            // B
```

The partial specialization #2 is more specialized than the partial specialization #1 because the function template B is more specialized than the function template A according to the ordering rules for function templates.

### 14.5.5.3 Members of class template specializations

1 The template parameter list of a member of a class template partial specialization shall match the template parameter list of the class template partial specialization. The template argument list of a member of a class template partial specialization shall match the template argument list of the class template partial specialization. A class template specialization is a distinct template. The members of the class template partial specialization are unrelated to the members of the primary template. Class template partial specialization members that are used in a way that requires a definition shall be defined; the definitions of members of the primary template are never used as definitions for members of a class template partial specialization. An explicit specialization of a member of a class template partial specialization is declared in the same way as an explicit specialization of the primary template.

```
// primary template
template<class T, int I> struct A {
    void f();
};

template<class T, int I> void A<T,I>::f() { }
// class template partial specialization
template<class T> struct A<T,2> {
    void f();
    void g();
    void h();
};

// member of class template partial specialization
template<class T> void A<T,2>::g() { }
// explicit specialization
template<> void A<char,2>::h() { }

int main()
{
      A<char,0> a0;
      A<char,2> a2;
      a0.f();                 // OK, uses definition of primary template's member
      a2.g();                 // OK, uses definition of
                              // partial specialization's member
      a2.h();                 // OK, uses definition of
                              // explicit specialization's member
      a2.f();                 // ill-formed, no definition of f for A<T,2>
                              // the primary template is not used here
}
```

2 If a member template of a class template is partially specialized, the member template partial specializations are member templates of the enclosing class template; if the enclosing class template is instantiated (14.7.1, 14.7.2), a declaration for every member template partial specialization is also instantiated as part of creating the members of the class template specialization. If the primary member template is explicitly specialized for a given (implicit) specialization of the enclosing class template, the partial specializations of the member template are ignored for this specialization of the enclosing class template. If a partial specialization of the member template is explicitly specialized for a given (implicit) specialization of the enclosing class template, the primary member template and its other partial specializations are still considered for this specialization of the enclosing class template.

```
template<class T> struct A {
    template<class T2> struct B {};                      // #1
    template<class T2> struct B<T2*> {};                 // #2
};
template<> template<class T2> struct A<short>::B {}; // #3
A<char>::B<int*>  abcip;    // uses #2
A<short>::B<int*> absip;    // uses #3
A<char>::B<int>  abci;      // uses #1
```

### 14.5.6 Function templates

1 A function template defines an unbounded set of related functions.
[Example: a family of sort functions might be declared like this:

```
template<class T> class Array { };
template<class T> void sort(Array<T>&);
```

— end example ]

函数模板定义了一个同类函数的无限集合

2 A function template can be overloaded with other function templates and with normal (non-template) functions. A normal function is not related to a function template (i.e., it is never considered to be a specialization), even if it has the same name and type as a potentially generated function template specialization.141

### 14.5.6.1 Function template overloading

1 It is possible to overload function templates so that two different function template specializations have the same type.

重载的函数模板可以使两个不同的函数模板特化具有相同的类型.

```
// file1.c
template<class T>
void f(T*);
void g(int* p) {
    f(p); // calls f<int>(int*)
}

// file2.c
template<class T>
void f(T);
void h(int* p) {
    f(p); // calls f<int*>(int*)
}
```

```
// file.c
template<class T>
void f(T*);

template<class T>
void f(T);

void h(int* p) {
    f(p); // calls f<int*>(int*)
    f(*p); // calls f<int>(int)
}
```

2 Such specializations are distinct functions and do not violate the one definition rule (3.2).

3 The signature of a function template is defined in 1.3. The names of the template parameters are significant only for establishing the relationship between the template parameters and the rest of the signature. [ Note: Two distinct function templates may have identical function return types and function parameter lists, even if overload resolution alone cannot distinguish them.

模板形参的名称仅在建立模板形参与签名的其余部分之间的关系时才是重要的.

```
template<class T> void f();
template<int I> void f();   // OK: overloads the first template
                            // distinguishable(可区分) with an explicit template argument list
```

4 When an expression that references a template parameter is used in the function parameter list or the return type in the declaration of a function template, the expression that references the template parameter is part of the signature of the function template. This is necessary to permit a declaration of a function template in one translation unit to be linked with another declaration of the function template in another translation unit and, conversely, to ensure that function templates that are intended to be distinct are not linked with one another.

当一个引用模板形参的表达式被用在函数模板声明中的函数形参列表或返回类型中, 这个引用模板形参的表达式会被作为这个函数模板的签名的一部分.

```
template <int I, int J> A<I+J> f(A<I>, A<J>);   // #1
template <int K, int L> A<K+L> f(A<K>, A<L>);   // same as #1
template <int I, int J> A<I-J> f(A<I>, A<J>);   // different from #1
```

[ Note: Most expressions that use(使用) template parameters use(用作) non-type template parameters, but it is possible for an expression to reference a type parameter. For example, a template type parameter can be used in the **sizeof** operator. — end note ]

### 14.5.7 Alias templates

1 A *template-declaration* in which the *declaration* is an *alias-declaration* (Clause 7) declares the *identifier* to be a *alias template*. An alias template is a name for a family of types. The name of the alias template is a *template-name*.

2 When a *template-id* refers to the specialization of an alias template, it is equivalent to the associated type obtained by substitution of its *template-arguments* for the *template-parameters* in the *type-id* of the alias template.

当一个*template-id*引用到一个别名模板的特化, 它等价于通过它的模板实参替换别名模板的*type-id*中的模板形参得到的相应的类型.

```
template<class T> struct Alloc { /∗ ... ∗/ };
template<class T> using Vec = vector<T, Alloc<T>>;      //别名模板的特化,          template <class T> using X = T; 这就不属于别名模板的特化.
Vec<int> v; // same as vector<int, Alloc<int>> v;       //通过将模板实参int替换别名模板的type-id中的模板形参T得到的相应类型, 也就是 vector<int, Alloc<int>> ; type-id 指 <> 之间的部分.

template<class T>
void process(Vec<T>& v)
{ /∗ ... ∗/ }

template<class T>
void process(vector<T, Alloc<T>>& w)
{ /∗ ... ∗/ }   // error: redefinition

template<template<class> class TT>
       void f(TT<int>);

f(v); // error: Vec not deduced

template<template<class,class> class TT>
void g(TT<int, Alloc<int>>);
g(v);   // OK: TT = vector
```

3 The *type-id* in an alias template declaration shall not refer to the alias template being declared. The type produced by an alias template specialization shall not directly or indirectly make use of that specialization.

```
template <class T> struct A;
template <class T> using B = typename A<T>::U;
template <class T> struct A {
    typedef B<T> U;
};
B<short> b; // error: instantiation of B<short> uses own type via A<short>::U
```

### 14.6 Name resolution

1 Three kinds of names can be used within a template definition:

— The name of the template itself, and names declared within the template itself.

— Names dependent on a *template-parameter* (14.6.2).

— Names from scopes which are visible within the template definition.

2 A name used in a template declaration or definition and that is dependent on a *template-parameter* is assumed not to name a type unless the applicable name lookup finds a type name or the name is qualified by the keyword **typename**.

一个被用在模板声明或定义中并且依赖模板形参的名字被假定为不是命名一个类型除非应用名称查找发现它是一个类型名或这个名字以typename限定.

```
// no B declared here

class X;

template<class T> class Y {
    class Z;         // forward declaration of member class
    void f() {
        X* a1;      // declare pointer to X
        T* a2;      // declare pointer to T
        Y* a3;      // declare pointer to Y<T>
        Z* a4;      // declare pointer to Z
        typedef typename T::A TA;
        TA* a5;     // declare pointer to T’s A
        typename T::A* a6;  // declare pointer to T’s A
        T::A* a7;       // T::A is not a type name:
                        // multiply T::A by a7; ill-formed,
                        // no visible declaration of a7
        B* a8;          // B is not a typename:
                        // multiply B by a8; ill-formed,
                        // no visible declarations of B and a8
    }
};
```

3 When a *qualified-id* is intended to(打算) refer to a type that is not a member of the current instantiation (14.6.2.1) and its *nested-name-specifier* refers to a dependent type, it shall be prefixed by the keyword **typename**, forming a *typename-specifier*. If the *qualified-id* in a *typename-specifier* does not denote a type, the program is ill- formed.

*typename-specifier:
    typename nested-name-specifier identifier
    typename nested-name-specifier templateopt simple-template-id*

4 If a specialization of a template is instantiated for a set of template-arguments such that the *qualified-id* prefixed by **typename** does not denote a type, the specialization is ill-formed. The usual qualified name lookup (3.4.3) is used to find the qualified-id even in the presence of **typename**.

如果为一组模板实参实例化一个模板特化("模板特化"指"模板实例"更好理解)使得带有**typename**前缀的*qualified-id*不代表一个类型, 那么这个特化是ill-formed.

> 在[14.1/2]中提到 “typename followed by a qualified-id denotes the type in a non-type 137parameter-declaration”, 这句话指的是实例化之前以typename作为限定id前缀, 而本段定义说的是实例化之后以typename作为限定id前缀, 并不冲突。
> 

```
struct A {
    struct X { };   //被`int X;`隐藏, 按照通常的名称查找规则.
    int X;
};
struct B {
    struct X { };
};
template<class T> void f(T t) {
    typename T::X x;
}
void foo() {
    A a;
    B b;
    f(b); // OK: T::X refers to B::X
    f(a); // error: T::X refers to the data member A::X not the struct A::X
}
```

5 A qualified name used as the name in a *mem-initializer-id*, a *base-specifier*, or an *elaborated-type-specifier* is implicitly assumed to(隐式假定为) name a type, without the use of the **typename** keyword. In a *nested-name-specifier* that immediately contains a *nested-name-specifier* that depends on a template parameter, the identifier or *simple-template-id* is implicitly assumed to name a type, without the use of the **typename** keyword. [ Note: The **typename** keyword is not permitted by the syntax of these constructs. — end note ]

6 If, for a given set of template arguments, a specialization of a template is instantiated that refers to a *qualified-id* that denotes a type, and the *qualified-id* refers to a member of an unknown specialization, the qualified-id shall either be prefixed by typename or shall be used in a context in which it implicitly names a type as described above.

如果对于一个给定的模板实参集合, 引用一个代表类型的限定Id的模板特化被实例化("模板特化"指"模板实例"更好理解), 并且这个限定id引用一个未知特化的成员, 那么这个限定id应该以typename作为前缀或者被用在上面描述的那些隐式作为类型名的上下文.

```
template <class T> void f(int i) {
    T::x * i;   // T::x must not be a type          //要么以typename限定, 要么用在隐式作为类型名的上下文, 否则, 如果它代表类型, 则ill-formed.
}

struct Foo {
      typedef int x;
};

struct Bar {
  static int const x = 5;
};

int main() {
    f<Bar>(1);  // OK
    f<Foo>(1);  // error: Foo::x is a type
}

```

7 Within the definition of a class template or within the definition of a member of a class template following the *declarator-id*, the keyword **typename** is not required when referring to the name of a previously declared member of the class template that declares a type. [ Note: such names can be found using unqualified name lookup (3.4.1), class member lookup (3.4.3.1) into the current instantiation (14.6.2.1), or class member access expression lookup(3.4.5) when the type of the object expression is the current instantiation(14.6.2.2). —end note ]

在类模板或类模板的成员定义的*declarator-id*之后, 当引用之前被声明为类型的类模板成员名时, 不要求**typename**关键字.

```
template<class T> struct A {
    typedef int B;
    B b; // OK, no typename required
};
```

8 Knowing which names are type names allows the syntax of every template definition to be checked. No diagnostic shall be issued for a template definition for which a valid specialization can be generated. If no valid specialization can be generated for a template definition, and that template is not instantiated, the template definition is ill-formed, no diagnostic required. If every valid specialization of a variadic template requires an empty template parameter pack, the template definition is ill-formed, no diagnostic required. If a type used in a non-dependent name is incomplete at the point at which a template is defined but is complete at the point at which an instantiation is done, and if the completeness of that type affects whether or not the program is well-formed or affects the semantics of the program, the program is ill-formed; no diagnostic is required. [ Note: If a template is instantiated, errors will be diagnosed according to the other rules in this Standard. Exactly when these errors are diagnosed is a quality of implementation issue. — end note ]

知道哪些名字是类型名可以检查每个模板定义语法. 对于可以生成有效特化的模板定义不需要发出诊断. 如果一个模板定义没有有效的特化(这里的特化表示实例)可以生成, 这个模板定义是ill-formed, 不要求诊断. 如果每个可变(参数)模板的有效的特化(这里的特化表示实例)要求一个空模板参数包, 这个模板定义是ill-formed, 不要求诊断. 如果一个被用在非依赖名称的类型在模板定义的点是不完整类型, 但是在实例化的点是完整的, 并且如果这个类型的完整性影响程序的well-formed与否, 或影响程序的语义, 这个程序是ill-formed, 不要求诊断.

```
int j;
template<class T> class X {
    void f(T t, int i, char* p) {
        t = i;  // diagnosed if X::f is instantiated
                // and the assignment to t is an error

        p = i;  // may be diagnosed even if X::f is
                // not instantiated

        p = j;  // may be diagnosed even if X::f is
                // not instantiated
    }
    void g(T t) {
        +;  // may be diagnosed even if X::g is
            // not instantiated
    }

    template<class... T> struct A {
        void operator++(int, T... t);               // error: too many parameters
    };
    template<class... T> union X : T... { };        // error: union with base class
    template<class... T> struct A : T..., T... { }; // error: duplicate base class
};
```

9 When looking for the declaration of a name used in a template definition, the usual lookup rules (3.4.1, 3.4.2) are used for non-dependent names. The lookup of names dependent on the template parameters is postponed(被推迟) until the actual template argument is known (14.6.2).

```
#include <iostream>
using namespace std;
template<class T> class Set {
    T* p;
    int cnt;
public:
    Set();
    Set<T>(const Set<T>&);
    void printall() {
        for (int i = 0; i<cnt; i++)
	        cout << p[i] << ’\\n’;
    }
};
```

in the example, i is the local variable i declared in printall, cnt is the member cnt declared in Set, and cout is the standard output stream declared in iostream. However, not every declaration can be found this way; the resolution of some names must be postponed until the actual template-arguments are known. For example, even though the name operator<< is known within the definition of printall() and a declaration of it can be found in <iostream>, the actual declaration of operator<< needed to print p[i] cannot be known until it is known what type T is (14.6.2). — end example ]

10 If a name does not depend on a *template-parameter* (as defined in 14.6.2), a declaration (or set of declarations) for that name shall be in scope at the point where the name appears in the template definition; the name is bound to the declaration (or declarations) found at that point and this binding is not affected by declarations that are visible at the point of instantiation.

如果一个名字不依赖于模板形参, 那么这个名字的声明应该在它出现在模板定义中的点是可见的, 这个名字被绑定到在那个点发现的声明, 这个绑定不受实例化时可见的声明的影响.

对于依赖模板形参的名字查找描述在[14.6.2/1].

```
int i = 0;
template <typename T> void ff(){
    ++i;            //1            //绑定到 ::i;
}

void g(){
    int i = 1;
    ff<int>();
};
```

```
void f(char);

template<class T> void g(T t) {
  f(1);             // f(char)
  f(T(1));          // dependent
  f(t);             // dependent
  dd++;             // not dependent
                    // error: declaration for dd not found
}

enum E { e };
void f(E);

double dd;
void h() {
  g(e);             // will cause one call of f(char) followed
                    // by two calls of f(E)
  g('a');           // will cause three calls of f(char)
}
```

11 [Note: For purposes of name lookup, default arguments of function templates and default arguments of member functions of class templates are considered definitions (14.5). — end note]

### 14.6.1 Locally declared names

1 Like normal (non-template) classes, class templates have an *injected-class-name* (Clause 9). The *injected-class-name* can be used as a *template-name* or a  *type-name*. When it is used with a *template-argument-list*, as a *template-argument* for a template *template-parameter*, or as the final identifier in the *elaborated-type-specifier* of a friend class template declaration, it refers to the class template itself. Otherwise, it is equivalent to the *template-name* followed by the *template-parameters* of the class template enclosed in <>.

在类模板中, 只有当 *injected-class-name* 被用作模板的模板参数或作为友元类模板声明时, *injected-class-name* 被视为类模板本身(即不带<type-id>), 除此之外,它等同于 `类模板名<模板形参列表>`

2 Within the scope of a class template specialization or partial specialization, when the **injected-class-name** is used as a **type-name**, it is equivalent to the **template-name** followed by the **template-arguments** of the class template specialization or partial specialization enclosed in <>.

```cpp
template<template<class> class T> class A { };
template<class T> class Y;
template<> class Y<int> {
	Y* p;                                 // meaning Y<int>
	Y<char>* q;                           // meaning Y<char>
	A<Y>* a;                              // meaning A<::Y>
	class B {
		template<class> friend class Y;     // meaning ::Y
	};
};
```

3 The *injected-class-name* of a class template or class template specialization can be used either as a template-name or a type-name wherever it is in scope.

```cpp
template <class T> struct Base {
	Base* p;
};
template <class T> struct Derived: public Base<T> {
	typename Derived::Base* p; // meaning Derived::Base<T>
};
template<class T, template<class> class U = T::template Base> struct Third { };
Third<Base<int> > t; // OK: default argument uses injected-class-name as a template
```

4 A lookup that finds an *injected-class-name* (10.2) can result in an ambiguity in certain cases (for example, if it is found in more than one base class). If all of the *injected-class-names* that are found refer to specializations of the same class template, and if the name is used as a template-name, the reference refers to the class template itself and not a specialization thereof, and is not ambiguous.

```cpp
template <class T> struct Base { };
template <class T> struct Derived: Base<int>, Base<char> {
	typename Derived::Base b;         // error: ambiguous
	typename Derived::Base<double> d; // OK         //这段话说的指 Derived::Base 指向类模板本身, 也就是它还是个类模板, 而不是任何特化, 所以不能直接定义变量, 而应该先实例化为具体类型.
};
```

5 When the normal name of the template (i.e., the name from the enclosing scope, not the *injected-class-name*) is used, it always refers to the class template itself and not a specialization of the template.

```cpp
template<class T> class X {
	X* p;           // meaning X<T>
	X<T>* p2;
	X<int>* p3;
	::X* p4; // error: missing template argument list
				 // ::X does not refer to the *injected-class-name*
				// ::X引用类模板本身, 所以是一个类模板的名字, 不能直接定义变量.
};
```

6 A *template-parameter* shall not be redeclared within its scope (including nested scopes). A template-parameter shall not have the same name as the template name.

```cpp
template <class T, int i>
class Y
{
	int T; // error: template-parameter redeclared
	void f()
	{
		char T; // error: template-parameter redeclared
	}
};

template <class X>
class X; // error: template-parameter redeclared
```

7 In the definition of a member of a class template that appears outside of the class template definition, the name of a member of the class template hides the name of a **template-parameter** of any enclosing class templates (but not a template-parameter of the member if the member is a class or function template).

在类模板之外定义的成员定义中, 类模板的成员名将隐藏它的封闭类模板的模板形参名.

```cpp
template <class T>
struct A
{
	struct B
	{ 
	};

	typedef void C;

	void f();

	template <class U>
	void g(U);
};

template <class B>
void A<B>::f()
{
	B b; // A’s B, not the template parameter            //A是f()的封闭类模板, B是A的类模板形参名, 所以A::B将隐藏形参B.
}

template <class B>
template <class C>
void A<B>::g(C)
{
	B b; // A’s B, not the template parameter
	C c; // the template parameter C, not A’s C         //C是g自己的模板形参名, 不是A的模板形参名.
}
```

8 In the definition of a member of a class template that appears outside of the namespace containing the class template definition, the name of a *template-parameter* hides the name of a member of this namespace.

```cpp
namespace N
{
	class C
	{
	};

	template <class T>
	class B
	{
		void f(T);
	};
}

template <class C>
void N::B<C>::f(C)
{
	C b; // C is the template parameter, not N::C
}
```

9 In the definition of a class template or in the definition of a member of such a template that appears outside of the template definition, for each base class which does not depend on a *template-parameter* (14.6.2), if the name of the base class or the name of a member of the base class is the same as the name of a template-parameter, the base class name or member name hides the template-parameter name (3.3.10).

```cpp
struct A
{
	struct B
	{
	};

	int a;
	int Y;
};

template <class B, class a>
struct X : A
{
	B b; // A’s B
	a b; // error: A’s a isn’t a type name
};
```

### 14.6.2 Dependent names

1 Inside a template, some constructs have semantics which may differ from one instantiation to another. Such a construct *depends* on the template parameters. In particular, types and expressions may depend on the type and/or value of template parameters (as determined by the template arguments) and this determines the context for name lookup for certain names. Expressions may be *type-dependent* (on the type of a template parameter) or *value-dependent* (on the value of a non-type template parameter). In an expression of the form:

*postfix-expression ( expression-listopt )*

where the *postfix-expression* is an *id-expression*, the *id-expression* denotes a *dependent name* if

— any of the expressions in the expression-list is a pack expansion (14.5.3),

— any of the expressions in the expression-list is a type-dependent expression (14.6.2.2), or

— if the *unqualified-id* of the *id-expression* is a *template-id* in which any of the template arguments depends on a template parameter.

If an operand of an operator is a type-dependent expression, the operator also denotes a dependent name. Such names are unbound and are looked up at the point of the template instantiation (14.6.4.1) in both the context of the template definition and the context of the point of instantiation.

2 [ Example:

```cpp
template <class T>
struct X : B<T>
{
	typename T::A *pa;
	void f(B<T> *pb)
	{
		static int i = B<T>::i;
		pb->j++;
	}
};
```

the base class name B<T>, the type name T::A, the names B<T>::i and pb->j explicitly depend on the template-parameter. —end example]

3 In the definition of a class or class template, if a base class depends on a **template-parameter**, the base class scope is not examined during unqualified name lookup either at the point of definition of the class template or member or during an instantiation of the class template or member.

```cpp
typedef double A;
template<class T> class B {
	typedef int A;
};
template<class T> struct X : B<T> {
	A a; // a has type double
};
```

The type name A in the definition of X<T> binds to the typedef name defined in the global namespace scope, not to the typedef name defined in the base class B<T>. — end example ]

```
struct A {
    struct B { /∗ ... ∗/ };
    int a;
    int Y;
    struct C {};
};
int a;
template<class T> struct Y : T {
    struct B { /∗ ... ∗/ };
    B b;                      // The B defined in Y
    void f(int i) { a = i; }  // ::a
    Y* p;                     // Y<T>
    C c;                      // error.
};
Y<A> ya;
```

The members A::B, A::a, and A::Y of the template argument A do not affect the binding of names in Y<A>. — end example ]

### 14.6.2.1 Dependent types

1 A name refers to the *current instantiation* if it is ...

2 The template argument list of a primary template is a template argument list in which the nth template argument has the value of the nth template parameter of the class template. If the nth template parameter is a template parameter pack (14.5.3), the nth template argument is a pack expansion (14.5.3) whose pattern is the name of the template parameter pack.

3 A template argument that is equivalent to a template parameter (i.e., has the same constant value or the same type as the template parameter) can be used in place of that template parameter in a reference to the current instantiation. In the case of a non-type template argument, the argument must have been given the value of the template parameter and not an expression in which the template parameter appears as a subexpression.

可以使用与模板形参等同的模板实参来代替模板形参引用当前实例

```
template <class T> class A {
  A* p1;                        // A is the current instantiation
  A<T>* p2;                     // A<T> is the current instantiation
  A<T*> p3;                     // A<T*> is not the current instantiation
  ::A<T>* p4;                   // ::A<T> is the current instantiation
  class B {
    B* p1;                      // B is the current instantiation
    A<T>::B* p2;                // A<T>::B is the current instantiation
    typename A<T*>::B* p3;      // A<T*>::B is not the current instantiation
  };
};
template <class T> class A<T*> {
  A<T*>* p1;                    // A<T*> is the current instantiation
  A<T>* p2;                     // A<T> is not the current instantiation
};
template <class T1, class T2, int I> struct B {
  B<T1, T2, I>* b1;             // refers to the current instantiation
  B<T2, T1, I>* b2;             // not the current instantiation
  typedef T1 my_T1;
  static const int my_I = I;
  static const int my_I2 = I+0;
  static const int my_I3 = my_I;
  B<my_T1, T2, my_I>* b3;       // refers to the current instantiation  // my_T1 等价于 T1, 使用两者作为第一个模板参数都可以引用当前实例.
  B<my_T1, T2, my_I2>* b4;      // not the current instantiation
  B<my_T1, T2, my_I3>* b5;      // refers to the current instantiation
};
```

4 A name is a **member of the current instantiation** if it is ...

5 A name is a member of an unknown specialization if it is ...

6 If a *qualified-id* in which the *nested-name-specifier* refers to the current instantiation is not a member of the current instantiation or a member of an unknown specialization, the program is ill-formed even if the template containing the **qualified-id** is not instantiated; no diagnostic required. Similarly, if the **id-expression** in a class member access expression for which the type of the object expression is the current instantiation does not refer to a member of the current instantiation or a member of an unknown specialization, the program is ill-formed even if the template containing the member access expression is not instantiated; no diagnostic required.

```
template<class T> class A {
    typedef int type;
    void f() {
        A<T>::type i;           // OK: refers to a member of the current instantiation
        typename A<T>::other j; // error: neither a member of the current instantiation nor
                                // a member of an unknown specialization
    }
};
```

7 If, for a given set of template arguments, a specialization of a template is instantiated that refers to a member of the current instantiation with a *qualified-id* or class member access expression, the name in the **qualified-id** or class member access expression is looked up in the template instantiation context. If the result of this lookup differs from the result of name lookup in the template definition context, name lookup is ambiguous. [ Note: the result of name lookup differs only when the member of the current instantiation was found in a non-dependent base class of the current instantiation and a member with the same name is also introduced by the substitution for a dependent base class of the current instantiation. — end note ]

8 A type is dependent if it is

— a template parameter,

— a member of an unknown specialization,

— a nested class or enumeration that is a member of the current instantiation,

— a cv-qualified type where the cv-unqualified type is dependent,

— a compound type constructed from any dependent type,

— an array type constructed from any dependent type or whose size is specified by a constant expression that is value-dependent,

— a simple-template-id in which either the template name is a template parameter or any of the template arguments is a dependent type or an expression that is type-dependent or value-dependent, or

— denoted by decltype(expression), where expression is type-dependent (14.6.2.2).

9 [ Note: Because typedefs do not introduce new types, but instead simply refer to other types, a name that refers to a typedef that is a member of the current instantiation is dependent only if the type referred to is dependent. —endnote]

### 14.6.2.2 Type-dependent expressions

1 Except as described below, an expression is type-dependent if any subexpression is type-dependent.

2 **this** is type-dependent if the class type of the enclosing member function is dependent (14.6.2.1).

3 An *id-expression* is type-dependent if it contains

— an *identifier* associated by name lookup with one or more declarations declared with a dependent type,

— a *template-id* that is dependent,

— a *conversion-function-id* that specifies a dependent type, or

— a *nested-name-specifier* or a *qualified-id* that names a member of an unknown specialization;

or if it names a static data member of the current instantiation that has type “array of unknown bound of T” for some T (14.5.1.3). Expressions of the following forms are type-dependent only if the type specified by the *type-id*, *simple-type-specifier* or *new-type-id* is dependent, even if any subexpression is type-dependent:

*simple-type-specifier ( expression-listopt )
::opt new new-placementopt new-type-id new-initializeropt
::opt new new-placementopt ( type-id ) new-initializeropt
dynamic_cast < type-id > ( expression )
static_cast < type-id > ( expression )
const_cast < type-id > ( expression )
reinterpret_cast < type-id > ( expression )
( type-id ) cast-expression*

4 Expressions of the following forms are never type-dependent (because the type of the expression cannot be dependent):

*literal
postfix-expression . pseudo-destructor-name
postfix-expression -> pseudo-destructor-name
sizeof unary-expression
sizeof ( type-id )
sizeof ... ( identifier )
alignof ( type-id )
typeid ( expression )
typeid ( type-id )
::opt delete cast-expression
::opt delete [ ] cast-expression
throw assignment-expressionopt
noexcept ( expression )*

5 A class member access expression (5.2.5) is type-dependent if the expression refers to a member of the current instantiation and the type of the referenced member is dependent, or the class member access expression refers to a member of an unknown specialization. [Note: In an expression of the form x.y or xp->y the type of the expression is usually the type of the member y of the class of x (or the class pointed to by xp). However, if x or xp refers to a dependent type that is not the current instantiation, the type of y is always dependent. If x or xp refers to a non-dependent type or refers to the current instantiation, the type of y is the type of the class member access expression. — end note ]

### 14.6.2.3 Value-dependent expressions

1 Except as described below, a constant expression is value-dependent if any subexpression is value-dependent.

2 An identifier is value-dependent if it is:

— a name declared with a dependent type,

— the name of a non-type template parameter,

— a constant with literal type and is initialized with an expression that is value-dependent.

Expressions of the following form are value-dependent if the *unary-expression* or expression is type-dependent or the *type-id* is dependent:

*sizeof unary-expression
sizeof ( type-id )
typeid ( expression )
typeid ( type-id )
alignof ( type-id )
noexcept ( expression )*

3 Expressions of the following form are value-dependent if either the *type-id* or *simple-type-specifier* is dependent or the expression or cast-expression is value-dependent:

*simple-type-specifier ( expression-listopt )
static_cast < type-id > ( expression )
const_cast < type-id > ( expression )
reinterpret_cast < type-id > ( expression )
( type-id ) cast-expression*

4 Expressions of the following form are value-dependent:

*sizeof ... ( identifier )*

5 An *id-expression* is value-dependent if it names a member of an unknown specialization.

### 14.6.2.4 Dependent template arguments

1 A type *template-argument* is dependent if the type it specifies is dependent.

2 A non-type *template-argument* is dependent if its type is dependent or the constant expression it specifies is value-dependent.

3 Furthermore, a non-type *template-argument* is dependent if the corresponding non-type *template-parameter* is of reference or pointer type and the *template-argument* designates or points to a member of the current instantiation or a member of a dependent type.

4 A template *template-argument* is dependent if it names a *template-parameter* or is a *qualified-id* that refers to a member of an unknown specialization.

### 14.6.3 Non-dependent names

1 Non-dependent names used in a template definition are found using the usual name lookup and bound at the point they are used.

```cpp
void g(double);
void h();

template<class T> class Z {
public:
  void f() {
    g(1);           // calls g(double)
    h++;            // ill-formed: cannot increment function;
                    // this could be diagnosed either here or at the point of instantiation
  }
};
void g(int);        // not in scope at the point of the template definition, not considered for the call g(1)
```

### 14.6.4 Dependent name resolution

1 In resolving dependent names, names from the following sources are considered:

— Declarations that are visible at the point of definition of the template.

— Declarations from namespaces associated with the types of the function arguments both from the instantiation context (14.6.4.1) and from the definition context.

对依赖名称的解析, 以下来源会被考虑:
- 在模板定义处可见的声明
- 来自于实例化的上下文和来自于定义的上下文中的函数实参类型所关联的命名空间中的声明

### 14.6.4.1 Point of instantiation

1 For a function template specialization, a member function template specialization, or a specialization for a member function or static data member of a class template, if the specialization is implicitly instantiated because it is referenced from within another template specialization and the context from which it is referenced depends on a template parameter, the point of instantiation of the specialization is the point of instantiation of the enclosing specialization. Otherwise, the point of instantiation for such a specialization
immediately follows the namespace scope declaration or definition that refers to the specialization.

对于一个函数模板特化, 一个成员函数模板特化, 或一个类模板特化的成员函数或静态数据成员, 如果这个特化由于被其他的模板特化引用而被隐式的实例化并且引用它的上下文依赖于一个模板形参,那么这个特化的实例化点是封闭它的特化的实例化点. 否则, 这样一个特化的实例化点紧跟在引用这个特化的声明或定义所在的命名空间作用域中的后面.

> [http://bbs.csdn.net/topics/390191104](http://bbs.csdn.net/topics/390191104)
> 

2 If a function template or member function of a class template is called in a way which uses the definition of a default argument of that function template or member function, the point of instantiation of the default argument is the point of instantiation of the function template or member function specialization.

3 For a class template specialization, a class member template specialization, or a specialization for a class member of a class template, if the specialization is implicitly instantiated because it is referenced from within another template specialization, if the context from which the specialization is referenced depends on a template parameter, and if the specialization is not instantiated previous to the instantiation of the enclosing template, the point of instantiation is immediately before the point of instantiation of the enclosing template. Otherwise, the point of instantiation for such a specialization immediately precedes the namespace scope declaration or definition that refers to the specialization.

4 If a virtual function is implicitly instantiated, its point of instantiation is immediately following the point of instantiation of its enclosing class template specialization.

5 An explicit instantiation definition is an instantiation point for the specialization or specializations specified by the explicit instantiation.

6 The instantiation context of an expression that depends on the template arguments is the set of declarations with external linkage declared prior to the point of instantiation of the template specialization in the same translation unit.

7 A specialization for a function template, a member function template, or of a member function or static data member of a class template may have multiple points of instantiations within a translation unit, and in addition to the points of instantiation described above, for any such specialization that has a point of instantiation within the translation unit, the end of the translation unit is also considered a point of instantiation. A specialization for a class template has at most one point of instantiation within a translation unit. A specialization for any template may have points of instantiation in multiple translation units. If two different points of instantiation give a template specialization different meanings according to the one definition rule (3.2), the program is ill-formed, no diagnostic required.

### 14.6.4.2 Candidate functions

1 For a function call that depends on a template parameter, the candidate functions are found using the usual lookup rules (3.4.1, 3.4.2, 3.4.3) except that:

— For the part of the lookup using unqualified name lookup (3.4.1) or qualified name lookup (3.4.3), only function declarations from the template definition context are found.

— For the part of the lookup using associated namespaces (3.4.2), only function declarations found in either the template definition context or the template instantiation context are found.

If the function name is an unqualified-id and the call would be ill-formed or would find a better match had the lookup within the associated namespaces considered all the function declarations with external linkage introduced in those namespaces in all translation units, not just considering those declarations found in the template definition and template instantiation contexts, then the program has undefined behavior.

### 14.6.5 Friend names declared within a class template

### 14.7 Template instantiation and specialization

1 The act of instantiating a function, a class, a member of a class template or a member template is referred to as *template instantiation*.

2 A function instantiated from a function template is called an instantiated function. A class instantiated from a class template is called an instantiated class. A member function, a member class, a member enumeration, or a static data member of a class template instantiated from the member definition of the class template is called, respectively, an instantiated member function, member class, member enumeration, or static data member. A member function instantiated from a member function template is called an instantiated member function. A member class instantiated from a member class template is called an instantiated member class.

3 An explicit specialization may be declared for a function template, a class template, a member of a class template or a member template. An explicit specialization declaration is introduced by **template<>**. In an explicit specialization declaration for a class template, a member of a class template or a class member template, the name of the class that is explicitly specialized shall be a *simple-template-id*. In the explicit specialization declaration for a function template or a member function template, the name of the function or member function explicitly specialized may be a *template-id*.

显式特化是指全特化, 对于(成员)函数模板, 如果模板模板实参能通过函数实参推导出来, 则可以省略type-id, 否则, (成员)函数模板的名字应该是template-id

```
template<class T = int> struct A {
    static int x;
};
template<class U> void g(U) { }
template<> struct A<double> { };    // specialize for T == double
template<> struct A<> { };          // specialize for T == int
template<> void g(char) { }         // specialize for U == char
                                    // U is deduced from the parameter type
template<> void g<int>(int) { }     // specialize for U == int
template<> int A<char>::x = 0;      // specialize for T == char
template<class T = int> struct B {
    static int x;
};
template<> int B<>::x = 1;          // specialize for T == int
```

4 An instantiated template specialization can be either implicitly instantiated (14.7.1) for a given argument list or be explicitly instantiated (14.7.2). A specialization is a class, function, or class member that is either instantiated or explicitly specialized (14.7.3).

> 特化("specialization") 这个概念泛指指的被显式指定或实例化产生的类, 函数, 类成员. 通常可能会遇到的"模板实例"其实就是指"实例化产生的类, 函数, 类成员", 但是在标准中叫"特化"。

5 For a given template and a given set of *template-arguments*,

— an explicit instantiation definition shall appear at most once in a program,

— an explicit specialization shall be defined at most once in a program (according to 3.2), and

— both an explicit instantiation and a declaration of an explicit specialization shall not appear in a program unless the explicit instantiation follows a declaration of the explicit specialization.

An implementation is not required to diagnose a violation of this rule.

6 Each class template specialization instantiated from a template has its own copy of any static members.

```
template<class T> class X {
    static T s;
};
template<class T> T X<T>::s = 0;
X<int> aa;
X<char*> bb;
```

X<int> has a static member s of type int and X<char*> has a static member s of type char*.

### 14.7.1 Implicit instantiation

1 Unless a class template specialization has been explicitly instantiated (14.7.2) or explicitly specialized (14.7.3), the class template specialization is implicitly instantiated when the specialization is referenced in a context that requires a completely-defined object type or when the completeness of the class type affects the semantics of the program. The implicit instantiation of a class template specialization causes the implicit instantiation of the declarations, but not of the definitions or default arguments, of the class member functions, member classes, scoped member enumerations, static data members and member templates; and it causes the implicit instantiation of the definitions of unscoped member enumerations and member anonymous unions. However, for the purpose of determining whether an instantiated redeclaration of a member is valid according to 9.2, a declaration that corresponds to a definition in the template is considered to be a definition.

除非一个类模板特化已经被显式实例化或显式特化,否则当这个模板特化被引用在一个要求完全定义的对象类型或者当类类型的完整性影响程序语义时, 这个类模板特化会被隐式的实例化.

```
template<class T, class U>
struct Outer {
    template<class X, class Y> struct Inner;
    template<class Y> struct Inner<T, Y>; // #1a
    template<class Y> struct Inner<T, Y> { }; // #1b; OK: valid redeclaration of #1a
    template<class Y> struct Inner<U, Y> { }; // #2
};
Outer<int, int> outer; // error at #2
```

Outer<int, int>::Inner<int, Y> is redeclared at #1b. (It is not defined but noted as being associated with a definition in Outer<T, U>.) #2 is also a redeclaration of #1a. It is noted as associated with a definition, so it is an invalid redeclaration of the same partial specialization.

2 Unless a member of a class template or a member template has been explicitly instantiated or explicitly specialized, the specialization of the member is implicitly instantiated when the specialization is referenced in a context that requires the member definition to exist; in particular, the initialization (and any associated side-effects) of a static data member does not occur unless the static data member is itself used in a way that requires the definition of the static data member to exist.

3 Unless a function template specialization has been explicitly instantiated or explicitly specialized, the function template specialization is implicitly instantiated when the specialization is referenced in a context that requires a function definition to exist. Unless a call is to a function template explicit specialization or to a member function of an explicitly specialized class template, a default argument for a function template or a member function of a class template is implicitly instantiated when the function is called in a context that requires the value of the default argument.

4 [Example:

```cpp
template<class T> struct Z {
    void f();
    void g();
};
void h() {
    Z<int> a;       // instantiation of class Z<int> required
    Z<char>* p;     // instantiation of class Z<char> not required
    Z<double>* q;   // instantiation of class Z<double> not required
    a.f();          // instantiation of Z<int>::f() required
    p->g();         // instantiation of class Z<char> required, and
                    // instantiation of Z<char>::g() required
}
```

Nothing in this example requires class Z<double>, Z<int>::g(), or Z<char>::f() to be implicitly instantiated. —end example ]

5 A class template specialization is implicitly instantiated if the class type is used in a context that requires a completely-defined object type or if the completeness of the class type might affect the semantics of the program. [Note: In particular, if the semantics of an expression depend on the member or base class lists of a class template specialization, the class template specialization is implicitly generated. For instance, deleting a pointer to class type depends on whether or not the class declares a destructor, and conversion between pointer to class types depends on the inheritance relationship between the two classes involved. —end note ]

```
template<class T> class B { /∗ ... ∗/ };
template<class T> class D : public B<T> { /∗ ... ∗/ };
void f(void*);
void f(B<int>*);
void g(D<int>* p, D<char>* pp, D<double>* ppp) {
    f(p);               // instantiation of D<int> required: call f(B<int>*)
    B<char>* q = pp;    // instantiation of D<char> required:
                        // convert D<char>* to B<char>*
    delete ppp;         // instantiation of D<double> required
}
```

6 If the overload resolution process can determine the correct function to call without instantiating a class template definition, it is unspecified whether that instantiation actually takes place.

```
template <class T> struct S {
    operator int();
};
void f(int);
void f(S<int>&);
void f(S<float>);
void g(S<int>& sr) {
    f(sr);  // instantiation of S<int> allowed but not required
            // instantiation of S<float> allowed but not required
};
```

7 If an implicit instantiation of a class template specialization is required and the template is declared but not defined, the program is ill-formed.

```
template<class T> class X;
X<char> ch;     // error: definition of X required
```

8 The implicit instantiation of a class template does not cause any static data members of that class to be implicitly instantiated.

9 If a function template or a member function template specialization is used in a way that involves overload resolution, a declaration of the specialization is implicitly instantiated (14.8.3).

10 An implementation shall not implicitly instantiate a function template, a member template, a non-virtual member function, a member class, or a static data member of a class template that does not require instantiation. It is unspecified whether or not an implementation implicitly instantiates a virtual member function of a class template if the virtual member function would not otherwise be instantiated. The use of a template specialization in a default argument shall not cause the template to be implicitly instantiated except that a class template may be instantiated where its complete type is needed to determine the correctness of the default argument. The use of a default argument in a function call causes specializations in the default argument to be implicitly instantiated.

11 Implicitly instantiated class and function template specializations are placed in the namespace where the template is defined. Implicitly instantiated specializations for members of a class template are placed in the namespace where the enclosing class template is defined. Implicitly instantiated member templates are placed in the namespace where the enclosing class or class template is defined.

```
namespace N {
    template<class T> class List {
    public:
        T* get();
    };
}
template<class K, class V> class Map {
    public:
        N::List<V> lt;
        V get(K);
};
void g(Map<const char*,int>& m) {
    int i = m.get("Nicholas");
}
```

a call of lt.get() from Map<const char*,int>::get() would place List<int>::get() in the namespace N rather than in the global namespace.

12 If a function template **f** is called in a way that requires a default argument to be used, the dependent names are looked up, the semantics constraints are checked, and the instantiation of any template used in the default argument is done as if the default argument had been an initializer used in a function template specialization with the same scope, the same template parameters and the same access as that of the function template f used at that point. This analysis is called *default argument instantiation*. The instantiated default argument is then used as the argument of f.

13 Each default argument is instantiated independently.

```cpp
template<class T> void f(T x, T y = ydef(T()), T z = zdef(T()));
class A { };
A zdef(A);

void g(A a, A b, A c) {
    f(a, b, c);     // no default argument instantiation
    f(a, b);    // default argument z = zdef(T()) instantiated
    f(a);       // ill-formed; ydef is not declared
}
```

14 [Note: 14.6.4.1 defines the point of instantiation of a template specialization. —end note]

15 There is an implementation-defined quantity that specifies the limit on the total depth of recursive instantiations, which could involve more than one template. The result of an infinite recursion in instantiation is  undefined.

```
template<class T> class X {
    X<T>* p;    // OK
    X<T*> a;    // implicit generation of X<T> requires
                // the implicit instantiation of X<T*> which requires
                // the implicit instantiation of X<T**> which ...
};
```

### 14.7.2 Explicit instantiation

1 A class, a function or member template specialization can be explicitly instantiated from its template. A member function, member class or static data member of a class template can be explicitly instantiated from the member definition associated with its class template. An explicit instantiation of a function template or member function of a class template shall not use the **inline** or **constexpr** specifiers.

2 The syntax for explicit instantiation is:

*explicit-instantiation:
    extern opt template declaration*

There are two forms of explicit instantiation: an explicit instantiation definition and an explicit instantiation declaration. An explicit instantiation declaration begins with the **extern** keyword.

3 If the explicit instantiation is for a class or member class, the *elaborated-type-specifier* in the declaration shall include a *simple-template-id*. If the explicit instantiation is for a function or member function, the unqualified-id in the declaration shall be either a template-id or, where all template arguments can be deduced, a template-name or operator-function-id. [Note: The declaration may declare a qualified-id, in which case the unqualified-id of the qualified-id must be a template-id. —end note ] If the explicit instantiation is for a member function, a member class or a static data member of a class template specialization, the name of the class template specialization in the qualified-id for the member name shall be a simple-template-id. An explicit instantiation shall appear in an enclosing namespace of its template. If the name declared in the explicit instantiation is an unqualified name, the explicit instantiation shall appear in the namespace where its template is declared or, if that namespace is inline (7.3.1), any namespace from its enclosing namespace set. [Note: Regarding qualified names in declarators, see 8.3. —end note ]

```
template<class T> class Array { void mf(); };
template class Array<char>;
template void Array<int>::mf();
template<class T> void sort(Array<T>& v) { /∗ ... ∗/ }
template void sort(Array<char>&);       // argument is deduced here
namespace N {
    template<class T> void f(T&) { }
}
template void N::f<int>(int&);
```

4 A declaration of a function template, a member function or static data member of a class template, or a member function template of a class or class template shall precede an explicit instantiation of that entity. A definition of a class template, a member class of a class template, or a member class template of a class or class template shall precede an explicit instantiation of that entity unless the explicit instantiation is preceded by an explicit specialization of the entity with the same template arguments. If the declaration of the explicit instantiation names an implicitly-declared special member function (Clause 12), the program is ill-formed.

5 For a given set of template arguments, if an explicit instantiation of a template appears after a declaration of an explicit specialization for that template, the explicit instantiation has no effect. Otherwise, for an explicit instantiation definition the definition of a function template, a member function template, or a member function or static data member of a class template shall be present in every translation unit in which it is explicitly instantiated.

6 An explicit instantiation of a class or function template specialization is placed in the namespace in which the template is defined. An explicit instantiation for a member of a class template is placed in the namespace where the enclosing class template is defined. An explicit instantiation for a member template is placed in the namespace where the enclosing class or class template is defined.

```
namespace N {
    template<class T> class Y { void mf() { } };
}
template class Y<int>;              // error: class template Y not visible
                                    // in the global namespace
using N::Y;
template class Y<int>;              // error: explicit instantiation outside of the
                                    // namespace of the template
template class N::Y<char*>;         // OK: explicit instantiation in namespace N
template void N::Y<double>::mf();   // OK: explicit instantiation
                                    // in namespace N
```

7 A trailing *template-argument* can be left unspecified(不指定) in an explicit instantiation of a function template specialization or of a member function template specialization provided it can be deduced from the type of a function parameter (14.8.2).

```cpp
template<class T> class Array { /∗ ... ∗/ };
template<class T> void sort(Array<T>& v) { /∗ ... ∗/ }

// instantiate sort(Array<int>&) - template-argument deduced
template void sort<>(Array<int>&);
```

8 An explicit instantiation that names a class template specialization is also an explicit instantiation of the same kind (declaration or definition) of each of its members (not including members inherited from base classes) that has not been previously explicitly specialized in the translation unit containing the explicit instantiation, except as described below. [Note: In addition, it will typically be an explicit instantiation of certain implementation-dependent data about the class. —end note ]

9 An explicit instantiation definition that names a class template specialization explicitly instantiates the class template specialization and is an explicit instantiation definition of only those members that have been defined at the point of instantiation.

命名了一个类模板特化的显式实例化定义显式地实例化了这个类模板特化, 并且这个显式实例化也是对那些仅在实例化点处定义的成员的显式实例化定义。

10 Except for inline functions and class template specializations, explicit instantiation declarations have the effect of suppressing(抑制) the implicit instantiation of the entity to which they refer. [Note: The intent is that an inline function that is the subject of an explicit instantiation declaration will still be implicitly instantiated when odr-used (3.2) so that the body can be considered for inlining, but that no out-of-line copy of the inline function would be generated in the translation unit.—end note ]

11 If an entity is the subject of both an explicit instantiation declaration and an explicit instantiation definition in the same translation unit, the definition shall follow the declaration. An entity that is the subject of an explicit instantiation declaration and that is also used in a way that would otherwise cause an implicit instantiation (14.7.1) in the translation unit shall be the subject of an explicit instantiation definition some-where in the program; otherwise the program is ill-formed, no diagnostic required. [Note: This rule does apply to inline functions even though an explicit instantiation declaration of such an entity has no other normative effect. This is needed to ensure that if the address of an inline function is taken in a translation unit in which the implementation chose to suppress the out-of-line body, another translation unit will supply the body.—end note ] An explicit instantiation declaration shall not name a specialization of a template with internal linkage.

12 The usual access checking rules do not apply to names used to specify explicit instantiations. [Note: In particular, the template arguments and names used in the function declarator (including parameter types, return types and exception specifications) may be private types or objects which would normally not be accessible and the template may be a member template or member function which would not normally be accessible. —end note ]

13 An explicit instantiation does not constitute a use of a default argument, so default argument instantiation is not done.

```
char* p = 0;
template<class T> T g(T x = &p) { return x; }
template int g<int>(int);       // OK even though &p isn’t an int.
```

### 14.7.3 Explicit specialization

1 An explicit specialization of any of the following:

— function template

— class template

— member function of a class template

— static data member of a class template

— member class of a class template

— member enumeration of a class template

— member class template of a class or class template

— member function template of a class or class template

can be declared by a declaration introduced by template<>; that is:

*explicit-specialization:
    template < > declaration*

显式特化(explicit specialization)就是通常所说的"全特化", "完全特化"。部分特化(partial specializations) [14.5.5] 就是通常所说的 "偏特化", 部分特化仅针对类模板.

```
template<class T> class stream;
template<> class stream<char> { /∗ ... ∗/ };
template<class T> class Array { /∗ ... ∗/ };
template<class T> void sort(Array<T>& v) { /∗ ... ∗/ }
template<> void sort<char*>(Array<char*>&) ;
```

Given these declarations, stream<char> will be used as the definition of streams of chars; other streams will be handled by class template specializations instantiated from the class template. Similarly, sort<char*> will be used as the sort function for arguments of type Array<char*>; other Array types will be sorted by functions generated from the template.

2 An explicit specialization shall be declared in a namespace enclosing the specialized template. An explicit specialization whose **declarator-id** is not qualified shall be declared in the nearest enclosing namespace of the template, or, if the namespace is inline (7.3.1), any namespace from its enclosing namespace set. Such a declaration may also be a definition. If the declaration is not a definition, the specialization may be defined later (7.3.1.2).

显式特化应该和被特化的模板位于同一命名空间.

显式特化的*declaratio-id*如果是非限定的, 则它应该被声明在

- 基本模板的最内层封闭命名空间, 或，
- 如果这个最内层封闭命名空间是内联的, 来自于它的封闭命名空间集合的任何命名空间.

> [7.3.1/8] The *enclosing namespace set* of O is the set of namespaces consisting of the innermost non-inline namespace enclosing an inline namespace O , together with any intervening inline namespaces.
> 

```cpp
namespace N{
	inline namespace Out{
		inline namespace In{
			template <typename T>
			struct S {};
		}
	}
                    //ok, N属于In的封闭命名空间集合中的一员.
	template <>
	struct S<void*> {};
}

namespace N{
	namespace Out{
		inline namespace In{
			template <typename T>
			struct S {};
		}
	}
                    //error, In的封闭命名空间集合不包括N.
	template <>     
	struct S<void*> {};
}
```

> [https://www.zhihu.com/question/54468727](https://www.zhihu.com/question/54468727)
> 

3 A declaration of a function template or class template being explicitly specialized shall precede the declaration of the explicit specialization. [Note: A declaration, but not a definition of the template is required. —end note ] The definition of a class or class template shall precede the declaration of an explicit specialization for a member template of the class or class template.

正在被显式特化的函数模板或类模板的声明应该在显式特化声明之前. (也就是说基本模板在模板特化前必须声明).

```
template<> class X<int> { /∗ ... ∗/ }; // error: X not a template
template<class T> class X;
template<> class X<char*> { /∗ ... ∗/ }; // OK: X is a template
```

4 A member function, a member function template, a member class, a member enumeration, a member class template, or a static data member of a class template may be explicitly specialized for a class specialization that is implicitly instantiated; in this case, the definition of the class template shall precede the explicit specialization for the member of the class template. If such an explicit specialization for the member of a class template names an implicitly-declared special member function (Clause 12), the program is ill-formed.

被隐式实例化的类特化的 member function, a member function template, a member class, a member enumeration, a member class template, or a static data member 可以被显式的特化， 在这种情况下, 类模板的定义应在类模板的成员的显式特化之前.如果类模板成员的这种显式特化命名了一个隐式声明的特殊成员函数, 则该程序是ill-formed.

```cpp
template <typename T>
class B
{};

template <>
B<int>::B()
{ /*explicit specializationdef body */} // this is forbidden by ISO c++
                                        // and when compiling with VS2013 gives compile error
                                        // cannot define a compiler-generated special member
                                        // function (must be declared in the class first)
```

```cpp
template <typename T>
class A
{
public:
    A()
    { /* some definition */}
};

template <>
A<int>::A()
{ /*explicit specialization def body*/} // this is OK
```

> [https://stackoverflow.com/questions/27806775/class-template-special-member-function-explicit-specialization](https://stackoverflow.com/questions/27806775/class-template-special-member-function-explicit-specialization)
> 

在非类模板中, 隐式声明的特殊成员函数就不能显式定义在类定义之外,  这里规定模板特化也一样.

```cpp
class B
{};

B::B()
{ }
```

5 A member of an explicitly specialized class is not implicitly instantiated from the member declaration of the class template; instead, the member of the class template specialization shall itself be explicitly defined if its definition is required. In this case, the definition of the class template explicit specialization shall be in scope at the point at which the member is defined.

The definition of an explicitly specialized class is unrelated to the definition of a generated specialization. That is, its members need not have the same names, types, etc. as the members of a generated specialization.

Members of an explicitly specialized class template are defined in the same manner as members of normal classes, and not using the template<> syntax. The same is true when defining a member of an explicitly specialized member class. However, template<> is used in defining a member of an explicitly specialized member class template that is specialized as a class template.

> [http://en.cppreference.com/w/cpp/language/template_specialization](http://en.cppreference.com/w/cpp/language/template_specialization)
> 

```cpp
template<class T> struct A {
    struct B { };
    template<class U> struct C { };
};
template<> struct A<int> {
    void f(int);    
};
void h() {
    A<int> a;
    a.f(16); // A<int>::f must be defined somewhere
}

// template<> not used for a member of an
// explicitly specialized class template
void A<int>::f(int) { /∗ ... ∗/ }
template<> struct A<char>::B {
    void f();
};

// template<> also not used when defining a member of
// an explicitly specialized member class
void A<char>::B::f() { /∗ ... ∗/ }

template<> template<class U> struct A<char>::C {
    void f();
};
// template<> is used when defining a member of an explicitly
// specialized member class template specialized as a class template

template<>
template<class U> void A<char>::C<U>::f() { /∗ ... ∗/ }
template<> struct A<short>::B {
    void f();
};
template<> void A<short>::B::f() { /∗ ... ∗/ } // error: template<> not permitted
template<> template<class U> struct A<short>::C {
    void f();
};
template<class U> void A<short>::C<U>::f() { /∗ ... ∗/ } // error: template<> required
```

6 If a template, a member template or a member of a class template is explicitly specialized then that specialization shall be declared before the first use of that specialization that would cause an implicit instantiation to take place, in every translation unit in which such a use occurs; no diagnostic is required. If the program does not provide a definition for an explicit specialization and either the specialization is used in a way that would cause an implicit instantiation to take place or the member is a virtual member function, the program is ill-formed, no diagnostic required. An implicit instantiation is never generated for an explicit specialization that is declared but not defined.

```cpp
class String { };
template<class T> class Array { /∗ ... ∗/ };
template<class T> void sort(Array<T>& v) { /∗ ... ∗/ }
void f(Array<String>& v) {
    sort(v); // use primary template
             // sort(Array<T>&), T is String
}
template<> void sort<String>(Array<String>& v); // error: specialization
                                                // after use of primary template
template<> void sort<>(Array<char*>& v);        // OK: sort<char*> not yet used
template<class T> struct A {
    enum E : T;
    enum class S : T;
};
template<> enum A<int>::E : int { eint };       // OK
template<> enum class A<int>::S : int { sint }; // OK
template<class T> enum A<T>::E : T { eT };
template<class T> enum class A<T>::S : T { sT };
template<> enum A<char>::E : int { echar }; // ill-formed, A<char>::E was instantiated
                                            // when A<char> was instantiated
template<> enum class A<char>::S : int { schar }; // OK
```

7 The placement of explicit specialization declarations for function templates, class templates, member functions of class templates, static data members of class templates, member classes of class templates, member enumerations of class templates, member class templates of class templates, member function templates of class templates, member functions of member templates of class templates, member functions of member templates of non-template classes, member function templates of member classes of class templates, etc., and the placement of partial specialization declarations of class templates, member class templates of non-template classes, member class templates of class templates, etc., can affect whether a program is

well-formed according to the relative positioning of the explicit specialization declarations and their points of instantiation in the translation unit as specified above and below. When writing a specialization, be careful about its location; or to make it compile will be such a trial as to kindle its self-immolation.

8 A template explicit specialization is in the scope of the namespace in which the template was defined.

```cpp
namespace N {
    template<class T> class X { /∗ ... ∗/ };
    template<class T> class Y { /∗ ... ∗/ };
    template<> class X<int> { /∗ ... ∗/ };  // OK: specialization
                                            // in same namespace
    template<> class Y<double>; // forward declare intent to
                                // specialize for double
}
template<> class N::Y<double> { /∗ ... ∗/ }; // OK: specialization
                                            // in same namespace
```

9 A *simple-template-id* that names a class template explicit specialization that has been declared but not defined can be used exactly like the names of other incompletely-defined classes (3.9).

```cpp
template<class T> class X; // X is a class template
template<> class X<int>;

X<int>* p; // OK: pointer to declared class X<int>
X<int> x;   // error: object of incomplete class X<int>
```

10 A trailing *template-argument* can be left unspecified in the *template-id* naming an explicit function template specialization provided it can be deduced from the function argument type.

尾随模板实参可以不指定在命名了显式函数模板特化的*template-id*, 只要它可以从函数实参类型推导出来.

```cpp
template<class T> class Array { /∗ ... ∗/ };
template<class T> void sort(Array<T>& v);

// explicit specialization for sort(Array<int>&)
// with deduced template-argument of type int
template<> void sort(Array<int>&);
```

11 A function with the same name as a template and a type that exactly matches that of a template specialization is not an explicit specialization (14.5.6).

某个函数与模板具有相同名字并且这个函数类型恰好匹配一个模板特化, 则这个函数不是一个显式特化.

12 An explicit specialization of a function template is inline only if it is declared with the **inline** specifier or defined as deleted, and independently of whether its function template is inline

```
template<class T> void f(T) { /∗ ... ∗/ }
template<class T> inline T g(T) { /∗ ... ∗/ }
template<> inline void f<>(int) { /∗ ... ∗/ } // OK: inline
template<> int g<>(int) { /∗ ... ∗/ } // OK: not inline
```

13 An explicit specialization of a static data member of a template is a definition if the declaration includes an initializer; otherwise, it is a declaration. [Note: The definition of a static data member of a template that requires default initialization must use a braced-init-list:

```cpp
template<> X Q<int>::x; // declaration
template<> X Q<int>::x (); // error: declares a function
template<> X Q<int>::x { }; // definition
```

14 A member or a member template of a class template may be explicitly specialized for a given implicit instantiation of the class template, even if the member or member template is defined in the class template definition. An explicit specialization of a member or member template is specified using the syntax for explicit specialization.

类模板的隐式实例化的成员或成员模板可以被显式的特化, 即使这个成员或成员模板被定义在类定义中. 类模板的成员或成员模板的显式特化使用显式特化语法指定.

```cpp
template<class T> struct A {
    void f(T);
    template<class X1> void g1(T, X1);
    template<class X2> void g2(T, X2);
    void h(T) { }
};

// specialization
template<> void A<int>::f(int);s

// out of class member template definition
template<class T> template<class X1> void A<T>::g1(T, X1) { }

// member template specialization
template<> template<class X1> void A<int>::g1(int, X1);

//member template specialization
template<> template<>
void A<int>::g1(int, char); // X1 deduced as char
template<> template<>
void A<int>::g2<char>(int, char); // X2 specified as char

// member specialization even if defined in class definition
template<> void A<int>::h(int) { }
```

15 A member or a member template may be nested within many enclosing class templates. In an explicit specialization for such a member, the member declaration shall be preceded by a template<> for each enclosing class template that is explicitly specialized.

```cpp
template<class T1> class A {
    template<class T2> class B {
        void mf();
    };
};
template<> template<> class A<int>::B<double>;
template<> template<> void A<char>::B<char>::mf();
```

16 In an explicit specialization declaration for a member of a class template or a member template that appears in namespace scope, the member template and some of its enclosing class templates may remain unspecialized, except that the declaration shall not explicitly specialize a class member template if its enclosing class templates are not explicitly specialized as well. In such explicit specialization declaration, the keyword ****template**** followed by a **template-parameter-list** shall be provided instead of the ****template<>**** preceding the explicit specialization declaration of the member. The types of the **template-parameters** in the **template-parameter-list** shall be the same as those specified in the primary template definition.

在命名空间作用域内出现的类模板成员或成员模板的显式特化声明中, 成员模板及其一些它的封闭类模板可能仍然是未被特定化的, 除非声明没有显式地特化类成员模板如果它的封闭类模板也没有被显式特化. 在这样一个显式特化声明中, 在成员的显式特化声明之前应该提供 *template <template-parameter-list>* 来替换 *template<>*.  在 *template-parameter-list* 中的 *template-parameters* 的类型应该与那些被指定基本模板定义中的类型相同.

```cpp
template <class T1> class A {
    template<class T2> class B {
        template<class T3> void mf1(T3);
        void mf2();
    };
};
template <> template <class X>
class A<int>::B {
    template <class T> void mf1(T);
};
template <> template <> template<class T>
void A<int>::B<double>::mf1(T t) { }    //ok
template <class Y> template <>
void A<Y>::B<double>::mf2() { } // ill-formed; B<double> is specialized but
                                // its enclosing class template A is not
```

17 A specialization of a member function template or member class template of a non-specialized class template is itself a template.

18 An explicit specialization declaration shall not be a friend declaration.

19 Default function arguments shall not be specified in a declaration or a definition for one of the following explicit specializations:

— the explicit specialization of a function template;

— the explicit specialization of a member function template;

— the explicit specialization of a member function of a class template where the class template specialization to which the member function specialization belongs is implicitly instantiated. [Note: Default function arguments may be specified in the declaration or definition of a member function of a class template specialization that is explicitly specialized. —end note ]