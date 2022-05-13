# std::addressof

Created: March 22, 2022 3:31 PM
Last Edited Time: April 6, 2022 11:13 AM

**C++14标准定义**

***template <class T> T* addressof(T& r) noexcept;***

Returns: The actual address of the object or function referenced by r, even in the presence of an overloaded operator&.  

**libstdc++ 的实现**

```cpp
template<typename _Tp>
inline _Tp*
__addressof(_Tp& __r) _GLIBCXX_NOEXCEPT
{
	return reinterpret_cast<_Tp*>(&const_cast<char&>(reinterpret_cast<const volatile char&>(__r)));
}

template<typename _Tp>
inline _Tp*
addressof(_Tp& __r) noexcept
{ return std::__addressof(__r); }
```

**为什么不直接使用取地址操作符**

因为 operator&() 允许重载, 如果直接使用取地址来实现的话, std::addressof 可能取不到真实的地址, 这将取决于被重载的 operator&() 如何实现.

```cpp
template <typename _Tp>
_Tp* __bad_addressof(_Tp& __r) { return &__r; }

struct T
{
	T* operator&() { return nullptr; }
};

T t;
__bad_addressof(t); // nullptr
```

**实现原理**

libstdc++ 对std::addressof的实现使用了三次转换

- reinterpret_cast<const volatile char&>(__r)
- const_cast<char&>(reinterpret_cast<const volatile char&>(__r))
- reinterpret_cast<_Tp*>(&const_cast<char&>(reinterpret_cast<const volatile char&>(__r)))

第一步转换的目的是将对象__r转换为 char & 类型的表示, 这样可以绕过对 _Tp::operator&() 的调用, 由于std::addressof的形参声明允许带有cv限定的实参, 所以模板参数推导后, __r 可能也是带有cv限定的类型, 为了reinterpret_cast 转换成功, 转换目标类型也都加上了 cv 限定, 就变成了 const volatile char &.

由于std::addressof要求的返回值类型为 _Tp*, 是不带cv限定的，而第一步为了__r 到 char& 的转换成功而添加了cv限定, 所以第二步转换的目的就是去掉第一步中添加的cv限定, 结果变成 char &

第三步首先对 char& 表示的 __r 执行取地址操作得到 char* 表示的 __r 的地址, 再使用 reinterpret_cast 将 char * 重新解释回 _Tp* .

[https://www.cxybb.com/article/dashuniuniu/79945154](https://www.cxybb.com/article/dashuniuniu/79945154)

[https://blog.csdn.net/dashuniuniu/article/details/79945154](https://blog.csdn.net/dashuniuniu/article/details/79945154)

[https://en.wikibooks.org/wiki/More_C%2B%2B_Idioms/Address_Of](https://en.wikibooks.org/wiki/More_C%2B%2B_Idioms/Address_Of)

[https://reviews.llvm.org/rGa58d430cace221fa3959962b8d374d968891aba3](https://reviews.llvm.org/rGa58d430cace221fa3959962b8d374d968891aba3)

[https://reviews.llvm.org/rG6cbd65d84df9a98493b8493536e3f9958f9275f8](https://reviews.llvm.org/rG6cbd65d84df9a98493b8493536e3f9958f9275f8)