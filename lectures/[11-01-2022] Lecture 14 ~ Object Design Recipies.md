# Lecture 14 - 01 November 2022

**CS 246** So far:
* Linux/Command Line Terminal Tools
    * `grep`
    * `make`
    * `bash`
* Basic C++
    * Seperate compilation
    * Standard Library
        * Strings, streams
    * QoL and safety improvments
* Objects
    * Big 5
        * Copy/Move semantics
        * Object Destruction
    * Methods
    * Constructors
    * Encapsulation
        * Design Patterns
            * Iterators (Assignment 3)

* (**Now**) The Design Recipie for Objects
    * How to describe objects

***
## Design Recipies
Right now, if I want to describe a `Vec` object, I would say something like:
```cpp
class Vec {
    private:
        int x, y;
    public:
        Vec(int x, int y): x(x), y(y) {}
        int getX() const { return x; }
        int getY() const { return y; }
};
```

### Problem:
* This is not a good way to describe objects
* It's not very human-readable (too verbose)
* Hard to communicate (ie: for group assignment)

### Solution: UML.
* Uniform Model Language (UML)
    * It's a box and arrow diagram for describing a class
```
// Uniorm Modelling Language (UML)

+-------------------+
|        Vec        |   // name of class
|-------------------|
|  - x : int        |   // fields (optional)
|  - y : int        |   // visibility: +public, -private
|-------------------| 
|  + getX : int     |   // Methods
|  + getY : int     |   // visibility: +public, -private
+-------------------+
```

**UML** also allows us to model the relationship *between* classes

### Example of Class Relationships

**Relationship 1**
```cpp
class Vec {
    float x, y;
    public:
        Vec(float x, float y): x(x), y(y) {}
        // ... some other methods
};

class Basis {
    Vec v1, v2;
    public:
        // [0, 1]^T and [1, 0]^T span R^2!
        Basis(): v1(1, 0), v2(0, 1) {}
}
```
* Note that a basis is **composed** of its two vectors.
    * In general, embedding subobjects as fields of another object is called **object composition**.

### Relation
* `Basis` "owns-a" `Vec` object.
    * Typically, if **A** "owns-a" **B**, then
        * **B** has no identity/independent existence outside of **A**
        * When **A** is destroyed, **B** is destroyed as well
        * When **A** is copied, so is **B** (deep copy)

### `Car` Example
* A `Car` owns its `Wheels`
    * When you destroy the `Car`, you destroy the `Wheels`
        * (like in real life)
    * When you copy a `Car`, you would also copy its `Wheels`
    * A `Car` is **composed** of its `Wheels` (and other parts)

### Relationships in UML
```
// UML Modelling relationships between classes
                      
+-------------------+ v1,v2 +-------------------+
|        Basis      |*----->|        Vec        |
|-------------------|     2 |-------------------|
|  - v1 : Vec       |       |  - x : float      |
|  - v2 : Vec       |       |  - y : float      |
|-------------------|       |-------------------|
|  + Basis()        |       |  + getX() : float |
+-------------------+       |  + getY() : float |
                            +-------------------+
```
* `*` at base of arrow `-->` means it is the owner object
    * `*` is a shaded diamond `<>`
* `>` head signifies owned object
* `v1, v2` signifies the fields of `Basis` which are of type `Vec`
* `2` signifies that there are two `Vec` objects in `Basis`

### Another Relation
* Car parts in a store/catalogue
    * Here, the parts "have-an" independent existence from the store
    * When the store is destroyed, the parts still exist
* Geese in a Pond
    * Here, the geese "have-a" pond
    * When the pond is destroyed, the geese still exist
    * When you make a duplicate pond, you don't duplicate the geese
    * The geese have an independent existence over the pond

These are examples of a **"has-a"** / **aggregation relationship**

* If **A** "has-a" **B**
    * **B** has an independent existence outside of **A**
    * When **A** is destroyed, **B** is not destroyed
    * When **A** is copied, **B** is shallow-copied (not deep copy)
    * **A** and its **copy** share the same **B**

### Typical relationships
* Pointer or reference fields
    * Maybe my `Pond` class holds onto an array of `Goose` objects
    ```cpp
    class Pond {
        Goose* g[10];
    }
    ```

### "Has-a" Relationship in UML
```
// UML Modelling relationships between classes
+-------------------+     g +-------------------+
|        Pond       |<>---->|        Goose      |
|-------------------|  O..* |-------------------|
|        [...]      |       |        [...]      |
```
* `<>` indicates base aggregation
    * `<>` is a hollow diamond for a "has-a" relationship
* `>` arrow tip indicates object being aggregated
* Optional multiplicity (number of objects) can be specified
    * A range of # of geese can be specified
        * `1..*` means 1 or more
        * `0..*` means 0 or more
        * `0..1` means 0 or 1
        * `1..1` means 1
* Optional fields can be specified on top of `>`
    * `g` is the field name of the `Goose` object in `Pond`

### Lets make Books
```cpp
class Book {
    string title, author;
    int pages;
    public:
        Book(string title, string author, int pages): title(title), author(author), pages(pages) {}
        // ... other methods
};

class Novel {
    string title, author;
    int pages;
    Plot plot;
    Genre g;
    public:
        Name(string title, string author, int pages, Plot plot, Genre g): title(title), author(author), pages(pages), plot(plot), g(g) {}
        // ... other methods
};

class ComicBook {
    string title, author;
    int pages;
    string superhero;
    string supervillain;
    public:
        ComicBook(string title, string author, int pages, string superhero, string supervillain): title(title), author(author), pages(pages), superhero(superhero), supervillain(supervillain) {}
        // ... other methods
};

class TextBook {
    string title, author;
    int pages;
    string topic;
    public:
        TextBook(string title, string author, int pages, string topic): title(title), author(author), pages(pages), topic(topic) {}
        // ... other methods
}
```
* There are many different types of books :)
* We could model books by a class for each type
    * But books alot of the same types of fields
        * `title`, `author`, `pages`
    * Only some fields are different

Can we write a function that will return the `author` of anything that looks like a book?

**Method 1:** Overload the `getAuthor` for each book type
```cpp
string getAuthor(const Book &b) {
    return b.author;
}

string getAuthor(const Textbook &b) {
    return b.author;
}
// ...
```
If we keep going with this, what if we want to make containers of books?

We should be able to store arrays of different types of books generically
* But we can't
```cpp
Book *shelf[10]; // can only hold pointers to Books
                 // Can't hold Novels, text books, etc.
TextBook *shelf[10] // can only hold pointers to TextBooks
                    // Can't hold Books, Novels, etc.
```

How do we abstract over **all** kinds of books?
**Method 2:** Use a container for `Book` objects
* Tagged/discriminated union
```cpp
struct GenericBook {
    int type; // type tag (tagged)
    union {
        Book *b;
        Novel *n;
        TextBook *t;
        ComicBook *c;
    };
};
```
* But this is fragile, and we have to remember to update the definition when we add a new book type
* This is unsafe, and fragile

### Solution?
```cpp
void** BookArray; // frick it.
```
* This is very dangerous
    * We can't even check if we are accessing the right type of book
    * We can't even check if we are accessing the right field of the book
    * *We don't need our safety equipment provided by the type system*

### Better Solution?
* Observation
    * Novels, Compic Books, TextBooks are all **extensions** of some underlying Book type.
    * Conclusion: **Thursday** lecture
