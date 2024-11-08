# Data structure and algorithms

- [Data structure and algorithms](#data-structure-and-algorithms)
  - [Hash tables](#hash-tables)
  - [Array](#array)
    - [In-place algorithm](#in-place-algorithm)
  - [Linked lists](#linked-lists)
    - [Inserting a node to the beginning of a singly linked list](#inserting-a-node-to-the-beginning-of-a-singly-linked-list)
    - [Deleting a node from a singly linked list](#deleting-a-node-from-a-singly-linked-list)
    - [Two-pointer in Linked List](#two-pointer-in-linked-list)
      - [1. Finding the middle of a linked list](#1-finding-the-middle-of-a-linked-list)
      - [2. Detecting a cycle in a linked list (floyd's cycle detection algorithm)](#2-detecting-a-cycle-in-a-linked-list-floyds-cycle-detection-algorithm)
  - [Stacks and Queues](#stacks-and-queues)

## Hash tables

Array and string questions are often interchangeable.

A _hash table_ is a data structure that maps keys to values for highly efficient lookup. To insert a new key and value, we do the following:

1. Computed the key's hash code, which will usually be an int or long. Note that two different keys could have the same hash code, as there may be an infinite number of keys and a finite number of ints. This is [known as Hash collision](https://en.wikipedia.org/wiki/Hash_collision),
2. Map the hash code to an index in the array.
3. At this index, there is a linked list of keys and values. Store the key and value in this index. We must use a linked list because of collisions: you could have two different keys with the same hash code, or two different hash codes that map to the same index.

If the number of collisions is very high the worst case runtime is _O(N)_, where _N_ is the number of keys. However, we generally assume a good implementation that keeps collisions to a minimum, in which case the lookup time is _O(1)_.

Alternatively, we can implement the hash table with a balanced binary search tree. This gives us an _O(log N)_ lookup time. The advantage of this is potentially using less space, since we no longer allocate a large array. We can also iterate through the keys in order, which can be useful sometimes.

## Array

In Swift, an `Array` is an ordered collection which offers random access. It also offers sequential access to its element since it conforms to the `Sequence` protocol.

### In-place algorithm

> In computer science, an in-place algorithm is an algorithm that operates directly on the input data structure without requiring extra space proportional to the input size. In other words, it modifies the input in place, without creating a separate copy of the data structure. An algorithm which is not in-place is sometimes called not-in-place or out-of-place. _[Source: Wikipedia](https://en.wikipedia.org/wiki/In-place_algorithm)_

In Swift, this is kinda similar to using an `inout` parameter. In other words, have to modify the input directly instead of copying the whole input, i.e., allocating more memory. For example, given an array of _n_ elements, to reverse it in-place, can swap the elements using the indices:

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

It is used in various algorithms such as bubble sort, comb sort, selection sort, insertion sort, heap sort, and Shell sort. These algorithms require only a few pointers, so their space complexity is `O(log n)`.

Swift uses [Timsort](https://en.wikipedia.org/wiki/Timsort) which is a hybrid, stable sorting algorithm, derived from merge sort and insertion sort, designed to perform well on many kinds of real-world data, refer to _[Sort.swift](https://github.com/swiftlang/swift/blob/main/stdlib/public/core/Sort.swift)_ file. It was implemented by Tim Peters in 2002 for use in the Python programming language. The algorithm finds subsequences of the data that are already ordered (runs) and uses them to sort the remainder more efficiently. This is done by merging runs until certain criteria are fulfilled.

## Linked lists

A linked list is a data structure that represents a sequence of nodes, in a singly linked list, each node points to the next node in the linked list. A doubly linked list gives each node pointers to both the next node and the previous node. When dealing with a coding challenge, need to understand if we're dealing with a singly linked list or a doubly linked list.

Unlike an array, a linked list does not provide constant time access to a particular "index" within the list. This means that if you'd like to find the `K`th element in the list, you will need to iterate through `K` elements.

The _benefit_ of a linked list is that you can _add/insert_ and _remove/delete_ items from the beginning of the list in constant time.

If we want to add a new value after a given node `prev`, we should:

1. Initialize a new node `cur` with the given value
2. Link the "next" field of `cur` to `prev`'s next node next
3. Link the "next" field in `prev` to `cur`

Unlike an array, we don’t need to move all elements past the inserted element. Therefore, you can insert a new node into a linked list in `O(1)` time complexity if you have a reference to `prev`, which is very efficient.

### Inserting a node to the beginning of a singly linked list

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

### Two-pointer in Linked List

The two-pointer technique in a linked list is effective for:

- **Finding the middle of a linked list**: A slow pointer and a fast pointer help locate the middle by advancing at different rates.
- **Detecting cycles**: The fast pointer moving faster will catch up to the slow pointer if a cycle exists.

#### 1. Finding the middle of a linked list

To find the middle of a linked list, you can use two pointers: a **slow pointer** and a **fast pointer**.

- The **slow pointer** moves one node at a time.
- The **fast pointer** moves two nodes at a time.

When the fast pointer reaches the end of the linked list (or cannot proceed further), the slow pointer will be at the middle of the list.

**Example:**

```swift
func findMiddle(_ head: ListNode?) -> ListNode? {
    var slow = head
    var fast = head

    while fast != nil && fast?.next != nil {
        slow = slow?.next
        fast = fast?.next?.next
    }

    return slow
}
```

- In this example, `slow` will point to the middle node when `fast` reaches the end.
- If the list has an even number of elements, `slow` will be at the second of the two middle nodes.

#### 2. Detecting a cycle in a linked list (floyd's cycle detection algorithm)

To detect a cycle in a linked list, you can use the **slow and fast pointer** approach, also known as [Floyd's Cycle Detection Algorithm](https://en.wikipedia.org/wiki/Cycle_detection#Floyd's_tortoise_and_hare).

- Again, you use two pointers: the slow pointer moves one step at a time, while the fast pointer moves two steps at a time.
- If there is a cycle, eventually, the fast pointer will "lap" the slow pointer, and they will meet at some point within the cycle.
- If there is no cycle, the fast pointer will reach the end of the list (`nil`).

**Example:**

```swift
func hasCycle(_ head: ListNode?) -> Bool {
    var slow = head
    var fast = head

    while fast != nil && fast?.next != nil {
        slow = slow?.next
        fast = fast?.next?.next

        if slow === fast {
            return true
        }
    }

    return false
}
```

- If `slow` and `fast` meet (`slow === fast`), then a cycle exists.
- If `fast` reaches `nil`, it means the linked list has no cycle.

These approaches improve the efficiency of common linked list operations, often reducing time complexity from `O(n^2)` to `O(n)` and allowing solutions to run in a single traversal.

## Stacks and Queues

The stack data structure is precisely what it sounds like: a stack of data. A stack used LIFO (last-in first-out) ordering. That is, as in a stack of dinner plates, the most recent item added to the stack is the first item to be removed.

A stack can be easily implemented either through an array or a linked list, as it is merely a special case of a list. In either case, what identifies the data structure as a stack is not the implementation but the interface: the user is only allowed to pop or push items onto the array or linked list, with few other helper operations.

A queue implements FIFO (first-in first-out) ordering. As in a line or queue at a ticket stand, items are removed from the data structure in the same order that they are added. A queue can be implemented with a linked list. In fact, they are essentially the same thing, as long as items are added and removed from opposite sides.

A queue, as a _front_ (that is, the element at index zero) and a _back_ which is the index of the last element.

It's possible to implement a queue using an Array. However, in an array-based queue, dequeuing is an `O(n)` operation because as the first element gets removed, all the other elements need to be shifted. A more efficient implementation is using a doubly linked list.

```markdown
+----+------+    +----+------+    +----+------+
| P  |  N   |<-->| P  |  N   |<-->| P  |  N   |
+----+------+    +----+------+    +----+------+
| 1  |  o---|--->| 2  |  o---|--->| 3  |  o   |
+----+------+    +----+------+    +----+------+

Key:
- P: Pointer to the previous node.
- N: Pointer to the next node.
- o: Data stored in the node (in this case, values 1, 2, 3).
- The arrows indicate the direction of the pointers between nodes.
```

Each node in the doubly linked list contains:

- A value (data) stored in the node.
- A pointer to the next node (`N`).
- A pointer to the previous node (`P`).
- This allows traversal in both directions: from the head to the tail and from the tail to the head.
