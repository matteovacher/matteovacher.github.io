+++
date = '2026-04-06T12:09:31+02:00'
draft = false
title = 'CMA-ES - Evolutionary Computation Elective by Prof. Dennis Wilson, ISAE-SUPAERO'
math = true
summary = 'Synthesis of the core concepts of the Covariance Matrix Adaptation Evolution Strategy (CMA-ES)'
+++

## Introduction

In the previous section on **Evolutionary Strategies** (ES), we saw how a population of individuals sampled around a center can approximate a gradient without ever computing a derivative. The key limitation, however, is that standard ES uses a fixed, isotropic distribution - a perfect sphere of noise. The algorithm does not learn the shape of the landscape it is exploring.

**CMA-ES** (Covariance Matrix Adaptation Evolution Strategy) solves exactly this problem. Instead of sampling noise from a fixed spherical distribution, it progressively deforms the sampling distribution to match the geometry of the search space.

---

## From ES to CMA-ES

### What does ES miss

In Canonical ES, the offspring are generated as follow :

$$x_i = x + \sigma \cdot \epsilon_i, \text{    with    } \epsilon_i \sim N(0, 1)$$

with :

- $x$ the current center of the population.
- $\sigma$ the step size (standard deviation), a fixed scalar.
- $\epsilon_i$ a noise vector drawn from a standard normal distribution (isotropic, i.e. all directions are likely to be chosen).

The problem is that many real-world landscapes are not isotropic. The optimum might be located at the end of a long, narrow valley -and a spherical distribution will sample mostly useless points on the sides of the valley instead of exploring along it. This is where an ellipsoidal distribution is better to explore the search space. Next question : How can we learn the local geometry of the landscape ? 

CMA-ES addresses this by replacing the fixed identity matrix $I$ with a **covariance matrix** $C$ that learns the local geometry of the landscape over generations.

---

### Rank-Based Weighting

Building on what we saw in ES, CMA-ES still selects the top $\mu$ individuals from a population of $\lambda$ offspring. The key is that it uses **fitness rank** rather than fitness values, which makes the algorithm robust to the scale and shape of the objective function.

The weights associated to the $\mu$ selected individuals follow (as seen in the previous article) a logarithmic decay :

$$w_i = \frac{\ln(\mu + \frac{1}{2}) - \ln(i)}{\sum_{j=1}^{\mu}\left [\ln(\mu + \frac{1}{2}) - \ln(j)\right]}, \quad i = 1, ..., \mu$$

with :

- $w_1 \geq w_2 \geq ... \geq w_\mu > 0$ and $\sum_{i=1}^{\mu} w_i = 1$.
- The best individual (rank 1) gets the most weight, the $\mu$-th individual gets the least but still got one because he belongs to the elites of the population.

This logarithmic scale makes sure that the best individuals drive the direction of update, but every selected individual contributes.

The weighted direction of movement (equivalent to an approximate gradient) is then :

$$s = \sum_{i=1}^{\mu} w_i \cdot \epsilon_{\sigma(i)}$$

with :

- $\epsilon_{\sigma(i)}$ the noise vector of the $i$-th best individual (sorted by rank).
- $s$ the weighted step direction - this is the signal CMA-ES uses to update both the center and the covariance matrix.

---

### The Covariance Matrix

The **covariance matrix** $C$ measures how much the different dimensions of the search space vary together. For instance, if $C_{i, j} = 1$, if the $i$-th coordinate increase, the $j$-th coordinate will increase as well. At initialization, it is set to the identity matrix :

$$C = I$$

which corresponds to an isotropic (spherical) distribution - no dimension is preferred over another. As the algorithm runs, $C$ is updated to reflect the directions in which the selected offsprings tend to be better.

Intuitively, if the best individuals are always found along a particular diagonal direction in the search space, $C$ will elongate the sampling distribution along that direction. To put it more concretely, if the best individuals are always located along a certain axis, the geometrical shape that will determine where to put the next offsprings will highly resemble to a linear ellipse in that direction. 

---

## Updating the Covariance Matrix

### The Rank-One Update

The simplest way to update $C$ is to use the **outer product** of the current step direction $s$ with itself :

$$C \leftarrow (1 - c_i) \cdot C + c_i \cdot s \cdot s^\top$$

with :

- $c_i \approx \frac{2}{n^2}$ the learning rate for the covariance matrix update, where $n$ is the dimension of the problem.
- $s \cdot s^\top$ a rank-one matrix that points in the direction we just moved - it "stretches" $C$ along that direction.
- $(1 - c_i)$ the forgetting factor - old information about $C$ decays slowly to make room for new observations.

Each generation, we add a small piece of information about which direction was productive, and we dilute the past history slightly. By pulling samples from a distribution fit to performant parts of the search space, there is a higher chance of sampling good individuals.

---

## Transforming the Distribution

Once $C$ is no longer the identity matrix, we can no longer simply sample $\epsilon \sim N(0, 1)$ and multiply by $\sigma$. We need to transform the distribution to match the geometry of $C$.

To do this efficiently, CMA-ES uses the **eigendecomposition** of $C$ :

$$C = P \cdot D \cdot P^\top$$

with :

- $P$ the matrix of **eigenvectors** of $C$ - the principal axes of the sampling ellipse. Said otherwise, $P$ is the change-of-basis matrix from the standard orthonormal basis to the eigenvector basis of matrix C
- $D$ a diagonal matrix with the **eigenvalues** of $C$ - the lengths of the principal axes.

A new offspring is then generated as :

$$x_i = x + \sigma \cdot P \cdot D^{1/2} \cdot \epsilon_i, \quad \epsilon_i \sim N(0, 1)$$

Geometrically, $D$ scales the noise and $P$ rotates it by a change of basis. The result is a sample from an ellipsoidal distribution aligned with the most productive directions of the landscape.

*It is in these moments that I highly want to thank my Maths teachers from my Preparatory School : Miss Benhamou (MPSI 1) and Mr Mohan (PSI * 1). By choosing this path I can now focus more on the content of the course on itself rather than on mathematical equations.*

---

## Step-Size Control

Adapting $C$ alone is not sufficient. If $\sigma$ (the overall scale) is too large, the population will fail to converge to the optimum. If it is too small, convergence stalls.

A simple method called the **one-fifth rule** gives the intuition : if more than one fifth of the offspring are better than the parent, the step size is too small and should increase; on the other side, if less than one fifth are better, it is too large and should decrease.

CMA-ES implements a more complicated version of this idea called **Cumulative Step-size Adaptation (CSA)**. Without going into the details of the formula (which involves an evolutionary path tracking the history of recent steps), the principle is the same as the one-fifth rule : the algorithm watches whether recent steps are consistently going in the same direction (increase $\sigma$) or cancelling each other (decrease $\sigma$). The key difference here, is that CSA accumulates this information over several generations rather than just looking at the last one, which makes it more stable. At this stage I would not be able to implement this by myself. 

---

## The CMA-ES Algorithm

Putting everything together :

---
**Algorithm 1: CMA-ES $(\mu, \lambda)$**
```text
Initialize center x, step size sigma, covariance matrix C = I
Initialize evolution paths s = 0, s_sigma = 0, c_i = 2/n^2

# Precompute weights
w = [log(mu + 1/2) - log(i) for i = 1 to mu]
w = w / sum(w)

For each generation g from 1 to G :

    # 1. Eigendecomposition of C
    P, D = eig(C)               # C = P * D * P^T
    D = sqrt(D)

    # 2. Generate lambda offspring
    For each individual i from 1 to lambda :
        epsilon[i] = Normal(0, 1, size=n)
        x_i = x + sigma * P * D * epsilon[i]
        f[i] = objective(x_i)
    End For

    # 3. Sort by fitness (best first)
    sorted_ids = argsort(f)

    # 4. Update center
    # y_i is the actual displacement from the center to offspring i
    y = [sigma * P * D * epsilon[i] for i = 1 to lambda]
    s = sum(w[i] * y[sorted_ids[i]] for i = 1 to mu)
    x = x + s

    # 5. Update covariance matrix 
    C = (1 - c_i) * C + c_i * s * s^T

    # 6. Update step size 
    sigma = csa(...)

End For

Return x or the best individual found
```
---

## In Practice

In the notebook of the elective, CMA-ES is applied to standard benchmark functions : the **Rosenbrock** function (a narrow curved valley, hard for isotropic methods) and the **shifted Rastrigin** function (highly multimodal, with many local optima).

The `pycma` library (actively maintained by Hansen, one of the original authors) provides a production-ready implementation :

```python
import cma
es = cma.CMAEvolutionStrategy(x0 = 2 * [0], sigma0 = 0.1, {'popsize': 20})
solutions = np.array(es.ask()) # ask for new offsprings 
es.tell(solutions, [optf(x[0], x[1]) for x in solutions]) # update the characteristics of the algorithm

# Then repeat for each generation
```

A few observations from the exercises in the notebook :
- On the **Rosenbrock** function, CMA-ES converges quickly because the covariance matrix learns to align with the valley direction.
- On the **shifted Rastrigin** function, success depends heavily on the initial $\sigma$ : too small and the algorithm gets trapped in a local optimum; large enough and CMA-ES can escape and find the global minimum.

---

## Final Remarks

CMA-ES represents a significant improvement over standard ES. By combining rank-based selection, covariance matrix adaptation, and cumulative step-size control, it turns the blind exploration of ES into a **self-adaptation, geometry-aware search**. The algorithm progressively learns not just where to go, but in what shape to explore.

Its main limitation is computational : the eigendecomposition of $C$ costs $O(n^3)$ per generation, which becomes prohibitive in very high dimensions (I don't know when $n$ is too big). 

