# Lecture 11 - October 20, 2022

## Midterm Cutoff is Today

### Last Week
0. Constructors
1. Copy Constructors
2. Move Constructors
3. Copy Assignment Operators (=)
4. Move Assignment Operators (=)
5. Destructors

* Copy Elision
* Implicit Conversions and `explicit`

### This Week
* Member Operators
* Arrays of Objects
* Constant Objects
* Static Members
* Invarients and Encpsulation

***

### Rule of 5
* If you have to implement one of them, you will likely have to implement **all** of them
    * ie: If you have to implement a destructor, you likely need a constructor to copy the fields of your object

### Implicit Conversions
* Example when it would be useful: for `ifstream`s!

```cpp
// Defenition of ifstream
ifstream(std::string fname);

// This string literal is a `const char *`
ifstream f{"midterm_solns.txt"};
```
* Implicit conversion converts your `const char *` to a `std::string` and then calls the constructor
* **NB**: If you don't want this behaviour, use the `explicit` keyword

### Member Operators
```cpp
struct Vec{
    int x, y;
    Vec(int x, int y): x(x), y(y) {}
}

// Overload the + operator
Vec operator+(const Vec &lhs, const Vec &rhs){
    return Vec(lhs.x + rhs.x, lhs.y + rhs.y);
}
```

```cpp
struct Node {
    // ...
    Node& operator=(const Node &rhs){
        // ...
    }

// makeNode() is our implementation in this example
// constructs a node, returning it by value
x = makeNode();
// calls x.operator=(makeNode())
```
}

* Other operators can also be defined as members of a class
* In this case, the operator acts on:
    * `this` (replacing the first argument), and the other arguments is passed in as a parameter

### Be careful overloading operators (Return Type + order of Parameters)
```cpp
Vec operator *(int k, Vec v); // Can't be implemented as a member
                              // Returns a Vec instead of LHS type (int)

Vec operator *(Vec v, int k); // can be implemented LHS == return type
```

### Member Operators for I/O
```cpp
cout << 5;
cout << vec;
```
* In particular, for **output/input** `<<` and `>>` can't be member operators of the class you are reading/writing from.
    * That is, `<<` and `>>` cant be a member of the `Vec` class

### Some Operators HAVE to be members
1. `operator =` (Assignment)
2. `operator []` (Array Indexing)
4. `operator ->` (Field Access for Pointers)
3. `operator ()` (Function Call Syntax)
5. `operator T` (`T` is any specific type) (Implicit Type Cast)


### Arrays of Objects
```cpp
struct Vec{
    int x, y;
    Vec(int x, int y): x(x), y(y) {}
}

int main() {
    Vec v[3];
}
```
* Compliation Error: `no matching constructor for initializing v`
* This is because the compiler doesn't know how to initialize the array of objects
    * It doesn't know how to initialize the `x` and `y` fields of the `Vec` objects
    * Complier tries to call the default constructor to initialize each element of the array `v`
    * The default constructor doesn't take any arguments, so it can't be used to initialize the array
    * MIL is `Vec(int x, int y): x(x), y(y) {}`

**Fixed but still scuffed**
```cpp
struct Vec{
    int x, y;
    Vec(int x, int y): x(x), y(y) {}
}

int main() {
    Vec v[3] = {Vec(1, 2), Vec(3, 4), Vec(5, 6)};
}
```

> But what if I want to allocate memory for this object on the **heap**? I will still have the original issue as I did when trying to initialize the array on the stack.

* ie: `Vec *v = new Vec[3];` will still fail.

#### Proper Solutions/Workarounds
* Define a default constructor  

* However, some classes don't admit meaningful defaults.
```cpp
// ie:
ifstream f; // correction Lecture 12: 
//             ifstream does have a default constructor
ofstream o;
```

So, for arrays of these objects, your most likely solution is to make an **array of pointers** to the objects
```cpp
ifstream **files = new ifstream*[3];
files[0] = new ifstream("midterm.txt");
files[1] = new ifstream("final.txt");
files[2] = new ifstream("grades.txt");
// These 3 operations can be done in a loop

// Downsides: 4 pointers will need to be `deleted` later
```

### Constant Objects
```cpp
void f(const Node &n) {
    n.print();
} // This won't work; print() is not a const function

// evil `print()` function
void Node::print() {
    this->val = 5;
}

// But what if our print function was normal?

void Node::print() {
    cout << this->val << endl;
} // Calling n.print() should still work?
```

* In order to check that a method is allowed to be called on **constant objects**, the compiler needs a special keyword.
    * You need to mark the function as a **constant method**

```cpp
struct Node {
    int val;
    void print() const;
}

// And in the implementation
void Node::print() const {
    cout << this->val << endl;
} // This will work
```

#### Escape Hatch
* Fields marked `mutable` can always be mutated.
    * For example, a field for profiling # calls to a method;
        * (Counting the number of times a method is called)

ie:
```cpp
struct Node {
    int val;
    void print() const;
    mutable int printCalls;
}

// And in the implementation
void Node::print() const {
    cout << this->val << endl;
} // This will work
```

### Static Members
* Now in previous example, we conted print calls *per object*
* How would we keep track of all calls to print over **all objects** of the class?

**Easy Answer, but Scuffed**: Make a global variable

**More efficient Answer:** *Static class fields*
* Instead of the number of calls **per individual node** 
* We count the number of calls to **ALL NODES**

```cpp
struct Node {
    int val; // value is stored PER node
    void print() const;
    static int printCalls; // value is shared accross ALL nodes
}

// define
int Node::printCalls = 0;

// to access we just do this
void Node::print() const {
    cout << this->val << endl;
    Node::printCalls++;
}
```

* **NB**: We can also `static` member functions
    * Similar syntax, just add `static` in declaration
    * Useful if you want a global function that is only relevant to the class

* In memory, n just really has `val` and `next` 
* `Node::calls` would be stored elsewhere in memory

### Example Problem
* Suppose our `Node` class is properly constructed (constructor, destructor, ...)
```cpp
int main() {
    Node n1 = Node{1, new Node {2, nullptr}};
    Node n2 = Node{2, nullptr};
    Node n3 = Node{3, &n2};
}

// Consider Node's destructor
~Node() {
    delete next;
}
```