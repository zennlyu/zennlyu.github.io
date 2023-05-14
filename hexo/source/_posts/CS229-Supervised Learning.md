---
title: CS229-Supervised Learning
categories: [machine learning]
tags: [machine learning]
---

## Supervised Learning Terms

- Regression
- Classification
- Terms:
  - input **Features**: $x^{(i)}$
  - output **Target**: $y^{(i)}$
  - **Training sample**: a pair $(x^{(i)},y^{(i)})$
  - **Training set**: a list of $n$ training examples {$(x^{(i)},y^{(i)});i=1,...,n$}
  - $\chi$ denote the space of input values, and $\gamma$ the space of output values. In this example, $\chi$ = $\gamma$ = R.
  - **Hypothesis**: $h: \chi \mapsto \gamma$ so that $h(x)$ is a “good” predictor for the corresponding value of y



## Housing Example Jump Start

First approximate y as a linear function of x:
$$
\begin{align}
h_\theta (x)=\theta_0+\theta_1x_1+\theta_2x_2
\end{align}
$$

- $\theta_i's$: **Parameters/Weights**, parameterizing the space of linear functions mapping from $\Chi$ to $\Upsilon$ 

- $h(x)$: No risk of confusion, then drop the $\theta$

Let $x_0$ =1, that $$h(x)=\begin{align}\sum_{i=0}^{d}\theta_ix_i\end{align} = \theta^Tx$$ 

- where on the right-hand side above we are viewing θ and x both as vectors, and here d is the number of input variables (not counting x0).

Then how to pick parameters $\theta$ :

- make $h(x)$ close to $y$, at least for the training examples we have. 

- To formalize this, we will define a function that measures, for each value of the $θ’$s, how close the $h(x^{(i)})$’s are to the corresponding $y^{(i)}$’s. We define the **cost function**: $\begin{align} J(\theta)=\tfrac{1}{2}\sum_{i=1}^{n}(h_\theta(x^{(i)})- y^{i})^2\end{align}$

- **ordinary least squares regression model**

  

### 1. LMS Algorithms

**choose $\theta$ to minimize $J(\theta)$:** search algorithm starts “initial guesses” for θ, and that repeatedly changes $θ$ to make $J(θ)$ smaller, until hopefully converge to a value of $θ$ that minimizes $J(θ)$.

Specifically, let’s consider the **gradient descent algorithm**, which starts with some initial $θ$, and repeatedly performs the update: 
$$
\begin{align}
\theta_j := \theta_j-\alpha\tfrac{\partial}{\partial\theta_j}J_\theta. 
\end{align}
$$

This update is simultaneously performed for all values of $j=0,...,d$

- $\alpha$: Learning rate



To implement the algorithm: **work out what is the partial derivative term** on the right hand side. 

- Let’s first work it out for the case of if we have only one training example (x,y), so that we can neglect the sum in the definition of J. We have:

$$
\begin{aligned}
\tfrac{\partial}{\partial\theta_j}J_\theta &= \tfrac{\partial}{\partial\theta_j}\tfrac{1}{2}(h_\theta(x)-y)^2 \\
& =2\cdot\tfrac{1}{2}(h_\theta(x)-y)\cdot\tfrac{\partial}{\partial\theta_j}(h_\theta(x)-y) \\
& = (h_\theta(x)-y)\cdot\tfrac{\partial}{\partial\theta_j}(\sum_{i=0}^{d}\theta_ix_i-y) \\
& = (h_\theta(x)-y)\cdot x_j
\end{aligned}
$$

For a single training example, this gives the update rule:1
$$
\theta_j := \theta_j+\alpha(y^{(i)}-h_\theta(x^{(i)}))x^{(i)}_{j}
$$
The rule : **LMS update rule** (“least mean squares”), and is also known as the **Widrow-Hoff learning rule**. 

- This rule has several properties that seem natural and intuitive. 
- For instance, the magnitude of the update is proportional to the error term (y(i) − hθ(x(i))); thus, for instance, if we are encountering a training example on which our prediction nearly matches the actual value of y(i), then we find that there is little need to change the parameters; 
- in contrast, a larger change to the parameters will be made if our prediction hθ(x(i)) has a large error (i.e., if it is very far from y(i)).



Derived the LMS rule:  only a single training example.  There are two ways to modify this method for a training set of more than one example.

**1st** **batch gradient descent**. 

- _ replace it with the following algorithm: Repeat Until Convergence ---

$$
Repeat\: Until\: Convergence - \{
\theta_j := \theta_j+\alpha\sum_{i=1}^{n}(y^{(i)}-h_\theta(x^{(i)}))x_j^{(i)},(for\:every\:j) 
\}
$$

- By grouping the updates of the coordinates into an update of the vector θ, rewrite:

$$
\theta := \theta+\alpha\sum_{i=1}^{n}(y^{(i)}-h_\theta(x^{(i)}))x^{(i)}
$$

- can easily verify the quantity in the summation in the update rule above is just $\tfrac{∂J(θ)}{∂θ_j}$(for the original definition of $J$). So, this is simply gradient descent on the original cost function$J$. This method looks at every example in the entire training set on every step, and is called **batch gradient descent**. Note that, while gradient descent can be susceptible to local minima in general, the optimization problem we have posed here

**2nd stochastic gradient descent / incremental gradient descent**
$$
\begin{aligned}
Loop\: \{ for\: i=1\: to\: n \{\\
\theta_j := \theta_j+\alpha(y^{(i)}-h_\theta(x^{(i)}))x_j^{(i)} \\
(for\:every\:j)  \} \\
\}
\end{aligned}
$$
By grouping the updates of the coordinates into an update of the vector θ, we can rewrite update (2) in a slightly more succinct way:
$$
\theta := \theta+\alpha(y^{(i)}-h_\theta(x^{(i)}))x^{(i)}
$$

### 2.The normal equation 
