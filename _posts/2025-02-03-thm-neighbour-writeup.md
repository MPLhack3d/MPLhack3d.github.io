---
title: "Neighbour"
date: 2025-02-03
image: /assets/img/general/CTFgeneral_image.jpg
description: Writeup of the TryHackMe-CTF Neighbour
categories: [TryHackMe, Easy]
tags: [linux, web, idor]
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
    <td>Neighbour</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Check out our new cloud service, Authentication Anywhere. Can you find other user's secrets?</td>
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
    <td><a href="https://tryhackme.com/room/neighbour">https://tryhackme.com/room/neighbour</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Neighbour.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.118.17
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-03 14:02 EST
  Nmap scan report for 10.10.118.17
  Host is up (0.046s latency).
  Not shown: 65533 closed tcp ports (conn-refused)
  PORT   STATE SERVICE
  22/tcp open  ssh
  80/tcp open  http

  Nmap done: 1 IP address (1 host up) scanned in 19.55 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80 10.10.118.17
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-03 14:03 EST
  Nmap scan report for 10.10.118.17
  Host is up (0.038s latency).

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 25:1b:e7:4c:70:43:ae:61:3c:51:94:ed:a8:68:e3:1f (RSA)
  |   256 dc:6b:bc:31:b1:62:be:bd:37:0f:9b:e1:30:ed:bb:9c (ECDSA)
  |_  256 d4:43:76:4f:1b:35:d1:e5:6c:b7:4e:74:2c:ba:cd:66 (ED25519)
  80/tcp open  http    Apache httpd 2.4.53 ((Debian))
  |_http-title: Login
  |_http-server-header: Apache/2.4.53 (Debian)
  | http-cookie-flags: 
  |   /: 
  |     PHPSESSID: 
  |_      httponly flag not set
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 8.03 seconds
```
From here I proceeded the analysis with port 80.

## Port 80 Analysis

The web server provided a login page:

![Login](/assets/img/tryhackme/Neighbour/thm_neighbour_1.jpg)

Below the login field, there was a hint:
```text
Don't have an account? Use the guest account! (Ctrl+U)
```

The `ctrl + u` combination opens the page source. In the source, there was a very useful comment:
```html
<!-- use guest:guest credentials until registration is fixed. "admin" user account is off limits!!!!! -->
```

I was able to login, but while the website itself wasn't that interesting, the URL caught my attention:
![Login](/assets/img/tryhackme/Neighbour/thm_neighbour_2.jpg)

This appeared to be an Insecure Direct Object Reference (IDOR). I was able to exploit this by changing the URL from:
```html
/profile.php?user=guest
```
to
```html
/profile.php?user=admin
```

and the flag was displayed:
```html
<h1 class="my-5">Hi, <b>admin</b>. Welcome to your site. The flag is: flag{[removed]}</h1>
```

solved! :)