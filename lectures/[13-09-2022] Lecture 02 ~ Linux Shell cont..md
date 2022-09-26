***

# Lecture 02 - 13 September 2022

## Sort a file 
* `sort`

## Deduplicate a file
* `uniq`
* Has to be a sorted list (SORT IT FIRST)

## Print out all files that start with `head`

```sh
cat head*
```

## Print out all files that start with `head` AND remove all duplicates

```sh
cat head* | sort | uniq
```

## String Substitution

### `echo`

```sh
echo "Hello World"
```

> output:

```sh
Hello World
```

### whoami

```sh
whoami
```

> output:
```sh
t54zheng
```

### String sub.

> Let's substitute the output of `whoami` into the string I am printing to `echo`

```sh
echo "Hi, I'm $(whoami)
```

> What about the time and date?

* use `date`

```sh
echo "It's currently $(date)"
```


### More applications

> I want to read a file `filetoread`, which contains the file name of a file that I want to read


```sh
cat $(cat filetoread)
```

### `cat` technicalities

> Are these the same?

```sh
echo "Its currently $(date)"

echo Its currently $(date)

```

> No. **Without quotes**, bash will take into account globbing patterns. It is safer to use quotes.

### What about this?

```sh
echo 'Its currently $(date)
```

> Output:
```sh
Its cirrently $(date)
```

> No! Using **single quotes** does not take into account the special characters we put into it.


## Replacing VSCode with Bash
* This sounds really hard. It seems like Bash cannot replace our favourite IDE.

> Think about if we want to search for a substring in a file.

## `egrep`

* `egrep <pattern> <file>`

* Patterns are in regex

***

### Examples

```sh
egrep "n" file.txt
```
> Finds all lines with a `n` in it.


```sh
egrep "cs" file.txt
```
> Finds all lines with `cs` in it.

```sh
egrep "cs246" file.txt
```
> Finds all lines with `cs246` in it.

## By default, `egrep` is case sensitive
* How can we make it match any pattern?

* Use `|` to make either pattern true
* Use `()` for grouping 

Example:

```sh
egrep "(C|c)s246" file.txt
```

```sh
egrep "(C|c)(S|s)246" file.txt
```

### Searching if a line has a n, p or l
```sh
egrep "n|p|l" file.txt
```
> ^ This was quite inconvenient, is there a simpler way?

> Yes!

```sh
egrep "[npl]" file.txt
```

### Negation

* What if I want to match everything other than some character?

> Character set negation
```sh
egrep "[^cC][sS]246" file.txt
```

### Optional Patterns
* I can optionally match patterns as well.

> Say I want to match `CS 246` and `CS246`

```sh
egrep "[cC][sS] ?246"
```

### Matching repeated patterns of arbitrary length

> Say I want to match `CSCS246` or `CS246` or `CSCSCS246` or `246`

* Use `+` to match **1** or more occurences of the preceding item
* Use `*` to match **0** or more occurences of the preceding item
* Use `?` to match **0** or **1** occurences only of the preceding item

```sh
egrep "([cC][sS])+246"
```
* This will match 0 to a arbitrarly large finite number of CS's, regardless of case, followed by one occurence of `246`

> Notice the grouping

### Anchors
* Let's only match lines that start with `lcs`
```sh
egrep "^l([cC][sS])+246"
```

* Similarly, mark the **end** of a ine with `$`
```sh
egrep "^l([cC][sS])+246$"
```

### Escaping Characters

* Let's say I have `^1234` in one of my files. How do I search it?

```sh
egrep "\^1234"
```

## Searching `man` pages
* Type in the `/` key, then type in the pattern

## Matching any one character

* `.` will match any single character
* Think of it like a single space wildcard (only 1 character long)

## Matching all lines with **two** characters
```sh
egrep "^..$" file.txt
```
## Matching all lines with an **even number of characters**

```sh
egrep "(..)*"
```

## Searching if there is a **single** `e` in a line
```sh
egrep "^[^e]*e[^e]*$"
```

## Testing `egrep` quickly

```sh
echo "somestring" | egrep "somepattern"
```

## Linux Permission Bits

```sh
ls -l
```

```
drwxr-sr-x  2 t54zheng t54zheng  0 Aug 23  2021 bin
drwxrws---  2 t54zheng cs135     0 Aug 23  2021 cs135
drwxrws---  4 t54zheng cs136     2 Apr  5 13:37 cs136
drwxrws---  2 t54zheng cs245     0 Aug 29 15:16 cs245
drwxrws---  3 t54zheng cs246     1 Sep  7 00:48 cs246
-rw-r-----  1 t54zheng t54zheng 16 Apr  5 13:40 index.html
drwxrws---  2 t54zheng t54zheng  0 Aug 23  2021 NexusAppData
drwxrws---  2 t54zheng t54zheng  0 Aug 23  2021 NexusDesktop
drwxrws---  2 t54zheng t54zheng  0 Aug 23  2021 NexusMyDocuments
drwxr-sr-x  2 t54zheng t54zheng  0 Apr  5 13:40 public_html
drwx--S--- 12 t54zheng t54zheng 10 Mar 21 11:01 SettingsPackages
drwx--S---  3 t54zheng t54zheng  1 Feb 18  2022 WINDOWS
drwxrws---  2 t54zheng t54zheng  0 Aug 23  2021 Windows2000
drwxrws---  2 t54zheng t54zheng  0 Aug 23  2021 Windows2000.V2drwxr-sr-x  2
```

* Bit `1` (first bit) - `d` for directory, `-` for file
* Bit `2-10` - Permissions that have been granted for each user

* Bit `2-4` - Permissions of Owner
* Bit `5-7` - Permission of User Group
* Bit `8-10` - Permission if you are not the owner or in the user group

Permission Flags:
* `r` - read
* `w` - write
* `x` - File can be executed (is a program)