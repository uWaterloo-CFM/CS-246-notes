# Final Review Session - 12/14/2022
* Extra sadge

#### Exam tip: 
* Do the questions you know how to do **first**

## Topics
* Multiple Choice
* T/F
* Short ans
* Long ans

### Multiple Choice
1. Which of the following doesn't use an interator to traverse a vector `v`?
- copy from slides 
Ans: (b)

2. How do we rethrow the same exception received no matter the type of the exception handler?
```cpp
catch(ExceptionType s){...}
```

Ans: (a) - `throw s` could rebuild the exception object, but it's not the same exception object

### True/False (explain why False)

1. The iterator pattern is used to abstract away implementation details while providing an interface for a user to traverse a data structure
* True

2.  The `auto` keyword allows a user o automatically allocate memory to an object
* False - `auto` is a type specifier that allows the compiler to infer the type of the variable


3. Composition is a has-a relationship, and object A has a B means that when A is deleted B isnt deleted
* False - Aggregation is a has-a relationship, composition is a owns-a relationship
* Aggregation is a has-a relationship (empty diamond)
* Composition is a owns-a relationship (filled diamond)

4. A subclass cannot access the private fields of the superclass
* True

5. The correct order when an object is created is:
    1. Space is allocated
    2. Subclass fields are created in order of declaration
    3. Superclass constructor is invoked
    4. Constructor body runs
* False, Superclass constructor is invoked second

6. **Slicing** refers to when a subclass constructor is used to create a superclass object and the subclass fields are lost
* True

7. `std::move(x)` is an rvalue
* True

8. An abstract class is a class that another class inherits from
* False, an abstract class is a class that has at least one pure virtual function. It can be inherited from.

9. Exception safety is when a program can throw an exception without crashing
* False, Exception safety is when a program can throw an exception without crashing and without leaving the program in an inconsistent state

10. We want low coupling and high cohesion
* True

### Short Answer
1. Why is it generally not a good idea to use protected fields?
* could potentially violate invariants, violates encapsulation, better to have protected getters and setters

2. Define polymorphism, why is it useful?
* Polymorphism describes being able to create different objects that share similar interfaces, but have different implementations. It is useful because it allows us to create a single interface that can be used to interact with different objects that correctly dispatch the correct method for the specific object type. All objects that share the same interface can be referred to as the superclass.

3. Why is it important to declare teh superclass destructor virtual?
* We want to make sure that the correct destructor is called, which properly cleans up the object.

#### pImpl
4. Change the following code for Vec into a `pImpl` implementation
```cpp
// vec.h
class Vec {
    int x, y;
    public:
        int getX();
        int getY();
};

// vec.cc
#include "vec.h"
int Vec::getX() { return x; }
int Vec::getY() { return y; }
```

**soln**
```cpp
// vec.h
class VecImpl;
class Vec {
    VecImpl *pImpl;
    public:
        int getX();
        int getY();
};

// vecImpl.h
class VecImpl {
    public:
        int x, y;
};

// vec.cc
#include "vec.h"
#include "vecImpl.h"
class VecImpl {
    int x, y;
    public:
        int getX() { return x; }
        int getY() { return y; }
};

int Vec::getX() { return pImpl->x; }
int Vec::getY() { return pImpl->y; }
```

5. Does this code call Book's move constructor? If not, why not?
```cpp
Text::Text(Text &&other) : Book{other}, topic{other.topic} {}
```
* Other is passed in as a r-value reference, so the object `other` itself is an l-value, even though it points to an r-value.
* If you want to use `std::move()` to turn the l-value into an r-value.

6. For an lvalue object x, what does std::move(x) do?
* casts x to an rvalue reference type, saying that x can be moved.

Implementation of `std::move()`
```cpp
#include <type_traits>
template <typename T>
typename std::remove_reference<T>::type&& 
move(T&& t) {
    return static_cast<typename std::remove_reference<T>::type&&>(t);
}
```

7. What problem does this code cause?
```cpp
class Book {Book& operator=(const Book &other);}
// TODO Copy from slide 33
```

8. Name one way to solve the mixed/partial assignment problem
    1. Dynamic cast, catch exception; not your problem
    2. Abstract parent class, assignment operator is protected

9. Does the following code compile? Why or why not? What gets produced if it does?
```cpp
template<typename T>
bool lessThan(T t, T s) {
    return t < s;
}

bool b = lessThan(3, 3.5f); // casts 3 to float
```
* This code does not compile since it is unable to deduce the type `T`. We can fix this by explicitly specifying the type `T` as a template parameter.  `bool b = lessThan<float>(3, 3.5f);`

10. Is this code exception safe?
```cpp
void f() {
    MyClass mc;
    MyClass *p = new MyClass;
    g(); // strongly exception safe
    h(); // strongly exception safe
    delete p; // p gets leaked if g or h throws
    // if h throws, g is not reversed
    // so the function is not strongly exception safe
}
```
* to make this code exception safe (basic exception guarantee), we make p a smart pointer.

11. What are the three levels of exception safety?
* Basic exception guarantee - no memory leaks, no dangling pointers, no data corruption
* Strong exception guarantee - no memory leaks, no dangling pointers, no data corruption, no resource leaks
* No-throw guarantee - no memory leaks, no dangling pointers, no data corruption, no memory leaks, no exceptions thrown

12. Name two reasons why C style casts are dangerous
* Can't figure out where the cast is happening (control-F)
* They are too powerful, and ripe for abuse
* at least when you use `reinterpret_cast` you know that you are doing something dangerous

**Issues with casting (in general, not just C)**
* Can access memory beyond the bounds of the object
* Can cast away constness/private-ness


## Long Answer Examples (Design Patterns)
1. Iterator
```cpp
// for Iterator, we need 5 things:
// 1. operator++
// 2. operator*
// 3. operator!=
// 4. begin()
// 5. end()

// you can see all 5 of them in the for loop
for (auto it = v.begin(); it != v.end(); ++it) {
    cout << *it << endl;
}

// but we use this loop
for (auto i: v) {
    cout << i << endl;
}
```

**Example**
```cpp
// copy from repo
```

2. Visitor Pattern + Observer/Subject
**Pokemon Example**
```cpp
// TODO: Copy from Slides
```
