# Declarations and definitions

Created: March 1, 2022 5:19 PM
Last Edited Time: April 6, 2022 12:02 PM

### 3.1 Declarations and definitions

1 A declaration (Clause 7) may introduce one or more names into a translation unit or redeclare names introduced by previous declarations. If so, the declaration specifies the interpretation and attributes of these names. A declaration may also have effects including:

— a static assertion (Clause 7), — controlling template instantiation (14.7.2), — use of attributes (Clause 7), and

— nothing (in the case of an *empty-declaration*).

### declaration

A declaration, it declares a function without specifying the function’s body

```cpp
int f(int);  //declares f
int f(int x) { return x+a; }    //defines f and defines x
```

it contains the **extern** specifier (7.1.1) or a *linkage-specification* (7.5) and neither an initializer nor a function- body

```cpp
extern int a;           //declares a
int a;                  //defines a
extern const int c;     //declares c
extern const int c = 1; //defines c
extern X anotherX;      //declares anotherX
X anX;                  //defines anX
extern void f();        //declares f
extern void f(){ }      //defines f
```

it declares a static data member in a class definition (9.2, 9.4),

```cpp
struct X {
	int x;          //defines non-static data member x
	static int y;   //declares static data member y
};
```

it is a class name declaration

```cpp
struct S { int a; int b; }; //defines S, S::a, and S::b
struct S;                   //declares S
```

it is an opaque-enum-declaration (7.2) *opaque-enum-declaration:* *enum-key attribute-specifier-seqopt identifier enum-baseopt ;*

```cpp
enum E : int ;              //declares E
enum E : int { A, B, C };   //defines E, E::A, E::B, E::C
```

it is a template-parameter (14.1)

```
template<typename T>    //declares T
```

it is a parameter-declaration (8.3.5) in a function declarator that is not the declarator of a function-definition

```cpp
int f(int x); // declares f and declares x
int f(int x) { return x+a; }    //defines f and defines x
```

it is a typedef declaration (7.1.3)

```cpp
typedef int Int //declares T
```

an alias-declaration (7.1.3),

```cpp
using T = S;    //declares T
```

a using-declaration (7.3.3)

```cpp
namespace N { int d; }  //defines N and N::d
namespace N1 = N;       //defines N1
using N::d;             //declares d
```

a static_assert-declaration (Clause 7) an attribute-declaration (Clause 7) an empty-declaration (Clause 7)

a using-directive (7.3.4).

```cpp
using namespace std;
```

[ Note: A class name can also be implicitly declared by an elaborated-type-specifier (7.1.6.3). — end note ]

### definition

A program is ill-formed if the definition of any object gives the object an incomplete type (3.9).