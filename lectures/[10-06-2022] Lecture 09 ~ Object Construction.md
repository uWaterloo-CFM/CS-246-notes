# Lecture 09 - September 06, 2022

## Data Structures in C++
```cpp
// in header file
struct Student {
    string name;
    int grade;
    Student(string &name) {
        // name already initialized to hold empty string.
        this->name = name; // reassigns name
    }
}
```

### You can't do these things
```cpp
struct MyStruct {
    const int stat_230_grade; // can't reassign in constructor body

    Textbook &stat_230_book; // same - how can we initialize this?
}
```

## Object Construction

```cpp
struct Student {
    string name;
    int grade;
    Student(string &name) {
        // name already initialized to hold empty string.
        this->name = name; // reassigns name
    }
}
```

```cpp
Student bob {"Bob"};
```
> How does C++ initlize this object?
1. Get memory for object 
    * `bob` is placed on the stack
2. Construct fields using default constructors
    * `name` is constructed with `string()`
3. Constructor body runs
    * In this example, reassigns name

We can control how fields are initialized/constructed using the **member initialization list** (MIL).

```cpp
// constructor: goes in .cc file
Student::Student(string name):
    name{name}, grade{100}

{
    // body goes here
    // or it can be EMPTY as we already initialized our fields
    //  to the right values
}
```

* As this avoids unnecessary construction, use the **MIL** to initialize when you can.
    * Initializing very large/complicated objects can be very slow
* Construction order is **not** specified by the **MIL**
    * It depends on the order that the fields appear in the class declaration.
    * **Exercise:** Try to add cout calls to your constructors


```cpp
struct Student {
    string name;
    int grade;
    Student(string &name) : name{name}, ... // can have MIL in .h file

    {
        // name already initialized to hold empty string.
        this->name = name; // reassigns name
    }
}
```

### Passing Objects by Value
```cpp
Student bob {"Bob"}
Student bobclone {bob}

void graduate(Student x);
graduate(bob); // This passes in a COPY of bob
```

* `bobclone` and `graduate` both construct copies of bob.

Since **copying** is supervasive, C++ provides a default **copy constructor** to construct objects by copying them.

* Formally, the copy constructor *[for class `C`] is a 1-arg constructor expecting a class `C&`

### Passing by Value instead of passing by value
* Remember, passing by value requires a copy.
```cpp
Student::Student(const Student &other):
    grade{other.grade}, name{other.name}
{}
```

### Custom Copy Constructor
* You might want to create a custom copy constructor if you need to copy a data structure like a **linked list**
    * If you copy by value, then your pointer will point to the original next element in the list

```cpp
struct Node {
    int v;
    Node *n;
}
```
* If we use the defauly copy constructor, we will copy linked lists that share their tails.
* The behaviour we want is for the copy to be an entirely disjoint linked list with the same value as the original linked list.

> So, let's create a custom copy constructor

```cpp
// Header
struct Node {
    int v;
    Node *n;
    Node(const Node &other);
}
```

```cpp
// .cc file
Node::Node(const Node &other):
    v{other.v}, n{!n? nullptr; new Node{*other.n}}
{
    
}
```

> Now, let's call the copy constructor

```cpp
Node *n = new Node{*other.n}
```
* When you initialize from another object
* When you copy an object when passing by value
* When an object is returned by value from a function

```cpp
// Initialize linked lists f and g
Node f{...}
Node g{...}

f = g;
```
* The copy constructor is not called when assigning g to f
* This instead calls the assignment operator (operator `=`)
* The compiler provides a default implementation if none is given

The default implementation just copies fields by value.
* So in `f = g`, by default it would:
    * `f.value = g.value`
    * `f.next = g.next`

**For Nodes, this is BAD**
* Results in dangling pointers. 
    * The original `f.next` is dangling. We have lost the reference to that pointer.
* So for our `Node` class (and in general), the assignment operator needs to:
    * Copy `n`
    * Delete old tail
        * The data is discarded after assignment

### Defining the Assignment Operator

#### Attempt 1
```cpp
Node& Node::Node operator =(const Node &other) {
    delete next;
    value = other.value;
    next = other.next? new Node{*other.next}; nullptr;
    return *this
}
```
* This implementation would not work if we did `f = f`
* Self assignment can be dangerous.
    * Here, other.next is next, so the memory is deleted.
        * Before the assignment operator can copy it.

* What if the call to `new` fails?

**Also**, if the allocation in assignment fails, `next` is left pointing to deleted memory.

#### Checking for self-assignment (Take 1)
* Add a guard to check whether or not you are setting it to itself.
    * If so, just `return *this`

**Modified (version 2)**
```cpp
Node& Node::Node operator =(const Node &other) {
    if (this == &other) return *this;

    delete next;
    value = other.value;
    next = other.next? new Node{*other.next}; nullptr;
    return *this
}
```

#### Inconsistent State after Failiure
* Only delete after successfully allocating/making copies

**Modified (version 3)**
```cpp
Node& Node::Node operator =(const Node &other) {
    if (this == &other) return *this;

    Node* tmp = next;
    next = other.next? new Node{*other.next}; nullptr;

    delete tmp;
    value = other.value;

    return *this
}
```

### Assignment-by-copy
```cpp
Node& Node::operator =(const Node &other) {
    Node tmp{other};
    swap(tmp.value, value);
    swap(tmp.next, next);
    return *this;
}
```
* This implementation avoids problems with sef-assignment
    * `tmp.next != this->next`
* but we never freed **tmp**?
    * What happens to the old `next`?
    * We stuffed the old tail (`next`) into `tmp`
* It would be nice to get `tmp` to clean up after itself after the function returns

* Our original implementation also **DID NOT** recurseively clean up the entire tail.

### Cleaning up in a class - Destructors
* We can override destruction behaviour by giving a **destructor** called when an object leaves scope or when an object is deleted.