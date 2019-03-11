## Element access ##

In this post we'll implement all of the element access member functions. All are pretty similar to `array`'s, so this will be a short post. In fact, they're so similar that explaining how they work would be redundant. I'll just present the code, and let it speak for itself (I think I'm getting lazy).

```cpp
// Under get_allocator
reference at(size_type pos)
{
    if (pos >= size()) throw out_of_range("hsl::vector::at");
    return arr[pos];
}

const_reference at(size_type pos) const
{
    if (pos >= size()) throw out_of_range("hsl::vector::at");
    return arr[pos];
}

reference operator[](size_type pos) { return arr[pos]; }
const_reference operator[](size_type pos) const { return arr[pos]; }

reference front() { return arr[0]; }
const_reference front() const { return arr[0]; }

reference back() { return arr[size() - 1]; }
const_reference back() const { return arr[size() - 1]; }

pointer data() noexcept { return arr.get(); }
const_pointer data() const noexcept { return arr.get(); }
```

`data` is the odd-man-out, here. A `unique_ptr`'s get member function just returns the underlying pointer.

## Iterators ##

Since things are going so quickly, I'll also squeeze `vector`'s iterator functions into this post, too. Like the element access member functions, these are almost identical to there `array` counterparts.

In `vector.hpp`:

```cpp
iterator begin() noexcept { return data(); }
const_iterator begin() const noexcept { return data(); }

iterator end() noexcept { return data() + size(); }
const_iterator end() const noexcept { return data() + size(); }

const_iterator cbegin() const noexcept { return begin(); }
const_iterator cend() const noexcept { return end(); }

reverse_iterator rbegin() noexcept { return reverse_iterator(end()); }
const_reverse_iterator rbegin() const noexcept
{ return const_reverse_iterator(end()); }

reverse_iterator rend() noexcept { return reverse_iterator(begin()); }
const_reverse_iterator rend() const noexcept
{ return const_reverse_iterator(begin()); }

const_reverse_iterator crbegin() const noexcept { return rbegin(); }
const_reverse_iterator crend() const noexcept { return rend(); }
```

## Swap ##

```cpp
// Under capacity functions
void swap(vector& other)
{
    hsl::swap(arr_size, other.arr_size);
    hsl::swap(arr_cap, other.arr_cap);
    hsl::swap(arr, other.arr);
}
```

```cpp
template<typename T>
void swap(vector<T>& lhs, vector<T>& rhs)
    noexcept(noexcept(lhs.swap(rhs)))
{ lhs.swap(rhs); }
```

## Relational operators ##

```cpp
// Under class definition
template<typename T>
bool operator==(const vector<T>& lhs, const vector<T>& rhs)
{
    return lhs.size() == rhs.size()
        && equal(lhs.begin(), lhs.end(), rhs.begin());
}

template<typename T>
bool operator!=(const vector<T>& lhs, const vector<T>& rhs)
{ return !(lhs == rhs); }

template<typename T>
bool operator<(const vector<T>& lhs, const vector<T>& rhs)
{
    return lexicographical_compare(
            lhs.begin(), lhs.end(),
            rhs.begin(), rhs.end());
}

template<typename T>
bool operator>(const vector<T>& lhs, const vector<T>& rhs)
{ return rhs < lhs; }

template<typename T>
bool operator<=(const vector<T>& lhs, const vector<T>& rhs)
{ return !(lhs > rhs); }

template<typename T>
bool operator>=(const vector<T>& lhs, const vector<T>& rhs)
{ return !(lhs < rhs); }
```
