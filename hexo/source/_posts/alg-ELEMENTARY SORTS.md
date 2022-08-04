---
title: coursera 2.1 ELEMENTARY SORTS
categories: [algorithms]
tags: [princeton-alg]
---

##  ‣ rules of the game

### Sample sort client 1

Goal. Sort any type of data.

Ex 1. Sort random real numbers in ascending order.

> seems artificial, but stay tuned for an application

```java
public class Experiment
{
    public static void main(String[] args)
    {
        int N = Integer.parseInt(args[0]); 
        Double[] a = new Double[N];
        for (int i = 0; i < N; i++)
            a[i] = StdRandom.uniform();
        Insertion.sort(a);
        for (int i = 0; i < N; i++)
            StdOut.println(a[i]):
    }
}
```

### Sample sort client 2

Ex 2. Sort strings from file in alphabetical order.

```java
public class StringSorter
{
    public static void main(String[] args)
    {
        String[] a = In.readStrings(args[0]); 
        Insertion.sort(a);
        for (int i = 0; i < a.length; i++)
            StdOut.println(a[i]);
    } 
}
```

### Sample sort client 3

Ex 3. Sort the files in a given directory by filename.

```java
import java.io.File;

public class FileSorter {
    public static void main(String[] args)
    {
        File directory = new File(args[0]);
        File[] files = directory.listFiles();
        Insertion.sort(files);
        for (int i = 0; i < files.length; i++)
            StdOut.println(files[i].getName());
    } 
}
```

### Callbacks

Q. How can sort() know how to compare data of type Double, String, and java.io.File without any information about the type of an item's key?

Callback = reference to executable code.

- Client passes array of objects to sort() function.
- The sort() function calls back object's compareTo() method as needed.

Implementing callbacks.

- Java: interfaces.
- C: function pointers.
- C++: class-type functors.
- C#: delegates.
- Python, Perl, ML, Javascript: first-class functions.

### Callbacks: roadmaps

![callback](/Users/mac/github/zennlyu.github.io/hexo/source/_posts/coursera-ELEMENTARY SORTS/callback.png)

### Total order

A total order is a binary relation $≤$ that satisfies:

- Antisymmetry: if $v≤w$ and $w≤v$, then $v=w$. 
- Transitivity: if $v≤w$ and $w≤x$, then $v≤x$. 
- Totality: either $v≤w$ or $w≤v$ or both.

#### Ex.

- Standard order for natural and real numbers.
- Chronological order for dates or times. 
- Alphabetical order for strings.
-  ...

#### Surprising but true.

The $<=$ operator for double is not a total order. (!)

> violates totality: (`Double.NaN <= Double.NaN`) is false

### Comparable API

#### Implement compareTo() so that v.compareTo(w)

- Is a total order.
- Returns a negative integer, zero, or positive integer
- if $v$ is less than, equal to, or greater than $w$, respectively. Throws an exception if incompatible types (or either is null).

#### Built-in comparable types. 

Integer, Double, String, Date, File, ... 

#### User-defined comparable types. 

Implement the Comparable interface.

### Implementing the Comparable interface

```java
public class Date implements Comparable<Date>
{
    private final int month, day, year;
    public Date(int m, int d, int y)
    {
        month = m;
        day   = d;
        year  = y;
    }
    public int compareTo(Date that)
    {
        if (this.year < that.year ) return -1;
        if (this.year > that.year ) return +1;
        if (this.month < that.month) return -1;
        if (this.month > that.month) return +1;
        if (this.day < that.day )   return -1;
        if (this.day > that.day )   return +1;
        return 0;
}
```

### Two useful sorting abstractions





##  ‣ selection sort

##  ‣ insertion sort

##  ‣ shellsort

##  ‣ shuffling

##  ‣ convex hull
