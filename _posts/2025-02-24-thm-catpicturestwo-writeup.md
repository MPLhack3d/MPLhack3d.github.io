---
title: "Cat Pictures 2"
date: 2025-02-24
image: /assets/img/tryhackme/Catpictures2/Catpictures2_image.jpg
description: Writeup of the TryHackMe-CTF Cat Pictures 2
categories: [TryHackMe, Easy]
tags: [linux, web, privesc]
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
    <td>Cat Pictures 2</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Now with more Cat Pictures!</td>
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
    <td><a href="https://tryhackme.com/room/catpictures2">https://tryhackme.com/room/catpictures2</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Cat Pictures 2.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.187.32       
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-24 14:34 EST
  Nmap scan report for 10.10.187.32
  Host is up (0.044s latency).
  Not shown: 65529 closed tcp ports (conn-refused)
  PORT     STATE SERVICE
  22/tcp   open  ssh
  80/tcp   open  http
  222/tcp  open  rsh-spx
  1337/tcp open  waste
  3000/tcp open  ppp
  8080/tcp open  http-proxy

  Nmap done: 1 IP address (1 host up) scanned in 27.07 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80,222,1337,3000,8080 10.10.187.32
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-24 14:36 EST
  Nmap scan report for 10.10.187.32
  Host is up (0.039s latency).

  PORT     STATE SERVICE VERSION
  22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 33:f0:03:36:26:36:8c:2f:88:95:2c:ac:c3:bc:64:65 (RSA)
  |   256 4f:f3:b3:f2:6e:03:91:b2:7c:c0:53:d5:d4:03:88:46 (ECDSA)
  |_  256 13:7c:47:8b:6f:f8:f4:6b:42:9a:f2:d5:3d:34:13:52 (ED25519)
  80/tcp   open  http    nginx 1.4.6 (Ubuntu)
  | http-robots.txt: 7 disallowed entries 
  |_/data/ /dist/ /docs/ /php/ /plugins/ /src/ /uploads/
  |_http-server-header: nginx/1.4.6 (Ubuntu)
  |_http-title: Lychee
  | http-git: 
  |   10.10.187.32:80/.git/
  |     Git repository found!
  |     Repository description: Unnamed repository; edit this file 'description' to name the...
  |     Remotes:
  |       https://github.com/electerious/Lychee.git
  |_    Project type: PHP application (guessed from .gitignore)
  222/tcp  open  ssh     OpenSSH 9.0 (protocol 2.0)
  | ssh-hostkey: 
  |   256 be:cb:06:1f:33:0f:60:06:a0:5a:06:bf:06:53:33:c0 (ECDSA)
  |_  256 9f:07:98:92:6e:fd:2c:2d:b0:93:fa:fe:e8:95:0c:37 (ED25519)
  1337/tcp open  waste?
  | fingerprint-strings: 
  |   GenericLines: 
  |     HTTP/1.1 400 Bad Request
  |     Content-Type: text/plain; charset=utf-8
  |     Connection: close
  |     Request
  |   GetRequest, HTTPOptions: 
  |     HTTP/1.0 200 OK
  |     Accept-Ranges: bytes
  |     Content-Length: 3858
  |     Content-Type: text/html; charset=utf-8
  |     Date: Mon, 24 Feb 2025 19:36:18 GMT
  |     Last-Modified: Wed, 19 Oct 2022 15:30:49 GMT
  |     <!DOCTYPE html>
  |     <html>
  |     <head>
  |     <meta name="viewport" content="width=device-width, initial-scale=1.0">
  |     <title>OliveTin</title>
  |     <link rel = "stylesheet" type = "text/css" href = "style.css" />
  |     <link rel = "shortcut icon" type = "image/png" href = "OliveTinLogo.png" />
  |     <link rel = "apple-touch-icon" sizes="57x57" href="OliveTinLogo-57px.png" />
  |     <link rel = "apple-touch-icon" sizes="120x120" href="OliveTinLogo-120px.png" />
  |     <link rel = "apple-touch-icon" sizes="180x180" href="OliveTinLogo-180px.png" />
  |     </head>
  |     <body>
  |     <main title = "main content">
  |     <fieldset id = "section-switcher" title = "Sections">
  |     <button id = "showActions">Actions</button>
  |_    <button id = "showLogs">Logs</but
  3000/tcp open  ppp?
  | fingerprint-strings: 
  |   GenericLines, Help, RTSPRequest: 
  |     HTTP/1.1 400 Bad Request
  |     Content-Type: text/plain; charset=utf-8
  |     Connection: close
  |     Request
  |   GetRequest: 
  |     HTTP/1.0 200 OK
  |     Cache-Control: no-store, no-transform
  |     Content-Type: text/html; charset=UTF-8
  |     Set-Cookie: i_like_gitea=112876139c098de2; Path=/; HttpOnly; SameSite=Lax
  |     Set-Cookie: _csrf=-OkSa4n9_rP8YBvKdDZuhp8DC506MTc0MDQyNTc3Nzk5MTc0MDg4MQ; Path=/; Expires=Tue, 25 Feb 2025 19:36:17 GMT; HttpOnly; SameSite=Lax
  |     Set-Cookie: macaron_flash=; Path=/; Max-Age=0; HttpOnly; SameSite=Lax
  |     X-Frame-Options: SAMEORIGIN
  |     Date: Mon, 24 Feb 2025 19:36:17 GMT
  |     <!DOCTYPE html>
  |     <html lang="en-US" class="theme-">
  |     <head>
  |     <meta charset="utf-8">
  |     <meta name="viewport" content="width=device-width, initial-scale=1">
  |     <title> Gitea: Git with a cup of tea</title>
  |     <link rel="manifest" href="data:application/json;base64,eyJuYW1lIjoiR2l0ZWE6IEdpdCB3aXRoIGEgY3VwIG9mIHRlYSIsInNob3J0X25hbWUiOiJHaXRlYTogR2l0IHdpdGggYSBjdXAgb2YgdGVhIiwic3RhcnRfdXJsIjoiaHR0cDovL2xvY2FsaG9zdDozMDAwLyIsImljb25zIjpbeyJzcmMiOiJodHRwOi
  |   HTTPOptions: 
  |     HTTP/1.0 405 Method Not Allowed
  |     Cache-Control: no-store, no-transform
  |     Set-Cookie: i_like_gitea=c21834adbec9e0ee; Path=/; HttpOnly; SameSite=Lax
  |     Set-Cookie: _csrf=LKAv4JVpwlAIX2xuAxuceNJsYI06MTc0MDQyNTc4MzMzNzYxNzc4Nw; Path=/; Expires=Tue, 25 Feb 2025 19:36:23 GMT; HttpOnly; SameSite=Lax
  |     Set-Cookie: macaron_flash=; Path=/; Max-Age=0; HttpOnly; SameSite=Lax
  |     X-Frame-Options: SAMEORIGIN
  |     Date: Mon, 24 Feb 2025 19:36:23 GMT
  |_    Content-Length: 0
  8080/tcp open  http    SimpleHTTPServer 0.6 (Python 3.6.9)
  |_http-title: Welcome to nginx!
  |_http-server-header: SimpleHTTP/0.6 Python/3.6.9
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

The website was a gallery that featured different cat pictures:

![Cat gallery](/assets/img/tryhackme/Catpictures2/thm_catpictures2_1.jpg)

After I downloaded them, it was intersting that all had the `jpeg` extension, except for one, which had a `jpg` extension. I started analyzing the `jpg` and found some hidden metadata:
```bash
$ exiftool f5054e97620f168c7b5088c85ab1d6e4.jpg
  [...]
  Title  : :8080/764efa883dda1e11db47671c4a3bbd9e.txt
  [...]
```

## Port 8080 Analysis

The website provided a default Nginx installation. Given the previous hint, I navigated to the `txt` file:
```text
note to self:

I setup an internal gitea instance to start using IaC for this server. It's at a quite basic state, but I'm putting the password here because I will definitely forget.
This file isn't easy to find anyway unless you have the correct url...

gitea: port 3000
user: [remove]
password: [remove]

ansible runner (olivetin): port 1337
```

## Port 3000 and 1337 Analysis

Because of the hint, I was able to log in to the `gitea` instance, which revealed the version number. First, I searched for exploits related to this version number, but didn't find anything useful. I checked `gitea` and was able to modify files. The Gitea repository referenced `ansible`, which is why I checked port 1337 and discoverd a working instance:

![Gitea version](/assets/img/tryhackme/Catpictures2/thm_catpictures2_2.jpg)

After testing the functionality, I discovered that when running the playbook, the code inside the Gitea instance was executed. Therefore, I was able to change the command parameter and obtain a reverse shell:

![Ansible Dashboard](/assets/img/tryhackme/Catpictures2/thm_catpictures2_3.jpg)

![Gitea repository](/assets/img/tryhackme/Catpictures2/thm_catpictures2_4.jpg)

After receiving the connection, I was able to retrieve the second flag:
```bash
bismuth@catpictures-ii:~$ ls
  flag2.txt
bismuth@catpictures-ii:~$ cat flag2.txt 
  [remove]
```

## Privilege Escalation root

I started by searching for privilege escalation methods and found a potentially vulnerable sudo version. I performed some research and found this <a href="https://github.com/worawit/CVE-2021-3156/blob/main/exploit_nss.py">Github</a> repository with a PoC exploit. After executing the Python script, I was able to retrieve the last flag:
```bash
bismuth@catpictures-ii:~$ python3 exploit.py 
# whoami
  root
# cd /root
# ls
  ansible  docker-compose.yaml  flag3.txt  gitea
# cat flag3.txt
  [remove]
```

solved! :)
