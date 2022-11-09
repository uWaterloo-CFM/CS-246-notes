# Tutorial 08 - November 09, 2022

## To-do
1. Check over midterm for any errors
2. If question has no feedback, I can request for it
3. Regrade deadline is November 10, 2022

## Virtual Methods
* By default, compiler chooses the method based on the pointer type
* To ensure that the right method runs no matter what, use the `virtual` keyword
* only have to define `virtual` **once at the most** base class
* use `override` keyword when defining methods in subclass
* if a class is **not meant to have subclasses**, you can declare it `final`
* `protected` members can be accessed by **subclasses** (like public for subclasses)

```cpp
class Base {
    public:
        void foo() { cout << "Hello" << endl; }
};

class Derived: public Base {
    // ...
    void foo() {
        cout << "bye" << endl;
    }
};

Base *d = new Derived;
d->foo(); // prints "Hello" from Base::foo(), NOT from Derived::foo()

Derived *d2 = d;
d2->foo();
```

**How to fix?**
* Use `virtual` keyword in `Base::foo()`

```cpp
class Base {
    public:
        virtual void foo() { cout << "Hello" << endl; }
};

class Derived final: public Base { // Have final keyword so we can't derive classes from Derived
    // ...
    void foo() override { // override doesn't do anything in program
                          // CHECKS to see if function signature matches virtual function
                          // Have it to make sure you don't mess up!
        cout << "bye" << endl;
    }
};
```
## Polymorphism
* By using `virtual` we ensured that the object type determines the metod ran, not the pointer type
// ... COPY FROM TUT8 Slides - Ahmad

### Example
(might help on A4)
```cpp
vector <Books *>b;
// ... fill b with lots of stuff

for (auto n:b) {
    // Will output isHeavy depending on the class of Book that is currently being iterated
    cout << n.isHeavy << endl; 
}
```

## Destructor
* To ensure that deletion through a superclass pointer works properly, we can declare the destructor to `virtual`.
* You can use the override keyword in subclass but you don't have to
    * Destructors always have the same signature
* ALWAYS make base class destructors `virtual`

```cpp
class X {
    int *x;
    X() {
        x = new int[5];
    }
    ~X() {
        delete[] x;
    }
};

class Y: public X {
    int *y;
    Y() {
        y = new int[5];
    }

    ~Y() {
        delete[] y;
    }
};

X *xy = new Y();
delete xy; // will only call X's destructor since xy is type X *
           // deletes x from X but not y from Y. Memory leak!
           // It doesn't know that y exists since it's a pointer to X
           // ALWAYS make dtor's virtual!
```

```cpp
class X {
    int *x;
    X() {
        x = new int[5];
    }
    virtual ~X() { // NOW it will call Y's destructor
        delete[] x;
    }
};
```

## Copy Constructor/Assignment
* The default (provided) copy constructor calls the superclass copy constructor
// TODO COPY FROM TUT 8 AND COPY EXAMPLE (Same as Ed's notes from LEC 16)


## Move Constructor/Assignment
// TODO COPY FROM TUT 8
* `std::move()` is needed
> Why?
* An R-value reference is a l-value the points to a R-value
* `std::move()` converts a l-value to an r-value
    * Allows you to tran an lvalue object like an rvalue
    * Used to force move operations on objects
    

```cpp
move (T &&other) {
    return other;
}
```

## Partial and Mixed Assignment
* Happens if you implement your = incorrectly
* Partial assignment is when you only assign some of the members of the class
// COPY FROM SLIDES

// MIGHT BE FINAL QNS^^
// WILL BE QN 2 on Q4

**How to implement `operator=` properly**
```cpp
class abstractBook {
        string author, title;
    protected:
        abstractBook &operator=(const abstractBook &other):
            author(other.author), title(other.title) {
                return *this;
            }
        // ...
};

class Comic: public abstractBook {
        string hero;
    // ...
    public:
        Comic &operator=(const Comic &other) {
            // Copy the fields from base class
            abstractBook::operator=(other);

            // Copy the remaining fields
            hero = other.hero;
            return *this;
        }
}
```

## Abstract Classes
// COPY FROM SLIDES
// READ THE UML STUFF - IMPORTANT

## Pure Virtual Methods
```cpp
class abstractBook {
        string author, title;
    protected:
        abstractBook &operator=(const abstractBook &other):
            author(other.author), title(other.title) {
                return *this;
            }
        // ...
    ~abstractBook() = 0; // Pure virtual destructor
};
```


## Templates
// TODO COPY FROM SLIDES

```
// UML FOR EXPRESSION

+--------------------------------+
| Expression                     | // italisize if pure virtual
+--------------------------------+
| - subexpression: Expression*   | // fields
+--------------------------------+
| +*prettyPrint()*: void         | // methods (italisize for pure virtual)
+--------------------------------+
                  ^
                  |
                  |
                  ----------------
                                    | IntExpression
                                    +------------------------+
                                    | iterator: Iterator     |
                                    +------------------------+
                                    | +*prettyPrint()*: void |
```
