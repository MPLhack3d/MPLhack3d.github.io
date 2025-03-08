---
title: "Surfer"
date: 2025-02-10
image: /assets/img/tryhackme/Surfer/Surfer_image.jpg
description: Writeup of the TryHackMe-CTF Surfer
categories: [Tryhackme, Easy]
tags: [linux, web, ssrf]
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
    <td>Surfer</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Surf some internal webpages to find the flag!</td>
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
    <td><a href="https://tryhackme.com/room/surfer">https://tryhackme.com/room/surfer</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Surfer.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.11.215 
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-10 11:19 EST
  Nmap scan report for 10.10.11.215
  Host is up (0.045s latency).
  Not shown: 65533 closed tcp ports (conn-refused)
  PORT   STATE SERVICE
  22/tcp open  ssh
  80/tcp open  http

  Nmap done: 1 IP address (1 host up) scanned in 39.58 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80 10.10.11.215
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-10 11:20 EST
  Nmap scan report for 10.10.11.215
  Host is up (0.041s latency).

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 43:a2:1e:75:89:99:23:72:cc:8f:36:d5:ef:3a:0c:41 (RSA)
  |   256 4c:1d:74:2f:75:14:54:30:52:80:ae:1f:10:e6:8b:80 (ECDSA)
  |_  256 3f:43:94:5c:b2:41:12:ab:1f:c2:9c:5c:85:be:16:71 (ED25519)
  80/tcp open  http    Apache httpd 2.4.38 ((Debian))
  | http-title: 24X7 System+
  |_Requested resource was /login.php
  |_http-server-header: Apache/2.4.38 (Debian)
  | http-cookie-flags: 
  |   /: 
  |     PHPSESSID: 
  |_      httponly flag not set
  | http-robots.txt: 1 disallowed entry 
  |_/backup/chat.txt
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 8.12 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

I found a website with a login page and proceeded with a gobuster scan.

![login page](/assets/img/tryhackme/Surfer/thm_surfer_1.jpg)

```bash
  $ gobuster dir --url http://10.10.11.215/ --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt 
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.11.215/
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /assets               (Status: 301) [Size: 313] [--> http://10.10.11.215/assets/]
  /vendor               (Status: 301) [Size: 313] [--> http://10.10.11.215/vendor/]
  /backup               (Status: 301) [Size: 313] [--> http://10.10.11.215/backup/]
  /internal             (Status: 301) [Size: 315] [--> http://10.10.11.215/internal/]
  /server-status        (Status: 403) [Size: 277]
```

While `gobuster` was searching for directories, I manually enumerate the website. I discovered a `robots.txt` with an interesting hint:
```html
User-Agent: *
Disallow: /backup/chat.txt
```
The `chat.txt` file contained the following conversation:
```text
Admin: I have finished setting up the new export2pdf tool.
Kate: Thanks, we will require daily system reports in pdf format.
Admin: Yes, I am updated about that.
Kate: Have you finished adding the internal server.
Admin: Yes, it should be serving flag from now.
Kate: Also Don't forget to change the creds, plz stop using your username as password.
Kate: Hello.. ?
```

From this, I gathered three important pieces of information:
1. there is some export2pdf tool
2. some "internal server" is hosting the flag
3. using the username as both username and password could help us log in

I then attempted to log in to the website using this information. It was straightforward, thanks to the details found in the chat file. Once logged in, I reached the dashboard, which revealed two interesting features:

![Dashboard](/assets/img/tryhackme/Surfer/thm_surfer_2.jpg)

Directly accessing `http://10.10.11.215/internal/admin.php` resulted in the following error:
```text
This page can only be accessed locally.
```
Next, I clicked on the Export to PDF button and intercepted the request using `Burp Suite`. As suspected, the parameter sent to `export2pdf.php` was named url and had the following default value:

```html
url=http%3A%2F%2F127.0.0.1%2Fserver-info.php
```

I modified the payload to retrieve the flag using the same URL encoding format and successfully obtained the following PDF:

![Flag](/assets/img/tryhackme/Surfer/thm_surfer_3.jpg)

solved! :)
