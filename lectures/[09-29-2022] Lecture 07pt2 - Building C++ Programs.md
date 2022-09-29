# Lecture 07 - 29 September 2022

## Building C++ Programs

### Seperate Compilation
* Split program into composable modules.

* This requires: 
    * Interface for the composable modules
    * Function prototypes (in `.h` files)
    * Implementation fo every provided function in the `.cc` file.

### Compilation

### First attempt
* Call compiler on all source filees.
`g++ vec.cc main.cc`

### Compiling for cheaper
* WHen you compile alot of files, the file size/memory usage could rise significantly
* WHat if I just want to update **one8* of my files?

### Solution: Object Files
* compile each source file to an object file.
    *  Link object files together at the end

```sh
g++ -o vec.o -c vec.cc
...


```sh
g++ -o vec.o
```

```
Link files together

g++ -o main main.o vec.o
```

**TL;DR** To link files together, I only need to compile the file I changed, then link them together.

Some problems of ths method:
* Need to remember dependencies
    * main.o depends on main.cc
    * vec.o depends on vec.cc and vec.h
* Need to run many commands when running the files
    * Lots of compiler calls

### Solution: Makefiles
* Makefiles are a way to automate the process of compiling and linking files together.

* `make`
    * Manages dependencies
    * manages recipes for building dependencies

**Example: Vector Tools makefile**

* `vi Makefile`
    * Make will automatically rebuild files if they are out of date with their depenencies.
    * Will use the depenancy graph from your Makefile to rebuild your program

```make
main: main.o vec.o
    g++ -o main main.o vec.o

main.o: main.cc vec.h
    g++ -o main.o -c main.cc

vec.o: vec.cc vec.h
    g++ -o vec.o -c vec.cc
```

**But we can make it better**
```make
CXX = g++ # Compiler
CXXFLAGS = -std=c++14 -Wall # Compiler flags for g++
OBJECTS = main.o vec.o # List of object files
PROGRAM = main # Name of the program

${PROGRAM}: ${OBJECTS}
    ${CXX} ${CXXFLAGS} -o ${PROGRAM} ${OBJECTS}

main.o: main.cc vec.h
vec.o: vec.cc vec.h

.PHONY: clean // PHONY only runs if you type `make clean`, you can have more than 1 ie: `.PHONY clean echo`
clean:
    rm ${OBJECTS} ${PROGRAM} // Removes all intermediary files.
```

* Make's default recipe is ${CXX} ${CXXFLAGS} -o main.o min.cc


### Print dependancies of a file (g++)
* `-MMD`
* `g++ -o vec.o -MMD -c vec.cc`
* `cat vec.d`