# Lecture 07 - 29 September 2022

## Building C++ Programs

### Preprocessor
* Transforms a source file before the compiler sees it.
* Nowadays, preprocessors are built into the compiler.

#### Preprocessor Directives
`#________` are preprocessor directives.

**Examples:**
* `#incude "f.h"` is executed by the preprocessor by including the contents of "f.h" durectky at the location where the include was used.

* `#define NAME VALUE` - Directive that your preprocessor can interpret. Preprocessor replaces all instances of NAME with VALUE in the source file. 
    * History: `#define` was used to define constants. Now, `const` is used to define constants.
    * Good use case:
    ```cpp
    #define MAX 30
    int a[MAX]; // Defines the max size of the array to a constant size. 
                // You can't do this with const.
    ```

#### Example: Cryptographic Security Levels
```cpp
#define SECURITYLEVEL 1
#if (SECURITYLEVEL == 1)
    long long key; // compiled when SECURITYLEVEL is defined

#elif (...) // can have multiple elifs

#else
    short key; // Otherwise
#endif
```

#### Disabling/Hiding Clode Blocks
* You can disable a code block without commenting everything out using preprocessor directives.
```cpp
#if 0
    code
    ...
    code
    ...
    code
#endif
// Effectively omments out the code in the conditional code block
```

#### Adjusting definitons when compiling code
```cpp
#include <iostream>
using namespace std;

int main() {
    cout << X << endl;
}
// We don't have X defined
```

> Here's how we define X
```sh
g++ -DX=15 print.cc
```
**NB:** The nae of the symbol used does not matter. Can be more than 1 character.
    * `-DDEBUG` and `-D DEBUG` are the same.
#### `IFDEF`
* `#ifdef` is a preprocessor directive that checks if a symbol is defined.

```cpp
#ifdef DEBUG
    code
#endif
// includes the code in the block anywhere DEGUG is defined.
```

**NB:** I can get this to start in debug more by defining debyg
```sh
g++ -DDEBUG print.cc
```

* Symbols can be defined on the command line using 
    * `g++ -D__=__ <file.cc>`
* The preprocessor will replace whatever symbol defined.

> Let's see what how the preprocessor works in action

```cpp
#include <stdio.h>

int main() {
    printf("Hello World!\n");
}
```

```sh
g++ -E hello.cpp -o hello.i
```

```cpp
# All of stdio.h is included here

.
.
.

# 873 "/usr/include/stdio.h" 3 4
}
# 2 "hello.cc" 2


# 3 "hello.cc"
int main() {
    printf("Hello, World!\n")
}
```