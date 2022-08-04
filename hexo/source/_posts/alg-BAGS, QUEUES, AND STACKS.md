---
title: coursera 1.3 BAGS, QUEUES, AND STACKS
categories: [algorithms]
tags: [princeton-alg]
---

# Stacks and queues

![sq](sq.png)

Fundamental data types.

| DATA STRUCTURE | ORDER |
| -------------- | ----- |
| stack          | LIFO  |
| queue          | FIFO  |

-  Value: collection of objects.
-  Operations: insert, remove, iterate, test if empty. 
-  Intent is clear when we insert.
-  Which item do we remove?

## Client, implementation, interface

### Benefits.

| DATA STRUCTURE | ORDER                                          |
| -------------- | ---------------------------------------------- |
| client         | program using operations defined in interface. |
| implementation | actual code implementing operations.           |
| interface      | description of data type, basic operations.    |

-  Client can't know details of implementation ⇒

   client has many implementation from which to choose.

-  Implementation can't know details of client needs ⇒

   many clients can re-use the same implementation.

-  **Design:** creates modular, reusable libraries.

-  **Performance:** use optimized implementation where it matters.

##  ‣ stacks

### Warmup API. Stack of strings data type.

![warmapi](warmapi.png)

### Warmup client. Reverse sequence of strings from standard input

```java
public static void main(String[] args)
{
    StackOfStrings stack = new StackOfStrings();
    while (!StdIn.isEmpty()) {
        String s = StdIn.readString();
        if (s.equals("-"))  StdOut.print(stack.pop()); 
        else                stack.push(s);
    }
}
```

### Stack: linked-list representation

```java
public class LinkedStackOfStrings
{
    private Node first = null;
    private class Node {
        // private inner class (access modifiers don't matter)
        String item;
        Node next; 
    }
    public boolean isEmpty()
    {  return first == null;  }

    public void push(String item)
    {
        Node oldfirst = first;
        first = new Node();
        first.item = item;
        first.next = oldfirst;
    }

    public String pop()
    {
        String item = first.item;
        first = first.next;
        return item;
    }
}
```

### Stack: linked-list implementation performance

**Proposition.** Every operation takes constant time in the worst case.

**Proposition.** A stack with *N* items uses $\sim40N$ bytes.

![bytes](bytes.png)

**Remark.** This accounts for the memory for the stack (but not the memory for strings themselves, which the client owns).

### Stack: array implementation

#### Array implementation of a stack.

![array](array.png)

-  Use array `s[]` to store `N` items on stack. 
-  `push()`: add new item at `s[N]`.
-  `pop()`: remove item from `s[N-1]`.

#### Defect. 

Stack overflows when `N` exceeds capacity. [stay tuned]

```java
public class FixedCapacityStackOfStrings
{
    private String[] s;
    private int N = 0;
    
    public FixedCapacityStackOfStrings(int capacity // a cheat (stay tuned))
    {  s = new String[capacity];  }

    public boolean isEmpty()
    {  return N == 0;  }

    public void push(String item)
    {  s[N++] = item; // use to index into array; then increment N }

    public String pop()
    { return s[--N]; // decrement N; then use to index into array } 
}
```

### Stack considerations

#### Overflow and underflow.

| Cases     | Consideration                                             |
| --------- | --------------------------------------------------------- |
| Underflow | throw exception if pop from an empty stack                |
| Overflow  | use resizing array for array implementation. [stay tuned] |

#### Null items: We allow null items to be inserted.

#### Loitering: Holding a reference to an object when it is no longer needed.

```java
public String pop() { return s[--N]; }	// Loitering.

// this version avoids "loitering": 
// garbage collector can reclaim memory only if no outstanding references
public String pop()
{	String item = s[--N]; s[N] = null;	return item;	}
```

##  ‣ resizing arrays

### Stack: resizing-array implementation

#### Q. How to grow and shrink array?

**Problem.** Requiring client to provide capacity does not implement API! 

##### First try.

- `push()`: increase size of array `s[]` by 1. 
- `pop()`:  decrease size of array `s[]` by 1.

##### Too expensive.

- Need to copy all items to a new array.

- Inserting first `N` items takes time proportional to $1+2+...+N\sim N^2/2$.

##### Challenge: Ensure that array resizing happens infrequently.

#### Q. How to grow array?

##### **A.** 

If array is full, create a new array of **twice**<u>("repeated doubling")</u> the size, and copy items.

```java
public ResizingArrayStackOfStrings()
{  s = new String[1];  }

public void push(String item)
{
    if (N == s.length) resize(2 * s.length);
    s[N++] = item;
}

private void resize(int capacity)
{
    String[] copy = new String[capacity];
    for (int i = 0; i < N; i++)
        copy[i] = s[i];
    s = copy;
}
```

##### Consequence. 

Cost of inserting first N items: Inserting first N items takes time proportional to N (not N 2 ).

$\begin{align} N_{1\ array\ access/push}+(2+4+8+...+N)_{k\ array\ accesses\ to\ double\ to\ size\ k,(ignoring\ cost\ to\ create\ new\ array)}~3N \end{align}$

![push](push.png)

#### Q. How to shrink array?

##### First try：Too expensive in worst case.

|          | Operation                                              |
| -------- | ------------------------------------------------------ |
| `push()` | double size of array `s[]` when array is full.         |
| `pop()`  | halve size of array `s[]` when array is one-half full. |

- Consider `push-pop-push-pop-...` sequence when array is full. 
- Each operation takes time proportional to $N$.

##### Efficient try.

|          | Operation                                                 |
| -------- | --------------------------------------------------------- |
| `push()` | double size of array `s[]` when array is full.            |
| `pop()`  | halve size of array `s[]` when array is one-quarter full. |

```java
public String pop()
{
    String item = s[--N];
    s[N] = null;
    if (N > 0 && N == s.length/4) resize(s.length/2);
    return item;
}
```

> **Invariant**: Array is between 25% and 100% full.

![trace](trace.png)

### performance

#### Amortized analysis. 

Average running time per operation over a worst-case sequence of operations.

#### Proposition.

Starting from an empty stack, any sequence of $M$ push and pop operations takes time proportional to $M$.

![resize](resize.png)

### memory usage

#### Proposition. 

Uses between ~ $8\ N$ and ~ $32\ N$ bytes to represent a stack with $N$ items.

- ~ 8 $N$ when full.
- ~ 32 $N$ when one-quarter full.

![one-quarter](one-quarter.png)

> This accounts for the memory for the stack (but not the memory for strings themselves, which the client owns).

#### resizing array vs. linked list

##### Tradeoffs.

Can implement a stack with either resizing array or linked list; save a link to the list; client can use interchangeably. Which one is better?

##### Linked-list implementation.

- Every operation takes constant time in the worst case.
- Uses extra time and space to deal with the links.

##### Resizing-array implementation.

- Every operation takes constant amortized time.
- Less wasted space.

![linked-list](linked-list.png)

##  ‣ queues

![queue](/Users/mac/github/zennlyu.github.io/hexo/source/_posts/coursera-BAGS, QUEUES, AND STACKS/queue.png)

linked-list representation

##  ‣ generics

Parameterized stack

##  ‣ iterators

### Design challenge

> Support iteration over stack items by client, without revealing the internal representation of the stack.

**Java solution**: Make stack implement the java.lang.Iterable interface.

### Iterators

- What is an Iterable?  -- Has a method that returns an Iterator.

```java
 public interface Iterable<Item>
{
   Iterator<Item> iterator();
}
```

- What is an Iterator? -- Has methods hasNext() and next().

```java
public interface Iterator<Item>
{
   boolean hasNext();
   Item next();
   void remove(); // optional; use at your own risk
}
```

- Why make data structures Iterable? -- Java supports elegant client code.

```java
for (String s : stack) StdOut.println(s); // shorthand
||
// longhand
Iterator<String> i = stack.iterator(); while (i.hasNext())
{
  String s = i.next();
  StdOut.println(s);
}
```

### Stack iterator: linked-list implementation

### Stack iterator: array implementation

### Bag API

![bag](/Users/mac/github/zennlyu.github.io/hexo/source/_posts/coursera-BAGS, QUEUES, AND STACKS/bag.png)

**Main application.** Adding items to a collection and iterating (when order doesn't matter).

**Implementation.** Stack (without pop) or queue (without dequeue).

##  ‣ applications

### Java collections library

#### List interface. 

`java.util.List` is API for an sequence of items.

#### Implementations. 

> caveat: only some operations are efficient

- `java.util.ArrayList` uses resizing array;
- `java.util.LinkedList` uses linked list.

#### `java.util.Stack`

- Supports `push()`, `pop()`, and and iteration.
- Extends `java.util.Vector`, which implements `java.util.List` interface from previous slide, including, `get()` and `remove()`. 
- Bloated and poorly-designed API (why?)

`java.util.Queue`： An interface, not an implementation of a queue. 

**Best practices**：Use our implementations of Stack, Queue, and Bag.

#### War story (from Assignment 1)

##### Generate random open sites in an *N*-by-*N* percolation system.

- Jenny: pick`(i,j)` at random; if already open, repeat. | Takes ~$c_1N^2$ seconds.

- Kenny: create a `java.util.ArrayList` of $N^2$ closed sites. Pick an index at random and delete. | Takes ~$c_2N^4$ seconds.

> Lesson. Don't use a library until you understand its API!
>
> This course. Can't use a library until we've implemented it in class.

#### Stack applications

-  Parsing in a compiler.
-  Java virtual machine.
-  Undo in a word processor.
-  Back button in a Web browser.
-  PostScript language for printers.
-  Implementing function calls in a compiler.

#### Function Calls

How a compiler implements a function.

> Recursive function: Function that calls itself.
>
> Note: Can always use an explicit stack to remove recursion

- Function call: push local environment and return address.
- Return: pop return address and local environment.
