+++
date = '2026-06-16T03:24:30+02:00'
draft = true
title = 'How to use Docker for Evogym'
summary = 'Here is a tutorial on how to use Docker for the Evogym library. This tutorial also contains general information on Docker'
+++

## Introduction 

When developing software or running your experiments, it may be required to work with several other people. During the test phases, everything might work very well on your computer, with your current OS and IDE. The problem here is that someone else might want to continue your work and test your code using a different OS or IDE, therefore *leading to multiple errors* not because the code is wrong but because the initial environment is different. The solution here is simple : **use Docker**.

Docker is an platform for developing and running applications. It allows you to to separate the applications or the experiments from the infrastructure. It is basically just running your application in a box that represent the universal infrastructure for the current work. 

## Docker Objects 

### Images 

An image is a **template with instruction**s** for creating the Docker container *(the box mentioned above)*. It is possible to us already existing images created by other people or to create your own. In order to create your own you need to create a Dockerfile. 

### Dockerfile

The Dockerfile is a file located at the root of the project that will contains the **commands to build an image**. This DOckerfile will contains different commands on each line, and each of these instruction will be converted into a layer on that image. The goal here is that if you add a line instruction to your Dockerfile, Docker will add a layer on the pre-existing image instead og going through the process of executing all the previous instructions again. 

### Containers 

A container is a **runnable instance of an image**. A container *is defined by its image*. It is basically an isolated box on your computer that contains everything you need to run your application or run your experiments. 

### Summary 

To run your application or run your experiments into an isolated and universal box, you need to be *inside a container*. The **container is the box**. But to run this container you need an **image**. An image is a template that contains every program or external dependency that you need to run your project. And to obtain that image you can either download a pre-existing one or create your own using a **Dockerfile**, a file that contains the different instructions to build the image by adding layers to the already existing base.



