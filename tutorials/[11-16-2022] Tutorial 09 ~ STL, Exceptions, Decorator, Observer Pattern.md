# Tutorial 09 - 16 November 2022
* Review Session is on 16 November 2022

## STL (Standard Template Library)
* C++ provides many useful templated data structures
* No need to code your own containers from scratch
* Automatically handle initialization, memory management, and cleanup
* Comes with iterators too

### Vector
Your new best friend
* Within the `<vector>` library
* Easy dynamic array
* Templated, so can be of any type
* If you need some dynamic list, use vectors

**Usage**
```cpp
#include <vector>
std::vector<int> v {4, 5}; //vector with elements 4 5
std::vector<int> v2 (4, 5); //vector with 4 elements equal to 5, 5 5 5 5
v.emplace_back(6); //4 5 6
v.pop_back(); //Removes last element, vector is now 4 5
cout << v.at(0) << endl //Prints 4
//Use iterators to remove items from inside a vector
auto it = v.erase(v.begin()); //Erases item 0
it = v.erase(v.begin() + 3); //Erases item 3
//Returns an iterator pointing at the first item after the erase
it = v.erase(it); //Erase the next item
it = v.erase(v.end() - 1); //Erase last item 
```

**Looping over Vectors**
```cpp
for(int i = 0; i < v.size(); ++i){
    cout << v[i] << endl;
} //Regular index looping

for(auto it = v.begin(); it != v.end(); ++it){
    cout << *it << endl;
}

for(auto n : v){
    cout << n << endl;
}

for(auto it = v.rbegin(); it != v.rend(); ++it){
    cout << n << endl;
}
``` 

* `emplace_back` is preferred over `push_back` because it constructs the object in place, rather than copying it

## Exceptions
When an error condition arises, the function raises an exception
* By default, execution stops when an error is raised
* We can write handlers to catch the exceptions when they are raised
and deal with them

`vector<T>::at` throws an exception if the index is out of bounds
```cpp
try{
    cout << v.at(3) << endl;
} catch(out_of_range r){
    cerr << "Range error: " << r.what() << endl;
}
```

### Stack Unwinding
When an exception is thrown, it is passed through all called functions
until it reaches a valid handler
* Referred to as stack unwinding

```cpp
void f(){ throw out_of_range{"f"}}; //Fills the .what() in }
void g(){ f();}
void h(){ g();}
void main(){
    try{h();} catch(out_of_range){
        //Only need out_of_range r if you're going to use r
    }
}
```

The exception is propagated back through the calls until it hits a handler
* If there is no matching handler, the program crashes
* A handler might do part of the recovery job ( i.e execute some corrective code) and then throw another exception
* You can daisy-chain exceptions

```cpp
try{ ... }catch(SomeError s){
    ...
    throw SomeOtherError{};
}
try{ ... }catch(SomeError s){
    ...
    throw; // rethrow the error
}
```

### `throw` vs. `throw s`
From the previous example, we assume SomeOtherError inherits from `SomeError`
* The handler takes a parameter of type `SomeError`
* If we call `throw s;`, the program will reconstruct the exception based on the value of `s`, so it would be of type `SomeError`
* This makes the exception lose its subtype when it's thrown again
* `throw;` will simply rethrow the same exception that was given to the
handler before it was parameterized into the supertype object s

## Futher Exception Notes
* A general handler can be written as:
```cpp
try {//whatever
    } catch(...){ // ACTUALLY WRITE ... to catch all exceptions
        // Actual ... catches all exceptions
    }
```
* You can throw/catch anything you want, doesn't have to be an object
* You can create your own exception like you would any other class
```cpp
class BadInput{//fields and methods
}
// ...
throw BadInput{};
```

### Destructors and Exceptions
**NEVER** let a destructor `throw` an exception
* If you do, your program will immediately abort
* Destructors are not allowed to throw exceptions
* They can (but shouldn't) by tagging them with noexcept(false)
* Destructor could throw an exception while stack unwinding is happening
* Now **there are two exceptions**, which C++ doesn't know how to handle

## Decorator Pattern
Goal is to add functionality or features to an object at runtime
* Class Component - defines the interface
* ConcreteComponent - implements the interface
* Decorators - all inherit from abstract Decorator, which inherits from Component
* Every Decorator is a Component and every Decorator has a pointer to the a component
* Whether it is "a has" a or "owns a" relationship depends on whether you consider the decorated object to be inseperable from its undecorated component
* Effectively a linked list

### UML for Decorator Pattern
![Decorator UML](https://i.gyazo.com/b46832addde51c1f244845d16f0abbc7.png)

**Pizza Example**
![Pizza Decorator UML](https://i.gyazo.com/ea50bee4e2c5197fb55acf8111fcf3a6.png)

**Whenever you add a decorator, you also form a list of concrete components that kind of loops around the decorator** (Like a linked list)
* Whether it is a has-a or owns-a is up to you (the programmer)

### Decorator Example
```cpp
class Pizza{
    public:
        virtual float price() const = 0;
        virtual string desc() const = 0;
        virtual ~Pizza(){}
};
class CrustAndSauce : public Pizza{
    public:
        float price() const override {return 5.99;}
        string desc() const override {return "Pizza :)";}
};

class Decorator : public Pizza{
    protected:
        Pizza *component;
    private:
        Decorator(Pizza *p) : component{p}{}
        ~Decorator(){ delete component;}
    };

class StuffedCrust : public Decorator{
    private:
        StuffedCrust(Pizza *p) : Decorator{p}{}
        float price() const override{ return component -> price() + 2.69;}
        float desc() const override{ return component -> desc() + " with stuffed crust";}
};

class Topping : public Decorator{
    string top;
    private:
        StuffedCrust(Pizza *p, string t) : Decorator{p}, top{t}{}
        float price() const override{ return component -> price() + 0.75;}
        float desc() const override{ return top + " " + component -> desc();}
};

/////////////////////////////////////////////////////////////////////
Client:
Pizza *p1 = new CrustAndSauce();
p1 = new Topping{"cheese", p1};
p1 = new Topping{"mushrooms", p1};
p1 = new StuffedCrust{p1};
cout << p1 -> desc() << " " << p1 -> price() << endl;
```

## Observer Pattern
* Publish - subscribe model
* One class: Publisher/subject -> generates data
* One or more classes: subscriber/observer -> read and react to data
* Can be many different kinds of observer objects

Sequence of method calls:
    1. Subject's state is updated
    2. Subject::notifyObservers calls each observer's notify() function
    3. Each observer calls ConcreteSubject::getState to query the state and update accordingly

```cpp
class Subject{
    vector<Observer*> observers;
    public:
        void attach(Observer *ob){ observers.emplace_back(ob);}
        void detach(Observer *ob){ //Remove from list
        }
        void notifyObservers(){
            for(auto ob : observers){
                ob.notify();
            }
        }
        virtual ~Subject() = 0;
};
    Subject::~Subject{} //in .cc file
```

**Usage**
```cpp
class Observer{
    public:
        virtual void notify() = 0;
        virtual ~ Observer(){}
};
```

### UML for Observer Pattern

![Observer UML](https://i.gyazo.com/a5617c64bc2c8dd2d390f0ccbd6cace9.png)
