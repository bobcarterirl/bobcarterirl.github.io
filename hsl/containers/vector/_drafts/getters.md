Today, we're continuing the theme of doing the easy parts first, and implementing `vector`'s 'getters'. In practice, what I mean my 'getters' are all of the capacity functions that are specified as `const`, plus `get_allocator`, which really doesn't fit anywhere else.

## max_size ##

We'll begin with `max_size`, which is the easiest since it only returns a constant, specifically the maximum, theoretical size of a `vector`. This comes down to the greatest possible value of `difference_type`. We can access this value using `numeric_limits`:

In `vector.hpp`:

```cpp
#include "limits.hpp"   // Create file and add "using std::numeric_limits"
// ...
template<typename T, typename Alloc = allocator<T>>
class vector
{
public:
    // ...
    size_type max_size() const noexcept
    {
        return numeric_limits::<difference_type>::max();
    }
};
```

## size ##

In the last post, we noted that the size of a `vector` can't be stored as a template argument. It isn't accessible through the `unique_ptr` to the underlying array or `allocator` either, so we'll have to make it a member, and we'll return that member with the `size` function:

In `vector.hpp`:

```cpp
template<typename T, typename Alloc = allocator<T>>
class vector
{
public:
    // ...
    size_type size() const noexcept
    {
        return arr_size;
    }

private:
    size_type arr_size;
};
```

## capacity ##

Changing the size of a `vector` requires dynamic allocation, which is expensive. It would be handy if our `vector` implementation would allocate more space than it currently needs, in situations where it's likely to need more later. But, if we were to enable this (and we will, since the standard requires it) the size of the `vector` would often be smaller than the size of the underlying array. Let's introduce some terminology, here: the number of elements in the `vector` is, naturally, called its 'size'; the number of elements in the underlying array -- *i.e.* the number of elements the vector *can* hold before it needs to allocate more space -- is called its 'capacity'.

We'll need to store the capacity in a separate member, and have a separate member function to access it:

In `vector.hpp`:

```cpp
template<typename T, typename Alloc = allocator<T>>
class vector
{
public:
    // ...
    size_type capacity() const noexcept
    {
        return arr_cap;
    }

private:
    size_type arr_cap;
};
```

## empty ##

`empty` is very simple. It returns `true` if the `vector`'s size is `0`, and false otherwise:

In 'vector.hpp`:

```cpp
template<typename T, typename Alloc = allocator<T>>
class vector
{
public:
    bool empty() const noexcept
    {
        return arr_size == 0;
    }
};
```

## get_allocator ##

So is `get_allocator`. It returns a copy of the `vector`'s allocator:

In 'vector.hpp`:

```cpp
template<typename T, typename Alloc = allocator<T>>
class vector
{
public:
    allocator_type get_allocator() const noexcept
    {
        return alloc;
    }
};
```

I edited the last post to include the definition of `alloc`, so check that, if you missed it.

That's all for the getters. Next time we'll handle the element accessors.
