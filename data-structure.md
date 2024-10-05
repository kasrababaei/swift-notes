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

### In-place algorithm

> In computer science, an in-place algorithm is an algorithm that operates directly on the input data structure without requiring extra space proportional to the input size. In other words, it modifies the input in place, without creating a separate copy of the data structure. An algorithm which is not in-place is sometimes called not-in-place or out-of-place. _[Source: Wikipedia](https://en.wikipedia.org/wiki/In-place_algorithm)_

In Swift, this is kinda similar to using an `inout` parameter. In other words, have to modify the input directly instead of copying the whole input, i.e., allocating more memory. For example, given an array of _n_ elements, to reverse it in-place, can swap the elements using the indicies:

```Swift
func reverseArray(_ array: inout [Int]) {
  var left = 0
  var right = array.count - 1
  
  while left < right {
    // Could also use the built-in swap(_:_:) method.
    let temp = array[left]
    array[left] = array[right]
    array[right] = temp
    
    left += 1
    right -= 1
  }
}
```

It is used in various algorithms such as bubble sort, comb sort, selection sort, insertion sort, heapsort, and Shell sort. These algorithms require only a few pointers, so their space complexity is `O(log n)`.

Swift uses [Timsort](https://en.wikipedia.org/wiki/Timsort) which is a hybrid, stable sorting algorithm, derived from merge sort and insertion sort, designed to perform well on many kinds of real-world data, refer to _[Sort.swift](https://github.com/swiftlang/swift/blob/main/stdlib/public/core/Sort.swift)_ file. It was implemented by Tim Peters in 2002 for use in the Python programming language. The algorithm finds subsequences of the data that are already ordered (runs) and uses them to sort the remainder more efficiently. This is done by merging runs until certain criteria are fulfilled.

## Linked Lists

A linked list is a data structure that represents a sequence of nodes, in a singly linked list, each node points to the next node in the linked list. A doubly linked list gives each node pointers to both the next node and the previous node. When dealing with a coding challenge, need to understand if we're dealing with a singly linked list or a doubly linked list.

Unlike an array, a linked list does not provide constant time access to a particular "index" within the list. This means that if you'd like to find the `K`th element in the list, you will need to iterate thorugh `K` elements.

The _benefit_ of a linked list is that you can _add/insert_ and _remove/delete_ items from the beginning of the list in constant time.

If we want to add a new value after a given node prev, we should:

1. Initialize a new node `cur` with the given value
2. Link the "next" field of `cur` to `prev`'s next node next
3. Link the "next" field in `prev` to `cur`

Unlike an array, we don’t need to move all elements past the inserted element. Therefore, you can insert a new node into a linked list in `O(1)` time complexity if you have a reference to `prev`, which is very efficient.

So it is essential to update head when adding a new node at the beginning of the list.

1. Initialize a new node cur
2. Link the new node to our original head node head
3. Assign cur to head

```Swift
var head = ListNode(2)
var tail = head

for value in 3...10 {
    let new = ListNode(value)
    tail.next = new
    tail = new
}

head // 2 -> 3 -> 4 -> 5 -> 6 -> 7 -> 8 -> 9 -> 10

let cur = ListNode(1)
cur.next = head
head = cur // 1 -> 2 -> 3 -> 4 -> 5 -> 6 -> 7 -> 8 -> 9 -> 10
```

### Deleting a node from a singly linked list

If we want to delete an existing node cur from the singly linked list, we can do it in two steps:

1. Find cur's previous node prev and its next node next
2. Link prev to cur's next node next

In our first step, we need to find out `prev` and `next`. It is easy to find out `next` using the reference field of `cur`. However, we have to traverse the linked list from the head node to find out `prev` which will take `O(N)` time on average, where `N` is the length of the linked list. So the time complexity of deleting a node will be `O(N)`.

The space complexity is `O(1)` because we only need constant space to store our pointers.

If we want to delete the first node, we can simply _assign the next node to head_.
