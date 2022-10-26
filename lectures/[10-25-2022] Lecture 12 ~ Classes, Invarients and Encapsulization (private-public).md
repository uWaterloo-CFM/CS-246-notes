# Lecture 12 - 25 October 2022

**Recall - something wrong with this code:**
```cpp
struct Node {
    int value;
    Node *next;

    ~Node() {delete *next;} 
};

int main() {
    Node n1 = {1, new Node{2, nullptr}};
    Node n2 = {2, nullptr};
    Node n3 = {3, &n2};
}
```

* When your `main` function returns, it uses the destructor to delete `n2`.
    * Can't delete stack-allocated memory
* Our **assumption** here was that `Node *next` was either:
    * `nullptr` or; 
    * heap-allocated.

This assumption that `Node` makes is an **invarient** of the `Node` class.

### Example - BST

```cpp
struct TreeNode {
    int v;
    TreeNode *l, *r;
};

bool search(TreeNode *t, int v) {
    if (t == nullptr) return false;
    if (v < t->v) {
        search(t->l);
    } else if (v > t->v) {
        search(t->r);
    }

    return true;
}
```
* `search` only works with a BST. (Left always smaller than right), `node->left` always smaller than `node->v` and `node->right `always larger than `node->v`

We can't really reason about our obejects without enforcing invarients.

To do so, we need to **encapsulate** our objects, hiding fields/methods so that external users cannot modify them and break our assumptions.

**Example**

```cpp
struct Vec {
    private:
        int x, y;
    public:
        Vec(int x, int y);
    // Indentation doesn't matter
    // public and private identify sections
    Vec operator+(Vec o);
}
```
* `x` and `y` can't be seen or changed outside of `Vec`

### what does this mean?

**This is Ok**
```cpp
Vec Vec::operator+(Vec o) {
    return Vec{x+o.x, y+o.y}
} // This is ok!
  // Function is a member of Vec.
```

** This won't work**
```cpp
int main() {
    Vec v{0, 1};
    v.x; // bad, not a member
}
```

### Fixing our List example

**Attempt 1**
* Make `next` private
```cpp
struct Node {
    private:
        Node *next; // Doesn't work to solve the entire problem
    public:
        ~Node(){delete next;}
} // Node can't be changed to point to the stack,
  //    but we still construct them on the stack
```
* Perhaps this is the wrong abstraction for a **linked list** as it exposes Node construction.

## Class
**Attempt 2**
* Why don't we make a linked list class? (Big 5)
* Use the `class` keyword!
    * Automatically defaults to private members

```cpp
class List {
    public:
        void push_front(int v);
        void pop_front(int v);
        int ith(int index);

        // Constructor/Destructor
        List();
        ~List();
        // You can implement the rest of the big 5 here
        // Assignment 3
    private:
        struct Node {
            Node *next;
            int value;

            // Destructor
            ~Node();
        };
        // Head of the list
        Node *head;
}; // Don't forget semicolon at end

// Implement member functions

// 1. Constructor
List::List(): head(nullptr) {}

// 2. Destructor
List::~List() {delete head;}

// 3. Node Destructor
List::Node::~Node() {delete next;}

// Push function
void List::push_front(int v) {
    head = new Node{head, v};
}

// Pop function
void List::pop_front() {
    if (head) {
        Node *tmp = head;
        head = head->next;
        tmp->next=nullptr; // so we can call destructor
        delete tmp;
    }
}

// ith
int List::ith(int index) {
    List *cur=head;
    while (i--) cur = cur->next;
    return cur->value;
}
```
* All nodes are allocated on the heap
* We now can guaruntee that it won't get its invarients violeted
* To check if the List and Node invarients hold now we only need to check the implementation of List and Node
    * Only our functions can interact with the private members, and none of our functions violate our invarients.
        * `head` is private to `List`
        * `Node` is private to `List`
    * Therefore our program is safe

### Downsides of Encapsulization
If you want to do anything else with a linked list?
    * You don't have access to the pointers
    * Our class hides implementation details

ie: If you wanted to double each node's values, you would need to add a new function to your `List` class
* Add a new method to your `List` interface

You can also just make do with the existing methods, but that is quite difficult.

## Good List Class
* Traversing a list

```cpp
class List {
    public:
        void push_front(int v);
        void pop_front(int v);
        int ith(int index);
        int size(); // new

        List();
        ~List();

    private:
        struct Node {
            Node *next;
            int value;
            ~Node();
        };
        Node *head;

    // ... Implementations
};

// using our size() method
int main() {
    // l is a fully generated linked List object
    for (int i = 0; i < l.size(); ++i) {
        cout << l.ith(i) << endl;
    }
}
```
* But this is slow! Each loop calls our `size()` method.
    * Ends up taking quadratic time (O(n^2))

***

**Let's try this again**
(No `size()` method)
```cpp
// executes f for each value in the object
void List::ForEach(void (*f)(int v)) {
    List *cur = head;
    while (cur) {
        f(cur->value);
        cur = cur->next;
    }
}

void print(int v) {
    cout << v << endl;
}

l.ForEach(print);
```
* This kind of sucks because it is hard to do complicated things or implement alot of different functions each time I want to iterate

### Design Patterns
* Iteration is a problem that naturally rises when dealing with data structures/containers.
    * Lists 
    * Vector
    * Maps/Dicts 
    * Sets
    * .. etc

* Common problems often have common, good solutions to that problem
    * These solutions are known as **Design Patterns**

1. Iterator Pattern
```cpp
int arr[10];

for (int i = 0; i < 10; ++i) {
    // do something with arr[i]
    cout << *(arr + i) << endl;
}
```

We can also just do it with pointers
```cpp
int arr[10];
for (int* p = arr; p < arr + 10; ++p) {
    cout << *p << endl;
    // *p is called selection  
}
```

* An **iterator object** abstracts over a notion of a pointer providing:
    * `operator*()` for getting the element the iterator points to
    * `operator++()` for moving the iterator to the next value

**Implementation
```cpp
class List{
    public:
        // define iterator class inside of the List class
        class iterator {
            Node *cur;
        // Inside the List class
        public:
            int operator*() {return cur->value;}
            iterator& operator++() {
                cur = cur->next;
                return *this;
            }
            bool operator==(const iterator &other) {
                return cur == other.cur;
                // Check if element inside of the 
                // container is equal to the other 
                // thing in the container
            }
            bool operator!=(const iterator &other) {
                return cur != other.cur;
            }
        }
    iterator begin() {return iterator{head};}
    iterator end() {return iterator{nullptr};}
}

for (List::iterator i = l.begin(); i != e.end(); ++i) {
    cout << *i << endl;
}
```