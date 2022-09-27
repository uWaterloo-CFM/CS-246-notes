# Lecture 6 - 27 September 2022

## Overview from Lecture 5

### C++ QoL
* Streams
    * File
    * String
    * Manipulators (Type Manipulation)

### Overloading
> The following definitions can all be made at the same time:
* `void twice(string s)`
* `void twice(int n)`
* `void twice(int a, int b)`

* I can't define a function with the following parameter list, though:
* `void twice(string z)`
    * Can't overload twice for a function that has the same parameter list


## Structures
```cpp
struct Node {
    int k;
    Node* next;
}
// in C: struct Node* next;
// ^^^^ This is not in C+++
```

## Constants

```cpp
const int maxGrade = 100;
// All const variables must be initialized (to some value)
// (otherwise, what's the point?)
// You can't mutate constants
```

## Functions that mutate variables
```cpp
// Pass the pointer into the function
void inc(int *n) {
    ++(*n);
}

int x = 5;
inc(&x);
cout << x
```

## References
**eg: Taking in Input**
```cpp
int x;

cin >> x;
// Why don't we need to use the pointer to x?
// Because C++ uses x as a reference to x, which 
//  automatically dereferences as necessary
```

```cpp
int y = 10;
int &z = y;
// z is a reference to y
// you cannot change what z points to

z = 5;
// changes y to hold 5
cout << y;

y = 6;
// z also produces 6

cout << z;
// &z is the same as &y
```

### Limitations

* References **must** be initialized
* References **must** refer to a **l-value**
    * **l-value**: Something with a memory address
* You can always set a reference to anything that you can grab the address of

---
* ie: You can't do this:
```cpp
int &x = z + a;
```

---

**NB:**
* You can't have a pointer to a reference.
* You can create references to pointers.

```cpp
int &x = y;
int &x = *z; // References to the value z points to
```

### Increment, with References
```cpp
int x = 10;

void inc(int &n) {
    ++n;
}
.
.
.
inc(x);
// We don't have to pass in a pointer, C++ will automatically
//  dereference if necessary
```

### `cin`, revisited
```cpp
int x;
cin >> x;
```

* `x` is passed by reference
* Strings passed into streams are also passed by reference

### Large Structs
> Let's say you wanted to print out a gigantic struct
```cpp
struct ReallyBig {
    char big[4000]
}

// Pass by value
void print(ReallyBig x) {
    ...
}

// Pass by reference
void print(ReallyBig &x) {
    ...
}
```

* It is MUCH faster to pass by reference
    * If passing by value, the program needs to allocate memory, and make a copy
    * If passing by reference, no need

### Referencing constants
```cpp
int x = 10;
void f(int &n);

f(x); // works
f(10+5) // doesn't work, cant reference an expression
```

* However, you can get around this
```cpp
void g(const int &n);
g(10 + 5); // ok now
// g expects a reference to a constant
```

### Why support passing references to constants?
* Whenever we add constants to parameters to a function:
    * The program allocates memory and makes a copy of each param to the stack
    * This could be bad
* If passing in references, this is not done.

**TL;DR**: Pass values by reference, when you can
* If you need to mutate the parameter, then pass by value
* Passing by reference helps speed up

## Memory
```cpp
struct Node {
    Node *next;
    int k;
}

// How do we allocate memory onto the heap to store a Node?
struct Node *p = malloc(sizeof(struct Node));

free(p); // Don't forget this!
```

**Malloc is unsafe**
* You need to remember to call `free`
    * For recursive structures, you need to recursively `free` Nodes
* You need to make sure you are allocating enough memory for the structure

### Memory in C++
* Memory management in C++ will automatically do all of these things for you!

```cpp
Node *p = new Node; // allocates right size.
                    // initializes  memory

delete p; // delete after
```

#### Allocating memory for an array of nodes
```cpp
Node *arr = new Node[10];

delete[] arr; // Don't interchange []! Only use delete[] for arrays!
              // And only use delete for single variables
```

**NB:** No `realloc()` in C++

```cpp
// Return node by value
Node getMeANode() {
    Node n;
    .... // set up n
    return n;
}

// Why Node or Node & in the position

// Return pointer to Node
Node *getMeANodeptr() {
    Node *n;
    ... //set up n
    return n;
}
```
* **First Example**: Return stack-allocated variables by value
* **2nd Example** Referencing and pointers that are returned need to point to heap or global (static) memory.

## Vectors + Operator Overloading
```cpp
struct Vec {
    int x;
    int y;
}

// Can I add vectors? 
// Can I multiply them by scalars?

// Adding vectors in C
Vec v1 = {0,0}
Vec v2 = {1,5}
vec_add{v1, v2}

// What about in C++?
Vec operator+ (const Vec &l, const Vec &r) {
    Vec res = {l.x + r.x, l.y + r.y};
    return res;
}

Vec operator* (int l, const vec &r) {
    Vec r = {l * r.x, l * r.y};
    return r;
}

// Can't do this:
v1 * 5 // our * operator only does scalar * vector, not other way around
```

## More special overloads
* Overloading `<<` and `>>` for I/O streams.

* For Output
```cpp
ostream &operator<< (ostream &out, const Vec &v) {
    out << "(" << v.x << ", " << v.y << ")" << endl;
    return out;
}
```

* For Input
```cpp
istream &operator>> (istream &in, Vec &v) {
    in >> v.x;
    in >> v.y;
    return in;
}
```