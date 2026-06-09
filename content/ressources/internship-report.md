+++
date = '2026-05-25T11:52:31+02:00'
draft = true
title = 'Internship Report'
math = true 
+++


## Introduction 

In this page, I will report what I will be doing in my internship and the different steps that compose this project. 
I begin this article with almost a month of delay since I was really involved in my search.

## Before and after coming to japan 

Before coming to Japan, Claus, my supervisor here in Japan told me to do Dennis' elective and to reproduce Fabio's work (Fabio is the student that work on the project before me). 
First I did the elective, managed to make some summary that will follow my path for a long time. Then I started to reproduce the work of Fabio.

### The Dockerfile problem 

By first looking at the architecture of the files, I noticed that the project contained a Dockerfile. I understood that it was meant to increase the reproducibility of the work but it wouldnt work due to some software version problem. I had to take a look into this and learn how to use and make my own file. I spent almost 3 or 4 days learning how to handle Docker. Then I built my own Dockerfile and as I'm writting these lines, I know that that my work can be easily reproduce from another computer. During my internship here I will make another article to explain how my work is arranged and built, how to begin a simulation, how to render it and other stuff. 

### The Code 

In order to reproduce the work of Fabio, I had to go into all of his files, lots of them werent related to the work he published, so I had to look deeply into how each file was working with another. By doing so, I finished by extracting the global layout of the project, and began to rewrite the code. I finally had a version of the work that would correspond at least a little to how I exploit the code. Let me explain, Fabio was using a 'parser', which is a way to give the hyperparameters in the command line of your computer, here in Docker COntainer. I was not completely satisfied with the way his code worked so I decided to include a config.json file that would allow me to get a better trace of the different experiment. 

### The Entity Component System 

Here I was in Tsukuba. On my first day at the lab, I met Claus, and as a first advice he told me to look intot what is called Entity Component System, but why ? It is because I expressed my desire to built a version of the project that would be more Object Oriented Programming. Comming from an engineering school, I wanted to implement what I had studied and I also wanted to make something that would be easier to understand. I spent I think the next week to implement my own version of the project with ECS. 

But first what is ECS ? 
Usually with Python you create a class, let's say sword for object that will act as a sword, then you create a sub-class fire-sword and ice-sword that both inherits the class sword. Then you realize that you want to have sword that deals both fire and ice damage. Here is the twist, now it seems really simple tyo do so. But with more and more object, the dependancies will become too complicated to handle and to keep track of. Instead, you could use ECS, you associate to the object an and Identity Number and you attached to him different component like the sword component and then the fire element. If you want it is easy to add an ice element to the sword. This is the main idea of ECS.

I build the ECS version of Fabio's project but never tested it and I think that I had many errors inside. Instead I realized that for the next student that will work on the project, it would be nice for him/her to have a manual of instruction of how the work is done...

### The Entity Component System project example 

In order to show an example, I decided to build an entire example project, the idea was that I would implement a really simple genetic algorythm with this method so that it would serve as a pillar for every other project. 

The project is available on GitHub [here](https://github.com/matteovacher/evogym-entity-component-system)

I spent another week building this full project, but I think it was worth it, it allowed me to better understand how to build an ECS and how to use the Evogym Library even better. 

I will not talk too much about this project because I will make an article on it later (curently writting it lmao)

## Hyperneat and Evogym

### What is Evogym ?

Well evogym is a Python library where you put a tiny robot in an environment, crossing bridges for example and you study how it performs amd how evolution or other ML thecniques can make him perform better. This tiny litlle robot is just a 2D matrix with different type of cells. 

Here is an example of the robot 

robot = [[0, 2, 2, 4, 0],
         [1, 1, 1, 1, 1],
         [2, 3, 3, 3, 2], 
         [4, 4, 4, 4, 4], 
         [3, 3, 0, 0, 4]]

Here the robot is a 5*5 matrix with the following configuration : [0 : empty, 1 : rigid, 2 : soft, 3 : horizontal, 4 : vertical]

See [here](https://evolutiongym.github.io/) for more information. 


### What is Hyperneat ?

Hyperneat is a way of evolving both the body and the controller at the same time. Basically a first NN called CPPN for Compositional Pattern Producing Network will be used to generate the weights of the body network and the controller network. The first one will create the body and the second one will pilot the body during the experiment. The only thing that evolves is the CPPN itself with the neast algorithm. This CPPN start as a simple NN without any hidden layer before the evolution process. Then during the process, its weight are changed, its topology evolve, and crossover only happens between look-alike individuals or should I say between individuals with similar genome not phenotype. Despite that, Fabio decided to implement a new version of Hyperneat where not only the genotypic distance was taken into account but also the phenotypic distance. 

### What is a CPPN ?

CPPN stands for Computational Pattern Producing Netwok ? Some might say that it is simply an ANN (Artificial Neural Network) but it is not. CPPNs in fact are iniatially not designed to have clear layers. They are just an oriented graph where the nodes are the neurons and the edges are the weights between the neurons. In addition to that each node has its own activation function. Furthermore the CPPN usually takes as entry some coordinates. The original paper on CPPN first uses it to creates images with patterns where on the other side, the Hyperneat paper uses it to generate neural networks. Therefore, on one side the CPPN has the coordinate of a point as input (in reality, in this paper the CPPN also had the distance to the center and a static bias of 1) and on the other, the CPPN has the coordinate of two points as input. 

### What is Indirect Encoding 

Indirect Encoding is the underlying process behing HyperNeat where the point is that sometimes, the genome is too big and the direct encoding has to explore ! solution at a time. On the mean time, HyperNeat demonstrates how it is possible to encode larger solution because CPPN encodes the soolution conceptually and the number of resolution only depends on the resolution that you make of the current problem. 

## My Idea 

The task I was given was to work on this work from Fabio Tanaka about HyperNeat and Soft Robots in Evogym. 

## Initial Idea

My initial idea was to explore a new way to encode individuals indirectly. For that I first thought about introducing the concept of diploidy in the project. My first suggestion was that I wanted to reproduce a male and a female, both with different characteristics. The male would a bigger body but the woman would have a denser controller network. I had this idea in front of the tournament system in my ECS example. In this tournament the individual that won would go to a pool and wait for another individual to come and join him in this pool. I wanted for some individual to wait longer, this is where i thought that having sexual indiviodual could help. The problem here is that it does not solve a problem happening in the simulation, and does not focus on something to look for. This is why I decided to go for a similar idea with the objective of focusing on different type of indirect encoding. 

## Another Idea 

In nature, we tend to observe that a population can adapt to a given environment quicly enough to survive. This is due to diploidy, in front of such a good looking behavior I decided to still focus on this idea but this time with a focus on dominance. The dominance is what tells which allele from the two genes should be expressed. The most common exemple here is blood type, all humans have version of this gene, an individual with the A allele and the O allele will express the A allele and its blood type will be A because A is the dominant allele here. Still O continues to exist and can be passed to the next generation. If the other parent also gives an O then the expression of the genes, ie the phenotype, will be O. This child will here express a different phenotype than the one expressed by his parents. Then What I m looking for in this type of genome is that maybe I will be able to explore diverse solutions without putting individuals into look-alike categories (which is exactly what the Neat algorithm is doing, putting individuals into species bring more diversity and allow bad performance in individuals with the expected outcome that these individuals will give better results in the future). By doing so if I do not push to much pressure in my tournament selection, maybe that low fitness individuals that dont produce too bad results will be able to reproduce and spread their genes in the population therefore allowing diverse solutions. 

I will talk later about this diploidy, first lets understand the initial results that I got with an indirect encoding. 

## First Algorithm  

'''text

initialize the population genome with random genes
select with dominance the expressed genome 

for i in range (generations) :
    create body_neural_network
    body = create_body(body_neural_network)

    if body is incorrect : 
       continue 

    create controller_neural_network 

    fitness = simulate(body, controller_neural_network)

    awaiting_to_reproduce = []
    new_population = [best_individual for _ in range(number_of_chose_one)]

    while new_population < max_population :
       awaiting_to_reproduce.append(tournament_winner(population))

       if len(awaiting_to_reproduce) == 2 :
           population.append(reproduce(parent1, parent2))

'''




## First Results

Despite running this experiment on a few generations, the results were bad. Not in the sense that my individuals would not performs well, in fact on the most simple environment I had very good results, but the twist here is that all my individuals would look the same : 

ind = [[4, 4, 4, 4, 4], 
       [4, 4, 4, 4, 4], 
       [4, 4, 4, 4, 4], 
       [4, 4, 4, 4, 4], 
       [4, 4, 4, 4, 4]]

What you see here is globally the form of these individuals. A robot full of vertical actuators that would act like a kangaroo. Even if the controller seems perfectly adapted to the robot (the robot was not extending all of his voxels in one time but those on the left side would contract and expand sooner than those on the right for example). The problem is that, these rob\ot would take control of the population too quicly. In fact, at the first generations, I had a great numner of robot full of 4 or 0. This indicated me that something was wrong in the creation of the body. 

## Explanation 

I see two possibilities here : 

- The substrate has side effect and the cppn is induced in error due to the shape that the substrate possesses 

- at the initialization I dont bring enough diversity in the population so my algorithm tends to converge faster towards an easy solution 

At first I didnt thought about the second solution since the hypothesis of side effect seamed to be the most plausible explanation. In order to correct this I decided to not represent the 5 possibilities of the type of voxels on a line ((-1, -0.5, 0, 0.5, 1) would be associated to (0, 1, 2, 3, 4)). Then what type of geometrical shape would allow me to give every type of voxel the same amount of chance ? The most simple solution for me was a circle. Each point would be position on one of the element of this set ${(cos(i*2*pi/5), sin(i*2*pi/5))} for i \in {0, 1, 2, 3, 4}$.

I still didnt analyze very precisely the different results but it seams, at first glance, that on the first generation I have a more diverse population. With 1/87 that a given individual on a given generation will be 50% like another random individual, I will also soon add a distance also in % for the expressed genotype, ie the cppn, now I just have raw numbers and distance that means something only if you also look at the config.json file. 

Well, this method has still brought to me some good results since my populatin would already look way mor different than the one I had before. 
The thing that I have to test now is to modify even more the shape of the substrate. Remember, I am working with basically a square, and what I did to counter the side effect was to add dimensions so that I could add a circular composant in my substrate. Still, I may still have other side effect, because my square is in fact a side effect by itself. This is why I should maybe try to represent my square shaped robot with a circular substrate. 

robot = [[0, 2, 2, 4, 0],
         [0, 2, 2, 4, 0],
         [0, 2, 0, 4, 0],
         [0, 2, 2, 4, 0],
         [0, 2, 2, 4, 4]]

Here for example, the 0 that you see in the middle of the robot is the center of the circles then the 8 point of the robot array around that 0 are the first circle. Each point on the subrate will be represented by an angle. Then, The last circle on the substrate will be the external layer ont he body of the robot. I will have to make more complete experience to see if there is a real impact, but I think it should really changes how the robot works.

Moreover after a quick time, the population would converge towards a certain type of body, even with diversity on the body on the first generation, the main reason lies in the initialization of the genome. My weights and biases were chosen between -1 and 1, the problem here is that i would not have enough diversity and then the solution would converge too fast toward a single genome, therefore augmenting the probability to possesses the same phenotype. To solve that I increased the range of the random selection to a dozen. The initial results are promising. More stable diversity in the population. What I am afraid of here is that the fitness of the best individual will progress too slowly. More I suspect that it only a matter of time before my population converges again towards a certain type of body. An interesting idea could that like us humans and animals, two individual with a too weak genotypic distance should not reproduce together. Therefore A more diverse populatuion would probably emerges. This is a diversity maintenance method. Another possibility is that because of the diploidy in my genome, I can always bring more diversity in the population, and a simple way to achieve that is by flipping the dominance of each gene more often. Let's see if it works if I increase the generation to a 100. 

The first results seems to look good. I have far more diversity and the genetic distance only goes down after a long time and right after an true elite emerge. To look the results ple\ase look after the analysis notebook, indicate the path. At this point, look for 1/91 and 2/1 and 3/2. The videos are already on youtube. 

## First Conclusion

We have first implemented a GA that evolves diploid individuals, this method is inspired on the one from Cara Reedy, where she uses diploid ANN that controls individuals in a simulation. In here paper she describes that each individual is composed of two chromosomes, one for the layer and one for the weight, for the crossover she just select one chromosome randomly from each parent and then mutate it. My method has quite a few ressemblance with hers : 
For the crossover I also chose randomly a whole chjromosome from each parent and then proceed to mutate it. 
The difference lies in the fact that lmy individuyakls posses a single chromosome and a fixed shape. In this chromosome, the genes represent the node and not the weights. As mentionned ion her article crossover of Neural Network seems to be something that has not yet been fully studied. Therefore knowing if crossover of NN breaks substructures seems to remain an open question. In order to avoid breaking substructures I decided that the whole gene expressed would be a node. Indeed, I think that making many chromosomes would break substructures, for example having a chromosome for the activation function and another one for the biases and another one for the weight, I think would break how the different ellement are working together. Therefore my substructure comes as a package and the one that is activated thank to the dominance is thge node with his activation function, bias, and weights copming from the previous layer. 

Moreover the indirect encoding is inpired from the work of Fabio Tanaka on single genome soft robot, here he uses the algorithm Hyperneat to evolve its robot. My method differs from him in the fact that I choosed a different subrate that for me will bring more diversity without using the speciation technic included in the NEAT library, and that I do not evolve the structure opf the CPPN to focus more on the evolution of diploid individuals. My point here is to not use any technic of speciation. 

Then from now on I will divide my work in two parts : 

- The evolution and comparison of haploid/diplouid individuals in stationary/changing environment. 

- The diversity on the first generated robot with different type of substrate, to see if a circular substrate would not be a better representation of a squared problem. 



### notes 

did with 2 dominances, and hyperneat as tanaka did 
but all individuals looks the same 
need to design again the substrate with some circle because not same distance, more will increase dominance because now 1/2 of genes are picked random;y 
