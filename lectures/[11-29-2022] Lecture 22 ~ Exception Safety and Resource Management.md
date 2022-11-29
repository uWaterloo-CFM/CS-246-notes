# Lecture 22 ~ 29 November 2022

## Exception Safety and Resource Management
```cpp
void f() {
    A* a = new A;
    g();  // if g() throws, then we leak a
    delete a;
} 
```

This is why we want to use smart pointers

### Smart Pointers

#### `unique_ptr`

```cpp
void f() {
    unique_ptr<A> a = make_unique<A>();
    // unique_ptr<A> a {new A}; also works

    g(); // even if g() throws, a will be deleted

    // we don't even need to delete a here
}
```
We can dereference `unique_ptr`, we can select fields of the object it points to, we can call methods on the object it points to.

**The one drawback** here is that it can be difficult to copy a `unique_ptr` (we can't copy it, we can only move it).

### Copying `unique_ptr`
* There is the issue of where we want to `delete` the memory allocated to the object. If we copy a `unique_ptr`, then we have two pointers to the same object, and we don't know which one should be responsible for deleting the object.

* Ownership can't be copied, but it can be transferred (`move`d).

**Moving unique pointer from one variable to another**
```cpp
using std::move;
void f() {
    unique_ptr<A> a = make_unique<A>();
    unique_ptr<A> b = move(a);
    // a is now null
    // b points to the same object as a did
}
```

### How `unique_ptr` works under the hood
```cpp
template <typename T> unique_ptr {
        T* ptr;
    public:
        unique_ptr(T* p) : ptr{p} {}
        ~unique_ptr() { delete ptr; }

        // move constructor
        unique_ptr(unique_ptr&& other) : ptr{other.ptr} {
            other.ptr = nullptr;
        }

        // move assignment
        unique_ptr& operator=(unique_ptr<T>&& other) {
            using std::swap;
            swap(ptr, other.ptr);
            return *this;
        }

        // no copy constructor or assignment
        unique_ptr(const unique_ptr<T>&) = delete;
        unique_ptr& operator=(const unique_ptr<T>&) = delete;

        T& operator*() { return *ptr; }
        T* operator->() { return ptr; }
}
```

## What if we still WANT to copy a `unique_ptr`?
* When we make a copy of a `unique_ptr` we do it because:
    * We want to pass it to a function that takes a `unique_ptr` by value
        * **Here** we can just pass it **by reference**
        * This passes a reference to the `unique_ptr`, not the `unique_ptr` itself, so ownership is not changed.
        * 1 `unique_ptr` for ownership, raw pointers for references.
    
### Shared ownership
* Here, we want to use `shared_ptr<T>`
* `shared_ptr<T>` is another smart pointer type which models shared ownership.
    * The memory is freed when the **last** `shared_ptr` pointing to it is destroyed.

```cpp
shared_ptr<A> a {new A};
shared_ptr<A> b = a;
// a and b both point to the same object
// a can be destroyed, but b will still be valid
// when a and b are BOTH destroyed, the object is deleted
```
* These are implemented by keeping track of the **reference count** of the object.
* Only deletes the object when the reference count is 0.
* **still no need to call `delete`** on a `shared_ptr` - deletes when popped out of scope.

### Dangers of `shared_ptr`
* `shared_ptr` is not guaranteed to be leak-free.
* If we have a `shared_ptr` that is never destroyed, then the object will never be deleted.

**Example, Linked List**
```cpp
struct Node {
    int value;
    shared_ptr<Node> next;
};

// if we make:
// [1 |-]-> [2 |-]-> [3 | ]-> nullptr
// this will delete properly

// However, this will fail:
// Reference count will never reach 0 if you have cyclic data sructures

// 1 points to 2, 2 points to 3, 3 points to 1
// [1 |-]-> [2 |-]-> [3 |-]-+
//  ^                       |
//  +-----------------------|
```
* `unique_ptr` is leak-free, but `shared_ptr` is not.

**Don't use `shared_ptr` for doubly-linked lists**

### Takeaway
* Smart pointers are a good way to manage resources.
* Makes it so that you don't have to remember when to delete/free things
    * Automatic cleanup
* `unique_ptr` is leak-free so use that as much as you can
    * You usually only have one owner of a resource
* `shared_ptr` is not leak-free, so use it with caution, and ONLY when you have shared ownership

## Other Resources
* Files are an example of an external resource

```cpp
void f() {
    int handle = open("Hello.txt", ...); // handle is an integer ID for the file used by the OS
    // do stuff with handle
    g();

    // close the handle
    close(handle);
}
```

```cpp
void f() {
    ifstream f {"Hello.txt"}; // handle is opened here
    // do stuff with f
    g();

    // no need to close f, ifstream will close it automatically
}
```

* The same problem of managing memory exists for other resources, like files, network connections, etc.
    * Alot of these rely on a stack-allocated wrapper object to manage the resource (automatic cleanup/destruction)
        * `ifstream` is an example a stack-allocated wrapper object to manage the file

## RAII (Resource Acquisition Is Initialization)
* RAII is a technique for managing resources
    * It is a way to make sure that resources are always cleaned up

## Back to Exception Safety
* A function is basic exception safe:
    * If an exception is thrown, the program ends up in some valid state
        * In particular, you don't violate your invarients and you don't have any memory leaks
        * Invarients preserved, no resources leaked

* A function `f` has the **strong exception safety guaruntee** if and only if
    * Whenever `f` throws or propagates an exception, it reverts to a state as if `f` was **never called**
        * You can roll back the changes that `f` did.

* A function `f` has the **no-throw guarantee** if and only if it never throws an exception, and always accomplishes what it is supposed to do.

```cpp
// even if g and h have strong exception safety:

void f() {
    g(); // finished
    h(); // throws exception
}

// f may not also have strong exception safety
// may not be in order to recover from an exception in h() since we need to roll back g()
// but we don't know how to roll back g.
```

**Also if `g()` has strong exception safety**
```cpp
void f() {
    ctr++;
    g();
} // f is not string exception safe, so ctr changes

void f() {
    g();
    ctr++;
} // f is string exception safe, so ctr is not changed
```

* It may not always be possible to have strong exception safety
* But you WANT strong exception safety in **containers**.
    * ie: `vector`, `lists`, etc
    * STL (standard template library) tries to have strong exception safety for containers

**Example: `vector::emplace_back()`**
* If the vector has sufficient capacity we simply construct an element at the end of the vector.

What if we run out of capacity?
* We would need to reallocate the vector to one with a larger size
    * This means we have to copy the old elements over
    * Old elements are then destroyed

Why can't I move the elements over?
* If my move initialization **fails** then I am left with a half-moved array.

If move could throw an exception, then moving elements would violate the **strong exception safety guaruntee**.

**In general** we can't move, we must copy.

However, if we have a **no-throw move constructor** then we can move. `emplace_back()` has overloads for both move and copy, and uses `move` if the object's move constructor is no-throw. Otherwise (like if there is no move ctor) it uses copy.

## Summary
* Vector and other STL containers follow **RAII**
* When a vector is constructed one gets a dynamic array
* When a vector is destroyed, the dynamic array is deleted

A vector of pointers does not follow RAII
* When a vector is destroyed, the pointers are not deleted
* This is because the vector does not know how to delete the pointers

**Solution:**
* Use a smart pointer
* `vector<unique_ptr<A>>` will follow RAII
* `vector<shared_ptr<A>>` will follow RAII
