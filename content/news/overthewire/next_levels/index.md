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

## Bandit 9 

### Goal

The goal of this level is to find the password stored in the file `data.txt` in one of the few human-readable strings, preceded by several ‘=’ characters.

### Theory && Solution

To solve this level, I'll mainly use the command `strings` which purpose is to print the printable character sequences that are at least 4 characters long and that are followed by an unprintable character. 

Let's begin by scanning the current repository and print the first lines of the file `data.txt` :

```text
bandit9@bandit:~$ ls -las
total 40
 4 drwxr-xr-x   2 root     root     4096 Jun 24 14:58 .
 4 drwxr-xr-x 150 root     root     4096 Jun 24 15:02 ..
 4 -rw-r--r--   1 root     root      220 Feb 13  2026 .bash_logout
 4 -rw-r--r--   1 root     root     3851 Jun 24 14:50 .bashrc
 4 -rw-r--r--   1 root     root      807 Feb 13  2026 .profile
20 -rw-r-----   1 bandit10 bandit9 19382 Jun 24 14:58 data.txt
```

and 

```text
bandit9@bandit:~$ head -3 data.txt
�
�j"+V[��ot��je}<��99��6I%���Q���
7���^X�����plM&]��Gl�J��O��))Q�u
                                1y1J�h?H�Z��g�̾Ε�ݭ3�r�&}�7��4�=�#5�e
                                                                   +�9���u*-D@6@�17�G��(]7�D[�������"�+v�$��5�8t�����h�Fr�N����
                         ��0�A�4�fҷ�)(C����0�6��|>������?�'��6���m�2�������<�t��%�a)�3��␦�\��*ڌ�`54�?Ws�����k�x�/=�&m���(�W���R0W�'��)�:%v��L�cW��7�0�1vB����Y}fti�&N�b>L*�z]U}yb�����/$H������r��\Ǖ�YH����of3Q������#w�CթԤ��0!M��
```

Along with the command `strings` I use the pipe and grep command to print the lines that include the `=` character since the password is supposed to be processed by several of this string. 

```text 
bandit9@bandit:~$ strings data.txt | grep ===
cL0========== the
========== password
>========== is
R========== Password Here 
```

And bingo, here it is. 

## Bandit 10

### Goal

The goal of this level is to find the password stored in the file data.txt, which contains base64 encoded data. 

### Theory && Solution

First, what are base64 encoded data ? Base 64 is a method of turning binary data into plain text using a safe alphabet of 64 characters. For example 1 byte corresponds to 8 bits, then a string of 3 bytes can be converted into 4 base64 characters. Why is that ? Because a single characters in base64 represents 6 binary bits, so 4 base64 characters represent a sequence of 24 bits, but 3 bytes also represent a sequence of 24 bits. 

The command `base64` is used to either encode or decode base64 data. The flag `-d` is used to decode the data which means going from a base64 string to the original data. 

Let's see what's inside the file `data.txt` :

```text
bandit10@bandit:~$ cat data.txt
VGhlIHBhc3N3b3JkIGlzIHBZZk9ZNkh3VXNEajVyTDlVdnloVTdNQ212OHZONVJvCg==
```

Of course we cannot read it, we need to decode it first and for that let's use the `base64` command with the `-d` option.

```text
bandit10@bandit:~$ base64 -d data.txt
The password is <password here>
```

Et voila, here is the password. 

## Bandit 11 

### Goal

The goal here is to find the password stored in the file `data.txt`, where all lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions. 

### Theory && Solution

To solve this level we need to use the `tr <set1> <set2>` command where <set1> is the set of characters we want to replace and <set2> is the set of characters we want to replace them with. 

To explore a little bit more before trying the final answer, let's use the command. 

```text 
bandit11@bandit:~$ cat data.txt | tr c a && cat data.txt
Gur anffjbeq vf TEBbmJCB8DlA0zTewHxVQ0JPLxMvDkeA
Gur cnffjbeq vf TEBbmJCB8DlA0zTewHxVQ0JPLxMvDkeA
```

As you can see there, I wanted to change the letter `c` with the letter `a` and it worked. We can also change whole set of letters. 

```text
bandit11@bandit:~$ cat data.txt | tr a-z b-za && cat data.txt
Gvs doggkcfr wg TEBcnJCB8DmA0aTfxHyVQ0JPLyMwDlfA
Gur cnffjbeq vf TEBbmJCB8DlA0zTewHxVQ0JPLxMvDkeA
```

Here I understood that this command changed the letters by replacing the letter by the next letter in the alphabet. 

Then we can try writting the final command : 

```text
bandit11@bandit:~$ cat data.txt | tr a-zA-Z n-za-mN-ZA-M
The password is <password here>
```

Et voila, here it is. I have switched the letters by exactly 13 positions.

## Bandit 12

### Goal 

The goal here is to find the password stored in the file `data.txt`, which is a hexdump of a file that has been repeatedly compressed. 

Wow, I've never seen a level this boring before. Right now I'm still decompressing all the files...

### Theory && Solution

I'll be quick there because I did'nt really enjoy this level...

First let's begin by copying the file into a directory in the `tmp` directory. For this let's use the `mkdir <path>` and the `cp <file> <path>` commands.

```text
bandit12@bandit:~$ mkdir ./../../tmp/simpleibaraki/ && cp data.txt ./../../tmp/simpleibaraki/
```

Then using the `cp` command again I create a folder where I can text anything on the file : 

```text
bandit12@bandit:/tmp/simpleibaraki$ mkdir ./try && cp data.txt ./try
```

Now, the first thing said about the file is that it is a hexdump of a file. We need to convert this hexdump into a binary and for that let's use the `xxd <file>` command with the `-r` option to convert the hexdump into a binary file. 

```text
bandit12@bandit:/tmp/simpleibaraki$ cd try && xxd -r data.txt > thisfile
bandit12@bandit:/tmp/simpleibaraki/try$ ls -las
total 16
 0 drwxrwxr-x 2 bandit12 bandit12    80 Aug 25 18:06 .
 0 drwxrwxr-x 4 bandit12 bandit12   100 Aug 25 18:06 ..
 4 -rw-r----- 1 bandit12 bandit12  2641 Aug 25 18:00 data.txt
12 -rw-rw-r-- 1 bandit12 bandit12 11273 Aug 25 18:06 thisfile
``` 

Here we used the `>` to redirect the output of the `xxd` command to the file `thisfile`.

```text
bandit12@bandit:/tmp/simpleibaraki/try$ file thisfile
thisfile: gzip compressed data, was "data2.bin", last modified: Wed Jun 24 14:58:46 2026, max compression, from Unix, original size modulo 2^32 580
```

The `file command told us that the file is a gzip compressed file. so let's try to decompress it with the `gunzip <file>` command. 

```text
bandit12@bandit:/tmp/simpleibaraki/try$ gunzip thisfile
gzip: thisfile: unknown suffix -- ignored
``` 

Well, I didn't know but here it is needed to call the file `thisfile.gz` instead of `thisfile`.

```text
bandit12@bandit:/tmp/simpleibaraki/try$ mv thisfile thisfile.gz && gunzip thisfile.gz
bandit12@bandit:/tmp/simpleibaraki/try$ ls -las
total 8
0 drwxrwxr-x 2 bandit12 bandit12   80 Aug 25 18:11 .
0 drwxrwxr-x 4 bandit12 bandit12  100 Aug 25 18:06 ..
4 -rw-r----- 1 bandit12 bandit12 2641 Aug 25 18:00 data.txt
4 -rw-rw-r-- 1 bandit12 bandit12  580 Aug 25 18:08 thisfile
bandit12@bandit:/tmp/simpleibaraki/try$ file thisfile
thisfile: bzip2 compressed data, block size = 900k
```

And here we are with again another compressed file...

```text 
bandit12@bandit:/tmp/simpleibaraki/try$ bunzip2 thisfile && ls -las
bunzip2: Can't guess original name for thisfile -- using thisfile.out
total 8
0 drwxrwxr-x 2 bandit12 bandit12   80 Aug 25 18:13 .
0 drwxrwxr-x 4 bandit12 bandit12  100 Aug 25 18:06 ..
4 -rw-r----- 1 bandit12 bandit12 2641 Aug 25 18:00 data.txt
4 -rw-rw-r-- 1 bandit12 bandit12  438 Aug 25 18:08 thisfile.out
bandit12@bandit:/tmp/simpleibaraki/try$ file thisfile.out
thisfile.out: gzip compressed data, was "data4.bin", last modified: Wed Jun 24 14:58:46 2026, max compression, from Unix, original size modulo 2^32 20480
```

Again, another compressed file...

```text
bandit12@bandit:/tmp/simpleibaraki/try$ mv thisfile.out thisfile.gz && gunzip thisfile.gz
bandit12@bandit:/tmp/simpleibaraki/try$ ls -las
total 24
 0 drwxrwxr-x 2 bandit12 bandit12    80 Aug 25 18:15 .
 0 drwxrwxr-x 4 bandit12 bandit12   100 Aug 25 18:06 ..
 4 -rw-r----- 1 bandit12 bandit12  2641 Aug 25 18:00 data.txt
20 -rw-rw-r-- 1 bandit12 bandit12 20480 Aug 25 18:08 thisfile
bandit12@bandit:/tmp/simpleibaraki/try$ file thisfile
thisfile: POSIX tar archive (GNU)
```

Well it never stops... 

```text
bandit12@bandit:/tmp/simpleibaraki/try$ tar -x thisfile
tar: Refusing to read archive contents from terminal (missing -f option?)
tar: Error is not recoverable: exiting now
bandit12@bandit:/tmp/simpleibaraki/try$ tar -xf thisfile
bandit12@bandit:/tmp/simpleibaraki/try$ ls -las
total 36
 0 drwxrwxr-x 2 bandit12 bandit12   100 Aug 25 18:26 .
 0 drwxrwxr-x 4 bandit12 bandit12   100 Aug 25 18:06 ..
 4 -rw-r----- 1 bandit12 bandit12  2641 Aug 25 18:00 data.txt
12 -rw-r--r-- 1 bandit12 bandit12 10240 Jun 24 14:58 data5.bin
20 -rw-rw-r-- 1 bandit12 bandit12 20480 Aug 25 18:08 thisfile
bandit12@bandit:/tmp/simpleibaraki/try$ file data5.bin
data5.bin: POSIX tar archive (GNU)
bandit12@bandit:/tmp/simpleibaraki/try$ tar -xf data5.bin
bandit12@bandit:/tmp/simpleibaraki/try$ ls -las
total 40
 0 drwxrwxr-x 2 bandit12 bandit12   120 Aug 25 18:27 .
 0 drwxrwxr-x 4 bandit12 bandit12   100 Aug 25 18:06 ..
 4 -rw-r----- 1 bandit12 bandit12  2641 Aug 25 18:00 data.txt
12 -rw-r--r-- 1 bandit12 bandit12 10240 Jun 24 14:58 data5.bin
 4 -rw-r--r-- 1 bandit12 bandit12   223 Jun 24 14:58 data6.bin
20 -rw-rw-r-- 1 bandit12 bandit12 20480 Aug 25 18:08 thisfile
bandit12@bandit:/tmp/simpleibaraki/try$ file data6.bin
data6.bin: bzip2 compressed data, block size = 900k
bandit12@bandit:/tmp/simpleibaraki/try$ bunzip2 data6.bin
bunzip2: Can't guess original name for data6.bin -- using data6.bin.out
bandit12@bandit:/tmp/simpleibaraki/try$ ls -las
total 48
 0 drwxrwxr-x 2 bandit12 bandit12   120 Aug 25 18:28 .
 0 drwxrwxr-x 4 bandit12 bandit12   100 Aug 25 18:06 ..
 4 -rw-r----- 1 bandit12 bandit12  2641 Aug 25 18:00 data.txt
12 -rw-r--r-- 1 bandit12 bandit12 10240 Jun 24 14:58 data5.bin
12 -rw-r--r-- 1 bandit12 bandit12 10240 Jun 24 14:58 data6.bin.out
20 -rw-rw-r-- 1 bandit12 bandit12 20480 Aug 25 18:08 thisfile
bandit12@bandit:/tmp/simpleibaraki/try$ file data6.bin.out
data6.bin.out: POSIX tar archive (GNU)
bandit12@bandit:/tmp/simpleibaraki/try$ tar -xf data6.bin.out
bandit12@bandit:/tmp/simpleibaraki/try$ ls -las
total 52
 0 drwxrwxr-x 2 bandit12 bandit12   140 Aug 25 18:29 .
 0 drwxrwxr-x 4 bandit12 bandit12   100 Aug 25 18:06 ..
 4 -rw-r----- 1 bandit12 bandit12  2641 Aug 25 18:00 data.txt
12 -rw-r--r-- 1 bandit12 bandit12 10240 Jun 24 14:58 data5.bin
12 -rw-r--r-- 1 bandit12 bandit12 10240 Jun 24 14:58 data6.bin.out
 4 -rw-r--r-- 1 bandit12 bandit12    79 Jun 24 14:58 data8.bin
20 -rw-rw-r-- 1 bandit12 bandit12 20480 Aug 25 18:08 thisfile
bandit12@bandit:/tmp/simpleibaraki/try$ file data8.bin
data8.bin: gzip compressed data, was "data9.bin", last modified: Wed Jun 24 14:58:46 2026, max compression, from Unix, original size modulo 2^32 49
bandit12@bandit:/tmp/simpleibaraki/try$ mv data8.bin data8.gz && gunzip data8.gz
bandit12@bandit:/tmp/simpleibaraki/try$ ls -las
total 52
 0 drwxrwxr-x 2 bandit12 bandit12   140 Aug 25 18:30 .
 0 drwxrwxr-x 4 bandit12 bandit12   100 Aug 25 18:06 ..
 4 -rw-r----- 1 bandit12 bandit12  2641 Aug 25 18:00 data.txt
12 -rw-r--r-- 1 bandit12 bandit12 10240 Jun 24 14:58 data5.bin
12 -rw-r--r-- 1 bandit12 bandit12 10240 Jun 24 14:58 data6.bin.out
 4 -rw-r--r-- 1 bandit12 bandit12    49 Jun 24 14:58 data8
20 -rw-rw-r-- 1 bandit12 bandit12 20480 Aug 25 18:08 thisfile
bandit12@bandit:/tmp/simpleibaraki/try$ file data8
data8: ASCII text
bandit12@bandit:/tmp/simpleibaraki/try$ cat data8
The password is <password here>
```

Well, I finally saw the end, it was basically just decompressing each file and renaming the file that needed to be renamed. One thing is that for the command `tar` to work you need to put the `-f` flag to indicate that the next thing coming is the file that you want to extract. Also the flag `-x` is there to indicate that we are extracting.



