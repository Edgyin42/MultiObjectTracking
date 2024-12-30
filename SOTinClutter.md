# Single object tracking in clutter:
## Challenges: 
1. Missed detections
2. Clutter detection: false detections that are unrelated 
3. Unknown data association

## Measurement models: 
### Model with known associations:
* The object is detected with probability $p^D(x_k)$, and then generates a measurement from $p(o_k|x_k) = g(o_k|x_k)$
* We use a matrix to represent object detections: <br> 
$$O_k =
\begin{cases} 
1 & \text{if object is undetected,} \\
o_k & \text{if object is detected.}
\end{cases}$$
* We use $|O_k|$ to denote the number of column vectors in $O_k$. 
* Given x_k, $|O_k|$ is Bernoulli distributed: 
$$
|O_k| = 
\begin{cases} 
    1 & \text{with probability } p^D(x_k), \\
    0 & \text{with probability } 1 - p^D(x_k).
\end{cases}
$$
* Using the matrix notation, we get: 
$$
p(O_k | x_k) =
\begin{cases} 
    1 - P^{D}(x_k) & \text{if } O_k = [], \\
    P^{D}(x_k) \cdot g(o_k | x_k) & \text{if } O_k = o_k.
\end{cases}
$$

* Using Chapman-Kolmogorov equation: <br> 
$$ρ(xₖ | O₁:ₖ₋₁) = ∫ π(xₖ | xₖ₋₁)ρ(xₖ₋₁ | O₁:ₖ₋₁)dxₖ₋₁$$
* Prediction: 
$$
p(x_k | O_{1:k}, x_{k-1}) =
\begin{cases} 
    p(x_k | O_{1:k-1}) \cdot (1 - P^{D}(x_k)) & \text{if } O_k = [], \\
    p(x_k | O_{1:k-1}) \cdot P^{D}(x_k) \cdot g(o_k | x_k) & \text{if } O_k = o_k.
\end{cases}
$$


With constant $p^D(x)$, we simplify it to: 
$$
p(x_k | O_{1:k}, x_{k-1}) =
\begin{cases} 
    p(x_k | O_{1:k-1}) & \text{if } O_k = [], \\
    p(x_k | O_{1:k-1}) & \text{if } O_k = o_k.
\end{cases}
$$

Example: 
Specifically, consider:

$$
\rho(x_k \mid O_{1:k-1}) = \mathcal{N}(x_k; \bar{x}_{k|k-1}, P_{k|k-1}),
$$

$$
g(o_k \mid x_k) = \mathcal{N}(o_k; H_k x_k, R_k).
$$

Then,

$$
\rho(x_k \mid O_{1:k}) = \mathcal{N}(x_k; \bar{x}_{k|k}, P_{k|k}),
$$

where

$$
\bar{x}_{k|k} = \bar{x}_{k|k-1}, \quad P_{k|k} = P_{k|k-1}
$$

when $ O_k = [] $, and when $ O_k = o_k$,

$$
\bar{x}_{k|k} = \bar{x}_{k|k-1} + K_k (o_k - H_k \bar{x}_{k|k-1}),
$$

$$
P_{k|k} = P_{k|k-1} - K_k H_k P_{k|k-1},
$$

where

$$
K_k = P_{k|k-1} H_k^T \left( H_k P_{k|k-1} H_k^T + R_k \right)^{-1}
$$

is the Kalman gain.
### Standard clutter model: 
Observed measurement matrix:

$$
Z_k = \Pi(O_k, C_k),
$$

where $\Pi$ randomly shuffles column vectors, and $ C_k $ is clutter. <br> 
Our model includes:  <br> 
* Number of detections: $|C_k|$
* Vectors in $C_k$

**A possible clutter model**

* Split volume into j cells,

$$
C_k = \Pi(C^{(1)}_k, \dots, C^{(j)}_k),
$$

where $ C^{(i)}_k $ denotes clutter in cell $ i $.

* $ C^{(1)}_k, \dots, C^{(j)}_k $ are assumed independent.

* $ |C^{(i)}_k| $ is Bernoulli with probability $ \frac{\lambda V}{j} $.

* Detections are uniformly distributed within their cells.

When sensor resolution approaches unlimited, we achieve new clutter model: <br> 
**A new clutter model**

* Let us increase $ j $ and make cells smaller!

**Consequences:**

* Possible to obtain more detections.
* Probability of detection in a single cell, $ \frac{\lambda V}{j} $, decreases.
* $ \mathbb{E}[|C_k|] = \lambda V $ for all $ j $.

* In the limit, as $ j \to \infty $:
    * $ |C_k| $ is Poisson distributed.
    * $ C_k $ is a **Poisson point process.**


### Poisson point processes: 
* The Poisson point process (PPP) is the default model for clutter $ C_k = [c_1^k, \dots, c_m^k] $.

**PPP introduced above**

* The number of clutter is

$$
m \sim \text{Poisson}(\lambda V).
$$

* Given $ m_k $, the vectors $ c_1^k, \dots, c_m^k $ are i.i.d.

$$
c_k \sim \text{unif}(V),
$$

where $ V $ is our field of view.

We want to parametrize PPPs more since $c_k^i$ is not necessarily uniformly distributed in V. 
* More generally, we parametrize PPPs using either
    - an **intensity function**, $ \lambda_c(c) \geq 0 $,
    - or a combination of
        $ \bar{\lambda}_c = \int \lambda_c(c) \, dc $ **rate**
        $ f_c(c) = \frac{\lambda_c(c)}{\bar{\lambda}_c} $ **spatial pdf**

* **Note:** the intensity can be computed from rate and spatial pdf:

$$
\lambda_c(c) = \bar{\lambda}_c f_c(c).
$$

**PPP distributions**

* For $ C_k = [c_1^k, \dots, c_{m_k}^k] $,

$$
p(C_k) = p(C_k, m_k) = p(m_k) p(C_k \mid m_k)
$$

$$
= \text{Poisson}(m_k; \lambda_c) \prod_{i=1}^{m_k} f_c(c_{k_i})
$$

$$
= \frac{\exp(-\lambda_c) \lambda_c^{m_k}}{m_k!} \prod_{i=1}^{m_k} \frac{\lambda_c(c_{k_i})}{\lambda_c}
$$

$$
= \exp(-\lambda_c) \prod_{i=1}^{m_k} \frac{\lambda_c(c_{k_i})}{m_k!}
$$


### Model with unknown associations (complete model):

**Objective:**
* $p(\mathbf{Z_k|x_k})$, needed in the update step
* $Z_k = \Pi (O_k, C_k)$ <br> 

**Main  challenges:** 
1. The width of $Z_k$ is random. 
2. We do not know whether any detection in $Z_k$ is an object detection. 

**Data associations**

* To describe the data association we use

$$
\theta = 
\begin{cases}
i > 0 & \text{if } z^i \text{ is an object detection}, \\
0 & \text{if the object is undetected}.
\end{cases}
$$

**Data association example**

* Suppose $ Z = [z^1, z^2, z^3] $. If $ \theta = 2 $, then $ z^2 $ is an object detection, whereas $ z^1 $ and $ z^3 $ are clutter. If $ \theta = 0 $, then the object is undetected, and $ z^1 $, $ z^2 $, and $ z^3 $ are all clutter detections.

**To find p(Z|x), we introduce the variables m and $\theta$**

* To find $ p(Z \mid x) $, we introduce the variables $ m $ and $ \theta $:

$$
p(Z \mid x) = p(Z, m \mid x)
$$

$$
= \sum_{\theta=0} p(Z, m, \theta \mid x)
$$

$$
= \sum_{\theta=0} p(Z \mid m, \theta, x) p(\theta, m \mid x)
$$

* $ p(Z \mid m, \theta, x) $ and $ p(\theta, m \mid x) $ have simple expressions. This enables us to **express the measurement model**!


Expression differ depending on: 
* $\theta$ = 0: object is not detected, m clutter detection. 
* $\theta$ = i > 0, m > 0: object is detected, m - 1 clutter detection, obejct is given index i. 
<br> 


When $ \theta = 0 $,

$$
p(Z \mid m, \theta, x) = \prod_{i=1}^{m} f_c(z^i)
$$

$$
p(\theta, m \mid x) = (1 - P^D(x)) \, \text{Poisson}(m; \lambda_c)
$$

which implies that

$$
p(Z, m, \theta \mid x) = (1 - P^D(x)) \, \text{Poisson}(m; \bar{\lambda}_c) \prod_{i=1}^{m} f_c(z^i)
$$

* Using $ \text{Poisson}(m; \lambda_c) = \frac{\exp(-\bar{\lambda}_c) \bar{\lambda}_c^m}{m!} $ and $ f_c(z) = \frac{\lambda_c(z)}{\lambda_c} $,

$$
p(Z, m, \theta \mid x) = (1 - P^D(x)) \frac{\exp(-\bar{\lambda}_c) \bar{\lambda}_c^m}{m!} \prod_{i=1}^{m} \frac{\lambda_c(z^i)}{\lambda_c}
$$

$$
= (1 - P^D(x)) \frac{\exp(-\bar{\lambda}_c)}{m!} \prod_{i=1}^{m} \lambda_c(z^i)
$$


When $ \theta = 1, \dots, m $,

$$
p(Z \mid m, \theta, x) = g_k(z^\theta \mid x) \prod_{i=1, i \neq \theta}^{m} f_c(z^i) = g_k(z^\theta \mid x) \prod_{i=1, i \neq \theta}^{m} \frac{f_c(z^i)}{f_c(z^\theta)}
$$

$$
p(\theta, m \mid x) = P^D(x) \, \text{Poisson}(m - 1; \lambda_c) \cdot \frac{1}{m}
$$

which implies that

$$
p(Z, m, \theta \mid x) = P^D(x) \, \text{Poisson}(m - 1; \lambda_c) \cdot \frac{1}{m} \cdot g_k(z^\theta \mid x) \prod_{i=1, i \neq \theta}^{m} f_c(z^i)
$$

* Using $ \text{Poisson}(m - 1; \lambda_c) = \frac{\exp(-\lambda_c) \lambda_c^{m-1}}{(m-1)!} $ and $ f_c(z) = \frac{\lambda_c(z)}{\lambda_c} $,

$$
p(Z, m, \theta \mid x) = P^D(x) \frac{\exp(-\lambda_c) \lambda_c^{m-1}}{(m-1)!} \cdot \frac{1}{m} \cdot g_k(z^\theta \mid x) \prod_{i=1}^{m} \frac{\lambda_c(z^i)}{\lambda_c}
$$

$$
= P^D(x) \, g_k(z^\theta \mid x) \frac{\exp(-\lambda_c)}{m!} \prod_{i=1}^{m} \lambda_c(z^i)
$$


Adding all together, 
* Putting these equations together,

$$
p(Z \mid x) = \sum_{\theta=0}^{m} p(Z, m, \theta \mid x)
$$

$$
= (1 - P^D(x)) \frac{\exp(-\lambda_c)}{m!} \prod_{i=1}^{m} \lambda_c(z^i) + \sum_{\theta=1}^{m} P^D(x) \, g_k(z^\theta \mid x) \frac{\exp(-\lambda_c)}{\lambda_c(z^\theta)} \prod_{i=1}^{m} \lambda_c(z^i)
$$

$$
= \left[ (1 - P^D(x)) + P^D(x) \sum_{\theta=1}^{m} \frac{g_k(z^\theta \mid x)}{\lambda_c(z^\theta)} \right] \frac{\exp(-\lambda_c)}{m!} \prod_{i=1}^{m} \lambda_c(z^i)
$$

### Normalizing the posterior mixture of densities: 
**Normalizing a mixture**

* If 

$$
p(x) \propto \sum_{\theta=0}^{m} g_{\theta}(x),
$$

it follows that

$$
p(x) = \sum_{\theta=0}^{m} w_{\theta} p_{\theta}(x),
$$

where $ p_{\theta}(x) \propto g_{\theta}(x) $ and $ w_{\theta} \propto \int g_{\theta}(x) \, dx $.

* **Note:** $ w_{\theta} $ should be normalized to become a pmf.

* Specifically, we can set

$$
\tilde{w}_{\theta} = \int g_{\theta}(x) \, dx,
$$

$$
p_{\theta}(x) = \frac{g_{\theta}(x)}{\tilde{w}_{\theta}},
$$

$$
w_{\theta} = \frac{\tilde{w}_{\theta}}{\sum_{i} \tilde{w}_{i}}.
$$

### General update equation: 
* **Measurement model:**

$$
p(Z \mid x) = \left[ (1 - P^D(x)) + P^D(x) \sum_{\theta=1}^{m} \frac{g(z^\theta \mid x)}{\lambda_c(z^\theta)} \right] \frac{\exp(-\lambda_c)}{m!} \prod_{i=1}^{m} \lambda_c(z^i)
$$

* **Posterior density:**

$$
p(x \mid Z) \propto p(x) p(Z \mid x)
$$

$$
\propto p(x) \left[ (1 - P^D(x)) + P^D(x) \sum_{\theta=1}^{m} \frac{g(z^\theta \mid x)}{\lambda_c(z^\theta)} \right]
$$

**Posterior probabilities and densities**

* We get $ p(x \mid Z) = \sum_{\theta} w_\theta p_\theta(x) $, where

   - $ \theta = 0 $
     - Object is undetected
     - 
       $$
       w_0 = \int p(x) (1 - P^D(x)) \, dx
       $$

       $$
       p_0(x) = \frac{p(x) (1 - P^D(x))}{\int p(x') (1 - P^D(x')) \, dx'}
       $$

   - $ \theta \in \{1, 2, \dots, m\} $
     - $ z^\theta $ is object detection
     - 
       $$
       w_\theta = \frac{1}{\lambda_c(z^\theta)} \int p(x) P^D(x) g(z^\theta \mid x) \, dx
       $$

       $$
       p_\theta(x) = \frac{p(x) P^D(x) g(z^\theta \mid x)}{\int p(x') P^D(x') g(z^\theta \mid x') \, dx'}
       $$

   and $ w_\theta \propto w_\theta $


* Recall the object measurement model

$$
p(O \mid x) = 
\begin{cases} 
1 - P^D(x) & \text{if } O = [] \\
P^D(x) g(o \mid x) & \text{if } O = o 
\end{cases}
$$

* Given $ O $, we get

$$
p(x \mid O) \propto 
\begin{cases} 
p(x) (1 - P^D(x)) & \text{if } O = [] \\
p(x) P^D(x) g(o \mid x) & \text{if } O = o
\end{cases}
$$

* By comparison,

$$
p_\theta(x) \propto 
\begin{cases} 
p(x) (1 - P^D(x)) & \text{if } \theta = 0 \\
p(x) P^D(x) g(z^\theta \mid x) & \text{if } \theta \in \{1, 2, \dots, m\}
\end{cases}
$$

**Note:** $ p_\theta(x) $ is identical to $ p(x \mid O) $, with $ O $ defined by $ \theta $ and $ Z $.

## Conceptual soluttion: 
### Main Result

- **Suppose**:
  $$
  p(x_{k-1} | Z_{1:k-1}) = \sum_{\theta_{1:k-1}} w^{\theta_{1:k-1}} p^{\theta_{1:k-1}}_{k-1|k-1}(x_{k-1}),
  $$
  where:
  $$
  w^{\theta_{1:k-1}} = \Pr[\theta_{1:k-1} | Z_{1:k-1}],
  $$
  and:
  $$
  p^{\theta_{1:k-1}}_{k-1|k-1}(x_{k-1}) = p(x_{k-1} | \theta_{1:k-1}, Z_{1:k-1}).
  $$

- **We can then express the predicted and updated densities as**:

  **Predicted density**:
  $$
  p(x_k | Z_{1:k-1}) = \sum_{\theta_{1:k-1}} w^{\theta_{1:k-1}} p^{\theta_{1:k-1}}_{k|k-1}(x_k),
  $$

  **Updated density**:
  $$
  p(x_k | Z_{1:k}) = \sum_{\theta_{1:k}} w^{\theta_{1:k}} p^{\theta_{1:k}}_{k|k}(x_k).
  $$
### Prediction Step

- **Given**:
  $$
  p(x_{k-1} | Z_{1:k-1}) = \sum w^{(\theta_{1:k-1})} \, \rho_{k-1|k-1}^{(\theta_{1:k-1})}(x_{k-1}),
  $$

- Then:
  $$
  p(x_k | Z_{1:k-1}) = \int p(x_k | Z_{1:k-1}, x_{k-1}) \, p(x_{k-1} | Z_{1:k-1}) \, dx_{k-1}.
  $$

  Expanding:
  $$
  p(x_k | Z_{1:k-1}) = \sum w^{(\theta_{1:k-1})} \int p(x_k | Z_{1:k-1}, x_{k-1}) \, \rho_{k-1|k-1}^{(\theta_{1:k-1})}(x_{k-1}) \, dx_{k-1}.
  $$

  Simplifies to:
  $$
  p(x_k | Z_{1:k-1}) = \sum w^{(\theta_{1:k-1})} \, \rho_{k|k-1}^{(\theta_{1:k-1})}(x_k).
  $$

- **Key Points**:
  - **Weights remain unchanged**.
  - Perform the **standard prediction of densities** for each hypothesis.


### Update Step 

**Measurement Model**

The measurement model is given as:
$$
p(z_k | x_k) = \frac{\left[(1 - P^D(x_k)) + P^D(x_k) \sum_{\theta=1}^{m_k} \frac{g_k(z^\theta | x_k)}{\lambda_c(z^\theta)}\right] \exp(-\lambda_c)}{m_k!} \prod_{i=1}^{m_k} \lambda_c(z^i),
$$
where:
- $ P^D(x_k) $: Probability of detection.
- $ g_k(z^\theta | x_k) $: Measurement likelihood.
- $ \lambda_c(z^\theta) $: Clutter intensity.
- $ m_k $: Number of measurements.

---

### Recursive Expression

For:
$$
p(x_k | Z_{1:k-1}) = \sum_{\theta_{1:k-1}} w^{(\theta_{1:k-1})} \, \rho_{k-1|k-1}^{(\theta_{1:k-1})}(x_k),
$$

this implies that:
$$
p(x_k | Z_{1:k}) \propto \sum_{\theta_{1:k-1}} w^{(\theta_{1:k-1})} \, \rho_{k-1|k-1}^{(\theta_{1:k-1})}(x_k) (1 - P^D(x_k))
$$

$$
+ \sum_{\theta_{1:k-1}} \sum_{\theta=1}^{m_k} \frac{1}{\lambda_c(z^\theta)} w^{(\theta_{1:k-1})} \, \rho_{k-1|k-1}^{(\theta_{1:k-1})}(x_k) P^D(x_k) g_k(z^\theta | x_k).
$$

## Posterior Density

The posterior density is given as:
$$
p(x_k | Z_{1:k}) = \sum_{\theta_{1:k-1}} w^{(\theta_{1:k-1})} \rho_{k-1|k-1}^{(\theta_{1:k-1})}(x_k) (1 - P^D(x_k))
$$
$$
+ \sum_{\theta_{1:k-1}} \sum_{\theta=1}^{m_k} \frac{1}{\lambda_c(z^\theta)} w^{(\theta_{1:k-1})} \rho_{k-1|k-1}^{(\theta_{1:k-1})}(x_k) P^D(x_k) g_k(z^\theta | x_k).
$$

---

## Posterior Probabilities and Densities

We obtain:
$$
p(x_k | Z_{1:k}) = \sum_{\theta_{1:k}} w_k^{(\theta_{1:k})} p_k^{(\theta_{1:k})}(x_k),
$$
where $ w_k^{(\theta_{1:k})} \propto w^{(\theta_{1:k})} $ and:

### Case 1: $\theta_k = 0$ (Object is undetected)
$$
w_k^{(\theta_{1:k})} = w^{(\theta_{1:k-1})} \int \rho_{k-1|k-1}^{(\theta_{1:k-1})}(x_k) (1 - P^D(x_k)) dx_k,
$$
$$
p_k^{(\theta_{1:k})}(x_k) = \frac{\rho_{k-1|k-1}^{(\theta_{1:k-1})}(x_k) (1 - P^D(x_k))}{\int \rho_{k-1|k-1}^{(\theta_{1:k-1})}(x_k) (1 - P^D(x_k)) dx_k}.
$$

---

### Case 2: $\theta_k \in \{1, 2, \dots, m_k\}$ (Object detection, $z^{\theta_k}$)
$$
w_k^{(\theta_{1:k})} = \frac{1}{\lambda_c(z^{\theta_k})} w^{(\theta_{1:k-1})} \int \rho_{k-1|k-1}^{(\theta_{1:k-1})}(x_k) P^D(x_k) g_k(z^{\theta_k} | x_k) dx_k,
$$
$$
p_k^{(\theta_{1:k})}(x_k) = \frac{\rho_{k-1|k-1}^{(\theta_{1:k-1})}(x_k) P^D(x_k) g_k(z^{\theta_k} | x_k)}{\int \rho_{k-1|k-1}^{(\theta_{1:k-1})}(x_k) P^D(x_k) g_k(z^{\theta_k} | x_k) dx_k}.
$$


---

### Key Notes

- For every pair of hypotheses, $(\theta_{1:k-1}, \theta_k)$, a **new hypothesis** is generated.
- This new hypothesis is indexed using the vector $\theta_{1:k}$, where $\theta_{1:k} = (\theta_{1:k-1}, \theta_k)$.


## Closed-Form Expressions

Given:
$$
\rho_{k-1|k-1}^{(\theta_{1:k-1})}(x) = N(x; x_{k-1|k-1}^{(\theta_{1:k-1})}, P_{k-1|k-1}^{(\theta_{1:k-1})}),
$$
$$
P^D(x) = P^D,
$$
$$
g_k(o_k | x_k) = N(o_k; H_k x_k, R_k),
$$
the posterior density $ p_k|k^{(\theta_{1:k})}(x) $ and weights $ w_k^{(\theta_{1:k})} $ are:

---

### Posterior Density
$$
p_k|k^{(\theta_{1:k})}(x) =
\begin{cases} 
N(x; x_{k-1|k-1}^{(\theta_{1:k-1})}, P_{k-1|k-1}^{(\theta_{1:k-1})}) & \text{if } \theta_k = 0, \\
N(x; x_{k|k}^{(\theta_{1:k})}, P_{k|k}^{(\theta_{1:k})}) & \text{if } \theta_k \in \{1, 2, \dots, m_k\}.
\end{cases}
$$

---

### Weights
$$
w_k^{(\theta_{1:k})} =
\begin{cases}
w_{k-1}^{(\theta_{1:k-1})} (1 - P^D) & \text{if } \theta_k = 0, \\
\frac{w_{k-1}^{(\theta_{1:k-1})} P^D}{\lambda_c(z^{\theta_k})} N(z^{\theta_k}; z_{k-1|k-1}^{(\theta_{1:k-1})}, S_{k-1|k-1}^{(\theta_{1:k-1})})^{-1} & \text{if } \theta_k \in \{1, 2, \dots, m_k\}.
\end{cases}
$$

---

### Definitions
- $\ x_{k|k}^{(\theta_{1:k})} $: Posterior mean given $ Z_{1:k} $ and $ \theta_{1:k} $.
- $ P_{k|k}^{(\theta_{1:k})} $: Posterior covariance given $ Z_{1:k} $ and $ \theta_{1:k} $.
- $ z_{k-1|k-1}^{(\theta_{1:k-1})} $: Predicted object measurement mean assuming the predicted density $ p_{k-1|k-1}^{(\theta_{1:k-1})}(x) $.
- $ S_{k-1|k-1}^{(\theta_{1:k-1})} $: Predicted object measurement covariance assuming the predicted density $ p_{k-1|k-1}^{(\theta_{1:k-1})}(x) $.


## Nearest neighbour algorithm: 
### Basic Idea

- Prune all hypotheses except the most probable one.

### Algorithm: The Nearest Neighbor (NN) Filtering Update

1. **Compute** $ w_k^\theta $, for $ \theta = 0, 1, \dots, m_k $.

2. **Find** 
   $$
   \theta_k^p = \arg\max_\theta w_k^\theta.
   $$

3. **Compute** $ x_k^{\theta_k^p} $ and $ P_k^{\theta_k^p} $.

4. **Set** 
   $$
   x_{k|k}^{\text{NN}} = x_k^{\theta_k^p} \quad \text{and} \quad P_{k|k}^{\text{NN}} = P_k^{\theta_k^p}.
   $$

### Note:
We then assume that:
$$
p(x_{k|k}^{\text{NN}} | Z_{1:k}) = \mathcal{N}(x_k; x_{k|k}^{\text{NN}}, P_{k|k}^{\text{NN}}).
$$
**PROS**: 
* Fast to implement 
* Works well in simple scenario

**CONS**: 
* Ignore uncertainties which increase the risk of losing track of object 
## Probabilistic Data Association Filtering
Ideas: Approximate the posterior with the same mean and covariance as $p^{PDA}(x_k|Z_{1:k})$, where 
The posterior is given by:

$$
p^{\text{PDA}}(x_k | Z_{1:k}) = \sum_{\theta=0}^{m_k} w_k^{\theta} \, p_k^{\theta_k}(x_k),
$$

where:

$$
p_k^{\theta_k}(x_k) = \mathcal{N}(x_k; x_k^{\theta_k}, P_k^{\theta_k}),
$$

and:
- $ w_k^{\theta} $ are the weights associated with each hypothesis $\theta$,
- $ x_k^{\theta_k} $ and $ P_k^{\theta_k} $ are the mean and covariance of the Gaussian distribution for hypothesis $\theta$. 

## Algorithm 1: The PDA Filtering Update

1. **Compute** $ w_k^\theta $, $ x_k^\theta $, and $ P_k^\theta $ for $\theta_k = 0, 1, \dots, m_k$.

2. **Set**:
   $$
   x_{k|k}^{\text{PDA}} = \sum_{\theta_k=0}^{m_k} w_k^\theta \, x_k^\theta.
   $$

3. **Compute**:
   $$
   P_{k|k}^{\text{PDA}} = \sum_{\theta_k=0}^{m_k} w_k^\theta \, P_k^\theta 
   + \sum_{\theta_k=0}^{m_k} w_k^\theta \, (x_k^\theta - x_{k|k}^{\text{PDA}}) (x_k^\theta - x_{k|k}^{\text{PDA}})^\top.
   $$

### Explanation:
- $ w_k^\theta $: Weight associated with hypothesis $\theta_k$.
- $ x_k^\theta $: Mean of the Gaussian for hypothesis $\theta_k$.
- $ P_k^\theta $: Covariance of the Gaussian for hypothesis $\theta_k$.
- $ x_{k|k}^{\text{PDA}} $: Posterior mean after probabilistic data association.
- $ P_{k|k}^{\text{PDA}} $: Posterior covariance after probabilistic data association.

**PROS**: 
* Fast to implement 
* Works well in simple scenario
* Acknowledges uncertainties slightly better than NN

**CONS**: 
* Perform poorly in complicated scenarios. 


## Gaussian Sum Filtering 
Basic idea: A sum of a few components that contribute significantly to the posterior 

3 ways to choose the components to add into the posterior. <br> 

1. Set the cap for the weight. If below that threshold, we wouldn't add the hypthesis to the posterior. 
2. Merging similar components 
3. Set the cap for the number of hypotheses. 

We can combine all of this in different ways. 

## Estimation Techniques

### Minimum Mean Square Error (MMSE) Estimation
- The **posterior mean**:
  $$
  \bar{x}_{k|k} = \mathbb{E}[X | Z_{1:k}] = \sum_h w_k^h x_{k|k}^h
  $$
  minimizes the **MMSE**:
  $$
  \mathbb{E} \left[ (X - \bar{x}_{k|k})^\top (X - \bar{x}_{k|k}) \,|\, Z_{1:k} \right].
  $$

---

### Most Probable Hypothesis Estimation
- For **multi-modal densities**, we may prefer the **most probable hypothesis**:
  $$
  h^* = \arg \max_h w_k^h
  $$
  and the corresponding estimate:
  $$
  \hat{x}_{k|k} = x_{k|k}^{h^*}.
  $$


**Pros and cons**

++ Significantly more accurate than NN and PDA. <br>
++ Complexity can be adjusted to computational resources. <br>
-- More complicated to implement than NN/PDA. <br>
-- More computationally demanding to run than NN/PDA. <br>

• **Note:** even though GSFs looks much more accurate than NN and PDA, the difference is mostly noticeable in medium-difficult settings.


## Gating: 

* Is a technique to disregard measurements without computing the weights. 
* In Gaussian, prefer using ellipsoidal gate 
* Need to balance the threshold G

- If $ G $ is small, there is a **high probability** that the object detection is outside the gate.

- Given $ h_{k-1} $ and $ \theta_k $, where $ \theta_k > 0 $, the **probability** that the object measurement is outside the gate is:
  $$
  P_G = \Pr \left[ d^2_{k-1, \theta_k}> G | h_{k-1, \theta_k} \right].
  $$

- It can be shown that:
  $$
  d^2_{k-1, \theta_k} | h_{k-1, \theta_k}\sim \chi^2(n_z).
  $$
    where $n_z$ is the dimension of measurement. 
- A common strategy is to set a **desired value** for $ P_G $, such as $ 99.5\% $, and use the **cumulative distribution function (CDF)** of $ \chi^2(n_z) $ to determine $ G $.
