# Table of Contents

[Universal function approximator](https://github.com/JIAOJIAOMEI/Universal-function-approximator-and-PINNs?tab=readme-ov-file#universal-function-approximator)


[Fourier feature mapping](https://github.com/JIAOJIAOMEI/Universal-function-approximator-and-PINNs?tab=readme-ov-file#fourier-feature-mapping)


[Different optimizers](https://github.com/JIAOJIAOMEI/Universal-function-approximator-and-PINNs?tab=readme-ov-file#different-optimizers)


[Solving first order ODEs with PINNs Example](https://github.com/JIAOJIAOMEI/Universal-function-approximator-and-PINNs?tab=readme-ov-file#solving-first-order-odes-with-pinns-example)


[Solving second order ODEs with PINNs Example](https://github.com/JIAOJIAOMEI/Universal-function-approximator-and-PINNs?tab=readme-ov-file#solving-second-order-odes-with-pinns-example)

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

def fourier_feature_mappping_functioN(x; \theta):
    return torch.cat([torch.siN(x; \theta), torch.cos(x)], dim=1)


class Fitter(torch.nn.Module):

    def __init__(self, numHiddenNodes):
        super(Fitter, self).__init__()
        self.fully_connected_1 = torch.nn.Linear(in_features=2, out_features=numHiddenNodes)
        self.fully_connected_2 = torch.nn.Linear(in_features=numHiddenNodes, out_features=1)

    def forward(self, x, activation_Function):
        first_layer_output = self.fully_connected_1(fourier_feature_mappping_functioN(x; \theta))
        first_layer_output = activation_Function(first_layer_output)
        y = self.fully_connected_2(first_layer_output)
        return y

```

# Different optimizers

Neural networks are trained by optimizers. These optimizers are used to minimize the loss function.
They are all based on gradient descent, but they have different ways to update the weights.
There is a common choice for the optimizer, which is Adam. If you don't know which optimizer to choose, then Adam.

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

You want to find out the function $f(x)$, but you only have $f\prime(x)$ and the initial condition $f(a) = A$ and the interval $[a,b]$. It is natural to use $g(x; \theta) = f(a) + (x-a) N(x; \theta)$ to approximate $f(x)$, where $N(x; \theta)$ is the neural network. It would be easier to understand why $g(x; \theta) = f(a) + (x-a) N(x; \theta)$ can approximate $f(x)$ if you know the interpolation method by using the Lagrange polynomial or taylor series.

Now, in this case, we have $g (x; \theta) = 1 + x N(x; \theta)$.

Since we are using $g(x; \theta)$ to approximate $f(x)$, it means that we want $g\prime(x; \theta) - f\prime(x) = 0$ for all $x \in [a,b]$. Hence, the loss function is

$$
L (\theta)= \sum_{i=1}^{n} (g\prime(x_i; \theta) - f\prime(x_i))^2
$$

where $x_i$ is the point in the interval $[a,b]$.

Basically, we don't get what we want, so we need to set a tolerance for the loss function. If the loss function is smaller than the tolerance, we stop training the neural network.

In our case, $g\prime(x; \theta) = N(x; \theta) + x N\prime(x; \theta)$

so the loss function is

$$
L (\theta)= \sum_{i=1}^{n} (N(x_i; \theta) + x_i N\prime(x_i; \theta) - f\prime(x_i))^2
$$

I think that is all.

One more thing,the final output is not the output of the neural network, but the output of $g(x; \theta) = f(a) + (x-a) N(x; \theta) $.

# Solving second order ODEs with PINNs Example

Similarly, you want to find out $f(x)$, but you only have $f\prime\prime(x)$ and boundary conditions $f(a) = A$ and $f\prime(a) = A\prime$ and the interval $[a,b]$. We can use

$$
g(x; \theta) = A + A\prime (x-a) + (x-a)^2 N(x; \theta)
$$

to approximate $f(x)$, where $N(x; \theta)$ is the neural network.

**Can't you see it ? $g(x; \theta) = A + A\prime (x-a) + (x-a)^2 N(x; \theta)$ is the first 2 terms of the taylor series of $f(x)$ at $x=a$.**

The Taylor series of a real or complex-valued function $f(x)$, that is infinitely differentiable at a real or complex number $a$, is the power series

$$
f(a)+\frac{f^{\prime}(a)}{1 !}(x-a)+\frac{f^{\prime \prime}(a)}{2 !}(x-a)^2+\frac{f^{\prime \prime \prime}(a)}{3 !}(x-a)^3+\cdots=\sum_{n=0}^{\infty} \frac{f^{(n)}(a)}{n !}(x-a)^n
$$

where $f^{(n)}(a)$ denotes the $n$-th derivative of $f$ evaluated at the point $a$, and $n!$ denotes the factorial of $n$.

Since we use $g(x; \theta)$ to approximate $f(x)$, it means that we want $g\prime\prime(x; \theta) - f\prime\prime(x) = 0$ for all $x \in [a,b]$. Hence, the loss function is

$$
L (\theta)= \sum_{i=1}^{n} (g\prime\prime(x_i; \theta) - f\prime\prime(x_i))^2
$$

where $x_i$ is the point in the interval $[a,b]$.

we have 

$$
g\prime(x; \theta) = A\prime + 2 (x-a) N(x; \theta) + (x-a)^2 N\prime(x; \theta)
$$

$$
g\prime\prime(x; \theta) = 2 N(x; \theta) + 4 (x-a) N\prime(x; \theta) + (x-a)^2 N\prime\prime(x; \theta)
$$

I think this is all for this case.

In another case, we have different boundary conditions, suppose that we have $f(a) = A$ and $f(b) = B$ and the interval $[a,b]$. We can use

$$
g(x; \theta) = A \frac{b-x}{b-a} + B \frac{x-a}{b-a} + (x-a)(b-x) N(x; \theta)
$$

to approximate $f(x)$, where $N(x; \theta)$ is the neural network. It makes sure that when $x=a$, $g(x; \theta) = A$ and when $x=b$, $g(x; \theta) = B$.

The derivative of $g(x; \theta)$ is more complicated, but the idea is the same. In this case, first derivative is

$$
g\prime(x; \theta) = \frac{B-A}{b-a} + (a+b-2x) N(x; \theta) + (x-a)(b-x) N\prime(x; \theta)
$$

and the second derivative is

$$
g\prime\prime(x; \theta) = -2 N(x; \theta) + 2 (a+b-2x) N\prime(x; \theta) + (x-a)(b-x) N\prime\prime(x; \theta)
$$

that is all.

I implemented an example here, this one is trying to solve:

$$
f^{\prime \prime}(x) = -\frac{1}{5} f\prime(x) - f(x) - \frac{1}{5} e^{-\frac{x}{5}} \cos(x)
$$

with exact solution $f(x) = e^{-\frac{x}{5}} \sin(x; \theta)$, $x \in [0, 10]$, $f(0) = 0$, $f\prime(0) = 1$.

![SecondOrderODE_Lagaris_problem_3.png](SecondOrderODE_Lagaris_problem_3.png)

**Activation functions are used to add non-linearity to the neural network.** 
You can think neural network as a black box, f(x) = NN(x; \theta), where NN is the neural network.
Since this world is not linear, or many tasks are not linear, we need to add non-linearity to the neural network so that 
it can approximate more complex functions.
But speaking of non-linearity, there are thousands of non-linear functions with different non-linearity, the activation functions such as ReLU, Sigmoid, Tanh, etc. are just a tiny part of them.
This is why I don't think neural networks can approximate any function; the ability of neural network is somehow limited by the activation functions. Because the non-linearity of neural networks is limited by the activation functions, the neural network can only approximate a certain range of functions, just a personal opinion.