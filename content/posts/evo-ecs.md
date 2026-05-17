+++
date = '2026-05-13T04:40:14+02:00'
draft = true
title = 'Evo Ecs'
math = true 
summary = 'Here I present my EvoGym-ECS package so that people can better handle EvoGym and their research.'
+++

Donc deja, on part de entity component system donc faudra faire un tuto dessus, ensuite on part du dockerfile et de github pour expliquer comment bien ttout installer pour que ca aille vite et bien detaillé. ensuite il faut expliquer les etapes de ecs et ce que c'est et comment je les implemente. ensuite montrer comment ca marche sur un exemple simple et concret. je pense faire evoluer un controller avec une morphologie fixe. avec tournament, mutation aleatoire de poids pour commencer et egalement croisement entre individual simplement avec on alterne un sur deu et on oroduit deux invidu. 

## Introduction 

Hello dear Reader, on this page I will explain my complete and systemic implementation of the EvoGym library. When I first read the work of my predecessor Fabio Tanaka, who did an **extraordinary work** on Single Genome Robot, I was really confused on the whole implementation of the different programs that co-existed in his workspace. To implement my own version of Hyper-Neat, I spent hours understanding what he has done and why. This is the main reason, I'll explain how my code works and how I will work in the future on the EvoGym library. This article is made for people who want to learn more about *Entity Component System*, or the *EvoGym Library* and how to use it, or *how I manage my projects* to reproduce my work. 

*To finish the introduction, you can find the exampleof such an implementation on a GitHub repository that I have made specially for this article [here](https://github.com/matteovacher/evogym-entity-component-system).

## What is Entity Component System ? 

First of all, this idea of using Entity Component System (ECS) has not emerged from me but from my supervisor (Claus Aranha, University of Tsukuba). I hope that this article will help others as much as it helped me to organize my work, and for that I want to thank Claus. 

### A brief example

Well, ECS is usually involved while creating video games, so the exemple I'm about to write will be about creating video games. I'm not a creator of video games but it surely joins one big interest of mine : Simulations. So, that being said, let's go back to creating video games. Imagine yourself, two seconds trying to implements NPC (Non Playable Characters) in a fantasy world where the hero has to explore a vast land with lots of race (Goblins, Elves...). In the mountains of the East you find Goblins, a cruel race that will steal every piece of gold that you carry. On the West part of this land resides dozens of Human towns with lots of market places. Well at some point a tiny and very kind goblin decided to quit this life of violence and decided to reside on one of the many human town and to become a merchant. Normally in Object-Oriented Programming you would implement a Monster Class and then implement a Goblin Class that inherits from the Monster one, then the same goes for humans and their merchants. At the end of the day, it appears complicated to add other features to our poor little Goblin that just want to live a normal life. 

Then, a simple way to implements all of this and to manage every entity without going deeply into Class inheritance is to think about adding component to our entity. For example, in the case of our charming Goblin, we would add to the entity id number 7476572645 the following components/features :

- **Skin Color :** green 
- **Length :** short (around 120cm probably) 
- **Ears :** long and pointed
- **Profession :** merchant
- **Health :** 100
- **Weapon :** axe

This is a simple example but this way we can simply destroy/create/manage the desired components/features.

### How to describe such an architecture ?

#### What's inside

As you've probably understood, ECS is a software architectural pattern. It contains 3 funddamental things :

- **Entity**, it is a unique ID that we give to every entity in the simulation. This ID will allow us to go get the component and to modify them on this given entity.

- **Components**, it is what will compose our entity, an entity can have as much as component as we want. But every entity must have one different component for a given type. For example, entity 793468682 can only have one component *Health*, not two. This will allow us to use dictionaries to store datas. 

- **Systems**, it is what will handle the registry of component and will write what different entity possesses. It's basically functions that takes every individuals possessing the same components and apply them functions to write information into the component registry 

#### How does it look ?

I have added here some other elements of the architecture that I will explain later.

``` 
project 
|
|-main.py
|-entity_manager.py
|-components.py
|-registry.py
|-world.py
|
|   systems
|---|
|   |-system1.py
|   |-system2.py
|
|   tools
|---|
|   |-tool1
|   |-tool2
```

### entity_manager.py 

The aim of this file is to create, destroy, know which entity is alive.
Your entity manager should looks like this :

```python
class EntityManager : 

    def __init__(self) : 
        self.next_id = 0 
        self._alive = set()

    def create_entity(self) : 
        id = self.next_id 
        self.next_id += 1
        self._alive.add(id)
        return id 
    
    def destroy_entity(self, entity_id) : 
        self._alive.remove(entity_id)

    def is_alive(self, entity_id) : 
        return entity_id in self._alive
```

Here, knowing which entity is alive will be useful for the simulation and not to run the programs on every entity created since the beginning of the simulation. 

### components.py 

In this file, we only implements the components. Remember that a component is an object that does not possess es any functions, it is just made to store data, not to process them (this will be the job of the systems).

Here is the example of a components file :

```python 
class GenomeComponent : 

    def __init__(self, connections, nodes) : 
        self.connections = connections 
        self.nodes = nodes 


class FitnessComponent : 

    def __init__(self, fitness, finished) : 
        self.fitness = fitness 
        self.finished = finished 

class ControllerComponent : 
    def __init__(self, node_evals, input_nodes, output_nodes) : 
        self.node_evals = node_evals  
        self.input_nodes = input_nodes 
        self.output_nodes = output_nodes 
```

See, we only store data and don't exploit them.

### registry.py 

Well, here we begin to explore more deeply the architecture. The idea to access the components of a given entity is to use the registry. 

The registry looks like this :

```python 
from components import * 

class ComponentRegistry : 

    def __init__(self) : 
        self.genome_registry = {}
        self.fitness_registry = {}
        self.controller_registry = {}


    # ADDER METHODS
    def add_genome(self, entity_id, connections, nodes) : 
        self.genome_registry[entity_id] = GenomeComponent(connections, nodes)
    
    def add_fitness(self, entity_id, fitness, finished) : 
        self.fitness_registry[entity_id] = FitnessComponent(fitness, finished)

    def add_controller(self, entity_id, node_evals, input_nodes, output_nodes) : 
        self.controller_registry[entity_id] = ControllerComponent(node_evals, input_nodes, output_nodes)
    
    # GETTER METHODS
    def get_genome(self, entity_id) : 
        return self.genome_registry[entity_id]

    def get_fitness(self, entity_id) : 
        return self.fitness_registry[entity_id]
    
    def get_controller(self, entity_id) : 
        return self.controller_registry[entity_id]
    
    # CHECKER METHODS 
    def has_genome(self, entity_id) : 
        return entity_id in self.genome_registry
    
    def has_fitness(self, entity_id) : 
        return entity_id in self.fitness_registry
    
    def has_controller(self, entity_id) :
        return entity_id in self.controller_registry 

    
    # ADVANCED GETTER METHODS 
    def get_all_id_with_genome(self) : 
        return self.genome_registry.keys()
    
    def get_all_id_with_fitness(self) : 
        return self.fitness_registry.keys()
    
    def get_all_with_controller(self) : 
        return self.controller_registry.keys()
    
    # MODIFIERS, please give an object 
    def modify_genome(self, entity_id, genome) : 
        self.genome_registry[entity_id] = genome
        
    def modify_fitness(self, entity_id, fitness) : 
        self.fitness_registry[entity_id] = fitness 
    
    def modify_controller(self, entity_id, controller) : 
        self.controller_registry[entity_id] = controller
    
    # CLEARER METHODS 
    def clear_all_except_genome(self) : 
        self.fitness_registry.clear()

    def clear_genome(self) : 
        self.genome_registry.clear()

    def clear_fitness(self) : 
        self.fitness_registry.clear()

    def clear_controller(self) : 
        self.controller_registry.clear()
```

So, here we have the registry that manges what component is associated to which entity. Moreover this registry is able to get a component given an ID, or to add a component to an entity, or to modify a component, or to remove a component, or to know if an entity has a component or not.

The only bothering thing here is that every time you want to create a new component for your project you have to add it to the registry and all of its associated methods.

### Tools 

Before showing you what systems look like I want to talk a little bit about tools. Tools are meant to allow you to handle components datas and to use them. For example here, in the components.py file above, I have a neural network component, but to create it I need the genome component, then according to what the genome says, i can create the neural network. This is exactly the role of the tools. I would implement the controller_operator.py file, and in it I will add a function that returns me the necessary elements to create a controller from the genome component.

Here is what the controller_operator.py file looks like :

```python
import math 

class ControllerOperator : 
    activation_function = math.tanh
    output_activation_function = lambda x : x 
    agregation_function = sum
    response = 1 
    bias = 0 


    def __init__(self) : 
        pass 

    def generate_controller_from_genome(self, genome) : 
        node_evals = []
        for index_of_layer in range(len(genome.nodes.keys())) : 
            if index_of_layer == 0 : 
                input_nodes = []
                for input_node in genome.nodes[index_of_layer] : 
                    input_nodes.append(input_node)
                previous_layer = genome.nodes[index_of_layer]
                continue 

            if index_of_layer == len(genome.nodes.keys()) - 1 : 
                output_nodes = []
                for output_node in genome.nodes[index_of_layer] : 
                    output_nodes.append(output_node)

            for node in genome.nodes[index_of_layer] :
                inputs_of_node = []
                for previous_node in previous_layer : 
                    weight = genome.connections[(previous_node, node)]
                    inputs_of_node.append((previous_node, weight))
                node_evals.append((node, self.activation_function, self.agregation_function, self.bias, self.response, inputs_of_node))

            previous_layer = genome.nodes[index_of_layer]
        return node_evals, input_nodes, output_nodes
    
    def activate(self, controller, input_values) :
        values = {}
        for key, value in zip(controller.input_nodes, input_values) : 
            values[key] = value
        
        for node, activation_function, agregation_function, bias, response, inputs_of_node in controller.node_evals : 
            node_inputs = []
            for previous_node, weight in inputs_of_node : 
                node_inputs.append(values[previous_node] * weight)
            entering_node = agregation_function(node_inputs)
            values[node] = activation_function(bias + response * entering_node)
        return [values[node] for node in controller.output_nodes]
```

Then, these tools will be used in the systems. This will allow the reader to easily understand what is going on in the different system file and to easily modify the different tools and systems in case of it is needed. 

### Systems


















