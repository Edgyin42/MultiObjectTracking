# Single object tracking in clutter:
## Challenges: 
1. Missed detections
2. Clutter detection: false detections that are unrelated 
3. Unknown data association

## Measurement models: 
The object is detected with probability $p^D(x_k)$, and then generates a measurement from p(o_k|x_k) = g(o_k|x_k)