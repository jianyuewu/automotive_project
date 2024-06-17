# Kalman filter
Different with low/high/band-pass filters.  
Kalman filter is mainy for removing Gaussion noises, by using data from different sensors.  
It can solve "Linear Quadratic Estimation Problem".  
Formula is: K_k = P_k|k-1 H_k^T (H_k P_k|k-1 H_k^T + R_k)^ (-1)  
Primarily it can be simplified and understand as weighted average.  
T = alpha * T1 + (1 - alpah) * T2;  
Instead of T = (T1 + T2) / 2;  
So, it can give more weight to more trustable device, like camera, lidar, GPS.  

# Types
## LKF(Linear Kalman Filter)
Linear Covariance Prediction/Update Equations.  
Pros: Efficient in linear systems due to use linear covariance prediction/ update equations.  
Cons: Only suitable for linear system.

## EKF(Extended Kalman Filter)
Smooth non-linear system. Linear relationship is approximated from the non-linear dynamics.  
Pros: Can linearize the state and observation models at each step.  
Cons: Lead to error accumulation, especially in highly nonlinear systems.  

## UKF(Unscented Kalman Filter)
Linear approximation of the uncertainty converances.  
Pros: Better at approximating the non-linear systems.  
Cons: Performance depends on the selection of sigma points and the param configurations.

# Implementation
Simulation can use C++ Eigen library, use matrix operations to speed up.
Code can refer to AKFSF-Simulation-CPP directory, forked from github, author is Suchit Kr.
Can checkout via "git submodule update --init ."

# References
https://github.com/technitute/AKFSF-Simulation-CPP  
https://www.youtube.com/watch?v=IFeCIbljreY  
https://www.youtube.com/watch?v=LioOvUZ1MiM
