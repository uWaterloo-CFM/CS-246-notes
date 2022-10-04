# Lecture 08 - 04 October 2022

## Object Oriented Programming

So far, we only know how to define structures.

```cpp
// student.h
struct Student {
    string name;
    int assns, mt, final_exam_grade;
}
```

**Object Oriented**
```cpp
// student.h
struct Student {
    string name;
    int assns, mt, final_exam_grade;
    float grade();
}
```

### Classes
* A class is a structure type coupled with member functions/methods representing operations one can perform on instances of that class.
    * In the example above, `grade()` is a method that will return the grade of that instance of that student.

### Objects
* An **object** is an instance of a class.

```cpp
#include <student.h>

...

student bob {"Bob", 40, 40, 42};
bob.grade(); // invoke grade on Bob
```

### How to we define methods in a class?

Here is the source file for `student.h`
```cpp
float Student::grade() {
    return assns * 0.4 + mt * 0.2 + final_exam_grade * 0.4;
}
```

#### `::`
* The `::` operator is called the **scope resolution operator**
    * eg. `Student::grade` - defines grade inside the Student `struct`
    * `std::cout` - runs `cout` inside of std

```cpp
#include <student.h>

...

student bob {"Bob", 40, 40, 42};
bob.grade(); // invoke grade on Bob > 41%

student josh {"Josh", 110, 100, 120};
josh.grade(); // invoke grade on Josh > 105%
```

* How does `.grade()` **get** the structure for the specific Student that it is invoked on.
    * Every method in a class has an implicit parameter called `this` that is a pointer to the object that the method is being invoked on.

So, for:
```cpp
bob.grade(); // .this = &bob
josh.grade(); // .this = &josh
```

So in reality, C++ is doing this:
```cpp
float Student::grade(Student *this) {
    return this->assns * 0.4 + this->mt * 0.2 + this->final_exam_grade * 0.4;
}
```

But C++ is smart enough to know that `this` is a pointer to the object that the method is being invoked on, so we can just use the variables directly.

### Constructors
* Custome code which runs when you build an object.

**Declare**:
```cpp
// student.h
struct Student {
    string name;
    int assns, mt, final_exam_grade;
    float grade();

    Student(string name, int assns, int mt, int final_exam_grade);
    // A method with no return type
    // It has the same name as the enclosing structure
}
```

```cpp
// student.cc
#include <student.h>

// Overloading Student
Student::Student(string name, int assns, int mt, int final_exam_grade) {
    this->name = name;
    this->assns = assns;
    this->mt = mt;
    this->final_exam_grade = final_exam_grade;
}

// ...

Student bob {"Bob", 40, 40, 42}; // calls above constructor
```

#### Constructor to clone a `Student` object
* Constructors can be overloaded. In this case, we have a constructor that takes in a `Student` object by reference and clones it.
```cpp
Student::Student(const Student &other) {
    // no need to use this-> because there are no conflicting names
    name = other.name;
    assns = other.assns;
    mt = other.mt;
    final_exam_grade = other.final_exam_grade;
}
```
We can now clone students
```cpp
Student bob {"Bob", 40, 40, 42}; // Calls 4-argument constructor
Student bob2 {bob}; // calls cloning constructor
                    // C++ knows to pass in bob by reference.
```