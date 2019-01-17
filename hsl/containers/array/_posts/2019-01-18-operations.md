---
title: HSL 1.1.5 – Array Operations
---

This post is part of a series. If you're new, see the intro post and table of contents, [here][IntroPost].

We're nearly done with `array`. This week, we're writing the last two member functions: `fill` and `swap`.

## fill ##

The [`fill`][StdArrayFill] method sets every element in the array to a given value. Interestingly, it happens to be the only member function of `array` that doesn't exist in `vector`. We could implement it as a for-loop: it wouldn't be difficult, and originally, this was how I did it. But, in the interest of not repeating myself, I've decided to use STL-algorithms wherever possible. We'll implement those algorithms ourselves soon enough, but for now, we can leverage them to make our job just a bit easier. In this case, we need [`fill_n`][StdFillN].

In `array.hpp`:

```cpp
#include "algorithm.hpp"    // Create file and add "using std::fill_n;"
//...
template<typename T, size_t N>
class array
{
public:
    // ...
    void fill(const_reference val) { fill_n(arr, N, val); }
};
```

## swap ##

[`swap`][StdArraySwap] swaps the contents of two arrays. As with `fill` we can implement it with an algorithm, but this time the [algorithm][StdSwap] is in the [`<utility>`][Utility] header. `swap` is `noexcept`, but only if we can guarantee that swapping the individual elements won't throw an exception. We can specify this using [`is_nothrow_swappable`][StdIsNothrowSwappable], which is defined in `<type_traits>`.

In `array.hpp`:

```cpp
#include "type_traits.hpp"  // Add "using std::is_nothrow_swappable;"
#include "utility.hpp"      // Add "using std::swap;"
//...
template<typename T, size_t N>
class array
{
public:
    // ...
    void swap(array<T, N>& other)
    noexcept(is_nothrow_swappable<T>::value)
    {
        hsl::swap(arr, other.arr);
    }
};
```


## swap specialization ##

The loose function version of [`swap`][StdSwapArray] is a template, and we need to create a partial specialization, to handle our `array` class. Since we've already written the member function version, this is trivial. This `swap` function is `noexcept` if whenever the member function version is. We can specify this using the [`noexcept operator`][NoexceptOperator].

In `array.hpp`:

```cpp
// Below get
template<typename T, size_t N>
constexpr void swap(array<T, N>& lhs, array<T, N>& rhs)
noexcept(noexcept(lhs.swap(rhs)))
{
    lhs.swap(rhs);
}
```

We're almost done with `array`. Next week, we'll implement comparisons.

[IntroPost]: /hsl/2018/11/16/introduction.html
[StdArrayFill]: https://en.cppreference.com/w/cpp/container/array/fill
[StdFillN]: https://en.cppreference.com/w/cpp/algorithm/fill_n
[StdArraySwap]: https://en.cppreference.com/w/cpp/container/array/swap
[StdSwap]: https://en.cppreference.com/w/cpp/algorithm/swap
[Utility]: https://en.cppreference.com/w/cpp/utility
[StdIsNothrowSwappable]: https://en.cppreference.com/w/cpp/types/is_swappable
[NoexceptOperator]: https://en.cppreference.com/w/cpp/language/noexcept
[StdSwapArray]: https://en.cppreference.com/w/cpp/container/array/swap2
