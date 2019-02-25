---
title: HSL 1.2.1 – Vector Skeleton
---


Now that our `array` implementation is complete, we can move on to the next
container, [`vector`][std::vector]. In this post, we'll create the basic
skeleton of our `vector` class, and then, in later posts, fill it in. That's
almost the same as with `array`, except I crammed array initialization into the
first post, because it was so simple. This time, we'll actually need to write
constructors, ourselves, so I've put that off until after we get all the easy
stuff done.

As always, when starting a new class (or anything, really) it's a good idea to
make clear exactly what our goal is. A `vector` is very similar to an `array`:
it contains a sequence of elements, all of which have the same type, and which
are contained in a contiguous chunk of memory (unlike a `list`) The difference
is that its size isn't fixed.

This has two immediate consequences: the first is that a `vector`'s size can't
be a template parameter, though the element type can. The second is that the
underlying array has to be dynamically allocated. The STL likes do handle this
through an [allocator][Allocator requirement]. The allocator's type can vary,
so we need to make it a template parameter, but it defaults to
[`allocator`][std::allocator], which is a wrapper for [`new`][new operator] and
[`delete`][delete operator]. Even with an allocator, we can use a
[`unique_ptr`][std::unique_ptr] to manage the underlying array's lifetime (with
one caveat, which will come up in a later post). This will save us some effort
-- it'll keep us from having to write a destructor, at any rate. With all this
in mind, we can begin our `vector` implementation:

In `vector.hpp`:

```cpp
#include "memory.hpp"   // Create this file, and add "using std::unique_ptr;"
                        // and "using std::allocator;"
namespace hsl
{

template<typename T, typename Alloc = allocator<T>>
class vector
{
private:
    unique_ptr<T[]> arr;
};

}
```


## Member types ##

Up to now, I've been defining member types as they became needed. This makes it
all too easy to forget one (I'm not sure I ever got around to defining
`array::difference_type`) and, to be honest, using `value_type` instead of `T`,
everywhere, is really just verbosity for verbosity's sake. So, before I go,
let's just go through and define all the member types.

In `vector.hpp`:

```cpp
#include "cstddef.hpp"  // For size_t and ptrdiff_t
// ...
template<typename T, typename Alloc = allocator<T>>
class vector
{
private:
    using AT = allocator_traits<Alloc>;     // Saves some space

public:
    using value_type = T;
    using allocator_type = Alloc;
    using size_type = size_t;
    using difference_type = ptrdiff_t;
    using reference = T&;
    using const_reference = const T&;
    using pointer = typename AT::pointer;
    using const_pointer = typename AT::const_pointer;
    using iterator = pointer;
    using const_iterator = const_pointer;
    using reverse_iterator = hsl::reverse_iterator<iterator>;
    using const_reverse_iterator = hsl::reverse_iterator<const_iterator>
    // ...
};
```

The above is pretty straightforward. Just like in `array`, pointers work just fine as iterators. The only odd thing is that we used `allocator_traits<Alloc>::pointer` for the `pointer` type. `AT::pointer` (as I've shortened it to) is almost always just defined as `T*`, but, in some cases, it could be what's called a fancy pointer. As for why it's `allocator_traits<Alloc>::pointer` instead of the more terse `Alloc::pointer`, the STL always access allocators through a traits class, I suppose so that some of the requirements for an allocator can be optional, without demanding that all allocators inherit from `allocator`.

[std::vector]: https://en.cppreference.com/w/cpp/container/vector
[Allocator requirement]: https://en.cppreference.com/w/cpp/named_req/Allocator
[std::allocator]: https://en.cppreference.com/w/cpp/memory/allocator
[new operator]: https://en.cppreference.com/w/cpp/language/new
[delete operator]: https://en.cppreference.com/w/cpp/language/delete
[std::unique_ptr]: https://en.cppreference.com/w/cpp/memory/unique_ptr
[std::allocator_traits]: https://en.cppreference.com/w/cpp/memory/allocator_traits
