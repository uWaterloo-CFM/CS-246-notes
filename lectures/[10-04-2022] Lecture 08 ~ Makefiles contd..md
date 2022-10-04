# Lecture 08 - 04 October 2022

## More Makefile Stuff

### `g++ -MMD -c -o main.i main.cc`
* `cat main.d`

> We can change our Makefile to account for this

```make
// TODO: Copy MakeFile from Camera Roll
```

* To adapt this makefile to any other program, i just need to update my `DEPENDANCIES` and `OBJECTS` variables.

#### `error: Redefinition of '...'`
* Common error that can happen when you include some header files in other files.
* This may be because you already included a header file in other file that you are including.
* ie: You are adding another header file to a file that already has that header file included.

> main.cc
```cpp
#include "linalg.h"
#include "vector.h"
...
```

> linalg.cc
```cpp
#include "vector.h" // This is already included in main.cc
...
```

* When the preprocessor sees the `#include "vector.h"` in `linalg.cc`, it will include it TWICE, so there will be an error.

#### This kinda sucks
* We have to keep track of which files transitively contain each other

> Solution: Use a **Header include guard**
* This is a way to prevent a header file from being included more than once.

> linalg.cc
```cpp
#ifndef VEC_H
#define VEC_H

// Definition of VEC_H
...

#endif
```

### Other things to keep in mind with header files
* Use include guards
* Use the keyword `extern` to declare a variable that is defined in another file. (**A Global Variable**)
    * Define the variable in the `.cc` **source file**
* Don't do `using namespace std` in your header file
* Don't compile header files
* Don't include source files


