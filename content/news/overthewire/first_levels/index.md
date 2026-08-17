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

## Bandit 1

### Goal

In this level, the goal is to : 

find the readme file (located in the home directory) and read it. 

### Theory && Solution

the file is supposed to be in the home directory. Then, the first to do is to look at the files located in the home directory. for that, you can use `ls`. This command basically lists the directory contents. I'll add a bit more flag to better read and understand the output. 
first the flag `-a` is used to list all the files, including the hidden ones. The second flag `-l` is used to list the files in a long format. The third is `-s` which is basically used to store the file. 

The first ccommand is therefore `ls -las`. 

we obtain this output :

text```
bandit1@bandit:~$ ls -las
total 24
4 -rw-r-----   1 bandit2 bandit1   33 Jun 24 14:58 -
4 drwxr-xr-x   2 root    root    4096 Jun 24 14:58 .
4 drwxr-xr-x 150 root    root    4096 Jun 24 15:02 ..
4 -rw-r--r--   1 root    root     220 Feb 13  2026 .bash_logout
4 -rw-r--r--   1 root    root    3851 Jun 24 14:50 .bashrc
4 -rw-r--r--   1 root    root     807 Feb 13  2026 .profile
```

Now we can understand that the file with the most probability of being the one that contains the password is the `-` file. To be sure we use the command 'file <file>' to get the type of the file. And here this how it renders `file ./-`. Here I use `./ because the `.` indicates the current directory and the `/-` the file in the current directory. It also avoid considering the `-` as a flag of the command `file`. Here is the output :

text```
bandit1@bandit:~$ file ./-
./-: ASCII text
```

now we know that the file is indeed a text file. So now we can just read it with the `cat` command and go to the next level. the final command is : `cat ./-`

## Bandit 2

### Goal 

The goal of this level is to : 

Find the password in a file called `--spaces in this filename--` located in the home directory.

### Theory && Solution

Again I begin by using the `ls` command and here is the result : 

text```
bandit2@bandit:~$ ls -las
total 24
4 -rw-r-----   1 bandit3 bandit2   33 Jun 24 14:59 --spaces in this filename--
4 drwxr-xr-x   2 root    root    4096 Jun 24 14:59 .
4 drwxr-xr-x 150 root    root    4096 Jun 24 15:02 ..
4 -rw-r--r--   1 root    root     220 Feb 13  2026 .bash_logout
4 -rw-r--r--   1 root    root    3851 Jun 24 14:50 .bashrc
4 -rw-r--r--   1 root    root     807 Feb 13  2026 .profile
```

We can see that the flag is present in the home directory. To read it we just use the `cat` command. `cat ./--spaces\ in\ this\ filename--` is a first way of doing it. Here the `\` are used to quote the name of the file. It tells the terminal that the space belongs to the name of the file. If I had use `cat ./--spaces in this filename--` the terminal would have consider each word as a file to read. I could also have used `cat './--spaces in this filename--'` but this time the `'` are used to quote the entire name of the file. This option can be preferred because it is more readable. 















