---
layout: post
title: Homebrew Standard Library – Intro
---

The idea for this series started out as a C++ tutorial. It would've started from scratch, with only plain C++ and the system calls in `unistd.h`, and built up good portions of the C and C++ standard libraries from there. That's probably a terrible way to learn C++, but it is an interesting one. I may come back to it, at some point, but it's morphed, in my mind, to something different.

In this series, I will create an implementation of the C++ Standard Library, but the approach won't be purely bottom-up. Rather, I'll replace pieces of it, bit-by-bit, until nothing of the original is left, like the [Ship of Theseus][ShipOfTheseus]. This series won't work as a C++ tutorial either, though it might be useful for those who know the language, but not the finer details of the Standard Library. The real purpose of this series is my own understanding. I'm not terribly familiar with any of the new features of C++14 or 17, I don't have more than cursory knowledge of template metaprogramming, and I'm sure I have a few other weak points I'm less aware of.

The standard I'll be following is C++17, which is the latest version, as I'm writing this. A copy of the standard costs [$116 for the PDF][ANSIWebStore], so I won't be buying it. Instead, I'll use [cppreference.com][CPPReference] and [cplusplus.com][CPlusPlus] as references (the former is more accurate, and the latter is easier to read, in my experience). I'm aiming for something like 95% compliance -- I'll make a few, purposeful, changes, in order to make things easier on me. To wit, my header files will end in `.hpp`. This is mostly just so that my text editor will do syntax highlighting automatically. Lazy, yes, but convenient. I'll also rename the `std` namespace to `hsl` (Homebrew Standard Library). This is because, until everything is completed, I'll need to include standard library headers, and something like this won't compile:

```cpp
#include <string>

namespace std
{

template<typename Char, typename Traits, typename Alloc>
class basic_string
{
public:
    // ...
};

}
```

Of course, this creates a different problem, namely that I'll wind up with a bunch of references to the `std` namespace in my own files, which I know I'll need to do a find-and-replace on, at some point. My solution is just to create aliases for any types or functions I use, in the `hsl` namespace.

You can access the source code for everything, and see my progress on [GitHub][GitHubRepo].

### Contents

* array


[ShipOfTheseus]: https://en.wikipedia.org/wiki/Ship_of_Theseus
[ANSIWebStore]: https://webstore.ansi.org/Standards/INCITS/INCITSISOIEC1488220172018
[CPPReference]: https://en.cppreference.com
[CPlusPlus]: http://www.cplusplus.com/
[GitHubRepo]: https://github.com/bobcarterirl/homebrew-stl
