+++
date = '2026-03-28T00:00:00+01:00'
draft = true
title = 'Step 1 of Ant Simulation Project : Pheromones'
math = true
summary = 'Building the pheromone grid : the substrate every ant reads from and writes to, grounded in biological and mathematical models.'
+++

## Introduction

In the previous post, I introduced the project and set up the technical environment. Now it is time to build the first real component of the simulation : the **pheromone grid**.

Before introducing any form of ant in the code, we need to build the substrate the ant will interact on. Each individual ant has a tiny brain, they don't act because the queen gave them an order. The queen itself, in fact has a purpose and a role, lay eggs, and not give them an order. Yet, they collectively build optimized trail networks and adapt to disruptions in seconds. The trick is that they do not need to talk to each other (depending on the species of course), they **write down the information on the ground and read it back, all from a shared memory (chemical memory)**.  

Reproducing this in simulation means building a data structure that every agent perceives, every agent modifies, and that evolves according to its own physical laws (that we have to respect) between agent interactions. That is the pheromone grid.

---

## Two Layers for One Grid

In real ant colonies, chemical communication does not rely on a single signal but on a combination of pheromones with complementary roles. Dussutour et al. (2009) experimentally showed that some species deposit a long-lasting pheromone while exploring their territory -  even in the absence of food - and a short-lasting, much stronger pheromone when returning to the nest with food. The first one builds a collective memory of the environment, while the second triggers targeted recruitment towards an identified food source. This distinction, grounded to real biology, justifies the use of two separate pheromone in the simulations. Rather than just simplifying the system to a single signal, keeping this duality opens the door to richer collective behaviors - and gives the neural network the freedom to exploit this complexity on its own. If possible we'll be able to implement a more complex system of pheromones. For now, we'll focus on the two following signals :

- **HOME pheromone :** left by ants leaving the nest, marking the explored territory.
- **FOOD pheromone :** left by ants carrying food, marking the route toward food sources for other individuals.

Together, these two signals are enough to produce the core emergent behavior we are looking for : the formation of efficient foraging trails from a colony of individually simple agents.

First, the grid is implemented as a single NumPy array of shape `(2, HEIGHT, WIDTH)`. The first axis selects the pheromone type; the two remaining axes are the spatial dimensions (2D). Keeping both layers in a single object is really convenient since it will reduce the time and memory complexity of the simulation : operations that apply to all pheromone types simultaneously, evaporation, diffusion, can be expressed as a single array operation with no Python loop which is much more efficient.

```python
self.grids = np.zeros((2, GRID_HEIGHT, GRID_WIDTH))
```

The two types of pheromones are called via class constants `HOME = 0` and `FOOD = 1`. All of this will allow me to review the code and the structure faster later : `grids[PheromoneGrid.HOME]` is simple in a way that `grids[0]` is not. 

---

## The Physics of Pheromones

Once deposited, a pheromone does not stay at the deposit point forever. To represent that, we use two physical processes shaping its evolution over time : **evaporation** and **diffusion**. Getting these two processes right is the core scientific work of this step of the simulation.

### Evaporation

First, evaporation is the simplest of the two to implement. Over time, we can simply imagine that the pheromone concentration decreases due to chemical degradation, substrate absorption and air condition. Then, what is the mathematical form of this decay ?

Edelstein-Keshet et al. (1995) gave us the answer. They use a **first-order kinetic model** (an exponential decay) for pheromone evaporation, it is for them the most convenient way of representing decay :

$$\frac{dC}{dt} = -r \cdot C$$

This is the same equation that governs radioactive decay, capacitor discharge, and countless other physical systems. Its solution is the exponential :

$$C(t) = C(0) \cdot e^{-rt}$$

Then we discretize this equation into a geometrical solution : 

$$C_{t+1} = C_t (1 - \rho)$$

In my code, `EVAPORATION_RATE` stores the factor $(1 - \rho)$, so that the update is a single multiplication without any loop over the entire 3D array :

```python
self.grids *= EVAPORATION_RATE
```

What makes this result practically useful is that Edelstein-Keshet also measured $r$ for several species, we'll see in the future if we use the different values : 

| Species               | Time of decay   | Rate $r$ in $s^{-1}$|
|-----------------------|-----------------|---------------------|
| Atta texana           | > 6 days        | very small          |
| Eciton burchelli      | 2 to 8 days     | $4 \times 10^{-7}$  |
| Iridomyrmex humilis   | ~30 min         | $5 \times 10^{-4}$  |
| Solenopsis saevissima | 104 s to 20 min | 0.008 to 0.12       |
| Myrmica rubra         | 2 to 3 min      | 0.005 to 0.008      |
| Pogonomyrmex badius   | ~35 sec         | 0.028               |

These values describes five orders of magnitude. Then, the choice of the species directly set the entire timescale of the simulation. Furthermore, we have to decide which configuration of the evaporation and diffusion we want to use because it will describe a certain species and set the time of the simulation. For example, a slow-evaporating species like *Atta texana* produces persistent trails while a fast-evaporating species like *Pogonomyrmex badius* produces short signal that requires a constant reinforcement. 

### Diffusion

But there is also another process that takes place : **Diffusion**. 

Molecules spread to the surrounding cells by diffusion, creating a spatial gradient that ants can detect and follow. This process is called **chemotaxis** and it is how ants detects trails. 

The governing equation is the classical diffusion (Partial Differential Equation) :

$$\frac{\partial C}{\partial t} = D \cdot \nabla^2 C$$

where $D$ is the diffusion coefficient. Solving this numerically is computationally expensive. 

To implement the diffusion of pheromones, we choose to use a **Gaussian filter** because of the fundamental mathematical equivalence between linear diffusion and a Gaussian convolution. As highlighted in Prof. Daniel Cremers' lectures (Variational Methods for Computer Vision), the solution to the linear diffusion equation is not just approximated by a Gaussian filter, it is the **exact solution**. 

This implementation strategy is a well-established standard in scientific computing : 

- **Weickert et al. (1998)** validate this solution. 

This method also guarantees us that : 

- **Mass conservation :** the total amount of pheromone in the grid is unchanged by diffusion alone. 
- **Convergence :** without any deposit, the grid will converge to a uniform concentration and then decay to zero with evaporation. 
- **Maximum-minimum principle :** no new concentration values are created.

Here is the implementation of the gaussian filter in the code (we will find the appropriate parameters later): 

```python
gaussian_filter(self.grids, 
    sigma = DIFFUSION_SIGMA,
    order = 0, 
    output = self.grids,    
    mode = 'constant', 
    cval = 0.0, 
    truncate = 7.0, 
    radius = None, 
    axes = (1,2) 
    ) 
```

Finding the good parameters for the diffusion process is a real challenge, in the literature, most of the models mix the evaporation and the diffusion processes, both Edelstein et al. (1995) and Watmough et al. (1995) do not consider the diffusion process by itself and therefore $D$ remains a free parameter. In order to find the good parameters, I will have to test different configuration and find the best trade off that respect the biological data find in the literature. 

The `axes=(1, 2)` argument deserves attention. Without it, `gaussian_filter` would treat the array as a 3D object and blur along all three axes - including the first one, which separates HOME and FOOD. The result would be a mix of the two pheromone types, which is physically wrong. Restricting the filter to the spatial axes only is mandatory and will allow the two grid to not mix together.

### The Complete Update

Combining both processes, the full pheromone update at each time step is :

$$C_{t+1} = \text{GaussianFilter}_\sigma\\left(C_t \cdot (1 - \rho)\right) + \text{Deposit}_t$$

Evaporation happens first, then diffusion spreads what remains, then new deposits are added by the ants. This ordering is a modeling choice : it means diffusion propagates the already-decayed signal. The alternative (diffusion then evaporation) is also possible. Neither is strictly more biologically correct - both of them are physically correct.

A final threshold removes values below $10^{-4}$ to prevent noise from accumulating over thousands of iterations and from making the calculations less stable :

```python
self.grids[self.grids < 0.0001] = 0
```

---

## From Grid to Screen

The pheromone grid is internal state - to observe it during development, we need to convert it to pixels. The `Environment` class handles this with a fully vectorized approach (RGB approach). Each cell on the grid is converted to a triplet of RGB values. R for red, G for green, B for blue. 

The rendering uses a **dominant pheromone** logic : at each cell, whichever concentration is higher determines the pixel color. HOME maps to one configured color (brown), FOOD to another (green), and the brightness scales linearly with concentration (The pheromone values are scaled from $0$ to $1$). The key is that this is done with NumPy operations over the entire grid at once - no Python loop over cells which is much slower than vectorized operations .

```python
home_stronger = home > food
red   = np.where(home_stronger, home * COLOR_HOME[0], food * COLOR_FOOD[0])
green = np.where(home_stronger, home * COLOR_HOME[1], food * COLOR_FOOD[1])
blue  = np.where(home_stronger, home * COLOR_HOME[2], food * COLOR_FOOD[2])
```

In addition, NumPy indexes arrays as `(row, column)`, which equals to `(y, x)`. Pygame's `surfarray`, however, expects `(width, height)`, which equals to `(x, y)`. A transposition is mandatory before passing the array to pygame, otherwise the entire image would be rotated by 90 degrees :

```python
pygame.surfarray.make_surface(np.transpose(env_surface, (1, 0, 2)))
```

Please note that we only transpose the array on the first and second dimensions, not the third since this one derives the color of the pixel. 

---

## Testing Without Ants

Since no ant agent exists yet, how do we verify the grid behaves correctly ? The answer is in `tests/tests_pheromones.py`. It reads mouse input each frame via `pygame.mouse.get_pressed()` and deposits pheromones wherever the cursor is held down. Left click is for HOME, right click is for FOOD.

This is enough to check if the grid is updated correctly : 
- Deposits appear at the correct location.
- Evaporation gradually turns the signal off over time, proportionally to concentration.
- Diffusion spreads the concentration in a smooth gradient - visible as the color spreads away from the deposit point each frame.
- The nest is displayed above the pheromones. 

{{< youtube objAaduVhqo >}}

What we can already observe at this stage is that the two pheromones spreads and evaporate independently of each other.

---

## What Remains Open

A few points need to be resolved before this step is fully validated :

**Diffusion coefficient $D$**: No biologically calibrated value exists for ant trail pheromones. It will be determined empirically by matching the simulated active space to the biological literature values.

**Deposit model for agents**: The classic ACO (Ant Colony Optimization) formula $\Delta\tau = Q / L_k$ (Dorigo et al. 2000) was derived for graph-based ACO, not for 2D spatial grids. An adapted deposit model will be needed once ant agents are introduced in Step 2. The biological evidence (Watmough and Edelstein-Keshet, 1995) suggests a deposit amount on the order of $0.6 \times C_s$, where $C_s$ is the saturation concentration, as a starting point. Despite the fact that this is a simplification, it is a good starting point. 

---

## Summary

The pheromone grid is a `(2, H, W)` NumPy array updated each tick by three sequential operations: geometrical decay (evaporation, derived in Edelstein-Keshet's first-order kinetic model), Gaussian convolution (diffusion, exactly equivalent to solving the linear PDE), and addition (deposit). The rendering converts concentration values to an RGB image using vectorized NumPy operations, with a transposition required for pygame Surface compatibility.

The grid is now physically coherent and visually observable. The next step is to populate it with agents.

---

## References

- Edelstein-Keshet L., Watmough J., & Ermentrout G. B. (1995). Trail following in ants:
  individual properties determine population behaviour.

- Weickert J., ter Haar Romeny, B. M., & Viergever M. A. (1998). Efficient and reliable
  schemes for nonlinear diffusion filtering.


- von Thienen W., Metzler D., Choe D.-H., & Witte V. (2014). Pheromone communication
  in ants: a detailed analysis of concentration-dependent decisions in three species.


- Dorigo M., Bonabeau E., & Theraulaz G. (2000). Ant algorithms and stigmergy.

- Watmough J., & Edelstein-Keshet L. (1995). Modelling the Formation of Trail Networks by Foraging Ants. 

- Dussutour A., Nicolis S.C., Shepard G., Beekman M., & Sumpter D.J.T. (2009). The role of multiple pheromones in food recruitment by ants. 