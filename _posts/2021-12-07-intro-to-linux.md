---
layout: post
title:  "introduction to linux"
date:   2021-12-07
categories: linux
---

After some teacher sponsor troubles, we finally got a meeting together last week, and we talked about Linux fundamentals.

The lecture can be accessed [here](https://docs.google.com/presentation/d/1mIdtKFzs44bbw47A4jj6A7PfgUJq5N-JG6MuYOOLW24/edit).

I made a bunch of tasks for people to familiarize themselves with Linux. For each, I'll explain the command I had in mind to achieve the result and what it does.


### tasks
##### Create your shell server account

We used the [webshell](https://shell.blairsec.mbhs.edu) to access the shared blairsec server without MCPS blocking us. If you want to make an account, contact one of the captains over something like Discord for the security key.

##### View your current username
`whoami`. Just in the command.

##### View your current directory (should be something like /home/)
`pwd`. Again, just in the command.

##### Create two directories with a single command
`mkdir directory1 directory2`. The `mkdir` command accepts multiple directory names.

##### List the directories and files of your home directory
`ls`. Just in the command. If you want to be fancy, `ls -a` shows the hidden files and directories too.

##### Delete the first folder without using rm (there’s a very similar command)
`rmdir directory1`. Though it might not be really useful, `rmdir` removes directories that are empty.

##### Create an empty file in the second folder
`cd directory2; touch file1.txt`. First we change directory to `directory2`, and then `touch command` creates an empty file.

You could also just use `touch directory2/file1.txt`

##### Change to the home directory
`cd ~`. The `~` means the home directory for the current user.

##### List all files in the new folder while you are in the home directory
`ls directory2`. The `ls` command can take optional arguments which are the names of the directories or files you want to `ls`.

##### Write the current time into the empty file without looking at a clock (commands only)
`date > file1.txt`. The `date` command returns a string containing date and time information, and the `> file1.txt` writes the output into `file1.txt`.

##### Make a copy of the file and keep it in the same directory
`cp file1.txt file2.txt`. When we `cp` with two arguments which are just two filenames, it creates a new file from the second name and copies the contents of the first into the second.

##### Write the contents of your home directory into the original file (overwriting)
`ls ~ > file1.txt`. `ls ~` lists the contents of the home directory, and `> file1.txt` writes the output into `file1.txt`.

##### Write the contents of your home directory into the copied file (no overwriting)
`ls ~ >> file2.txt`. `ls ~` lists the contents of the home directory, and `>> file1.txt` writes the output into `file2.txt`.

##### Make a copy of the folder you created
`cp -r directory2 directory3`. `cp -r` recursively copies files from one directory to another. Note that `directory3` doesn't exist, so it is created and just the  files are put into the new directory:

```
.
├── directory2
│   ├── file1.txt
│   └── file2.txt
└── directory3
    ├── file1.txt
    └── file2.txt
```

If `directory3` already existed, then it would copy `directory2` itself into `directory3`:

```
.
├── directory2
│   ├── file1.txt
│   └── file2.txt
└── directory3
    └── directory2
        ├── file1.txt
        └── file2.txt
```


##### Delete the original folder
`rm -r directory2`. The `rm` command deletes stuff, and `-r` is the recursive flag so directories will be deleted. You could also use `-rf` like me if you don't want deletion confirmation in certain situations.

##### List the permissions of the copied folder and every file inside of it
`ls -l directory3`. The `-l` command flag means to use "long listing format", which gives more details including file permissions. 

##### Make the second file un-writable for the current user
`chmod u-w directory3/file2.txt`. `chmod` modifies file permissions. `u` refers to the file's owner (**u**ser) which is us (in contrast to `g` which refers to the file's **g**roup, or `o` which refers to **o**ther people). 

##### Try to write to the second file and see it fail (permission denied)
To test this, I used `echo 1 > directory3/file2.txt`. `echo 1` prints out a `1` and `>` tries to write that into `directory3/file2.txt`. However, we get back `bash: directory2/file2.txt: Permission denied`, which means our permission setting worked and we cannot write to the file.

##### Get an ASCII locomotive to run across your screen (try making it fly too)
`sl` command. It was specifically created to annoy people who frequently typo `ls`. To make it fly, use `-F`. There are a bunch of other cool options in the man(ual) page for it - `man sl` to view it.

![result.png](/writeups/assets/images/12-07-21/sl.png)

See you at the next meeting, whenever that is!

~ josh

