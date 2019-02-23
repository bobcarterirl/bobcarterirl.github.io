Now that our `array` implementation is complete, we can move on to another container, namely `vector`. In this post, we'll create the basic skeleton of our `vector` class, and then, in later posts, fill it in. That's almost the same as with `array`, except I crammed array initialization into the first post, because it was so simple. This time, we'll actually need to write constructors, ourselves, so I've put that off until after we get all the easy stuff done.

As always, when starting a new class (or anything, really) it's a good idea to make clear exactly what we need to do. A `vector` is very similar to an `array`: it contains a sequence of elements, all of which have the same type, and which are contained in a contiguous chunk of memory (unlike a `list`) The difference is that its size isn't fixed.

This has two immediate consequences: the first is that a `vector`'s size can't be a template parameter, though the element type can. The second is that the underlying array has to be dynamically allocated. The STL likes do handle this through an allocator. The allocator's type can vary, so we need to make it a template parameter, but it defaults to `allocator`, which is a wrapper for `new` and `delete`. Even with an allocator, we can use a `unique_ptr` to manage the underlying array's lifetime (with one caveat, which will come up in a later post). This will save us some effort -- it'll keep us from having to write a destructor, at any rate. With all this in mind, we can begin our `vector` implementation:

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

Up to now, I've been defining member types as they became needed. This makes it all too easy to forget one (I'm not sure I ever got around to defining `array::difference_type`) and, to be honest, using `value_type` instead of `T`, everywhere, is really just verbosity for verbosity's sake. So, before I go, let's just go through and define all the member types.

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
    using pointer = typename AT::pointer;   // Almost certainly the same as T*
    using const_pointer = typename AT::const_pointer;
    using iterator = pointer;               // Pointers still work fine
    using const_iterator = const_pointer;
    using reverse_iterator = hsl::reverse_iterator<iterator>;
    using const_reverse_iterator = hsl::reverse_iterator<const_iterator>
    // ...
};
```
