# Single object tracking in clutter:
## Challenges: 
1. Missed detections
2. Clutter detection: false detections that are unrelated 
3. Unknown data association

## Measurement models: 
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
* Specifically, consider 
  ρ(xₖ | O₁:ₖ₋₁) = N(xₖ; x̄ₖ|ₖ₋₁, Pₖ|ₖ₋₁),  
  g(αₖ | xₖ) = N(αₖ; Hₖxₖ, Rₖ).

* Then, ρ(xₖ | O₁:ₖ) = N(xₖ; x̄ₖ|ₖ, Pₖ|ₖ) where  
  x̄ₖ|ₖ = x̄ₖ|ₖ₋₁  
  Pₖ|ₖ = Pₖ|ₖ₋₁  

when Oₖ = [], and, when Oₖ = αₖ,  
x̄ₖ|ₖ = x̄ₖ|ₖ₋₁ + Kₖ(αₖ - Hₖx̄ₖ|ₖ₋₁)  
Pₖ|ₖ = Pₖ|ₖ - KₖHₖPₖ|ₖ₋₁  

where Kₖ = Pₖ|ₖ₋₁Hₖᵀ(HₖPₖ|ₖ₋₁Hₖᵀ + Rₖ)⁻¹ is the Kalman gain.  