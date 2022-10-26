# Tutorial 06 - 26 October 2022

## R-value references
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

### Rvalue References with functions
```cpp
int &&x;
x = add(3, 5);
// temp = return value = 8 and x -> temp

int add(int x, int y) {
    return x + y;
}
```

### Move Constructor/Assignment

#### Copy and Swap idiom
* Use the copy-swap idiom to make things easier for **move constructors/assignment**

* need to `#include <utility>` for `std::swap`

**Usage**
```cpp
void Yakuza::swap(Yakuza& other){
    // steal from TUT 06 slides
}
```

#### Move Constructor
* Used when you want to move all contents of one object into a new one
* Meant to avoid copying data needlessly

**Usage:**
```cpp
Yakuza::Yakuza(Yakuza&& other) : name{other.name},
    title{other.title}, numSubords{other.numSubords} {
        for (int i = 0; i < numSubords; ++i) {
            subordinates[i] = other.subordinates[i];
            other.subordinates[i] = nullptr;
        }
}
```

#### Move Assignment
* Move data from one object to another
* Try to avoid copying data, just move it
* Don't worry about deleting old data because other will automatically go out of scope.

**Usage:**
```cpp
Yakuza& operator=(Yakuza&& other){
    swap(other); // using earlier def of swap
    return *this; // Whenever you overload an operator, 
                  // you must return a reference to the object
                  // because it lets chaining work
}
```

## Big 5 Example (Again)
```cpp
#include <iostream>
#include <utility>

using namespace std;

struct Yakuza{
    Yakuza* subordinates[5]; // static set max to 5, no doubling
    string name;
    string title;
    int numSubords;

    // Big 5

    // 0. Constructor
    Yakuza(string name, string title);

    // 1. Destructor
    ~Yakuza();

    // 2. Move Constructor
    Yakuza(Yakuza &&other);

    // 3. Copy Constructor
    Yakuza(const Yakuza &other);

    // 4. Copy Assignment
    Yakuza& operator=(const Yakuza& other);

    // 5. Move Assignment
    Yakuza& operator=(Yakuza&& other);

    void swap(Yakuza& other); // HIGHLY RECOMMENDED
    bool add(string name, string title);
    bool remove(string name);
    void setTitle(string newTitle);
};

// Implement Big 5
// 0. Constructor
Yakuza::Yakuza(string name, string title): 
    name{name}, title{title}, numSubords{0} {
        for (int i = 0; i < 5; ++i) {
            subordinates[i] = nullptr;
        }
}

// 1. Destructor
Yakuza::~Yakuza() {
    // tip: Destructor should "emulate" constructor
    for (int i = 0; i < numSubords; ++i) {
        delete subordinates[i]; // resursive call to destructor
        // can't just call delete[] since the array is stack allocated
        // The YAKUZAs are heap allocated
    }
}

// 2. Move Constructor
Yakuza(Yakuza &&other) {
    // Copy from slides
}

// 3. Copy Constructor
Yakuza(const Yakuza &other): 
    name{other.name}, title{other.title}, 
    numSubords{other.numSubords} {
        for (int i = 0; i < numSubords; ++i) {
            // Recursive deep copy on Yakuza{*other.subordinates[i]}
            subordinates[i] = new Yakuza{*other.subordinates[i]};
        }
}

// 4. Copy Assignment
Yakuza& operator=(const Yakuza& other) {
    Yakuza temp = other; // copy constructor
    swap(temp); // swap with temp
    return *this;
}

// 5. Move Assignment
// SUPER easy to change copy assignment to move assignment
// STEPS: add & to parameter, remove temp, swap other
Yakuza& operator=(Yakuza&& other) {
    swap(other); // swap with temp
    return *this;
}

// Rest of  - todo

ostream& operator<<( ostream & out, const Yakuza & y ); //Should output name and title of y and all subordinatres

bool Yakuza::add(string name, string title = "chimpira"){
    if(numSubords > 5){
        cout << "You cannot have any more subordinates!" << endl;
        return false;
    }

    subordinates[numSubords] = new Yakuza{name, title};
    numSubords++;

    return true;
}

bool Yakuza::remove(string name){
    for(int i = 0; i < numSubords; i++){
        if(subordinates[i] -> name == name){
            delete subordinates[i];
            return true;
        }
    }
    return false;
}

void Yakuza::swap(Yakuza& other) {
// todo
}


void Yakuza::setTitle(string newTitle){
    title = newTitle;
}

int main(){

    Yakuza majima{"Goro Majima", "Lord of the Night"};
    majima.add("Nishida");
    majima.add("Daisaku Minami");
    majima.add("Ryota Kawamura");
    majima.add("Gary \"Buster\" Holmes", "King of the Ring");

    Yakuza majimaShadowClone{majima};
    cout << "Copy Constructor" << endl;
    cout << majimaShadowClone << endl << endl;
    
    Yakuza moveMajima = std::move(majimaShadowClone); //Function to force move constructor
    cout << "Move Constructor" << endl;
    cout << moveMajima << endl << endl;

    majimaShadowClone = std::move(moveMajima);
    cout << "Move Assignment" << endl;
    cout << majimaShadowClone << endl << endl;

    majima.setTitle("Mad Dog of Shimano");
    majimaShadowClone = majima;
    cout << "Copy Assignment" << endl;
    cout << majimaShadowClone << endl << endl;
}
```


## Invarients
* Our classes have a set of assumptions (**invarients**) that must be true
* We can't assume users will always use the class properly
* We need to force users to use objects properly!

How do we do this?
## Encapsulation
* We can abstract functionality away from the user and let them interact with our objects using only defined functions
* Lets us regain control of our objects to ensure they are used properly

### Keywords
* `class` - same as `struct` but sets fields to `private` by default
    * Always use `class` from now on instead of `struct`
* `const functions` - defines a function to be const, within which no variables can be altered
    * // copy example from slides
* `mutable` - copy from slides
* `static` - Copy from slides
