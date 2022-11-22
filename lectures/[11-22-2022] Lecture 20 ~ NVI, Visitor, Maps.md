# Lecture 20 - 22 November 2022

## Last week:
* Observer Pattern
    * subscribing and reacting to updates
* Factory (Method) Pattern
    * Constructing objects dynamically
* Template Method Pattern
    * Ordering *specific behaviour*

## Today:
* Finish NVI (Non-Virtual Idiom) Pattern
* Visitor Pattern
* Maps
* pImpl (Pointer to Implementation) Idiom (jk haha thursday)

***

## Non-Virtual Idiom
When you declare a **public virtual method** you are in effect doing two things:
1. Declaring an interface to an object
    * Derived classes can do some wack stuff (insert their own custom code without verifying it follows the invarients you had for the base class)
2. Declaring a **hook** that derived classes can `override` so that they can insert their own custom behaviour.

These two goals are often **at odds** with each other because you want to be able to change the behaviour of the base class without having to change the derived classes.
* How do we check that a derived class's `override` respects our interface?
    * Well, you really can't, you gave the derived class access to your virtual methods.    

If you want to preserve the invarients of your base class, you probably dont want to give the derived classes access to your virtual methods.

### Fix (NVI)
* We can extend the template method pattern to a logical conclusion:
    * Declare **no** public virtual methods (0 public virtual methods)
    * Declare private virtual methods (called from your public methods) whenever behaviour needs to be overridden.
        * You can have checks in the public methods to make sure the private virtual methods are called with the correct invarients.
    
### Example
```cpp
class DigitalMedia {
    public:
        void play() {doPlay();}
    private:
        virtual void doPlay() {...}
};
```
**By doing this**
1. With proper protection we can ensure `doPlay()` does not violate any invarients.
    * Add some checks in `play()` to make sure `doPlay()` is called with the correct invarients.
    * We can check if `doPlay()` was overriden with the correct invarients.
        * Checks against if `doPlay()` was overriden with stuff we didn't it to.
2. We can extend `play()`/change `play()`'s contract without having to worry about if our derived classes accidentally override `play()`.
    * i.e. to add copyright checks when we don't want it to.
    * **if we didn't do this method of implementation, any derived class from DigitalMedia would have to override `play()` to add copyright checks.**

## Visitor Pattern
* Comes up fairly commonly as a concequence of the design decisions made in C++.

```cpp
Turtle* redTurtle = new RedTurtle();
redTurtle->drawShell(); // dispatches the correct drawShell command for RedTurtles (RedTurtle::drawShell())

```

```
        +---------+    +-----------------+
        | Weapons |    |     Enemies     |
        +---------+    +-----------------+
             |                  |
             |                  |
             |                  |
    ------------         -----------------
    |           |        |               |
    |           |        |               |
+-------+ +---------+   +----------+    +----------+
| Resume| | All-    |   | Waterloo |    | MarmoSet |
|       | | nighter |   | Works    |    |          |
+-------+ +---------+   +----------+    +----------+
```
Weapons and Enemies react differently based on the typ of Enemy and the type of weapon.

**In an ideal C++:**
*`virtual (Enemy, Weapon)::strike`* would get us what we want, but this is not in C++ (yet)

To make this behaviour that depends/dispatches dynamically based on the Ememy and Weapon types, we use the Visitor Pattern.

Virtual metods only allow for a single dispatch.

**e.g.**
`virtual Enemy::strikeBy(Weapon &w)` dispatches based off of Enemy type but not weapon type. (weapon specified by user)

*Conversely*
Similar story for
`virtual Weapon::strike(Enemy &e)` dispatches based off of Weapon type but not Enemy type. (enemy specified by user)

We can't use this tuple-method of calling that we want, **however** we can use a combination of method pattern overloading and overriding to do proper dynamic double dispatch.

```cpp
class Enemy {
    virtual void beStruckWith(Weapon &w) = 0; // then we can provide overrides for waterlooworks, marmoset
};

class WaterlooWorks: public Enemy {
    void beStruckWith(Weapon &w) override {
        w.strike(*this); // call w's stick on dereference of this
    }
};

// much of the same thing
class MarmoSet: public Enemy {
    void beStruckWith(Weapon &w) override {
        w.strike(*this); // Literally the same thing as WaterlooWorks
    }
};
```

Why do we duplicate the code?
* By doing this, we actually gain a bit of information.
* When calling `beStruckWith()` on a specific Enemy (either MarmoSet or WaterlooWorks), our program will dispatch the correct `strike()` command for the specific Enemy.

**Now in Weapon**
```cpp
class Weapon {
    virtual void strike(WaterlooWorks &w) = 0; // strike a WaterlooWorks enemy specifically
    virtual void strike(MarmoSet &m) = 0; // strike a MarmoSet enemy specifically
                                          // DONT MAKE THE MISTAKE OF TAKING IN Enemy &e
};

// Now in our weapons classes, we can provide specific implementaitons for each enemy type 
// For example, an resume to MarmoSet won't do anything, but an all-nighter to Marmoset does something
// An all-nighter to waterlooworks probably won't help, but a resume will be helpful
class Resume: public Weapon {
    void strike(WaterlooWorks &w) override {
        cout << "Defeated WW, earned an Elon internship" << endl;
    }
    void strike (MarmoSet &m) override {
        cout << "COMPILATION ERROR" << endl;
    }
};

int main() {
    Enemy *e = new WaterlooWorks();
    Weapon *w = new Resume();

    // make beStruckWith public in Enemy class I think we need to do
    e->beStruckWith(*w); // dispatches the correct strike command for WaterlooWorks (Resume::strike(WaterlooWorks &w))
}
```

```
                    +-------------------+
                    |       Book        |
                    +-------------------+
                              |
                              |
                              |
                ------------------------------
                |                            |
                |                            |
        +---------------+             +---------------+
        |     Novel     |             |    MathNEWS   |
        +---------------+             +---------------+
```
Here is a `Book` class hierarchy. What if I wanted to add new behaviour to `Book`s?

**For example** an **`excite()`** method?
* If we did so, not only do we need to change `Book` but `Novel` and `MathNEWS` as well.
* This is annoying, especially as class hierarchies get bigger, and may not be possible at all.
    * i.e. if the class hierarchy is given to you as a library.

***
**Aside**
In functional languages like Racket, or Haskell, we can add new behaviour to existing classes by defining a new function.

***

**In C++**: We will reuse the Visitor Pattern
```cpp
class Book {
    virtual void accept(BookVisitor &b) {b.visit(*this);} // this is type Book*
};                                                        // calls BookVisitor::visit(Book &b)

class Novel: public Book {
    void accept(BookVisitor &b) override {b.visit(*this);} // this is type Novel*
};                                                         // calls BookVisitor::visit(Novel &n)

class MathNEWS: public Book {
    void accept(BookVisitor &b) override {b.visit(*this);} // this is type MathNEWS*
};                                                         // calls BookVisitor::visit(MathNEWS &m)

// BookVisitor must include overloads for Book, Novel and MathNEWS
class BookVisitor {
    virtual void visit(Book &b) = 0;
    virtual void visit(Novel &n) = 0;
    virtual void visit(MathNEWS &m) = 0;
};

class BookExciter: public BookVisitor {
    void visit(Book &b) override {
        cout << "Book is excited" << endl;
    }
    void visit(Novel &n) override {
        cout << "Novel is excited" << endl;
    }
    void visit(MathNEWS &m) override {
        cout << "MathNEWS is excited" << endl;
    }
};

int main() {
    BookVisitor &b = BookExciter();
    book->accept(b);
}
```

#### Another Example
```cpp
class Catalogue: public BookVisitor {
    void visit(Book &b) override {
        ...b.getAuthor() ...
        cout << "Book is in catalogue" << endl;
    }
    void visit(Novel &n) override {
        ...n.getGenre()...
        cout << "Novel is in catalogue" << endl;
    }
    void visit(MathNEWS &m) override {
        ... m.getCoverArt() ...
        cout << "MathNEWS is in catalogue" << endl;
    }
};
```
See? We just added another behaviour to our `Book` class hierarchy without changing any of the classes themselves!

cool beans

## Dictionary
* To declare a dictionary, we use the `std::map` class.
```cpp
std::map<k, u>; // k is the key type, u is the value type
                // k has to be orderable (i.e. has <, >, <=, >=, ==, !=)
                // k must have these operators defined^^

std::map<string, int> grades;

grades["Faizaan"] = 90;
grades["Sam"] = 100;

std::cout << "Faizaan has grades " << grades["Faizaan"] << std::endl;

std::cout << "Eric has grades " << grades["Eric"] << std::endl; // Eric is not in the map
                                                                // Maps Eric -> 0
                                                                // return 0

// Remove key from map
grades.erase("Faizaan");

// To check if a key is in the map:
grades.count(key) // = 1 if in map, 0 if not (log(n) time)
grades.find(key) // returns an iterator to the key, or grades.end() if not in map
```
