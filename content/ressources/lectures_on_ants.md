+++
date = '2026-03-27T22:24:32+01:00'
draft = true
title = 'Documentation for my project'
math = true 
summary = 'This serves as a summary (made by Claude) for helping me with the writing of my articles'
+++

# Reading Notes -- Pheromone Grid  Made by Claude will be used as a summary to find again the results 

## Sources used
- Edelstein-Keshet, Watmough & Ermentrout (1995). Trail following in ants: individual
  properties determine population behaviour. Behav Ecol Sociobiol 36:119-133.
- Weickert et al. (1998). Efficient and reliable schemes for nonlinear diffusion filtering.
  IEEE Trans Image Processing 7(3):398-410.
- von Thienen, Metzler, Choe & Witte (2014). Pheromone communication in ants:
  a detailed analysis of concentration-dependent decisions in three species.
  Behav Ecol Sociobiol 68:1611-1627.
- Dorigo, Bonabeau & Theraulaz (2000). Ant algorithms and stigmergy.
  Future Generation Computer Systems 16:851-871.
- Laptik (2012). Ant System with distributed values of pheromone evaporation.
  Elektronika ir Elektrotechnika 18(8):69-72.

---

## 1. Evaporation

### Mathematical form

Edelstein-Keshet (1995) assumes simple linear kinetics with fixed rate constant $r$:

$$\frac{dP}{dt} = -r \cdot P \quad \Rightarrow \quad P(t) = P(0) \cdot e^{-rt}$$

Discrete-time exact analog (standard in ACO, Dorigo 2000):

$$C_{t+1} = (1 - \rho) \cdot C_t$$

where $\rho \in (0, 1)$ is the evaporation coefficient per time step.
Conversion from continuous rate: $\rho = 1 - e^{-r \cdot \Delta t}$, with $\Delta t$ the simulation time step in seconds.

Edelstein-Keshet simulations discretize differently ("level decreases by 1 unit per step"),
which is a linear approximation requiring normalized deposits. The exponential form above
is more general and is what we implement.

### Biological values by species (Edelstein-Keshet 1995, Table 1 + Table 2)

| Species               | Trail duration (TD)       | Decay rate $r$ ($s^{-1}$)   |
|-----------------------|---------------------------|------------------------------|
| Atta texana           | > 6 days                  | very small (unspecified)     |
| Eciton burchelli      | 2.25 -- 8.25 days         | $\approx 4 \times 10^{-7}$   |
| Iridomyrmex humilis   | ~30 min                   | $\approx 5 \times 10^{-4}$   |
| Myrmica rubra         | 2 -- 3 min                | 0.005 -- 0.008               |
| Solenopsis saevissima | 104 s (glass) / 20 min    | 0.008 -- 0.12                |
| Pogonomyrmex badius   | ~35 s                     | 0.028                        |

$r$ spans ~5 orders of magnitude. Species choice directly determines `EVAPORATION_RATE`.
No species has been selected yet for this simulation.

Additional longevity data (von Thienen 2014, unpublished):
- Euprenolepis procera: trail still active after 22 h
- Linepithema humile: trail still active after 22 h
- Lasius niger: no significant activity after 160 min; mean lifetime ~47 min
  (Beckers et al. 1993, cited in von Thienen 2014)

### External factors (von Thienen 2014)
Temperature, rain, dew, substrate type all affect $r$.
No quantified values for these factors in any of the sources read.

### Saturation (Edelstein-Keshet 1995)
Deposit accumulates up to a ceiling $P_{max}$ beyond which ants cannot distinguish two trails
(von Thienen 2014: response probability plateaus at high concentrations).
-> confirms keeping clip at MAX_PHEROMONE in config.py.

---

## 2. Diffusion

### Key theoretical result (Weickert 1998)

Linear isotropic diffusion:

$$\frac{\partial C}{\partial t} = D \cdot \nabla^2 C$$

solved up to time $T$ is mathematically equivalent to convolution with a Gaussian kernel
of standard deviation:

$$\sigma = \sqrt{2DT}$$

Per time step ($T = \Delta t$):

$$\boxed{\sigma = \sqrt{2 D \cdot \Delta t}}$$

This is the exact justification for using `scipy.ndimage.gaussian_filter` to implement
diffusion. Once $D$ and $\Delta t$ are chosen, $\sigma$ is determined. No approximation.

### Why linear diffusion, not nonlinear (Weickert 1998)

Weickert introduces nonlinear (anisotropic) diffusion to preserve edges in images:
diffusivity $g(|\nabla C|)$ is a decreasing function of the gradient magnitude, so smoothing
is strong inside regions and weak across boundaries.

For pheromones, we want the opposite: uniform isotropic spreading, no edge preservation.
-> Linear diffusion (standard Gaussian, $g \equiv 1$) is the correct choice.

### Guaranteed properties of the Gaussian filter (Weickert 1998)

- Average grey level invariance: total pheromone mass is conserved per diffusion step.
- Maximum-minimum principle: no new concentration values are created, no overshoots.
- Convergence: without deposit, the grid converges to uniform concentration then to zero.

All three are biologically desirable.

### Biological constraint on $\sigma$ (Edelstein-Keshet 1995, citing Bossert & Wilson 1963)

A trail from a single Solenopsis saevissima ant remains detectable up to ~28 cm.
-> $\sigma$ (in physical units via cell size in cm) should not exceed this radius after
   the time steps corresponding to one ant traversal. Use as a sanity check only.

### Diffusion coefficient $D$

No biologically calibrated value in any of the sources read.
-> $D$ is a free parameter. Calibrate empirically by checking that simulated gradients
   are detectable by ants at realistic distances.

---

## 3. Deposit

### Edelstein-Keshet (1995) continuous model

Distinguishes exploratory ants ($L$, laying fresh trails) from followers ($F$, reinforcing):

$$\frac{dT}{dt} = v \cdot L + a \cdot F - \Gamma \cdot T$$

where $v$ = marking rate of explorers, $a$ = marking rate of followers,
$\Gamma$ = pheromone decay rate (same as $r$ above).

Simple trail length behind one ant: $d_s = v / \Gamma$
Follower spacing on trail: $d_f = a / \Gamma$

Explorer/follower state must appear in the ant's state vector (Step 2, not Step 1).

### ACO deposit formula (Dorigo 2000)

$$\Delta\tau_{ij} = \frac{Q}{L_k}$$

with $L_k$ = tour length, $Q$ = constant.
NOT applicable: defined for weighted graph edges. On a 2D grid, each ant deposits a fixed
amount to the cell it currently occupies.

### Information capacity and deposit quantities (von Thienen 2014)

$$IC = \frac{\text{pheromone quantity in gland}}{tdt_{75} \text{ (detection threshold per cm)}}$$

| Species               | $IC$ (cm / gland) |
|-----------------------|-------------------|
| Linepithema humile    | 13,450            |
| Lasius niger          | 1,094             |
| Euprenolepis procera  | 197               |

Deposit concentration for Linepithema humile -- contradictory measurements:
- Choe et al. (2012): 0.3 pg/cm  (direct chemical measurement of natural trails)
- von Thienen (2014): 18.5 pg/cm (range 9.9--33.3, estimated from behavioral response)
- Discrepancy: factor 30--100x, explicitly noted as unresolved in von Thienen (2014).

-> Absolute deposit values are unreliable. Use a normalized DEPOSIT_AMOUNT
   (fraction of MAX_PHEROMONE) and calibrate empirically.

---

## 4. Models considered and rejected

### Watmough & Edelstein-Keshet (cited in prior notes)
- Evaporation: linear decay $C_{t+1} = \max(0,\, C_t - 1)$, not exponential.
- Diffusion: explicitly excluded from their model (assumed negligible vs ant walking speed).
-> Not applicable: we want diffusion, and we use exponential decay.

### Deneubourg pillar model (Dorigo 2000, Sec. 1)
Reaction-diffusion model for termite nest building. No overlap with pheromone trails.

### Deneubourg choice function -- exponent assumption (von Thienen 2014)

Classical form: $p = (k + c_L)^b \,/\, [(k + c_L)^b + (k + c_R)^b]$ with $b \approx 2$ assumed.

Von Thienen measures $b$ empirically for three species:

| Species / experiment       | $b$ measured  |
|----------------------------|---------------|
| E. procera detection       | 1.05          |
| L. humile detection        | 0.60          |
| L. niger detection         | 1.21          |
| Discrimination (all)       | mostly < 1.1  |

-> $b \approx 2$ is NOT empirically supported. Relevant for Step 2 (ant decisions).

### Laptik (2012) -- distributed evaporation in ACO
Distributes $\rho$ spatially across TSP nodes. Designed for graph optimization.
-> Not applicable. File as a possible future experiment.

---

## 5. Ant perception and Weber's law (von Thienen 2014)

Not needed for Step 1, but will constrain Step 2.

Ants follow Weber's law: discrimination depends on concentration RATIO, not absolute level.
Valid range spans factor 16--2048x depending on species.

75% discrimination threshold (ratio needed to prefer the stronger trail):

| Species           | $tds_{75}$ |
|-------------------|------------|
| E. procera        | 2.14x      |
| L. humile         | 2.48x      |
| L. niger          | 3.82x      |

75% detection threshold (minimum detectable concentration, in units of base concentration $bc$):

| Species           | $tdt_{75}$  |
|-------------------|-------------|
| L. humile         | 0.05 $bc$   |
| L. niger          | 0.16 $bc$   |
| E. procera        | 0.89 $bc$   |

Lapse rates $\lambda$ (behavioral errors / inverse trail fidelity):
- E. procera: $\lambda = 0.05$ (high fidelity, mostly pheromone-guided)
- L. humile:  $\lambda = 0.11$
- L. niger:   $\lambda = 0.18$--$0.21$ (uses optical cues too)

Psychometric function (PF) fits empirical data as well or better than the Deneubourg choice
function -> prefer PF over Deneubourg for ant decision modeling in Step 2.

---

## 6. Implementation formula retained for Step 1

$$C_{t+1} = \mathrm{gaussian\_filter}(C_t,\, \sigma) \cdot (1 - \rho) + \delta_t$$

Parameters:
- $\sigma = \sqrt{2 D \cdot \Delta t}$ -- $D$ free, calibrate empirically
- $\rho =$ `EVAPORATION_RATE` -- fix after choosing target species
- $\delta_t =$ `DEPOSIT_AMOUNT` at current ant cells -- calibrate empirically
- `MAX_PHEROMONE` = ceiling on $C$ -- confirmed biologically (saturation)

Order: diffuse first, evaporate, then add deposit.

Open questions before fixing parameters:
1. Target species? -> determines $r$ -> $\rho = 1 - e^{-r \Delta t}$
2. Time step $\Delta t$? -> needed to convert $r \to \rho$ and $D \to \sigma$
3. Grid cell size in cm? -> needed to calibrate $\sigma$ against the 28 cm active space bound