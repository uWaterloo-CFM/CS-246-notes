# TUT 1 - 14 Sept 2022

# Environment Constants
* `$HOME` - Home directory
```sh
echo $HOME
```

* `$PWD` is the absolute path of your current directory. You can `echo` this too

## More complicated Output Redirection
> Suppose we have some program `printer` that will print even numbers to stdout and odd numbers to stderr (standard error)

> What if we want to redirect the output to stdout to `stdout.txt` and output to stderr to `stderr.txt`?

```sh
./printer > stdout.txt 2> stderr.txt
```


### Redirect standard output to standard error
```
echo "ERROR" 1>&2
```

* `./printer &> out.txt` Redirects all outputs (stdout and stderr) to `out.txt`

* This is also equivalent: `./printer 2>out 1>&2`

## Count number of occurences of words
> Suppose we want to output the top 10 occuring words to `top10.txt`

```sh
sort wordCollection | uniq - c | sort -n | tail > top10.txt
```

## Embedded Commands
* Subshell can be used to embed commands as command line arguments to scripts.
* Subshell will evaluate the embedded commands from the inside - out.
```sh
egrep $(cat file) myfile.txt
```

* Surrounding with back ticks, \` instead of `$()` work the same way but you can't nest them.

## Types of Quotes
* Double Quotes supress globbing, but not others. It allows for variable substitution
* Single Quotes supress everything inside of single quotes. It supresses variable substitution and embedded commands

## Useful Globbing Patterns
* `.*` in the pattern says we don't care about what, or how many occurences of what is in-between the patterns immediately to the left and right of it.

ie: `"[aeiou].*[aeiou]"

## Bash Variables
* In bash, a variable is declared as follows: `var=42` (Note the lack of spaces!)

* Accessing the value in a variable can be done with: `$var`

## Program Exit Codes
* When a program completes, it always returns a status code to signify the program was a success
* IN bash, if a program is successful, the exit code is 0. Otherwise it will return an exit code other than 0.

* The exit code is stored in the `?` variable. Access it with `$?`