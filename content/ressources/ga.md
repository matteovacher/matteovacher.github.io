+++
date = '2026-03-16T15:42:15+01:00'
draft = true
title = 'Ga'
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

### The (1+1) Evolutionary algorithm 

*In Evolutionary Strategies (Es), population management is defined by the*


