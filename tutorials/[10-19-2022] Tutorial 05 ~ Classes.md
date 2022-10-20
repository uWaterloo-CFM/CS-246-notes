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
* Copy from slides

### Move Constructor

### Move Assignment Operator