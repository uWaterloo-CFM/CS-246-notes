# Lecture 05 - 22 September 2022

## C++ Cont.

### Chaining I/O Operators

* After using a stream with, `<<` and `>>`, returns a copy of the stream that was passed in. 
* This way, we can chain together streams

ie:

```cpp
#include <iostream>
using namespace std;

int main() {
  int i;
  while (true) {
    cin >> i;
    if (cin.fail()) break;
    cout << i << endl;
  }
}
```

* `cout << i << endl;` `cout << i` returns a `cout` stream, so you can pass `endl` into it.

### I/O Manipulators
* `#include <iomanip>`
* Lets you modify what you output into the format that you want
* Can let you change `cout` to be in whatever base you want (Base 16 for `hex`)

* **NB:** States persist after using I/O Manipulators

```cpp
#include <iostream>
#include <iomanip>
using namespace std;

int main() {
  int i;
  while (true) {
    cin >> i;
    if (cin.fail()) break;
    cout << hex << i << endl; // Output as hex
    cout << dec; // Switch cout mode back to decimal now
    cout << "Hello World " << 16 << endl; // Will now output 16 in base-10 instead of hex
  }
}
```

### Types of I/O Manips
* Review: [CPPref.com I/O Manipulation](https://en.cppreference.com/w/cpp/io/manip)

> Why Not `printf()`?

* We use streams in C++ in the interest of safety

> Example

```c
#include <stdio.h>

int main() {
        printf("Hello World from %s\n", 42);
        return 0;
}
```

Will not work since it is trying to print a string at adress 42. It will try to look for a string at address 42, which will produce an undesireable output

### Strings in C
* A string in C is a null-terminated array of characters
* This causes alot of issues: What if you forget to null-terminate your string?
* Memory corruption if you forget to null-terminate
* You have to manage memory/buffers, `malloc(), realloc()`

### Strings in C++
* Strings in C++ are alot better
* C++ has its own string datatype

#### `#include <string>`
* This package includes the `string` data type

```cpp
string s = "Hello";
```

We can use this to work with strings really easily
```cpp
#include <iostream>
#include <string>
using namespace std;

int main() {
        string s = "Hello, World!";
        cout << s << endl;
        return 0;
}
```

### `String` Library

* The `string` library also gives us some useful commands
* String comparison

```
s1 <= s2

s1 != s2

s1 == s2
```


### Input

* The following program echoes an input in C
```c
#include <stdio.h>

int main() {
        char buffer[10];
        scanf("%s", buffer);
        printf("%s\n", buffer);
        return 0;
}
```
* This program will **break** if the input is larger than **9** characters.
* We can fix this in O(n) (amortized) if we use the doubling method, but this is annoying

* The following program does the same thing in C++
* Also in O(n), but does not use any special memory management.

```cpp
#include <string>
#include <iostream>

using namespace std;

int main() {
        string s;
        cin >> s;
        cout << s << endl;
        return 0;
}
```

### File Streams in C++
* `getline(cin, s)` - reads next line into `s`.

* in-file and out-file streams
```
ifstream in {"file"} called in
    input stream from file
  
ofstream out {"file"} called out
    output stream writing to "file"
```

```cpp
#include <iostream>
#include <fstream>
using namespace std;

int main () {
  ifstream file{"suite.txt"};
  string s;
  while (file >> s) {
    cout << s << endl;
  }
}
```

### String Streams

* Replacement for `scanf`

```cpp
#include <sstreams>

istring stream s{str}

// input stream reads from str
```

### More String Streams

* Replacement for `printf`
* `ostringstream out;` constructs on output string stream
* `out.str()` gets the contents of `out` as a c++ string

```cpp
#include <iostream>
#include <string>
#include <sstream>
using namespace std;

int main () {
  ostringstream ss; // ss stores the concatenated input
  int lo {1}, hi {100};
  ss << "Enter a # between " << lo << " and " << hi;
  string s {ss.str()};
  cout << s << endl;
}
```

### Type conversions
* You can use `stringstreams` to convert types

```cpp
#include <iostream>
#include <sstream>
using namespace std;

int main() {
  string str = "100"; // a variable of string data type

  int num; // a variable of int data type

  // using the stringstream class to insert a string and
  // extract an int
  stringstream ss;
  ss << str;
  ss >> num;

  cout << "The string value is " << str << endl;
  cout << "The integer representation of the string is " << num << endl;
}
```


# Reading in `ints` using StringStreams

```cpp
#include <iostream>
#include <sstream>
using namespace std;

int main () {
  string s;
  while (cin >> s) {
    istringstream ss{s};
    int n;
    if (ss >> n) cout << n << endl;
  }
}
```


## Functions in `C++` - Default Values

* Let's write a function that prints out all words from a file
* If no file is given, print out the words in `suite.txt`

```cpp
#include <iostream>
#include <string>
#include <fstream>

using namespace std;

void printSuiteFile(string name = "suite.txt") {
        ifstream file(name);
        string s;
        while (file >> s) cout << s << endl;
}

int main() {
        printSuiteFile();
        printSuiteFile("readInts.cc");
}
```

* `parameter = value` for default vaules passed to parameter when not specified
* Parameters with default values have to be the last parameters of that function.

## Overloading
* Functions in c++ can have functions with different parameter lists share the same name
* But the parameter lists have to be **distinct**. The lists can have different types and that makes them distinct.
* The compiler will use context clues (var type) to assign the function accordingly.

