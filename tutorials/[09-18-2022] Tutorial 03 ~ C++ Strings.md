# Tutorial 04 - September 28, 2022

## Strings

### Indexed String Connectors
```cpp
// TODO: Copy for Tut slides
```

### String Operators
* Concatenation using `+`, `+=`
* Others: `length`, `clear`, `substr`, `find`, `rfind`, `replace`

* See more: [https://en.cppreference.com/w/cpp/string/basic_string](https://en.cppreference.com/w/cpp/string/basic_string)


### String comparison
* Lexicographical comparison using:
    * `==`, `!=`, `<`, `>`, `<=`, `>=`

* `str1.length() == str2.length()` compares string lengths

## Streams

### Input Streams
* `<stream> >> <string>`
* `<stream> >> <int>`
* `eof()`
* `fail()`
* `clear()`
* `ignore()` - Skips next character in the string

### Output Streams
* `<stream> << <var>` - Puts the information stored in `var` into `stream`

### I/O Streams
* `#include <iostream>`

### File Streams
* `#include <fstream>`

#### Types of file streams
* `std::ifstream` - Input file stream
* `std::ofstream` - Output file stream

#### Opening a file
* `std::ifstream file("file.txt");`

### String Streams

* `#include <sstream>`
* `std::ostringstream`
* `std::istringstream`
* `str()`

* `const char *p = s.str().c_str();` - `c_str()` turns a c++ string into a c string

## Overloading
* Functions can be overloaded if the parameter list is different
* **NB:** Functions cannot be overloaded based on return type alone!

## Default Parameters
* `void foo(int n = 75);`

### We can call foo:
* `foo();` - Calls foo with the default value of 75
* `foo(5);` - Calls foo with the value of 5
* **NB**: In a function declaration, default variables must come last

#### Example:
```cpp
void foo(int n = 75, char c = 'a'); // OK

void foo(int n = 75, char c); // not OK
```

## Example: Complex number multiplcation
```cpp
// TODO: Copy from TUTORIAL Slides
```


