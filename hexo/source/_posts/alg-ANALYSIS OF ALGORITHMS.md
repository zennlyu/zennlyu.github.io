---
title: coursera 1.4 ANALYSIS OF ALGORITHMS
categories: [algorithms]
tags: [princeton-alg]
---

## ‣ Introduction

### Stake holder

- Programmer
- Client
- Theoretician 
- Blocking and tackling 

### Successes

##### FFT algorithm: Discrete Fourier transform.

- Break down waveform of N samples into periodic components. 
- Applications: DVD, JPEG, MRI, astrophysics, ....
- Brute force: N<sup>2</sup> steps.
- FFT algorithm: `NlogN` steps, enables new technology.

##### Barnes-Hut algorithm: N-Body Stimulation

- Simulate gravitational interactions among N bodies. 
- Brute force: N<sup>2</sup> steps.
- Barnes-Hut algorithm: `NlogN` steps, enables new research

### Scientific method

> A framework for predicting performance and comparing algorithms.

- **Observe** some feature of the natural world.
- **Hypothesize** a model that is consistent with the observations. Predict events using the hypothesis.
- **Verify** the predictions by making further observations.
- **Validate** by repeating until the hypothesis and observations agree.

### Principles

- Experiments must be reproducible.
- Hypotheses must be falsifiable.

### Example 3-SUM : Brute-force Algorithm

Given *N* distinct integers, how many triples sum to exactly zero ---- Deeply related to problems in computational geometry.

```java
public class ThreeSum {
	public static int count(int[] a) {
		int N = a.length;
		int count = 0;
		for (int i=0; i<N; i++) 
			for (int j = i+1; j < N; j++) 
				for (int k = j+1; k < N; k++) // check each triple
					if (a[i] + a[j] + a[k] == 0) // for simplicity, ignore integer overflow
						count++;
		return count;
	}

	public static void main(String[] args) {
		int[] a = In.readInts(args[0]);
		StdOut.println(count(a));
	}
}
```

### Messure Running Time

```java
public static void main(String[] args)
{
	int[] a = In.readInts(args[0]);
	Stopwatch stopwatch = new Stopwatch();
	StdOut.println(ThreeSum.count(a)); 
	double time = stopwatch.elapsedTime();
}
```

#### Empirical analysis

#### Data analysis

###### Standard plot.

![standard_plot](standard_plot.png)

###### Log-log plot.

Plot running time *T*(*N*) vs. input size *N* using **log-log scale.**

![log-log](log-log.png)

###### Regression.

###### Hypothesis.

#### Prediction and validation

![log-log](hyp.png)

#### Doubling hypothesis - Quick way to estimate *b* in a power-law relationship

![log-log](double-hyp1.png)

![log-log](double-hyp2.png)

#### Experimental algorithmics

![exp-anly](exp-anly.png)

### ‣ Mathematical models for running time

Total running time: sum of cost × frequency for all operations.

- Need to analyze program to determine set of operations.
- Cost depends on machine, compiler.
- Frequency depends on algorithm, input data.

![log-log](mathmodel.png)

#### Example 1-SUM:

> Q. How many instructions as a function of input size *N* ?

```java
int count = 0;
for (int i = 0; i < N; i++)
   if (a[i] == 0)
      count++;
```

| operation            | Frequency |
| -------------------- | --------- |
| variable declaration | 2         |
| assignment statement | 2         |
| less than compare    | N +1      |
| equal to compare     | N         |
| array access         | N         |
| increment            | N to 2 N  |



#### Example 2-SUM

##### Q. How many instructions as a function of input size *N* ?

```java
int count = 0;
for (int i = 0; i < N; i++)
   for (int j = i+1; j < N; j++)
      if (a[i] + a[j] == 0)
				count++;
```

| operation            | Frequency                  | Tilde Notation              |
| -------------------- | -------------------------- | --------------------------- |
| variable declaration | N+2                        | ~$N$                        |
| assignment statement | N+2                        | ~$N$                        |
| less than compare    | 1/2(N+1)(N+2)              | ~$\frac{1}{2}N^2$           |
| equal to compare     | 1/2 N(N-1)                 | ~$\frac{1}{2}N^2$           |
| array access         | N(N-1)                     | ~$N^2$                      |
| increment            | 1⁄2 N (N − 1) to N (N − 1) | ~$\frac{1}{2}N^2$ to ~$N^2$ |

##### Simplification of calculation：

1. cost model: Use some basic operation as a proxy for running time.

2. tilde notation

   > Technical definition: $f(N)$~$g(N)$ means $$ \lim_{N \to \infty} \frac{f(N)}{g(N)}=1 $$

   - Estimate running time (or memory) as a function of input size *N*. 
   - Ignore lower order terms.
     - when *N* is large, terms are negligible 
     - when *N* is small, we don't care

##### Q. Approximately how many array accesses as a function of input size *N* ?

A. ~$N^2$ array accesses.

**Bottom line**. Use cost model and tilde notation to simplify counts

#### Example 3-SUM

> Q. Approximately how many array accesses as a function of input size *N* ?

```java
int count = 0;
for (int i = 0; i < N; i++)
   for (int j = i+1; j < N; j++)
      for (int k = j+1; k < N; k++)
         if (a[i] + a[j] + a[k] == 0)
            count++;
```

A. ~$\frac{1}{2}N^3$ array accesses.

#### Estimating a discrete sum

Q. How to estimate a discrete sum?

A1. Take discrete mathematics course.

A2. Replace the sum with an integral, and use calculus!

![log-log](dscrtmath.png)



### ‣ Order-of-growth classifications

the small set of functions suffices to describe order-of-growth of typical algorithms. —— $1$, $logN$, $N$, $NlogN$, $N^2$, $N^3$, $2^N$

![log-log](orderofgrowth.png)

Bottom line. Need linear or linearithmic alg to keep pace with Moore's law.

![log-log](classification.png)

### ‣ theory of algorithms

#### Type of analyses

- Best case. Lower bound on cost.

> Determined by “easiest” input. 
>
> Provides a goal for all inputs.

- Worst case. Upper bound on cost.

> Determined by “most difficult” input.
>
> Provides a guarantee for all inputs.

- Average case. Expected cost for random input.

> Need a model for “random” input. 
>
> Provides a way to predict performance.

Actual data might not match input model?

- Need to understand input to effectively process it. 

- Approach 1: design for the worst case.

- Approach 2: randomize, depend on probabilistic guarantee.

#### **Goals:** 

- Establish “difficulty” of a problem. 
- Develop “optimal” algorithms.

#### **Approach.** 

- Suppress details in analysis: analyze “to within a constant factor”. 
- Eliminate variability in input model by focusing on the worst case.

#### **Optimal algorithm.**

- Performance guarantee (to within a constant factor) for any input. 
- No algorithm can provide a better performance guarantee.

#### **Commonly-used notations**

![log-log](notation.png)

#### Algorithm design approach

##### Start.

- Develop an algorithm. 
- Prove a lower bound.

##### Gap?

- Lower the upper bound (discover a new algorithm). 
- Raise the lower bound (more difficult).

##### Golden Age of Algorithm Design.

- 1970s-. Steadily decreasing upper bounds for many important problems. 
- Many known optimal algorithms.

##### Caveats.

- Overly pessimistic to focus on worst case?
- Need better than “to within a constant factor” to predict performance.

### ‣ memory

#### Basics

- Bit. 0 or 1. 
- Byte. 8 bits.
- Megabyte (MB). 1 million or 220 bytes. 
- Gigabyte (GB). 1 billion or 230 bytes.
- 64-bit machine. We assume a 64-bit machine with 8 byte pointers.
  - Can address more memory. 
  - Pointers use more space.（some JVMs "compress" ordinary object pointers to 4 bytes to avoid this cost）

#### Typical memory usage for primitive types and arrays

##### for primitive types

| Type    | Bytes |
| ------- | ----- |
| boolean | 1     |
| bytes   | 1     |
| char    | 2     |
| int     | 4     |
| float   | 4     |
| long    | 8     |
| double  | 8     |

##### for one-dimensional arrays

| Type     | bytes   |
| -------- | ------- |
| char[]   | 2N + 24 |
| int[]    | 4N + 24 |
| double[] | 8N + 24 |

##### for two-dimensional arrays

| Type         | bytes   |
| ------------ | ------- |
| `char[][]`   | ~ 2 M N |
| `int[][]`    | ~ 4 M N |
| `double[][]` | ~ 8 M N |

#### Typical memory usage for objects in Java

- Object overhead. 16 bytes.
- Reference. 8 bytes.
- Padding. Each object uses a multiple of 8 bytes.

![log-log](dataobj.png)

![log-log](virgin-string.png)

#### Memory Usage Summary

Total memory usage for a data type value:

- Primitive type: 4 bytes for int, 8 bytes for double, ... 
- Object reference: 8 bytes.
- Array: 24 bytes + memory for each array entry. 
- Object: 16 bytes + memory for each instance variable + 8 bytes if inner class (for pointer to enclosing class). 
- Padding: round up to multiple of 8 bytes.

**Shallow memory usage:** Don't count referenced objects.

**Deep memory usage:** If array entry or instance variable is a reference, add memory (recursively) for referenced object.