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
