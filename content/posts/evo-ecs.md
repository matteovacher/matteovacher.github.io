+++
date = '2026-05-13T04:40:14+02:00'
draft = true
title = 'Evo Ecs'
math = true 
summary = 'Here I present my EvoGym-ECS package so that people can better handle EvoGym and their research.'
+++

Donc deja, on part de entity component system donc faudra faire un tuto dessus, ensuite on part du dockerfile et de github pour expliquer comment bien ttout installer pour que ca aille vite et bien detaillé. ensuite il faut expliquer les etapes de ecs et ce que c'est et comment je les implemente. ensuite montrer comment ca marche sur un exemple simple et concret. je pense faire evoluer un controller avec une morphologie fixe. avec tournament, mutation aleatoire de poids pour commencer et egalement croisement entre individual simplement avec on alterne un sur deu et on oroduit deux invidu. 

## Introduction 

Hello dear Reader, on this page I will explain my complete and systemic implementation of the EvoGym library. When I first read the work of my predecessor Fabio Tanaka, who did an **extraordinary work** on Single Genome Robot, I was really confused on the whole implementation of the different programs that co-existed in his workspace. To implement my own version of Hyper-Neat, I spent hours understanding what he has done and why. This is mainly because of this reason that I'll explain how my code works and how I will work in the future on the EvoGym library. This article is made for people who want to learn more about *Entity Component System*, or the *EvoGym Library* and how to use it, or just *how I manage my projects* to reproduce my work.

*To finish the introduction, you can find the example of such an implementation on a GitHub repository that I have made specially for this article [here](https://github.com/matteovacher/evogym-entity-component-system).

## What is Entity Component System ? 

First of all, this idea of using Entity Component System (ECS) has not emerged from me but from my supervisor (Claus Aranha, University of Tsukuba). I hope that this article will help others as much as it helped me to organize my work, and for this particular reason, I wanted to thank Claus.

### A brief example

Well, ECS is usually involved while creating video games, therefore the example I'm about to write will be about the creation of video games. I'm not a creator of video games myself, but it surely joins one big interest of mine : Simulations. So, that being said, let's go back to creating video games. Imagine yourself, two seconds trying to implements Non Playable Characters (NPC) in a fantasy world where the hero has to explore a vast land filled with lots of available races for these NPCs (Goblins, Elves...). In the mountains of the East you find Goblins, a cruel race that will steal every piece of gold that you carry. On the West part of this land resides dozens of Human towns with lots of market places. Well at some point a tiny and very kind goblin decided to quit his life of violence in the mountains and decided to reside on one of the many human town and to become a merchant. Normally in Object-Oriented Programming you would implement a Monster Class and then implement a Goblin Class that inherits from the Monster one, then the same goes for humans and their merchants. At the end of the day, it appears complicated to add other features to our poor little Goblin that just want to live a normal life as merchant.  

Then, a simple way to implements all of this, and to manage every entity without going deeply into Class inheritance is to think about adding component to our entity/goblin. For example, in the case of our charming Goblin, we would add to the entity id number 7476572645 the following components/features :

- **Skin Color :** green 
- **Length :** short (around 120cm probably) 
- **Ears :** long and pointed
- **Profession :** merchant
- **Health :** 100
- **Weapon :** axe

This is a simple example but this way we can simply destroy/create/manage the desired components/features of every entity in the game.

### How to describe such an architecture ?

#### What's inside

As you've probably understood, ECS is a software architectural pattern. It contains 3 fundamental things :

- **Entity**, it is a unique ID that we give to every entity in the simulation. This ID will allow us to go get the components of a given entity and to modify them.

- **Components**, it is what will compose our entity, an entity can have as much as components as we want. But every entity must have one different component for a given type. For example, entity 793468682 can only have one component *Health*, not two. This will allow us to use dictionaries to store data and access them quickly. Here you might say : 'Then it is completely useless if my entity can only have one weapon and not two'. Well, if you want any character to possess more than one weapon simply add another slot of weapon, but in each slot, only a single weapon will be stored. 

- **Systems**, it is what will handle the registry of components and will write what different entity possesses. It's basically functions that takes every individuals possessing the same components and write/modify information directly in the component registries.

#### What does it look like ?

I have added here some other elements of the architecture that I will explain later.

```text 
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

Here, knowing which entity is alive will be useful for the simulation and not to run the programs on every entity created since the beginning of the simulation. Moreover, if your experience uses lots of RAM, you will be able to remove the old entity from their registry, therefore guaranteeing a lower memory usage and longer runs. 

### components.py 

In this file, we only implements the components. Remember that a component is an object that does not possess any function, its unique purpose is to store data, not to process them (this will be the job of the systems).

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

See, we only store data and don't try to process them. 

### registry.py 

Well, here we begin to explore more deeply the architecture. The idea to access the components of a given entity is to use the registry. Basically, the registry stores, in dictionaries, the id of an individual and the object component that store the data.

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

So, here we have the registry that manages what component is associated to which entity. Moreover this registry is able to get a component given an ID, or to add a component to an entity, or to modify a component, or to remove a component, or to know if an entity has a component or not.

The only bothering thing here is that every time you want to create a new component for your project, you have to add it to the registry and write all of its associated methods which can consume time that you don't want to spend on this.

### Tools

Before showing you what a typical system looks like, I want to talk a little bit about tools. Tools are meant to allow you to handle components data and to use them. For example here, in the components.py file above, I have a neural network component, but to create it, I need to extract the building information from the genome component. This is exactly the role of the tools. Here it would be extracting the building information from the genome component and create all of the data necessary to create a neural network component. To achieve this, I would simply write this function in a controller_operator.py file.
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

Then, these tools will be used in the systems. This will allow the reader to easily understand what is going on in the different system files and to easily modify the different tools and systems if it is required.  

### Systems

Systems are meant to to write/modify data from the registries. 

I think that here an example is ten times more valuable than explanations. The following evaluation_system.py file will take all the individual alive and then add their controller to the registry before evaluate them and proceed to add their fitness to the registry. Here goes the file : 

```python
import numpy as np 

class EvaluationSystem : 
    def __init__(self, entity_manager, controller_operator, config, robot_simulator, reporter_tool, parallel_tool) :
        self.config = config
        self.entity_manager = entity_manager 
        self.controller_operator = controller_operator
        self.robot_simulator = robot_simulator
        self.reporter_tool = reporter_tool 
        self.parallel_tool = parallel_tool 
        self.generation = 1
    
    def __str__(self) : 
        return "EvaluationSystem, evaluate al individuals in the current population and add their fitness to registry"

    def process(self, registry) : 

        self.reporter_tool.start_generation(self.generation)
        self.generation += 1

        entity_ids = [id for id in registry.get_all_id_with_genome() if self.entity_manager.is_alive(id)]

        for entity_id in entity_ids : 
            genome = registry.get_genome(entity_id)
            node_evals, input_nodes, output_nodes = self.controller_operator.generate_controller_from_genome(genome)
            registry.add_controller(entity_id, node_evals, input_nodes, output_nodes)

        controllers = [registry.get_controller(entity_id) for entity_id in entity_ids]
        body = self.config.body 
        bodies = [np.array(body) for _ in range(len(entity_ids))] 

        function = self.robot_simulator.simulate
        chunk = list(zip(entity_ids, bodies, controllers))

        results = self.parallel_tool.run(function, chunk)
        fitnesses = []
        ids = []
        for entity_id, fitness, finished in results : 
            ids.append(entity_id)
            fitnesses.append(fitness)
            registry.add_fitness(entity_id, fitness, finished)

        fitnesses = np.array(fitnesses)
        arg_sorted_fitnesses = np.argsort(fitnesses)
        bests = []
        number_of_reported_individuals = self.config.number_of_reported_individuals
        for taken in range(number_of_reported_individuals) : 
            id = arg_sorted_fitnesses[len(entity_ids) - 1 - taken]
            bests.append((ids[id], fitnesses[id]))
            
        
        self.reporter_tool.bests(bests)

```

### world.py

One of the last thing that we must do is to assemble all the different systems together in a world. This world is also an object, it contains all the different systems and the registry. Here goes the example of the world.py file :

```python
from registry import ComponentRegistry

class World : 
    def __init__(self) : 
        self.registry = ComponentRegistry()
        self._builder_systems = []
        self._step_systems = []
        self.all_systems = []

    def add_builder_system(self, system) : 
        self._builder_systems.append(system)
        self.all_systems.append(system)
    
    def add_step_system(self, system) : 
        self._step_systems.append(system)
        self.all_systems.append(system)

    def reset(self) : 
        self.registry.clear_all_except_genome()

    def build(self) : 
        for system in self._builder_systems : 
            system.process(self.registry)

    def step(self) : 
        for system in self._step_systems : 
            system.process(self.registry)
```

### main.py 

The last thing to do is to put all of this together in a main file to run the simulation. Here goes the example of the main.py file :

```python
import os
import json  

from config import Config  
from entity_manager import EntityManager 
from world import World
 
from systems.build_system import BuildSystem
from systems.evaluation_system import EvaluationSystem
from systems.tournament_system import TournamentSystem

from tools.controller_operator import ControllerOperator
from tools.genome_operator import GenomeOperator
from tools.robot_simulator import RobotSimulator
from tools.parallel_tool import ParallelTool
from tools.reporter_tool import ReporterTool

from results_manager.results_saver import ResultsSaver


def main() : 
    entity_manager = EntityManager()
    world = World()


    config_path = input("\nEnter the path to the config file from the configs folder (can be just config.json) : ")
    local_dir = os.path.dirname(os.path.abspath(__file__))
    config_path_final = os.path.join(local_dir, "configs", config_path)
    with open(config_path_final, 'r') as f : 
        config = json.load(f)

    config = Config(config)

    results_saver = ResultsSaver()
    results_saver.add_results_path()

    controller_operator = ControllerOperator()
    robot_simulator = RobotSimulator(config, controller_operator)
    genome_operator = GenomeOperator(config, robot_simulator)
    parallel_tool = ParallelTool(config)
    reporter_tool = ReporterTool(config)


    build_system = BuildSystem(config, entity_manager, genome_operator, reporter_tool)
    evaluation_system = EvaluationSystem(entity_manager, controller_operator, config, robot_simulator, reporter_tool, parallel_tool)
    tournament_system = TournamentSystem(entity_manager, config, genome_operator, reporter_tool)

    world.add_builder_system(build_system)
    world.add_step_system(evaluation_system)
    world.add_step_system(tournament_system)

    world.build()

    for generation in range(config.generations) : 
        world.step()

    results_saver.save_results(world.registry, config, config_path)

if __name__ == "__main__" : 
    main()

```

### Intermediate Conclusion

Now we have all the necessary information to implement a simulation using ECS. This was the first part of the article, next I'll explain how to install the different necessary libraries and use Docker to run all of your EvoGym simulations in a container. Moreover, I will shortly explain how I use the Evogym library to reproduce my work.

## Docker and Dockerfile



