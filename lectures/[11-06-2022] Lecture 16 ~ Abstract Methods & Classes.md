# Lecture 16 - 06 November 2022

### Last Week
* Inheritance
* Virtual Methods for dynamic dispatch

### Now

```cpp
class X {
        int *a; // int array
    public:
        x(): a{new int[5]} {}
        ~x() { delete[] a; }
};

class Y: public X {
        // Hidden away fields (already in X)
        // int *a;
        int *b;
    public:
        y(): b{new int[7]} {}
        ~y() { delete[] b; }
}

int main() {
    Y *y = new Y();
    delete y;

    // NO LEAKS ABOVE THIS LINE //

    X* x = new Y(); // Here, compiler sees a X
    delete x; // Calls ~X(), just deletes a, won't delete b from Y()
}
```

Just like methods, if we want to call the destructor corresponding to the object's **actual** type, we need to use **virtual destructors**.

**Takeaway:** in classes that can be subclassed, declare destructors to be **virtual**.

### Fix:
```cpp
class Y: public X {
        // Hidden away fields (already in X)
        // int *a;
        int *b;
    public:
        y(): b{new int[7]} {}
        virtual ~y() { delete[] b; }
}
```

For classes which are not to be subclassed
* OK to have non-virtual destructor
* Good to mark class as `final`
```cpp
class Z final {
    // ...
};
```

#### Example
```cpp
class Student {
    protected:
        int courses;
    public:
        virtual int fees() const;
};

// Regular student w/o coop
// Inherits the fields and methods of Student
// Students don't take PD courses, write Coop reports, pay less tuition
class Regular: public Student {
    public:
        virtual int fees() const override {
            // Less expensive fee schedule
            return 1400 * courses;
        }
};

// Student that has coop
// Inherits still the fields and methods of Student
// Co-op students pay more fees, take PD courses, write Coop reports
class Coop: public Student {
    public:
        // More expensive fee schedule
        virtual int fees() const override {
            return 1700 * courses;
        }
};
```

Is there such a thing as a **generic** student that is not coop or non-coop?

**No**. We can't have a student that is neither coop nor non-coop.

**Problem:** We want to model students
* We want a base class that models what can *be done* to a student
    * Defines an **interface** of how a student object should behave.
* We have concrete student subclasses (implementations)
* It doesn't really make sense to ask how methods should behave in base class defining said interface.

> How do we make sure that we never call methods on the base `Student` class?

**Solution**:
* **Just say that the method doesn't have an implementation in the base class**
* Declare the method; if it has no body, just make it `= 0`!
```cpp
class Student {
    protected:
        int courses;
    public:
        virtual int fees() const = 0; // add = 0 to the end
                                      // Tells compiler that these methods
                                      // have no body.
                                      // "Nothing to see here, just a declaration"
};
```
Now, we can create base classes that have no implementation for methods, and we can use them for other methods **only**.
* These methods are known as **abstract methods**

* Abstract classes cannot be instantiated. They can only be used as a base class for other classes.

* **Important!** If ANY method in the class is abstract `= 0`, then the class is abstract.
    * Keep in mind that abstract classes can have non-abstract methods.
    * If another method has no implementation, make it abstract too

ie:
```cpp
Student bob; // will not compile as Student is abstract
```

* Subclasses of an abstract class are **themselves abstract** *unless* they provide overrides for **every** abstract method in the base class.
    * Use `override` keyword

* **Abstract classes** have no implementation for methods, so they cannot be instantiated.
    * They can only be used as a base class for other classes.
* **Concrete classes** must provide implementations for all abstract methods in the base class.

* A class is concrete if it is not abstract

* Alternatively, abstract methods are **pure virtual methods**

***
### UML Diagrams for Abstract Classes

```
// example UML in ascii for the Regular and Coop student class above

               +-----------------+  
               |                 |
               |     Student     |          
               |                 |
               +-----------------+
                        ^
                        |                   
          |--------------------------|
          |                          |
+-------------------+     +-------------------+
|                   |     |                   |
|     Regular       |     |       Coop        |
|                   |     |                   |
+-------------------+     +-------------------+
|        ...        |     |        ...        |
+-------------------+     +-------------------+

```

*** 

### Inheritance and Copy Semantics
```cpp
class Book {
        string title, author;
        int pages;
    public:
        // book has big 5 defined
};

class TextBook: public Book {
        string topic;
    public:
        Text(...);
        // But does not have Big 5 defined
};

int main() {
    TextBook stats = {"Statistics", 
                      "Prof",
                      1000, 
                      "Statistics"}
    
    // Let's make a copy of the textbook
    TextBook pirated {stats};
}
```

### What is going on in memory
```
      [stats]                     [pirated]
+------------------+         +------------------+
|                  |         |                  | // Book's Fields
| - Statistics     |         | - contents will  |
| - Prof           |         | depend on Book's |
| - 1000           |         | copy ctor        | 
+------------------+         +------------------+
| - Statistics     |         | - Statistics     |  // TextBook's field
+------------------+         +------------------+
```

* To copy `Book`'s fields, it calls `Book`'s copy ctor
* To copy `TextBook`'s fields, it calls `TextBook`'s copy constructor by value

* There's similar behaviour for other default big 5 operations.
* Here's how we replicate this behaviour with out own code

## Big 5 for Inheritance
```cpp
// Copy ctor
TextBook(const TextBook &t):
    Book{t}, // Call Book's copy ctor
             // pass whole object, Book's copy ctor
             // can only see the Book portion of t's fields
    topic{t.topic} // Copy TextBook's field
    {}

// Move ctor (first try)
TextBook(TextBook &&t):
    Book{t},  // calls Book's copy ctor, NOT move
    topic{t.topic} // cals string copy NOT move
    {}
// This DOESN'T work because r-value references are l-values.

// There is a way to get around this
// std::move() turns our l-values back into r-value references

// Move ctor (correct)
TextBook(TextBook &&t): 
    Book{std::move(t)}, // calls Book's move ctor
    topic{std::move(t.topic)} // calls string's move ctor
    {}

// do similar thins for move/copy assignment

// Copy Assignment
TextBook& operator=(const TextBook &t) {
    if (this == &t) return *this;
    Book:::operator=(t); // copies Book portion
    topic = t.topic; // copies TextBook portion
    return *this;
}

// Move Assignment
TextBook& operator=(TextBook &&t) {
    if (this == &t) return *this;
    Book::operator=(std::move(t)); // moves Book portion
    topic = std::move(t.topic); // moves TextBook portion
    return *this;
}
```

```cpp
TextBook stats = {"Statistics", 
                  "Prof",
                  1000, 
                  "Statistics"};

TextBook calc = {"Integration",
                 "Prof2",
                 1000,
                 "Calculus"};

stats = calc; // just works

// but now consider //

// Now,
// we did not do stats = calc.
Book *bc = &calc;
Book *bs = &stats;

*bs = *bc; // doesn't work
// Calls the Book copy assignment operator
// Assigns Book portion correctly
// But does not touch the field from TextBook (topic)

*bs = *bc; // This actually points to a
           // calculus-titled book about statistics
```
* What we just did at the bottom is a **partial assignment**
    * We only assigned the Book portion of the object
    * We did not assign the TextBook portion of the object
* What if we made assignment virtual?

### Back to Big 5
```cpp
class Book {
    public:
        virtual Book& operator=(const Book &b);
};

class Text {
    public:
        // To override, we need to override the base class's method
        // Our header would have to look like this
        virtual Book& opreator=(const Book &b) override;
        // Can't construct/assign ANY book to a textbook
};
```

```cpp
Comic c = {"Stan", "Thor", 40, "Comedy"};
Text t;
t = c;
// This would work with our definition above since
// Comic is a Book and Text is a Book, but THIS DOESNT MAKE SENSE
// Therefore, we can't make = virtual
```
* By making assignment virtual, we do something called **mixed assignment**
    * Allowing assignment between all subclasses of the same base class.
    * Probably not what you want
* We shouldnt be able to copy a comic book into a textbook.
* We *maybe* can assign a textbook to a book, and just chop off the extra fields, but this might not be good.

**Solution**
* Make assignment inaccessible through book.

```cpp
class Book {
    protected:
        operator= // ...
};
```
* Make it `protected` so my derived classes can use
    * Protected means derived classes can use fields and methods
