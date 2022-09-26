# Lecture 01 - 08 September 2022

## Logistics: Object-Oriented Software Development

> What are we doing in this course?

* C++ - `More saftey than C, like in Racket, but faster speed`
* Object-oriented Paradigm - `Previously covered: Imperative (C)`
* Being free from abstractions --> Learn how to write and compile programs and interact with your computing using Linux Shell
* How software is developed

Edward Lee - `e45lee@uwaterloo.ca`

***

### Assignments, Grades, and Exams

* 40% Final
* 20% Midterm
* 25% Assignments (4 Assignments)
* 15% Group Project (1-2 people, maybe 3)

MUST Pass Assignments, Miderm, Final

Assignments are split into 2 parts, each part is due at the end of the week
* 1st part consists of writing tests
* 2nd part consists of submitting the solution

A0, A1 are shell assignments

A2, 3, 4 are coding assignments
***

## Lecture 1

### Linux Shell

Logging in:
```sh
ssh t54zheng@linux.student.cs.uwaterloo.ca
```

How to tell if you are running bash:
```sh
echo $0
```

> `-bash`

If this is ANYTHING ELSE: [www.student.cs.uwaterloo.ca/password](www.student.cs.uwaterloo.ca/password)

### Reading Files

Let's read a list of words on the server
```sh
cat /usr/share/dict/words
```

#### File/Folder Structure
* `words` is a **file** in the *directory* `/usr/share/dict/`

NB: `/` is the **root** directory

### Text Files in Linux
* All lines in Linux end with a newline `\n` character
* **Even the last line!**
* Marmoset needs files to end in a newline (For `vi`, no extra newline is required)

### Kill a currently running program
* Type `Control + C`

### List a set of files in the working directory
* `ls` outputs a list of files in the directory that you are currently working with
* `ls -a` will show a list of **all** files, including hidden files!

NB: Files/folders starting with `.` are **hidden**! (These are called **Hidden Files**)

### Show me the current folder I'm working in
* `pwd` - Print Working Directory
* Outputs the directory that you are working in

### Change Directories
* `cd dir` changes your directory into the inputted directory (can be absolute or relative)
* `cd ~` changes your directory to your default home directory (**specific** for your user)

NB: `~` != `/`

* `.` is your current directory
* `./test.txt` is exactly the same as `test.txt`
* `cd ..` changes your directory back to the parent directory of the directory that you are in
* `cd ../../` brings you up 2 directories


### Absolute vs. Relative Paths
* Paths do not always have to start with a `/`
* Paths that do not start with `/` are interpreted **relative** to your current working directory
* Paths that start with `/` are **always** absolute, and will be relative to the root directory `/`

### Typing `cat` without any args
* `cat` will just echo whatever it is given
* You *can* kill it using `Control + C`

> But how do we stop cat?
* `Control + D` Sends an end-of-file indicator
* `Control + C` Kills


### Writing stuff into files using `cat`

```sh
cat >246.txt
```

Redirects whatever you typed into shell into `246.txt`

### Redirecting input
```sh
cat <246.txt
```

Inputs the contents of `246.txt` into `cat`


### Input Redirection

* `cmd < input` redirects input
* `cmd > output` redirects output

### Line numbers 
* `cat -n <246.txt` enumerates lines of text in `246.txt`

### End text files with newline!
* If you do not insert a newline to the end of a text file, and then try reading it using `cat`, stuff goes weird

### How to check
* `wc file` will count lines, words, bytes

### Command streams
* in: Standard input (keyboard)
* out: Standard output (prints to terminal)
* `another one!` out: Standard Error (goes to standard output and prints error in terminal) - Unbuffered, will be written IMMEDIATELY 
* Even if program dies, the error message is still printed (seperate from standard output, which is buffered)

### Redirect Standard Error messages
```shell
wc notafile 2>error.txt >count.txt
```

* Will not add anything to error.txt if everything worked fine
* Will redirect error

### Operating on a bunch of files

* `wc` actually can operate on a list of files
```sh
wc a.txt b.txt c.txt d.txt sample.txt test.txt ...
```

### Globbing Pattern
> All text files (pattern matching)
```sh
wc *.txt
```

* `wc` will match any file that **ends** with `.txt`


### Head
* Head will only read the **head** of any file, up to the specified number of lines or bytes

```sh
head -n 10 words.txt
```
* Prints the first 10 lines

```sh
head -n [file]
```
* Reads off the first k lines of file

### Piping
```sh
head -10 words.txt | wc
```
> Pipes the output of `head` into the input for `wc`

