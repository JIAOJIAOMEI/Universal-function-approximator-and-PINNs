[TOC]





# Universal function approximator

Neural networks are universal function approximators. 
Given enough neurons and layers, a neural network can approximate **any** function.
(I don't really believe this, but I agree that neural networks are very powerful function approximators.)
If you are interested in the proof, you can read the paper [Multilayer feedforward networks are universal approximators](https://github.com/JIAOJIAOMEI/Universal-function-approximator-and-PINNs/blob/main/1989-Multilayer%20feedforward%20networks%20are%20universal%20approximators.pdf).

![FunctionApproximation.png](FunctionApproximation.png)

```python

class Fitter(torch.nn.Module):

    def __init__(self, numHiddenNodes):
        super(Fitter, self).__init__()
        self.fully_connected_1 = torch.nn.Linear(in_features=1, out_features=numHiddenNodes)
        self.fully_connected_2 = torch.nn.Linear(in_features=numHiddenNodes, out_features=1)

    def forward(self, x, activation_Function):
        h = activation_Function(self.fully_connected_1(x))
        # Linear activation function used on the outer layer
        y = self.fully_connected_2(h)
        return y

```

# Fourier feature mapping

Fourier's transformation is indeed function approximation.
It can approximate any function with a series of sine and cosine functions.

**Since that Fourier's transformation and neural networks are both function approximators, can we combine them together?**

if you want more details, you can read the paper [Fourier Features Let Networks Learn High Frequency Functions in Low Dimensional Domains](https://github.com/JIAOJIAOMEI/Universal-function-approximator-and-PINNs/blob/main/Fourier%20Features%20Let%20Networks%20Learn%20High%20Frequency%20Functions%20in%20Low%20Dimensional%20Domains.pdf).

![Fourier_FunctionApproximation.png](Fourier_FunctionApproximation.png)

```python

def fourier_feature_mappping_function(x):
    return torch.cat([torch.sin(x), torch.cos(x)], dim=1)


class Fitter(torch.nn.Module):

    def __init__(self, numHiddenNodes):
        super(Fitter, self).__init__()
        self.fully_connected_1 = torch.nn.Linear(in_features=2, out_features=numHiddenNodes)
        self.fully_connected_2 = torch.nn.Linear(in_features=numHiddenNodes, out_features=1)

    def forward(self, x, activation_Function):
        first_layer_output = self.fully_connected_1(fourier_feature_mappping_function(x))
        first_layer_output = activation_Function(first_layer_output)
        y = self.fully_connected_2(first_layer_output)
        return y

```

# Different optimizers

![Optimiser_Compare.png](Optimiser_Compare.png)

# Solving first order ODEs with PINNs Example

The idea is based on this paper [Artificial Neural Networks for Solving ODEs and PDES](https://github.com/JIAOJIAOMEI/Universal-function-approximator-and-PINNs/blob/main/1998-Artificial%20Neural%20Networks%20for%20Solving%20ODEs%20and%20PDES.pdf),
the code implementation is based on this [Neural Networks for Solving Differential Equations](https://github.com/JIAOJIAOMEI/Universal-function-approximator-and-PINNs/blob/main/main%20reference%20for%20this%20project.pdf), I made some modifications.

To solve:

$$
\begin{equation}
\frac{d f(x)}{d x}=x^3+2 x+x^2 \frac{1+3 x^2}{1+x+x^3}-\left(x+\frac{1+3 x^2}{1+x+x^3}\right) f(x)
\end{equation}
$$


for $x \in[0,2]$, with $f(0)=1$.

The true solution is 


$$
\begin{equation}
f(x)=\frac{e^{-\frac{x^2}{2}}}{1+x+x^3}+x^2
\end{equation}
$$

I compared normal networks and fourier networks, the results are shown below.

![FirstOrderODE_Lagaris_problem_1.png](FirstOrderODE_Lagaris_problem_1.png)

I will explain the idea; it is very, very simple.

You want to find out the function $f(x)$, but you only have $f\prime(x)$ and the initial condition $f(a) = A$ and the interval $[a,b]$. It is natural to use $g(x) = f(a) + (x-a) N(x,\theta)$ to approximate $f(x)$, where $N(x,\theta)$ is the neural network. It would be easier to understand why $g(x) = f(a) + (x-a) N(x,\theta)$ can approximate $f(x)$ if you know the interpolation method by using the Lagrange polynomial.

Now, in this case, we have $g(x) = 1 + x N(x,\theta)$.

Since we are using $g(x)$ to approximate $f(x)$, it means that we want $g\prime(x) - f\prime(x) = 0$ for all $x \in [a,b]$​. Hence, the loss function is 


$$
L = \sum_{i=1}^{n} (g\prime(x_i) - f\prime(x_i))^2
$$


where $x_i$ is the point in the interval $[a,b]$.

Basically, we don't get what we want, so we need to set a tolerance for the loss function. If the loss function is smaller than the tolerance, we stop training the neural network.

In our case, $g\prime(x) = N(x,\theta) + x N\prime(x,\theta)$

so the loss function is 


$$
L = \sum_{i=1}^{n} (N(x_i,\theta) + x_i N\prime(x_i,\theta) - f\prime(x_i))^2
$$


I think that is all.

One more thing,the final output is not the output of the neural network, but the output of $g(x) = f(a) + (x-a) N(x,\theta) $.

# Solving second order ODEs with PINNs Example

