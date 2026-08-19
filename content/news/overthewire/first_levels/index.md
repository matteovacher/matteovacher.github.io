+++
date = '2026-08-17T14:00:02+02:00'
draft = false 
title = 'Bandit - First Levels'
summary = 'Summary of the first levels of the Bandit challenge.'
+++

## OverTheWire 

Over the wire [see here](https://overthewire.org/wargames/), is a website that offers a large variety of wargames designed to teach differents aspect of security, ranging from basic Linux commands in Bandit to more advanced topics in other game. 

I'll propose here my solutions to the first levels of the Bandit challenges which cover the basic Linux commands. 

## Bandit 0  

On this level you just have to log into the game using SSH. basically SSH (Secure Shell) is a cryptographic protocol that is used to surely connect to remote systems over an unsecured network. Each transfer is encrypted to protect against attacks. 

### Goal 

Log into the game with ssh 

the server is bandit.labs.overthewire.org

the port is 2220

the username is bandit0

the password is bandit0

Find the `readme` file located in the home directory and read it. 

### Theory && Solution 

When you work with your favorite terminal, you can probably use `ssh` to connect to a remote server. If you need to learn more about any command, you can use `man <command>` to get the documentation. 

to connect to a server, the simplest way is to use `ssh <username>@<server>` and then enter the password. Basically here, it should be `ssh bandit0@bandit.labs.overthewire.org` and the password is `bandit0`. But the task is also about connecting on the good port. The solution is then `ssh bandit0@bandit.labs.overthewire.org -p 2220` where the flag `-p` is used to specify the port.

Another fun way to do this is to look up for the IP address of the server with either `nslookup <server>` or `ping <server>`. then instead of `ssh <username>@<server>`, you can use `ssh <username>@<ip>`.

Then using `cat <file>` to read the file you can read the password to enter the next level which gives us `cat ./readme`.

## Bandit 1

### Goal

In this level, the goal is to : 

find the `-` file (located in the home directory) and read it. 

### Theory && Solution

the file is supposed to be in the home directory. Then, the first to do is to look at the files located in the home directory. for that, you can use `ls`. This command basically lists the directory contents. I'll add a bit more flag to better read and understand the output. 
first the flag `-a` is used to list all the files, including the hidden ones. The second flag `-l` is used to list the files in a long format. The third is `-s`, which prints the allocated size of each file in blocks -- that's the number in the leftmost column of the output.

The first command is therefore `ls -las`. 

we obtain this output :

```text
bandit1@bandit:~$ ls -las
total 24
4 -rw-r-----   1 bandit2 bandit1   33 Jun 24 14:58 -
4 drwxr-xr-x   2 root    root    4096 Jun 24 14:58 .
4 drwxr-xr-x 150 root    root    4096 Jun 24 15:02 ..
4 -rw-r--r--   1 root    root     220 Feb 13  2026 .bash_logout
4 -rw-r--r--   1 root    root    3851 Jun 24 14:50 .bashrc
4 -rw-r--r--   1 root    root     807 Feb 13  2026 .profile
```

Now we can understand that the file with the most probability of being the one that contains the password is the `-` file. To be sure we use the command `file <file>` to get the type of the file. And here this how it renders `file ./-`. Here I use `./ because the `.` indicates the current directory and the `/-` the file in the current directory. It also avoid considering the `-` as a flag of the command `file`. Here is the output :

```text
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

```text
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

## Bandit 3

### Goal

The goal of this level is to :

find the password located in an hidden file in the `inhere` directory. 

### Theory && Solution

First we go in the `inhere` directory and with the help of the `ls` command we list the files in the directory> This gives us the following output :

```text
bandit3@bandit:~$ cd inhere/ && ls -las
total 12
4 drwxr-xr-x 2 root    root    4096 Jun 24 14:59 .
4 drwxr-xr-x 3 root    root    4096 Jun 24 14:59 ..
4 -rw-r----- 1 bandit4 bandit3   33 Jun 24 14:59 ...Hiding-From-You
```

Now we can see that the hidden file is called `...Hiding-From-You`. Now we just have to read it using `cat ./...Hiding-From-You`

Here, it is important to notice the that the flag `-a` of the `ls` command is used to list all the files, including the hidden ones and that is the key to solve this level. 

## Bandit 4 

### Goal

The goal here is to : 

Find the password stored in the only human-readable file in the inhere directory. 

### Theory && Solution

Here I begin by going in the `inhere` directory. Then I use the `ls` command to list the files in the directory. Here is the output :

```text
bandit4@bandit:~$ cd inhere/ && ls -las
total 48
4 -rw-r----- 1 bandit5 bandit4   33 Jun 24 14:59 -file00
4 -rw-r----- 1 bandit5 bandit4   33 Jun 24 14:59 -file01
4 -rw-r----- 1 bandit5 bandit4   33 Jun 24 14:59 -file02
4 -rw-r----- 1 bandit5 bandit4   33 Jun 24 14:59 -file03
4 -rw-r----- 1 bandit5 bandit4   33 Jun 24 14:59 -file04
4 -rw-r----- 1 bandit5 bandit4   33 Jun 24 14:59 -file05
4 -rw-r----- 1 bandit5 bandit4   33 Jun 24 14:59 -file06
4 -rw-r----- 1 bandit5 bandit4   33 Jun 24 14:59 -file07
4 -rw-r----- 1 bandit5 bandit4   33 Jun 24 14:59 -file08
4 -rw-r----- 1 bandit5 bandit4   33 Jun 24 14:59 -file09
4 drwxr-xr-x 2 root    root    4096 Jun 24 14:59 .
4 drwxr-xr-x 3 root    root    4096 Jun 24 14:59 ..
```

Here I can see that the `inhere` directory has different files and that I cannot distinguish which one of them is the human readable file. For that I'll use the `file <file>` command to get the information about each file in the current directory. Here is the command and the output :

```text
bandit4@bandit:~/inhere$ file ./* | grep text
./-file07: ASCII text
./-file09: Motorola S-Record; binary data in text format    
```

Let's decrypt the command. The command `file <file>` gives the type of the file and. I use the `./*` to apply the command to all the files in the current directory. It is precisely the `*` that tells the terminal to apply the command to all the files, it basically means "everything inside". The `|` is called a pipe and is there to take the output of the command on the left and give it as input to the command on the right. The next command is `grep <pattern> <file>` where the pattern is the string that we want in the file. Here, with the pipe, we only use `grep <pattern>` because the pipe send the output of the previous command as the input of the `grep`command. 

Now we can see that the only file that is human readable is the `-file07` file. The final command is therefore `cat ./-file07`. 

## Bandit 5 

### Goal

The goal here is to : 
find the password stored in a file somewhere under the inhere directory and that has all of the following properties :

human-readable,
1033 bytes in size,
not executable

### Theory && Solution

Again we begin by going in the `inhere` directory. Then I use the `ls` command to list the files in the directory. Here is the output :

```text
bandit5@bandit:~$ cd inhere/ && ls
maybehere00  maybehere02  maybehere04  maybehere06  maybehere08  maybehere10  maybehere12  maybehere14  maybehere16  maybehere18
maybehere01  maybehere03  maybehere05  maybehere07  maybehere09  maybehere11  maybehere13  maybehere15  maybehere17  maybehere19
```

Let's check the `maybehere00` directory :

```text
bandit5@bandit:~/inhere$ cd maybehere00 && ls -las
total 72
 4 -rwxr-x---  1 root bandit5 1039 Jun 24 14:59 -file1
12 -rw-r-----  1 root bandit5 9388 Jun 24 14:59 -file2
 8 -rwxr-x---  1 root bandit5 7378 Jun 24 14:59 -file3
 4 drwxr-x---  2 root bandit5 4096 Jun 24 14:59 .
 4 drwxr-x--- 22 root bandit5 4096 Jun 24 14:59 ..
 4 -rwxr-x---  1 root bandit5  551 Jun 24 14:59 .file1
 8 -rw-r-----  1 root bandit5 7836 Jun 24 14:59 .file2
 8 -rwxr-x---  1 root bandit5 4802 Jun 24 14:59 .file3
 8 -rwxr-x---  1 root bandit5 6118 Jun 24 14:59 spaces file1
 8 -rw-r-----  1 root bandit5 6850 Jun 24 14:59 spaces file2
 4 -rwxr-x---  1 root bandit5 1915 Jun 24 14:59 spaces file3
```

By doing this again, I understand that each directory contains many files, and the one that I want is located somewhere. To find it let's use the `find <starting point> <filters>` command that scans through the starting directory and all its subfolders to locate a file based on specific criteria. 

Let's use the following command :

```text
bandit5@bandit:~/inhere$ find ./ -type f -size 1033c ! -executable
./maybehere07/.file2
```

Bingo ! We managed to find only one compatible file. Let's explain the command. The `./` is used to indicate the starting directory, the `-type f` means that we are looking for a file, the `-size 1033c` means that the must has a size of 1033 bytes, and the `! -executable` means that the file is not executable. We use the `!` to indicate that we want to exclude the executable files. Normally, I would have used : 

```text
bandit5@bandit:~/inhere$ find -type f -size 1033c ! -executable -exec file {} +
./maybehere07/.file2: ASCII text, with very long lines (1000)
```

Here, we used the `-exec <command> {} +` option to execute a command on each file sent back with the command `find`. In this case, the `file` command is used to determine the type of the file. We can see that the file returned is human-readable. If there were many files returned we could have used `| grep` to filter only the files that are human-readable.

The final command is to read the file : `cat ./maybehere07/.file2`.

## Bandit 6

### Goal

The goal here is to :

Find the password stored somewhere on the server and has all of the following properties:

owned by user bandit7,
owned by group bandit6,
33 bytes in size

### Theory && Solution

For this level, no need to go in the `inhere` directory, we have to find the password somewhere on the whole server. With this, goal in mind let's directly use the `find` command : 

```text
bandit6@bandit:~$ find / -type f -size 33c -user bandit7 -group bandit6
find: ‘/snap’: Permission denied
find: ‘/lost+found’: Permission denied
...............
find: ‘/sys/fs/bpf’: Permission denied
find: ‘/tmp’: Permission denied
```

As we can see, we mostly obtain error messages telling us that we cannot access the directory. So, we'll just shut them down to see if we can at least extract something from the `find` command. Here, the `<starting point>` is basically the root of the system `/`, and I add two new filters : The `-user` filter for the user and the `-group` filter for the group.

```text
bandit6@bandit:~$ find / -type f -size 33c -user bandit7 -group bandit6 2> /dev/null
/var/lib/dpkg/info/bandit7.password
```

Bingo ! Without any error message, there is only one file compatible with all the options. But what did I do ? Well a StackOverflow answer tells us the answer, and I'll just copy paste the answer here : 

```text
As the other answers state, you can use command 2> /dev/null to throw away the error output from command

But what is going on here?

> is the operator used to redirect output. 2 is a reference to the standard error output stream, i.e. 2> = redirect error output.

/dev/null is the 'null device' which just swallows any input provided to it. You can combine the two to effectively throw away output from a command.

Full reference:

    - > /dev/null throw away stdout
    - 1> /dev/null throw away stdout
    - 2> /dev/null throw away stderr
    - &> /dev/null throw away both stdout and stderr
```

Perfect, but I would normally use the following command :

```text
bandit6@bandit:~$ find / -type f -size 33c -user bandit7 -group bandit6 2> /dev/null -exec file {} + | grep text
/var/lib/dpkg/info/bandit7.password: ASCII text
```

This command applies for every file that matches the filters, (and without error message), the command `file`. The output of this `file` command is then piped to the `grep` command to filter only the files that are human-readable.

The final command is therefore `cat /var/lib/dpkg/info/bandit7.password`. 

## Bandit 7

### Goal 

The goal here is to find the password stored in the file `data.txt` next to the word millionth. 

### Theory && Solution

Here, I'll begin by using the `ls` command to confirm that the file is in the current directory.

```text
bandit7@bandit:~$ ls -las
total 4108
   4 drwxr-xr-x   2 root    root       4096 Jun 24 14:59 .
   4 drwxr-xr-x 150 root    root       4096 Jun 24 15:02 ..
   4 -rw-r--r--   1 root    root        220 Feb 13  2026 .bash_logout
   4 -rw-r--r--   1 root    root       3851 Jun 24 14:50 .bashrc
   4 -rw-r--r--   1 root    root        807 Feb 13  2026 .profile
4088 -rw-r-----   1 bandit8 bandit7 4184396 Jun 24 14:59 data.txt
```

And there it is ! If we try to read the file `data.txt` we can see that the file is too long. The first I did it I had to stop the command with `Ctrl+C`... Instead of that I'll use again the pipe and the `grep` command. Here is the command and the output :

```text
bandit7@bandit:~$ cat data.txt | grep millionth
millionth       password here
```

With this command we give all the text of the file to the `grep` command and we ask it to take the line with the word `millionth` and give it to us. This way we can read the password. We could also have more simply used `grep millionth data.txt`. I just prefer the first one. 
































