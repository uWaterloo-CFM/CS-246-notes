# Lecture 18 - 15 November 2022

## Logistics
1. Review session probably on Thursday, somewhere in MC, post questions on the Piazza post.
2. Instructors will not be able to provide course notes, but will post summaries on important points.
3. If you need notes, email the ISA.

***

## Error Handling

**Problem**

Where errors are reported is often not where you can deal with them (when program is compiled).

```cpp
vector<int> v = {1, 2, 3};
v.at(3000) = 10; // this doesn't work

// this checks if the index is in range, but how?

v[1000] = 5; // can we do this?
```

Here, `v.at()` doesn't know how to deal with the **bad index.** 
* ie: It has no code to handle what to do when someone tries to access an index that is out of range.
* It just throws an exception, which is a signal that something went wrong, but it can't recover from it. It just terminates the program.
* **Is there a way for the user to implement some code to handle this in order to recover and keep the program running as intended?**

### Solutions?

We could use error codes, but:
1. They tend to be repetitive (lots of error 'handl-ly' logic).
    ```cpp
    if (error1) {
        // ...
    } else if (error2 ...) {
        // ...
    } else if // ...
    // ...
    // THIS SUCKS
    ```
2. They're fragile (need to remember to free resources)
    ```cpp
    int *arr = new arr[10]
    if (someCodethatCanFail()) {
        // ok, arr is freed
    } else {
        failiure
        // people usually forget to free memory here, causing mem leaks
    }
    ```
3. Error propogation
    * How do we report errors? Do we need a frick ton of error codes?

## Actual Solution! - Exceptions and Exception Handling
We **throw** exceptions to indicate errors. We can **catch** them to handle the *specific* errors using an exception handler to deal with them in the way that *we actually want to*.

**Syntax + Example**
```cpp
void f() {
    vector v{1, 2, 3};
    v.at(1000); // throws out_of_range exception
}
void g() {f();} // g just calls f
void h() {g();} // h just calls g

int main() {
    try {
        h(); // runs h and installs the exception handler block
    } catch (out_of_range e) { // throws an exception object e, can call it anything
        // This is the exception handler, JUST for out_of_range
        cout << "caught an out of range error" << e.what() << endl;
    }
}
```

#### Benefits
* As we unwind the stack, we also automatically destroy objects allocated on the stack. The vector we allocated in `f()` is automatically destroyed when we catch the exception in `main()` (and we don't have to worry about freeing it since it was stack allocated).

#### Throwing Exceptions
```cpp
void f() {
    throw out_of_range {'some message'};
} // throws an exception object of type out_of_bounds with the message
```

* `out_of_range` is just a class...
* There are alot of exceptions in the standard library
* There is a **hierarchy** of exceptions, and a main exception that **all** exceptions inherit from. The generic **`exception`** class.

#### `exception` Class
* If we wanted main to handle all subtypes of exceptions, we could just catch `exception` instead of `out_of_range`.
```cpp
void f() {
    throw out_of_range {'some message'};
} // throws an exception object of type out_of_bounds with the message

int main() {
    try {
        f();
    } catch (exception e) {
        cout << "caught an exception" << e.what() << endl;
        // Even though f() doesnt always finish running, the vector is still destroyed
    }
}
```

* More exceptions found [Here](https://en.cppreference.com/w/cpp/error/exception)

#### Should we catch exceptions by reference? (`exception &e`)
* We don't want to copy the exception object, because it's expensive (copying)
```cpp
catch(exception e) {
    // makes a new std::exception object by making a copy of
    // the originally thrown exception
}

// vs

catch(exception &e) {
    // just makes a reference to the original exception object
    // more optimal
}
```
**TL;DR** Catch exceptions by reference, DONT COPY IT ANYMORE
* `catch(exception &e)` is your friend, use it

### Propagating exceptions
```cpp
try {
    // ...
} catch (exception &e) {
    // ...
    throw e; // rethrow a COPY of the exception
}
```
* This code actually creates a new `exception` object by **copying** e. (sometimes it could do it by **moving** e).

#### Correct way to rethrow exceptions
The way you want to do it properly is just to call `throw;` without any arguments.

```cpp
try {
    // ...
} catch (exception &e) {
    // ...
    throw; // rethrow the original exception
}
```

### Making our own exceptions
```cpp
class BadInput {}; // could also be class BadInput: public exception {};
                   // if we want to also catch it with generic exception

try {
    if (cin >> n) throw BadInput{};
} catch (BadInput &e) {
    // you don't NEED to use e if you don't want to
    cerr << "ERROR" << endl;
}
```

#### You can do this even without objects
```cpp
try {
    throw 42; // WARNING: Can only throw ONE exception at a time
} catch (int x) { // binds x to 42
    cout << x << endl; // prints 42
}
```

#### Note
* Only one exception can be thrown at a time
* **Destructors should NEVER throw exceptions**
    * There could potentially be two exceptions alive on the stack at the same time.
    * If this happens, runtime kills the program.
