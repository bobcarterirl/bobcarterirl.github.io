---
layout: post
title: HSL 1.1.2 – Array Element Access
---

Welcome to Homebrew Standard Library.  In this series, we reimplement the C++ Standard Library piece-by-piece, to understand it better. You can find the introduction and table of contents [here][IntroPost]. The full source code for this project is available on [GitHub][GitHubRepo].

At this point, we can instantiate our `array` class, but it isn't very useful, yet, since we can't access its elements. (I mean, we can, but we're not supposed to.) In this post, we'll fix that.

As mentioned in the last post, `array`'s interface takes after both `vector` and `tuple`. [`vector`][StdVector] has five ways to access its elements:

```cpp
std::vector<int> vec{1, 2, 3, 4, 5};

vec[0];         // Returns refrence to given element
vec.at(0);      // Same, but w/ bounds checking
vec.front();    // Returns reference to first element
vec.back();     // Same, but for last element
vec.data();     // Returns pointer to underlying array
```

[`tuple`][StdTuple], meanwhile, has two:

```cpp
std::tuple tup{1, true, "three"};

std::get<0>(tup);   // Returns reference to given element
std::get<int>(tup); // Returns reference to element w/ given type
```

That last one only works for `tuple`s where each element has a different type. In an `array`, every element has the same type, so it doesn't apply. But every other way of accessing the elements of a `vector` or `tuple` also works for an `array`:

```cpp
hsl::array arr{1, 2, 3, 4, 5};

arr[0];
arr.at(0);
arr.front();
arr.back();
arr.data();

hsl::get<0>(arr);
```

We'll begin with the subscript operator.

## Subscript operator

`array`'s [subscript operator][StdArraySubscriptOperator] is nothing more than a wrapper for the subscript operator on the underlying array. It returns a [reference][Reference] to the element at the given index. There are two overloads: one for normal circumstances, and the other for `const array`s. Both versions are [`constexpr`][Constexpr], allowing them to be used at compile-time. This is possible because `array` meets the qualifications for a [literal type][LiteralType] (as long as its `value_type` is literal).

In `array.hpp`:

```cpp
template<typename T, size_t N>
class array
{
public:
    // ...
    using size_type         = size_t;
    using reference         = value_type&;
    using const_reference   = const value_type&;
    // ...

    constexpr reference operator[] (size_type n) { return arr[n]; }
    constexpr const_reference operator[] (size_type n) const { return arr[n]; }
};
```

## at
The [`at`][StdArrayAt] member function is almost identical, except that it throws an [`out_of_range`][StdOutOfRange] exception when we try to go off the end of the `array`. Like the subscript operator, it has two overloads (one `const` and one not) and both are `constexpr`.

In `array.hpp`:

```cpp
#include "stdexcept.hpp"
//...
template<typename T, size_t N>
class array
{
public:
    // ...
    constexpr reference at(size_type n)
    {
       if (n >= N) throw out_of_range("hsl::array::at");
       return arr[n];
    }

    constexpr const_reference at(size_type n) const
    {
       if (n >= N) throw out_of_range("hsl::array::at");
       return arr[n];
    }
};
```

Since `out_of_range` is a Standard Library type, we need to create an alias for it. I've already shown our method for doing that, so I'll omit the code snippet.

## front and back

[`front`][StdArrayFront] and [`back`][StdArrayBack] follow the same pattern as the last two member functions.

In `array.hpp`:

```cpp
template<typename T, size_t N>
class array
{
public:
    // ...
    constexpr reference front() { return arr[0]; }
    constexpr const_reference front() const{ return arr[0]; }

    constexpr reference back() { return arr[N-1]; }
    constexpr const_reference back() const { return arr[N-1]; }
};
```

## data

[`data`][StdArrayData] follows the same pattern, too, but also has a [`noexcept`][NoEXCEPT] Specifier.

In `array.hpp`

```cpp
template<typename T, size_t N>
class array
{
public:
    // ...
    using pointer       = T*;
    using const_pointer = const T*;
    // ...

    constexpr pointer data() noexcept { return arr; }
    constexpr const_pointer data() const noexcept { return arr; }
};
```

## get

Lastly, we come to the [`get`][StdGet] function. There are two things that set this function apart from the above (besides the fact it's not a member function):

1. It does bounds-checking, but, unlike `at`, it does it at compile-time, using a [`static_assert`][StaticAssert]. This is possible because both the index and `array` size are passed in as template arguments.
2. It has an additional pair of overloads, which return an [r-value reference][Reference] and a `const` r-value reference, respectively.

In `array.hpp`:

```cpp
// Below deduction guide
template<typename I, typename T, size_t N>
constexpr T& get(array<T, N>& arr) noexcept
{
    static_assert(I < N, "Index out of bounds!");
    return arr[I];
}

template<typename I, typename T, size_t N>
constexpr T&& get(array<T, N>&& arr) noexcept
{
    static_assert(I < N, "Index out of bounds!");
    return move(arr[I]);
}

template<typename I, typename T, size_t N>
constexpr const T& get(array<T, N>& arr) noexcept
{
    static_assert(I < N, "Index out of bounds!");
    return arr[I];
}

template<typename I, typename T, size_t N>
constexpr const T&& get(array<T, N>&& arr) noexcept
{
    static_assert(I < N, "Index out of bounds!");
    return move(arr[I]);
}
```

In the next post, we'll implement capacity checking.

[IntroPost]: /hsl/2018/11/16/introduction.html
[GitHubRepo]: https://github.com/bobcarterirl/homebrew-standard-library
[StdVector]: https://en.cppreference.com/w/cpp/container/vector
[StdTuple]: https://en.cppreference.com/w/cpp/utility/tuple
[StdArraySubscriptOperator]: https://en.cppreference.com/w/cpp/container/array/operator_at
[Constexpr]: https://en.cppreference.com/w/cpp/language/constexpr
[LiteralType]: https://en.cppreference.com/w/cpp/named_req/LiteralType
[StdArrayAt]: https://en.cppreference.com/w/cpp/container/array/at
[StdOutOfRange]: https://en.cppreference.com/w/cpp/error/out_of_range
[StdArrayFront]: https://en.cppreference.com/w/cpp/container/array/front
[StdArrayBack]: https://en.cppreference.com/w/cpp/container/array/back
[StdArrayData]: https://en.cppreference.com/w/cpp/container/array/data
[Noexcept]: https://en.cppreference.com/w/cpp/language/noexcept_spec
[StdGet]: https://en.cppreference.com/w/cpp/container/array/get
[StaticAssert]: https://en.cppreference.com/w/cpp/language/static_assert
[Reference]: https://en.cppreference.com/w/cpp/language/reference
