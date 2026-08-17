+++
date = '2026-17-08T14:00:02+02:00'
draft = true 
title = 'Bandit - First Levels'
summary = 'summary of the first levels of the Bandit challenge.'
+++

## OverTheWire 

Over the wire [see here](https://overthewire.org/wargames/), is a website that offers a large variety of wargames designed to teach differents aspect of security, ranging from basic Linux commands in Bandit to more advanced topics in other game. 

I'll propose here my solutions to the first levels of the Bandit challenges which cover the basic Linux commands. 

## Bandit 0 

On this level you just have to log into the game using SSH. basically SSH (Secure Shell) is a cryptographic protocol that is used to surely connect to remote systems over an unsecured network. Each transfer is encrypted to protect against attacks. 

### Goal 

Log intto the game with ssh 

the server is bandit.labs.overthewire.org

the port is 2220

the username is bandit0

the password is bandit0

### Theory && Solution 

When you work with your favorite terminal, you can probably use `ssh` to connect to a remote server. If you need to learn more about any command, you can use `man <command>` to get the documentation. 

to connect to a server, the simplest way is to use `ssh <username>@<server>` and then enter the password. Basically here, it should be `ssh bandit0@bandit.labs.overthewire.org` and the password is `bandit0`. But the task is also about connecting on the good port. The solution is then `ssh bandit0@bandit.labs.overthewire.org -p 2220` where the flag `-p` is used to specify the port.

Another fun way to do this is to look up for the IP address of the server with either `nslookup <server>` or `ping <server>`. then instead of `ssh <username>@<server>`, you can use `ssh <username>@<ip>`.

Then using `cat <file>` to read the file you can read the password to enter the next level.










