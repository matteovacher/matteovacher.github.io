+++
date = '2026-03-16T15:42:15+01:00'
draft = true
title = 'Genetic Algorithms (GA)'
math = true
description = 'Synthesis of the core concepts of Genetic Algorythms (GA)'
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


