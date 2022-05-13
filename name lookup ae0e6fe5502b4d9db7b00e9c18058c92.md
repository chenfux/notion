# name lookup

Created: March 1, 2022 5:19 PM
Last Edited Time: April 12, 2022 2:34 PM

# Name lookup

### 3.4 Name lookup

1 Name lookup associates the use of a name with a declaration (3.1) of that name.

Name lookup shall find an unambiguous declaration for the name (see 10.2).

Name lookup may associate more than one declaration with a name if it finds the name to be a function name; the declarations are said to form a set of overloaded functions (13.1).

Overload resolution (13.3) takes place after name lookup has succeeded.

The access rules (Clause 11) are considered only once name lookup and function overload resolution (if applicable) have succeeded. 

访问控制规则(pubilc, protected, private)仅在名称查找和重载解析都成功的情况下才被考虑

Only after name lookup, function overload resolution (if applicable) and access checking have succeeded are the attributes introduced by the name’s declaration used further in expression processing (Clause 5).

2 A name “looked up in the context of an expression” is looked up as an unqualified name in the scope where the expression is found.

3 The *injected-class-name* of a class (Clause 9) is also considered to be a member of that class for the purposes of name hiding and lookup.

### 3.4.1 非限定名称查找(Unqualified name lookup)

1 In all the cases listed in 3.4.1, the scopes are searched for a declaration in the order listed in each of the respective categories; name lookup ends as soon as a declaration is found for the name. If no declaration is found, the program is ill-formed.

2 The declarations from the namespace nominated by a using-directive become visible in a namespace enclosing the using-directive; see 7.3.4. For the purpose of the unqualified name lookup rules described in 3.4.1, the declarations from the namespace nominated by the using-directive are considered members of that enclosing namespace.

3 The lookup for an unqualified name used as the postfix-expression of a function call is described in 3.4.2. [ Note: For purposes of determining (during parsing) whether an expression is a postfix-expression for a function call, the usual name lookup rules apply. The rules in 3.4.2 have no effect on the syntactic interpretation of an expression.

```cpp
typedef int f;
namespace N
{
	struct A
	{
		friend void f(A &);
		operator int();
		void g(A a)
		{
			int i = f(a); // f is the typedef, not the friend function: equivalent to int(a)
		}
	};
}
```

Because the expression is not a function call, the argument-dependent name lookup (3.4.2) does not apply and the friend function f is not found. — end note ]

根据[7.3.1.2/3], 类A中的友元函数声明是f函数的首次声明, 所以f将被视为最内层封闭命名空间N中的一个成员, 但是在命名空间N的作用域中并没有提供一个与f(的友元声明)匹配的函数声明, 所以通过限定、非限定名称查找都不会找到N::f, 而按照[3.4.1/8]的查找顺序, 名称查找f将找到::f, 这使得`f(a)`被解析为`simple-type-specifier ( expression-list opt )`, 而不会被解析为`postfix-expression ( expression-list opt )`, 因此[3.4.2]的 argument-dependent lookup (3.4.2) 不会被应用.

```cpp
namespace N
{
	struct A;
	void f(A &) {}
	struct A
	{
		friend void f(A &);
		operator int();
		void g(A a)
		{
			int i = f(a); // error: find N::f 
		}
	};
}
```

命名空间N中提供了与f的友元声明匹配的函数声明, 因此非限定名称查找将找到N::f, 而它的返回值类型为 void, 所以这里的赋值会失败.

```cpp
typedef int f;
namespace N
{
	struct A
	{
		friend void f(A &);
		operator int();
		void g(A a)
		{
			int i = f(a); // f is the typedef, not the friend function: equivalent to int(a)
		}
	};
	void f(A &) {}
}
```

命名空间N中提供了与f的友元声明匹配的函数声明, 按照[7.3.1.2/3], N::f允许被通过限定、非限定名称查找发现, 也就是说满足了[7.3.1.2/3]的可被查找的要求, 再按照[3.4.1/8]非限定查找规则, 是查找不到N::f, 所以仍然找到::f

```cpp
typedef int f;
namespace N
{
	struct A
	{
		friend void f(A &);
		operator int();
		void g(A a);
	};
	void f(A &) {}
}
void N::A::g(N::A a)
{
	int i = f(a); // error: find N::f
}
```

按照[3.4.1/8]非限定查找规则, 可以查找到 N::f. (如果没有 friend void f(A&) 的声明,  则f不满足[7.3.1.2/3]的可被查找的要求, 所以按照[3.4.1/8]的非限定查找规则,也找不到N::f,  而且因为::f的存在, 不能应用参数依赖查找)

```cpp
namespace N
{
	struct A
	{
		friend void f(A &);
		operator int();
		void g(A a)
		{
			int i = f(a); // error: ADL查找发现 N::f
		}
	};
}
```

命名空间N中没有提供与f的友元声明匹配的函数声明, 按照[7.3.1.2/3], N::f不允许被通过限定/非限定名称查找发现, 所以, 按照[3.4.1/8]的非限定查找查找不到N::f, 但是, 按照[3.4.1/3]的规则, 根据 [], f 被解析为`postfix-expression ( expression-list opt )`, 并发生参数依赖查找, 因此为f找到友元声明f(N::f). 

[Note: [https://stackoverflow.com/questions/31772588/c-name-lookup-example-from-the-standard](https://stackoverflow.com/questions/31772588/c-name-lookup-example-from-the-standard) —end note]

全局命名空间中使用的名称查找 

4 A name used in global scope, outside of any function, class or user-declared namespace, shall be declared before its use in global scope.

```cpp
int n = 1;     // declaration of n
int x = n + 1; // OK: lookup finds ::n
int z = y - 1; // Error: lookup fails
int y = 2;     // declaration of y
```

用户声明(user-declared)的命名空间中使用的名称查找 

5 A name used in a user-declared namespace outside of the definition of any function or class shall be declared before its use in that namespace or before its use in a namespace enclosing its namespace.

```cpp
int n = 1; // declaration
namespace N
{
	int m = 2;
	namespace Y
	{
		int x = n; // OK, lookup finds ::n
		int y = m; // OK, lookup finds ::N::m
		int j = y; // OK, lookup finds ::N::Y::y
		int z = k; // Error: lookup fails
	}
	int k = 3;
}
```

命名空间成员函数中使用的名称查找 

6 A name used in the definition of a function following the function’s *declarator-id*28 that is a member of namespace N (where, only for the purpose of exposition, N could represent the global scope) shall be declared before its use in the block in which it is used or in one of its enclosing blocks (6.3) or, shall be declared before its use in namespace N or, if N is a nested namespace, shall be declared before its use in one of N’s enclosing namespaces.

```cpp
namespace A
{
	namespace N
	{
		void f();
	}
}
void A::N::f()
{
	i = 5; // The following scopes are searched for a declaration of i:    
		     // 1) outermost block scope of A::N::f, before the use of i    
		     // 2) scope of namespace N    
		     // 3) scope of namespace A    
		     // 4) global scope, before the definition of A::N::f
}
```

```cpp
namespace A
{
	namespace N
	{
		void f();
		int i = 3; // found 3rd (if 2nd is not present)
	}
	int i = 4; // found 4th (if 3rd is not present)
}
int i = 5; // found 5th (if 4th is not present)
void A::N::f()
{
	int i = 2; // found 2nd (if 1st is not present)
	while (true)
	{
		int i = 1; // found 1st: lookup is done
		std::cout << i;
	}
} 
// int i; // not found
namespace A
{
	namespace N
	{ 
		// int i; // not found
	}
}
```

类中使用的名称查找 

7 A name used in the definition of a class X outside of a member function body or nested class definition29 shall be declared in one of the following ways: — before its use in class X or be a member of a base class of X (10.2), or — if X is a nested class of class Y (9.7), before the definition of X in Y, or shall be a member of a base class of Y (this lookup applies in turn to Y ’s enclosing classes, starting with the innermost enclosing class),30 or — if X is a local class (9.8) or is a nested class of a local class, before the definition of class X in a block enclosing the definition of class X, or — if X is a member of namespace N, or is a nested class of a class that is a member of N, or is a local class or a nested class within a local class of a function that is a member of N, before the deﬁnition of class X in namespace N or in one of N ’s enclosing namespaces.

```cpp
namespace M
{
	class B
	{
	};
}
namespace N
{
	class Y : public M::B
	{
		class X
		{
			int a[i];
		};
	};
}
// The following scopes are searched for a declaration of i:
// 1) scope of class N::Y::X, before the use of i
// 2) scope of class N::Y, before the definition of N::Y::X
// 3) scope of N::Y’s base class M::B
// 4) scope of namespace N, before the definition of N::Y
// 5) global scope, before the definition of N
```

[ Note: When looking for a prior declaration of a class or function introduced by a friend declaration, scopes outside of the innermost enclosing namespace scope are not considered; see 7.3.1.2. — end note ] [ Note: 3.3.7 further describes the restrictions on the use of names in a class definition. 9.7 further describes the restrictions on the use of names in nested class definitions. 9.8 further describes the restrictions on the use of names in local class definitions. — end note ]

```cpp
namespace M
{
	// const int i = 1; // never found
	class B
	{
		// const const int i = 3; // found 3nd (but later rejected by access check)
	};
}
// const int i = 5; // found 5th
namespace N
{
	// const int i = 4; // found 4th
	class Y : public M::B
	{
		// static const int i = 2; // found 2nd
		class X
		{
			// static const int i = 1; // found 1st
			int a[i]; // use of i
					  // static const int i = 1; // never found
		};
		// static const int i = 2; // never found
	};
	// const int i = 4; // never found
}
// const int i = 5; // never found
```

类成员函数定义中使用的名称查找

8 A name used in the definition of a member function (9.3) of class X following the function’s declarator-id31 or in the *brace-or-equal-initializer* of a non-static data member (9.2) of class X shall be declared in one of the following ways: — before its use in the block in which it is used or in an enclosing block (6.3), or — shall be a member of class X or be a member of a base class of X (10.2), or — if X is a nested class of class Y(9.7),shall be a member of Y,or shall be a member of a base class of Y (this lookup applies in turn to Y’s enclosing classes, starting with the innermost enclosing class),32 or — if X is a local class (9.8) or is a nested class of a local class, before the definition of class X in a block enclosing the definition of class X, or — if X is a member of namespace N,or is a nested class of a class that is a member of N,or is a local class or a nested class within a local class of a function that is a member of N, before the use of the name, in namespace N or in one of N ’s enclosing namespaces.

```cpp
class B
{
};
namespace M
{
	namespace N
	{
		class X : public B
		{
			void f();
		};
	}
}
void M::N::X::f() { i = 16; } // The following scopes are searched for a declaration of i:
							  // 1) outermost block scope of M::N::X::f, before the use of i
							  // 2) scope of class M::N::X
							  // 3) scope of M::N::X’s base class B
							  // 4) scope of namespace M::N
							  // 5) scope of namespace M
							  // 6) global scope, before the definition of M::N::X::f
```

[ Note: 9.3 and 9.4 further describe the restrictions on the use of names in member function definitions. 9.7 further describes the restrictions on the use of names in the scope of nested classes. 9.8 further describes the restrictions on the use of names in local class definitions. — end note ]

```cpp
class B
{
	// int i; // found 3rd
};
namespace M
{
	// int i; // found 5th
	namespace N
	{
		// int i; // found 4th
		class X : public B
		{
			// int i; // found 2nd
			void f();
			// int i; // found 2nd as well
		}; 
	  // int i; // found 4th
	}
} 
// int i; // found 6th
void M::N::X::f()
{
	// int i; // found 1st
	i = 16;
	// int i; // never found
}
namespace M
{
	namespace N
	{
		// int i; // never found
	}
}
```

友元函数定义中使用的名称查找

9 Name lookup for a name used in the definition of a friend function (11.3) defined inline in the class granting friendship shall proceed as described for lookup in member function definitions. If the friend function is not defined in the class granting friendship, name lookup in the friend function definition shall proceed as described for lookup in namespace member function definitions.

```cpp
int i = 3; // found 3rd for f1, found 2nd for f2
struct X
{
	static const int i = 2; // found 2nd for f1, never found for f2
	friend void f1(int x)
	{
		// int i; // found 1st
		i = x; // finds and modifies X::i
	}
	friend int f2();
	// static const int i = 2; // found 2nd for f1 anywhere in class scope
};
void f2(int x)
{
	// int i; // found 1st
	i = x; // finds and modifies ::i
}

//对f1中使用的i按照类成员函数中使用的名称查找规则来查找
//对f2中使用的i按照命名空间成员函数中使用的名称查找规则来查找
```

成员函数的友元声明中使用的名称查找

10 In a friend declaration naming a member function, a name used in the function declarator and not part of a template-argument in the declarator-id is first looked up in the scope of the member function’s class (10.2). If it is not found, or if the name is part of a template-argument in the declarator-id, the look up is as described for unqualified names in the definition of the class granting friendship.

```cpp
// the class whose member functions are friended
struct A
{
	typedef int AT;
	void f1(AT);
	void f2(float);
	template <class T>
	void f3();
}; 
// the class that is granting friendship
struct B
{
	typedef char AT;
	typedef float BT;
	friend void A::f1(AT);	 // lookup for AT finds A::AT
	friend void A::f2(BT);	 // lookup for BT finds B::BT
	friend void A::f3<AT>(); // lookup for AT finds B::AT
};
```

默认参数中使用的名称查找

11 During the lookup for a name used as a default argument (8.3.6) in a function *parameter-declaration-clause* or used in the expression of a mem-initializer for a constructor (12.6.2), the function parameter names are visible and hide the names of entities declared in the block, class or namespace scopes containing the function declaration. [ Note: 8.3.6 further describes the restrictions on the use of names in default arguments. 12.6.2 further describes the restrictions on the use of names in a ctor-initializer . — end note ]

```cpp
class X
{
	int a, b, i, j;

public:
	const int &r;
	X(int i) : r(a),   // initializes X::r to refer to X::a
			   b(i),	     // initializes X::b to the value of the parameter i
			   i(i),	     // initializes X::i to the value of the parameter i
			   j(this->i)  // initializes X::j to the value of X::i
	{
	}
}; 
// clang 9.0(clang-900.0.38) / gcc 7.1 都认为 `int b = a` 中的a为形参a, 
// 而不是 ::a, 符合标准, 所以两者都给出编译错误.
int a;
int f(int a, int b = a); // error: lookup for a finds the parameter a, not ::a and parameters are not allowed as default arguments

struct T
{
	static const int i = 0;
	//clang 9.0 认为 `int j = i` 中的i为形参i, 编译失败, 符合标准, 因为形参i会隐藏包含f的那个类X中的i的其他实体声明, g++ 7.1 认为 i 是 T::i, 成功编译, 可能不符合标准
	void f(int i, int j = i);
};
```

枚举定义中使用的名称查找. 

12 During the lookup of a name used in the *constant-expression* of an *enumerator-definition*, previously declared *enumerators* of the enumeration are visible and hide the names of entities declared in the block, class, or namespace scopes containing the enum-specifier.

```cpp
const int RED = 7;
enum class color
{
	RED,
	GREEN = RED + 2, // RED finds color::RED, not ::RED, so GREEN = 2
	BLUE = ::RED + 4 // qualified lookup finds ::RED, BLUE = 11
};
```

静态数据成员中使用的名称查找. 

13 A name used in the definition of a static data member of class X (9.4.2) (after the qualified-id of the static member) is looked up as if the name was used in a member function of X. [ Note: 9.4.2 further describes the restrictions on the use of names in the definition of a static data member. — end note ]

```cpp
struct X
{
	static int x;
	static const int n = 1; // found 1st
};
int n = 2; // found 2nd.
int X::x = n; // finds X::n, sets X::x to 1, not 2
```

```cpp
struct X
{
	int x;
	static const int n = 1; // found 1st
	void f();
};
int n = 2; // found 2nd.
void X::f()
{
	x = n; // finds X::n, sets X::x to 1, not 2
}
```

14 If a variable member of a namespace is defined outside of the scope of its namespace then any name that appears in the definition of the member (after the declarator-id) is looked up as if the definition of the member occurred in its namespace.

```cpp
namespace N
{
	int i = 4;
	extern int j;
}
int i = 2;
int N::j = i; // N::j == 4
```

function-try-block的处理程序中使用的名称查找. 15 A name used in the handler for a function-try-block (Clause 15) is looked up as if the name was used in the outermost block of the function definition. In particular, the function parameter names shall not be redeclared in the exception-declaration nor in the outermost block of a handler for the function-try-block. Names declared in the outermost block of the function definition are not found when looked up in the scope of a handler for the function-try-block. [ Note: But function parameter names are found. — end note ]

```cpp
int n = 3; // found 3rd
int f(int n = 2) // found 2nd
try {
   int n = -1;  // never found
} catch(...) {
   // int n = 1; // found 1st
   assert(n == 2); // loookup for n finds function parameter f
   throw;
}
```

### 3.4.2 参数依赖查找(Argument-dependent name lookup)

1 When the *postfix-expression* in a function call (5.2.2) is an *unqualified-id*, other namespaces not considered during the usual unqualified lookup (3.4.1) may be searched, and in those namespaces, namespace-scope friend function declarations (11.3) not otherwise visible may be found. These modifications to the search depend on the types of the arguments (and for template template arguments, the namespace of the template argument).

```cpp
namespace N
{
	struct S
	{
	};
	void f(S);
}
void g()
{
	N::S s;
	f(s);	  // OK: calls N::f
	(f)(s); // error: N::f not considered; parentheses                    
			    // prevent argument-dependent lookup
}
```

> 参数依赖查找的一个前提是postfix-expression出现在函数调用中作为unqualified-id,必须要符合这个前提, 参考[3.4.1/3]说明.
> 

2 For each argument type T in the function call …

ADL查找的候选命名空间和类集合

对于函数调用中的每个实参类型T, 存在一组零个或多个要被考虑的关联命名空间关联类的集合. 这一组命名空间和类的集合完全由函数的实参的类型(和模板模板参数的命名空间)决定.

*typedef names* 和 *using-declarations* 所指定的类型不会影响这个集合. 命名空间和类的集合以下面的方式确定:

**如果T是一个 基本类型(3.9)**, 它的关联的命名空间和类集合都为空.

**如果T是一个类类型(包括联合)**, 它的关联类是: (1) 它本身, (2) 作为它成员的类(如果有), (3) 它的直接或间接基类. 它的关联名称空间是: 它的每个关联类的最内层封闭命名空间. (就是包含每个关联类的最内层命名空间); 此外, 如果T是一个类模板特化, 它的关联类和关联名称空间也包含: 模板实参类型(除模板模板参数)所关联的类或命名空间, 任何模板模板参数所属的命名空间, 任何被用作模板模板参数的成员模板所属的类,

**非类型模板参数不会关联任何命名空间和类;**

**如果T是一个枚举类型**, 它关联的命名空间是: 包含这个枚举声明的最内层命名空间, 如果它是一个类成员, 则它的关联类是: 它所属的类 否则, 它不关联任何类.

**如果T是一个U类型的指针或U类型的数组**, 它的关联命名空间和类是: U所关联的命名空间和类。

**如果T是一个函数类型**, 它关联的命名空间和类是: 它的参数类型和返回值类型所关联的那些命名空间和类.

**如果T是一个类X的成员函数指针**, 它关联的命名空间和类是: 它的函数参数类型, 返回值类型, 以及X 它们全部的关联命名空间和类.

**如果T是一个类X的数据成员指针**, 它关联的命名空间和类是: 这个成员的类型和类X所关联的全部命名空间和类.

如果一个被关联命名空间是内联的, 那么包含它的命名空间也被包含到关联集合中. 如果一个被关联命名空间直接包含某些内联的命名空间, 那么这些内联的命名空间也会被包含在这个集合中.

**如果实参是一个重载函数/函数模板的地址(&X)或名字(X)**, 则它关联的命名空间和类是: 这个重载函数集合中所有函数所关联的命名空间和类的并集. 即: 所有函数的返回值类型和形参类型所关联的命名空间和类构成的集合; 另外, 如果上述的这个重载函数集合是以*template-id*命名的, 它的关联类和关联命名空间集合也包含它的类型模板参数和模板模板参数.

3 Let X be the lookup set produced by unqualified lookup (3.4.1) and let Y be the lookup set produced by argument dependent lookup (defined as follows). If X contains — a declaration of a class member, or — a block-scope function declaration that is not a using-declaration, or — a declaration that is neither a function or a function template then Y is empty. Otherwise Y is the set of declarations found in the namespaces associated with the argument types as described below. The set of declarations found by the lookup of the name is the union of X and Y . [ Note: The namespaces and classes associated with the argument types can include namespaces and classes already considered by the ordinary unqualified lookup. — end note ]

```cpp
namespace NS
{
	class T
	{
	};
	void f(T);
	void g(T, int);
}
NS::T parm;
void g(NS::T, float);
int main()
{
	f(parm); // OK: calls NS::f
					 // 通过非限定查找没有找到任何f的声明, 所以X集合为空集, 而Y包含NS::f, 
					 // 所以,这里调用的f为NS::f
	extern void g(NS::T, float);
	g(parm, 1); // OK: calls g(NS::T, float)
							// 通过非限定名称查找发现::g, 
						  // 符合"a declaration that is neither a function or a function template", 所以g的查找集合X包含::g, Y为空集, 所以这里调用的是::g
}
```

```cpp
namespace A
{
	struct X;
	struct Y;
	void f(int);
	void g(X);
}
namespace B
{
	void f(int i)
	{
		f(i); // calls B::f (endless recursion)
	}
	void g(A::X x)
	{
		g(x); // Error: ambiguous between B::g (ordinary lookup) and A::g (argument-dependent lookup)
	}
	void h(A::Y y)
	{
		h(y); // calls B::h (endless recursion): ADL examines the A namespace but finds no A::h, so only B::h from ordinary lookup is used
	}
}
```

```cpp
namespace A
{
	struct S
	{
		void f(S){};
	}; /*    int f(S){        return 0;    };    */
}
namespace B
{
	void g(A::S s)
	{
		f(s); // error: 'f' was not declared in this scope
			    // 对查找集合Y的定义, 仅包括参数类型所关联的命名空间(namespaces associated with the argument types),
			    // 不包括类作用域, 所以集合Y为空, 集合X也为空
	};
};
```

除了下面三种情况, 对候选命名空间的查找等价于该对命名空间的限定查找.

4 When considering an associated namespace, the lookup is the same as the lookup performed when the associated namespace is used as a qualifier (3.4.3.2) except that: 1 限定查找会考虑using指令, ADL查找不考虑. — Any using-directives in the associated namespace are ignored.

2 根据[7.3.1.2/3], 某些友元声明对限定查找不可见, 但是对ADL查找可见. — Any namespace-scope friend functions or friend function templates declared in associated classes are visible within their respective namespaces even if they are not visible during an ordinary lookup (11.3).

3 ADL查找仅查找函数名 — All names except those of (possibly overloaded) functions and function templates are ignored.

```cpp
template <typename T>
struct number
{
	number(int);
	friend number gcd(number x, number y) { return 0; }; // definition within a class template unless a matching declaration is provided gcd is an invisible (except through ADL) member of this namespace
};														 
void g()
{
	number<double> a(3), b(4);
	a = gcd(a, b); // finds gcd because number<double> is an associated class, making gcd visible in its namespace (global scope) (如上定义 2.)
	//  b = gcd(3,4); // Error; gcd is not visible  (实参关联的命名空间、类为空, 查找集合X,Y都为空集, 所以没有执行ADL查找).
}
```

### 3.4.3 限定名称查找(Qualified name lookup)

1 The name of a class or namespace member or enumerator can be referred to after the :: scope resolution operator (5.1) applied to a *nested-name-specifier* that denotes its class, namespace, or enumeration.

If a :: scope resolution operator in a *nested-name-specifier* is not preceded by a *decltype-specifier*, lookup of the name preceding that :: considers only namespaces, types, and templates whose specializations are types. If the name found does not designate a namespace or a class, enumeration, or dependent type, the program is ill-formed.

```cpp
class A
{
public:
	static int n;
};
int main()
{
	int A;
	A::n = 42; // OK
	A b; // ill-formed: A does not name a type.     
		   // 通过非限定查找发现局部变量A
}
```

```cpp
struct X
{
	static int n;
};
int X::n = 0;
X A;
int main()
{
	// A::n = 0; // error: 'A' is not a class, namespace, or enumeration
				 // 根据[3.4.3/1], A::中::前面的名称A不是一个decltype-specifier,
				 // 所以对A的查找仅考虑"namespaces, types, and templates whose specializations are types",
				 // 因此找不到符合的 A.
	decltype(A)::n = 0; // decltype(A)::中::的前面是一个decltype-specifier, 所以对A的查找没有限制, 所以通过非限定名称查找A可以找到变量A, 通过decltype(A)算出其类型为 X
}
```

3 In a declaration in which the *declarator-id* is a *qualified-id*, names used before the *qualified-id* being declared are looked up in the defining namespace scope; names following the *qualified-id* are looked up in the scope of the member’s class or namespace.

```cpp
class X
{
};
class C
{
	class X
	{
	};
	static const int number = 50;
	static X arr[number];
};
X C::arr[number]; // ill-formed: equivalent to: ::X C::arr[C::number]; not to: C::X C::arr[C::number];
				  // 根据"names used before the qualified-id being declared are looked up in the defining namespace scope;",
				  // ::arr属于正在被声明的限定id(the qualified-id being declared), 在其之前使用的名字X和C会被查找在当前声明发生的作用域,
				  // 也就是全局作用域, 但是由于无法找到::X而失败.
				  // 根据"names following the *qualified-id* are looked up in the scope of the member’s class or namespace.",
				  // ::arr之后的名字number会在arr所属的类、命名空间中查找, 所以找到C::number
```

仅由`::`限定的名称查找 4 A name prefixed by the unary scope operator :: (5.1) is looked up in global scope, in the translation unit where it is used. The name shall be declared in global namespace scope or shall be a name whose declaration is visible in global scope because of a using-directive (3.4.3.2). The use of :: allows a global name to be referred to even if its identifier has been hidden (3.3.10).

5 A name prefixed by a *nested-name-specifier* that nominates an enumeration type shall represent an enumerator of that enumeration.

析构函数名称查找总结 6 If a *pseudo-destructor-name* (5.2.4) contains a *nested-name-specifier*, the *type-names* are looked up as types in the scope designated by the *nested-name-specifier*. Similarly, in a *qualified-id* of the form: *`nested-name-specifier<sub>opt</sub> class-name :: ~ class-name`* the second class-name is looked up in the same scope as the first.

[3.4.3/6]是对pseudo-destructor-name的查找规则, 但是[5.4.2/2]中包含着一个前提: `The left-hand side of the dot operator shall be of scalar type. The left-hand side of the arrow operator shall be of pointer to scalar type.` 当pseudo-destructor-name被用来表示非类类型的析构函数名的时候, 仅在满足此前提的情况下, 对pseudo-destructor-name实施[3.4.3/6]的查找才有意义.

*pseudo-destructor-name*

​ *nested-name-specifieropt type-name :: ~ type-name ​ nested-name-specifier template simple-template-id :: ~ type-name ​ nested-name-specifieropt ~ type-name ​ ~ decltype-specifier*

```cpp
namespace X
{
	struct C
	{
		typedef int i;
	};
	typedef float i;
};
typedef double i;
int main(int argc, char *argv[])
{
	extern int *p;
	p->X::C::~i(); // ok, 找到X::C::i;    
				   //1) 根据[5.2.4/2], p属于标量类型指针, 所以->后面的`X::C::~i`将被视为`pseudo-destructor-name`.
				   //2) 根据`pseudo-destructor-name`的语法形式, `X::C::~i`将被解析为 `nested-name-specifier ~ type-name`.  (而不能是 `nested-name-specifier type-name :: ~ type-name` 因为[5.4.2/2]明确要求后者中的两个`type-name`必须是相同的标量类型(scalar type), 而这里的C与int显然不符).    
				   //3) 根据[3.4.3/6], `X::C::~i` 中的`nested-name-specifier`部分指的是`X::C::`, 所以type-name `i` 将被查找在`X::C::`指定的作用域中, 所以找到X::C::i, 即使也存在::i, X::i, 但都不会被考虑.
}
```

```cpp
struct C
{
	typedef int I;
};
typedef double I;
int main(int argc, char *argv[])
{
	typedef double I;
	extern int *p;
	p->C::I::~I(); // ok, 找到C::I
				   //1) 根据[5.2.4/2], p属于标量类型指针, 所以->后面的`C::I::~I`将被视为`pseudo-destructor-name`.
				   //2) 根据`pseudo-destructor-name`的语法形式,`C::I::~I`中的第一个I为标量类型, 所以pseudo-destructor-name只能被解析为 `nested-name-specifier type-name :: ~ type-name`, 
				   //3) 根据[3.4.3/6], 其中的nested-name-specifier部分指的是`C::`, 所以~I中的I被查找在C::指定的作用域中, 可以找到C::I. 而局部作用域和全局作用域中的I都不会被考虑(即使找不到C::I).
}
```

```cpp
namespace X
{
	typedef long I;
}
typedef int I;
int main(int argc, char *argv[])
{
	extern int *p;
	p->X::~I(); //error
						  //[5.2.4/2]要求p应该和I是相同的类型, 但是由于[3.4.3/6]的查找规则只能找到X::I, 它是long类型, 而p是指向int类型对象的指针.
}
```

```cpp
namespace X
{
	typedef long I1;
	typedef int I2;
}
int main(int argc, char *argv[])
{
	extern int *p;
	p->X::I1::~I2(); //error 
					 //1) 根据[5.2.4/2], p属于标量类型指针, 所以->后面的`X::I1::~I2`将被视为`pseudo-destructor-name`.
					 //2) 根据`pseudo-destructor-name`的语法形式,`X::I1::~I2`中的I1为标量类型, 所以pseudo-destructor-name只能被解析为 `nested-name-specifier type-name :: ~ type-name`
					 //3) 因此根据[3.4.3/6], 查找到第一个type-name I1的类型为long, 第二个type-name I2的类型为int 
					 //4) 根据[5.2.4/2], 当这种形式的语法被作为pseudo-destructor-name时, 其中的两个type-name应该是相同的标量类型, 但是这里实际上两者不同, 所以这个pseudo-destructor-name不符合作为析构函数名的要求.
}
```

```cpp
namespace X
{
	typedef int I1;
}
typedef int I2;
extern int *p;
p->X::I1::~I2(); // error
								 // 因为X::I1是标量类型, 所以`X::I1::~I2`只能被解析成 `nested-name-specifier type-name :: ~ type-name`,
							   // 根据[3.4.3/6]的查找规则, I2应该在X::内查找, 但是I2仅声明在全局作用域
```

```cpp
typedef int I2;
int main(int argc, char *argv[])
{
	typedef int I1;
	extern int *p;
	p->I1::~I2(); // ok   
				  //1) 根据[5.2.4/2], p属于标量类型指针,所以->后面的`I1::~I2`将被视为`pseudo-destructor-name`
				  //2) 根据`pseudo-destructor-name`的语法形式,`I1::~I2`只能被解析为`type-name :: ~ type-name`
				  //3) [3.4.3/6]只定义了带有`nested-name-specifier`的pseudo-destructor-name查找规则, 而没有明确说明这种省略`nested-name-specifier`的pseudo-destructor-name的查找规则. 仅从实现(gcc 7.1/clang 9.0)上来看, I1和I2都是查找在p的上下文.
}
```

```cpp
struct A
{
	~A();
	typedef int I1, I2;
};
int main()
{
	A *p;
	p->I1::~I2(); // error
				  //1) 根据[5.2.4/2], p不属于标量类型指针, 所以->后面的`C::I::~I`不会被为`pseudo-destructor-name`, 因此按照pseudo-destructor-name的查找规则来查找`I1::~I2`是无意义的.
				  //2) 标准中并没有(没找到?)对这种语法(对象表达式p是类类型,但后面的qualified-id是标量类型)做出ill-formed的明确说明, 但是在[3.4.5]中枚举了所有被允许的情况, 所以反过来看,这种语法并不是被允许的, 而其中语法上最接近的就是[3.4.5/4], 所以, 在此暂且认为, 这是对[3.4.5/4]的一种误用. 
				  //	(从gcc编译错误上看(error: 'I1' is not a base of 'A'), 它在试图将I1作为A的基类看待, 这也验证了上面的分析)
}
```

下面几个示例中, 作为析构函数名的id-expression都是qualified-id, 且它们的语法的形式都符合: `nested-name-specifier<sub>opt</sub> class-name :: ~ class-name` 因此, 根据[3.4.3/6]的定义,其中第二个class-name的名称查找与第一个class-name查找在相同的作用域. 另外, 根据[3.4.5/4]的定义, 第一个class-name将被查找在两个作用域中: 首先是对象表达式(object expression)的类作用域中, 如果找到, 则使用它. 否则, 将查找整个前缀表达式(postfix-expression)的上下文.

到此, 针对这类的析构函数名的查找规则就被确定. 但是这规则中实际上还是有一些细节没有被明确说明, 参见[issue 399](http://www.open-std.org/jtc1/sc22/wg21/docs/cwg_active.html#399).

```cpp
struct A
{
	~A();
};
typedef A AB;
int main()
{
	AB *p;
	p->AB::~AB(); // ok, 对`~AB`中的AB查找与::之前的AB查找作用域相同, 而对::之前的AB查找首先是对象表达式p的类作用域A, 没有找到A::AB, 之后查找p的上下文, 找到::AB
}
```

```cpp
struct A
{
	typedef A C;
};
typedef A B;
void f(B *bp)
{
	bp->B::~B(); // okay B found by normal lookup 
	bp->C::~C();  // okay C found by class lookup
	bp->B::~C();  // B found by normal lookup C by class -- okay (in gcc 7.1 and clang 9.0)    
	bp->C::~B();  // C found by class lookup B by normal -- okay (in gcc 7.1 and clang 9.0)
}
```

```cpp
struct S
{
	struct C
	{
		~C() {}
	};
};
typedef S::C D;
int main()
{
	D *p;
	p->C::~D(); // valid in gcc 7.1 and clang 9.0
}
```

```cpp
struct S
{
	struct C
	{
		~C() {}
	};
};
typedef S::C D;
typedef S::C C;
int main()
{
	D *p;
	p->C::~D(); // valid in gcc 7.1 and clang 9.0
}
```

对`~type-name`作为析构函数名时的名称查找.[3.4.5/3] If the unqualified-id is ~type-name, the type-name is looked up in the context of the entire postfix-expression. If the type T of the object expression is of a class type C, the type-name is also looked up in the scope of class C. At least one of the lookups shall find a name that refers to (possibly cv-qualified) T.

```cpp
typedef int I;
void f()
{
	int *p;
	p->~I(); // ok, 在p的上下文中查找, 找到 ::I
}
```

```cpp
struct A
{
	typedef int I;
};
void f()
{
	A *p;
	p->~I(); // error
				   // 通过对p的上下文查找没有发现名字I, 之后对p的类作用域A中查找到A::I, 所以查找过程是没有错误的,
					 // 但是A::I是int类型, 根据[12.4/13], 要求~后边的名字type-nam或decltype-specifier必须代表这个类的名字(A),
				   // 而这里I是int, 所以这个构造是错误的.
}
```

```cpp
struct test
{
	struct some_name
	{
	};
};

int main(int argc, char *argv[])
{
	typedef test some_name;
	test t;
	t.~some_name();
	return 0;
}
// 在gcc 7.1 和 clang 9.0 上编译结果不一样
// clang: ok
// gcc: error: the type being destroyed is 'test', but the destructor refers to 'test::some_name'
// 按照[3.4.5/3], 应该是clang是对的
// https://groups.google.com/a/isocpp.org/forum/#!topic/std-discussion/rhHqhbrjdf0
```

```cpp
struct A
{
};
struct B
{
	struct A
	{
	};
	void f(::A *a);
};
void B::f(::A *a)
{
	a->~A(); // OK: lookup in *a finds the injected-class-name 
					 // (也许对a的上下文进行查找的时候可以优先找到B::A, 但它不符合[12.4/13]的要求, 所以才会继续查找a的类类型::A)
}
```

> C++98/03标准与C++11标准对这段的定义有很大区别, 具体演化过程参见: http://www.open-std.org/jtc1/sc22/wg21/docs/cwg_defects.html#244 http://www.open-std.org/jtc1/sc22/wg21/docs/cwg_active.html#399 http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n1938.pdf http://www.open-std.org/jtc1/sc22/wg21/docs/cwg_active.html#555
> 

### 3.4.3.1 Class members

类成员名称查找规则. 1 If the *nested-name-specifier* of a *qualified-id* nominates a class, the name specified after the *nested-name-specifier* is looked up in the scope of the class (10.2), except for the cases listed below. The name shall represent one or more members of that class or of one of its base classes (Clause 10). [ Note: A class member canbereferredtousingaqualified-idatanypointinitspotentialscope(3.3.7). —endnote]Theexceptions to the name lookup rule above are the following: — a destructor name is looked up as specified in 3.4.3; — a conversion-type-id of a conversion-function-id is looked up in the same manner as a conversion-type-id in a class member access (see 3.4.5); — the names in a template-argument of a template-id are looked up in the context in which the entire postfix-expression occurs. — the lookup for a name specified in a using-declaration (7.3.3) also finds class or enumeration names hidden within the same scope (3.3.10).

2 In a lookup in which the constructor is an acceptable lookup result and the *nested-name-specifier* nominates a class C: (C++14/17标准已经将这句话变更为: In a lookup in which function names are not ignored33 and the nested-name-specifier nominates a class C: 33) Lookups in which function names are ignored include names appearing in a nested-name-specifier, an elaborated-type-specifier, or a base-specifier.)

— if the name specified after the *nested-name-specifier*, when looked up in C, is the *injected-class-name* of C (Clause 9), or 如果被指定在nested-name-specifier后面的名字被查找在C中,并且是C的injected-class-name, 或 — in a using-declaration (7.3.3) that is a member-declaration, if the name specified after the nested-name-specifier is the same as the identifier or the simple-template-id’s template-name in the last component of the nested-name-specifier, the name is instead considered to name the constructor of class C. [ Note: For example, the constructor is not an acceptable lookup result in an elaborated-type-specifier so the constructor would not be used in place of the injected-class-name. — end note ] Such a constructor name shall be used only in the declarator-id of a declaration that names a constructor or in a using-declaration.

```cpp
struct A
{
	A();
};
struct B : public A
{
	B();
};
A::A() {}
B::B() {}
B::A ba; // object of type A
				 // A不是B的`injected-class-name` ${??}
A::A a;	 // error, A::A is not a type name        
				 //1) 根据[3.4.3.1/1], `A::A`中的第二个A应该查找在第一个A的作用域中,
				 //2) 根据C++14/17[3.4.3.1/2]的描述, 这里的 `A::A` 不是`nested-name-specifier`, 不是`elaborated-type-specifier`, 不是`base-specifier`, 
				 // 所以`A::A`属于函数名不被忽略的查找, 并且满足`the nested-name-specifier nominates a class C` 所以根据
				 // - if the name specified after the *nested-name-specifier*, when looked up in C, is the *injected-class-name* of C (Clause 9) ... the name is instead considered to name the constructor of class C.     
				 // 导致A::A被认为是析构函数, 而不是类型.
struct A::A a2; // object of type A                
							  // 这是`elaborated-type-specifier`, 不满足`In a lookup in which function names are not ignored`, 所以后续的描述对这不适用.
								// 根据[], struct可以用来指定 elaborated-type-specifier.
```

```cpp
struct A
{
};
int main()
{
	A::A a; // error, 原因同上.
}
```

```cpp
class foo
{
public:
	foo(){};
};
foo a;			 // legal
foo::foo b;		 // illegal
foo::foo::foo c; // illegal
```

```cpp
namespace fred
{
	struct A
	{
		A();
	};
}
fred::A::A::A::A() {} // ok
int main()
{
	fred::A a0;
	fred::A::A a1;		 // error
	fred::A::A::A a2;	 // error
	fred::A::A::A::A a3; // error

	return 0;
}
```

```cpp
struct A
{
};
int main(int argc, char *argv[])
{
	A a;
	a.A::A(); //不允许直接调用构造函数的标准[]
	return 0;
}
```

> http://www.open-std.org/jtc1/sc22/wg21/docs/cwg_defects.html#147 http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2512.html http://www.open-std.org/jtc1/sc22/wg21/docs/cwg_defects.html#1310 https://gcc.gnu.org/bugzilla/show_bug.cgi?id=11764 https://stackoverflow.com/questions/32006122/injected-class-name-compiler-discrepancy https://stackoverflow.com/questions/7394492/does-inheriting-constructors-work-with-templates-in-c0x https://stackoverflow.com/questions/29681449/program-being-compiled-differently-in-3-major-c-compilers-which-one-is-right http://www.open-std.org/jtc1/sc22/wg21/docs/cwg_defects.html#318
> 

3 A class member name hidden by a name in a nested declarative region or by the name of a derived class member can still be found if qualified by the name of its class followed by the :: operator.

```cpp
struct A
{
	void f()
	{
		int i;
		i = 0; // ok        A::i = 0; //ok
	}
	int i;
};
struct B : public A
{
	int i = 0;
	void g()
	{
		i = 0;	  // ok, refers to B::i;
		A::i = 1; // ok, refers to A::i;
	}
};
```

### 3.4.3.2 Namespace members