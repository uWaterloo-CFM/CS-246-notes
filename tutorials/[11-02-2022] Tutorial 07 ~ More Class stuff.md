# Tutorial 07 - 11 November 2022

## Topics
* Nested Classes
* Iterators
* System Modeling
* Inheritance

## Nested Classes
* Can define a class inside of another class
* Allows one to restrict usage of a class from an outsider
* Often used to achieve encapsulation

`list.h`
```cpp
class List{
    struct Node; //List::Node not accessible outside List
    Node* theList = nullptr; int length = 0;
    public:
        void addToFront(int n);
        ...//remaining fields
}
```

`list.cc`
```cpp
#include "list.h"
struct List::Node{ // Nested Class
                   // We can access this because it was defined in the .h file
    int data;
    Node* next;
    ... //remaining definitions
    ~Node(){delete next;}
    }

    void List::addToFront(int n){
        theList = newNode{n, theList};
        length++;
    }
```

## Iterators
* Sometimes, encapulsation can cause problems
* Traversal is simple enough for a linked list or array, but more complex structures traversal may require more information that is locked behind encapsulation
* But encapsulation is too vital to program safety to remove
* We can use the iterator pattern to solve this issue

### Instead, use Iterator pattern
```cpp
class List{
    struct Node; Node *theList = nullptr; int length = 0;
    public:
        class Iterator{
        Node *p;
    public:
        Iterator &operator++(){p = p -> next; return *this;}
        bool operator !=(Iterator &other) const{return p!= other.p;}
        int &operator*(){return p -> data;}
        }
    
    // These are private, used to help implement other public methods
    Iterator begin{ return Iterator {theList};}
    Iterator end(){ return Iterator{nullptr};}
};

// Client
int main(){
    List l; l.addToFront(1); l.addToFront(2); l.addToFront(3);
    for(List::Iterator it = l.begin(); it != l.end(); ++it){
        cout << *it << " ";
    }
}
```
* Possible Final question: Why is the Iterator pattern used?
    * Does everything without the User knowing how everything works
    * Abstracts away the implementation details

### `auto`
* Infers the type of the variable from the rvalue
```cpp
int y = 5;
auto x = y // int
```

**Shortcut for Iteration over Objects**
* C++ provides the auto keyword as a shortcut for iteration over an
object
* Available for any class with:
    * `begin()` and `end()` methods that produce `Iterator` objects
    * `Iterator` objects must have `++`, `!=`, and `*` operators defined
Usage:
```cpp
List l;
// construct l ...

for(auto it = l.begin(); it != l.end(); ++it){
    cout << *it << " ";
}
// Or use the range-based method
// for (auto element : Name of Object){ stuff with element ... }
// add & next to element if you want to REFERENCE the values
// when dealing with element, they are dereferenced DEPENDING on what
//  your operator* does.
for (auto &n : l){
    cout << n << " ";
}
```

## `friend`
* Previous implementation allows a client to directly create an iterator
* This violates encapsulation; they should only be allowed to use `begin()` and `end()`
* Can't just make the constructor private, then `List` can't use it
* We can use the `friend` keyword to allow `List` to access `Iterator`'s constructor

```cpp
class List {
    // ...
    public:
        class Iterator{
                Node *p;
                Iterator(Node *p): p{p} {}
            public:
                // ...
                friend class List;
                // Only List can access Iterator's private constructor
                // but Iterator is nested in List, so Iterator can access
                //  List's private fields
        };
    // ...
};
```

* List can still make iterators (because they're friends), but client can only call begin/end
* Give your classes as few friends as possible as it weakens encapsulation
* If you need to provide access to private fields make accessor/mutator methods

```cpp
class Vec{
        int x, y;
    public:
        int getX() const{return x;} // accessor
        void setY(int z){y = z;} // mutator
};
```

* Keep in mind the security vulerabilities you can expose using `friend`
* Doing it in a "Hacky" non-thought out way can cause unexpected side-effects that can violate your class's invarients. 
    * (ie: initializing some fields to nullptr)
    * Setting fields to negative values (or out-of-bounds)
    * Cyclical pointers
* If you need to give your friend access to a private field, make a method that returns a reference to that field
    * Can add checks to make sure that you are not violating the class's invarients

## Making operators friends
* No need if you have accessors
* Otherwise you can define the operator as a friend 
    * If you're ever just like "frick it"
        * But this is often a bad idea
```cpp
class Vec{
        // ...
        friend ostream &operator<<(ostream &out, const Vec& v);
    };

// defined outside class
ostream &operator<<(ostream &out, const Vec& v){
    return out << v.x << " " << v.y;
    // Can just use accessors here instead of making the operator a friend
}
```

## System Modeling (UML)
### Composition
* Known as an “owns a” relationship
* If A owns a B, then typically B has no existence outside of A
    * If A is destroyed, so is B
    * If A is copied, B is copied (deep copy)
Eg. A yakuza family owns its yakuza
![](https://i.gyazo.com/2ef15808e7f17321644ddb0d9485c31d.png)

### Aggregation
* Known as a “has a” relationship
* If A has a B, then typically B can exist outside of A
    * If A is destroyed, B lives on
    * Copies of A are shallow copies (both share the same B)
* Eg. a Yakuza family has a business
    * If family goes down, business is probably fine, someone else picks it up
    * If business goes down, the family is probably fine
* **Shallow copy the pointer, don't make a new object and use a different pointer to a seperate deep copy of the object**

    ![](https://i.gyazo.com/b8450f1fb2567f3458f5d7fbc96c65cd.png)

### Encapsulation
* We can abstract functionality away from the user and let them interact with our objects using only defined functions
* This lets us regain control of our objects and ensure they are used properly


## Inheritance
    * Also known as specialization

### Specialization (Inheritance)
* Can define a “base class” (superclass) that covers functionality common to many ”derived classes” (subclass)
* Subclasses get all fields that the superclass has, as well as all its methods
* Can be used for any method that required the superclass
* Subclasses cannot access `private` fields of the superclass
* Can define fields as `protected` which allows subclasses to access them
    * use is similar to `private` / `public`, stuff indented in them
    * Usually not the best idea for safety reasons
    * Make accessors/mutators instead

```cpp
// Superclass (Book)
class Book{
        string title;
        string author;
        int length;
    protected:
        string getTitle const;
        void setAuthor(string newAuthor);
    public:
        Book(string title, string author, int length);
        bool isHeavy() const;
};

// Subclasses
class Text : public Book{
        string topic;
    public:
        Text(string title, string author, int length, string topic) : 
            Book{title, author, length}, topic{topic}{}
    };
    class Comic : public Book{
            string hero;
        public:
            Comic(string title, string author, int length, string hero) : 
                Book{title, author, length}, hero{hero}{}
};
```

**What's the point of Inheritance?**
* We can now easily define an array or linked-list of `Book`s and have contents be any subclass of `Book` as well.

### Updated list of things that happen when an object is created:
1. Space is allocated
2. Superclass constructor is invoked **(New)**
3. Fields are created in declaration order
4. Constructor body runs

## Inheritance Modeling
* Known as an "is-a" relationship (A `Text` is a `Book`)
![](https://i.gyazo.com/ebc60064b74f06371220eb7e88af3275.png)