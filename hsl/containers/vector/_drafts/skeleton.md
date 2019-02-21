Now that our `array` implementation is complete, we can move on to another container, namely `vector`. In this post, we'll create the basic skeleton of our `vector` class, and then, in later posts, fill it in. That's almost the same as with `array`, except I crammed array initialization into the first post, because it was so simple. This time, we'll actually need to write constructors, ourselves, so I've put that off until after we get all the easy stuff done.

Now, as always, when starting a new class (or function or project or anything, really) it's a good idea to make clear exactly what we're trying to create. A `vector` is very similar to an `array`: it contains a sequence of elements, all of which have the same type, and which are contained in a contiguous chunk of memory (unlike a `list`) The difference is that its size isn't fixed. A `vector` is capable of growing and shrinking, though the latter is rare, and the standard states that the `shrink_to_fit` function doesn't even have to do anything.


A vector's size can't be stored as a template parameter

In `vector.hpp`:

```cpp
namespace hsl
{

template<typename T>
class vector
{
};

}
```

A vector uses dynamic allocation to store its elements. We can do this with a `unique_ptr`

In `vector.hpp`:

```cpp
template<typename T>
class vector
{
private:
    unique_ptr<T[]> arr;
};
```

The size isn't stored in the pointer, either, so we'll need to store it in a member. The # of elements in a vector doesn't have to be the # of elements it can hold, so we need to keep track of both. The former is called the size, the latter, teh capacity
In `vector.hpp`:

```cpp
template<typename T>
class vector
{
private:
    size_t arr_size;
    size_t arr_cap;
    unique_ptr<T[]> arr;
};
```

STL likes to use allocators for dynamic allocation, so we'll need to provide the allocator type as a template param

In `vector.hpp`:

```cpp
template<typename T, typename Alloc = allocator<T>>
class vector
{
private:
    Alloc alloc;
    size_t arr_size;
    size_t arr_cap;
    unique_ptr<T[], function<void(T*)>> arr;
};
```

We'll also do all the member types at once

In `vector.hpp`:

```cpp
template<typename T, typename Alloc = allocator<T>>
class vector
{
private:
    using AT = allocator_traits<Alloc>; // Saves some space 

public:
    using value_type = T;
    using allocator_type = Alloc;
    using size_type = size_t;
    using difference_type = ptrdiff_t;
    using reference = value_type&;
    using const_reference = const value_type&;
    using pointer = typename AT::pointer;   // Almost certainly the same as T*
    using const_pointer = typename AT::const_pointer;
    using iterator = pointer;   // Pointers still work fine as iterators
    using const_iterator = const_pointer;
    using reverse_iterator = hsl::reverse_iterator<iterator>;
    using const_reverse_iterator = hsl::reverse_iterator<const_iterator>

private:
    allocator_type alloc;
    size_type arr_size;
    size_type arr_cap;
    unique_ptr<value_type[], function<void(pointer)>> arr;
};
```
