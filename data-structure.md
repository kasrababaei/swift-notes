# Data Structure and Algorithms

## Hash Tables

Array and string questions are often interchangaeable.

A _hash table_ is a data structure that maps keys to values for highly efficient lookup. To insert a new key and value, we do the following:

1. Computed the key's hash code, which will usually be an int or long. Note that two different keys could have the same hash code, as there may be an inifinite number of keys and a finite number of ints. This is [known as Hash collision](https://en.wikipedia.org/wiki/Hash_collision),
2. Map the hash code to an index in the array.
3. At this index, there is a linked list of keys and values. Store the key and value in this index. We must use a linked list because of collisions: you could have two different keys with the same hash code, or two different hash codes that map to the same index.

If the number of collisions is very high the worst case runtime is _O(N)_, where _N_ is the number of keys. However, we generally assyme a good implementation that keeps collisions to a minimum, in which case the lookup time is _O(1)_.

Alternatively, we can implement the hash table with a balanced binary search tree. This gives us an _O(log N)_ lookup time. The advantage of this is potentially using less space, since we no longer allocate a large array. We can also iterate thourgh the keys in order, which can be useful sometimes.

## Array

In Swift, an `Array` is an ordered collection which offers random access. It also offeres sequential access to its element since it conforms to the `Sequence` protocol.
