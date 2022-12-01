# Lecture 23 - 01 December 2022

## Living life dangerously in C++

### Recall C
* C is a statically-typed language

#### Casting
```c
// Let's write in C

// Let's make an array of 10 integers
int *arr = malloc(10 * sizeof(int)); // Values in C are just bits in memory

// Types are just HOW you interpret the bits in memory

// a long is 8 bytes
long l = (long) arr; // I can cast the memory address to point to a long

float f = 5.0; 
int i = (int) f; // gives me 5
```

* Casting is both useful, but can be **very dangerous**

ie:
```c
int *v = (int *) 0x40; // is this ok?
```

### `static_cast<T> (v)`
Casts `v` to type `T` as long as conversion is **well-defined**

**ie:**
```cpp
// Back in C++ now
float f = 5.0;
int v = static_cast<int>(f); // v = 5

// Classes
Book *b = newTextBook{"STAT 230"};
Text *t = static_cast<Text *>(b); // only use static_cast() if you ACTUALLY KNOW
                                  // that you can cast it
```

* What if I don't know if I can cast it?

### `dynamic_cast<T> (v)`
* Lets you cast from a pointer to a base class to a pointer to a derived class
    * Also works with **references**
    * If the cast is not valid, it returns `nullptr`
    * If the cast is valid, it returns a pointer to the derived class

```cpp
Book *b = newTextBook{"STAT 230"};
Text *t = dynamic_cast<Text *>(b);
Comic *c = dynamic_cast<Comic *>(b); // c = nullptr

Comic &c = dynamic_cast<Comic &>(*b); // throws bad_cast exception    
                                      // since b can be nullptr
                                      // Having reference to NULL value sucks
```

```cpp
// Effect of f now depends on what subclass of b is
void f(Book *b) {
    if (dynamic_cast<Comic *>(b)) {
        // does code that we know works with comic here
    } else if (dynamic_cast<Text *>(b)) {
        // does code that we know works with text here
    }
}

// Generally, this is not really recommended, but here if you need to use it
// Instead just use visitor pattern or inheritance
```

### `const_cast<T> (v)`
* Lets you cast away **constness**

```cpp
const int grade = 65;

// We want to change this grade :) 
// But we can't ... unless??
int *p = const_cast<int *>(&grade);
*p = 100; // he he he haw

// CPP lets you forget about const here
// grade is still const, but the value is 100 now.
```

```cpp
// More useful use case
void g(int * n) {
    // does not actually mutate what n points to
    // And we know this
}

const int* x;
g(const_cast<int *>(x)); // this is ok
```

### `reinterpret_cast<T> (v)`
* Lets you reinterpret the bits of `v` as a COMPLETELY different type
* Most of the time, the result of this is undefined behavior

```cpp
Student s{"Bob"};

Turtle *t = reintrepret_cast<Turtle*>(&s);
// Same bits as s, but now we can use t as a turtle

t->wackWithStick(); // undefined behavior
                    // Probably won't work as you intended
```

How can we use this to our advantage

```cpp
#include <iostream>
#include <string>
using namespace std;

class Secret {
    string s;
    public:
        Secret(string s) : s{s} {}

};

// s is private so we can't reveal this secret ... unless
void reveal(Secret &s) {
    cout << s.s; // Doesn't work, can't access private field of Secret
}

//... Unless
struct Unsecret {
    string s;
};

// ver 2
void reveal(Secret &s) {
    Unsecret *u = reinterpret_cast<Unsecret *>(&s);
    cout << u->s;
}

int main() {
    Secret s{"earth is flat"};
    
    reveal(s); // Your secret is now leaked!
}
```

Why can we do this?
* It's because `Secret` and `Unsecret` have the same memory layout
* If you **do** want to use the raw bits of another class, `reinterpret_cast` is the way to go

### Something Interesting
```cpp
#include <iostream>
#include <cstdint>
using namespace std;

class A {
    // A has no fields (this is important!)
    A& operator+(const A& other) {
        return *reinterpret_cast<A*> (
            // reinterpret_cast to integers
            reinterpret_cast<uint64_t>(this) + reinterpret_cast<uint64_t>(&other));
    }
};

int main() {
    A* x = reinterpret_cast<A*>(1);
    A* y = reinterpret_cast<A*>(7);
    A& z = *x + *y; // x and y are seen by the compiler as references to A, but are really references to integers
                    // Our operator+ is called, which is also very important
    cout << reinterpret_cast<uint64_t>(&z) << endl; // 8
}
```

```cpp
// Now this program will crash with the bar field
#include <iostream>
#include <cstdint>
using namespace std;

class A {
    int bar;
    A& operator+(const A& other) {
        return *reinterpret_cast<A*> (
            reinterpret_cast<uint64_t>(this) + reinterpret_cast<uint64_t>(&other));
    }
};

int main() {
    A* x = reinterpret_cast<A*>(1);
    A* y = reinterpret_cast<A*>(7);
    A& z = *x + *y;
    cout << reinterpret_cast<uint64_t>(&z) << endl; // 8
}
```
***
### Methods

`objdump  a.out` to see the assembly code

* When we define methods, they are secretly functions, but with a hidden parameter `this`
    * `this` is a pointer to the object that the method is being called on

**But when we make a method `virtual`**
* The compiler implements this differently compared to a normal method

```cpp
#include <iostream>
using namespace std;

struct Vec {
    int x; int y;
    int m() {return x + y; }
};

struct Vec2 {
    int x; int y;
    virtual int m() {return x + y; }
};

int main() {
    Vec x; x.x=1; x.y=2;
    Vec2 y; y.x=1; y.y=2;

    cout << sizeof(x) << " " << sizeof(y) << endl;
    Vec *p = reinterpret_cast<Vec*>(&y);
    cout << p->x << " " << p->y << endl;
}
```
* We get that Vec2 is 16 bytes, but Vec is 8 bytes
* also p->m() returns some random number

**Why is this?**
* When we make myself a method `virtual`, the compiler adds a third hidden field to the class
    * This field is a pointer to a function
    * This function is the implementation of the method
    * This function is located in the **vtable**

```
Book              [vtable]           [correct implementation]
    vtable ----> isHeavy() ------> Text::isHeavy()
                 isHeavy() ------> Comic::isHeavy()
                 ...
    title
    author
    price
    pages
```
* At runtime, the compiler will set the vtable pointer to the correct implementation of the method


## Multiple Inheritance

```cpp
struct Kangaroo {
    int pouch;
};

struct Human {
    string brain;
};

struct KangarooHuman: public Kangaroo, public Human {
    // can define methods that will work BOTH with the Kangroo part and the Human part
    void f() {
        cout << "brain has" << brain << " and pouch has " << pouch;
    }
};

int main() {
    KangarooHuman h;
    h.brain = "rockets!";
    h.pouch = 10;
    h.f();
}
```

* This is really bad
`Kangaroos` and `Humans` are both `Mammals`, so why don't we make them inherit from there?
* Hint: `Kangaroo` and `Human` then each have a `heart`.

```cpp
struct Mammal {
    bool heart;
};

struct Kangaroo: public Mammal {
    int pouch;
};

struct Human: public Mammal {
    string brain;
};

struct KangarooHuman: public Kangaroo, public Human {
    // can define methods that will work BOTH with the Kangroo part and the Human part
    void f() {
        cout << "brain has" << brain << " and pouch has " << pouch <<
        " and heart is " << (heart == true ? string{"On"} : string{"Off"});
    }
};

int main() {
    KangarooHuman h;
    h.brain = "rockets!";
    h.pouch = 10;
    h.heart = true;
    h.f();
}
```
* This thing throws an error because since both `Kangaroo` and `Human` inherit from `Mammal`
* A `KangarooHuman` then has two hearts, since it has a heart from its `Kangaroo` component, and a heart from its `Human` component 

**Fix (disambiguate)**
```cpp
struct KangarooHuman: public Kangaroo, public Human {
    // can define methods that will work BOTH with the Kangroo part and the Human part
    void f() {
        cout << "brain has" << brain << " and pouch has " << pouch <<
        " and Kangaroo heart is " << (Kangaroo::heart == true ? string{"On"} : string{"Off"}) <<
        " and Human heart is " << (Human::heart == true ? string{"On"} : string{"Off"});
    }
};

int main() {
    KangarooHuman h;
    h.brain = "rockets!";
    h.pouch = 10;

    // Initiate each heart separately
    h.Kangaroo::heart = true;
    h.Human::heart = false;
    h.f();
}
```

### Virtual inheritance
**What if we only want to inherit one copy of the heart?**
```cpp
// Mark inhertiances as virtual
struct Mammal {
    bool heart;
};

struct Kangaroo: virtual public Mammal {
    int pouch;
};

struct Human: virtual public Mammal {
    string brain;
};

struct KangarooHuman: virtual public Kangaroo, virtual public Human {
    // can define methods that will work BOTH with the Kangroo part and the Human part
    void f() {
        cout << "brain has" << brain << " and pouch has " << pouch <<
        " and heart is " << (heart == true ? string{"On"} : string{"Off"});
    }
};

int main() {
    KangarooHuman h;
    h.brain = "rockets!";
    h.pouch = 10;

    // Only one heart in KangarooHuman now since we only inherit one copy of the heart
    h.heart = true;
    h.f();
}
```

* If your classes have the **same** class names, then you will still have to disambiguate (`virtual` won't do anything for you)
    * ie: `Kangaroo::heart` and `Human::heart` are still needed if they didn't inherit from Mammal, but each had a `heart` field
        * or if you didn't mark the inheritance as `virtual`
