+++
date = '2026-03-25T00:53:30+01:00'
draft = false
title = 'Evolutionary Strategies (ES) - Evolutionary Computation Elective by Prof. Dennis Wilson, ISAE-SUPAERO'
math = true 
summary = 'Synthesis of the core concepts of Evolutionary Strategies (ES)'
+++

## Introduction 

Evolutionary Strategies (ES) are a powerful class of stochastic search algorithms that have been used to solve a wide range of optimization problems. While traditional optimization often relies on calculating exact derivatives, ES excels in environments where the objective function is unknown, non-differentiable, or noisy. In the notebook of the elective, we used standard test functions like the **Himmelblau** function or the **Rosenbrock** function. 

## Evolutionary Strategies

Building on what we previously explored in the **Genetic Algorithms** (GA) section, we now dive into **Evolutionary Strategies** (ES). While GA often focuses on discrete populations, ES is the fundamental tool for continuous optimization.

### From Individual Survival to Population Exploration 

As we've seen before : 
- **The (1 + 1) ES :** is the simplest form of ES, a single parent produces one child and the parent is replaced only if the child is better. 
- **The (1 + \lambda) ES :** is a more complex form of ES, a single parent produces $\lambda$ children and the parent is replaced only if the best child is better.

### Exploration vs Speed

While the (1 + 1) ES moves quickly, the (1 + \lambda) ES is more exploratory, which avoid getting stuck in local optima. But be careful, generating too many points (will give you better data on the landscape but :) can become computationally expensive (at least for complex simulations). 

### Approximating the Gradient 

What is brilliant with ES is that it allows us to approximate the gradient of a function **without** calculating its derivative. 

#### Based on Performance

The algorithm samples multiple points around the parent using for example a normal distribution and by observing which offspring perform better than the others, we can identify a promising direction to move in the search space. 

#### Normalization 

In order to make this process robust, we normalize the fitness function. 

#### The Approximation  

After some non intuitive calculus, we can approximate the gradient of the function using the following formula : 

$$ \nabla f = \pm \frac{A \cdot N}{\lambda} $$

with : 

- $N$ is the **matrix of noise vectors** sampled from a normal distribution $N(0, 1)$. Here, for the whole population, $N$ is a matrix of size $[\lambda, d]$ where $d$ is the dimension of the problem. 

- $A$ is the **standardized fitness score** of the population. It is a vector that contains the standardized fitness score of each individual in the population. This means that for each individual $i$, $A_i = \frac{f(x_i) - \mu(f(x))}{\sigma(f(x))}$ where $f(x)$ is the fitness function and $\mu(f(x))$ and $\sigma(f(x))$ are the mean and standard deviation of the population. 

- $\lambda$ is the **number of offspring**.

#### The Learning Rate

Lastly we define the new center of the future normal distribution by : 

$$ x = x + \alpha \frac{A \cdot N}{\lambda} $$

#### The Algorithm 

With all of this we can build our algorithm : 

--- 
**Algorithm 1: Evolutionary Strategies $(\mu, \lambda)$**
```text
Initializing parent x in the search space
Set learning rate alpha and the population size of the offspring lambda and the standard deviation of the noise sigma

For each generation g from 1 to G : 

    # 1. Generating the offsprings 
    N = [N1, N2, ..., Nlambda] from a normal distribution N(0, 1)
    F = [f1=0, f2=0, ..., flambda=0]

    For each offspring i from 1 to lambda : 
        ind_i = x + sigma *N[i]
        F[i] = f(ind_i)

    # Here we can keep track of the best individual in the population. 

    End For 

    # 2. Normalization
    mu_f = mean(F)
    sigma_f = std(F)

    A = [A1, A2, ..., Alambda] = [(f1 - mu_f) / sigma_f, (f2 - mu_f) / sigma_f, ..., (flambda - mu_f) / sigma_f]

    # 3. Approximation of the gradient
    gradient_approx = dot(A, N) / lambda

    # 4. Update parent 
    x = x + alpha * gradient_approx

End For 

Return x or the best individual in the population. 
```
--- 
#### To Summary 

In this approach we turned a blind and random search into a gradient descent. We no longer need to know the derivative of our landscape, we just **feel** where we should move.

### Rank-Based Updates

In ES, **rank-based updates** are a robust method for moving toward a solution by focusing on the relative ordering of individuals rather than their absolute fitness scores. 

#### The concept

Instead of focusing on fitness values to approximate a gradient, the algorithm **sorts the individuals** (offsprings here) from the best to the worst based on their performance. 

- **Selection of $\mu$ individuals :** Usually, not all individuals are useful for finding the center of the search. In the notebook, we typically select the best 50% percent of the population. 

- **Weight update :** Then, we create a set of weights that decreases logarithmically from the best to the worst selected individual. This ensures that the most promising individuals are given more importance and influence on the algorithm moves. 

#### Why 

- **Reducing noises :** By only using ranks, worst individuals do not count as the best ones, and the algorithm is not bothered by individuals with bad performance. 

- **Avoiding local optima :** focusing only on the top individuals prevent from getting pull away from a nearby optimum. 

- **Invariance property :** I didn't really notice this one but : rank-based updates are **invariant to order preserving transformations**. That means if you scale the fitness values (multiply by a scalar take the log), the update remains exactly the same as long as the ranking remains the same. 

#### The Algorithm

Here is the associated algorithm : 

--- 
**Algorithm 2: Rank-based Evolutionary Strategies $(\mu, \lambda)$**

```text
Initializing parent x in the search space
Set the learning rate sigma (or standard deviation) and the population size of the offspring lambda and the number of elites mu 

# Compute the weights based on log rank 
w = [w1 = (log(mu + 1/2) - log(1)), w2 = (log(mu + 1/2) - log(2)), ..., wmu = (log(mu + 1/2) - log(mu))] 
w = w / sum(w)

For each generation g from 1 to G : 

    epsilon = Normal(0, 1, size=(lambda, d)) # where d is the dimension of the problem
    f = [f1, f2, ..., flambda]

    # 1. Generating the offsprings
    For each individual i from 1 to lambda : 

        x_i = x + sigma * epsilon[i]
        f[i] = f(x_i)

    end For 

    # 2. Sorting the offsprings
    sorted_f = argsort(f) # where the best is at index 0
    
    # 3. Approximation of the gradient
    gradient_approx = dot(w, epsilon[sorted_f[:mu]]) = N[first]*w1 + N[second]*w2 + ... + N[last]*wmu

    # 4. Update parent 
    x = x + sigma * gradient_approx

End For 

Return x or the best individual in the population.
```
---

This algorithm is called **Canonical ES**.








