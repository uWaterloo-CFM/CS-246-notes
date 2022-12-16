# Lecture 19 - 17 November 2022

## Observer Pattern
* Model a publish-subscribe model
* Publisher/Subject generates data
* Subscriber/Observer class:
    1. Receives data
    2. Reads to class

There are amny kinds of observer objects, and the subject can't (*and shouldn't*) be concerned with all of the details of all of these observers.

### UML
```
+-----------------------+
| *subject*             | *<stuff>* means italiacized
+-----------------------+                    +-----------------------+
| +attach(observer)     |<>----------------->| *observer*            |
| +detach(observer)     | has-a relationship +-----------------------+
| +notifyObservers()    |                    | +*notify()*           |
+-----------------------+                    +-----------------------+
             ^                                           ^
             |                                           |
             |                                           |
             |                                           |
             |                                           |
+-----------------------+                    +-----------------------+
| ConcreteSubject       | <----------------<>| ConcreteObserver      |
+-----------------------+                    +-----------------------+
| +getState()           |                    | +notify()             |
+-----------------------+                    +-----------------------+
```

1. When a subject is updated, we call `notifyObservers`
2. `notifyObservers` iterates over each observer attached to the subject, by calling notify.
3. Notify (in each observer) reacts to the ubdate by interating with the subject it is observing.
    * In this example, it does so by `getState()`

**NB:** The concrete subject keeps track of the observers by holding a container of subjects. (ie: `vector`)

### Example - Observers and Twitter
```cpp
class Tweeters: public Subject  {
        ifstream tweetDeck;
        string currentTweet;
        string name;
    public:
        Tweeters(string name, string filename): 
            name(name), tweetDeck(filename) {}
        string getName() {return name;}
        string getState() {
            return currentTweet;
        }
        bool tweet() {
            return getline(tweetDeck, currentTweet);
        }
}

class Follower: public Observer {
        Tweeter *f;
        string hash; // hastag to follow (not a cryptographic hash!)
    public:
        Follower(Tweeter *f, string hash): f(f), hash(hash) {
            f->attach(this);
        }
        ~Follower() {
            f->detach(this);
        }

        void notify() override {
            // find hashtag in tweet (uses string::find)
            if (f->getState().find(hash) != -1) {
                cout << f->getName() << " tweeted: " << f->getState() << endl;
            }
        }
}

// IMPLEMENTAIONS OF BASE CLASSES

// Observer just notifies the notify class
class Observer {
    public:
        virtual void notify() = 0;
};

// Subject is the one that is being observed
class Subject {
        vector<Observer*> observers;
    public:
        // add o to back of list
        virtual void attach(Observer *o) {obs.emplace_back(o);};
        // detach o from list
        virtual void detach(Observer *o) {
            for (auto it = observers.begin(); it != observers.end(); ++it) {
                if (*it == o) {
                    observers.erase(it);
                    return;
                }
            }
        }
        virtual void notifyObservers() {
            for (Observer* o: observers) ob->notify();
        }
};
```

```cpp
int main() {
    Tweeter elon {"Elon", "teslamemes.txt"};
    Follower SEC {&elon, "#insidertrading"};

    while (elon.tweet()) {
        elon.notifyObservers();
    }
}
```

## Dynamically Create Objects
### How Objects are Constructed right now:
* `Tweeter T = new Tweeter{"Kanye", "shoes.txt"};`

Here, we statically [*at compile time*] wrote down what object we wanted to make.

### How to Dynamically Create Objects
**Minecraft Example**

```
                     +-----------------------+
                     | Enemy                 |
                     +-----------------------+
                                ^
                                |
                                |
                                |
                    ----------------------------------------------------------
                    |                          |                             |
                    |                          |                             |
            +---------------+           +---------------+             +---------------+  
            | Zombie        |           | Creeper       |             | Arts Students | // They might not belong
            +---------------+           +---------------+             +---------------+

                    +--------------------+
                    | Level              |
                    +--------------------+
                              |
                              |
                    ----------------------
                    |                    |
                    |                    |
            +---------------+    +---------------+
            | Castle        |    | MC            | // MC like the math building
            +---------------+    +---------------+
```

Now, the type of Enemies you make depends on the type of Level you have. 

So we could do some if statements: 
```cpp
Level *l = ...;
if (l->getType() == 'MC') {
    return new ArtsStudent();
} else if (...) {
    ...
}
// But this is bad, since it gets very cumbersome
```

**Bettwer Way**
#### Factory Method Pattern (virtual constructor pattern) [Will make A4Q2 very easy]
* Create an object without exposing the creation logic to the client
    * The client only knows about the interface
    * Object creation abstracted away from users

Following our example, we just ask the Levels to make enemies
* Use polymorphism to create enemies depending on the level.

```cpp
// Level is itself a FACTORY for enememy classes
class Level {
    public:
        virtual Enemy *makeEnemy() = 0;
};

// A4Q2: You can just construct a new Operator very easily now (no if checks...)
class MC: public Level {
    public:
        Enemy *makeEnemy() override {
            return new ArtsStudent();
        }
};
```

## Template method pattern
### Drawing Turtles
We have red and green tutles, and we would like to draw turtles.
* We could have red and green turtles override a draw method defined in a `Turtle` base class
    * But this isn't ideal as the only difference here is in how the shell needs to be drawn.
Instead, implement what you can in the base class, and install `hooks` to what you need to be implemented (ie: `drawShell`)
* Implement as much of draw as you can in Turtle
    * ie: draw head, feet, legs, etc.
* Expose a **hook** for what needs to change **ie:** `drawShell`
```cpp
class Turtle {
    public:
        void draw() {
            // Call private helper functions
            drawHead(); // fully implemented in Turtle
            drawFeet(); // fully implemented in Turtle
            drawLegs(); // fully implemented in Turtle
            drawShell(); // we overload this later
        }
    private:
        void drawHead(); {:^)}
        void drawFeet(); {::}
        void drawLegs(); {||||}

        virtual void drawShell() = 0; // hook
};

// Now in my RedTurtle and GreenTurtle, I only need to
// implement overrides for drawShell instead of all of them
//  Less Reused code!
class RedTurtle: public Turtle {
    private:
        void drawShell() override {
            [A red shell]
        }
};

class GreenTurtle: public Turtle {
    private:
        void drawShell() override {
            [A green shell]
        }
};
```

This is called the **Template Method Pattern**. It is used for partially overriding behaviour in a base class.

* Make a hook - a virtual private method wrapping that behaviour.

***
When we define a **public virtual method**:
1. We define an **interface** to the object
2. Install a hook where subclasses can insert specialized behaviour
3. There's a *promise* that the method will be implemented in a subclass
    * More on this later
***
## Group assignment Details:
* Group of 3
* There will be a list of projects we can do sometime close to this weekend
* Projects are small little games that we can build in the console (command line based)
    * Minigames
    * GUI for bonus marks >.>
        * X11
        * Any GUI library "should" work
* Apply skills learned in this class to design your first small video game.
