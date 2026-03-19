+++
date = '2026-03-16T15:42:15+01:00'
draft = true
title = 'Genetic Algorithms (GA) - Evolutionary Computation Elective by Prof. Dennis Wilson, ISAE-SUPAERO'
math = true
summary = 'Synthesis of the core concepts of Genetic Algorythms (GA)'
+++

## Introduction to Evolutionary Computation 

### Individuals and Genes 

The basixc unit of an EA (Evolutionary Algorythm) is the individual, which represents a potential solution to a given problem. In this tutorial, individuals are represented by binary strings, consisting of genes that are either 0 or 1. Each individual of the population is initialized with random genes and a default fitness score of zero. 

### Evaluation and Objective Functions

What is an objective function ? 

An **objective function** (or fitness function) is a function that gives a score to the individual (potential solution) i.e. tells us how good is the individual on the given problem. On a given problem, there could be multiple objective functions. Each of them will give a different score to the individual. On the first Notebook we have two objective functions : 
* **OneMax :** This function returns the sum of all the bits in the genotype. The aim of this function is to find the individual with the maximum number of 1 bits.
* **LeadingOnes :** This functions counts the number of consecutive starting 1s from the left, stopping at the first 0. 

### The (1+1) Evolutionary Algorithm 

*In Evolutionary Strategies (ES), population management is defined by the $(\mu / \rho , \lambda)$ or $(\mu / \rho + \lambda)$ notation. Here, $\mu$ is the number of parents, then $\rho$ is the number of parents involved in creating the next generation, the offspring, and finally, $\lambda$ is the number of offspring. Also, note that the notation with the $+$ indicates that the parents creating the offspring can be part of the next generation while the $,$ indicates that the parents will nit be part of the next generation. For the $+$ notation, whether the parents or included or not in the next generation depends entirely on their fitness compared to the fitness of their offspring*

#### Well then, what is the $(1 + 1)$ EA ? 

This algorithm follows really simple steps : 
1. **Initialization :** Create an individual by choosing random bit strings.
2. **Mutation :** Create a child by flipping each bit of the parent with a certain probability. 
3. **Selection :** Select (by comparing the fitness function) the best individual between the parent and the child. 
4. **Iteration :** repeat the process until a stopping condition is met.

#### Performance 

This algorithm is the most simple that can be done. His performance depends primarily on the the problem and on the objective function. 

### The $(1 + \lambda$) EA 

#### The Algorithm 

In the previous algorithm a single parent parent produces a single offspring. In this algorithm, a single parent produces $\lambda$ offspring. The best child is then compared to the parent and if the child has a higher fitness the parent is replaced by the child. 

#### The Performance

The $(1 + \lambda)$ usually requires fewer generations to converge, it performs more evaluations per generation ($\lambda$ to be exact). In order to compare the $(1 + 1)$ and the $(1 + \lambda)$ algorithm, we need to compare the total number of evaluation and not only the number of generations. 

### More on These Algorithms 

Finally the choice of the mutation rate and the population size significantly impact the performance of the EA. 


## Genetic Algorithms 

### What are GA ? 

GA are a class of stochastic optimization techniques that are higly adaptable to specific problem constraints. GA can for example replace gradient based optimization algorithms. In the notebook of th elective we study the case of the Traveling Salesman Problem (TSP), a problem where a salesman must visit every city once in a list of cities by minimizing the total distance traveled. 

GA are basically composed of different step : 
1. **The population :** The population is a set of individuals that represent potential solutions to the problem.
2. **The Evaluation :** Each individual is evaluated using a fitness function to determine their "fitness". Here, we want to minimize the distance traveled in 2D space. 
3. **The Selection :** The selection is then the process of selecting the best individuals from the population. Several methods are available. 
4. **The Crossover :** The crossover is the process of creating new individuals by combining the genes of two parents. 
5. **The Mutation :** The mutation is then the process of modifying the genes of an individual to explore new solutions. 

By repeating these steps, we can generate a new population of individuals that are more likely to be good solutions to the problem. 

### The Population : 

The first step in a GA is defining the representation of a solution, known as the genome. For the TSP, a genome is represented as a permutation of city indices, ensuring each city is visited exactly once.

- Unlike simpler ES, GA typically use a large population size, typically around 100-1000.

- Maintaining a large population size allows for more diversity in the population, which can lead to better solutions by looking for multiples different potential solutions at the same time. 

- This diversity is the key to avoiding being trapped in a local optimum. 

### The Evaluation : 

To guide the evolution, every individual in the population must be assessed by an **objective function** to determine their fitness. 

- In the context of the TSP, the fitness is measured by the total distance traveled by the salesman. 

- The objective function of an individual $I$ (which is represented by a list of the cities visited) is then expressed as : 
$$f(I) = \sum_{i=1}^{n-1} \text{dist}(p_{i-1}, p_i)$$
with : 
    - $n$ is the number of cities.
    - $p_{i}$ is the index of the city at position $i$ in the tour.
    - $\text{dist}(a, b)$ is the pre-computed distance between city $a$ and city $b$ from the distance matrix.

### The selection 

The selection is the process of deciding which individuals will be selected and will pass their genes to the next generation. 
The goal is to favor the best-performing individuals while preserving the diversity in the group. 

Here, there exists different type of selection methods : 

- **Truncation Selection :** This really simple method (compared to CMA-ES, which we'll see later) simply takes the top percentage of the population. It is a really efficient method but in nature for example it does not preserve the diversity of the population. It is the same here, it may loose diversity too quickly. 

- **Fitness Proportionate Selection :** This gives every individual a chance to be selected based on a probability relative to their fitness.

- **Tournament Selection :** We first choose a subset of individuals randomly from the population. Then the best among them is given the right to reproduce. This method relies more on the rank than on the absolute value of the fitness which means that if you are selected to reproduce it is because you are the best of the subset and not because you crush your opponents. 

### The Crossover : 

The crossover is the "sexual" part of the algorithm where the information from two parents is combined to create an offspring. 

- In many problem, a one point crossover is used to swap genetic segments after a random point. 

- Nevertheless, for the TSP, standart crossover can break constraints by creating an offspring that visit a city twice. 

- To solve this, we use a specialized operator like the Edge Recombination Operator (ERX), which builds a child by prioritizing the existing links between the cities find in both parents. 

### The Mutation : 

The mutation step is then necessary to introduce entirely new genetic material into the population. 

- Usually mutation modifies an individual by changing only a small percentage of its genes (usually at a rate of 1/n_genes). 

- To respect TSP constraints, we swap the order of a single random pair of cities. 

- This step is vital for maintaining genetic diversity and allows the algorithm to search outside the current combinations already existing in the population. 

a continuer mais ensuite le prog prends 20 % meilleur avant de faire tournament faudra expiquer 




