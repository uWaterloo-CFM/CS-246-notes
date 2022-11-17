# Special "Because we are all stupid" Review Session - 17 November 2022

## Topics
* Iterators
* Inheritance
* System Modeling (UML)
* Design Patterns

### Iterators
* Meant to **abstract** away the traversal of a data structure away from the client
* Rather than having to traverse a data structure themselves, you can **force** them to use an iterator
    * Makes accessing data safer and easier
    * Prevents the client from accidentally modifying data they shouldn't
* Iterators have a reference to the data structure it is iterating over

#### How to build an Iterator
* Need to reference to the underlying object you're iterating over
* Need to define what it means to iterate (i.e. move to the next element) over to an object (prefix ++)
* Need a way to check if two iterators are (not) equal (==, !=)
* Need a way to return data from the iterator (unary *)
* Need to define `end()` in a way such that it references some location **after** the entire list, **not the last element** in the list.
* `begin()` points to the first element in the list

#### TierList example
* A TierList is a list of lists
```
TierList
    List** tiers // array of pointers to lists
        List* 0 --> [1, 2, 3, 4 \]
        List* 1 --> [5, 6, 7, 8 \]
        List* 2 --> [9, 10, 11, 12 \]

    int numTiers;
```
* We want to be able to iterate over the entire list, not just one tier

**Iterator**
* Make sure you have a reference to the thing you are iterating over
```
TierList Iterator
    List** tiers // points to the same thing as the TierList
    int curTier; // current tier
    List::Iterator it; // current list iterator I NEEDED THIS
```

**`TierList::operator++`**
* `++it` should move to the next element in the current Tier.   
    * If it reaches the end of the current Tier, it moves to the next non-empty Tier in the list, or to the end if there are no more non-empty tiers
```cpp
TierList::Iterator& TierList::Iterator::operator++() {
    ++it; // move to the next element in the current tier
    if (it == tiers[curTier]->end()) { // if we reached the end of the current tier
        // move to the next non-empty tier
        while (++curTier < numTiers && tiers[curTier]->empty()) {
            // do nothing, ++curTier moves to next
        }
        if (curTier < numTiers) { // if we found a non-empty tier
            it = tiers[curTier]->begin(); // set the iterator to the beginning of the new tier
        }
    }
    return *this;
}

// Define end of TierList as the end of the last non-empty tier
TierList::Iterator TierList::end() {
    int lastTier = numTiers - 1;
    while (lastTier >= 0 && tiers[lastTier]->empty()) {
        --lastTier;
    }
    return Iterator(tiers, lastTier, tiers[lastTier]->end());
}

// Define start of list
TierList::Iterator TierList::begin() {
    int curTier = 0;
    while (curTier < numTiers && tiers[curTier]->empty()) {
        ++curTier;
    }
    if (curTier < numTiers) {
        return TierList::Iterator(tiers, numTiers, curTier, tiers[curTier]->begin());
    } else {
        return TierList::Iterator(tiers, numTiers);
    }
}
```

### Inheritance/Polymorphism
SAME AS HIS LEC SLIDES JUST LOOK AT THESE BRO

### UML
LOOK AT SLIDES

## Eric's Part - Pokemon (Decorators)
* Charizard
    - Fire/Flying

How do we make the animals hit each other?

* Charizard gets hit by surf

Surf: water type

Water does 2x damage to fire
Fairy does 1x damage to fire 
...
all of the combinations, you get a ton of relationships

### Alternative: Define the relationship between types ONCE then reuse/combine them
* You can just **decorate** the pokemon with its type(s)
* Charizard
    - Fire
    - Flying

```
fire -> water 0.5x
fire -> grass 2x
...
```

I can just declare an **AbstractPokemon**
```cpp
Pokemon *p = new BasePokemon{"Charizard"};
p = new FireType{p};
p = new FlyingType{p};
```

You end up with a linked list like
```
FlyingType -> FireType -> BasePokemon
```

```cpp
p->hitBy(WaterMove m ...);
```
### UML Diagram is the same as the one from Ed's Lecture slides
