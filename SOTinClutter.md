# Single object tracking in clutter:
## Challenges: 
1. Missed detections
2. Clutter detection: false detections that are unrelated 
3. Unknown data association

## Measurement models: 
* The object is detected with probability $p^D(x_k)$, and then generates a measurement from $p(o_k|x_k) = g(o_k|x_k)$
* We use a matrix to represent object detections: 
\[
O_k =
\begin{cases} 
1 & \text{if object is undetected,} \\
o_k & \text{if object is detected.}
\end{cases}
\]
* We use $|O_k|$ to denote the number of column vectors in $O_k$. 
* Given x_k, $|O_k|$ is Bernoulli distributed: 
$$|O_k| = 
\begin{cases} 
    1 &  with probability p^D(x_k) \\
    0  & with probability 1 - p^D(x_k). 
\end{cases}
$$