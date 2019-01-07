---
title: HSL 1.1.4 – Array Iterators
---

Welcome to Homebrew Standard Library. In this series, we reimplement the C++ Standard Library piece-by-piece, to understand it better. You can find the introduction and table of contents [here][IntroPost]. The full source code for this project is available on [GitHub][GitHubRepo].

As mentioned last time, there's one small feature of C-style arrays which we haven't yet implemented, namely [`std::begin`][StdBegin] and [`std::end`][StdEnd]. We won't be implementing those today, and probably not any time soon, either. But we will implement `array`'s [`begin`][StdArrayBegin] and [`end`][StdArrayEnd] member functions, which is enough to make their loose function equivalents work.

If `std::begin` is passed a C-style array, it returns a pointer to the beginning of the array. If it's passed anything else, it merely calls the `begin` member function on whatever's passed into it. So

```cpp
hsl::array<int, 5> arr;

hsl::begin(arr);    // is the same as
arr.begin();
```

`end`, `rbegin`, `rend`, `cbegin`, `cend`, `crbegin`, and `crend` work likewise. (Incidentally, C++17 adds `size`, `empty`, and `data` functions which work the same way.) So, to make these functions work, we only need to define the appropriate member functions.

## begin and end

Before we implement the iterator member functions, we need to decide how we're going to implement `array` iterators, themselves. We could use a class, and for certain containers we'll have to, but for `array` that would only be over-complicating things. As mentioned above, calling `begin` or `end` on a C-style array returns a pointer, and, as it turns out, a pointer is perfectly sufficient for our needs.

As with element access, `begin` and `end` have two overloads, each: one for `const array`s, and one for non-`const`. The implementation is trivial, we just want to call `begin`/`end` (the loose function versions) on the underlying array. There's no reason to duplicate our work, after all.

In `array.hpp`:

```cpp
#include "iterator.hpp"
// ...
template<typename T, size_t N>
class array
{
public:
    // ...
    using iterator = pointer;
    using const_iterator = const_pointer;
    // ...
    constexpr iterator begin() noexcept { return hsl::begin(arr); }
    constexpr const_iterator begin() const noexcept { return hsl::begin(arr); }

    constexpr iterator end() noexcept { return hsl::end(arr); }
    constexpr const_iterator end() const noexcept { return hsl::end(arr); }
    // ...
};
```

Remember to create aliases for `std::begin`, `std::end` in `iterator.hpp`.

## rbegin and rend

[`rbegin`][StdArrayRBegin] and [`rend`][StdArrayREnd] are almost the same, except that they return [`reverse_iterator`][StdReverseIterator]s. `reverse_iterator` is defined in [`<iterator>`][IteratorHeader], all we need to do is instantiate the template with a normal iterator type.

In `array.hpp`:

```cpp
template<typename T, size_t N>
class array
{
public:
    // ...
    using reverse_iterator          = hsl::reverse_iterator<iterator>;
    using const_reverse_iterator    = hsl::reverse_iterator<const_iterator>;
    // ...
    constexpr reverse_iterator rbegin() noexcept { return hsl::rbegin(arr); }
    constexpr const_reverse_iterator rbegin() const noexcept
    {
        return hsl::rbegin(arr);
    }

    constexpr reverse_iterator rend() noexcept { return hsl::rend(arr); }
    constexpr const_reverse_iterator rend() const noexcept
    {
        return hsl::rend(arr);
    }
    // ...
};
```

Add aliases for `std::reverse_iterator`, [`std::rbegin`][StdRBegin], and [`std::rend`][StdREnd] to `iterator.hpp`.

## cbegin, cend, crbegin, and crend

The remaining iterator member functions are just like the above, except they only have a `const` overload.

In `array.hpp`:

```cpp
template<typename T, size_t N>
class array
{
public:
    // ...
    constexpr const_iterator cbegin() const noexcept
    {
        return hsl::cbegin(arr);
    }
    constexpr const_iterator cend() const noexcept { return hsl::cend(arr); }

    constexpr const_reverse_iterator crbegin() const noexcept
    {
        return hsl::crbegin(arr);
    }

    constexpr const_reverse_iterator crend() const noexcept
    {
        return hsl::crend(arr);
    }
    // ...
};
```

Add aliases for `std::cbegin`, `std::cend`, `std::crbegin`, and `std::crend` to `iterator.hpp`, and we're done.

[IntroPost]: https://bobcarterirl.github.io/hsl/2018/11/16/introduction.html
[GitHubRepo]: https://github.com/bobcarterirl/homebrew-standard-library
[StdBegin]: https://en.cppreference.com/w/cpp/iterator/begin
[StdEnd]: https://en.cppreference.com/w/cpp/iterator/end
[StdArrayBegin]: https://en.cppreference.com/w/cpp/container/array/begin
[StdArrayEnd]: https://en.cppreference.com/w/cpp/container/array/end
[StdArrayRBegin]: https://en.cppreference.com/w/cpp/container/array/rbegin
[StdArrayREnd]: https://en.cppreference.com/w/cpp/container/array/rend
[StdReverseIterator]: https://en.cppreference.com/w/cpp/iterator/reverse_iterator
[IteratorHeader]: https://en.cppreference.com/w/cpp/iterator
[StdRBegin]: https://en.cppreference.com/w/cpp/iterator/rbegin
[StdREnd]: https://en.cppreference.com/w/cpp/iterator/rend

