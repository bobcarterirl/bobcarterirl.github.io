---
layout: post
title: HSL 1.1.1 – Array Initialization
---

Welcome to Homebrew Standard Library.  In this series, we reimplement the C++ Standard Library piece-by-piece, to understand it better. This is the second entry in the series. You can find the introduction and table of contents [here][IntroPost]. The full source code for this project is available on [GitHub][GitHubRepo]. In this post, we'll create a basic skeleton for our `array` class, and enable aggregate initialization and proper type deduction.

An [`array`][StdArray] is essentially a [C-style array][CStyleArray], with an STL-style interface. Like a C-style array, it has a fixed size and can contain only one data type. This sets it apart from the two most similar STL container types, `vector` (whose size can change) and `tuple` (which can hold multiple types). Its interface, meanwhile, is a combination of these latter two. We'll see exactly how in later posts. For now, let's begin our implementation as a simple wrapper for a C-style array. Remember that, to declare an array, we need to specify two things:

1. The type of its elements, and
2. Its length.

```cpp
T arr[N];   // Declares an array, named "arr", containing N elements of type T
```

It only makes sense then, that an `array` is a [class template][ClassTemplate] with these two as parameters:

In `array.hpp`:

```cpp
namespace hsl   // See the intro post for why this is "hsl" and not "std"
{

template<typename T, size_t N>
class array
{
private:
    T arr[N];
};

}
```

Two quick things we need to settle before moving on: First, [`size_t`][StdSizeT] is a C Standard Library type, meaning we need to redefine it. There are several headers which define `size_t`, but the one that's relevant right now is [`cstddef`][CStdDef] which contains only `size_t`, `ptrdiff_t` (which we'll need later) and a few other types, all of which are returned by built-in expressions. We'll need to create real definitions for them eventually, but for now, let's just import `size_t` into the `hsl` namespace:

In `cstddef.hpp`

```cpp
#include <cstddef>

namespace hsl
{

using std::size_t;

}
```

In `array.hpp`

```cpp
#include "cstddef.hpp"
// ...
```

Contrary to popular belief, there is no `size_t` defined in the global namespace. Most implementations define it there, anyway, but it isn't standard.

Second, the standard defines several [member types][StdArray] in `array`, and most of the rest of the class is defined in terms of these. We'll define these types as we need them, and then fill in whatever we don't use, at the end. For the moment we only need one -- `value_type`. With it, `array` now looks like this:

In `array.hpp`:

```cpp
template<typename T, size_t N>
class array
{
private:
    using value_type = T;

    value_type arr[N];
};
```

Now that we have the skeleton of our `array` implementation, we can consider [constructors and destructors][StdArray]. There aren't any. Or rather, they're implicitly defined. We can construct an `array` in four ways:

```cpp
hsl::array<int, 5> arr1;        // Default constructor
hsl::array arr2(arr1);          // Copy constructor
hsl::array arr3 = arr1;         // Copy assignment
hsl::array arr4{1, 2, 3, 4, 5}; // Aggregate initialization
```

But wait, [aggregate initialization][AggregateInitialization] doesn't work for classes with private data members. No problem -- we just make our underlying array public. It's not an elegant solution (since the standard doesn't let us access the underlying array), but it's the only one that works.

In `array.hpp`:

```cpp
template<typename T, size_t N>
class array
{
public:
    /...
    // Must be public for aggregate initialization to work.
    // Don't access directly!
    value_type arr[N];
};
```

Now that all the data members are public, if we try to define an `array` with the aggregate initialization line up there... it still won't compile. The reason is that we didn't provide template arguments, and the compiler can't deduce what they should be. Of course, all the information is there -- it's an array of 5 `int`s -- but the compiler can't figure that out using the normal rules. Prior to C++17, there would've been nothing we could do, but now we have [deduction guides][DeductionGuide] which let us give the compiler a nudge in the right direction.

A deduction guide looks like a constructor, that is, if a constructor could be declared with a [trailing return type][TrailingReturnType]:

```cpp
// This has to be outside the class definition
hsl::array(int, int, int, int, int) -> hsl::array<int, 5>;

hsl::array arr = {1, 2, 3, 4, 5};   // Success!
```

Anytime we define an `array` without specifying the template arguments, the compiler looks through all the deduction guides (some of which are defined implicitly) for the best match, just like if it were looking for the right function overload to use.

In this example, the compiler sees that we're trying to construct an `array` using 5 `int`s, and looks for a matching deduction guide. It finds the one we provided, and concludes that we meant to create an `array` containing 5 `int`s. It should be noted that, despite appearances, a deduction guide is not a constructor. After deducing the template arguments, the compiler forgets about which deduction guide it used, and only then determines how to initialize our `array`. In our case, of course, it uses aggregate initialization.

The standard specifies one [deduction guide][ArrayDeductionGuide] for `array`:

In `array.hpp`:

```cpp
// Below class definition
template<typename T, typename... U>
array(T, U...) -> array<T, sizeof...(U) + 1>;
```

This is an extension of the above, in the form of a [variadic template][VariadicTemplate]. It says that when we define an `array` using one or more arguments, the first argument determines the element type, and the number of elemtents determines the size. There's one problem with this. Can you see it? Consider what happens when we define this array:

```cpp
hsl::array arr = {1u, -5};
```

The first argument is an `unsigned int`, so that's what type `arr` will contain, but the second argument is negative, meaning it'll wind up as a very large, positive number (viz. `UINT_MAX - 4`). Ideally, we'd like to automatically find some type that suits all the arguments, but the standard would rather be pragmatic, and declares a program ill-formed if:

1. The compiler needs to use the deduction guide, and
2. The arguments aren't all the same type.

We can enforce this by changing the deduction guide to use [`enable_if`][StdEnableIf] and [`is_same`][StdIsSame]:

In `array.hpp`

```cpp
template<typename T, typename... U>
array(T, U...) -> array<
    enable_if_t<(is_same_v<T, U> && ...), T>,
    sizeof...(U) + 1>;
```

If all the arguments have the same type, `enable_if_t` resolves to `T`. If not, `enable_if_t` is undefined, and the program won't compile.

`enable_if_t` and `is_same_v` are defined in `type_traits`, so we'll need to take a moment to create aliases for them.

In `type_traits.hpp`:

```cpp
#include <type_traits>

namespace hsl
{

using std::enable_if_t;
using std::is_same_v;

}
```

In `array.hpp`:

```cpp
#include "cstddef.hpp"
#include "type_traits.hpp"
// ...
```

That's it: we should now be able to initialize an `array` in all four ways mentioned above, and the compiler should be able to deduce the template arguments we intended, even when we don't provide any. In the next post, we'll discuss how to access the elements of an `array`.

[IntroPost]: /hsl/2018/11/16/introduction.html
[GitHubRepo]: https://github.com/bobcarterirl/homebrew-standard-library
[StdArray]: https://en.cppreference.com/w/cpp/container/array
[CStyleArray]: https://en.cppreference.com/w/cpp/language/array
[ClassTemplate]: https://en.cppreference.com/w/cpp/language/class_template
[StdSizeT]: https://en.cppreference.com/w/cpp/types/size_t
[CStdDef]: https://en.cppreference.com/w/cpp/header/cstddef
[AggregateInitialization]: https://en.cppreference.com/w/cpp/language/aggregate_initialization
[DeductionGuide]: https://en.cppreference.com/w/cpp/language/class_template_argument_deduction
[TrailingReturnType]: https://en.cppreference.com/w/cpp/language/function
[ArrayDeductionGuide]: https://en.cppreference.com/w/cpp/container/array/deduction_guides
[VariadicTemplate]: https://en.cppreference.com/w/cpp/language/parameter_pack
[StdEnableIf]: https://en.cppreference.com/w/cpp/types/enable_if
[StdIsSame]: https://en.cppreference.com/w/cpp/types/is_same
