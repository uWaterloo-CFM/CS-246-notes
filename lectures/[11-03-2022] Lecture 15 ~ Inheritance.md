# Lecture 15 - 03 November 2022

```cpp
class Book {
        string author, title;
        int pages;
    public:
        // ctr
        Book(string author, string title, int pages);
};

class Novel {
        string author, title;
        int pages;
        string genrel;
    public:
        // ctr
        Novel(...);
};

class Textbook {
        string author, title;
        int pages;
        string topic;
    public:
        // ctr
        Textbook(...);
};
```

* We have a lot of code duplication

**What can we do?**

*Ideas*
* A `Book` is a `Book`
* A `Novel` is a `Book`, but with some extra fields
* A `Textbook` is also a `Book`, but with some extra fields

## Inheritance
* Objects are often natural extensions of other objects
```
Book |--> Textbook
     |--> Novel
```
* Inheritance is a way to model this relationship

### Implementation

```cpp
// Updated class headers that now inherit from Book
class Book {
        string author, title;
        int pages;
    public:
        // ctr
        Book(string author, string title, int pages);
};

class Novel: public Book {
        // remove fields that already exist in Book
        // [x] string author, title;
        // [x] int pages;
        string genrel;
    public:
        // ctr
        Novel(...);
};

class Textbook: public Book {
        // remove fields that already exist in Book
        // [x] string author, title;
        // [x] int pages;
        string topic;
    public:
        // ctr
        Textbook(...);
};
```

**When we implement inheritance:**

    (ie: When we make `Textbook` (or `Novel`) inherit from `Book`)

* We give both `Textbook` and `Novel` a copy of `Book`'s fields and Methods
* This allows us treat `Novels` and `Textbooks` as `Books` as well.
* Any method that can be invoked on `Books` can be invoked on `Textbooks` and `Novels`
    * Since there's a portion of the `Textbook` or `Novel` instance that looks **exactly** like a `Book` instance.
    * Therefore you can run **any** method that works on `Book`!!

**In memory**
```
// A full Novel object
+-----------------+
| Full Book       |
| Object          |
|-----------------|
| extra Novel     |
| data            |
+-----------------+
```

This poses a question:
> How do we construct `Textbook` or `Novel`?
* Do we have to initialize the fields of ALL fields (including those inherited from `Book`?)

**First Attempt**
```cpp
// Just brute-force initialize all fields in Novel and Book collectively
class Novel: public Book {
        string genre
    public:
        Novel(string author, string title, int pages, string genre):
            author{author}, title{title}, pages{pages}, genre{genre} {}
};
```
* Does this work?
    * No, because `author`, `title`, and `pages` are private fields in `Book`
        * We can't access them directly
    * **What actually happens:** When we construct a `Novel`, we also construct a `Book`
        * As Book has no default constructor, we need to **specify** how to build this `Book` object.
        * We do this by passing the `author`, `title`, and `pages` to the `Book` constructor.
        * Refer to memory diagram if this is hard to understand

**Problem**: Every `Novel` contains a `Book` that must be constructed

**Solution**: Use MIL
```cpp
// Use MIL
class Novel: public Book {
        string genre
    public:
        Novel(string author, string title, int pages, string genre):
            // initialize the Book portion of Novel
            Book{author, title, pages}, 
            Genre{genre} {}
};
```

### How objects are *actually* built
1. **Before** (lies and deceit)
    i. Reserve Space for the object

    ii. Initialize the object's fields according to MIL

    iii. Run constructor body
    
2. **After** (the truth) - Inheritance
    i. Reserve Space for the object
    
    ii. Before initializing fields, construct base **subobjects**

    iii. Now we can initialize the **base object's** fields according to MIL

    iv. Run constructor body

    * Here, **subobject** is the `Book` portion of `Novel`
    * **base object** is the `Novel`

**How do we access the private fields of `Book` in `Novel`?**

**Solution 1**: Use *protected* fields

* `protected` weakens private to grant access to fields from yourself, friends, and **derived classes**.
    * ie: `Novel` is a derived class of `Book`
 
**This works, but it is flawed**
* Can lead to us accidentally violating our invariants
    * Weakens encapsulation
    * Derived classes can violate invariants of base classes

**Solution 2**: Use `protected` **accessor** methods (*getters* and *setters*)

* ie: implement `getAuthor() {return author}`
* This better protects base class invarients
    * Setter can check to see if valid data is being passed into the object's field.
    * Getter can return a copy of the field, rather than the field itself
        * This prevents the derived class from modifying the base class's field

### UML
```cpp

            +-----------------+ // Models "is-a" inheritance
            | Book            | // relationship
            +-----------------+
                      ^
                      |
                      |
            --------------------
            |                  |
            |                  |
            |                  |
    +-----------------+  +-----------------+
    | Novel           |  | Textbook        |
    +-----------------+  +-----------------+
```

### Adding Methods
```cpp
class Book {
        string author, title;
        int pages;
    public:
        Book(string author, string title, int pages);
        
        // private methods
        bool isHeavy() {
            return pages > 400;
        }
}
```

Now methods in `Textbook` may operate differently than methods in `Book`

For example, a 400-page book might be heavy *reading* for a book, but only *average* reading for a `Textbook`.

We can **override** methods in derived classes to change their behavior.

```cpp
class Book {
    protected:
        string author, title;
        int pages;
    public:
        Book(string author, string title, int pages);
        
        // private methods
        bool isHeavy() {
            return pages > 400;
        }
};
```

```cpp
class Textbook: public Book {
        string topic;
    public:
        Textbook(...);
        
        // override isHeavy()
        bool isHeavy() override {
            return pages > 800; // pages here is protected
        }
};
```

And for novel,
```cpp
class Novel: public Book {
        string genre;
    public:
        Novel(...);
        
        // override isHeavy()
        bool isHeavy() override {
            return pages > 600; // pages here is protected
        }
};
```

We can override the behaviour of methods defined in a base class (`Book`) by redefining them in a subclass (`Textbook`)

**Note**: `override` is a keyword that tells your compiler to check that definition in derived class **overrides** existing method in base class.

**Counter Example**
```cpp
struct B {
    void f();
};

struct C: public B {
    void f(int); // error: f(int) does not override a function in B
};
// different function signatures
```
* `override` is useful here to prevent tis mistake.
* Compiler will check that the method in the derived class **overrides** the method in the base class.
* Overload is for different signatures, overriding is for the same signature. For different classes we want to *override* methods

## Now we can do this
```cpp
int main() {
    Novel n{"JRR Tolken", "Lord of the Rings", 300, "Adventure"};
    Textbook t{"Stewart", "Calculus", 1000, "Math"};
    Book b{"alot of authors...", "C++", 3000};
}

cout << b.isHeavy(); // true
cout << n.isHeavy(); // false
cout << t.isHeavy(); // true
```

### Caveats
* **Overriding** is not the same as **overloading**
    * Overloading is when you have multiple methods with the same name, but different signatures
    * Overriding is when you have multiple methods with the same name and signature, but different behavior

* Constructing `Book` from `Novel`
    ```cpp
    // Call default (shallow) copy constructor for book
    // Book copy constructor copies fields that it knows about
    // Extra fields in Novel are not copied
    // The book copy constructor just views a Novel as a Book
    //  by just looking at the Book portion of the novel

    // No memleak possible, it's just a copy
    Book b = n;
    b.isHeavy();
    // returns true, calls Book::isHeavy();
    ```

Here, we construct a `Book` by copying the fields a `Book` knows about from the `Novel` object.

### Polymorphism
* **Polymorphism** is the ability to treat objects of different types in the same way.
    * ie: `Book` and `Novel` are both `Book`s
    * ie: `Book` and `Textbook` are both `Book`s
    * ie: `Book`, `Novel`, and `Textbook` are all `Book`s
* This entire note shows how to make classes **polymorphic**

**More Caveats**
* When passing an object by value, ie:
    ```cpp
    // pass by value
    void printHeavy(Book b) {
        cout << b.isHeavy();
    }

    printHeavy(n);
        // n.isHeavy() is false
        // printHeavy will print true
        // printHeavy will call Book::isHeavy()
    ```
* When passing by reference:
    ```cpp
    // pass by reference (or by pointer)
    void printHeavy(Book &b) {
        cout << b.isHeavy();
        // or b->isHeavy(); if passing by pointer (Book *b)
    }

    printHeavy(n); // n is a Novel
        // n.isHeavy() is false
        // printHeavy will print true
        // printHeavy will still call Book::isHeavy()
    ```
    * A pointer to a `Book` can point to a `Novel` or `Textbook`
        * The compiler still just treats as seeing the `Book` **portion** of `Novel` or `Textbook`
        * `this` = `&n`
    * **Why?**
        * Fields are still located in the same *relative* positions in memory
            * So when the function expects a `Book` pointer/reference, it just treats it as a book
            * Calls the `isHeavy` method that was defined in the `Book` class, not the `Novel` class
    * So how do we do this properly?
        * We want our `printHeavy` implementation to print the `isHeavy()` method of the object that was passed in
        * To do this, we have to envode some extra **(runtime)** information that allows us to dispatch to the correct impementation of `isHeavy()` given the **type of the object** that was passed in.
            * If `b` points to a `Texbook`, we should call `Textbook`'s implementation of `isHeavy()` - `Textbook::isHeavy()`
            * If `b` points to a `Novel`, we should call `Novel`'s implementation of `isHeavy()` - `Novel::isHeavy()`
        * This can be done wit **virtual functions**.

### Virtual Functions
* **Virtual functions** are methods that can be overridden in derived classes.
* You **have** to mark virtual functions as `virtual` in the base class, but you done **have** to in the derived class. (though it is recommended).
    * As soon as you mark a method as `virtual` in your base class, all methods in derived classes **automatically** become virtual, without you explicitly marking them as such.
        * but marking makes things more clear
```cpp
class Book {
    protected:
        string author, title;
        int pages;
    public:
        Book(string author, string title, int pages);
        
        // private methods
        virtual bool isHeavy() {
            return pages > 400;
        }
};

class Novel {
    ...
    virtual bool isHeavy() override {
        return pages > 600;
    }
};
```

#### Cost
* This comes at a cost: space and time since the compiler needs to do some logic to dispatch the correct method. (looks up a table)
* This is why we don't have it done by default (speed maximization)
    * Left optional for when you need it

### Subclasses of subclasses
* If we want to, for example make `MathTextbook` from `Textbook`, we would construct it with `Textbook` as the base class.
