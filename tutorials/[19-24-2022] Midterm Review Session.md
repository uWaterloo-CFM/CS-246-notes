# October 24, 2022 - Midterm Review Session

## hello sir good luck on ur midterm :))))))

## Midterm Coverage: Up to Static member functions and mutable keyword
* Thursday Lecture for Ed's class

## Topics
1. True/False section
2. Bash
3. Regex+Globbing
4. C++ Short Answer
5. C++ Long Answer
6. Big 5 Full Question (Question on MT will be easier)

***

### 1. True/False

1. After this statement is executed, space for *p, the value p is pointing to, is allocated on the heap:
```cpp
int *p = new int
```

> True

2. This code causes 3 to be printed
```cpp
int main() {
    int i = 3;
    int *p = &i;
    *p = 5;
    cout << i;
}
```
> False, outputs 5

3. This code causes 2 to be printed:

```cpp
void f(int &a, int *b, int c) {
    a = 1; *b = 2; c = 3;
}

int main() {
    int i = 0;
    f(i, &i, i);
    cout << i;
}
```
> True: does it in order 
* Pass by reference still sets i to 1 when a = 1

4. The following code prints out `4 5`

```cpp
int a = 1;
int b = 2;
int &f(int &c) {
    c = 4;
    return b;
}

int main(){
    f(a) = 5;
    cout << a << " " << b;
}

```

> True, `f(a) = 5` becomes `b = 5` since `f` returns a reference. `return b` returns a reference to `b`, it's like using a reference parameter.

#### The `&` operator
* `&p` can get you the address of p
* `int &p` can get you the reference to p
* Passing by reference lets you interact with the exact variable instead of a copy of it

5. Can we create a reference to a reference
```cpp
int x = 5;
int &y = x;
int &&z = y;
```
> NO!

6. Can we create a pointer to a reference?
```cpp
int x = 5;
int &y = x;
int *z = y;
```
> NO! Can create weird daisy-chain pointers
* `y` is pretty much `x`. So when you do `int *z = y` it's like saying `int *z = y` but `y` is an integer, not a pointer to an integer!

=> References are the "Deepest" Down in memory you can go. Don't try to refer/make pointers to them

7. Can you create an array of references?
```cpp
int x = 5;
int x2 = 12;
int *&y = new int&[2]{x, x2}
```
> No

8. Can you create a reference to a pointer?
```cpp
int *x = new int[2]{1,2};
int *&y = x;
```
> Yes
* Remember, with references, you can just replace them in place with whatever they are referring to. You are just making a reference to a memory address.

### 2. Bash - 
## IMPORTANT: Write SHEBANG LINE
1. Write a bash script that prints out the lines in files in the current directory that contain the string "disco"

```sh
#!/bin/bash
for file in $(ls); do
    if [ -f "${file}" ]
    cat "${file}" | egrep "disco\." # Escape the .
done
```

```sh
#!/bin/bash
cat * | egrep "disco"
```

2. Write a program called `checkSum` that adds up all the numbers in a file and checks if it is larger than an input sum. The program takes in two parameters, the first being the name of a file containing whitespace separated numbers, and the second sum to beat

```sh
#!/bin/bash
checkSum() {
    sum=0
    for num in $(cat "${1}"); do
        sum=$((${sum}+${num}))
    done

    if [ ${sum} -gt ${2} ]; then
        echo "Larger"
    else
        echo "Smaller"
    fi
}

checkSum(${1} ${2})
```
### 3. Regex+Globbing
1. Write a regular expression to match:
`https://www.freddddddddyfreaker.gov/blog?post=123`

where:
* `https://` and `www.` are both optional
* There can be 2 or more `d`s in `freddy`
* The post id can be at most 3 digits long

`(https://)?(www\.)?fred(d)+yfreaker\.gov/blog\?post=[0-9]([0-9])?([0-9])?`

2. Use `ls` and a globbing pattern to print out all the files with extension `.txt`. Then use `ls` and `egrep` to print out all files with extenstion `.txt`

* `ls *.txt`
* `ls | egrep "^.*\.txt$"`
* Directories are files

3. Use `ls` and a globbing pattern to print out all files with the form `abc.xyz` where `xyz` is a three character extension. THen use `ls` and `egrep` to achieve the same result.

`"^[A-Za-z0-1][A-Za-z0-1][A-Za-z0-1]\.[A-Za-z][A-Za-z][A-Za-z]$"`

* `ls abc.???`
* `ls | egrep "^abc\....$"`


### 4. C++ Short Answer
1. What is wrong with this code?
```cpp
// Single arugment constructor
Node::Node(int data, Node *next = nullptr) : 
    data{data}, next{next}{}

void foop(int i){// does something with the int}
void foo(Node n){// does something with the Node}

int main(){
    foo(123);
}
```
* What is the issue?
* When you put in an `int` into `foo`, you use **implicit conversion** that uses the single argument constructor and passes in a single node into `foo`
* The problem is that you probably meant to use `foop`

* How to fix this? 
* we need to fix the implicit conversioj
* Put `explicit` in front of constructor
```cpp
// Single arugment constructor (explicit)
// Good practice to always use  `explicit`
explicit Node::Node(int data, Node *next = nullptr) : 
    data{data}, next{next}{}

void foop(int i){// does something with the int}
void foo(Node n){// does something with the Node}

int main(){
    foo(123);
}
```

2. What is the problem with defining an outstream operator as a member function?

Why is defining an addition operator as a member function fine?
```cpp
struct Node{
    int data; Node* next;
    ostream& operator<<(std::ostream & out); // why is this bad
    Node operator+(const Node& n); // And this is okay
};
```
* If you define an operator as a member function, it implcitly assumes the first parameter passed into it is of type `Node`

But normally, people do this:
```cpp
Node N;
cout << N; // Can't happen, first thing has to be type Node

// Need to do this
N << cout; // This is really bad style and confusing
```

* Addition operator is okay because it doesn't matter if you define it as a member function. It will be `Node` on the left hand and right hand side anyway.


3. Write an operator for the `Node` class
* Important, you CANT copy streams. You can only handle by reference
```cpp
// overload operator<<
ostream& operator<<(ostream &out, const Node &n) {
    // Pointer to const Node , not a const pointer
    for (const Node* p = &n; p! = nullptr; p = p->next) {
        out << p->data << " ->";
    }
    out << "nullptr"; // artistic end of linked list
    return out;
}
```

Same thing, but in a `while` loop
```
const Node *p = &n;
while (p != nullptr) {
    out << p->data << " ->";
    p = p->next;
}
```

4. Write a constant multiplication operator for the node class that multiples every element within it by the constant. Make sure it works no matter what side the constant is on.

```cpp
Node operator*(const Node &n, int k) {
    Node copyN = n; // invokes the copy constructor
                    // Copy assignment works on INITIALIZED Variables
    const Node *p = &copyN;
    while (p != nullptr) {
        p->data *= k;
        p = p->next;
    }
    return copyN
}

Node operator*(int k, const Node &n) {
    return n * v;
}
```

### 5. C++ Long Answer
1. Write a program to read in tokens from a file named `words.txt` until EOF. Each word should be output on a new line

```cpp
#include <fstream>
#include <string>
using namespace std;
///////////////////
// Anything above is optional on the midterm

int main() {
    ifstream file{"words.txt"};
    string word;

    while (true) {
        word << file;
        if (file.fail()) break;

        cout << word << endl;
    }
} 
```

2. Write a header file called `mario.h` for a class called Mario. Do not write any implementation for any functions. This class keeps track of Mario's position with the integer parameters x and y, his current integer speed, and a string representing his form. Mario can be "small", "big" or "fire".

The class has two constructors, one that takes in parameters for his position and his form, and one that takes just his form. Make sure that there is no ambiguity with the single argument constructor. The class should also have a method called takeDamage(). **Ensure that this file can only be included once.**

```cpp
// Include Guard
#ifndef MARIO_H
#define MARIO_H

#include <string>
struct Mario {
    int x; int y; int speed;
    std::string form; // since you are not using namespace std
    Mario(int x, int y, std::string form);
    explicit Mario(std::string form); // explicit to prevent 
                                      // implicit conversion 
    void takeDamage();
}
#endif
```

### 6. Big 5 Full Question (Question on MT will be easier)

#### Big 5:
0. Constructor
1. Copy Constructors
2. Move Constructors
3. Copy Assignment Operators (=)
4. Move Assignment Operators (=)
5. Destructors

# Yakuza Look at Repo when done

```cpp
struct Yakuza {
    Yakuza* subordinates[5]; // doubles if no capacity
    string name;
    string title;
    int numSubords;

    // 0. Constructor
    Yakuza(string name, string title);

    // 1. Destructor
    ~Yakuza();

    // 2. Copy Constructor
    Yakuza(const Yakuza &other);

    // 3. Copy Assignment
    Yakuza(Yakuza &&other); // && is rvalue reference

    // 4. Move Assignment
    Yakuza &operator=(const Yakuza &other);

    // 5. Move Constructor
    Yakuza &operator=(Yakuza &&operator) // && is rvalue reference

    // Easy swap function
    void swap(Yakuza &other);

    // Takes in a name, adds name to subordinates
    bool add(string name); 

    // Takes in name, removes name from subordinates
    bool remove(string name);
}

// Define our Big 5 outside of struct

// Destructor
Yakuza::~Yakuza() {
    // iterate over array of Yakuza pointers and delete them

    for (int i = 0; i < numSubords ++i) {
        delete subordinates[i]; // resursively calls destructor
    }
    // Don't need to delete the subordinates list 
    //   since it is allocated on the stack.

}

// Copy Construction
Yakuza::Yakuza(const Yakuza &other):
    name{other.name}, title{other.title}, 
    numSubords{other.numSubords} {
    for (int i = 0; i < numSubords; ++i) {
        subordinates[i] = new Yakuza{other.subordinates[i]};
    }
}

// Move Constructor (Steal!)
// && signifies rvalue reference
Yakuza::Yakuza(Yakuza &&other) {
    // No need to swap these since these are
    //  elementary fields

    // Can also just use MIL here, probably better
    name = other.name;
    title = other.title;
    numSubords = other.numSubords;
    
    for (int i = 0; i < numSubords; ++i) {
        subordinates[i] = other.subordinates[i];

        // Stealing, not sharing! 
        // set the other fields to nullptr

        other.subordinates[i] = nullptr;
    }
}

// Copy Assignment
Yakuza& Yakuza::operator=(const Yakuza &other) {
    Yakuza tmp = other; // invokes copy constructor
    swap(temp);
    return *this;
}

// Move Assignment 
Yakuza& Yakuza::operator=(const Yakuza &other) {
   swap(other);
   return *this;
}

// Swap Function
void Yakuza::swap(Yakuza& other){
    using std::swap;
    swap(name, other.name);
    swap(title, other.title);
    swap(subordinates, other.subordinates);
    swap(numSubords, other.numsubords);
}

// Constructor
        name{name}, title{title} {
            for (int i = 1; i < 5; ++i) {
                subordinates[i] = nullptr;
            }
        }

```

## Makefile
* Short Ans
* Multiple Choice
* ie: What is PHONY

* Unlikely they ask us to write a Makefile from scratch

#### Makefile
![Makefile Rant](https://discord.com/channels/817888857239322624/969983580463845386/1033498651370197033)

* Convention: Name is `Makefile`

### Rules
General Structure
```
* Target: prereqs
    recipe
```
2. 
```
vec: main.o vec.o
    g++ ... main.o vec.o -o vec
```
3. 
```
main.o: main.cc
    g++ ... main.cc -o main.o
```

Basically I have to go down this chain and tell `make` how to get everything

### Variables
* We can make things easier with variables
```
EXEC=vec
OBJECTS=main.o vec.o
${EXEC}: ${OBJECTS} // EXEC depends on OBJECTS
    g++ ... ${OBJECTS} -o ${EXEC}
```

### Implicit Rules
```
stem.o
```
* Make will find whatever has the stem `stem`, and finds a file called `stem.cc`. It then compiles it using the default CXX (compiler) and CXXFLAGS

```
stem.o
    ${CXX} ${CXXFLAGS} stem.cc -o stem.o
```

You can declare `CXX` and `CXXFLAGS` yourself

### Updating
```
CXX=g++
CXXFLAGS=-std=c++14 -Wall -g -MMD // -MMD generates a bunch of .d files
EXEC=vec
OBJECTS=main.o vec.o
DEPENDS=${OBJECTS:.o=.d}
${EXEC}: ${OBJECTS} // EXEC depends on OBJECTS
    ${CXX} ${CXXFLAGS} ${OBJECTS} -o ${EXEC}

-include ${DEPENDS}
```

```
main.cc:
    #include "vec.h"

main.d

main.o: main.cc vec.h
// No need to specify recipe, already finds it all
```

## Clean Command
```
PHONY: clean
// this is literally just a rule with no tokens
// Give it a command to run
clean:
    rm ${EXEC} ${OBJECTS} ${DEPENDS}
```

* We need PHONY because then Make will try to make a program called `clean`.
* PHONY skips all of make's name checking, other time-wasting checks


### Common Idiom

```
.PHONY: all
all:
    make a3q3
```

I can then just type 
`make clean` to clean, or
`make all` to make al 

## Working Makefile

```
// TODO: Add working Makefile from notes here
```

