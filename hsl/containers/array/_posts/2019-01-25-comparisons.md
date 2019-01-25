---
title: HSL 1.1.6 – Array Comparisons
---

This post is part of a series. If you're new, see the intro post and table of contents, [here][IntroPost].

This week, we'll be implementing the comparison operators for array. There are six of them, but they're all dependent on only two STL algorithms.

## operator== ##
The [equality operator][StdArrayOperatorEquals] should give a good indication of how these will go. It should return true if every element of one array is equal to its counterpart in the other. We'll implement it using [`equal`][StdEqual] from the algorithm library, which does exactly this for any range.

In `array.hpp`:

```cpp
// Below tuple_size
template<typename T, size_t N>
bool operator== (const array<T, N>& lhs, const array<T, N>& rhs)
{
    return equal(lhs.begin(), lhs.end(), rhs.begin());
}
```

## operator< ##
The less-than operator works the same way, but uses [`lexicographical_compare`][StdLexicographicalCompare] instead of `equal`. Both that function and ours find the first element which differs between two ranges. It returns true if that element is less in the first range than in the second.

In `array.hpp`:

```cpp
// Below operator==
template<typename T, size_t N>
bool operator< (const array<T, N>& lhs, const array<T, N>& rhs)
{
    return lexicographical_compare(
        lhs.begin(), lhs.end(),
        rhs.begin(), rhs.end());
}
```

## operator!=, <=, >= > ##
The other operators are defined in terms of the two above. Not-equal returns exactly what you'd expect. Greater-than is the same as less-than, but with the arrays switched around. Less-than-or-equal-to returns not greater-than, and greater-than-or-equal-to returns non less-than.

```cpp
// Below operator<
template<typename T, size_t N>
bool operator!= (const array<T, N>& lhs, const array<T, N>& rhs)
{ return !(lhs == rhs); }

template<typename T, size_t N>
bool operator> (const array<T, N>& lhs, const array<T, N>& rhs)
{ return rhs < lhs; }

template<typename T, size_t N>
bool operator<= (const array<T, N>& lhs, const array<T, N>& rhs)
{ return !(lhs > rhs); }

template<typename T, size_t N>
bool operator>= (const array<T, N>& lhs, const array<T, N>& rhs)
{ return !(lhs < rhs); }
```

Next week, we'll finish up `array` by handling the special case that occurs for an `array` with zero elements.


[IntroPost]: /hsl/2018/11/16/introduction.html
[StdArrayOperatorEquals]: https://en.cppreference.com/w/cpp/container/array/operator_cmp
[StdEqual]: https://en.cppreference.com/w/cpp/algorithm/equal
[StdLexicographicalCompare]: https://en.cppreference.com/w/cpp/algorithm/lexicographical_compare
