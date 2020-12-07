---
title: "std::is_trivial implementation"
date: 2020-12-04T17:22:11+03:00
draft: true
categories:
  - C++
tags:
  - c++
  - stl
  - typetraits
---

Recently I've tried to write several data structures and I wanted different allocations depending on if the structure is [trivial](https://en.cppreference.com/w/cpp/named_req/TrivialType) or not.
I know that there is a [std::trivial](https://en.cppreference.com/w/cpp/types/is_trivial), but I don't want to use std, so I've started looking through the different type_traits implementation.
And found that in each STL implementation for GCC, Clang and Microsoft are using compiler intrinsics.
Here is a full list of them: https://clang.llvm.org/docs/LanguageExtensions.html#type-trait-primitives  
In that list, you can see which of the intrinsics can be used across different compilers.
Btw, I noticed that [MVC's documentation for type traits](https://docs.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2010/ms177194(v=vs.100)?redirectedfrom=MSDN) is outdated for the C++17, and later,
I guess that some intrinsics that were starting with "**__has**" prefix will start with "**__is**" now or deprecated.

So here is my implementation of **is_trivial**:  

```c++
    template <typename T, T v>
    struct integral_constant
    {
        static constexpr const T value = v;
    };

    template <bool Bool>
    using bool_constant = integral_constant<bool, Bool>;

    template <typename T>
    struct is_trivial : bool_constant<__is_trivial(T)>
    {};
```

[GCC's typetraits](https://github.com/gcc-mirror/gcc/blob/master/libstdc++-v3/include/std/type_traits):  

```c++
  /// is_trivial
  template<typename _Tp>
    struct is_trivial
    : public integral_constant<bool, __is_trivial(_Tp)>
    {
      static_assert(std::__is_complete_or_unbounded(__type_identity<_Tp>{}),
	"template argument must be a complete class or an unbounded array");
    };
```

[Clang's typetraits](https://github.com/llvm/llvm-project/blob/master/libcxx/include/type_traits):  
```c++
// is_trivial;

template <class _Tp> struct _LIBCPP_TEMPLATE_VIS is_trivial
#if __has_feature(is_trivial) || defined(_LIBCPP_COMPILER_GCC)
    : public integral_constant<bool, __is_trivial(_Tp)>
#else
    : integral_constant<bool, is_trivially_copyable<_Tp>::value &&
                                 is_trivially_default_constructible<_Tp>::value>
#endif
    {};

```

  
[MVC's typetraits](https://github.com/microsoft/STL/blob/master/stl/inc/type_traits):  
```c++
// STRUCT TEMPLATE is_trivial
#if 1 // TRANSITION, VSO-119526 and LLVM-41915
template <class _Ty>
struct is_trivial : bool_constant<__is_trivially_constructible(_Ty) && __is_trivially_copyable(_Ty)> {
    // determine whether _Ty is a trivial type
};
```
Something similar can be found in EASTL (type_pod.h) and Boost libraries. All versions seem pretty identical. In Clang's version, there are some backward compatibility checks.
Implementations of **integral_constant**, **bool_constant** you can find from the links above.
And for C++17 we can use the inlined variables feature and make **is_trivial_v**.  

```c++
template <typename T>
inline constexpr bool is_trivial_v = is_trivial<T>::value;
```
