# Lecture 10 - October 18, 2022

# Assignment Operator(s), Destructors

### Linked List example
```cpp
Node& operator = (const Node& other)
{
    if (this == &other) return *this;
    Node *tmp = next;
    value = other.value
    next = new Node (*other.next);

    delete tmp;
    return this;
}
```

```
f = [2 -]->[4 -]->[6 /]
g = [3 -]->[5 -]->[7 /]
```

> What happens when we do `*f = *g`?

```c
g = [3 -]->[5 -]->[7 /]

f = [2 -]------{>[4 -]->[6 /] } {NEW}

// BUT

[4 ] // deleted - `delete tmp`
[6 /] // Memory Leak!
```
* In CS 136, we would have written a helper function to help us recursively/iteratively destroy the list until it reached the end.

***
* Wouldn't it be good, though to arrange for object destruction when:
    * An object on the heap is deleted?
    * An object goes out of scope?

## Destructors!
* When an object goes out of scope or is `deleted`:
    * Its **destructor** runs
    * Then field destructors
    * Then the memory for that object is freed

### Defining Destructors   
* Destructors are defined with the `~` symbol

```cpp
struct Node {
    // ...

    ~Node() {
        delete next;
    } // resursively deletes next until it gets to nullptr!
}
```
* Remember, destructors run when the object is `deleted`, so when you `delete next`, it recurses and runs the destructor on the next node, and so on.
* This is also why we don't need to write a helper function to delete the list.

***
## Move Constructors

```cpp
Node plus1(Node other) {
    for (Node *p = &other; p != nullptr; p = p->next) {
        p->value++;
    }
    return other;
}

Node h = plus1(*f);
```
### Sequence of Operations
1. We copy `*f` to pass into `plus1`
2. Copy other to some temporary variable outside of the scope of `plus1`
3. Copy that temporary variable into `h`
4. Delete the temporary variable after `plus1` returns

### This is a bit redundant. Can we do better?
* Can we just steal the object's value?

### Yes - R-Value References
* R-Value References are references to temporary objects
* They are used to steal the value of a temporary object
* They are declared with `&&` ie: `Node &&other`

We often copy from temporary objects, and this copying is inefficient so we instead can just **move** these fields to the variable that is assigned to the return value of the function as the temporary is going away once the function returns.

```cpp
struct Node {
    // ...

    Node(Node &&other)
        : value(other.value) next(other.next)
        {
            other.next = nullptr;
        }
}
```

### Move Assignment Operator
```cpp
Node& operator = (Node &&other) {
    if (this == &other) return *this;

    delete next;
    next = other.next;
    value = other.value;
    other.next = nullptr;
    return *this;
}
```

***
### Most common way of implementing the Move Operator - `swap`
* Now, `other` is going away
* We need to also send off our tail
* So we will swap the fields of `other` and `this`
    * use `std::swap()` to swap the fields of `other` and `this`
    * `std::swap()` is a function that swaps the values of two variables
* `NB:` if you `using namespace std` then you don't need `std::`

```cpp
Node &operator = (Node &&other) {
    if (this == &other) return *this;

    std::swap(value, other.value);
    std::swap(next, other.next);
    return *this;
}
```

***
### alternate method: Use a Copy and Move Constructor
* With a copy and move constructor defined, we could also instead have a single unified assignment.

```cpp
Node operator = (Node other) {
    std::swap(value, other.value);
    std::swap(next, other.next);
    return *this;
}
```

* Operator
    * Will copy values that need to be copied anyways
* With a move constructor, you can efficiently move rvalues passed by value
* This might not be the most efficient
* Sometimes you'll want a seperatore move and copy assignment operator.

***

## In Summary
* Objects have:
    0. Constructors
    1. Copy Constructors
    2. Move Constructors
    3. Copy Assignment Operators (=) **|**
    4. Move Assignment Operators (=) **|** => (3, 4) These can be unified 
    5. Destructors

* For **1-4**, implementing one probably means implementing the other
* You should probably know how to implement **1-5**.
    * **0** is not any single thing, but a collection of things
* This is known as the **rule of 5**

***
* You can never rely on C++ that a certain constructor will always be called.

```cpp
// Our Constructor could be called in any of 3 places
Vec makeVec() {
    return Vec{0, 0} // 1. Can be called when Vec is initialized
    // 2. Can be called when Vec is being returned - Copy/Move
}

Vec v = makeVec(); // Copy or move from temporary into v
```
* There are 3 possible constructor invocations
* However, only 1 constructor:
    * The first call may be made 

The compiler is allowed to optimize away these copies by setting up the stack frame appropriately. 
    * This is known as **copy elision**.

```cpp
Vec doSomething(Vec v) {...}

doSomething(makeVec())
```
* The compiler can skip the `make Vec()` call and just pass the `Vec` directly into `doSomething` without copying it.
* This optimization can happen if it would **change the behaviour of your program**


**TL;DR** Don't rely on the compiler to call the constructor you want.

***

### Constructors being called unintentionally

Some constructors can be **unintentionally** called.

```cpp

struct Node {
    // ...
    Node(int v): value(v), next(nullptr) {}
};

Node n = 5;
```
* This code actually calls the constructor `Node(5)`

```cpp
Node n = 5;
Node g(6, nullptr);
g = 7; // Also implicitly calles Node(7)
```
* Single argument constructors are used by the compiler to perform implicit conversions.
* If you **don't** want this behaviour, you HAVE to mark your constructor as `explicit`

```cpp
struct Node {
    // ...
    explicit Node(int v): value(v), next(nullptr) {}
};
```
* Now, the compiler will not call the constructor for you.

***
## Overloading: Assignment Operators vs. Operators
* Assignment operators are special

```cpp
Vec operator + (vec 1, vec r);
    // defined outside of Vec class

Node& operator = (Node &&other);
    // defined inside of Node class
```
* We can also define other operators as members of the class
* Unary operators do not take in any arguments (barring special cases)

* **When** an operator is defined as a member, this takes the place of the first (often left) parameter

EX
```cpp
struct Vec {...}
    // The left/first argument is `this`