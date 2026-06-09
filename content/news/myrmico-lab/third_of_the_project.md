+++
date = '2026-04-06T00:00:00+01:00'
draft = true
title = 'Step 3 (Perhaps Final) of Ant Simulation Project : Colony and Emergence'
math = true
summary = 'Study of the design of the colony and emergence of collective behaviors.'
+++

## Introduction

***Careful : This is a really long article. If you want to see the videos of the simulation, all of them will be displayed below. I also want to add that this article will probably be the last of this series since it took me an incredible amount of time and that I also want to explore other things in the future. I am now in Tsukuba and I am looking forward writing an article about my current project. I also want to master C++. I guess all of this will take me a lot of time since I am really dedicated on these projects. Anyway this series gave me a first glimpse of what research could look like and I really like it. It also gave me the opportunity to master a few tools I never had the chance to use before. I am writing these lines after having finish this article and the videos. The only video that will be displayed in the introduction is the video where everything works :)***

{{< youtube QgjBLVlBrh0 >}}

--- 

In the previous post, we built the `Ant` class : movement by angle, rebound on the walls, pheromones deposit and antennas positions. But the antennas were only computed, not used. The ants were walking randomly and did not read the pheromones they were depositing.

Step 3 is about connecting the sensory inputs to the movement. The rule is still hand-crafted - a simple weighted difference between the left and the right antenna - but it is enough to observe the **first emergent trails**. The goal of this step is to see a collective structure appear from purely local decisions, without any central coordination.

The path to get there was not straight. Many parameters were tested, many things did not work, and the model itself evolved during the process (2 antennas became 3, a single evaporation rate became two, the deposit rule was rewritten). The article below describes the final state of the model and the reasoning behind each choice.

This article, by its complexity, is a really long one, and if you are more interested in the results than in the process (which I totally understand), I highly recommend you to skip all this article to directly go see the videos at the end of it. 

---

## The Starting Point

Before describing the final model, it is useful to show where Step 3 actually started. Here is the `config.py` at the very beginning of this step, before any of the changes described below :

---

```python
# GRID AND PHEROMONE 
EVAPORATION_RATE = 0.997                # between 0 and 1, the higher the slower the evaporation
DIFFUSION_SIGMA = 0.3                   # between 0 and inf, the higher the more the pheromone spreads, but also the more it evaporates
GRID_WIDTH = 360                        # width of the grid in cells, also in pixels if cell size is 1, max 2880 for 8k screen
GRID_HEIGHT = 240                       # height of the grid in cells, also in pixels if cell size is 1, max 1920 for 8k screen
CELL_SIZE = 3                           # size of each cell in pixels
FPS = 24                                # frames per second
WINDOW_WIDTH = GRID_WIDTH*CELL_SIZE     # width of the window = width*cellsize (2880 pixels max )
WINDOW_HEIGHT = GRID_HEIGHT*CELL_SIZE   # height of the window = height*cellsize (1920 pixels max )
PHEROMONE_DEPOSIT = 0.7                 # amount of pheromone deposited by an ant at each step, between 0 and 1

# ENVIRONMENT
COLOR_HOME = (139, 90, 43)              # brown color for the pheromone leading to the nest
COLOR_FOOD = (50, 205, 50)              # green color for the pheromone leading to the food
COLOR_BACKGROUND = (0, 0, 0)            # black color for the background
COLOR_NEST = (255, 255, 255)            # white color for the nest
NEST_RADIUS = 2                         # radius of the nest in cells
NEST_X = 100                            # coordinates of the nest
NEST_Y = 100                            # coordinates of the nest

# ANT
N_ANTS = 20                             # number of ants in the simulation, must be an integer greater than 0
LENGTH_ANTENNA = 0.5                    # length from the head to the tip of the antenna in cells, must be greater than 0
ANGLE_ANTENNA = np.pi/4                 # angle between the direction of the ant and the direction of the antenna in radians, between 0 and pi/2, if 0 then antennas are in the same direction as the ant, if pi/2 then antennas are perpendicular to the direction of the ant
COLOR_ANT = (255, 165, 0)               # orange color for the ants
MAX_FOOD_CARRIED = 0.5                  # maximum amount of food an ant can carry, between 0 and 1, if 0.5 then an ant can carry half of a food source
FOOD_COLLECT_AMOUNT = 0.5               # amount of food an ant can collect at one time, between 0 and MAX_FOOD_CARRIED
EAT_DURATION = 8                        # number of steps an ant needs to eat a food source, during this time the ant cannot move or interact with other food sources, must be an integer
ANTENNA_WEIGHT = np.pi/3                # weight of the pheromone bias on the ant's direction, between 0 and pi/2, if 0 then the ant ignores the pheromones, if pi/2 then the ant turns directly towards the strongest pheromone
TRESHOLD_FOOD = 0.45                    # threshold of food carried for an ant to switch from following home pheromone to following food pheromone, between 0 and MAX_FOOD_CARRIED
RANDOM_DIR = np.pi/8                    # maximum random change in direction for an ant at each step, between 0 and pi, if 0 then the ant never changes direction randomly, if pi then the ant can turn in any direction at each step
HALF_LENGTH_BODY = 0.5                  # half of the length of the ant's body in cells, used for drawing the ant as a line before adding the antenna 

# FOOD SOURCES 
N_FOOD_TYPES = 2                        # number of different types of food sources, must be an integer greater than 0
COLOR_APHID = (255, 220, 0)             # color yellow for aphids, which are a type of food source that can recharge 
COLOR_SUGAR = (100, 200, 255)           # color blue for sugar, which is another type of food source that does not recharge
RECHARGE_RATE_APHID = 0.01              # recharge rate for aphids, between 0 and 1
RECHARGE_RATE_SUGAR = 0                 # recharge rate for sugar, must be 0 since sugar does not recharge
```

---

From this starting configuration, many things evolved. One of the very first changes was about the behavior at the food source. In the initial version, once an ant had finished eating, she just kept walking in the **same direction** she was heading before, and simply crossed the food source without turning back to the nest. To fix this, I added an automatic **U-turn right after food collection**, so that the ant now faces the opposite direction when she leaves the source. It is a small change, but it is the one that unlocked the very first complete `nest -> food -> nest` cycles.

---

## A Simple Angular Rule

The ant has two antennas, left and right. Each one reads the pheromone concentration at its position. If the concentration on the left is higher, the ant should turn to the left, and the opposite if the right is higher. The simplest way to translate this into code is a weighted difference :

$$\delta\theta = W \cdot (C_L - C_R) + U(-\delta_r, \delta_r)$$

where $W$ is `ANTENNA_WEIGHT`, $C_L$ and $C_R$ are the concentrations at the left and right antenna, and the last term is a uniform random noise of amplitude `RANDOM_DIR`. The noise is here to keep an exploration component - without it, the ants would stick to the strongest trail and never find new food sources.

**Which pheromone to follow ?** An ant with food wants to go back to the nest, an ant without food is looking for a source. So the rule is simple :

- `food_carried > THRESHOLD` -> follows the `HOME` pheromones
- `food_carried <= THRESHOLD` -> follows the `FOOD` pheromones

This hand-crafted rule is not the final objective. The final objective is to replace it with a neural network that learns the mapping from sensory inputs to movement. But before that, we need a baseline that works - something good enough to produce emergent trails, so we can study them and later compare them with what the network produces.

---

## From Two Antennas to Three

The left/right rule alone is not enough. In straight lines it works fine, but in a curve the ant often leaves the trail. My hypothesis is that during the turn the two antennas miss the trail at the same time : the trail is thin, and if none of the two antennas happens to be on it, the ant reads zero on both sides. The angular bias becomes null, only the random noise remains, and the ant drifts away from the path.

To fix that, a third antenna is added in front of the ant. The rule becomes :

- if $C_F \geq \max(C_L, C_R)$ -> the concentration is highest in front, so we do not bias the direction and the ant keeps going straight (up to the random noise).
- otherwise -> we apply the same left/right weighted difference as before.

This is inspired by Draft et al. (2018). They show that during trail tracking, carpenter ants oscillate their antennas perpendicularly to the trail. Each sweep of the head produces a **temporal concentration gradient** : a short moment of high concentration when the antenna crosses the trail, followed by a lower signal. In our model the oscillation is not explicit, but the front antenna plays the same role : it tells the ant "you are still on the trail, keep going".

One more geometric detail. At first, the antennas were starting from the center of the ant, which is biologically wrong : the real antennas start from the head, not from the middle of the body. A new constant `HALF_LENGTH_BODY` was added so that the antennas start at the head position, computed as `(x + HALF_LENGTH_BODY * cos(theta), y + HALF_LENGTH_BODY * sin(theta))`.

---

## A Step-Counting Deposit

Until now, an ant was depositing a constant amount of pheromone at each step. The problem is that the trail looks the same everywhere along the path : no information about the distance to the source or to the nest. If a network (or a hand-crafted rule) only reads the concentration, it cannot tell if it is close to the source or far from it.

The solution used here is to make the deposit **decrease with the number of steps** since the last reset (arrival at the nest, or collection of food). At each step, the deposit value is multiplied by a factor $\lambda < 1$ :

$$\text{deposited}_{t+1} = \lambda \cdot \text{deposited}_t$$

In the code, $\lambda$ is the constant `DECAY_FACTOR_STEP`. This produces a gradient of intensity along the trail : strong at the start (just after the nest or just after the food), weak at the end. An ant that reads a rising concentration is moving toward the source of the trail, an ant that reads a falling concentration is moving away from it.

Biologically, this is not completely arbitrary. Wittlinger et al. (2006), in their "stilts and stumps" experiment, show that ants count their steps to estimate the distance walked. Collett et al. (2025) cite this as one of the core mechanisms used by ants for navigation. So a deposit that depends on the number of steps since the last event is a reasonable inspiration, even if the real chemical process is more complex than a simple geometric decay.

One important nuance here : the Wittlinger experiment was done on *Cataglyphis*, the desert ant. The stride-counting mechanism (a pedometer-like integrator of the steps) is specific to this genus, which is adapted to navigate in open terrain with almost no pheromone trails and almost no visual landmarks (except perhaps the sun). Applying this mechanism directly to forest ants or to *Pheidole* species is therefore a **simplification** : I keep it here because the idea of a distance-dependent deposit is useful for the model, not because every ant species really does this.

---

## Two Pheromones, Two Dynamics

In Step 1 we introduced two layers of pheromones, `HOME` and `FOOD`, but they were sharing the same evaporation rate and the same diffusion sigma. After many simulations it became clear that this symmetric treatment was not the right choice.

The main inspiration comes from Dussutour et al. (2009), who studied *Pheidole megacephala* and described how the colony uses two chemical signals of very different nature. One is a long-lasting exploration signal, deposited in many places across the territory, which works as a slow memory of where the ants have been. The other is a short-lived and stronger recruitment signal, deposited by ants returning from a food source, which triggers a fast collective response. A personal discussion with Guy Theraulaz later confirmed this reading and added a useful nuance : the `HOME` signal in our model is probably closer to a passive body odor trail (cuticular hydrocarbons) than to a true recruitment pheromone.

Translated into the code, this gives two separate constants for each process :

```python
EVAPORATION_RATE_HOME = 0.999
EVAPORATION_RATE_FOOD = 0.996
DIFFUSION_SIGMA_HOME = 0.25
DIFFUSION_SIGMA_FOOD = 0.275
```

The `HOME` trail lasts longer and is slightly less diffused, which fits the role of a stable map of the colony's territory. The `FOOD` trail is more volatile and spreads a bit more, which makes sense for a signal that is supposed to attract other ants quickly and then disappear when the source is entirely consumed.

---

## A More Realistic Deposit Rule

The first version of the deposit was the simplest possible : at each step, we add `value` to the cell and clip the result to 1.

```python
self.grids[type_of_pheromone, y, x] = min(1, self.grids[type_of_pheromone, y, x] + value)
```

On paper it works, but in the simulation the cells crossed by many ants saturate to 1 almost immediately and stay there forever. Everywhere the ants pass often, the concentration is a flat surface at 1, with no gradient left for the antennas to read. 

A better rule is to deposit an amount that depends on how much space is still available on the cell. The new formula is :

$$C_{t+1} = C_t + value \cdot (1 - C_t)$$

where $v$ is the amount the ant wants to deposit and $C_t$ is the current concentration. When the cell is empty ($C_t = 0$), we add the full $value$. When the cell is full ($C_t = 1$), we add nothing. In between, the deposit is proportional to the remaining margin $(1 - C_t)$. The concentration converges asymptotically to 1 without ever getting stuck on the saturation surface.

```python
current = self.grids[type_of_pheromone, y, x]
self.grids[type_of_pheromone, y, x] = min(1, current + value * (1 - current))
```

The `min(1, ...)` stays as a safety, but mathematically the sum cannot exceed 1 anymore.

Beyond the realism argument, there is also a secondary benefit that is relevant for the bug we observed before : when two `HOME` trails arrive at the nest from two different directions, their cells near the nest used to stay at 1 in a large saturated zone. Diffusion then spread this surface into neighbor cells on both sides, which is how the two trails were merging into a single wide zone where the ants could not distinguish one path from the other. With the new rule, cells near the nest no longer saturate permanently (or at least not immediately), so the diffusion from these cells injects less pheromone into the neighbors and the two trails stay more distinct for longer.

Honestly, I do not think this will fix the whole problem. The main reason why ants get lost on a merged trail is probably the simplicity of the model itself : the ant has no memory, and only reads three scalar values at each step. If two trails merge, the left/right/front rule just follows the strongest gradient and does not care about where this gradient actually leads. The deposit rule change is a real improvement, but the deeper problem will probably only be solved when the hand-crafted rule will be replaced by a network that can learn to integrate information over time.

---

## Separating Physics from Rendering

At the start of the project, ants and food sources were displayed at their real size inside the simulation, which means very small : a few pixels wide. It was hard to see what was going on, and I was spending a lot of time with my face stuck to the screen trying to follow the trajectories by eye.

At some point I decided to separate the visual rendering from the physical simulation. The ants and the aphids are now drawn much larger on the screen than they actually are in the underlying grid. From a readability point of view, this makes observation much more comfortable. It also has a side effect that was not planned at the beginning but turned out to be useful.

The side effect is about how the ant detects a food source or the nest. The detection is based on a circle : the aphid has a radius, the nest has a radius. An ant is considered "on" a food source when the **center of the ant** (not its drawn body, not its rays of perception) enters the circle of the source. Same rule for the nest : the center of gravity of the ant has to enter the circle of the nest for the food to be delivered. This is a simple geometric test, independent of the visual size.

Before this change, the detection was more fragile : ants were sometimes visually on the food or on the nest but not triggering the interaction because of small rounding issues. With the new rule, the detection is clean and happens every time the center enters the circle. The collection rate and the nest return rate both improved after this change, without changing anything in the behavior of the ants themselves.

---

## A Calibration Tool for the Parameters

The number of parameters grew quickly during this step : two evaporation rates, two diffusion sigmas, one deposit decay factor, the antenna length, the antenna angle, the random noise amplitude, the bias weight, and so on. Each time one of them is changed, it is not always obvious whether the new configuration is still in a biologically reasonable range or not.

Two values in particular were the real blocker for me at the beginning of the step, and I wanted a fast way to read them before each run :

- the **antenna separation**, which is the distance between the left and the right antenna tips, given `LENGTH_ANTENNA`, `ANGLE_ANTENNA` and `HALF_LENGTH_BODY`. If the two antennas are too close, they read almost the same concentration and the left/right difference is always close to zero. If they are too far apart, one of them is always outside the trail and no useful signal can be extracted.
- the **effective sigma** of a trail at its half-life, which combines evaporation and diffusion into a single number that tells how wide a trail actually looks on the grid at a typical moment of its life. This is the width that the antenna separation has to be matched against.

These two geometric values were the main reason I asked for this script, but they were not the only ones. I also asked for most of the other metrics because I already knew more or less which parameters would be important. For example, a signal-to-noise ratio between the angular bias and the random noise was obviously needed, but I did not know exactly how to formulate the signal part of the ratio, so I let the LLM take care of the exact expression. I checked the formulas afterwards and they were consistent. Being in an engineering school, I can usually tell when a derivation is correctly done, even if I did not write it myself :)

The script does not run any simulation, it only reads `config.py` and prints the derived values. It was built to display cleanly in the terminal so that I can evaluate all parameters within a few seconds.

The interesting part here is not the formulas themselves but the fact that I could run this script many times during a debug session, instead of launching dozens of simulations in the void and guessing what was wrong from the visuals alone.

---

## The Final Parameters

After all the iterations described in the previous sections, here is the `config.py` that the simulation is running on now. These are the values that were finally adopted for the current state of the project :

---

```python
import numpy as np 

# GRID AND PHEROMONE 
EVAPORATION_RATE_HOME = 0.999           # between 0 and 1, the higher the slower the evaporation
EVAPORATION_RATE_FOOD = 0.9985          # between 0 and 1 
DIFFUSION_SIGMA_HOME = 0.25             # between 0 and inf, the higher the more the pheromone spreads, but also the more it evaporates
DIFFUSION_SIGMA_FOOD = 0.275            # between 0 and inf, must be greater than sigma home because evap is less for home 
GRID_WIDTH = 600                        # width of the grid in cells, also in pixels if cell size is 1, max 2880 for 8k screen
GRID_HEIGHT = 400                       # height of the grid in cells, also in pixels if cell size is 1, max 1920 for 8k screen
CELL_SIZE = 2                           # size of each cell in pixels
FPS = 45                                # frames per second
WINDOW_WIDTH = GRID_WIDTH*CELL_SIZE     # width of the window = width*cellsize (2880 pixels max )
WINDOW_HEIGHT = GRID_HEIGHT*CELL_SIZE   # height of the window = height*cellsize (1920 pixels max )
PHEROMONE_DEPOSIT = 1                   # amount of pheromone deposited by an ant at each step, between 0 and 1

# ENVIRONMENT
COLOR_HOME = (139, 90, 43)              # brown color for the pheromone leading to the nest
COLOR_FOOD = (50, 205, 50)              # green color for the pheromone leading to the food
COLOR_BACKGROUND = (0, 0, 0)            # black color for the background
COLOR_NEST = (255, 255, 255)            # white color for the nest
NEST_RADIUS = 20                        # radius of the nest in cells
NEST_X = int(GRID_WIDTH/2)              # coordinates of the nest, must be an integer 
NEST_Y = int(GRID_HEIGHT/2)             # coordinates of the nest, must be an integer 

# ANT
N_ANTS = 200                            # number of ants in the simulation, must be an integer greater than 0
LENGTH_ANTENNA = 1.5                    # length from the head to the tip of the antenna in cells, must be greater than 0
ANGLE_ANTENNA = np.pi/3                 # angle between the direction of the ant and the direction of the antenna in radians, between 0 and pi/2, if 0 then antennas are in the same direction as the ant, if pi/2 then antennas are perpendicular to the direction of the ant
COLOR_ANT_HOME = (255, 165, 0)          # orange color for the ants exploring 
COLOR_ANT_FOOD = (0, 200, 80)           # different green color for ants carrying food 
MAX_FOOD_CARRIED = 0.5                  # maximum amount of food an ant can carry, between 0 and 1, if 0.5 then an ant can carry half of a food source
FOOD_COLLECT_AMOUNT = 0.5               # amount of food an ant can collect at one time, between 0 and MAX_FOOD_CARRIED
EAT_DURATION = 8                        # number of steps an ant needs to eat a food source, during this time the ant cannot move or interact with other food sources, must be an integer
ANTENNA_WEIGHT = np.pi/2.5              # weight of the pheromone bias on the ant's direction, between 0 and pi/2, if 0 then the ant ignores the pheromones, if pi/2 then the ant turns directly towards the strongest pheromone
LIM_ANGLE = np.pi/3                     # maximum angle an ant can turn at each step
TRESHOLD_FOOD = 0.45                    # threshold of food carried for an ant to switch from following home pheromone to following food pheromone, between 0 and MAX_FOOD_CARRIED
RANDOM_DIR = np.pi/25                   # maximum random change in direction for an ant at each step, between 0 and pi, if 0 then the ant never changes direction randomly, if pi then the ant can turn in any direction at each step
HALF_LENGTH_BODY = 1.5                  # half of the length of the ant's body in cells, used for drawing the ant as a line before adding the antenna 
ANT_RADIUS = 2                          # must be integer for arrays and only for the visual here 
DECAY_FACTOR_STEP = 0.995               # decay factor applied to the pheromone deposit at each step, between 0 and 1, slowly decrease the amount of pheromone deposited 
NEST_DURATION = 12                      # number of steps an ant needs to stay at the nest after depositing food, during this time the ant cannot move or interact with food sources, must be an integer    
NEST_ANGLE_VARIATION = np.pi/3          # maximum random change in direction for an ant when leaving the nest, between 0 and pi, if 0 then the ant leaves the nest in a straight line, if pi then the ant can leave the nest in any direction

# FOOD SOURCES 
N_FOOD_TYPES = 2                        # number of different types of food sources, must be an integer greater than 0
COLOR_APHID = (255, 220, 0)             # color yellow for aphids, which are a type of food source that can recharge 
COLOR_SUGAR = (100, 200, 255)           # color blue for sugar, which is another type of food source that does not recharge
RECHARGE_RATE_APHID = 0.01              # recharge rate for aphids, between 0 and 1
RECHARGE_RATE_SUGAR = 0                 # recharge rate for sugar, must be 0 since sugar does not recharge
FOOD_RADIUS = 3                         # radius of the food source in cells, must be an integer
```

---

Compared to the starting config at the top of the article, almost every value has changed : the grid is larger, the number of ants is ten times higher, the deposit decay `DECAY_FACTOR_STEP` and the angular clipping `LIM_ANGLE` are new, the evaporation rates and the diffusion sigmas are now split per pheromone type, and several constants related to the ant's body geometry (`HALF_LENGTH_BODY`, `ANT_RADIUS`) were added along the way. Each of these changes is the direct consequence of one of the sections above.

---

## Where We Stand Now

After all these iterations, the simulation finally produces something that looks like collective behavior. With 200 ants on a 600x400 grid, the three-antennas rule, the step-counting deposit, differentiated evaporation rates, and the proportional deposit, a few things work well and a few things are still broken.

### What works

- Trails appear and get reinforced over time. They are not perfectly stable, but they are visible and they last long enough to be useful.
- Ants come back to the nest often enough to keep the `HOME` and `FOOD` cycles running together, which was not really the case in the first iterations.
- The curve problem described earlier is strongly reduced. Ants leave the trail much less often in sharp turns than they used to.

### What is still broken

- When two `HOME` trails are close to each other, they merge through diffusion into a single path, and the ants stuck on this merged zone never go back to the nest. But what is interesting is that after some time, because of evaporation, this dead trail gets replaced by a new more efficient one that another group of ants has reinforced in the meantime. This is actually one of the main objectives of the simulation : adaptability, a colony that does not rely on a single trail but keeps rewriting its own map over time.
- The biggest current limitation is that when an ant arrives perpendicularly to a trail, it just passes through it without turning. The reason is that the gradient along the trail, the one pointing toward the nest (or toward the food, depending on the trail type), is not strong enough to trigger a clear response. If I increased this gradient, the ants that are already far from the nest would have their own deposit decayed to almost zero because of `DECAY_FACTOR_STEP`, and they would stop reinforcing the trail at long distance. I could not find how to make the parameters evolve correctly to balance both ends, so I left the behavior as it is for now. A related problem comes after the detection itself : even when an ant does notice a trail, it often joins it without knowing which direction is the right one, again because the gradient along the trail is too weak. There are a few rare cases where an ant arriving perpendicularly actually turns in the correct direction, but they are the exception, not the rule. I hope the neural network in Step 4 will handle this better than the current hand-crafted rule.

### A map creation tool for the next step

To prepare for Step 4, I also added a map creation tool on the side. The tool lets me build a custom map (nest position, food sources, obstacles if I add them later), save it if I want, and when the simulation starts it now asks which map to load. This will be useful for training : the colony will not always see the same environment, and the network will have to generalize across different map layouts.

### Videos

Four videos show the evolution of the simulation across this step, from the first chaotic runs to the current state.

- Here are the chaotic runs, I forgot what were the different parameters across them, but we can clearly see that something is not working correctly. 

{{< youtube s6q2DrQZxxw >}}

{{< youtube gHzIVaAsBrU >}}

{{< youtube ZALTSHO15Xo >}}

- Here is the current state, we can clearly the creation of path and the emergence of collective behavior : 

{{< youtube QgjBLVlBrh0 >}}

---

## What's Next

Step 3 ends here with a hand-crafted rule that mostly works : ants follow trails, come back to the nest, trigger the recruitment cycle, and sometimes rebuild a new trail when the old one gets corrupted. But it is not possible to keep tuning the parameters by hand, and rewriting new rules for every situation would never end.

Step 4 is about replacing the hand-crafted rule with a **neural network** trained for this task. I do not know yet exactly how the network will look. What I know is that I will use a **genetic algorithm** to evolve the weights of the ants. The key point here is that I want a **collective reward, not an individual one**. A single ant that brings back food is not doing well by itself, it only matters because the colony as a whole finds the source and exploits it. So the fitness score will be computed at the colony level (amount of food brought back, etc.), and each individual ant will be evolved based on the performance of the whole colony. The inputs are not fully decided yet, but I will try to keep them based on the antennas only.

---

## References

- Collett T., Graham P., Heinze S. (2025). The neuroethology of ant navigation.
- Draft R.W., McGill M.R., Kapoor V., Murthy V.N. (2018). Carpenter ants use diverse antennae sampling strategies to track odor trails.
- Wittlinger M., Wehner R., Wolf H. (2006). The ant odometer : stepping on stilts and stumps.
- Dussutour A., Nicolis S.C., Shepard G., Beekman M., Sumpter D.J.T. (2009). The role of multiple pheromones in food recruitment by ants.
- Personal discussion with Guy Theraulaz at EvoStar, Toulouse.