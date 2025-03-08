---
title: "Grep"
date: 2025-02-20
image: /assets/img/tryhackme/Grep/Grep_image.jpg
description: Writeup of the TryHackMe-CTF Grep
categories: [Tryhackme, Easy]
tags: [linux, web]
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
    <td>Grep</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>A challenge that tests your reconnaissance and OSINT skills.</td>
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
    <td><a href="https://tryhackme.com/room/greprtp">https://tryhackme.com/room/greprtp</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Grep.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.28.42            
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-20 05:01 EST
  Nmap scan report for 10.10.28.42
  Host is up (0.042s latency).
  Not shown: 65531 closed tcp ports (conn-refused)
  PORT      STATE SERVICE
  22/tcp    open  ssh
  80/tcp    open  http
  443/tcp   open  https
  51337/tcp open  unknown

  Nmap done: 1 IP address (1 host up) scanned in 14.13 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80,443,51337 10.10.28.42
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-20 05:04 EST
  Nmap scan report for 10.10.28.42
  Host is up (0.042s latency).

  PORT      STATE SERVICE  VERSION
  22/tcp    open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 08:d9:c8:27:bc:25:ac:20:18:38:da:b7:bb:dd:12:17 (RSA)
  |   256 e9:9c:bd:70:39:8e:86:d0:30:aa:83:49:be:5b:b7:3c (ECDSA)
  |_  256 14:ec:b2:de:68:df:bb:89:75:ba:48:cf:b2:2c:b2:2c (ED25519)
  80/tcp    open  http     Apache httpd 2.4.41 ((Ubuntu))
  |_http-title: Apache2 Ubuntu Default Page: It works
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  443/tcp   open  ssl/http Apache httpd 2.4.41
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  | tls-alpn: 
  |_  http/1.1
  | ssl-cert: Subject: commonName=grep.thm/organizationName=SearchME/stateOrProvinceName=Some-State/countryName=US
  | Not valid before: 2023-06-14T13:03:09
  |_Not valid after:  2024-06-13T13:03:09
  |_ssl-date: TLS randomness does not represent time
  |_http-title: 403 Forbidden
  51337/tcp open  http     Apache httpd 2.4.41
  |_http-title: 400 Bad Request
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  Service Info: Host: ip-10-10-28-42.eu-west-1.compute.internal; OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/
  Nmap done: 1 IP address (1 host up) scanned in 26.93 seconds
```
From here I proceeded the analysis port by port.

## Port 443 Analysis

I started by checking the certificate and found a domain name `grep.thm`:

![Certifcate domain](/assets/img/tryhackme/Grep/thm_grep_1.jpg)

After adding it into my `hosts` file I was able to access the webpage. The website had two options `Register` and `Login`, so I went for `Register`. After I inserted something random I got an error message:

![Original API key](/assets/img/tryhackme/Grep/thm_grep_2.jpg)

Since projects are mostly hosted on github I searched for the `SearchME` and found a repository that appeared to be related to the CTF. I examined `register.php` for modifications, where the valid API key was likely stored. After a while I found the valid key:

![Registration Process](/assets/img/tryhackme/Grep/thm_grep_3.jpg)

By intercepting the registration process, I was able to modify the API key, successfully register, and retrieve the flag. I continued with a directory enumeration to gain more insights into the application:

```bash
$ gobuster dir --no-tls-validation --url https://grep.thm/public/html/ --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x html,php,txt
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     https://grep.thm/public/html/
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Extensions:              html,php,txt
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.html                (Status: 403) [Size: 274]
  /.php                 (Status: 403) [Size: 274]
  /index.php            (Status: 200) [Size: 1471]
  /login.php            (Status: 200) [Size: 1981]
  /register.php         (Status: 200) [Size: 2346]
  /admin.php            (Status: 403) [Size: 0]
  /upload.php           (Status: 403) [Size: 0]
  /logout.php           (Status: 200) [Size: 154]
  /dashboard.php        (Status: 403) [Size: 0]
```

The `/upload.php` endpoint seemed interesting, but when I tried to upload a PHP reverse shell I, encountered the following error message:

```json
{"error":"Invalid file type. Only JPG, JPEG, PNG, and BMP files are allowed."}
```

To bypass the filter I changed the magic numbers of the php reverse shell using <a href="https://hexed.it/">hexed.it</a> to look like a JPG file:

```bash
$ file shell.php
shell.php: JPEG image data
$ xxd shell.php | head
00000000: ffd8 ffe0 0010 4a46 3c3f 7068 700a 2f2f  ......JF<?php.//
00000010: 2070 6870 2d72 6576 6572 7365 2d73 6865   php-reverse-she
00000020: 6c6c 202d 2041 2052 6576 6572 7365 2053  ll - A Reverse S
```

After uploading the file again the response looks good: 

```json
{"message":"File uploaded successfully."}
```

Through manual exploration, I located the upload directory and the uploaded PHP file at:

```html
https://grep.thm/api/uploads/
```

I started the reverse shell and received the connection. Since a flag wasn't required for this CTF I proceeded in the website directory. My focus was on database credentials or similar information, as I needed to find the admin's email and password. In the website directory I found a backup directory with a sql script:

```bash
www-data@ip-10-10-8-62:/var/www/backup$ ls
	users.sql
```

In the script, I found an interesting section:

```sql
[...]
INSERT INTO `users` (`id`, `username`, `password`, `email`, `name`, `role`) VALUES
(1, 'test', '$2y$10$dE6VAdZJCN4repNAFdsO2ePDr3StRdOhUJ1O/41XVQg91qBEBQU3G', 'test@grep.thm', 'Test User', 'user'),
(2, 'admin', '[removed]', '[removed]', 'Admin User', 'admin');
[...]
```

I saved the admin password hash to a file and ran Hashcat to crack it. After some time, I aborted the process and moved on. I checked the last port which I didn't had a look on. In the certificate was another URL called: `leakchecker.grep.thm`. After adding the URL to the hosts file, I was able to access the site. The application asked for an email adress. I entered the previously found email and retrieved the password:

![Leakchecker](/assets/img/tryhackme/Grep/thm_grep_4.jpg)

solved! :)
