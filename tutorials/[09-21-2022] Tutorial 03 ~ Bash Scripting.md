# Tutorial 3 - 21 September 2022

## Bash Scripting
* Try to always have a shebang line `#!/bin/bash`
* You will usually have to `chmod` execute perms to a bash file you write

* You can get around having to give execute perms by running `bash filename`

## Passing peramaters to functions

```sh
#!/bin/bash

subroutine() {
  echo $1
  echo $(1%00)
}

subroutine $(($1 * 1000))
```

* The above program passes the first argument into `subroutine()`, but multiplied by 1000.

```sh
#!/bin/bash

sum() {
    echo $(( ${1} + ${2} ))
}

product() {
    echo $(( ${1} * ${2} ))
}

sum $( product ${1} ${1} ) $( product ${2} ${2} )
```

### Exercise
> Write a program that takes two arguments, `ext1`, `ext2`, and renames all files with extension ext1, and rename it to end with ext2.

## if statements
```sh
for var in $(seq 1 10); do
  echo $var
done

# Alternate Method
for var in {1 2 ... 10}; do
  echo $var
done
```


## Test command
* A test program is [, and in format [cond]
* `man [` pulls up the manual

## Testing
* Check implementation from `~/cs246/1229/tutorials/tut02/gan_wang/testing`
* Test suites have to be implemented to ensure that your program is robust
* Test cases should **each** test for something meaningful. Do NOT have redundant tests

# Types of tests
* General/basic tests - santify checks
* Equivalence classes - Differently sized inputs, including empty input

## Testing Exercise
* We have a compiled program test which is invoked as follows:
* `./test <testnum>`
* Where `<testnum>` is an integer between 1 and 8 denoting the number for the test you want to run. 
* The program then expects from standard input lines of integers separated by whitespace. The program takes a
line of integers and tests if they satisfy a certain property (which will depend on `<testnum>`). 
* The program will return ‘true’ for each line of numbers which satifies the property and ‘false’ for all other input. 

* For example, for test number 1, the input:
`2 4 6`

* returns ‘true’.

* In check.h, for each test number there is a “Claim” claiming what that test number’s property is. 
* Some of these claims are correct, and others are incorrect. Come up with some test cases (i.e. inputs) and determine which claims are correct and which are incorrect.

> check.h, nb: test.cc has all of the implementations

```cpp
#include <string>

// Claim: Tests if all numbers have the same parity.
bool test1(std::string);

// Claim: Tests if the numbers are in decreasing order.
bool test2(std::string);

// Claim: Tests if there are at most 3 numbers.
bool test3(std::string);

// Claim: Tests if the numbers sum to at least 25.
bool test4(std::string);

// Claim: Tests if the numbers sum to more than 0.
bool test5(std::string);

// Claim: Tests if all the numbers are positive.
bool test6(std::string);

// Claim: Tests if at some point, the cumulative sum is at least 2.
bool test7(std::string);

// Claim: Tests if first <= last number.
bool test8(std::string);
```

# Debuggig Bash Scripts

* Adding `-x` to the end of the shebang line gives you a list of commands that are being executed by the bash file
* `#!/bin/bash -x`
