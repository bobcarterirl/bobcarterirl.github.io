Now that our `array` implementation is complete, we can move on to another container, namely `vector`. In this post, we'll create the basic skeleton of our `vector` class, and then, in later posts, fill it in. That's almost the same as with `array`, except I crammed array initialization into the first post, because it was so simple. This time, we'll actually need to write constructors, ourselves, so I've put that off until after we get all the easy stuff done.

As always, when starting a new class (or anything, really) it's a good idea to make clear exactly what we need to do. A `vector` is very similar to an `array`: it contains a sequence of elements, all of which have the same type, and which are contained in a contiguous chunk of memory (unlike a `list`) The difference is that its size isn't fixed. This means two things: the first is that a `vector`'s size can't be a template parameter, though the element type can. The second is that the underlying array has to be dynamically allocated. We can use a `unique_ptr` to manage the underlying array's lifetime. This will save us some effort -- it'll keep us from having to write a destructor, at any rate. Knowing this, we can begin our `vector` implementation with a very simple skeleton.

In `vector.hpp`:

```cpp
#include "memory.hpp" // Create this file, and add using std::unique_ptr

namespace hsl
{

template<typename T>
class vector
{
private:
    unique_ptr<T[]> arr;
};

}
```


## Allocator ##

Since we know that we'll be dynamically allocating our underlying array, we should at least consider how we're going to do that. What I mean is that the Standard Library likes to use allocator classes to handle dynamic allocation. The type of allocator is a template parameter, and it defaults to `hsl::allocator<T>` (which is a wrapper for `new` and `delete`).

In `vector.hpp`:

```cpp
#include "memory.hpp" // Add using std::allocator
//...
template<typename T, typename Alloc = allocator<T>>
class vector
{
private:
    Alloc alloc;
    unique_ptr<T[]> arr;
};
```

The preferred way of allocating or freeing memory with an allocator is by using `hsl::allocator_traits`.

```cpp
hsl::allocator<int> alloc;
hsl::size_t n = 5;

using AT = hsl::allocator_traits<hsl::allocator<int>>;

// Unlike new, allocate doesn't construct anything
int* arr = AT::allocate(alloc, n);
for (hsl::size_t i = 0; i < n; ++i)
{
    AT::construct(alloc, arr + i, i + 1);
}

// Nor does deallocate destroy anything
for (hsl::size_t i = 0; i < n; ++i)
{
    // Of course, destroying ints is pointless, but it would be necessary, if
    // this array contained something else.
    AT::destroy(alloc, arr + i);
}
AT::deallocate(alloc, arr, n);
```

But this poses a small problem for us, since, by default, `unique_ptr` uses `default_delete` (i.e. the `delete` operator) to deallocate its memory. Thankfully, a `unique_ptr` can optionally use a custom deleter for that purpose. A deleter is a functor that takes a `unique_ptr`'s underlying pointer as an argument. When we construct the `unique_ptr` to manage our underlying array, we can also pass in a lambda to destroy its elements and deallocate its memory.

```cpp
auto del = [&](int* p)
{
    for (hsl::size_t i = 0; i < n; ++i)
    {
        AT::destroy(alloc, arr + i);
    }
    AT::deallocate(alloc, arr, n);
};

hsl::unique_ptr<int> arr(AT::allocate(alloc, n), del);
```

With this in mind, we can create a member function (we'll call it `allocate`) to abstract away the process. `allocate` will take the number of elements we want to allocate and return a `unique_ptr` to the new array.


In `vector.hpp`:

```cpp
#include <functional> // Create this file, and add using std::function
//...
template<typename T, typename Alloc = allocator<T>>
class vector
{
private:
    using AT = allocator_traits<Alloc>;
    // unique_ptr now takes the deleter type as a 2nd template param
    using Array_type = unique_ptr<T[], function<void(T*)>>;

    Alloc alloc;
    Array_type arr;

    Array_type allocate(size_t n)
    {
        auto del = [this, n](T* p)
        {
            for (size_t i = 0; i < n; ++i)
            {
                AT::destroy(this->alloc, p + i);
            }
            AT::deallocate(this->alloc, p, n);
        };

        return Array_type(AT::allocate(this->alloc, n), del);
    }
};
```

There's no need to construct the elements of the new array in `allocate`. After all, we might want to allocate more space than we currently need, if we expect a `vector` to grow later on. In this case, it would be a waste of time to construct a bunch of objects in the extra space, only to have to do so again when the `vector` actually grows to fill it.


## Member types ##

Up to now, I've been defining member types as they became needed. This makes it all too easy to forget one, in case it's never used (I'm not sure I ever got around to defining `array::difference_type`) and, to be honest, using `value_type` instead of `T`, everywhere, is really just verbosity for verbosity's sake. So, before I go, let's just go through and define all the member types.

In `vector.hpp`:

```cpp
template<typename T, typename Alloc = allocator<T>>
class vector
{
private:
    using AT = allocator_traits<Alloc>;     // Needs to go up top

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
