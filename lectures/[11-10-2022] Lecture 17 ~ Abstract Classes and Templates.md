# Lecture 17 - 10 November 2022


## Copy and Inheritance
```cpp
// Consider the following class hierarchy
class Book {...}
class Text: public Book {...} // Text(Books) are Books (is-a relationship)

Text t1{...}, t2{...};
Book *b {&t1}, *b2{&t2}; // b, b2 are book pointers

*b1 = *b2; // does partial assignment of two textbooks
           // Only copies the Book portion of both Text(books)
```

Mutation and reassignment pose problems in conjunction with inheritance.
* What if we assigned through base class pointers?
    * Partial assignment

* Virtual?
    * Mixed assignment
        * (`Text` from `Comic`)?

## Solutions?
* There's really no easy solution
* But we can write some code to prevent ourselves from writing unsafe code **(like the example above)**

### Questions
> Do we actually need/want copy and move assignment?
 * **If not** -- make `operator=` private 
 * Or `=operator() = delete` (delete the function) 
 * Both methods will tell the compiler to not generate a default version of the function

> If we need copy/move assignment
* Make `operator=` **`protected`** in base (`Book`) class.
    * So derived classes (`Text`) can still be reassigned.

Additionally, it probably makes sense to prevent **concrete** `Book` objects from being constructed.
* **Solution** Make `Book` abstract
* **NB**: Concrete classes are those that can be instantiated (i.e. `Book` objects can be created)

## Abstract Classes
* To make a class abstract, we need to pick a method and make it abstract
    * Abstract methods are those that are declared, but not defined

> What if I can't make any of my methods `virtual`?
* **Easy shortcut/hack** Just make the *destructor* `virtual`! (this usually works)
```cpp
class AbstractBook {
    public:
        virtual ~AbstractBook() = 0; // pure virtual destructor
                                     // There is no destructor body
};
```

Now, **pure virtual functions** can still have implementations!
* Here, `~AbstractBook()` *needs* an implementation.

#### Steps:
1. Declare `virtual ~AbstractBook() = 0` in the class
2. Define `AbstractBook::~AbstractBook() { /* destructor body */ }` outside the class
    i. The destructor body is defined outside the class
    ii. Destructor body can be empty if there is nothing to delete.
    iii. The destructor body **can** be non-trivial (i.e. not empty) *(actually deletes stuff if needed)*

***

## Templates

```cpp
// Linked-list of strings
class List {
    struct Node {
        string data; // data HAS to be type string
        Node *next;
    };
    // ...
};
```
* What if we need to store other types of data? (other than `string`)

What about a list of `int`
* **Crappy Solution**: Copy and paste it, and just change `string` to `int`

```cpp
// Linked-list of ints
class List {
    struct Node {
        int data; // data HAS to be type int
        Node *next;
    };
    // ...
};
```

*Copy and pasting is bad* -- Ed

```
                +--------+
                | object |
                +--------+
                     |
                     |
              -----------------
              *Every data type* 
```

```cpp
class List {
    struct Node {
        object *data;
        Node *next;
    };
};
```
* This gives us a list which can hold any `object` type, **but** we can't tell what we actually stored without runtime checks.
*Kind of like using `void *` in cpp* -Ed
*Java did this* -Ed

Also, not everything is an object
* C++ has primitive types that are not objects (unlike other OO languages)
    * Primitive types: `int`, `double`, `char`, `bool`, etc.

Ideally, we want a `List` class which accepts a **type** as a parameter
* We want `List` to be polymorphic over a **type** variable

This is why we have **Templates**

### Templates, actually now
```cpp
// T controls what type the List stores
template <typename T> class List {
        struct Node {
            T data;
            Node *next;
        };

    public:
        class Iterator {
                friend class List<T>; // specify which TYPE of List it is friends with
                Node *cur;
                explicit Iterator(Node *n): cur{cur} {}

            public:
                // Returns a reference to let you mutate data
                T& operator*() { return cur->data; }
                
                // Returns a copy of the data (read-only)
                // header can also be `const T& operator*() const`
                T operator*() const { return cur->data; }
                Iterator& operator++() { cur = cur->next; return *this;}
                // ...
        }

        // Some methods for the List class
        void addToFront (const T& n);
        T ith(int i);
};
```

*Templates are a compiler-provided way that copies and pastes code for you* - Ed

#### Usage
```cpp
List<int> l1; // list of ints
// Compiler looks for a template for List, then specializes the definition
// by replacing T with int and generates a copy of it for you.
```
The compiler needs to know (when `List` is instantiated here) the actual definitions of the methods of `List`
* Hence for templates, definitions are given in-line (class) declarations
    * `=>` you can't do seperate compilation for templates
* Therefore, you NEED TO define the methods in the header (.h) file
    * Can't do `List::addToFront(const T& n) { /* ... */ }` in the .cpp file
        * This is seperate compilation
    * instead **DO IT INSIDE OF THE `.h` FILE**
***
> **Side note, outside of course scope**

##### Template specialization
* Lets you specialize classes for a specific type
* Boolean vectors are *specialized* (`vec<bool>`) to be more efficient by being an array of `bits`. (ie `010100011`) (Google `bitset`)
***

### Using iterators with templates
```cpp
for (List<int>::Iterator i = l1.begin(); i != l1.end(); ++i) {
    cout << *i << endl;
}
// can shorten with auto

for (auto e: l1) {
    cout << e << endl;
}
```

### Standard Template Library
*C leaves you on a deserted island with some matches and firewood*
*C++ leaves you on a deserted island with some tools to help with your survival!* 
`-` Ed

* The Standard Template Library gives you **generic lists**, **sets**, **dictionaries**, and **dynamic arrays**.
* **NB**: The STL is a **library** of **templates**. It is not a language feature.

### Vectors - dynamic arrays (A4!)
```cpp
#include <vector>
using namespace std;

// Constructing vectors
vector<int> v{4, 5, 6} // constructs a vector {4, 5, 6}
// can be any data type, so long as each element has the same type
```
* Vectors implement dynamic arrays
* Gives you O(1) (amortized) time for babk insertion

```cpp
v.emplace_back(7); // adds 7 to the end of the vector
// use emplace if we can (emplace uses move)
// you can use push_back if you want to copy
// v = {4, 5, 6, 7}
```
* You can treat vectors as arrays

```cpp
for (int i = 0; i < v.size(); ++i) {
    cout << v[i] << endl;
} // works just like if v was an array
```

### Vector iterator
```cpp
// Can use auto if you want
for (vector<int>::iterator i = v.begin(); i != v.end(); ++i) {
    cout << *i << endl;
}

// Or just use a foreach loop (has iterator design pattern)
for (auto e: v) {
    cout << e << endl;
}
```

### Vector, but reverse
```cpp
for (vector<int>::reverse_iterator i = v.rbegin(); i != v.rend(); ++i) {
    cout << *i << endl;
}
```
* Iterates elements in reverse order

### Removing elements from vectors
```cpp
v.pop_back(); // removes last element
```

Vectors are **guaranteed** to be dynamically sized arrays, with the same performance characteristics.

Better to use vectors than writing yor own implementation.

### Erase
* `v.erase(v.begin())` - erases 1st element
* `v.erase(v.begin() + 2)` - erases 3rd element
* `v.erase(v.end() - 1)` - erases last element
    * By convention, end() iterator is 1 **after** last element
* Invalid positions result in undefined behaviour
* Will shift the array back to fill the gap

```cpp
v = {4, 5, 6, 7};

// Read the 1001th element
v[1000]; // undefined behaviour

v.at(1000); // like v[1000] except it checks if the index is valid
// throws an exception if the index is invalid
```

## Next class,
* Exceptions
