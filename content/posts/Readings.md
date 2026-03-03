+++
date = '2026-02-22T00:55:18+01:00'
draft = true
title = 'Readings'
+++

## Readings, results, discussion and possibility of research 

Hello everyone, 

Thanks to Amany Amin I am now in possession of 10 articles and I am sure that these papers will help me through my project. Here I'll try to note important results from different papers and try to organize them to highlight future possible research.

--- 

### Emergent regulation of ant foraging frequency through a computationally inexpensive forager movement rule *Baltiansky, L., Frankel, G., & Feinerman, O. (2023)*

**Key Question:** How does a colony regulate its foraging frequency based on collective hunger without a central controller or complex individual computations ?

#### Key results & mechanism

* **The Walk Rule :** foragers entering the nest after foraging follow simple rules : 
    * over a certain treshold : the ant walks deeper in the nest (inward).
    * beside a certain treshold : the ant walks toward the entrance of the nest (outward). 
* **Linearity :** there is a linear relationship between the colocny satation and the forage frequency. 
    * if the colony is hungry, many receivers available, the foragers are empty quicker, foragers exist quicker 
    * if the colony is satiated, few receivers available, the foragers go deeper into the nest, exit is delayed. 

#### Possibility of Research

* **Inexpensive computation :** The ants don't need to sens global rate of change, they only react to their close environment and interactions with trophallaxis. 
* **Linearity :** Does my simulation show a liinear decrease as the global satiation increase ? 

#### Note 

* Maybe it could be interesting to see if in simulations some ants specialize in foraging and other as workers in the nest and observe their proportions (for non polymorphic species). More The food amount transferred is a stochastic fraction of the receiver’s unfilled crop space. This fraction follows an exponential distribution with a mean of 0.15. If the forager’s available food is less than this calculated amount, she transfers her entire current stock.

--- 

### Dynamics of cooperative excavation in ant and robot collectives *S Ganga Prasath, Souvik Mandal, Fabio Giardina, Jordan Kennedy, Venkatesh N Murthy, L Mahadevan (2022)*

**Key question :** How do individuals with limited cognitive capacities cooperate to solve complex excavation tasks, and is it possible to model a robotic excavation ? 

#### Key results & observations

* **Phases of cooperation :** ants *(here Camponotus pennsylvanicus)* go through differents phases (here is a classic scheme) : 
    * first confused, the ants began to explore randomly
    * second then proceed to begin excavation at a random point 
    * third exploit a specific tunnel at a specific location by conecntrating effort

* **Excavation Rate & Cooperation :** The cllective success is governed by two parameters : 
    * cooperation : it measures how much pheromone is released which indicates communications between ants
    * Excavation : represents the frequency at which individuals move substrate from one place to another

#### Possibility of research : 

* Here the number of workers is fixed, it could be interesting to look after the total number of worker in my simulations (for example, too much workers could lead to an overstimulated colony)
* More it seems interesting to look upon the case of a deficient individual and see its consequences over the colony 

#### Note : 

* **Communication :** back at the nest, an ant that brought food could, by physical contact (let's say the distance between two ants is under 0.2 of their body length), gives information about the direction of the position of the food (so that the other ant could get quicly to the pheromone trail). 


### Foraging ants as liquid brains: Movement heterogeneity shapes collective efficiency *Fernández-López, P., Oro, D., Lloret-Cabot, R., Genovart, M., Garriga, J., & Bartumeus, F. (2025)*

**Key Question :** How does the hetrogeneity of individual movement behaviors (scouts vs recruits) shape collective foraging, its efficiency and the connectivity within a colony operating as a "liquid brain" ?

#### Key results & Mechanisms : 

* **Liquid brain concept :** The colony is a living system without central nervous system. Instead, the colony is shaped by collective intelligence, local interactions that build the whole behavior of the colony. These local interactions are maintained and reshaped by the movement of individuals. 

* **Behavior heterogeneity :** There are two distinct roles for the exploration phase : 
    * scouts : these ants usually have a higher speed and tend to move on a straight line. 
    * recruits : these ants usually gather near the nest entrance. 

* **Transition mechanism :** The discovery of food triggers positive social feedback (recruitment) through physical contact, shifting ants (recruits) from undirected and diffusive movement to source oriented movements. 

* **Efficiency and hubs :** A high proportion of scouts accelerates resource discovery while recruits near the nest entrance act as communication hubs where the information spread quickly. 

#### Possibility for Research : 

* **Proportion :** looking after the proportion of individuals for each role and see if some individuals adapt to the environment or if they maintain consistent roles.

* **Communication :** looking after the diffusion of information as movement seems to be the primary factor to maintain information flow. 

#### Note : 
The study shows that the collective intelligence of the colony *(Aphaenogaster senilis)* does not rely on pheromones but on the ability of movement to dynamically reconfigure the interaction network. This behavioral diversity allows for an adaptive balance between the trade-off of exploring new areas and exploiting known resources.






