+++
date = '2026-04-02T00:00:00+01:00'
draft = false
title = 'Step 2 of Ant Simulation Project : Ant Agent'
math = true
summary = 'Implementation of the ant agent : orientation by angle, movement, reflection on walls, pheromone deposit.'
+++

## Introduction 

In the previous post, I introduced the pheromone grid and the rules behind the pheromones evaporation. Now, we'll look into the **ant agent** that will move around the grid, deposit pheromones, and interact with the environment. 

To do this we have implemented the `Ant` class in the `core/ant.py` file.

---

## An Angle instead of an Index

To represent the ant and its position on the grid, we have two possibilities : either to use the ant position as an index in a two-dimensional array (like the pheromones grids), or to use an angle to represent the direction and position of the ant. 

Instead of using 8 discrete directions, and therefore using 8 `if/elif` statements to determine the next position, we will use an floating angle $\theta \in [0, 2\pi]$. At each time-step the spatial-step is simply : 

$$\Delta x = \cos(\theta + \delta \theta), \Delta y = \sin(\theta + \delta \theta)$$.

Here $\delta \theta$ is a random value sampled from a uniform distribution in $[-\pi/6, \pi/6]$. Its role is only to display the ants movements on the screen. Later, this value will be the output of a neural network.

Then, we only have to take the integer of these values to determine the next position on the surface that we'll give to pygame. 

---

## The Movement of the Ant 

### How do the ants move ? 

The core method of the `Ant` class is `move(delta_theta, put_pheromones, value_pheromone)`. This method will move the ant by modifying the current direction by `delta_theta`, and will also add pheromones if `put_pheromones` is `True`. The `value_pheromone` parameter is the value of the pheromone to be added.

Here we only manage and execute the movement of the ant. The value of `delta_theta` will be the **output of the neural network**.

The new position is then : 

$$x_{t+1} = x_t + \cos(\theta_{t+1}), \quad y_{t+1} = y_t + \sin(\theta_{t+1})$$

### The oscillation of the ants, a normal behavior

This point deserves further explanation. Draft et al. (2018) showed that ants move in an oscillating pattern (sinusoidal, in zigzag). This behavior is not accidental, it is a universal functioning behavior. It exists in species that follow pheromones but also in species like *Cataglyphis bombycina* that rely on vision. 

This oscillation has two purposes : 

- It maximizes the detection surface of the pheromones by sweeping over a wide range of directions. 
- It allows visual corrections to occur at the peaks and valley of the oscillation when the head is stable. 

In our case, we'll begin to simulate ants without vision. This oscillation may or may not be visualized in the simulation. The neural network will have to reproduce this behavior by learning and not by encoded rules. 

--- 

## Rebound on the Walls

Because the simulation is basically a grid, the ants can only move inside this grid. If the new position is outside the grid, we put the ant back on the grid with `numpy.clip`. But the problem is what happens next. If the **direction remains the same** then, the ant will forever be stuck on the edges of the grid. 

To avoid this, the direction is reversed by a simple symmetry along the axis of the edge : 
- **Vertical wall :** $x$ is out of bounds, $\theta \leftarrow \pi - \theta$, we inverse the horizontal component. 
- **Horizontal wall :** $y$ is out of bounds, $\theta \leftarrow -\theta$, we invert the vertical component.

With this method the ant always goes back towards the inside of the grid. 

In the real world, ants have a tendency to **follow the walls (Thigmotaxism)**. This behavior is also not encoded in the ant and we hope that it will emerge and be learned by the neural network if we give it the detection of the wall as an input. 

--- 

## The Pheromones

At each time step, the ant will deposit or not pheromones on the current cell. **We decided to encode which type of pheromones to deposit depending on the situation** : 
- `has_food = True` : the ant has food and deposits a green pheromone leading to the food source. 
- `has_food = False` : the ant has no food and deposits a brown pheromone.

As said in the first article, it is the base of **stigmergy**, the indirect communication between ants via the trails. Again, as already cited in Step 1, this system with two pheromones is biologically inspired : Dussutour et al. (2009) has shown that some species like *Pheidole megacephala* deposit a long lasting pheromone while exploring and another short lasting pheromone  while bringing back food. 

--- 

## The Antennas for the detection of the Gradient of Pheromones

### The Method `get_antenna_pos`

The method `get_antenna_pos` returns the positions of the two antennas. 

$$\text{left antenna} = (x + L \cdot \cos(\theta + \alpha); y + L \cdot \sin(\theta + \alpha))$$
$$\text{right antenna} = (x + L \cdot \cos(\theta - \alpha); y + L \cdot \sin(\theta - \alpha))$$

where $\alpha$ is `ANGLE_ANTENNA` and $L$ is the length of the antenna `LENGTH_ANTENNA`.

The values of the pheromones at these positions will be **inputs of the neural network.**

This bilateral system of gradient detection is a well-documented mechanism in ants. Collett et al. (2025) recall the foundational experiment by Hangartner (1967) on *Lasius fuliginosus* : with one antenna removed, the ant follows the edge of the trail detected by the remaining antenna. With both antennas surgically crossed, the ant becomes unable to follow the trail. These results prove that the ant's brain indeed computes a **bilateral spatial gradient** - it compares the concentration on the left and right and turns toward the maximum. This is exactly what the network inputs in our model will do.

Hangartner (1967) also provides a key quantitative data : the detection threshold for the bilateral gradient is about 1/10. One third of the workers respond to a concentration ratio of 1/10 between the two antennas. This is an order of magnitude useful for evaluating whether the neural network correctly exploits the difference between the two antennal inputs.

### Biological Calibration of the angle $\alpha$ and the length $L$

Draft et al. (2018) provide the most precise quantitative data available on antenna angles in *Camponotus pennsylvanicus*. The relevant parameter for our model is $\theta$, defined as the angle between the body axis and the line connecting the head to the antenna tip -- which is exactly what `ANGLE_ANTENNA` represents in our code.

From the cumulative distributions of $\theta$ in Figure 3C of the paper, the three behavioral modules show distinct distributions :

| Behavioral Module | $\theta$ (approx. median) | Interpretation |
|---|---|---|
| Probing | ~30 deg | Antennas pointing forward, close to the body axis |
| Trail following | ~50 deg | Semi-spread antennas |
| Sinusoidal / Exploratory | ~60-70 deg | Widely spread, nearly perpendicular antennas |

The key data for our model is the **trail following** module. During precise trail tracking, the antennas are excluded from the central zone of approximately 2 mm width (the trail width) and move perpendicularly to the direction of motion, which corresponds to a $\theta$ value of approximately 50 deg. We also know from the paper that the antenna length is approximately 0.5 times the body length (by approximation from the head-to-centroid distance of about 4.0 mm used as normalization unit). 

The current values `ANGLE_ANTENNA = pi/4` (45 deg) and `LENGTH_ANTENNA = 1` (L = 1 represents the distance from the ant's center to the antenna tip, combining approximately half the body length (~0.5) and the antenna length (~0.5 body length)) are therefore a reasonable approximation of trail following behavior.

It could be interesting in a later stage to set `ANGLE_ANTENNA` as an **output of the neural network**, so that the ant dynamically adapts its antenna configuration depending on its behavioral state - exactly as real ants switch between probing, trail following and sinusoidal modules.

--- 

## Stock t - 1 : Model Hypotheses 

In addition to the two values of pheromones at instant $t$ we consider adding the two values of pheromones at instant $t-1$. This would grant the model an explicit access to the **temporal gradient**. 

The question is now, do ants use this type of information ? 

The response from the literature is nuanced. Collett et al. (2025) show, citing Draft et al. (2018) on carpenter ants, that during trail tracking, the antennas oscillate perpendicular to the trail and that brief contact with the trail produces a high temporal concentration gradient. But this temporal gradient **emerges from the ant's oscillating movement** - it is not an explicit memory of a past value. The ant does not "store" $C(t-1)$ : it experiences it through its movement.

**As a result**, storing $t-1$ in our model is an **original modeling hypothesis**, not directly observed in ants. It is inspired by another mechanism and provides the neural network with an additional signal that could be useful. It is an experimental extension of the model, not a biological constraint.

--- 

## The Food, Sources, Collect and Transport

### Two types of Food Sources 

In the model, we consider two types of food sources with each their own purposes : 

- **The Aphids :** A consistent number of ant species have been observed feeding on aphids, a common example is the ginger wood ant *Formica rufa* that raises aphids like cattle. Aphids provide a constant source of sugar for the colony. In the simulation, we hope that ants will build a consistent network of pheromones to guide them towards the aphid source. Each aphid will have a `RECHARGE_RATE` and at each time step, the aphid will stock this amount of food. 

- **The Sugar :** The sugar will represent a non-persistent food source for the ant. If a sugar source is consumed, it will never recharge again. We will be the only one deciding whether or not it is added on the map. We hope that ants will still keep looking for other sources of food instead of aphids to feed their colony.

### The Food Transport

We changed the former model where each ant had a boolean attribute `has_food` to a float attribute that derives better the natural behavior of ants. This attribute is called `food_carried` and `food_carried` $\in [0, MAX FOOD CARRIED]$





This choice is made to highlight that ants do not always carry the maximum amount of food they can hold. For instance, if the food source is almost consumed, the ant goes back with what's left which is compatible with a partially-consumed source. 

### The method `interact()` 

The way the ant thinks is simple and directly inspired from what everyone can observe at their own scale : 

1. **If the ant is on the food source and that she still has food to carry**, she takes whatever she can from the food source in the limit of `FOOD_COLLECT_AMOUNT` and what's left in the source. 
2. **If the ant carries food and that she reaches the nest**, she will deposit the food in the nest. 

### The collect pause : `EAT_DURATION`

When an ant collects food, she stays still for a few time steps. This behavior is directly inspired from nature. Ants don't collect the food right away, they wait a few time steps before moving to the nest. 

--- 

### The Threshold `THRESHOLD_FOOD`

An ant that carries very few food will put `HOME` pheromones and not `FOOD` pheromones. This is directly inspired from the fact that ants don't deposit pheromones if they have very little food. This avoids 2 problems : 

1. **A representation problem :** an ant that carries $0.01$ units of food doesn't really find food, so she needs to keep going and look for food. 

2. **A numerical problem :** instead of comparing units with `==` which is dangerous with float number, we use this threshold to solve the problem. 

---

## The Results 

Here is a video showing how the model that we currently have works : 

--- 

{{< youtube J5r5mvSEyc4 >}}

--- 

On the video, we can see that ants may find the food sources, whether it is sugar or aphids, then they deposit the right pheromone but are not able to find their way back to the nest because we have not implemented the sensory mechanism. Still 2 ants happen to go back to the nest increasing the total number of food collected. Then they deposit again the right type of pheromone. 

### What to do next ? 

The ants currently move and deposit pheromones, but they do so blindly. They cannot yet read the pheromone gradient with their antennas, which means the trail system has no effect on their behavior. This is the core problem to address in the next step.

Step 3 — Colony and Emergence will focus on connecting the sensory inputs to the movement. Concretely, this means reading the pheromone concentrations at the two antenna positions and using that difference to bias `delta_theta`. At this stage, the rule will still be hand-crafted - a simple weighted difference between left and right antenna - but it will be enough to observe the first emergent trails. The goal is to see a collective structure appear from purely local decisions, without any central coordination.

## References

- Collett T., Graham P., Heinze S. (2025). The neuroethology of ant navigation. 
- Draft R.W., McGill M.R., Kapoor V., Murthy V.N. (2018). Carpenter ants use diverse antennae sampling strategies to track odor trails. 
- Hangartner W. (1967). Spezifitat und Inaktivierung des Spurpheromons von Lasius fuliginosus. 
- Dussutour A. et al. (2009). The role of multiple pheromones in food recruitment by ants.  
