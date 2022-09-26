# Lecture 04 - 20 Sept 2022

## Shell scripting, contd.

**NB:** All lecture examples can be found in `~/cs246/1229/lectures/shell/scripts`

**RenameCPP**
```sh
#!/bin/bash
# Renames all .cpp files in the current directory to .cc

for name in *.cpp do
        mv ${name} ${name%cpp}cc
done
```

**countWords**
```sh
#!/bin/bash
# countWords word file
#  Prints the number of times word occurs in file

usage() {
        echo "${0} word file"
        exit 1
}

# Do we have exactly 2 command-line arguments or not?
if [ $# -ne 2 ]; then
        usage
fi

counter=0  # Initialize the counter to 0.

#Iterate over every word in the file, ${2}.
for word in $(cat "${2}"); do
  if [ ${word} == "${1}" ]; then
    counter=$((counter + 1))
  fi
done
echo ${counter}
```

### Last thing for Shell Scripting


**payday**

```sh
#!/bin/bash
# Returns the date of the next payday (last Friday of the month)
# Examples:
# payday (no arguments) -- gives this month's payday
# payday June 2020 -- gives payday in June 2020

usage () {
  echo "$0 [month year]"
  exit 1
}

report () {
  if [ $# -eq 3 ]; then
     echo -n ${2} ${3}
  else
     echo -n "This month"
  fi
  echo -n "'s payday is on the "
  if [ $1 -eq 31 ]; then
    echo "31st."
  elif [ $1 -eq 22 ]; then
    echo "22nd."
  elif [ $1 -eq 23 ]; then
    echo "23rd."
  else
    echo "${1}th."
  fi
}

if [ $# -ne 0 -a $# -ne 2 ]; then
  usage
fi

report $(cal $1 $2 | awk '{print $6}' | grep "[0-9]" | tail -1) $1 $2
```

* `report $(cal $1 $2 | awk '{print $6}' | grep "[0-9]" | tail -1) $1 $2` shows you how to pass arguments into the function

## Testing

* Program correctness proofs are hard (formal proofs)
* Instead, let's go with testing.

> When do you test?

* Before we even start **writing** the code
* Continuously, to verify your program as you develop.
* Testing doesn't ensure correctess, only ensures that it lacks incorrectness in the test cases you have written.
* Testing relies on experience, it is ad-hoc. There is no general way that you should go about it.

### Methods to test your code

1. Manual Testing
* PR Reviews
* Manual (human-driven) interactions with program.
* Manual testing does not scale.

2. Automated Testing
* Create a testing tool, and a **test suite** of test inputs and expected behaviours.
* This tool automatically checks our program by making sure that the outputs of the program match the expected outputs in the test suite. (Checks against the test suite)

**NB:** In this course, our test suites are going to be a set of inputs, and the expected outputs. Just strings to standard output/files.

## Black-Box Testing
* Set of test cases designed without knowledge of how the implementation works.

#### Ex. Various classes of inputs
* Test each class (for example, positive and negative numbers)

### Boundary Cases (edge cases)
* Test edge/boundary cases for cases where they exist
* For example, make sure the program works for each corner of the grid

### Extreme cases
* Check extreme values that are valid inputs
* For example, check if the calculator works at near the 32-bit integer limit

### Base Cases
* Test base cases 

### Intuition and experience.
* Test early and test often

## White-Box Testing

### Coverage Testing
* Testing with implementation details
* Know how your program works, so you can test all of the lines in your program
* If you have an `if` statement that changes behaviour depending on the value of an inout, **test around** these value(s)!
* Test every function

## Performance Testing
* Make sure program is fast/efficient enough

## Regression Testing
* Testing to check that new changes don't break existing behaviour

**NB:** Don't test invalid inputs unless spec otherwise specifies.
* Does the program need to exit gracefully?
* Do we care about whether or not the program breaks on invalid inputs?