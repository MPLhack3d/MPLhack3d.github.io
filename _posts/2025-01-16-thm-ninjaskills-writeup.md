---
title: "Ninja Skills"
date: 2025-01-16
image: /assets/img/tryhackme/NinjaSkills/NinjaSkills_image.jpg
description: Writeup of the TryHackMe-CTF Ninja Skills
categories: [Tryhackme, Easy]
tags: [bash, linux, scripting]
---

## Challenge Description
<center>
<table>
  <tr>
    <td>Platform</td>
    <td>TryHackMe</td>
  </tr>
  <tr>
    <td>Name</td>
    <td>Ninja Skills</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Practise your Linux skills and complete the challenges.</td>
  </tr>
  <tr>
    <td>Difficulty</td>
    <td>Easy</td>
  </tr>
  <tr>
    <td>OS</td>
    <td>Linux</td>
  </tr>
  <tr>
    <td>Link</td>
    <td><a href="https://tryhackme.com/r/room/ninjaskills">https://tryhackme.com/r/room/ninjaskills</a></td>
  </tr>
</table>
</center>

The challenge description give use SSH credentials to directly connect to the machine.

```text
  Let's have some fun with Linux. Deploy the machine and get started.

This machine may take up to 3 minutes to configure.

(If you prefer to SSH into the machine, use the credentials new-user as the username and password)

Answer the questions about the following files:

8V2L
bny0
c4ZX
D8B3
FHl1
oiMO
PFbD
rmfX
SRSq
uqyw
v2Vb
X1Uy
The aim is to answer the questions as efficiently as possible.
```
The last sentence seems to indicate that automation, i.e. scripting, may be necessary

## Connect
The ssh connection didn't work directly, so grab youself some coffee and wait a bit. 
- Username: new-user
- Password: new-user

```bash
Last login: Wed Jan  8 19:37:08 2025 from ip-10-100-1-175.eu-west-1.compute.internal
████████╗██████╗ ██╗   ██╗██╗  ██╗ █████╗  ██████╗██╗  ██╗███╗   ███╗███████╗
╚══██╔══╝██╔══██╗╚██╗ ██╔╝██║  ██║██╔══██╗██╔════╝██║ ██╔╝████╗ ████║██╔════╝
   ██║   ██████╔╝ ╚████╔╝ ███████║███████║██║     █████╔╝ ██╔████╔██║█████╗  
   ██║   ██╔══██╗  ╚██╔╝  ██╔══██║██╔══██║██║     ██╔═██╗ ██║╚██╔╝██║██╔══╝  
   ██║   ██║  ██║   ██║   ██║  ██║██║  ██║╚██████╗██║  ██╗██║ ╚═╝ ██║███████╗
   ╚═╝   ╚═╝  ╚═╝   ╚═╝   ╚═╝  ╚═╝╚═╝  ╚═╝ ╚═════╝╚═╝  ╚═╝╚═╝     ╚═╝╚══════╝
        Let the games begin!
[new-user@ip-10-10-87-136 ~]$
```

## Challenge
As the challenge said we need to answer questions about the mentioned files. In the users home directory there is a 'files' directory:
```bash
[new-user@ip-10-10-87-136 ~]$ ls
files
[new-user@ip-10-10-87-136 ~]$ cd files/
[new-user@ip-10-10-87-136 files]$ ls -al
total 8
drwxrwxr-x 2 new-user new-user 4096 Oct 23  2019 .
drwx------ 3 new-user new-user 4096 Oct 23  2019 ..
```
The folder is empty. After some manual searching I found the first file in `/home`:
```bash
[new-user@ip-10-10-87-136 home]$ ls
ec2-user  newer-user  new-user  v2Vb
```
The task is to search all the given files, analyze them and answer the question. But we need to do this as efficiently as possible.

For solving this challenge I wrote the following small bash script:
```bash
#!/bin/bash

while IFS= read -r line
do
        path=$(find / -name $line -print 2>/dev/null)   # search for the file
        if [ -z "$path" ]; then                         # check if the path is empty 
            continue                                    # skip if so
        fi
        echo -n "$path          "                       # print out the filepath
        eval "$2 \"$path\"" 2>/dev/null                 # execute the provided command
done < "$1"                                             # read the file list file
```
I created the file using `nano` and made it executable:
```bash
[new-user@ip-10-10-87-136 ~]$ nano thm_finder.sh
[new-user@ip-10-10-87-136 ~]$ chmod +x thm_finder.sh
```
Next we need to create a file with all the file names we have received. I simply called it `file_list`.
The list could be as long as possible, so I think we've reached the efficient part here.

Now we can call the script with the following syntax:
```bash
./thm_finder.sh file_list "command you want to execute"
# e.g. the following command exectues 'ls' on all files:
./thm_finder.sh file_list "ls"
```

With this preperation, we can answer the questions one by one:
1. `./thm_finder.sh file_list "ls -al"`
2. `./thm_finder.sh file_list "grep -Eo \"([0-9]{1,3}\.){3}[0-9]{1,3}\""`
3. `./thm_finder.sh file_list "sha1sum"`
4. `./thm_finder.sh file_list "wc -l"`
5. `./thm_finder.sh file_list "ls -ln"`
6. `./thm_finder.sh file_list "ls -al"`

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> There is one file which could not be found, have that in mind :)
{: .prompt-tip }
<!-- markdownlint-restore -->

solved! :)
