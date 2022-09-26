# Lecture 03 - 15 September 2022

## Permissions cont

```sh
ls -l
```

```
-rw-r----- 1 t54zheng cs246 0 Sep 15 14:33 test.txt
```

```
_(type)  _ _ _ (user perms)  _ _ _(group perms)  _ _ _(other user perms)

* r - read permission
* w - write permission
* x - execute permission
* s - File should be executed as a particular user (do not worry abt this for this course)
* All other bits are beyond the scope of the course
```

## Modifying Permissions
```sh
chmod <mode> <file>
```

> Make `cs246` read, write, and executable by people other than me and not in my group

* I need to set the `other` section's bits to `rwx`

```sh
chmod o=rwx cs246.txt
```

### Removing Permissions
```sh
chmod o-x cs246.txt
```

* `o-x` removes all permissions after `x` from other user perms.

* similarly, `o+x` will add `x` back to other user perms.

* using just `+x`, without any target, this will apply it to all


### Groups
* `u` - User
* `g` - Group
* `o` - Other
* `a` - Everyone

### Args
* `+` - add all perms following `+`
* `-` - Remove all perms following `-`
* `=` - Set permissions to the ones inputted after `=`. Omitted perm bits are set to `-` (disabled)

### Directory Permissions
* Dictories stater with `d`

* `r` - Read grants list directory contents
* `w` - Write grants writing/deleting from the directory (and subdirectories)
* `x` - Permission to traverse the directory

> Traverse?

It gives you the ability to open the directory (`cd` into the directory)

## Programming in Bash - Shell Scripts

### Shebang line
starting bash scripts
* This has to be the FIRST line of your shell script
* This is the interpeter to run the file with
* Lines that start with `#` is a comment

```sh
#!/bin/bash

```


> Write a shell script that Prints my name, the date and the current directory

```sh
#!/bin/bash

# print my name
whoami
# print the date
date
# print the current directory
pwd
```

### Next steps
```sh
./first.sh
```

^ This does not run, no permissions!

```sh
ls -l
```

I don't have execute perms!
```
-rw-r----- 1 t54zheng cs246 92 Sep 15 15:02 first_shell_script.sh
```

```sh
chmod u+x first_shell_script.sh
```

```
t54zheng
Thu 15 Sep 2022 03:04:28 PM EDT
/u7/t54zheng/cs246/1229/lectures/LEC_03
```

> Yay!


### Variables

```sh
#!/bin/bash

x=1

echo "x=${x}"

x=2
echo "x=${x}"

# Be careful using quotes!
x=3
echo 'x=${x}' "x=${x}" x=${x}
```

> Output
```sh
x=${x} x=3 x=3
```

### Not using `{}`
* you can still access variables without the {}: ie: `echo $x`
* This is however, NOT recommended if you want to print anything AFTER it
* ie: `echo ${x}nd` vs. `echo $xnd`

## Variable Types
* Variables **ONLY** hold strings!


## Declaring variables just in the shell environment
* We can still just declare any variable interacivley in the command line


### `set`

* All variables that are currently defined

* `echo ${PATH}` - colon-seperated list of directories that gives a list of all programs that shell can look at to try running your file

## Passing in arguments

* `$1, $2, $3 $..` are a list of all arguments passed into the command

> args.sh

```sh
#!/bin/bash

echo ${1}
```

```
./args.sh hi
```

> Output:
```
hi
```

> Write a program that takes in a word and checks if it exists in the english dictionary

```sh
#!/bin/bash

word=$1

egrep "^${word}$" /usr/share/dict/words
```

## Conditional Statements (if, else, else if) - Password Checker
> Test if a password is a good password or not. Print out if a password is good or not

```sh
if test; then
   commands...

[elif test; ...]
[else ...]
```

```sh
#!/bin/bash

# A strong password:
# DOes not have a word that exists in the english dict

word=$1
egrep "^${word}$" /usr/share/dict/words > /dev/null

# $?
if [ $? -eq 0 ]; then
        echo "Bad password"
else
        echo "Maybe OK"
fi
```


## Functions

```sh
#!/bin/bash

# A strong password:
# DOes not have a word that exists in the english dict

usage() {
        echo "usage: $0 <password>"
}

if [ $# -ne 1 ]; then
        usage
        exit 1
fi

word=$1
egrep "^${word}$" /usr/share/dict/words > /dev/null

# $?
if [ $? -eq 0 ]; then
        echo "Bad password"
else
        echo "Maybe OK"
fi
```

## Useful special variables
* `$?` - last return value
* `$#` - number of arguments

## Loops
> Print numbers from 1 to whatever was passed into the program

### Math in bash
* `$((expr))` - arithmetic expression
```sh
x=1
while [ $x -le $1 ]; do
        echo $x
        x=$((x + 1))
done
```