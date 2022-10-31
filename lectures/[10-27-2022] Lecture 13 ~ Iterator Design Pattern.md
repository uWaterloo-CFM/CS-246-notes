# Lecture 13 - 27 October 2022

## Iterators
All Iterators have these basic elements
* `operator*()` - dereferencing/accessing values
* `operator++()` - accessing next element
* `operator!=()` and `operator==()` - for comparison

### What our class looks like:

```cpp
class List {
    public:
        class Iterator {
            // Note that cur is private (implicit by Iterator class)
            //  Protects List's invarients 
            //  Protects Node's pointers
            Node* cur; // the thing we are iterating over
                       // Think of this as arr[i]
            public:
            // We do have a constructor for Iterator
            explicit Iterator(Node* p) {...}
            
            int operator*() {
                return cur->value;
            }
};
```

**How do we construct iterators and how do we give users access to these?**

```cpp
// This is how we iterate over a normal array using
//  pointer arithmetic
int arr[10] = {1, 2, 3, ..., 10};
for (int *p=arr; p < arr + 10; p++) {
    ...
    ...
}
```
**In general, we have the notion of:**
* where to start (`arr`)
* where to stop (`arr + 10`)

How do we do this with our `List` class?
* Provide **methods** in our `List` class that gives us access to these things
    * `List` needs to be able to produce these 2 **public** methods:
        * `start` iterator
        * `end` iterator (Also known as termination iterator)

```cpp
List {
    // ... Implementations from above
    public:
        Iterator begin() {return Iterator{head};} // uses Iterator's ctr
        Iterator end() {return Iterator{nullptr};} // users Iterator's ctr
}

int main() {
    List l;
    l.push_front(2);
    l.push_front(3);
    l.push_front(4);
    for (List::Iterator p = l.begin(); p!= l.end(); ++p) {
        cout << *p << endl; // Calls operator* and << operator implictly
    }
}
```

**Improvement**
* We can make `*p` return a reference to allow mutation
* We can change the `ith` element of the linked list
* We just need to return a **reference** to the int 
```cpp
int &operator() {return cur->value;}
```

**What if we want both?**
* We can overload the function and have both
```cpp
int operator*() const {
    return cur->value
}

int &operator*() {
    return cur->value;
}
```

**Improvement 2**
* What if we don't want to call `List::Iterator` each time
* Use **Type Inferences**

### Type Inferences for Iterator Variables
* Use the `auto` keyword to let the interpreter to **inference** the type 
```cpp
for (auto p = l.begin(); p != l.end(); ++p) {
    // Auto infers the type of p based on its initializer
    // ...
}
```
* Makes our life a little easier

**Important 3**
* Notice that iterators all kind of look the same (same syntax, operations)
* They have the same methods
* **Therefore, the the compiler has a shorthand for these**
    * Our operator can abstract over these complications to give a general notion for containers containing iterators with a `start` and `end`.
    * Iterator Range-Based Syntax

```cpp
int main() {
    List l;

    // shorthand
    for (int x:l) {
        cout << x << endl;
    }

    // equivalent to
    for (auto i = l.begin; i != l.end(); ++i) {
        // this step gets you the value of x
        int x = *l;
        cout << x << endl;
    }
}
```
* You can change the type (`int` in this example) to any of the type you want

**NB** To use this shorthand you NEED
* `operator*()` - dereference (returns the type of the value)
* `operator+()`
* `operator!=()`
* `operator==()`

**Improvement 4**
```cpp
int main() {
    List l;

    // shorthand
    for (auto x:l) {
        cout << x << endl;
    }
}
```
* We can use auto even! Everything will just WORK!
* **Downsides**
    * Can't let you start and end at different points


***

## Friendship! (`friend`)
When we built our constructor for our Iterator, we made it `public`

```cpp
class Iterator {
    public: 
        Iterator(Node *p); // public
}
```
* This exposes `Iterator`'s constructor, which is not what we want.
* This lets the user construct an arbitrary `Iterator` over an arbitrary `Node` pointer.
* We should only be able to get iterators from:
    * `List::begin()`
    * `List::end()`
    * copies of existing iterators
    * `iterator++`

* Why don't we just make the constructor private?
    * This doesnt work as `List` can't make iterators.
    * NB: `private` `public` was always for encapsulation and to make sure invarients were'nt violated.
* **imporant**: but `List` and `Iterator` are **coupled** naturally
* We don't really need to enforce any encapsulation boundaries between a `List` and its `Iterator`.
    * While still protecting `List` and `Iterators` from **external** users outside of `List`

Basically, we want our `Iterator` constructor to be protected (`private`) but still callable by `List`

* Now, to allow access to Iterator's private fields and methods from `List` we declare `List` to be a **`friend`** of `Iterator`.

**Example**
```cpp
class Iterator {
    private:
        Iterator(Node *p);
        // ...
    friend class List;
    // List is a friend of Iterator
    // List gains access to the private fields of Iterator
}
```
* If a class **A** is a friend of class **B**, **A** gains access to the fields of **B**.
```cpp
class B {
    // ...
    friend class A; // A is a friend of B
}
// We can access B's private fields in A
```
* `friendship` is NOT symmetric/transitive
    * `Iterator` is not a friend of `List`

Example:
```
A is a friend of B
B is a friend of C

A is not a friend of C
(unless otherwise explicitly stated)
```

**New friendship**
* Friendship weakens encapsulization as anyone who is a friend can use your private fields and methods
* But Friends are hugely useful

This is useful for iterators because you want to be able to construct them.

#### Other good uses of Friends
```cpp
class Vec {
    int x, y; /// Private to Vec by default
}

// Now some operators need to be external to Vec:
Vec operator*(int, const Vec& v);
ostream& operator <<(ostream&s, const Vec& v);
// These two operators^^ probably want to access Vec's private fields
```

**Solution**
```cpp
// Just declare them as fields
struct Vec {
    // ...
    friend Vec operator*(int k, const Vec&);
    friend ostream &operator<<(ostream &s, const Vec &v);
}
```

***
