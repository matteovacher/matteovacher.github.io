+++
date = '2026-08-19T10:49:41+02:00'
draft = true
title = 'Bandit - The Next Part'
summary = 'Summary of the next levels of the Bandit challenge'
+++

Hello again, I'll continue to explain the next levels of the Bandit challenges. 

## Bandit 8 

### Goal

The password here is stored in the file `data.txt` and is the only line of text that occurs only once

### Theory && Solution

The command `uniq <file>` is the key to solve this level. The `uniq` command can report or omit repeated lines. The thing is that it can only filter adjacent matching lines, which means that if you have a that occur more than once but are not adjacent to each other, they will not be filtered out. In order to filter the lines, we first need to use the `sort` command to sort the lines of the file.

First, let's try the following command :

```text 
bandit8@bandit:~$ sort -g data.txt | head -5
AbdnbiBbaVYF6bUDsSIaWCRNdVmr7b8h
AbdnbiBbaVYF6bUDsSIaWCRNdVmr7b8h
AbdnbiBbaVYF6bUDsSIaWCRNdVmr7b8h
AbdnbiBbaVYF6bUDsSIaWCRNdVmr7b8h
AbdnbiBbaVYF6bUDsSIaWCRNdVmr7b8h
```

Here we used the `sort` command to sort according to the string numerical value with the `-g` option. We then used the `head` command to print the first 5 lines of the sorted file. If we printed more lines, we would see that they are sorted. 

The final command is the following :

```text
bandit8@bandit:~$ sort -g data.txt | uniq -u
password here 
```

I used here the `uniq` command with the `-u` option to filter out the lines that occur more than once. The output is then the only line of text that occurs only once. Along with the `uniq` command I used the pipe `|` to put sorted lines to the `uniq` command.

