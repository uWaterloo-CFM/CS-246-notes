# Lecture 18, pt. 2 - 15 November 2022

## Design Patterns
* Design patterns are useful because they let us easily solve problems that we commonly encounter in software development.

We encountered the **Iterator** design pattern previously. **Why?**
* We had some list `l`, what if we wanted to print every element in `l`?
```cpp
List l; // linked list
for (auto i:l) {
    cout << i
};

TierList t;
for (auto i:t) {
    cout << i
};

Vector<int> v{1, 2, 3};
for (auto i:v) {
    cout << i
};

// It's all the same way!
```
* The reason why we come up with these design patterns is so we can abstract over the details without caring too much about the underlying implemtnation.

The Iterator design patten allows us to abstract away iteration over the underlying container implementation. (ie: we can iterate over `List`, `TierList`, `Vector`, etc. without caring about the underlying implementation)

How this actually works though, is because the compiler knows that `List`, `TierList`, and `Vector` all have a `begin()` and `end()` method, so it can use the same code for all of them. It expands to include what we have and compiles that instead.

We still can't write a function which generalizes over iterator.

**Example**
> Write a `foreach` function which takes an arbitrary iterator.

The compiler needed:
* container: `begin()` `end()`
* Iterator: `operator++` `operator*` `operator!=`

If we want to implement the design pattern, we need to provide the above five implementations ALWAYS. If we wanted to implement a generic iterator, we could define a pure virtual base class iterator with those five methods `AbstractIterator` which specifies what ALL iterators have to implement.

```cpp
void foreach(void (*f)(int), AbstractIterator &start, AbstractIterator &end) {
    for (AbstractIterator &i = start; i != end; ++i) {
        f(*i); // we need to call opertaor* to get the value
    }
}

class AbstractIterator {
    public:
        virtual AbstractIterator& opertaor++() = 0;
        virtual int* operator*() = 0;
        virtual bool operator!=(AbstractIterator &other) = 0;
}
```

* This might not be an *ideal* solution, since we need to compare `AbstractIterator`, which will have to have runtime type checks to prevent **mixed comparisons**.

* You need alot of references to the iterators, which might not be what you want since it implies that you can modify the iterator.

* Something about how start() and begin() return by value, not reference.

### The correct way to implement this with iterators is wit TEMPLATES

This method above still works however, with something called **Decorators**

## Decorators
Let's think about Pizzas.

Types of Pizzas
* Cheese
* Pepperoni
* Hawaiian
* BBQ Chicken

Let's say you are making a Pizza-ordering app. You need to **model** ALL of the different types of Pizzas that could possibly exist. 

We also need to dynamically add toppings to the pizza or change pizzas entirely. (ie: you can add toppings to the pizza at runtime)

**How do we model this?**
* Have one unique class for each possible pizza order 
    * This is not scalable, since we have to create a new class for each possible pizza order. (ie: if we have 10 toppings we have to create 2^10 = 1024 classes)

We can't use discrete classes for each pizza type since there's an exponential number of possible pizza types.  (makes App size too big) and we can't dynamically add toppings to the pizza or change pizzas.

**What do we do?**
* Every `Pizza` has a `crust`, and some `sauce`
    * So there is a base form of pizza.
* We can add toppings to the `Pizza`
    * We add functionality (flavour) to our `Pizza`
    * The toppings *decorate* the underlying base `Pizza` object

### Decorator Pattern
* The decorator pattern is a design pattern that allows us to add functionality to an object at runtime.

**UML**
```
                        +-------------+
                        |   Pizza     |
                        +-------------+   decorated
                        | +desc       |<---------------
                        | +cost       |   (<-- arrow) |
                        +-------------+               |
                            ^     ^ (hollow ^ arrow)  |
                            /     \                   |
                           /       \                  |
                          /         \                 |
                         /           \                |
                        /             \               |
                       /               \              |
                +-----------+      +-----------+      |
                |  Crust    |      |  Pizza    | <>---- (hollow diamond <>)
                | and sauce |      | Decorators|
                +-----------+      +-----------+
                                          ^ (Hollow ^ arrow)
                                          |
                                          |
                                __________|___________________
                                |             |              |
                        +-----------+    +----------+  +----------+
                        | Pineapple |    |    Ham   |  | IceCream |
                        +-----------+    +----------+  +----------+
```

### Decorator Pattern in C++
```cpp
class Pizza {
    virtual float price() = 0;
    virtual string desc() = 0;
};

class CrustAndSauce: public Pizza {
    string desc() {
        return "Pizza with crust and sauce";
    }
    float price() {
        return 10.0;
    }
};

class PizzaDecorator: public Pizza {
    protected:
        Pizza *d;
        PizzaDecorator(Pizza *d): d{d} {}
};

class Pineapple: public PizzaDecorator {
    public:
        Pineapple(Pizza *d): PizzaDecorator{d} {}
        string desc() {
            return d->desc() + " with Pineapple";
        }
        float price() {
            return d->price() + 1.5;
        }
}; // We can add more than 1 pineapple. We can add as many as we want
```

### Ownership
* We need to be careful about ownership when using decorators.
* We need to make sure that we don't delete the underlying pizza object when we delete the decorator.
