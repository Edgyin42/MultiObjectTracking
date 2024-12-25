We use a set
$$x_k = \{x_k^1, x_k^2, ..., x_k^{n_k}\}$$
to represent the state. 

Both $x_k$ and $z_k$ are random finite sets. 
Bayesian filtering recursion: <br> 
* Prediction: $p(x_k | z_{1:k-1}) = \int p(x_k | x_{k-1}) p(x_{k-1} | z_{1:k-1}) dx_{k-1}$ <br>
*  Update: $p(x_k | z_{1:k}) = \frac{p(z_k | x_k) p(x_k | z_{1:k-1})}{\int p(z_k | x_k) p(x_k | z_{1:k-1}) dx_k}$

**Definition**: A random finite set (RFS) is random variable whose possible outcomes are sets with a finite number of unique elements. 

**Multiobject PDFs**: 
1. A multiobject PDF, $p_x({x^1, x^2, ..., x^n})$, is a non-negative function on set that integrates to 1. 
2. It captures both the distribution over cardinality and the distribution over the elements of the set. (given the cardinality)
3. Invariant to the order: 
$$
p_x(\{x^1, x^2\}) = p_x(\{x^2, x^1\})
$$


**Convolution theorem for independent RFSS**

* If $ x^1, x^2, \dots, x^n $ are independent RFSs (Random Finite Sets), then $ x = x^1 \cup x^2 \cup \dots \cup x^n $ has the multi-object pdf:

$$
p_x(x) = \sum_{\{x^1, x^2, \dots, x^n\} \vdash x} \prod_{i=1}^n p_{x_i}(x_i),
$$

where:

- $\{x^1, x^2, \dots, x^n\} \vdash x$ indicates the summation is over all partitions of $x$ such that:
$$
  x^1 \cup x^2 \cup \dots \cup x^n = x, \quad \text{and} \quad x^i \cap x^j = \emptyset \; \text{for} \; i \neq j.
$$


   where the summation is taken over all mutually disjoint (and possibly empty) sets $ x^1, x^2, \dots, x^n $ whose union is $ x $.


**Set integrals: definition**

For $ f: F(D) \to \mathbb{R} $, the set integral is defined as:

$$
\int f(x) \, \delta x = \sum_{i=0}^{\infty} \frac{1}{i!} \int f(\{x^1, \ldots, x^i\}) \, dx^1 \ldots dx^i
$$

This can be written as:

$$
= f(\emptyset) + \sum_{i=1}^{\infty} \frac{1}{i!} \int f(\{x^1, \ldots, x^i\}) \, dx^1 \ldots dx^i
$$
**Expected values**

For $ f: F(D) \to \mathbb{R} $, the expected value is

$$
E[f(x)] = \int f(x) p_x(x) \, \delta x = \sum_{i=0}^{\infty} \frac{1}{i!} \int f(\{x^1, \ldots, x^i\}) p_x(\{x^1, \ldots, x^i\}) \, dx^1 \ldots dx^i
$$


**Cardinality distributions**

* The cardinality distribution of an RFS, $ x \sim p_x(\cdot) $, is

$$
p_x(n) = \Pr[|x| = n].
$$

* Let the Kronecker delta function be denoted

$$
\delta_{ij} =
\begin{cases}
1 & \text{if } i = j, \\
0 & \text{otherwise}.
\end{cases}
$$

* It then holds that

$$
\Pr[|x| = n] = E[\delta_{n - |x|}]
$$

$$
= \sum_{i=0}^{\infty} \frac{1}{i!} \int \delta_{n - i} p_x(\{x^1, \ldots, x^i\}) \, dx^1 \ldots dx^i
$$

$$
= \frac{1}{n!} \int p_x(\{x^1, \ldots, x^n\}) \, dx^1 \ldots dx^n.
$$
