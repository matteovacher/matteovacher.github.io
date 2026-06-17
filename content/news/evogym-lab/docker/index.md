+++
date = '2026-06-16T03:24:30+02:00'
draft = true
title = 'How to use Docker for Evogym'
summary = 'Here is a tutorial on how to use Docker for the Evogym library. This tutorial also contains general information on Docker'
+++

## Introduction 

When developing software or running your experiments, it may be required to work with several other people. During the test phases, everything might work very well on your computer, with your current OS and IDE. The problem here is that someone else might want to continue your work and test your code using a different OS or IDE, therefore *leading to multiple errors* not because the code is wrong but because the initial environment is different. The solution here is simple : **use Docker**.

Docker is an platform for developing and running applications. It allows you to to separate the applications or the experiments from the infrastructure. It is basically just running your application in a box that represent the universal infrastructure for the current work. 

In this article, I will explain how to use Docker with the example of my project. I'll let the reader find additional information if my tutorial don't cover everything (I am not building an application, but running experiments).

## Docker Objects 

### Images 

An image is a **template with instruction**s** for creating the Docker container *(the box mentioned above)*. It is possible to us already existing images created by other people or to create your own. In order to create your own you need to create a Dockerfile. 

### Dockerfile

The Dockerfile is a file located at the root of the project that will contains the **commands to build an image**. This DOckerfile will contains different commands on each line, and each of these instruction will be converted into a layer on that image. The goal here is that if you add a line instruction to your Dockerfile, Docker will add a layer on the pre-existing image instead og going through the process of executing all the previous instructions again. 

### Containers 

A container is a **runnable instance of an image**. A container *is defined by its image*. It is basically an isolated box on your computer that contains everything you need to run your application or run your experiments. 

### Summary 

To run your application or run your experiments into an isolated and universal box, you need to be *inside a container*. The **container is the box**. But to run this container you need an **image**. An image is a template that contains every program or external dependency that you need to run your project. And to obtain that image you can either download a pre-existing one or create your own using a **Dockerfile**, a file that contains the different instructions to build the image by adding layers to the already existing base.

## How to create your Dockerfile 

Beginning with the basics, create a file called `Dockerfile` at the root of the project. 

Then we need need to create the first instruction for the image, ie the base image. A base image is a pre-built Docker image that will start as the first layer for the final image. The first instruction is `FROM python:3.10-slim`. This command will install python 3.10 on along with the minimal Debian package (the minimal package come from the `slim` in the command line, without it a bigger version of Debian would be installed). Debian is a free and open source operating system (OS) based on the Linux kernel and the GNU system. Here the 3.10 version of Python is chosen because Evogym only supports Python 3.7 to 3.10 (on most operating system, see the installation tutorial [here](https://github.com/EvolutionGym/evogym)). 

Here comes the second line of the Dockerfile, which purpose is to set the current working directory : `WORKDIR /app`. It is basically the same command as the `cd` command in the terminal. It is possible to work directly in the root of the container image but it is considered a good practice to work . It allows you to avoid typing the full path over and over. The future commands will automatically be executed in this directory. 

Next, you need to install the different software for running correctly Evogym. Here comes the new line : `RUN apt-get update && apt-get install -y xorg-dev libglu1-mesa-dev xvfb`. The `RUN` instruction executes commands in the image. Each `RUN` will add a new layer to the image. Also, please notice that on this line we use `&&` which means that we execute to `RUN` in one. Instead of having 2 layers, we end up only having 1. The `apt-get update` instruction will simply update the list of available packages for our Debian system. But, What are we supposed to install ? The response lies again in the installation tutorial [here](https://github.com/EvolutionGym/evogym) :

- xorg-dev libglu1-mesa-dev are required for a linux OS (as said in the tutorial) 

- xvfb is required because when saving the videos of the robot you'll need to produce images without printing it on the screen. I'll explain l ater this part. 








