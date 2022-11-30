# Lecture 11 ~ 30 November 2022

* Notes: There will be a final exam review session by Ahmad probably December 13/14.
* **NB**: Exam tip: Read the Tutorials to do better on Exam (duh!)
    * Why do we need to use RAII
    * What problem does RAII solve?
    * What is the issue with the visitor pattern?
        * Repetitive code (but where?)
    * What problem is X design pattern designed to solve?
    * What is the issue with the X design pattern?


## Topics
* pIml Idiom
* Measures of Design QUality
* Model-View-Controller
* Exception Safety
* Exception Safety Guarantees

### pImpl Idiom
* Oftentimes, clients don't really care about te private implementation details of your object
* You can hide the implementation details of your object 
    * Most of the time, you don't need to expose the implementation details of your object
* You can therefore use a **pointer to implementation object** (pImpl)
* Client doesn't have to recompile .h files if pIml changes, only whatever .cc files use the pImpl
    * Recompile .h files means that the user doesn't have to change the content of the .h file so when you **include** it, it isn't recompiled

#### Examples

`XWindowImpl.h`
```cpp
#include <X11/Xlib.h>
struct XWindowImpl{
    Display *d;
    Window w;
    unsigned long colours[10];
};
```

`Window.h`
```cpp
class XWindowImpl;
class XWindow {
    XWindowImpl *pImpl;
    public:
        ...
};
```

`Window.cc`
```cpp
#include "Window.h"
#include "XWindowImpl.h"
XWindow::XWindow(...) : pImpl{ new XWindowImpl} {...}
//Other methods use d, w, s, etc fields
//Replace with pImpl -> d, pImpl -> w etc.
```

### Measures of Design Quality

#### Coupling
* How much distinct program modules depend on each other

(Sorted from Low to High Coupling)
**Low Coupling**
* Modiles communicate via function calls with basic parameters/results
* Modules pass arrays and structs back and forth
* Modules affect each other's control flow
* Modules have access to each other's implementation (*friendship*)
**High Coupling**
***
* High Coupling `=>` changes to one module require greater changes to the other module
* High Coupling makes it harder to reuse inidividual modules (We really like to reuse code)
* **You want low coupling**

#### Cohesion
* How closely elements of a module relate to each other

(List sorted by low cohesion to high cohesion)

**Low Cohesion**
* An arbitrary grouping of unrelated elements (`<utility>`)
* Elements share a common theme, are otherwise unrelated (`<algorithm>`)
* Elements manipulate state over the life of an object (file manipulation)
* Elements pass data to each other
* Elements cooperate to perform exactly 1 task (Parse a string into a tree) 
**High Cohesion**

***
* Low Cohesion `=>` Poorly organized code, hard to understand
* Low Cohesion makes it harder to reuse the module
* **You want high cohesion**

### Model-View-Controller
* Example with low coupling and high cohesion
* Separates the distinct notions of (High cohesion, low coupling):
    * The data (or state - "model")
    * The representation of the data ("view")
        * Typically this is done with the **observer pattern**
    * The control for the manipulation of the data ("controller")
* Model can have multiple views (text and graphics)
    * Observers (views) do not really care about the details of the models
    * Just make it display whatever it is given
* Controller mediates control flow through model and view
* MVC promotes reuse by this decoupling (view and model can be reused for literally anything)
**This is technically not a design pattern**
* It's an architectural model that uses design patterns

![MVC example](https://i.gyazo.com/ec67b11935213d2bf8513fd413e5703f.png)

### Exception Safety
// TODO COPY FROM TUT 11 SLIDES
