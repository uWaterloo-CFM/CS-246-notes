# Lecture 24 - 06 December 2022

## COVERAGE - None of this stuff is going to be on the final!

### Problem
We want a function that returns the max of a, and b given some data type T.

#### In C
We have to write an implementation for each possible data type
```c
char max_char(char a, char b);
short max_short(short a, short b);
float max_float(float a, float b);
```
* This is boring, silly and stupid, we are unnecessarily duplicating code.

### Macros in C
```c
// As a macro
#define MAX(a, b)
    (((a) < (b)) ? (b) : (a))
```
* This sucks, because we need so many brackets ([Not again!](https://racket-lang.org/))
    * These brackets are needed because we need to substitute the macro anywhere
    `max(1, 2) * max (2, 1)` - Substituting into expressions for example
* Also things like `MAX(x++, y++)` are not allowed, because the macro is expanded before the    expression is evaluated (postfix notation things)

##$$ In C++
```cpp
template <typename T> classList {
    struct Node {
        T elem;
        ...
    };
}

template <typename T>
    T max(T a, T b) {
        if (a < b) return b;
        else return a;
    }

int main() {
    int a = 3;
    int b = 4;
    int c = max(a, b); // Works!

    float x = 2.0;
    float y = 3.5;
    int z = max(x, y); // 3.5
}
```
* **Notice** We didn't have to write `max<float>(x, y)` or anything like that
    * This is because the compiler can infer the type from the arguments (Introduced in C++17)

#### Explicitly, we can also write
```cpp
max<int>(x, y);
```
* `max()` can be defined on ANY type (including structs, classes) that have the `<` operator overloaded (defined)

***
## Abstract Base Class for Iterator

```cpp
// What we had before
class AbstractIterator {
    // ... take implementation from the previous notes
};

void for_each(AbstractIterator& start, AbstractIterator& end, void (*f)(int)) {
    for (; start != end; ++start) {
        f(*start);
    }
}
```
**Flaws**
1. Must pass by reference as Abstract Iterator is an abstract class
2. Pure virtual functions for `operator++` and `operator*` `operator!=` are not defined
    ```cpp
    virtual bool operator!=(const AbstractIterator& other) const = 0;
    ```
        * Defining/Implementing comparison across two arbitrary iterators is painful

Say we want to implement a `TierIterator` that iterates over a `TierList`
```cpp
class TierIterator: public AbstractIterator {
    bool operator!=(const AbstractIterator& other) const override {
        if (TierIterator* t = dynamic_cast<TierIterator*>(&other)) {
            return ... something here
        } else {
            return false;
        }
    }
};
```
* Implementing the iterators this way are not good

### Solution
**With Templates**
```cpp
// This is a GENERIC ITERATOR CLASS, not only for TierList
template <typename InputIter, typename Func>
    // HAS TO BE BY COPY, not by reference, because we need to increment it
    void for_each(InputIter start, InputIter end, Func f) {
        for (; start != end; ++start) {
            f(*start);
        }
    }

int main() {
    int a[5] = {1, 2, 3, 4, 5};
    for_each(a, a + 5, print);
    void print(int n) {
        std::cout << n << std::endl;
    }
}
```

### `#include <algorithm>`
Provides a collection of generic iterator algorithms

**Examples**
```cpp
// std::find - returns an iterator to the first element in the range [start, end) that is equal to val
//                  returns end if no such element is found
template<typename InputIter, typename T> InputIter find(InputIter start, InputIter end, const T& val) {
    while (start != end && *start != val) ++start;
    return start;
}

// std::count - returns the number of elements in the range [start, end) that are equal to val

// std::copy - copies the elements in the range [start, end) to another range beginning at start
//                  returns an iterator to the end of the destination range
template<typename InputIter, typename OutputIter> OutputIter copy(InputIter start, InputIter end, OutputIter dest) {
    while (start != end) {
        *dest = *start;
        ++start;
        ++dest;
    }
    return dest;
}

// std::transform - applies a function to each element in the range [start, end) and stores the result in another range
template<typename InputIter, typename OutputIter, typename Func> OutputIter transform(InputIter start, InputIter end, OutputIter dest, Func f) {
    while (start != end) {
        *dest = f(*start);
        ++start;
        ++dest;
    }
    return dest;
}

// usage of transform
int a[5] = {1, 2, 3, 4, 5};
int b[5];
int square(int n) { return n * n; }
transform(a, a + 5, b, square);
// In fact, anything that support the function call operator can be used as a function

class Plus1 {
    int operator()(int n) { return n + 1; }
};

Plus1 p;
transform(a, a + 5, b, Plus1());

// Why is this useful?
// We can use this to to implement generic functions
// Instead of ONLY adding 1 to each element, we can add any number
class Adder {
    int m;
    public:
        Adder(int n): m(n) {}
        int operator()(int n) { return n + m; }
};

transform(a, a + 5, b, Adder(3)); // adds 3 to each element

// We can do really cool things
// ie: Incremented added, (add m to n_0, then add m+1 to n_1, then add m+2 to n_2, etc)
class BadAdder {
    int m;
    public:
        BadAdder(int n): m(n) {}
        int operator()(int n) {
            return (n + m++);
        }
};
```

***
## Lambda Expressions
```cpp
// anonymous function that can be passed as an argument to another function
transform(a, a + 5, b, [](int n) { return n + 1; });
```
* Lambdas are anonymous local functions-like objects the can capture variables by reference in the []
    * Stuff in square brackets are the captures
    * The captures are optional

### Other Iterator
```cpp
ostream_iterator<T>(ostream &o);

ostream_iterator<int> w(cout);

*w++ = 5;
back_insert_iterator<T>(container &c);
// uses push_back to write elements to ctr
```

## Functional vs Object-oriented Languages
* TL;DR
    * If you have a fixed data type definition, use a functional language so you can always add more functions
    * If you have fixed functions, use an object-oriented language so you can always add more data types
