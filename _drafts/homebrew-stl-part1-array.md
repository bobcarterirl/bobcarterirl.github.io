---
layout: post
title: "Homebrew Standard Library – Part 1: Array"
---

This is the first post in a series where I reimplement the C++ STL, for learning puposes. I decided to start with `std::array` since I figured it would be the simplest to implement. The full code is available on [GitHub][homebrew-stl].

The `std::array` (or `hsl::array` in my case, to avoid name conflicts) is very similar to a C-style array. They both have a fixed size (unlike a `std::vector`) and can contain only one data type (unlike a `std::tuple`). But `std::array`'s interface is a blend of those of `std::vector` and `std::tuple`.

It's elements can be accessed in all the same ways as `std::vector`. Like a C-style array, this includes the subscript operator, but it also includes the `at` method, which adds bounds-checking -- an advantage over a C-style array. They can also be accessed with the `std::get` function like `std::tuple`. This also performs bounds checking, but it does so at compile-time.

It has the standard methods for getting iterators to `begin` and `end`.

It has `empty`, `size` and `max_size` methods like `std::vector`. `max_size` always returns the same thing as `size`, and `empty` only returns true if `size() == 0`. They exist only to make the interface more `std::vector`-like. The `std::tuple_size` helper class also works. Like a C-style array, you can also find the size with `sizeof(my_array) / sizeof(my_array[0])` if you want to do things the tedious way. This works because the only data a `std::array` stores is its underlying C-style array. The size is a template argument, and so, doesn't take up any memory.

All the same member types are defined for `std::array` as for `std::vector`. And, for consistency's sake, the member types can also be found using the `std::tuple_element` helper class.

It also has `std::vector`s swap method.

`std::array`s can be compared with relational operators -- another advantage over C-style arrays.

The only method `std::array` has to itself is `fill`, which does exactly what the name implies.

As for constructors, it only has what's implitly defined. This means a default constructor, copy, copy assignment, and, interestingly, aggregate initialization. This means it can be constructed using an initializer-list, like a `std::vector` or C-style array, but without the overhead of the former.

`std::array`s can be returned from functions.

* * *

The one downside is that a `std::array`'s size must be known at compile-time, while a C-style array's size can be determined at run-time. If you find yourself needing this, use a `std::vector` even if you know you won't have to resize it, the benefits of an STL container outweigh the fact this allocates from the heap.

```cpp
void foo(int x)
{
    int arr1[x];                // OK
    std::array<int, x> arr2;    // invalid, since x could be anything
}
```


[homebrew-stl]: https://github.com/bobcarterirl/homebrew-stl
