---
title: coursera 1.5 UNION_FIND
categories: [algorithms]
tags: [princeton-alg]
---

##  ‣ DYNAMIC CONNECTIVITY

### Given a set of N objects.

- Union command: connect two objects.
- Find/connected query: is there a path connecting the two objects?

### Modeling the objects

Applications involve manipulating objects of all types.

- Pixels in a digital photo.
- Computers in a network.
- Friends in a social network. 
- Transistors in a computer chip. 
- Elements in a mathematical set. 
- Variable names in Fortran program. 
- Metallic sites in a composite system.

When programming, convenient to name objects 0 to N –1.

- Use integers as array index.

- Suppress details not relevant to union-find.

  > (can use ST to translate from site names to integers: stay tuned (Chapter 3))

### Modeling the connections

We assume "is connected to" is an equivalence relation:

- **Reflexive:** *p* is connected to *p*.
- **Symmetric:** if *p* is connected to *q*, then *q* is connected to *p*. 
- **Transitive:** if *p* is connected to *q* and *q* is connected to *r*, then *p* is connected to *r*.

### Implementing the operations

- **Find query.** Check if two objects are in the same component.
- **Union command.** Replace components containing two objects with their union.

### Union-find data type (API)

**Goal.** Design efficient data structure for union-find.

-  Number of objects *N* can be huge.
-  Number of operations *M* can be huge.
-  Find queries and union commands may be intermixed.

![UF](UF.png)

### Dynamic-connectivity client

- ・Read in number of objects *N* from standard input. 
- ・Repeat:
  - –  read in pair of integers from standard input
  - –  if they are not yet connected, connect them and print out pair

```java
public static void main(String[] args)
{
	int N = StdIn.readInt(); 
	UF uf = new UF(N);
	while (!StdIn.isEmpty()) {
		int p = StdIn.readInt(); 
		int q = StdIn.readInt(); 
		if (!uf.connected(p, q)) {
			uf.union(p, q);
			StdOut.println(p + " " + q); 
		}
	} 
}
```



##  ‣ QUICK-FIND

### Eager approach

```java
public class QuickFindUF
{
    private int[] id;

    public QuickFindUF(int N)
    {
        id = new int[N];
        for (int i = 0; i < N; i++) {
            id[i] = i;
        }
    }

    public boolean connected(int p, int q)
    {   return id[p] == id[q];  }

    public void union(int p, int q)
    {
        int pid = id[p];
        int qid = id[q];
        for (int i = 0; i < id.length; i++) {
            if (id[i] == pid)
                id[i] = qid;
    }
}
```

#### Data Structure

- Integer array `id[]` of length N.
- Interpretation: `p` and `q` are connected `iff` they have the same id.

#### Find.

Check if p and q have the same id.

#### Union. 

To merge components containing `p` and `q`, change all entries whose id equals `id[p]` to `id[q]`.

### Quick-Find is too slow

**Cost model.** Number of array accesses (for read or write).

**order of growth of number of array accesses**:

| Algorithms | Initialize | Union | Find |
| ---------- | ---------- | ----- | ---- |
| Quick-find | N          | N     | 1    |

**Union is too expensive** :It takes $N^2$ array accesses to process a sequence of $N$ union commands on $N$ objects.

#### Quadratic algorithms do not scale



##  ‣ QUICK-UNION

### Lazy approach

```java
public class QuickUnionUF
{
    private int[] id;

    public QuickUnionUF(int N)
    {		// set id of each object to itself (N array accesses)
        id = new int[N];
        for (int i = 0; i < N; i++) id[i] = i;
    }

    private int root(int i)
    {		// chase parent pointers until reach root (depth of i array accesses)
        while (i != id[i])  i = id[i];
        return i;
    }

    public boolean connected(int p, int q)
    // check if p and q have same root (depth of p and q array accesses)
    {   return root[p] == root[q];  }

    public void union(int p, int q)
    {		// change root of p to point to root of q (depth of p and q array accesses)
        int i = root(p);
        int j = root(q);
        id[i] = j;
    }
}
```

#### Data structure

- Integer array `id[]` of length `N`. 
- Interpretation: `id[i]` is parent of `i`. 
- Root of i is `id[id[id[...id[i]...]]]`(keep going until it doesn’t change (algorithm ensures no cycles))

#### **Find.** 

Check if p and q have the same root.

#### **Union.** 

To merge components containing p and q, set the id of p's root to the id of q's root.

### Quick-union is also too slow

Cost model. Number of array accesses (for read or write).

| Algorithms              | Initialize | Union                                     | Find |
| ----------------------- | ---------- | ----------------------------------------- | ---- |
| Quick-find              | N          | N                                         | 1    |
| Quick-union(worst case) | N          | $N^†$($†$ includes cost of finding roots) | N    |

#### Quick-find defect.

- Union too expensive ( $N$ array accesses).
- Trees are flat, but too expensive to keep them flat.

#### Quick-union defect.

- Trees can get tall.
- Find too expensive (could be *N* array accesses).

##  ‣ IMPROVEMENT

### 1 Weighted quick-union.

- Modify quick-union to avoid tall trees.

- Keep track of size of each tree (number of objects).

- Balance by linking root of smaller tree to root of larger tree.

  > (reasonable alternatives: union by height or "rank")

![weight](weight.png)



#### Quick-union and weighted quick-union example

![quick-union](quick-union.png)

#### Weighted quick-union: Java implementation

##### Data structure.

Same as quick-union, but maintain extra array `sz[i]` to count number of objects in the tree rooted at `i`.

##### Find. 

Identical to quick-union. 

```java
return root(p) == root(q);
```

##### Union. 

Modify quick-union to:

- Link root of smaller tree to root of larger tree. 
- Update the `sz[]` array.

```java
int i = root(p);
int j = root(q);
if (i == j) return;
if (sz[i] < sz[j])  {   id[i] = j; sz[j] += sz[i];    }
else                {   id[j] = i; sz[i] += sz[j];    }
```

#### Weighted quick-union analysis

##### Running time.

- Find: takes time proportional to depth of *p* and *q*. 
- Union: takes constant time, given roots.

##### Proposition. 

($lg=base-2$ logarithm) Depth of any node $x$ is at most $lgN$

![log](log.png)

##### Pf. When does depth of *x* increase?

Increases by $1$ when tree $T_1$ containing $x$ is merged into another tree $T_2$.

- The size of the tree containing $x$ at least doubles since $|T_2|≥|T_1|$. 

- Size of tree containing $x$ can double at most $lgN$ times. Why?

  ![log](log.png)

| Algorithms              | Initialize | Union                                     | Find  |
| ----------------------- | ---------- | ----------------------------------------- | ----- |
| Quick-find              | N          | N                                         | 1     |
| Quick-union(worst case) | N          | $N^†$($†$ includes cost of finding roots) | N     |
| weighted QU             | N          | $lgN^†$                                   | $lgN$ |

Q. Stop at guaranteed acceptable performance? 

A. No, easy to improve further.

### Improvement 2: path compression

#### Quick union with path compression. 

Just after computing the root of *p*, set the id of each examined node to point to that root.

![id](id.png)

#### Path compression: Java implementation

##### Two-pass implementation: 

add second loop to root() to set the id[] of each examined node to the root.

##### Simpler one-pass variant: 

Make every other node in path point to its grandparent (thereby halving path length).

```java
private int root(int i)
{
    while (i != id[i]) 
    {
        id[i] = id[id[i]];  // only one extra line of code !
        i = id[i];
    }
    return i; 
}
```

##### In practice.

No reason not to! Keeps tree almost completely flat.

#### Weighted quick-union with path compression: amortized analysis

##### Proposition. |[Hopcroft-Ulman, Tarjan]

Starting from an empty data structure, any sequence of $M$ union-find ops on $N$ objects makes  $≤c\;(N+M\;lg*N)$ array accesses.

- Analysis can be improved to $N+M\;α(M,N)$. 
- Simple algorithm with fascinating mathematics.

**iterate log function**:

| N           | lg* N |
| ----------- | ----- |
| 1           | 0     |
| 2           | 1     |
| 4           | 2     |
| 16          | 3     |
| 65536       | 4     |
| $2^{65536}$ | 5     |



##### Linear-time algorithm for *M* union-find ops on *N* objects?

- Cost within constant factor of reading in the data.
- In theory, WQUPC is not quite linear.
- In practice, WQUPC is linear.

##### Amazing fact. |[Fredman-Saks]

> in "cell-probe" model of computation

No linear-time algorithm exists



#### Summary

##### Bottom line.

Weighted quick union (with path compression) makes it possible to solve problems that could not otherwise be addressed.

##### Ex. | [109 unions and finds with 109 objects]

- WQUPC reduces time from 30 years to 6 seconds.
- Supercomputer won't help much; good algorithm enables solution.

##  ‣ APPLICATIONS

### Union-find applications

- Percolation.
- Games (Go, Hex).
- Dynamic connectivity.  ✓ 
- Least common ancestor.
- Equivalence of finite state automata. 
- Hoshen-Kopelman algorithm in physics. 
- Hinley-Milner polymorphic type inference. 
- Kruskal's minimum spanning tree algorithm. 
- Compiling equivalence statements in Fortran. 
- Morphological attribute openings and closings. 
- Matlab's bwlabel() function in image processing.

### Percolation

#### A model for many physical systems:

![percolation](percolation.png)

- *N*-by-*N* grid of sites.
- Each site is open with probability $p$ (or blocked with probability $1–p$). 
- System percolates `iff` top and bottom are connected by open sites.

#### Likelihood of percolation

> Depends on site vacancy probability *p*.

![perrcolations](perrcolations.png)

#### Percolation phase transition

When $N$ is large, theory guarantees a sharp threshold $p^*$.

- $p>p^*$: almost certainly percolates.
- $p<p^*$: almost certainly does not percolate.

Q. What is the value of $p^*$ ?

A. About 0.592746 for large square lattices.

> constant known only via simulation

![p*](p*.png)

#### Monte Carlo simulation

- Initialize $N-by-N$ whole grid to be blocked.
- Declare random sites open until top connected to bottom. 
- Vacancy percentage estimates $p^*$.

![opensite](opensite.png)

#### Dynamic connectivity solution to estimate percolation threshold

##### Q. How to check whether an *N*-by-*N* system percolates?

##### Q. How to check whether an *N*-by-*N* system percolates?

- Create an object for each site and name them $0$ to $N^2–1$.

- Sites are in same component if connected by open sites.

- Percolates iff any site on bottom row is connected to site on top row.

  > (brute-force algorithm: N 2 calls to connected())

![top-down](top-down.png)

##### Clever trick. 

- Introduce 2 virtual sites (and connections to top and bottom).

- Percolates iff virtual top site is connected to virtual bottom site.

  > (efficient algorithm: only 1 call to connected())

![per](per.png)

##### Q. How to model opening a new site?

A. Mark new site as open; connect it to all of its adjacent open sites.

> (up to 4 calls to union())

![perco](perco.png)



### Percolation Exercise

#### Java Environment

```java
import edu.princeton.cs.algs4.StdRandom;
import edu.princeton.cs.algs4.StdStats;
import edu.princeton.cs.algs4.WeightedQuickUnionUF;
```

Note that *your* code must be in the *default package*; if you use a `package` statement, the autograder will reject your submission.

#### Percolation. 

Given a composite systems comprised of randomly distributed insulating and metallic materials: what fraction of the materials need to be metallic so that the composite system is an electrical conductor? 

Given a porous landscape with water on the surface (or oil below), under what conditions will the water be able to drain through to the bottom (or the oil to gush through to the surface)? 

Scientists have defined an abstract process known as *percolation* to model such situations.

#### The model.

We model a percolation system using an *n*-by-*n* grid of *sites*.

Each site is either *open* or *blocked*. 

A *full* site is an open site that can be connected to an open site in the top row via a chain of neighboring (left, right, up, down) open sites. We say the system *percolates* if there is a full site in the bottom row. 

In other words, a system percolates if we fill all open sites connected to the top row and that process fills some open site on the bottom row. (For the insulating/metallic materials example, the open sites correspond to metallic materials, so that a system that percolates has a metallic path from top to bottom, with full sites conducting. 

For the porous substance example, the open sites correspond to empty space through which water might flow, so that a system that percolates lets water fill open sites, flowing from top to bottom.)

#### The problem.

In a famous scientific problem, researchers are interested in the following question: 

if sites are independently set to be open with probability $p$ (and therefore blocked with probability $1−p$), what is the probability that the system percolates? When $p$ equals 0, the system does not percolate; when $p$ equals 1, the system percolates. 

The plots below show the site vacancy probability $p$ versus the percolation probability for 20-by-20 random grid (left) and 100-by-100 random grid (right).

![site-vacancy](site-vacancy.png)

When *n* is sufficiently large, there is a *threshold* value $p^*$ such that when $p<p^*$ a random *n*-by-*n* grid almost never percolates, and when $p>p^*$, a random *n*-by-*n* grid almost always percolates.

No mathematical solution for determining the percolation threshold $p^*$ has yet been derived. Your task is to write a computer program to estimate $p^*$.

#### Percolation data type.

To model a percolation system, create a data type `Percolation` with the following API:

```java
public class Percolation {

    // creates n-by-n grid, with all sites initially blocked
    public Percolation(int n)

    // opens the site (row, col) if it is not open already
    public void open(int row, int col)

    // is the site (row, col) open?
    public boolean isOpen(int row, int col)

    // is the site (row, col) full?
    public boolean isFull(int row, int col)

    // returns the number of open sites
    public int numberOfOpenSites()

    // does the system percolate?
    public boolean percolates()

    // test client (optional)
    public static void main(String[] args)
}
```

##### Corner cases.

By convention, the row and column indices are integers between 1 and $n$, where $(1,1)$ is the upper-left site: Throw an `IllegalArgumentException` if any argument to `open()`, `isOpen()`, or `isFull()` is outside its prescribed range. 

Throw an `IllegalArgumentException` in the constructor if $n≤0$.

##### Performance requirements.

The constructor must take time proportional to *n*2; all instance methods must take constant time plus a constant number of calls to `union()` and `find()`.

#### Monte Carlo simulation.

To estimate the percolation threshold, consider the following computational experiment:

- Initialize all sites to be blocked.
- Repeat the following until the system percolates:
- Choose a site uniformly at random among all blocked sites.
- Open the site.
- The fraction of sites that are opened when the system percolates provides an estimate of the percolation threshold.

For example, if sites are opened in a 20-by-20 lattice according to the snapshots below, then our estimate of the percolation threshold is 204/400 = 0.51 because the system percolates when the 204th site is opened.

![opensites](opensites.png)

By repeating this computation experiment *T* times and averaging the results, we obtain a more accurate estimate of the percolation threshold. Let *xt* be the fraction of open sites in computational experiment *t*. The sample mean x⎯⎯⎯x¯ provides an estimate of the percolation threshold; the sample standard deviation *s*; measures the sharpness of the threshold.

$$\begin{align} \overline{x}=\frac{x_1+x_2+...+x_T}{T}, \tag{1}s^2=\frac{(x_1-\overline{x})^2+(x_2-\overline{x})^2+...+(x_T-\overline{x})^2}{T-1} \end{align}$$

Assuming *T* is sufficiently large (say, at least 30), the following provides a 95% confidence interval for the percolation threshold:

$$\begin{align} \overline{x}-\frac{1.96s}{\sqrt{T}}, \tag{2}\overline{x}+\frac{1.96s}{\sqrt{T}}, \end{align}$$

To perform a series of computational experiments, create a data type `PercolationStats` with the following API.

```java
public class PercolationStats {

    // perform independent trials on an n-by-n grid
    public PercolationStats(int n, int trials)

    // sample mean of percolation threshold
    public double mean()

    // sample standard deviation of percolation threshold
    public double stddev()

    // low endpoint of 95% confidence interval
    public double confidenceLo()

    // high endpoint of 95% confidence interval
    public double confidenceHi()

   // test client (see below)
   public static void main(String[] args)
}
```

Throw an `IllegalArgumentException` in the constructor if either *n* ≤ 0 or *trials* ≤ 0.

Also, include a `main()` method that takes two *command-line arguments* *n* and *T*, performs *T* independent computational experiments (discussed above) on an *n*-by-*n* grid, and prints the sample mean, sample standard deviation, and the **95% confidence interval** for the percolation threshold. Use [`StdRandom`](https://algs4.cs.princeton.edu/code/javadoc/edu/princeton/cs/algs4/StdRandom.html) to generate random numbers; use [`StdStats`](https://algs4.cs.princeton.edu/code/javadoc/edu/princeton/cs/algs4/StdStats.html) to compute the sample mean and sample standard deviation.

```shell
~/Desktop/percolation> java-algs4 PercolationStats 200 100
mean                    = 0.5929934999999997
stddev                  = 0.00876990421552567
95% confidence interval = [0.5912745987737567, 0.5947124012262428]

~/Desktop/percolation> java-algs4 PercolationStats 200 100
mean                    = 0.592877
stddev                  = 0.009990523717073799
95% confidence interval = [0.5909188573514536, 0.5948351426485464]

~/Desktop/percolation> java-algs4 PercolationStats 2 10000
mean                    = 0.666925
stddev                  = 0.11776536521033558
95% confidence interval = [0.6646167988418774, 0.6692332011581226]

~/Desktop/percolation> java-algs4 PercolationStats 2 100000
mean                    = 0.6669475
stddev                  = 0.11775205263262094
95% confidence interval = [0.666217665216461, 0.6676773347835391]
```

#### Analysis of running time and memory usage (optional and not graded). 

> Implement the Percolation data type using the quick find algorithm in QuickFindUF.

Use Stopwatch to measure the total running time of PercolationStats for various values of n and T. 

- How does doubling n affect the total running time? 
- How does doubling T affect the total running time? 

- Give a formula (using tilde notation) of the total running time on your computer (in seconds) as a single function of both n and T.
- Using the 64-bit memory-cost model from lecture, give the total memory usage in bytes (using tilde notation) that a Percolation object uses to model an n-by-n percolation system.
- Count all memory that is used, including memory for the union–find data structure.

Now, implement the Percolation data type using the weighted quick union algorithm in WeightedQuickUnionUF. Answer the questions in the previous paragraph.
