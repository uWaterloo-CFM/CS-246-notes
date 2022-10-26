# Tutorial 05 - October 19, 2022

## Classes
* A Class is a structure that can contain functions (member functions)
* Member functions can have an implicit `this` parameter keyword
* `this` is a pointer to the class the member function is running for.

**Usage**
```cpp
struct Monster {
    string name;
    int health;
    void takeDamage(int dagage);
};
void Monster::takeDamage(int damage) {
    this->health -= damage; // this-> is optional
}
```

### `TODO: Copy from TUT5 slides`

### The Big 5
0. Constructor 
    * Creating a new object
1. Destructor 
    * Ran when an object is deleted
2. Copy Constructor 
    * Using an object as the parameter to a new object
3. Copy Assignment Operator
    * Moving data from one object to another when constructing
4. Move Constructor
    * ...
5. Move Assignment Operator
    * ...


### Constructors
* Benefit of using clases over simple structs
* Can define a specific function to rn when an object is created
...

... copy from slides

```cpp
Monster::Monster(string name, int health = 100) { // def
    this->name = name;
    this->health = health;
}
Monster::Monster() { // default constructor
    this->name = "Monster";
    this->health = 100;
}
```

**Valid ways to initialize `Monster`**
```cpp 
Monster m1 {"Monster", 100}; // Uses [def]
Monster m2 {"Monster"}; // uses [def] and default health = 100
Monster m3;  // [uses default constructor]
```

### Member Initialization List (MIL)
* **Ideally** best way to initialize class parameters
* Allows const and reference parameters and objects with no default constructor to be defined during construction

**Usage (MIL)**
```cpp
Monster::Monster(string name, int health = 100) :
    name{name}, health{health} {
        // You can also run any code you want here
        // ie:
        cout << this->name << endl;
    }
Monster::Monster() : name{"Monster"}, health{100} {}
```

### Copy Constructor
* copy from slides

* A copy Constructor is created by defauls
    * Default copy constructor just **shallow copies** all parameters
* You need to define a Copy Constructor if you want a **deep copy**
    * Usually for memory/pointer-related stuff
        * ie: Linked Lists, Trees, etc.
    * Recursively copies

### Destructors
* copy from slides

* Built in destructor calls destructors for all fields that are also objects
* We need to define our own destructor for cases where we need to do more than just delete the object
    * Usually for memory/pointer-related stuff
        * ie: Linked Lists, Trees, etc.
    * Recursively deletes

### Copy Assignment Operator
* copy from slides
* **IMPORTANT**: Check if `this == &other` to avoid self-assignment
    * ie: `a = a;` would cause a problem
    * Could lead to memory leaks
* **Tertiary if statements**:
    * `variable = (condition) ? expressionTrue : expressionFalse;`
        * DOES NOT set `variable = condition`

### Rvalue References
* Required for Move Constructors/Assignment (Move semantics)
* Similar to lvalue references, but for rvalues
* Designed to avoid needless copies by referencing a temporary value
rather than storing it into a variable
* An rvalue is something that can appear on the right side of an
assignment (e.g 3+5, foo(), etc.)
* Not really practical, effectively implemented for move semantics
(covered next) and perfect forwarding (not covered in the course)

**In a normal assignment**
```cpp
x = 3 + 5;
```

**The compiler does the following:**
```
int x;
int temp = 3 + 5; // Creates a copy of x
x = temp;
```

**With rvalue references, the compiler does the following:**
```cpp
int &&x;
x = 3 + 5; -> temp = 8 // 3 + 5 is an rvalue
x -> temp // temp is not deleted

// so x refers to temp instead of copying it to x
```

### Move Constructor
* Used when you want to move all contents of one object into a new one
* Meant to avoid copying data needlessly
Usage: 
```cpp
Yakuza::Yakuza(Yakuza&& other){
    delete boss;
    name = other.name;
    boss = other.boss;
    other.boss = nullptr;
}
```

### Move Assignment Operator

* Move data from one object to another
* Again, trying to avoid copying data, just moving it
* Don’t have to worry about deleting old data because other will
automatically go out of scope
Usage: 
```cpp
    Yakuza& operator=(Yakuza&& other){
    swap(other); //using earlier def of swap
    return *this;
    }
```