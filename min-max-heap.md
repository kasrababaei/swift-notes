# Min-max heap

In computer science, a [heap](https://en.wikipedia.org/wiki/Heap_(data_structure)) is a tree-based data structure that satisfies the heap property: In a max heap, for any given node C, if P is a parent node of C, then the key (the value) of P is greater than or equal to the key of C. In a min heap, the key of P is less than or equal to the key of C. The node at the "top" of the heap (with no parents) is called the root node.

A min-max heap is a complete binary tree containing alternating min (or even) and max (or odd) levels. Even levels are for example 0, 2, 4, etc, and odd levels are respectively 1, 3, 5, etc. We assume in the next points that the root element is at the first level, i.e., 0.

Each node in a min-max heap has a data member (usually called key) whose value is used to determine the order of the node in the min-max heap.

The root element is the smallest element in the min-max heap.

One of the two elements in the second level, which is a max (or odd) level, is the greatest element in the min-max heap.

Let *x* be any node in a min-max heap.

- If *x* is on a min (or even) level, then *x.key* is the minimum key among all keys in the subtree with root *x*.
- If *x* is on a max (or odd) level, then *x.key* is the maximum key among all keys in the subtree with root *x*.

A node on a min (max) level is called a min (max) node.

A **max-min heap** is defined analogously; in such a heap, the maximum value is stored at the root, and the smallest value is stored at one of the root's children.

![Example of Min-max heap](https://upload.wikimedia.org/wikipedia/commons/5/50/Min-max_heap.jpg)
