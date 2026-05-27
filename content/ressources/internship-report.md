+++
date = '2026-05-25T11:52:31+02:00'
draft = true
title = 'Internship Report'
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



### notes 

did with 2 dominances, and hyperneat as tanaka did 
but all individuals looks the same 
need to design again the substrate with some circle because not same distance, more will increase dominance because now 1/2 of genes are puicked randomly 
