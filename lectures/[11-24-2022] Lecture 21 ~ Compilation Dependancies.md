# Lecture 21 - 24 November 2022

## Compilation Dependencies
```cpp
class XWindow {
    Display *d;
    int s;
    GC gc;
    Window w;
    unsigned long colours;
};
```
**Implementation** 
* Details of te XWindow class

Inaccessible, but they still are visibile; ie: we still need to `#include X11/Xlib.h` to use the class.

**Compilation Dependencies**
```
a.h
    class A {...};

b.h
    class B {...};

c.h
    class C {A field;}; // i.e class A

d.h
    class D {A * field}

e.h
    class E {A method(A);}
```

For class B in `b.h`, we include `a.h` as we need `A`'s full declaration in order to extend it so that we can determine how objects of type `B` are laid out.

For class `C`, while we don't care about `A`'s methods, we need to `include "a.h"/A`'s full declaration as we need `A`'s size to lay out C's memory.

**Now**, in class `D`, a pointer to `A` has a fixed size that is known ahead of time. We only care that `A` is a valid type, *hence* a forward class declaration is sufficient.

**NB** Your compiler needs to know all of this AHEAD of time, it can't dynamically create space once it figures out how much memory is needed.

That is, remember **not** to have cyclical references! (See Tutorial 10 for more details)

***

## pImpl Idiom
* We can fix the problem of cyclical references by using the **pImpl idiom**.
Decouples the interface from the implementation. (Can hide the fact that you need to include other libaries to the user)
    * ie: `X11 XWindow from A4`


We can shift fields in a `XWindowImpl` class and use a forward declaration.

```cpp
// XWindowImpl.h
struct XWindowImpl { // maybe this should say class?
    Display *d;
    int s;
    GC gc;
    Window w;
    unsigned long colours;
};

// someotherfile.cc
#include "XWindowImpl.h"

class XWindowImpl; // forward declaration
class XWindow {
    XWindowImpl *impl;
};

XWindow::XWindow() {
    impl {XWindowImpl{...}};
}

XWindow::~XWindow() {
    delete impl;
}
```

* Field access now goes through the `impl` pointer.

***

## Design Quality

* **Coupling** - a measure of how modules depend on each other.
    * When code is **tightly coupled**, it becomes hard to reason/debug/test moduled individual, independently, in isolation.
        * If you code is **tightly coupled**, it is usually a sign of bad design.
        * ie: structs and arrays are tightly coupled, sharing data using global variables/changing control flow is tightly coupled.
            * ie: using structs and arrays AS INTERFACES
            * returning structs and arrays from functions should only be **read only**.
    * **Loose coupling** is desirable.
        * **Loose coupling** is achieved by:
            * Modules communicate over well-defined, well-encapsulated interfaces
                * Functions returing basic data/objects
            * More objects to encapsulate private implementation details

    * Code that is tightly coupled share implementation details (friend classes)

* **Cohesion** is a measure of how closely elements of a module are related to each other.
    * Code which is not cohesive is often disorganized and hard to work with.
    * **Low cohesion** examples:
        * `<utility>` - no theme, just a bunch of random things
        * `<algorithm>` - more cohesive (but not by much), has a theme at least

    * **High cohesion** is when you have modules wich are reponsible for **one** well-defined task
        * Examples:
            * `<vector>`
            * `<string>`
            * `<map>`

## Chess Example
```cpp
class ChessBoard {
    ...
    // should ChessBoard be responsible for printing the board?
    cout << "Board is " << board << endl;
};
```
* No, our board is coupled tightly with the means of printing it. (should implement an observer???)

```cpp
class ChessBoard {
    istream &in;
    ostream &out;
    ...

    out << "Board is " << board << endl;
}
```
* This example is less coupled, and more **easily** testable.
    * We can change what streams we give to the `ChessBoard`

But this is still not great, `ChessBoard` is coupled to *textual output**
* We can't use it for graphical display

Ideally a chessboard should not be concerned with how it's being displayed.

Ideally for high cohesion/low coupling classes, adhere to the **single responsibility principle**.

**So where?**
* `main` - no, as it's hard to reuse.

## MVC Pattern
* In this model, instead of the chessboard printing to the screen, have a `BoardView` class and perhaps **a method for returning the pieces and positions on a board**.

## Exception Safety
```cpp
void f() {
    string a = "Hi";
    string *b = new String {"World"};
    g();
    delete b;
}
```
* Does `f()` leak memory?

What if `g()` throwns an exception?

We unwind stack through f hopefully destoring a, but we don't delete b

* We can solve this with a try/catch g()

```cpp
try {
    g();
} catch (...) {
    delete b;
    throw;
}
```
* This is fragile
*"Some companies won't let you do this, like Google. At least it was like that when I did my internship there"* - Ed

